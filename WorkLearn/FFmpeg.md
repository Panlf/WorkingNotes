# FFmpeg知识点
## 使用FFmpeg合并音视频文件
```
ffmpeg -i video.m4s -i audio.m4s -c:v copy -c:a aac -strict experimental -shortest final.mp4
```