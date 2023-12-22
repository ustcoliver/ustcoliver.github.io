---
title: zsh 配置与美化
draft: false
tags:
  - basic-exp
---


本文介绍 linux 如何安装`zsh`和配置`oh-my-zsh`的过程

## 安装 zsh

- **Arch**

```bash
sudo pacman -S zsh
```

- **Debian/Ubuntu**

```shell
sudo apt install zsh
```

## 安装、配置 oh-my-zsh

- **Curl**

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

- **wget**

```shell
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

### oh-my-zsh 插件

- **zsh-autosuggestions 自动提示插件**

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

- **zsh-syntax-highlightling 语法高亮插件**

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

### oh-my-zsh 主题： powerlevel10k

oh-my-zsh 有很多 [主题](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)
这里选择可配置性最强的[powerlevel10k](https://github.com/romkatv/powerlevel10k)

![效果演示](https://raw.githubusercontent.com/romkatv/powerlevel10k-media/master/prompt-styles-high-contrast.png)

```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

#### 字体
    

linux本机或windows ssh连接，要想显示丰富的图标，需要在本地主机上安装字体。
linux远程主机则可以跳过。
- Hack [下载](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.0.2/Hack.zip)
- Meslo [下载](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.0.2/Meslo.zip)
- JetBrains [下载](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.0.2/JetBrainsMono.zip)

这几个是比较受欢迎的，也可以去[Nerd fonts网站](https://www.nerdfonts.com/font-downloads)自行选择喜欢的

- Windows
    
下载字体，解压后即可右键点击安装。或者使用 scoop 安装：

```powershell
scoop install Hack-NF-Mono
```

- Arch

Arch 由于软件包丰富，可以直接使用pacman安装

```shell
sudo pacman -S ttf-hack-nerd ttf-jetbrains-mono-nerd ttf-meslo-nerd
```

- Debian/Ubuntu

Debian/ubuntu则需要下载、解压字体后，将字体放置于`/usr/share/fonts`文件夹中
```shell
sudo cp -r Hack/ /usr/share/fonts
```

### 配置生效

编辑~/.zshrc 文件，修改文中内容如下

- **主题**

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

- **插件**

```bash
plugins=(git extract z zsh-autosuggestions zsh-syntax-highlighting )
```

其中 `git`插件包含了一系列 alias [详情](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git)
extract 插件

最后

```bash
source ~/.zshrc
```

使配置生效。

## 个性化配置

### 添加 alias

为某些常用命令添加 alias
编辑`~/.zshrc`,在文件末尾添加

- **vim**
    如果使用 neovim，则改为 nvim 即可

```bash
alias v='vim'
```

- **zshrc**
    如果需要经常编辑`~/.zshrc`文件，如添加`$PATH`, 通过 alias 添加各种自己常用的命令等
    则可以添加如下两行，快速编辑和生效

```bash
alias szrc="source ~/.zshrc"
alias vzrc="vim ~/.zshrc"
```

#### eza

eza 是一个新的 ls 命令，能够为文件和文件夹添加颜色和图标区分等丰富功能。

![效果](https://assets.oliverustc.top/cfoliver/2023/12/9e70e5a25f78a5a6a82c25ce330cdd65.gif)

- **Arch 安装**

Arch 直接使用pacman安装
```
sudo pacman -S eza
```

- **Debian/Ubuntu 安装**

```shell
# 不要直接复制此链接,去github下载最新的release
wget https://github.com/eza-community/eza/releases/download/v0.17.0/eza_x86_64-unknown-linux-gnu.zip
unzip eza-xxx.zip 
sudo install ezz /usr/local/bin/
```

编辑`~/.zshrc`

```
alias ls="eza --icons"
alias lt="ls -T"
alias la="ls -a"
```

以上只是 exa 的简单利用，更多功能参见[介绍](https://github.com/eza-community/eza?tab=readme-ov-file#command-line-options)

### 修改`.p10k.zsh`

powerlevel10k 支持显示很多内容，这里简单介绍我正使用的功能
打开`.p10k.zsh`文件，搜索`POWERLEVEL9k_RIGHT_PROMPT_ELEMENTS`，可以看到很多功能，

- 注释 `context`, 这一项开启后，通过 ssh 连接此机器时，会显示`username@hostname`，没什么用
- 将`load, disk_usage, ram, ip`取消注释, 分别对应了 cpu、硬盘、内存使用情况和当前 ip 地址

> 尤其是 ip，我需要 ssh 连接很多台机器，但容易多个终端跳来跳去绕晕时，通过 ip 快速找到需要的机器终端
>
> 如果打开了 ip，默认还会显示网络上传、下载速度，过分挤占了右侧空间，如果要关闭流量速度显示：
> 搜索`POWERLEVEL9K_IP_CONTENT_EXPANSION` 将
> `typeset -g POWERLEVEL9K_IP_CONTENT_EXPANSION='$P9K_IP_IP${P9K_IP_RX_RATE:+ %70F⇣$P9K_IP_RX_RATE}${P9K_IP_TX_RATE:+ %215F⇡$P9K_IP_TX_RATE}'`
> 改为
> `typeset -g POWERLEVEL9K_IP_CONTENT_EXPANSION='$P9K_IP_IP'`
> 这样会在右侧仅显示 ip 而不显示上传下载流量

### 关闭自动更新

在`.zshrc`文件找到`zstyle ':omz update' mode disable`将其取消注释

## 最终效果

![最终效果](https://assets.oliverustc.top/cfoliver/2023/12/09c9b75d4c4297505c4dd17f5436c0bb.png)

## 参考链接

[oh-my-zsh github 仓库](https://github.com/ohmyzsh/ohmyzsh)

[autosuggestion 插件](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md)

[syntax-highlightling 插件](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)

[powerlevel10k 仓库](https://github.com/romkatv/powerlevel10k)

[eza](https://github.com/eza-community/eza)