# deepStream 

platfrom : Jetson Xavier

https://developer.nvidia.com/deepstream-sdk

## Install

- 여러 방법이 있지만, jetson 보드를 사용한다면,
- apt를 통해 쉽게 설치 가능

### additional Lib

```shell
sudo apt install \
libssl1.0.0 \
libgstreamer1.0-0 \
gstreamer1.0-tools \
gstreamer1.0-plugins-good \
gstreamer1.0-plugins-bad \
gstreamer1.0-plugins-ugly \
gstreamer1.0-libav \
libgstrtspserver-1.0-0 \
libjansson4=2.11-1
```

### librdkafka

```shell
git clone https://github.com/edenhill/librdkafka.git
cd librdkafka
git reset --hard 7101c2310341ab3f4675fc565f64f0967e135a6a
./configure
make
sudo make install
sudo mkdir -p /opt/nvidia/deepstream/deepstream-5.1/lib
sudo cp /usr/local/lib/librdkafka* /opt/nvidia/deepstream/deepstream-5.1/lib
```

### deepStream sdk

```shell
sudo vi /etc/apt/sources.list.d/nvidia-l4t-apt-source.list
# edit nvidia-l4t-apt-source.list
deb https://repo.download.nvidia.com/jetson/common r32.5 main	#repo change
sudo apt-get update
sudo apt-get install deepstream-6.0
```



## SDK Path

```sh
/opt/nvidia/deepstream/deepstream-6.0/
```

- 해당 폴더에서 수행해도 되지만,
- root 디렉토리이므로 통째로 home으로 옮겨도 됨



## deepStream with YOLOv3

```shell
cd [path]/deepstream/deepstream-6.0/sources/objectDetector_Yolo
```

- 해당 폴더로 이동

```shell
./prebuild.sh
```

- 해당 shell script를 동작시키면 yoloV2,V3의 weight 등을 다운로드 함.

```shell
cd ./nvdsinfer_custom_impl_Yolo
```

- 해당 폴더로 이동
- 해당 폴더에서 make 수행

```shell
export CUDA_VER=10.2	# check your cuda ver using nvcc --version

make		# if make in root Dir, "sudo CUDA_VER=10.2 make"
```



## Active App

```shell
cd [path]/deepstream/deepstream-6.0/bin
```

- 예제 프로그램이 모여있음
- yolo 예제를 실행하기 위해서는 다음과 같이 입력

```shell
deepstream-app -c ../sources/objectDetector_Yolo/deepstream_app_config_yoloV3.txt
```

- ERROR : yolo 

  - 관련 에러가 발생하면, yolov3.weight, yolov3.cfg 등을 
  - deepstream/deepstream-6.0/bin 으로 이동

  ```shell
  cp [path]/deepstream/deepstream-6.0/sources/objectDetector_Yolo/yolo* [path]/deepstream-[ver]/bin/
  ```

  

- Delay : Building tensorRT engine 

  - 첫 동작 시에는 해당 engine을 생성하느라 4~5분정도 소요됨.
  - 첫 동작 이후 model_b1_gpu0_iny8.engine 파일을 
  - [path]/deepstream/deepstream-6.0/bin 에서 확인할 수 있음
  - 이를 objectDetector_Yolo로 이동

  ```shell
  cp [path] deepstream-[ver]/bin/model_b1_gpu0_iny8.engine [path]/deepstream/deepstream-6.0/sources/objectDetector_Yolo
  ```

- 이후 app 동작 시에는 engine 과정이 생략되는데 이를 위해서는 deepstream_app_config_yoloV3.txt를 수정해야 함.

### txt rewrite

```markdown
################################################################################
# Copyright (c) 2019-2020, NVIDIA CORPORATION. All rights reserved.
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
################################################################################

[application]
enable-perf-measurement=1
perf-measurement-interval-sec=5
#gie-kitti-output-dir=streamscl

[tiled-display]
enable=1
rows=1
columns=1
width=1280  # 출력 영상 크기
height=720  # 출력 영상 
gpu-id=0
#(0): nvbuf-mem-default - Default memory allocated, specific to particular platform
#(1): nvbuf-mem-cuda-pinned - Allocate Pinned/Host cuda memory, applicable for Tesla
#(2): nvbuf-mem-cuda-device - Allocate Device cuda memory, applicable for Tesla
#(3): nvbuf-mem-cuda-unified - Allocate Unified cuda memory, applicable for Tesla
#(4): nvbuf-mem-surface-array - Allocate Surface Array memory, applicable for Jetson
nvbuf-memory-type=0

[source0]
enable=1
# Type - 1=CameraV4L2 2=URI 3=MultiURI
type=1
uri=file:///home/carnavi/workspace/video/FCWS.avi	
# 사용할 영상 소스 변경
num-sources=1
gpu-id=0
# (0): memtype_device   - Memory type Device
# (1): memtype_pinned   - Memory type Host Pinned
# (2): memtype_unified  - Memory type Unified
cudadec-memtype=0

# camera 사용시, 주석 해제
# camera-id=0
# camera-width=1280
# camera-height=720
# camera-fps=30

[sink0]
enable=1
#Type - 1=FakeSink 2=EglSink 3=File
type=2
sync=0		# 영상 사용시, 프레임 일치시키고자 할 경우 1로 세트
source-id=0
gpu-id=0
nvbuf-memory-type=0

[osd]
enable=1
gpu-id=0
border-width=1
text-size=15
text-color=1;1;1;1;
text-bg-color=0.3;0.3;0.3;1
font=Serif
show-clock=0
clock-x-offset=800
clock-y-offset=820
clock-text-size=12
clock-color=1;0;0;0
nvbuf-memory-type=0

[streammux]
gpu-id=0
##Boolean property to inform muxer that sources are live
live-source=0
batch-size=1
##time out in usec, to wait after the first buffer is available
##to push the batch even if the complete batch is not formed
batched-push-timeout=40000
## Set muxer output width and height
width=1280	# 객체 인식 시 이미지 사이즈
height=720	# 객체 인식 시 이미지 사이즈
##Enable to maintain aspect ratio wrt source, and allow black borders, works
##along with width, height properties
enable-padding=0
nvbuf-memory-type=0

# config-file property is mandatory for any gie section.
# Other properties are optional and if set will override the properties set in
# the infer config file.
[primary-gie]
enable=1
gpu-id=0
model-engine-file=model_b1_gpu0_int8.engine # 주석 처리되어있는데
# 첫 구동 이후 해당 파일이 생성되면,
# 주석 해제
labelfile-path=labels.txt
batch-size=1
#Required by the app for OSD, not a plugin property
bbox-border-color0=1;0;0;1
bbox-border-color1=0;1;1;1
bbox-border-color2=0;0;1;1
bbox-border-color3=0;1;0;1
interval=2
gie-unique-id=1
nvbuf-memory-type=0
config-file=config_infer_primary_yoloV3.txt

[tracker]
enable=1
# For NvDCF and DeepSORT tracker, tracker-width and tracker-height must be a multiple of 32, respectively
tracker-width=640
tracker-height=384
ll-lib-file=/opt/nvidia/deepstream/deepstream-6.0/lib/libnvds_nvmultiobjecttracker.so
# ll-config-file required to set different tracker types
# ll-config-file=../../samples/configs/deepstream-app/config_tracker_IOU.yml
ll-config-file=../../samples/configs/deepstream-app/config_tracker_NvDCF_perf.yml
# ll-config-file=../../samples/configs/deepstream-app/config_tracker_NvDCF_accuracy.yml
# ll-config-file=../../samples/configs/deepstream-app/config_tracker_DeepSORT.yml
gpu-id=0
enable-batch-process=1
enable-past-frame=1
display-tracking-id=1

[tests]
file-loop=0
```



## 동작 결과

