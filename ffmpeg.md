# ffmpeg使用指南

## 提取视频

```shell
ffmpeg -i input.mp4 -vcodec copy -an output.mp4
# -an: 去掉音频；-vcodec:视频选项，一般后面加copy表示拷贝
```

## 提取音频

```shell
ffmpeg -i input.mp4 -acodec copy -vn output.mp3
# -vn: 去掉视频；-acodec: 音频选项， 一般后面加copy表示拷贝

ffmpeg -i source_video.avi -vn -ar 44100 -ac 2 -ab 192 -f mp3 sound.mp3
# 192kb/s
```

## 音频转换
```shell
ffmpeg -i 1.mp3 -ac 1 -ar 16000 1.wav
# 转换成 16khz wav 音频
```

## 视频音频合成
```shell
ffmpeg -y –i input.mp4 –i input.mp3 –vcodec copy –acodec copy output.mp4
# -y 覆盖输出文件

ffmpeg -i video.mp4 -i genaudio.mp3 -map 0:v -map 1:a -c:v copy -c:a copy output.mp4 -y
# pick stream
```

## 视频分割
```shell
ffmpeg -ss 00:00:00 -t 00:00:30 -i test.mp4 -vcodec copy -acodec copy output.mp4

ffmpeg -ss 00:00:00 -i sample.mp4 -c copy -t 600 dst.mp4

ffmpeg -i somefile.mp3 -f segment -segment_time 3 -c copy out%03d.mp3
```
参数说明：
* -ss：指定从什么时间开始
* -t：指定需要截取多长时间
* -i：指定输入文件
* -segment_time：每个文件的时间长度（单位秒）

## 视频拼接
设定待合并视频文件`list.txt`
```txt
file ./1.mp4
file ./2.mp4
```

```shell
ffmpeg -f concat -i list.txt -c copy concat.mp4
```

## 提取关键帧
```shell
ffmpeg -i 1.mp4 -an -vf select='eq(pict_type\,I)' -vsync 2 -s 1920*1080 -f image2 dstPath/image-%02d.jpg
```
参数说明：
* -i：输入文件，这里的话其实就是视频；
* -vf：是一个命令行，表示过滤图形的描述；选择过滤器select会选择帧进行输出，pict_type和对应的类型，PICT_TYPE_I 表示是I帧，即关键帧；
* -vsync 2：阻止每个关键帧产生多余的拷贝；
* -f image2 image-%02d.jpeg：将视频帧写入到图片中，样式的格式一般是: “%d” 或者 “%0Nd”
* -s：分辨率

## 合并字幕文件到mp4
```shell
ffmpeg -i sample_video_ffmpeg.mp4 \
  -f srt -i sample_video_subtitle_ffmpeg.srt \
  -c copy -c:s mov_text \
  -metadata:s:s:0 language=eng \
  output_soft_english.mp4
```

## 去掉输出信息
```shell
ffmpeg -hide_banner -loglevel error
# 用于shell调用
```
ref:
- https://www.baeldung.com/linux/subtitles-ffmpeg