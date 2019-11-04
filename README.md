# 语音识别

本项目是基于PaddlePaddle的[DeepSpeech](https://github.com/PaddlePaddle/DeepSpeech)项目修改的，方便训练中文自定义数据集。

本项目使用的环境：
 - Python 2.7
 - PaddlePaddle 1.6.0

## 目录

- [环境搭建](#环境搭建)
- [数据准备](#数据准备)
- [训练模型](#训练模型)
- [评估和预测](#评估和预测)
- [项目部署](#项目部署)

## 环境搭建

 - 请提前安装好显卡驱动，然后执行下面的命令。
```shell script
# 卸载系统原有docker
sudo apt-get remove docker docker-engine docker.io containerd runc
# 更新apt-get源 
sudo apt-get update
# 安装docker的依赖 
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
# 添加Docker的官方GPG密钥：
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 验证拥有指纹
sudo apt-key fingerprint 0EBFCD88
# 设置稳定存储库
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

 - 安装Docker
```shell script
# 再次更新apt-get源 
sudo apt-get update
# 开始安装docker 
sudo apt-get install docker-ce
# 加载docker 
sudo apt-cache madison docker-ce
# 验证docker是否安装成功
sudo docker run hello-world
```

 - 安装nvidia-docker
```shell script
wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker_1.0.1-1_amd64.deb
sudo dpkg -i /tmp/nvidia-docker*.deb && rm /tmp/nvidia-docker*.deb
```

 - 下载 PaddlePaddle Docker 镜像
```shell script
sudo nvidia-docker pull hub.baidubce.com/paddlepaddle/deep_speech_fluid:latest-gpu
```

- git clone 本项目源码
```bash
git clone https://github.com/yeyupiaoling/DeepSpeech.git
```

- 运行 PaddlePaddle Docker 镜像
```shell script
sudo nvidia-docker run -it -v $(pwd)/DeepSpeech:/DeepSpeech hub.baidubce.com/paddlepaddle/deep_speech_fluid:latest-gpu /bin/bash
```

 - 安装 PaddlePaddle 1.6.0，因为这个项目必须要在 PaddlePaddle 1.6.0 版本以上才可以运行。
 
```shell script
pip install paddlepaddle-gpu==1.6.0.post107 -i https://mirrors.aliyun.com/pypi/simple/
```

## 数据准备

 - 首先进行到本项目的脚本文件夹中，我们所有程序都使用脚本执行。
```shell script
cd DeepSpeech/run/
```

 - 本项目提供了下载公开的中文普通话语音数据集，分别是Aishell，Free ST-Chinese-Mandarin-Corpus，THCHS-30 这三个数据集，总大小超过28G。
```shell script
sh download_public_data.sh
```

 - 如果开发者有自己的数据集，可以使用自己的数据集进行训练，当然也可以跟上面下载的数据集一起训练。自定义的语音数据需要符合一下格式：
    1. 语音文件需要放在`DeepSpeech/dataset/audio/`目录下，例如我们有个`wav`的文件夹，里面都是语音文件，我们就把这个文件存放在`DeepSpeech/dataset/audio/`。
    2. 然后把数据列表文件存在`DeepSpeech/dataset/annotation/`目录下，程序会遍历这个文件下的所有数据列表文件。例如这个文件下存放一个`my_audio.txt`，它的内容格式如下。每一行数据包含该语音文件的相对路径和该语音文件对应的中文文本，要注意的是该中文文本只能包含纯中文，不能包含标点符号、阿拉伯数字以及英文字母。
```shell script
./dataset/audio/wav/0175/H0175A0171.wav 我需要把空调温度调到二十度
./dataset/audio/wav/0175/H0175A0377.wav 出彩中国人
./dataset/audio/wav/0175/H0175A0470.wav 据克而瑞研究中心监测
./dataset/audio/wav/0175/H0175A0180.wav 把温度加大到十八
```
 

 - 然后执行下面的数据集处理脚本，这个是把我们的数据集生成三个JSON格式的文件，分别是`manifest.dev、manifest.test、manifest.train`。然后计算均值和标准差用于归一化，脚本随机采样2000个的语音频谱特征的均值和标准差，并将结果保存在`mean_std.npz`中。建立词表。最后建立词表，把所有出现的字符都存放子在`zh_vocab.txt`文件中，一行一个字符。以上生成的文件都存放在`DeepSpeech/dataset/`目录下。
```shell script
sh prepare_train_data.sh
```


## 训练模型

 - 在执行训练之前，我们先来下载官方的预训练模型和官方提供的超大语言模型文件，这些文件将存放在`DeepSpeech/models`目录下。
```shell script
sh download_model.sh
```

 - 执行训练脚本，开始训练语音识别模型， 每训练一轮保存一次模型，模型保存在`DeepSpeech/models/checkpoints`目录下。
```shell script
sh train.sh
```

## 评估和预测

 - 在训练结束之后，我们要使用这个脚本对模型进行超参数调整，提高语音识别性能。
```shell script
sh hyper_parameter_tune.sh
```

 - 我们可以使用这个脚本对模型进行评估，通过字符错误率来评价模型的性能。
```shell script
sh eval.sh
```

 - 使用下面的脚本执行预测，获取预测的情况。
```shell script
sh infer.sh
```

## 项目部署
