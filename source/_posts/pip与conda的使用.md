---
title: pip与conda的使用
date: 2020-02-10
tags:
- python
category:
- python
abstract: 常用的pip与conda命令
---
## pip的使用
### linux系统下安装pip
```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
pip -V　　#查看pip版本
```

### 国内镜像的添加
```
pip install -i https://pypi.douban.com/simple/
pip install -i https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
```
### 基本的pip命令
```
pip show --files SomePackage  查看包的信息
pip list --outdated  查看过期的包
pip install --upgrade SomePackage 升级某包
pip install -r requirements.txt --proxy=代理服务器IP:端口号
pip install --download d:/python27/packs pandas 下载包到指定路径
```

## conda的使用
### conda的基本命令
```
conda env list 查看当前环境的虚拟环境

Linux: source activate your_env_name(虚拟环境名称)激活环境
       source deactivate  取消环境

Windows: activate your_env_name(虚拟环境名称)
         deactivate
         
conda remove -n your_env_name --all  删除使用环境

conda info -e 查看当前系统的环境

conda list -n env_name 指定查看某环境下安装的package
```
### 为conda添加镜像
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
```

### 使用conda安装任意版本的python
```
conda create --name python37env python=3.7
conda create --name python27env python=2.7
```
