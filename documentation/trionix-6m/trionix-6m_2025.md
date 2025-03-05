---
layout: page
title: Trionix-6M
# permalink: /trionix-6M/
---

<!-- # Trionix-6M  -->

## ТЕХНИЧЕСКИЕ ХАРАКТЕРИСТИКИ

> [!CAUTION]
> Не является действующей инструкцией

| ПАРАМЕТР | ЗНАЧЕНИЕ |
| :--- | :--- |
| ГАБАРИТЫ (Ф х в)| 41 x 45 мм |
| ВЕС (сухой) | 0.16 кг |
| ДЛИНА КАБЕЛЯ (не менее) | 0.3 м |
| МАКСИМАЛЬНАЯ ДАЛЬНОСТЬ АКУСТИЧЕСКОЙ СВЯЗИ<sup>[1](#footnote1)</sup> | 200 м |
| ПОЛОСА ЧАСТОТ | 22 .. 29 кГц |
| ЭНЕРГОПОТРЕБЛЕНИЕ | 0.0055 Вт |
| НАПРЯЖЕНИЕ ПИТАНИЯ | 5 .. 10 В |
| ДИАПАЗОН РАБОЧИХ ТЕМПЕРАТУР | 0 .. 30 °С |
| МАКСИМАЛЬНАЯ ГЛУБИНА ПОГРУЖЕНИЯ | 300 м |
| ПРОБУЖДЕНИЕ ВНЕШНЕГО ОБОРУДОВАНИЯ | Подача логической единицы длительностью 90 секунд | 
| ЧИСЛО ВОЗМОЖНЫХ АДРЕСОВ | 64 |
| НАСТРОЙКА АДРЕСА | по UART, с подключением к надводному блоку пробуждения [F4105-SU](F4105_SU_Specification_ru.md) |

for build the client app install the following packages

```shell
 mvn install:install-file -Dfile="client/libs/sdl2gdx-1.0.4.jar" -DgroupId="com.sdl2gdx" -DartifactId=sdl2gdx -Dversion="1.0.4" -Dpackaging=jar
```


## How to run client and server in simulation mode

### Run server
```
 java -jar .\trionix.jar -profile robot-local-mock
```

### Run client
```
 java -jar .\Trionix_terminal.jar --spring.profiles.active=simulator

```
# Camera

![Screenshot](/documentation/trionix_6m_cam.jpg)

## How to run camera server on RaspberryPi 

### RTSP
```
cd /home/pi/gst-rtsp-server-1.19.2/build/examples

./test-launch "v4l2src device=/dev/video11 ! video/x-h264, width=640, height=480, framerate=30/1 ! h264parse config-interval=1 ! rtph264pay name=pay0 pt=96"
```

### MJPEG
```

```
### Server enviroment configuration guide
https://docs.google.com/document/d/14aLXrTUmcMpBPiyxBX0RoBi-IZIExmlPnaSNsFlmQGg/edit?usp=sharing

## Additional commands

### Launch rtsp-server

```
./test-launch "v4l2src device=/dev/video11 ! video/x-h264, width=640, height=480, framerate=30/1 ! h264parse config-interval=1 ! rtph264pay name=pay0 pt=96"
```

```
/home/xxx/gst-rtsp-server-1.19.2/build/examples/test-launch "( rpicamsrc awb-mode=tungsten preview=false rotation=180 ! video/x-h264, width=1920, height=1080, framerate=15/1, profile=constrained-baseline ! h264parse ! queue ! rtph264pay name=pay0 config-interval=2 pt=96 )"
```

### Show rtsp-stream

```
gst-launch-1.0 rtspsrc location=rtsp://raspberrypi:8554/test/ latency=10 ! decodebin ! autovideosink
```

### Record stream

```
gst-launch-1.0 -e rtspsrc location=rtsp://raspberrypi:8554/test/ ! queue ! rtph264depay ! h264parse ! mp4mux ! filesink location=result3.mp4
```

### UDP streaming

```
gst-launch-1.0 -v v4l2src device=/dev/video0 num-buffers=-1 ! image/jpeg, width=640, height=480, framerate=30/1, format=MJPG ! jpegdec ! jpegenc ! rtp
jpegpay ! udpsink host=192.168.1.49 port=5200
```

## UDP multicast

### Check interfaces (Windows)
```
netsh interface ip show joins
```

### Stream 

```
gst-launch-1.0 -v v4l2src device=/dev/video0 num-buffers=-1 ! image/jpeg, width=640, height=480, framerate=30/1, format=MJPG ! jpegdec ! jpegenc ! rtpjpegpay ! udpsink host=224.0.0.251 port=5000 auto-multicast=true sync=false
```
### Receive

```
gst-launch-1.0 -v udpsrc multicast-group=224.0.0.251 auto-multicast=true port=5000 ! application/x-rtp, media=video, clock-rate=90000, payload=96 ! rtpjpegdepay ! jpegdec ! videoconvert ! autovideosink
```

## TCP

### Stream

```
gst-launch-1.0 -v v4l2src device=/dev/video0 num-buffers=-1 ! image/jpeg, width=1920, height=1080, framerate=30/1, format=MJPG ! jpegdec ! jpegenc ! tcpserversink  host=192.168.1.105 port=5000
```

### Low cpu

```
 gst-launch-1.0 -v v4l2src device=/dev/video0 num-buffers=-1 ! image/jpeg, width=1280, height=720, framerate=30/1, format=MJPG ! tcpserversink  host=192.168.1.105 port=5000
```

### Receive

```
gst-launch-1.0 tcpclientsrc host=192.168.1.105 port=5000 ! jpegdec ! videoconvert ! autovideosink
```

## v4l2

https://people.eng.unimelb.edu.au/pbeuchat/asclinic/software/workflow_usb_camera_settings.html

### List devices

```
v4l2-ctl --list-devices
```

### List controls

```
v4l2-ctl --device=<device> --list-ctrls
v4l2-ctl --device=<device> --list-ctrls-menus
```

### Get the value currently set for a particular control using the following:

```
v4l2-ctl --device=<device> --get-ctrl=<ctrl>
```

Where <device> is replaced as usual, and <ctrl> is replaced by the name of the control as listed by the --list-ctrls command. Note that the --get-ctrl option can be passed a comma separated list of control names to retrieve the values of.

### Set the value for a particular control using the following:

```
v4l2-ctl --device=<device> --set-ctrl=<ctrl>=<val>
```

### Check the available video formats

```
v4l2-ctl -d /dev/video0 --list-formats-ext
v4l2-ctl -d /dev/video0 --list-formats-ext
```

### Get and set the focus

```
v4l2-ctl --device=/dev/video0 --get-ctrl=focus_auto,focus_absolute
```

```
v4l2-ctl --device=/dev/video0 --set-ctrl=focus_auto=0
v4l2-ctl --device=/dev/video0 --set-ctrl=focus_absolute=0
```

### Get and set the resolution

The options for the camera resolution are grouped under what is termed “video capture format options”. To display the manual entry for these options, use the following command:

```
v4l2-ctl --help-vidcap
```

To get and display the current video capture format options for a particular device, use the following:

```
v4l2-ctl --device=<device> --get-fmt-video
```

To set the video capture settings format options for a particular device, use the following:

```
v4l2-ctl --device=<device> --set-fmt-video=width=<w>,height=<h>,pixelformat=<pf>,field=<f>,colorspace=<c>,xfer=<xf>,ycbcr=<y>,quantization=<q>,premul-alpha,bytesperline=<bpl>
```

The --set-fmt-video command can be used with a sub-set of the options shown above. To set the resolution (width and height) of the camera images captured by /dev/video0, use the following:

```
v4l2-ctl --device=/dev/video0 --set-fmt-video=width=640,height=480
```

### Display everything about a device

```
v4l2-ctl --device=<device> --all
```

## Service

https://transang.me/three-ways-to-create-a-startup-script-in-ubuntu/

## Environment Configuration Guide
https://docs.google.com/document/d/1ETiChGfqR9AypelnjyXJ1QMxhF7vVIYx02T4HcPE9vY/edit?usp=sharing