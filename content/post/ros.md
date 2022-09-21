---
title: "ROS+Singularity+Azure Kinectで幸せになる（ならん）"
date: 2022-09-21T14:18:35+09:00
draft: true
categories:
    - Tech
tags:
    - ROS
    - Singularity
    - AzureKinect
    - めんどくさいオブ・ザ・イヤー2023受賞
---

# モチベーション
澪田：「ROSでAzure Kinectを動かしたいッス！」<br />
澪田：「でもPCの環境を汚したくないッス！」<br />
みたいなときに使えるかもしれない。
前提として述べておくと、基本的にネイティブのUbuntuにそのままROSを載せて動かすほうがよっぽど楽である。<br />
PC間通信も楽だし、USB周りも楽、何より基本的なトラブルシューティングがずっと調べやすい。<br />
ちゃんとSLAMとかやりたいならSinguralityでROSを動かして、なんてはためんどくさいことせずに環境構築をやったほうが良い。<br />
Singularityについての詳細な説明は省く。<br />
要はDockerだけど/devとかをいい感じにしてくれる上にROOT権限がなくても色々できるわけだ。(fakeroot権限のみはrootユーザに登録してもらわないとダメだけど)

# 準備
	- Azure Kinect
	- 強めのUbuntu PC（Ubuntuは18.04以上)
	- Singularityのインスコ

# やること
以下のDockerfileチックなdefファイルを使ってimageを作成する。

~~~
Bootstrap: docker
From: nvidia/cuda:11.6.0-devel-ubuntu18.04

%environment
        export CUDA_PATH=/usr/local/cuda
        export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
        source /opt/ros/melodic/setup.bash

%post
	# mirrorを設定
        perl -p -i.bak -e 's%(deb(?:-src|)\s+)https?://(?!archive\.canonical\.com|security\.ubuntu\.com)[^\s]+%$1http://ftp.riken.jp/Linux/ubuntu/%' /etc/apt/sources.list

        export DEBIAN_FRONTEND=noninteractive
        export TZ=Asia/Tokyo
        export CUDA_PATH=/usr/local/cuda
        export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

        apt-get -y update
        apt-get -y dist-upgrade
        apt-get -y install wget git build-essential curl lsb-release gcc

        # Install ROS
        sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
        curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -

        apt-get -y update
        apt-get -y install software-properties-common ros-melodic-desktop-full python3-catkin-tools python3-wstool python3-pip python3-pycryptodome liblz4-1 liblz4-dev ros-melodic-rospy-message-converter ros-melodic-ros-numpy
        pip3 install -U pip
        pip3 install --upgrade setuptools
        pip3 install opencv-python requests numpy rospkg gnupg opencv-contrib-python roslibpy
        pip3 install roslz4

        # for Azure Kinect DK
        curl -sSL https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
        apt-add-repository https://packages.microsoft.com/ubuntu/18.04/prod
        apt-get update
	
	# For installing ver 1.4 k4a packages
        echo 'libk4a1.4 libk4a1.4/accepted-eula-hash string 0f5d5c5de396e4fee4c0753a21fee0c1ed726cf0316204edda484f08cb266d76' | debconf-set-selections
        echo 'libk4a1.4 libk4a1.4/accept-eula boolean true' | debconf-set-selections
        echo 'libk4abt1.1 libk4abt1.1/accepted-eula-hash string 03a13b63730639eeb6626d24fd45cf25131ee8e8e0df3f1b63f552269b176e38' | debconf-set-selections
        echo 'libk4abt1.1 libk4abt1.1/accept-eula boolean true' | debconf-set-selections
        apt install -y k4a-tools=1.4.1 libk4a1.4-dev
        apt install -y libk4abt1.1-dev
~~~

色々な試行錯誤の上たどり着いたdefファイルですので、愛情がこもっています。<br />
特にk4aパッケージ周りは結構めんどくさいということを覚えておいてください。<br />
後は肝心のことなんですが、udevのrulesの編集が必要です。<br />
これだけはsudo権限を使って使うPCに書き込んでおく必要があります。<br />

~~~
git clone https://github.com/microsoft/Azure-Kinect-Sensor-SDK.git
sudo cp Azure-Kinect-Sensor-SDK/scripts/99-k4a.rules /etc/udev/rules.d/
~~~

ちなみに、image上でudev/rules.dを弄ってもAzure Kinectを接続出来ないので注意してください。<br />
以上でAzure Kinectでk4aviewerの起動、ROSでのメッセージの受信などができます。<br />
ついでに、bodytrackingのパッケージもいれてあるので姿勢推定も行えます。<br />











