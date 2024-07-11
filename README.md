# Autoware.universe-with-AWSIM
This repo shows detailed steps to joint test autoware.universe with AWSIM on Jetson Orin Devkit and x86 host machine.  This can largely save your time to figure out issues during learning official docs about deploying awsim and autoware.universe on your machine.

## A simple pipeline of deploying Autoware.Universe on Jetson Orin Devkit and joint testing with AWSIM.

### Prerequisites
#### Hardwares
1. A host PC with GPU card. Mine is a laptop with a RTX 2070 gpu card, 30 series is better.
2. Jetson Orin 32G devkit. If you have a 64G one, that is more powerful and better.
3. One 1000Mbps Router or Switcher. You need this to connect the host PC with your Jetson Orin devices.
4. A Monitor and mouse&keyboard. In the first time, you need these to start configuring your Jetson Orin devices.

#### Software & OS
1. Nomachine is a remote desktop tool for viewing your jetson orin's screen on your PC easily.
2. Jetpack-6.0 is necessary which need to install on your jetson orin, or the following docker container might not work well.
3. Ubuntu-22.04 is better which need to install on your host PC, or the AWSIM might not run well.(BTW, I have tested using a Ubuntu18.04, it can also run well.)



### Step1: Configure your Jetson Orin and host PC

#### Jetson Orin configurations
You have to configure some contents in the bashrc file of your orin.
Copy and paste following configurations without changes into your bashrc file.
 ```
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

# DDS
if [ ! -e /tmp/cycloneDDS_configured ]; then
    sudo sysctl -w net.core.rmem_max=2147483647
    sudo ip link set lo multicast on
    touch /tmp/cycloneDDS_configured
fi

 ```

#### x86 host PC configurations
Also on your host PC's bashrc file, similar configurations are necessary.
```
export ROS_LOCALHOST_ONLY=0
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

if [ ! -e /tmp/cycloneDDS_configured ]; then
    sudo sysctl -w net.core.rmem_max=2147483647
    sudo ip link set lo multicast on
    touch /tmp/cycloneDDS_configured
fi
```


### Step2: Pull and run a pre-compiled version Docker image
You can pull the docker image form my dockerhub
```
$ docker pull docker pull 1429053840/autoware.universe-awsim-1.2.2:r36.3.0-arm64-v0.1
$ sudo rocker --nvidia --x11 --user --network host   -- 1429053840/autoware.universe-awsim-1.2.2:r36.3.0-arm64-v0.1
```
when it shows error, you need to change the --gpus all to --runtime=nvidia, as this is not supported on ARM platform.
A similar one is like bellow, yours will be different.
```
$ docker run --rm -it --network host   --runtime=nvidia  -v /home/lz/Documents:/home/lz/Documents  -e DISPLAY -e TERM   -e QT_X11_NO_MITSHM=1   -e XAUTHORITY=/tmp/.dockero6k0y78q.xauth -v /tmp/.dockero6k0y78q.xauth:/tmp/.dockero6k0y78q.xauth   -v /tmp/.X11-unix:/tmp/.X11-unix   -v /etc/localtime:/etc/localtime:ro  99c3b767c2c1
```


### Step3: Runing the AWSIM on your host PC

##### Download the AWSIM source
```
wget https://github.com/tier4/AWSIM/releases/download/v1.2.2/AWSIM_v1.2.2.zip
```

if your have not installed other relevant dependencies, like libvulkan1 or GPU driver, you need refer to the official quick start demo guide,
using this link: https://tier4.github.io/AWSIM/GettingStarted/QuickStartDemo/

#### run the self-Driving simulation
```
chmod +x <path to AWSIM folder>/AWSIM.x86_64
./<path to AWSIM folder>/AWSIM.x86_64
```

### Step4: Launch autoware.universe in the docker container

Before launch autoware, you still have to download the map resources using following commands.
```
$ cd /
$ wget https://github.com/tier4/AWSIM/releases/download/v1.1.0/nishishinjuku_autoware_map.zip
$ unzip nishishinjuku_autoware_map.zip
```
You also have to run once this following command outside docker container on your jetson orin device.
Otherwise, the rivz2 cannot be displayed on your jetson orin ubuntu22.04 desktop.
```
$ xhost+docker # or xhost+
```

```
$ cd /autoware
$ source install/setup.bash
$ ros2 launch autoware_launch e2e_simulator.launch.xml vehicle_model:=sample_vehicle sensor_model:=awsim_sensor_kit map_path:=/nishishinjuku_autoware_map
```

### Running effect
Two screenshots are showed bellow:
![demo screenshot1](<assets/jetson orin-universe1.png>)

![demo screenshot1](<assets/jetson orin-universe2.png>)

A video demo can be found on Bilibili website:
https://www.bilibili.com/video/BV1naa7eoESm/?share_source=copy_web&vd_source=81fc800c9cfa4893666931076308fa4b

### More Options
If you want to run autoware.universe and AWSIM on one PC, you can refer to my blog through this link:

https://www.gputek.cn:8093/2023/06/08/008-AutonomousDriving/02-Autoware.universe/Ubuntu18%E5%AE%89%E8%A3%85AWSIM%E8%BF%90%E8%A1%8CAutoware-Universe/index.html 
