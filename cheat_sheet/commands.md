
## Additional Gstreamer commands
1. [GStreamer command-line cheat sheet](https://github.com/matthew1000/gstreamer-cheat-sheet)
2. [Gstreamer cheat sheet (Wiki)](http://wiki.oz9aec.net/index.php/Gstreamer_cheat_sheet)


### List video cameras (Ubuntu)
```
ls -ltrh /dev/video*

> crw-rw----+ 1 root video 81, 1 sep 20 08:43 /dev/video1
> crw-rw----+ 1 root video 81, 0 sep 20 08:43 /dev/video0
```

### Check video camera format (Ubuntu)
```
apt install v4l-utils

v4l2-ctl -d /dev/video0 --list-formats-ext

```



## Mixing

- How to mix two test streams (**videobox**, **videomixer**)

      gst-launch-1.0 videomixer name=mixer ! gtksink \
      videotestsrc ! videoscale ! video/x-raw,width=960,height=540 ! \
      tee name=t !  queue ! videoconvert ! videobox left=-960 ! videoconvert ! mixer. \
      t. ! queue ! videobox left=0  ! videoconvert ! mixer.
      
## Join multiple video parts in one video

    gst-launch-1.0 splitmuxsrc location=video*.mp4 ! h264parse ! mp4mux ! filesink location=out.mp4
    
### Using FFMPEG

    ffmpeg -f concat -safe 0 -i <(for f in ./*.mp4; do echo "file '$PWD/$f'"; done) -c copy all.mp4

## Splitting

- Split single video into 60 sec pieces:

      gst-launch-1.0 filesrc location=video.mp4 ! qtdemux ! h264parse ! splitmuxsink max-size-time=60000000000 location=piece%02d.mp4
      
## Networking

### TCP

#### Test stream (videotestsrc)

- send

      gst-launch-1.0 videotestsrc ! videoscale ! video/x-raw,width=100,height=100 ! \
      x264enc tune="zerolatency" ! mpegtsmux ! tcpserversink host=127.0.0.1 port=5000
    
- receive

      gst-launch-1.0 tcpclientsrc port=5000 host=127.0.0.1 ! tsdemux ! \
      h264parse ! avdec_h264 ! videoconvert ! gtksink


#### From webcamera (v4l2src)

- send 

      gst-launch-1.0 v4l2src ! jpegenc ! tcpserversink host=localhost port=5000
    
- receive

      gst-launch-1.0 tcpclientsrc port=5000 ! jpegdec ! videoconvert ! autovideosink sync=true
      
## Video Record

### From webcam
```
gst-launch-1.0 v4l2src ! videoconvert ! x264enc tune=zerolatency pass=quant !  h264parse ! mp4mux ! filesink location=video.mp4 -e
```

### RTSP

#### Recording video to multiple files (duration 1 minute)
    gst-launch-1.0 rtspsrc location=rtsp://... drop-on-latency=true latency=0 ! \
    rtph264depay ! h264parse ! splitmuxsink location=video%02d.mp4 max-size-time=60000000000
    
    
## FPS measurements

### RTSP

    gst-launch-1.0 -v rtspsrc location=rtsp://... drop-on-latency=true latency=0 ! \
    decodebin ! videoconvert ! video/x-raw,format=RGB ! videoconvert ! \
    fpsdisplaysink video-sink=fakesink signal-fps-measurements=True
    
    
### Jetson
    gst-launch-1.0 filesrc location=video.mp4 ! qtdemux name=demux1 demux1.video_0 ! \
    queue ! h264parse ! omxh264dec ! fpsdisplaysink video-sink=fakesink signal-fps-measurements=True


## Format Conversion

### H264 -> RGBA

#### Jetson
     gst-launch-1.0 -v -e filesrc location=video2560.mp4 ! qtdemux name=demux1 demux1.video_0 ! queue ! \
     h264parse ! omxh264dec ! nvvidconv ! video/x-raw,format=RGBA ! \ 
     fpsdisplaysink video-sink=fakesink signal-fps-measurements=True
     
### MPEG -> MP4
     gst-launch-1.0 filesrc location=video.mpeg ! mpegpsdemux ! h264parse ! mp4mux ! filesink location=video.mp4 sync=False -e
     
## Take snapshot

### RTSP
     gst-launch-1.0 -v rtspsrc location=... drop-on-latency=true latency=0 num-buffers=1 ! \
     decodebin  !  videoconvert ! jpegenc ! multifilesink location=img101.jpg
     
     
## Video Processing

### Change Width/Height (Using videoscale plugin)
- scales video to Full HD (1920x1080)

      gst-launch-1.0  filesrc location=video.mp4 ! decodebin ! videoconvert ! videoscale ! \
      video/x-raw,width=1920,height=1080 ! videoconvert ! \
      x264enc tune=zerolatency bitrate=16384 ! filesink location=video.mp4 -e
      
### Change FPS of video (Using videorate plugin)
- making video with 15 FPS

      gst-launch-1.0  filesrc location=video.mp4 ! decodebin ! videoconvert ! \ 
      videorate ! video/x-raw,framerate=15/1 ! videoconvert ! \
      x264enc tune=zerolatency bitrate=16384 ! filesink location=video.mp4
      
## Streaming from Youtube
    gst-launch-1.0 souphttpsrc is-live=true location="$(youtube-dl -f mp4 -g https://www.youtube.com/watch?v=xjDjIWPwcPU)" \
    ! decodebin ! videoconvert ! gtksink sync=false
**Note**: 
- [youtube-dl](https://youtube-dl.org/) allows to get HTTPS URL for youtube video link. 


## [Splitting/Sharing pipeline](https://github.com/matthew1000/gstreamer-cheat-sheet/blob/master/sharing_and_splitting_pipelines.md#summary-of-methods-to-share-and-split-pipelines)
   
