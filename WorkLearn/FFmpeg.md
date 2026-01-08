# FFmpeg知识点
## 使用FFmpeg合并音视频文件
```
ffmpeg -i video.m4s -i audio.m4s -c:v copy -c:a aac -strict experimental -shortest final.mp4
```

## FFmpeg播放rtsp视频流
```
ffplay -rtsp_transport tcp rtsp://admin:12345@192.168.1.64:554/h264/ch1/sub/av_stream
```