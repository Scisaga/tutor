## 


## 日志
查看容器日志文件路径：
```sh
docker inspect --format='{{.LogPath}}' ${container-name}
```
清空容器日志：
```sh
$ sudo sh -c 'echo "" > $(docker inspect --format="{{.LogPath}}" ${container-name})'
```
检视容器日志：
```sh
# 最新100条
docker logs ${container-name} --tail 100
# 最近1小时
docker logs ${container-name} --since 1h
# 最近一小时中的最新100条
docker logs ${container-name} --tail 100 --since 1h
```
