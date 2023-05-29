# Ubuntu 基础软件安装与配置

<!--more-->

本文介绍在`Ubuntu21.10`上如何安装基础软件和配置镜像，用作备忘，不用再一个一个查


## docker

### 安装
```bash
sudo apt install docker.io docker-compose
#新建docker用户组，ubuntu应该会在apt install 后自动帮你创建
sudo groupadd docker
sudo usermod -aG docker $USER
sudo systemctl enable docker
sudo systemctl restart docker
```
注销后重新登录即可执行`docker version`验证安装
### 配置docker镜像

```shell
# 如果没有此文件夹就新建一个
cd /etc/docker
sudo vim /etc/docker/daemon.json
```
文件中写入以下内容

```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### 验证安装
```shell
docker version
docker-compose -v
```

## golang

### apt 安装
可能版本较低，但是应该问题不大
```shell
sudo apt install golang
```
### 安装最新版本
```shell
wget https://go.dev/dl/go1.18.3.linux-amd64.tar.gz
```
```shell
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz
```
编辑`.zshrc`,在最后一行
```shell
export PATH=$PATH:/usr/local/go/bin
```
`source ~/.zshrc`生效

### golang 镜像
在`.zshrc`中追加
```shell
# 配置 GOPROXY 环境变量
export GO111MODULE=on
export GOPROXY=https://goproxy.cn
```
`source ~/.zshrc`生效
### 验证设置
```shell
go env
```

## Python

### 安装pip
ubuntu 会自带python3，这里只介绍pip

```shell
sudo apt install python3-pip
```

### 配置镜像
#### 科大源
**临时使用**
```shell
pip install -i https://mirrors.ustc.edu.cn/pypi/web/simple package
```
**设为默认**
``` shell
pip config set global.index-url https://mirrors.ustc.edu.cn/pypi/web/simple
```
#### 清华源
**临时使用**

```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```
**设为默认**
```shell
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

## conda 安装
### 获取安装脚本
```shell
wget https://repo.anaconda.com/miniconda/Miniconda3-py39_4.12.0-Linux-x86_64.sh
```
### 安装
```shell
chmod +x Miniconda3-py39_4.12.0-Linux-x86_64.sh
./Miniconda3-py39_4.12.0-Linux-x86_64.sh
```

## Rust 安装

```shell
curl https://sh.rustup.rs -sSf | sh
```
### 添加到PATH
```shell
export PATH=$PATH:$HOME/.cargo/bin
```
### 验证安装
```shell
cargo --version
cargo 1.62.0 (a748cf5a3 2022-06-08)
```

## nvm 安装

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```
### 加载设置
```shell
source ~/.zshrc
```
### 安装node v8
```shell
nvm install 8
nvm use 8
```
同理可以安装其他版本，使用use切换即可
### npm 淘宝镜像
```shell
npm config set registry https://registry.npm.taobao.org
```

## zsh 
zsh 相比于bash来说，功能更加丰富，可以安装各种插件和主题，[安装教程](https://olivergogogo.xyz/posts/zsh_customize/)

记得按照教程中知识在windows中安装字体，将`windows terminal`中的字体改为新字体即可加载所有图标

## 一键安装脚本
准备写一个一键安装并配置设置的脚本，但是没时间，暂时搁置，

~~或者以后说服自己这东西没有写的必要，哈哈哈哈~~


