## 连接

```shell
# 重置连接(USB访问) 设备重启后使用
adb usb
```

## 网络
```shell
# 设置代理
adb shell settings put global http_proxy <address>:<port>
```

## 进程
```shell
# 启动
adb shell am start -n com.android.calendar/com.android.calendar.LaunchActivity
adb -s <UDID> shell am start -n com.google.android.contacts/com.android.contacts.activities.PeopleActivity
# 清除数据
adb shell pm clear com.test.abc
# 删除进程
adb shell am kill <package_name>
```

### 内存管理
```shell
adb shell top
adb shell busybox free -m
```

## 用户交互
返回桌面
```shell
am start -a android.intent.action.MAIN -c android.intent.category.HOME
# OR
adb shell input keyevent KEYCODE_HOME
```

## 文件传输
```shell
adb -s <UDID> push Desktop/ca.crt /data/local/tmp
```