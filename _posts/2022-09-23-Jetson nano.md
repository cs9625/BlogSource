---
layout: post
title:  "NVIDIA Jetson Nano Developer Kit"
date:   2022-09-23 10:20:05
categories: embedded
tags:
  - embedded
author: Panda
images:
---


> 본 블로그는 SEOUL G-캠프의 jetson nano 유튜브 강의 내용을  기반으로 작성함.   
> [유튜브 강의 바로가기](https://www.youtube.com/playlist?list=PLieE0qnqO2kQufTeWH2e3R1JXKV1d3zmS)      
> [관련자료 Github 링크](https://github.com/jugfk)   


#### 1. Jetson Nano   
  - 암A57 쿼드코어와 128개 맥스웰 GPU 코어가 함께 내장되어 있다.
  <img src="/images/blogs/embedded/jetson-01.JPG" width="40%" height="30%" title="px(픽셀) 크기 설정" alt="jetson"></img>
  ![Jetson Nano]({{"/images/blogs/embedded/jetson-01.JPG"| relative_url}}){: width="50%" height="50%"}

  - 소형 SO-DIMM(Dual-In-Memory Module) 폼팩터로 제공

  - Jetson Nano Dev Kit vs Rasberry Pi 3B+
  ![사양비교]({{"/images/blogs/embedded/jetson-02.JPG"| relative_url}})
  ![성능비교]({{"/images/blogs/embedded/jetson-03.JPG"| relative_url}})

### Jetson Nano 냉각 팬 제어   

1. 수동방식
    ```python
    import numpy
    ```

    ```bash
    $sudo jetson_clocks

    $sudo sh -c 'echo 255 > /sys/devices/pwm-fan/target_pwm'
    $sudo sh -c 'echo 127 > /sys/devices/pwm-fan/target_pwm'
    $sudo sh -c 'echo 0 > /sys/devices/pwm-fan/target_pwm'
    ```

2. 자동방식 (부팅시에 자동으로 동작)   
    - 파이썬 모듈 설치    
        ```
        $sudo apt update
        $sudo apt install -y python3-dev
        ```

    - 설치 드라이버 다운로드   
        ```
        $git clone https://github.com/jugfk/jetson-fan-ctl.git
        $cd jetson-fan-ctl/
        $-sudo sh install.sh
        ```
    - Adjust the fan speed:
        ```
        $sudo vi /etc/automagic-fan/config.json
        ```
    - Check the fan status:
        ```
        $sudo service automagic-fan status
        ```

### Jetson Nano 스왑 설정
   
 ```
-- swap 활성화 여부 확인
$free -m

-- swapon 명령으로 스왑 상태를 표시(ZRAM에 의한 것인지 여부 확인)
$swapon -s
```

> ZRAM 이란?
> - 일종의 swap 이지만 Disk I/O 를 일으키지 않고 일정 조건에 의해서 swap 할 메모리를 우선 압축하도록 하는 기능
> - 메모리상에 존재하는 데이터들은 상당히 압축률이 높은 특성을 갖는다는 전재를 깔고 도입되는 사항이며
> - 압축 알고리즘은 빠르다고 알려진 LZO (zip 에서 사용하는) 등을 많이 사용.
> - 생각보다 메모리를 많이 아낄 수 있고 성능손실이 크지 않다고 알려져 있어서 대부분의 임베디드 시스템에는 (특히 NAS, 안드로이드) 많이 도입되는 기능

SWAP 설정 방법
1. 전용 파티션을 만들고 파티션 전체를 스왑으로 사용하는 방법 (절차 복잡)
2. 파일로 스왑을 만드는 방법 (쉽게 만들 수 있음)
    ```
    -- swap 파일 생성
    $sudo dd if=/dev/zero of=/var/swapfile bs=1G count=4

    -- /var/swapfile을 스왑파일로 사용하기 위해 초기화
    $sudo mkswap /var/swapfile

    -- root 사용자만 액세스 활 수 있도록 파일 권한 변경(변경하지 않으면 스왑파일을 마운트 할 때 경고가 표시됨)
    $sudo chmod 600 /var/swapfile

    -- Jetson Nano 시작할 때 자동으로 스왑을 마운트하도록 /etc/fstab 파일에 아래 내용 추가
    $sudo vi /etc/fstab
    --> /var/swapfile  none    swap    swap    0   0

    -- 즉시 스왑 파일을 활성화 하려면 다음과 같이 "swapon" 명령 실행
    $sudo swapon /var/swapfile

    -- free 명령으로 스왑이 확장 된 것을 확인
    $free -m

    -- swapon 명령으로 스왑 상태 표시
    $swapon -s
    ```

### Jetson 파워모드 전환

Jetson Nano에는 전원 관리 IC가 탑재되어 있으며, 소비 전력을 최적화 하면서 성능수행을 조정하는 등 고급 전원 관리 시스템을 구현하고 있음. 초기 상태에서 **5W**(저전력)과 **MAXN**(최대성능 발휘) 두개의 파워 모드가 설정되어 있으며 쉽게 전환 사용할 수 있다. 기본 파워 모드는 MAXN로 성정되어 있음.

* 제공 기능   
    CPU 코어의 인에이블/디스에이블   
    CPU의 동작 주파수   
    GPU TPC의 인에이블/디스에이블   
    GPU의 동작 주파수   
    (EMC)의 동작 주파수 메모리   
    

* 각 파워 모드의 설정   
  
    | 모드 이름 | MAXN | 5W |
    | ------ | -------- | ---------- |
    | 소비전력 | 10W | 5W|
    | 모드ID | 0 | 1 |
    | 온라인 CPU 코어수 | 4 | 2 |
    | CPU 최대 주파수 | 1479 | 918 |
    | GPU TPC | 1 | 1 |
    | GPU 최대 주파수(MHz) | 921.6 | 640 |
    | 메모리 최대 주파수(MHz) | 1600 | 1600 |
   
1. GUI 이용 전환   
    ![파워모드 전환]({{"/images/blogs/embedded/jetson-04.JPG"| relative_url}})
2. tegrastats 명령 실행 (드롭다운 메뉴에서 "Run tegrastats")   
    - cpu와 gpu의 성능 지수 확인
    - jetson nano의 소비 전력이 증가하고 전압이 저하되면 'System is now being throttled."라는 알림 표시됨. 
    ```
    $ tegrastats (1초 마다 로그 출력)
    $ tegrastats --interval 5000 (5초 마다 로그 출력)
    ```
  
3. nvpmodel 명령   
    - 파워 모드를 MAXN로 변경
        ```
        $sudo nvpmodel -m 0
        ```
    - 파워 모드를 5W로 변경
        ```
        $sudo nvpmodel -m 1
        ```        

    - 현재 파워 모드 확인
        ```
        $sudo nvpmodel -q
        $sudo nvpmodel -q --verbos (자세한 설명 표시)
        ```        
  

### Jetson Nano에 Tensorflow 설치

1. 파이썬 기본 버전 변경   
    -- 파이썬 설치 확인   
    ```bash
    $ python -V
    Python 2.7.17

    $ which python
    /usr/bin/python

    $ ls -al /usr/bin/python
    lrwxrwxrwx 1 root root 9 Apr 16  2018 /usr/bin/python -> python2.7

    $ ls /usr/bin/ | grep python
    aarch64-linux-gnu-python3.6-config
    aarch64-linux-gnu-python3.6m-config
    aarch64-linux-gnu-python3-config
    aarch64-linux-gnu-python3m-config
    dh_python2
    dh_python3
    python
    python2
    python2.7
    python3
    python3.6
    ...

    ```

    -- Update-alternatives로 파이썬 버전 등록 및 변경
    ```bash
    -- 이미 등록된 것이 있는 지 확인
    $ sudo update-alternatives --config python
    update-alternatives: error: no alternatives for python

    -- update-alternatives에 등록
    $ sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
    $ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.6 2

    -- 파이썬 버전 선택
    $ sudo update-alternatives --config python
    There are 2 choices for the alternative python (providing /usr/bin/python).

    Selection    Path                Priority   Status
    ------------------------------------------------------------
    * 0            /usr/bin/python3.6   2         auto mode
    1            /usr/bin/python2.7   1         manual mode
    2            /usr/bin/python3.6   2         manual mode

    Press <enter> to keep the current choice[*], or type selection number: 2

    -- 파이썬 버전 확인
    $ python --version
    Python 3.6.9

    $ ls -al /usr/bin/python
    lrwxrwxrwx 1 root root 24 Sep 24 04:02 /usr/bin/python -> /etc/alternatives/python
    ```
    참고   
    - Alternatives는 파이썬 뿐 아니라 Java 등의 모든 프로그램의 버전을 관리하는 데 사용할 수 있음.

2. 우분투 패키지 업그레이드

    ```
    -- 설치 가능한 리스트를 업데이트
    $ sudo apt-get update

    -- apt-get update로 가져온 각 패키지들의 최신 버전에 맞게 업그레이드
    $ sudo apt-get upgrade
    ```

3. pip 설치
    ```
    $ sudo apt-get install python-pip
    $ sudo apt-get install python3-pip
    $ python -m pip install --upgrade pip
    ```

    pip3를 pip로 변경하기
    ```
    vi ~/.bashrc

    -- 아래 라인 추가
    alias pip = "pip3"
    ```

4. jtop 설치 (시스템 모니터링)
    ```
    $ sudo -H pip install -U jetson-stats

    if don't have pip
    ($ sudo apt install python-pip)

    $ sudo reboot

    -- jetson-stats 실행
    $ jtop
    ```

5. Tensorflow 설치
   
   참조 : [https://docs.nvidia.com/deeplearning/frameworks/install-tf-jetson-platform/index.html](https://docs.nvidia.com/deeplearning/frameworks/install-tf-jetson-platform/index.html)

   ```
   $ sudo pip3 install --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v46 tensorflow==2.6.0+nv21.11
   ```

6. 가상환경 설치
   ```
   $ sudo apt-get install python3-venv
   $ python -m venv venv --system-site-packages
   $ source venv/bin/activate
   ```

### 온도 모니터하기
1. 온도센서 정보
    ```
    $ cat /sys/devices/virtual/thermal/thermal_zone*/type
    ```

2. 온도표시
    ```
    $ cat /sys/devices/virtual/thermal/thermal_zone*/temp
    ```

3. 파이썬 온도센서 그래프 표시
    ```
    $ sudo apt install libfreetype6-dev
    $ sudo pip3 install numpy
    $ sudo pip3 install matplotlib
    $ git clone https://github.com/jetsonworld/jetson-thermal-monitor.git
    $ cd jetson-thermal-monitor
    $ python3 jetson_temp_monitor.py
    ```

4. 자바스크립트 온도그래프 만들기   
   - Node.js, npm, node-red 이용   

   ```
   $ sudo apt update
   $ sudo apt install nodejs
   $ sudo apt install npm
   $ sudo npm install -g --unsafe-perm node-red (오류 시 npm 최신으로 업데이트 )
        -- npm cache clean --force
        -- npm install -g n
        -- sudo n stable
   $ node -v
   $ npm -v
   $ node-red (Ctrl + C)

   $ cd .node-red
   $ npm install node-red-dashboard
   $ wget https://raw.githubusercontent.com/jetsonworld/javascript-thermal-monitor/master/flows.json
   $ node-red 

   -- http://127.0.0.1:1880/ 접속
   -- flows.json 파일 import 하여 배포 후 확인
   ```
    

### 헤드리스화 하기   
   - GUI를 실행하지 않도록 하여 가용 메모리 크리 늘리거나 CPU 부하 낮춤.

1. GUI 자동 시작 정지   
    - 모드 확인     
    ```
    $ free -m
    $ sudo apt install screenfetch
    $ systemctl get-default
    graphical.target
    ```

    - CLI 모드로 변경   
    ```
    $ systemctl set-default multi-user.target
    $ sudo reboot
    $ free -m (메모리 사용량 비교)
    # CLI 모드에서도 아래 명령을 통해 Graphic 환경 이용 가능   
    $ startx 
    ```

    - 다시 GUI 모드로 변경    
    ```
    $ sudo systemctl set-default graphical.target
    $ sudo reboot
    ```

2. CLI 환경에서 WIFI 연결하기   
    ```
    $ nmcli connection show
    $ nmcli device wifi list
    $ sudo nmcli device wifi connect Hub-name password xxxxxxxxx
    ```

3. microUSB로 연결하기
    - AC 어댑터 사용 시 microUSB 포트 PC의 USB 포트에 연결하여 PC에서 Jetson Nano 엑세스 가능 (어댑터 연결 시 J48 쇼트시켜야 함). 
    - 연결하면 자동으로 장치 드라이버가 설치되어 네트워크가 설정됨.
    - 네트워크가 설정되면 Jetson Nano가 DHCP 서버가 되고, PC에 IP 주소가 할담 됨.

4. 원격 데스크톱 제어
    - realVNC 이용 (https://www.realvnc.com/en/connect/download/viewer/)
    - vino-server 구동 (실패...)   
    ```
    $ gsettings set org.gnome.Vino require-encryption false
    $ /usr/lib/vino/vino-server
    ```

### 도커 컨테이너 사용하기
1. 도커 기반 기술   
   
   - 리눅스 커널에서 제공하는 컨테이너 기술 이용
   - 리눅스 커널의 cgroups와 namespaces 기술 이용
   - 컨테이너는 가상화가 아닌 격리 기술
   - Jetson Nano에 디폴트로 설치되어 있음.

2. 도커 컨테이너 기본 조작
   - 도커 버전 확인   
   ```
   $ sudo docker version
   ```

   - NVIDIA 컨테이너 런타임 확인   
   ```
   $ sudo docker info | grep nvidia
   $ sudo dpkg --get-selections | grep nvidia
   ```
    
   - Hello Word 실습   
   ```
   $ sudo docker run hello-world
   $ sudo docker images
   ```

   - Node-RED를 도커 컨테이너에 설치하기   
   ```
   $ sudo docker run -it -p 1880:1880 --name mynodered nodered/node-red    
   -- 브라우저 접속 후 확인
   http://127.0.0.1:1880/
   ```

   - NGINX Web Server 실행
   ```
   $ mkdir example
   $ cd example
   $ wget https://raw.githubusercontent.com/jugfk/startDockerContainer/master/Dockerfile
   $ sudo docker build --tag hello:0.1 .
   $ sudo docker run --name hello-nginx -d -p 80:80 -v /root/data:/data hello:0.1
   $ sudo docker ps
   $ sudo docker images
   ```

   - 도커 명령어

   ```
   # List containers   
   $ sudo docker ps   
   $ sudo docker ps -a  

   # List port mappings or a specific mapping for the container   
   $ sudo docker port mynodered   

   # List the most recently created images   
   $ sudo docker images   

   # Get real time events from the server   
   $ sudo docker events   

   # Display system-wide information   
   $ sudo docker info   

   # Docker STATS   
   $ sudo docker stats   

   # restart container
   $ sudo docker start [컨테이너 name]
   $ sudo docker stop [컨테이너 name]

   # Remove one or more containers   
   $ sudo docker rm [컨테이너ID]
   $ sudo docker rm `docker ps -a -q` (모두삭제)

   # Remove one or more images   
   $ sudo docker rmi [이미지ID]
   $ sudo docker rmi -f [이미지ID] (컨테이너가 있어 삭제 안될경우 컨테이너까지 삭제)
   ```
   ![도커 명령어]({{"/images/blogs/embedded/jetson-05.JPG"| relative_url}})

### 파워, 리셋, 와이파이, 그리고 블루투스 설치

1. Jetson Nano의 전원을 ON/OFF 하는 방법   
    - microUSB 전원 케이블을 사용   
    - J40 헤더 전원 스위치 연결   
        ![전원스위치1]({{"/images/blogs/embedded/jetson-06.JPG"| relative_url}})
        ![전원스위치2]({{"/images/blogs/embedded/jetson-07.JPG"| relative_url}})   
    - 리셋 스위치   
        ![리셋 스위치]({{"/images/blogs/embedded/jetson-08.JPG"| relative_url}})
    - 전원버튼과 리셋버튼의 전체 회로도
        ![회로도]({{"/images/blogs/embedded/jetson-09.JPG"| relative_url}})

2. 와이파이와 블루투스
   ![호환모델]({{"/images/blogs/embedded/jetson-10.JPG"| relative_url}})
   위의 두 모델 Jetson Nano와 호환 확인 됨.

### GPIO (General-Purpose Input/Output)
   - Jetson Nano에는 40핀 확장 헤더(J41)가 준비되어 있음.
   ![확장헤더사양]({{"/images/blogs/embedded/jetson-11.JPG"| relative_url}}) 
   - 180도 회전
   ![확장헤더사양]({{"/images/blogs/embedded/jetson-12.JPG"| relative_url}}) 
   - GPIO 샘플 파이썬 코드 다운로드
   ```
   $ git clone https://github.com/NVIDIA/jetson-gpio.git
   $ cd jetson-gpio
   $ sudo python3 setup.py install
   $ sudo groupadd -f -r gpio
   $ sudo user mod -a -G gpio username
   $ cd samples
   $ python3 샘플코드.py
   ```

### 딥러닝 골격 검출 프로그램  
  - USB 웹카메라 연결   
  - [골격 감지 응용 프로그램 링크](https://github.com/jugfk/tf-pose-estimation)

### ROS
- 오픈소스
- meta-operating system for robot.
- 로봇응용 프로그램 개발을 위한 운영체제와 같은 로봇 플랫폼.
- 하드웨어 추상화, 하위 디바이스 제어, 일반적으로 사용되는 기능의 구현, 프로세스간의 메시지 패싱, 패키지 관리, 개발환경에 필요한 라이브러리와 다양한 개발 및 디버깅 도구를 제공
![ROS]({{"/images/blogs/embedded/jetson-13.JPG"| relative_url}}) 
- 설치 및 구동   
    [https://github.com/jugfk/installROS](https://github.com/jugfk/installROS)
- 레퍼런스   
    [Getting Started with ROS on Jetson Nano](https://www.stereolabs.com/blog/ros-and-nvidia-jetson-nano)   
    [로보티즈 표박사님 네이버 카페](https://cafe.naver.com/openrt/2360)   

### YOLO (You Only Look Once)
- 물체 검출 알고리즘(Object Detection)
- 핵심 아이디어 : 객체 탐지를 단일 회귀 문제로 다시 구성하는 것
- YOLO는 네트워크의 최종 출력단에서 경계박스를 위치 찾기와 클래스 분류가 동시에 이뤄진다. 단 하나의 네트워크가 한번에 특징도 추출하고, 경계박스도 만들고, 클래스도 분류한다. 그러므로 간단하고 빠르다.
- 설치
    [Jetson Nano에서 OpenCV 4.1.1 하기](https://github.com/jugfk/BuildOpenCV)

- 레퍼런스   
    [You Only Look Once : Unified, Real-Time Object Detection
](https://blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=sogangori&logNo=220993971883&parentCategoryNo=6&categoryNo=&viewDate=&isShowPopularPosts=true&from=search)   
    [2017년5월13일 라즈베리파이2에 YOLO2 구현](https://github.com/leehaesung/YOLO-Powered_Robot_Vision)

- 관련논문   
    [YOLO1(2016)](http://arxiv.org/abs/1506.02640)   
    [YOLO2(2017)](https://arxiv.org/abs/1612.08242)   
    [YOLO3(2019)](https://arxiv.org/abs/1804.02767)   
    
### MQTT(Message Queuing Telemetry Transport)
- ISO 표준 발행-구독 기반의 메시징 프로토콜
- TCP/IP 프로토콜 위에서 동작
- M2M, IOT를 위한 통신 프로토콜
- 최소한의 전력과 패킷량으로 통신
- IOT와 모바일 어플리케이션 등의 통신에 적합
- HTTP, TCP등의 통신과 같이 클라이언트-서버 구조로 이루어지는 것이 아닌, 브로커(Broker), 발행자(Publisher), 구독자(Subscriber) 구조로 이루어짐
![MQTT]({{"/images/blogs/embedded/jetson-14.JPG"| relative_url}}) 
-  응용사례   
  ![MQTT]({{"/images/blogs/embedded/jetson-15.JPG"| relative_url}}) 
- MQTT 브로커   
    * Mosquitto ([https://mosquitto.org](https://mosquitto.org)) - 무료
    * HiveMQ ([https://www.hivemq.com](https://www.hivemq.com))
    * mosca ([http://www.mosca.io](http://www.mosca.io))
    * ActiveMQ ([https://activemq.apache.org](https://activemq.apache.org))
    * RabbitMQ ([https://www.rabbitmq.com](https://www.rabbitmq.com))

- 실습   
  
    ```
    - Setting up MQTT v3.1:
    $ sudo apt-get update
    $ sudo apt-get install -y mosquitto mosquitto-clients
    $ sudo pip install paho-mqtt

    - Stop & Start mosquitto broker:
    $ sudo /etc/init.d/mosquitto stop
    $ sudo /etc/init.d/mosquitto start

    - Check the mosqutto condition:
    $ sudo netstat -nap | grep mosquitto

    - Testing MQTT:
    $ mosquitto
    $ mosquitto_sub -v -t 'topic/test'
    $ mosquitto_pub -t 'topic/test' -m 'helloWorld'
    ```

### node-red
[Getting Started](https://nodered.org/docs/getting-started/)
    
### OpenCV (Open Source Computer Vision)
- 실시간 이미지 프로세싱에 중점을 둔 라이브러리
  
```
===================================================================================
How To Install OpneCV 4.4.0 on Jetson Nano:
===================================================================================
- OpenCV 4.4.0
1. Download the instruction script:
wget https://raw.githubusercontent.com/jetsonworld/BuildOpenCV/master/18_How_To_Install_OpenCV_On_Jetson_Nano.txt
cat 18_How_To_Install_OpenCV_On_Jetson_Nano.txt

1. Updating the packages:
sudo apt update
sudo apt install -y build-essential cmake git libgtk2.0-dev pkg-config  libswscale-dev libtbb2 libtbb-dev
sudo apt install -y python-dev python3-dev python-numpy python3-numpy
sudo apt install -y curl

2. Install video & image formats:
sudo apt install -y  libjpeg-dev libpng-dev libtiff-dev 
sudo apt install -y libavcodec-dev libavformat-dev
sudo apt install -y libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
sudo apt install -y libv4l-dev v4l-utils qv4l2 v4l2ucp libdc1394-22-dev


3. Download OpenCV & Contribs Modules:
curl -L https://github.com/opencv/opencv/archive/4.4.0.zip -o opencv-4.4.0.zip
curl -L https://github.com/opencv/opencv_contrib/archive/4.4.0.zip -o opencv_contrib-4.4.0.zip

4. Unzipping packages:
unzip opencv-4.4.0.zip 
unzip opencv_contrib-4.4.0.zip 
cd opencv-4.4.0/

5. Create directory:
mkdir build
cd build

6. Build Opencv using Cmake:
cmake     -D WITH_CUDA=ON \
        -D CUDA_ARCH_BIN="5.3" \
        -D CUDA_ARCH_PTX="" \
        -D ENABLE_PRECOMPILED_HEADERS=OFF \
        -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.4.0/modules \
        -D WITH_GSTREAMER=ON \
        -D WITH_LIBV4L=ON \
        -D BUILD_opencv_python2=ON \
        -D BUILD_opencv_python3=ON \
        -D BUILD_TESTS=OFF \
        -D BUILD_PERF_TESTS=OFF \
        -D BUILD_EXAMPLES=OFF \
        -D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr/local ..


7. Compile the OpenCV with Contribs Modules:
(4 Hours ~)
make -j4
sudo make install

8. Check the OpenCV Version on Terminal:
python3 -c 'import cv2; print(cv2.__version__)'

# References
# OpenCV configuration options reference: https://docs.opencv.org/4.5.1/db/d05/tutorial_config_reference.html
# OpenCV modules: https://docs.opencv.org/4.4.0/
===========================================================================================================
```

### JetPack 설치

1. Host 환경 구성
   - Ubuntu 18.04 (VM) 준비
     - 디스크 100 GB 이상 확보
     - sudo apt update
     - sudo apt upgrade
     - sudo apt install openssh-server
     - sudo apt install ./sdkmanager_[version]-[build#]_amd64.deb (sdkmanager_1.8.1-10392_amd64.deb 미리 준비)
     - sdkmanager 
   

2. SDK Manager를 이용해 Jetson Nano 이미지 빌드 및 설치





  