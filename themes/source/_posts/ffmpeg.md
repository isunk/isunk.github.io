---
title: FFmpeg
tags: ffmpeg
categories: ffmpeg
---

# [FFmpeg](http://ffmpeg.org/)

## 参数说明

### 基本选项

| 参数名 | 参数说明 | 参数示例 |
| --- | --- | --- |
| -i        | 指定输入视频文件  | -i input.mp4 |
| -y        | 覆盖已有文件      | -y |

### 视频选项

| 参数名 | 参数说明 | 参数示例 |
| --- | --- | --- |
| -vcodec   | 视频编码格式，取值："libx264"（libx264 解码器）、"h264"（H.264 编码） | -vcodec libx264 |
| -s        | 视频分辨率 | -s 540x960 |
| -r        | 视频帧速率，单位：帧/秒 | -r 25 |
| -b        | 视频码率，单位：bit/s 或 bps | -b 800000 |

### 音频选项

| 参数名 | 参数说明 | 参数示例 |
| --- | --- | --- |
| -acodec   | 音频编码，取值："aac"（AAC 编码） | -acodec aac |
| -ac       | 音频声道个数，取值："1"（单声道）、"2"（立体声） | -ac 2 |
| -ar       | 音频采样频率，单位：Hz | -ar 44100 |
| -b:a（旧版本写法：-ab） | 音频比特率，单位：bit/s 或 bps，一般选择 32、64、96、128 | -b:a 128 或 -ab 128 |

- 视频文件大小计算公式  
    ```bash
    视频文件大小（byte） ≈ 总比特率（bps） / 8 * 播放时长（s） = (视频码率（bps） + 音频码率（bps）) / 8 * 播放时长（s）
    # 例如，某视频文件，总比特率为 862 kbps（其中视频码率为 734 kbps，音频码率为 128 kbps），时长为 00:00:15，则文件大小约为 862 kbps / 8 * 15 s = 107.75 KB * 15 s = 1616.25 KB
    ```

### 常用命令

```bash
# 查看视频信息
ffmpeg -i input.mp4

# 视频压缩
# ffmpeg -i input.mp4 -vcodec libx264 -s 540x480 -r 25 -b 800000 -acodec aac -ac 2 -ar 44100 -ab 128 -y output.mp4
ffmpeg -i input.mp4 -vcodec libx264 -r 25 -b 800000 -acodec aac -ac 2 -ar 44100 -ab 128 -y output.mp4

# 视频压缩，并推送至流媒体服务器
ffmpeg \
    -i input.mp4 \
    -vcodec libx264 -r 25 -vf scale=-1:720 \ # 视频分辨率 720p，帧率 25 fps
    # -acodec aac -ac 2 -ar 44100 -b:a 128k \
    -acodec copy \
    -f flv rtmp://127.0.0.1:1935/

# 视频裁剪、压缩
ffmpeg \
    -i "input.mp4" \
    -ss 00:01:00 -t 00:01:00 \ # 从 01:00 开始，截取长度为 01:00 时间的视频
    # -vcodec copy \ # 保持原分辨率
    -vf scale=-1:480 \ # 等比例压缩至 *x480 分辨率
    -acodec copy \
    output.mp4

# 循环推流、[添加时间水印](https://blog.csdn.net/ternence_hsu/article/details/102540626)
ffmpeg \
    -re \ # 表示按帧率发送，推流时一定要加这个参数，否则 ffmpeg 会以最高速率不断发送
    -stream_loop -1 \ # 循环推流次数，-1 表示无限次数
    -i test.h264 \
    # -vcodec copy -acodec copy \
    -vf "settb=AVTB,setpts='trunc(PTS/1K)*1K+st(1,trunc(RTCTIME/1K))-1K*trunc(ld(1)/1K)',drawtext=fontsize=20:fontcolor=white:text='%{localtime}.%{eif\:1M*t-1K*trunc(t*1K)\:d}'" -c:v libx264 -an \ # 添加本地时间水印,打印详细日期并且精确到毫秒
    -f flv \
    -y rtmp://127.0.0.1:1935/live/test

# 将图片合成视频
ffmpeg \
    -loop 1 \ # 循环读取输入文件
    -i input.jpg \ # 指定输入文件
    -t 15 \ # 生成 15 秒的视频
    output.mp4
ffmpeg \
    -f image2 \
    -r 0.2 \ # 输入帧率(在 -i 之前为输入帧率，默认为 25)，如 0.2 表示每秒读取 0.2 张图片，即每张图片播放 5 秒
    -i input-%d.jpg \ # 读取图片如 input-0.jpg, input-1.jpg, input-2.jpg
    -r 10 \ # 输出帧率(在 -i 之前为输入帧率，默认为 25)，即视频帧率
    output.mp4

# 将图片合成视频，实时添加时间水印，并且推流
ffmpeg \
    -re
    -loop 1 \
    -i input.jpg \
    -vf "setpts=PTS,realtime,setpts=PTS,drawtext=fontfile=arial.ttf:x=w-tw:fontcolor=white:fontsize=30:text='%{localtime\:%H\\\:%M\\\:%S}'" \ # 添加时间水印，按照实时速率
    -vcodec h264 \ # 转换成 H.264 视频格式
    -f flv \
    rtmp://127.0.0.1:1935/live/test
```

## 视频转码

```bash
# 将 input.avi 转码为 output.mp4
ffmpeg -i input.avi output.mp4

# 将 input.avi 转码为 output.ts
ffmpeg -i input.mp4 output.ts

# 转码为码流原始文件
ffmpeg –i test.mp4 –vcodec h264 –s 352*278 –an –f m4v test.264
ffmpeg –i test.mp4 –vcodec h264 –bf 0 –g 25 –s 352*278 –an –f m4v test.264

# 转码为封装文件
ffmpeg –i test.avi -vcodec mpeg4 –vtag xvid –qsame test_xvid.avi

# 将input.avi转码成output.ts，并设置视频的码率为640kbps（-bf B帧数目控制，-g 关键帧间隔控制，-s 分辨率控制）
ffmpeg -i input.avi -b:v 640k output.ts
```

## 提取音频、视频

```bash
# 提取音频
ffmpeg -i test.mp4 -acodec copy -vn output.aac # 默认 mp4 的 audio codec 是 aac
ffmpeg -i test.mp4 -acodec aac -vn output.aac # 如果不是 aac 格式，可以都转为 aac 

# 提取视频
ffmpeg -i input.mp4 -vcodec copy -an output.mp4
```

## 视频剪切

```bash
# 从时间为 00:00:15 开始，截取 5 秒钟的视频
ffmpeg -ss 00:00:15 -t 00:00:05 -i input.mp4 -vcodec copy -acodec copy output.mp4
```
> -ss 表示开始切割的时间，-t 表示要切多少

## 码率控制

```bash
# 把码率从原码率（如 10 Mbps）转成 2 Mbps 码率，这样其实也间接让文件变小了
# ffmpeg -i input.mp4 -b:v 2000k output.mp4
ffmpeg -i input.mp4 -b:v 2000k -bufsize 2000k output.mp4
ffmpeg -i input.mp4 -b:v 2000k -bufsize 2000k -maxrate 2500k output.mp4
```

## 视频编码格式转换

```bash
# 比如一个视频的编码是 MPEG4，先转成 H264 编码
ffmpeg -i input.mp4 -vcodec h264 output.mp4
# H264 编码转成 MPEG4
ffmpeg -i input.mp4 -vcodec mpeg4 output.mp4

# # 如果 ffmpeg 编译时，添加了外部的 x265 或者 X264，那也可以用外部的编码器来编码
# ffmpeg -i input.mp4 -c:v libx265 output.mp4
# ffmpeg -i input.mp4 -c:v libx264 output.mp4
```

## 只提取视频 ES 数据

```bash
ffmpeg –i input.mp4 –vcodec copy –an –f m4v output.h264
```

## 过滤器

```bash
# 将输入的 1920 x 1080 缩小到 960 x 540 输出
ffmpeg -i input.mp4 -vf scale=960:540 output.mp4 # 如果540不写，写成-1，即scale=960:-1, 那也是可以的，ffmpeg会通知缩放滤镜在输出时保持原始的宽高比

# 为视频添加 logo
ffmpeg -i input.mp4 -i logo.png -filter_complex overlay output.mp4 # 左上角位置
# ffmpeg -i input.mp4 -i logo.png -filter_complex overlay=W-w output.mp4 # 右上角位置
# ffmpeg -i input.mp4 -i logo.png -filter_complex overlay=0:H-h output.mp4 # 左下角位置
# ffmpeg -i input.mp4 -i logo.png -filter_complex overlay=W-w:H-h output.mp4 # 右下角位置

# 去掉视频的 logo
ffmpeg -i input.mp4 -vf delogo=0:0:220:90:100:1 output.mp4 
# 语法：-vf delogo=x:y:w:h[:t[:show]]
# x:y 离左上角的坐标
# w:h logo 的宽和高
# t: 矩形边缘的厚度默认值 4
# show：若设置为 1 有一个绿色的矩形，默认值 0
```

## 截取视频图像

```bash
ffmpeg -i input.mp4 -r 1 -q:v 2 -f image2 pic-%03d.jpeg

# 从 input.mp4 的第 20s 时间开始，往下 10s，即 20 ~ 30s 这 10 秒钟之间，每隔 1s 就抓一帧，总共会抓 10 帧
ffmpeg -i input.mp4 -ss 00:00:20 -t 10 -r 1 -q:v 2 -f image2 pic-%03d.jpeg 
```

> -r 表示每一秒几帧  
> -q:v 表示存储 jpeg 的图像质量，一般 2 是高质量。

## 序列帧与视频的相互转换

```bash
# 把 darkdoor.[001-100].jpg 序列帧和 001.mp3 音频文件利用 mpeg4 编码方式合成视频文件 darkdoor.avi
ffmpeg -i 001.mp3 -i darkdoor.%3d.jpg -s 1024x768 -author fy -vcodec mpeg4 darkdoor.avi

# 还可以把视频文件导出成 jpg 序列帧
ffmpeg -i bc-cinematic-en.avi example.%d.jpg
```

## 抓屏

```bash
# 最简单的抓屏
ffmpeg -f gdigrab -i desktop out.mpg

# 以 15 的帧率抓屏 20 秒，保存为视频格式是 vp9 的 mkv 文件
ffmpeg -f gdigrab -t 20 -r 15 -i desktop -vcodec libvpx-vp9 capture-vp9.mkv

# 以 15 的帧率抓屏 10 秒，保存为视频格式是 vp9，分辨率是 720 x 420 的 mkv 文件
ffmpeg -f gdigrab -t 10 -r 15 -i desktop -vcodec libvpx-vp9 -s 720x420 vp9-720-420.mkv

# 以 15 的帧率抓屏 20 秒，抓屏范围，以点(100, 60)开始，大小 600 x 480，保存为视频格式是 264 的 mp4 文件
ffmpeg -f gdigrab -video_size 600x480 -offset_x 100 -offset_y 60 -t 20 -r 15 -i desktop -vcodec libx264 x264.mp4

# 抓屏 10 秒，并生成 gif 图片
ffmpeg -f gdigrab -t 10 -r 15 -i desktop -pix_fmt rgb24 capture.gif
```

## 其他用法

- 对视频进行TS切片（转化成HLS）

    - 直接切片（hls）  
        `ffmpeg -i video.mp4 -c:v libx264 -c:a aac -strict -2 -f hls -hls_list_size 0 -hls_time 5 video.m3u8`

        > - `-hls_time n` 设置每片的长度，默认值为 2。单位为秒  
        > - `-hls_list_size n` 设置播放列表保存的最多条目，设置为 0 会保存有所片信息，默认值为 5  
        > - `-hls_wrap n` 设置多少片之后开始覆盖，如果设置为 0 则不会覆盖，默认值为 0。这个选项能够避免在磁盘上存储过多的片，而且能够限制写入磁盘的最多的片的数量  
        > - `-hls_start_number n` 设置播放列表中 sequence number 的值为 number，默认值为 0

    - 转码为 ts 文件并生成 m3u8（segment）

        1. 将 mp4 转为完整的 ts  
            `ffmpeg -i out.mp4 -c copy -bsf h264_mp4toannexb output.ts`

        2. 将 ts 切片，并生成 m3u8 文件  
            `ffmpeg -i output.ts -c copy -map 0 -f segment -segment_list playlist.m3u8 -segment_time 10 output%03d.ts`

- 将文件当做直播送至 live  
    `ffmpeg -re -i video.mp4 -c copy -f flv rtmp://127.0.0.1/live/stream_name`  
	`ffmpeg -re -i video.mp4  -vcodec copy -acodec copy -f rtsp rtsp://127.0.0.1/live/stream_name`

- 读取摄像头、话筒的数据
    - 使用 VFW 读取摄像头的数据（采集 10秒，采集设备为 vfwcap 类型设备，第 0 个 vfwcap 采集设备（如果系统有多个 vfw 的视频采集设备，可以通过 -i num 来选择），每秒 8 帧，输出方式为文件，格式为 mp4）  
        `ffmpeg -t 10 -f vfwcap -i 0 -r 8 -f mp4 cap.mp4`

    - 使用 DirectShow 读取摄像头的数据  
        1. <span id="query_device">列出所有设备</span>，查找设备名称（如摄像头为"WebCam SC-0311139N"，话筒为"楹﹀厠椋?(Realtek High Definition Audio)"(ANSI 乱码"楹﹀厠椋?"转 UTF-8 之后为"麦克风")）  
            `ffmpeg -list_devices true -f dshow -i dummy`
        2. 获取摄像头数据  
            - 从摄像头读取数据并编码为 H.264，最后保存成 mycamera.mkv  
                `ffmpeg -f dshow -i video="WebCam SC-0311139N" -vcodec libx264 mycamera.mkv`
            - 获取摄像头数据，编码为 H.264，封装为 UDP 并发送至组播地址  
                `ffmpeg -f dshow -i video="WebCam SC-0311139N" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f h264 udp://233.233.233.223:6666`
            - 获取摄像头数据，编码为 H.264，封装为 RTP 并发送至组播地址  
                `ffmpeg -f dshow -i video="WebCam SC-0311139N" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f rtp rtp://233.233.233.223:6666>test.sdp`
            - 获取摄像头数据，编码为 H.264，并发送至 RTMP 服务器  
                `ffmpeg -f dshow -i video="WebCam SC-0311139N" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f flv rtmp://127.0.0.1/live/livestream`
            - 采集当前摄像头及音频数据，视频 h264、音频 faac 压缩后发布至 RTMP 服务器  
	            `ffmpeg -f dshow -s 640*480 -i video="WebCam SC-0311139N":audio="麦克风 (Realtek High Definition Audio)" -vcodec libx264 -b 600k -acodec libfaac -ab 128k -f flv rtmp://192.168.1.200/live/video`

    - 屏幕录制
        - linux 下使用 x11grab 录制屏幕  
            `ffmpeg -f x11grab -s 1600x900 -r 50 -vcodec libx264 –preset:v ultrafast –tune:v zerolatency -crf 18 -f mpegts udp://localhost:1234`  
        - 将屏幕录制后编码为 H.264 并保存为本地文件（在 Windows 平台下，使用 -dshow 取代 x11grab）  
            `ffmpeg -f dshow -i video="screen-capture-recorder" -r 5 -vcodec libx264 -preset:v ultrafast -tune:v zerolatency MyDesktop.mkv`
        - 将屏幕录制后编码为 H.264 并封装成 UDP 发送到组播地址  
            `ffmpeg -f dshow -i video="screen-capture-recorder" -r 5 -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f h264 udp://233.233.233.223:6666`
        - 将屏幕录制后编码为 H.264 并封装成 RTP 并发送到组播地址  
            `ffmpeg -f dshow -i video="screen-capture-recorder" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f rtp rtp://233.233.233.223:6666>test.sdp`
        - 编码为 H.264，发布 RTMP  
            `ffmpeg -f dshow -i video="WebCam SC-0311139N" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f flv rtmp://127.0.0.1/live/livestream`
    
    - gdigrab 屏幕录制（另一种屏幕录制的方式）  
        简单的抓屏  
        `ffmpeg -f gdigrab -i desktop out.mpg`  
        从屏幕的（10,20）点处开始，抓取 640 x 480 的屏幕，设定帧率为 5  
        `ffmpeg -f gdigrab -framerate 5 -offset_x 10 -offset_y 20 -video_size 640x480 -i desktop out.mpg`
    
    - 录制视频、音频（按 q 键停止录制）
        ```bash
        # 查看可用设备名字，如 "WebCam SC-0311139N"、"麦克风 (Realtek High Definition Audio)"
        # ffmpeg -list_devices true -f dshow -i dummy
        # 查看（视频录制）设备的可选参数
        # ffmpeg -f dshow -list_options true -i video="WebCam SC-0311139N"

        # 录制视频（摄像头）
        ffmpeg -f dshow -rtbufsize 702000k -i video="WebCam SC-0311139N" v-out.mp4 # "WebCam SC-0311139N" 为设备名称

        # 录制声音
        ffmpeg -f dshow -i audio="麦克风 (Realtek High Definition Audio)" a-out.aac # "麦克风 (Realtek High Definition Audio)" 为设备名称
        ffmpeg -f dshow -i audio="@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{2F4C5655-04D4-4202-B399-8515DE44DD04}" a-out.aac # "@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{2F4C5655-04D4-4202-B399-8515DE44DD04}" 为 Alternative name

        # 同时录制声音和视频（摄像头）
        ffmpeg -f dshow -i video="screen-capture-recorder":audio="virtual-audio-capturer" av-out.mp4
        ffmpeg -f gdigrab -video_size 1366x768 -framerate 30 -pixel_format yuv420p -i video="screen-capture-recorder":audio="virtual-audio-capturer" av-out.mp4

        # 同时录制音频和视频（桌面）
        ffmpeg -f gdigrab -i desktop -f dshow -i audio="麦克风 (Realtek High Definition Audio)" av-out.mp4
        ```

- 分离视频音频流
    ```bash
    ffmpeg -i input_file -vcodec copy -an output_file_video　　//分离视频流
    ffmpeg -i input_file -acodec copy -vn output_file_audio　　//分离音频流
    ```

- 视频解复用
    ```bash
    ffmpeg –i test.mp4 –vcodec copy –an –f m4v test.264
    ffmpeg –i test.avi –vcodec copy –an –f m4v test.264
    ```

- 视频封装
    ```bash
	ffmpeg –i video_file –i audio_file –vcodec copy –acodec copy output_file
    ```

- 视频剪切
    ```bash
    ffmpeg –i test.avi –r 1 –f image2 image-%3d.jpeg # 提取图片
    ffmpeg -ss 0:1:30 -t 0:0:20 -i input.avi -vcodec copy -acodec copy output.avi # 剪切视频
    ```
	> -r 提取图像的频率，-ss 开始时间，-t 持续时间

- 视频录制  
	`ffmpeg –i rtsp://192.168.3.205:5555/test –vcodec copy out.avi`

- YUV序列播放  
	`ffplay -f rawvideo -video_size 1920x1080 input.yuv`

- YUV序列转AVI  
	`ffmpeg –s w*h –pix_fmt yuv420p –i input.yuv –vcodec mpeg4 output.avi`

- 将直播媒体保存至本地文件  
	`ffmpeg -i rtmp://server/live/streamName -c copy dump.flv`

- 将其中一个直播流，视频改用 h264 压缩，音频不变，送至另外一个直播服务流  
	`ffmpeg -i rtmp://server/live/originalStream -c:a copy -c:v libx264 -vpre slow -f flv rtmp://server/live/h264Stream` 

- 将其中一个直播流，视频改用 h264 压缩，音频改用 faac 压缩，送至另外一个直播服务流  
	`ffmpeg -i rtmp://server/live/originalStream -c:a libfaac -ar 44100 -ab 48k -c:v libx264 -vpre slow -vpre baseline -f flv rtmp://server/live/h264Stream`

- 将其中一个直播流，视频不变，音频改用 faac 压缩，送至另外一个直播服务流  
	`ffmpeg -i rtmp://server/live/originalStream -acodec libfaac -ar 44100 -ab 48k -vcodec copy -f flv rtmp://server/live/h264_AAC_Stream`

- 将一个 JPG 图片经过 h264 压缩循环输出为 mp4 视频  
	`ffmpeg.exe -i INPUT.jpg -an -vcodec libx264 -coder 1 -flags +loop -cmp +chroma -subq 10 -qcomp 0.6 -qmin 10 -qmax 51 -qdiff 4 -flags2 +dct8x8 -trellis 2 -partitions +parti8x8+parti4x4 -crf 24 -threads 0 -r 25 -g 25 -y OUTPUT.mp4`

- 将普通流视频改用 h264 压缩，音频不变，送至高清流服务(新版本 FMS live=1)  
	`ffmpeg -i rtmp://server/live/originalStream -c:a copy -c:v libx264 -vpre slow -f flv "rtmp://server/live/h264Stream live=1〃`

- 最简单的抓屏  
	`ffmpeg -f gdigrab -i desktop out.mpg`

- 从屏幕的（10,20）点处开始，抓取 640x480 的屏幕，设定帧率为 5  
	`ffmpeg -f gdigrab -framerate 5 -offset_x 10 -offset_y 20 -video_size 640x480 -i desktop out.mpg `

- ffmpeg 从视频中生成 gif 图片  
	`ffmpeg -i capx.mp4 -t 10 -s 320x240 -pix_fmt rgb24 jidu1.gif`

- ffmpeg 将图片转换为视频  
	假如你的图片在c:\temp\下面。那么通过下面的命令就可以将这个目录下面的图片转换成视频。这里面有个要求就是你的图片全部是自然数为文件名， 001, 002, 003这样的。前面要几个0取决于你的图片的个数，如109张，那么就是3-1=2个0，从001 到109，如果是1009张就是0001到1009.  
	`ffmpeg -f image2 -i c:\temp\d.jpg test.mp4`  
	你可以指定编码格式  
	`ffmpeg -f image2 -i c:\temp\d.jpg -vcodec libx264 test.mp4`  
	也许你还想指定输出帧率  
	`ffmpeg -f image2 -i c:\temp\d.jpg -vcodec libx264 -r 10  test.mp4`  
	这样输出的test.mp4就是每秒播放10帧了  
	如果你要指定码率  
	`ffmpeg -f image2 -i c:\temp\d.jpg -vcodec libx264 -r 10 -b 200k  test.mp4`  
	注意这里的200k的单位bit/s.  
	也许你要问那么到底最后生成的文件的参数都是什么样子，比如我们刚才设置的参数，还有我们没有设置的参数  
	那么这个命令就可以帮你搞定  
	`ffmpeg -i test.mp4`

- 将一个高清流，复制为几个不同视频清晰度的流重新发布，其中音频不变  
	`ffmpeg -re -i rtmp://server/live/high_FMLE_stream -acodec copy -vcodec x264lib -s 640×360 -b 500k -vpre medium -vpre baseline rtmp://server/live/baseline_500k -acodec copy -vcodec x264lib -s 480×272 -b 300k -vpre medium -vpre baseline rtmp://server/live/baseline_300k -acodec copy -vcodec 264lib -s 320×200 -b 150k -vpre medium -vpre baseline rtmp://server/live/baseline_150k -acodec libfaac -vn -ab 48k rtmp://server/live/audio_only_AAC_48k`

- 功能同上一样，只是采用 -x264opts 选项  
	`ffmpeg -re -i rtmp://server/live/high_FMLE_stream -c:a copy -c:v x264lib -s 640×360 -x264opts bitrate=500:profile=baseline:preset=slow rtmp://server/live/baseline_500k -c:a copy -c:v x264lib -s 480×272 -x264opts bitrate=300:profile=baseline:preset=slow rtmp://server/live/baseline_300k -c:a copy -c:v x264lib -s 320×200 -x264opts bitrate=150:profile=baseline:preset=slow rtmp://server/live/baseline_150k -c:a libfaac -vn -b:a 48k rtmp://server/live/audio_only_AAC_48k`

- 提取视频的 YUV 原始数据  
    `ffmpeg -i input.mp4 output.yuv`

- 抽取某一帧 YUV：抽出 jpeg 图片，然后把 jpeg 转为 YUV  
    `ffmpeg -i input.mp4 -ss 00:00:20 -t 10 -r 1 -q:v 2 -f image2 pic-%03d.jpeg`  
    `ffmpeg -i pic-001.jpeg -s 1440x1440 -pix_fmt yuv420p xxx3.yuv`

> 常用参数说明：
> - 主要参数：  
>       -i 设定输入流    
>       -f 设定输出格式  
>       -ss 开始时间 
> - 视频参数：  
>     -b 设定视频流量，默认为200Kbit/s  
>     -r 设定帧速率，默认为25  
>     -s 设定画面的宽与高  
>     -aspect 设定画面的比例  
>     -vn 不处理视频  
>     -vcodec 设定视频编解码器，未设定时则使用与输入流相同的编解码器
> - 音频参数：  
>     -ar 设定采样率  
>     -ac 设定声音的Channel数  
>     -acodec 设定声音编解码器，未设定时则使用与输入流相同的编解码器   
>     -an 不处理音频

# FFplay

- 设置代理并播放
    ```bash
    export http_proxy=http://xxx:80
    export https_proxy=http://xxx:80
    
    ffplay "https://xxxx/xxx.m3u8"
    ```

- 播放 test.mp4 ，播放完成后自动退出  
    `ffplay -autoexit test.mp4`
    
- 以 320 x 240 的大小播放 test.mp4  
    `ffplay -x 320 -y 240 test.mp4`
    
- 将窗口标题设置为 "myplayer"，循环播放 2 次  
    `ffplay -window_title myplayer -loop 2 test.mp4`
    
- 播放 双通道 32K 的 PCM 音频数据  
    `ffplay -f s16le -ar 32000 -ac 2 test.pcm`

- 直接播放摄像头的数据（"WebCam SC-0311139N"为[设备名称](#query_device)）  
    `ffplay -f dshow -i video="WebCam SC-0311139N"`

- 播放流  
    `ffplay rtmp://127.0.0.1:1935/live/oceans`  
    `ffplay http://127.0.0.1:80/hls/oceans.m3u8`