## 安装
```shell
sudo apt-get install xdotool
```

## 命令
鼠标移动
```shell
xdotool mousemove x y
```
鼠标点击
```shell
xdotool click [num] 
      1	鼠标左键
      2	鼠标中键
      3	鼠标右键
      4	滚轮向前
      5	滚轮向后
```
需要执行多个命令时，可以将命令写到shell文件中
```shell
xdotool [filename]
# 想要连接两个键，可以在它们之间使用“+”操作符
xdotool key [name of the key] 
# 输入
xdotool type  
# 在打开的窗口中搜索对应名称的窗口，并聚焦于该窗口，然后模拟击键
xdotool search --name [name of the window] key [keys to press] 
```
键盘相关操作：
```shell
# 模拟回车
xdotool key KP_Enter
# 模拟按下左侧shift键(不弹起)
xdotool keydown Shift_L
# 模拟弹起shift
xdotool keyup Shift_L
# 模拟按下A并弹起
xdotool key a
# 模拟ctrl+a
xdotool key Ctrl+a
# 模拟按下向上方向键
xdotool key Up
```
鼠标相关操作：
```shell
# 获取鼠标位置
xdotool getmouselocation
# 模拟鼠标移动
xdotool mousemove X坐标值 Y坐标值
# 模拟鼠标左键点击1次
xdotool click 1
# 模拟鼠标中键（滚轮键）点击1次
xdotool click 2
# 模拟鼠标右键点击1次
xdotool click 3
```
拖拽示例：
```shell
xdotool mousedown 1
sleep 0.5
xdotool mousemove_relative --sync 200 200
sleep 0.5
xdotool mouseup 1
```

## Chrome
```shell
xdotool search "google chrome" windowactivate # 需要窗口管理器的支持，xdotool 会遍历所有窗口，一旦发现标题栏内容包含指定的关键词，就返回该窗口。如果没有符合规则的窗口，那么什么也不会发生。
xdotool search "google chrome" windowsize 640 480 # 指定chrome浏览器的大小
xdotool search "google chrome" windowactivate --sync windowmove 50 50 # 将浏览器移动到指定位置
xdotool search "google chrome" windowactivate --sync mousemove --window %1 0 0 # 将鼠标移动到窗口的左上角
```