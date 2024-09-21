## 使用LetsEncrypt申请泛域名证书

```
snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot
certbot certonly --manual -d *.<domain> -d <domain> --server https://acme-v02.api.letsencrypt.org/directory
```

参考：
1. https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal
2. https://blog.csdn.net/wc810267705/article/details/79917688

## Nginx 配置
说明：
- 一个域名，两个子域名，分别逆向代理到内网服务
- 配置文件位置: /etc/nginx/sites-available/<domain>
- 启用时，创建软连接到: /etc/nginx/sites-enabled

```nginx configuration
server {
    listen *:80;
    server_name <domain> <domain_1> <domain_2>;
    server_tokens off;
    
    root /var/www/html;

    # http --> https
    location / {
        return 301 https://$host$request_uri;
        rewrite ^(.*) https://$server_name$1 permanent;
    }

    location /.well-known/ {
    }
}

server {

    root /dev/null;
    server_name <domain_1>;
    charset UTF-8;
    access_log /var/log/nginx/<domain_1>.access.log;
    error_log /var/log/nginx/<domain_1>.error.log;
    
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;

    ssl_protocols SSLv2 SSLv3 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_session_timeout 5m;

    client_max_body_size 0;
    chunked_transfer_encoding on;
    
    gzip on;

    location / {

        proxy_read_timeout      300;
        proxy_connect_timeout   300;
        proxy_redirect      off;

        proxy_http_version 1.1;

        proxy_set_header    Host        $http_host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto   $scheme;

        proxy_pass http://<host_1>:<port_1>;
    }
}

server {

    root /dev/null;
    server_name <domain_2>;
    charset UTF-8;
    access_log /var/log/nginx/<domain_2>.access.log;
    error_log /var/log/nginx/<domain_2>.error.log;

    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
    
    ssl_protocols SSLv2 SSLv3 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_session_timeout  5m;

    client_max_body_size    0;
    chunked_transfer_encoding   on;

    location / {
        proxy_set_header  Host          $http_host;   # required for docker client's sake
        proxy_set_header  X-Real-IP     $remote_addr; # pass on real client's IP
        proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header  X-Forwarded-Proto $scheme;
        proxy_read_timeout          900;
        proxy_pass http://<host_2>:<port_2>;
    }
}
```