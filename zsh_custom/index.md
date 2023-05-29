# Zsh配置与美化


# zsh 配置与美化

zsh相对bash来说是一个功能更加丰富的shell

本文用作zsh的定制，包括图标美化，系统状态提示，命令高亮，命令历史提示等。

~~寻找的教程都不太满意，所以自己写一个备忘。~~

## 我的效果

![zsh-done.png](/images/zsh_custom/zsh-done.png)

### 所有效果

![https://raw.githubusercontent.com/romkatv/powerlevel10k/master/powerlevel10k.png](https://raw.githubusercontent.com/romkatv/powerlevel10k/master/powerlevel10k.png)

## 安装zsh

```bash
sudo apt install zsh
```

## oh my zsh 安装

> Oh My Zsh is a delightful, open source, community-driven framework for managing your Zsh configuration. It comes bundled with thousands of helpful functions, helpers, plugins, themes, and a few things that make you shout…
> 

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
或者
```shell
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

## plugins 插件

在`$HOME/.oh-my-zsh/plugins`中很多自带的插件，只需在`.zshrc`文件中`plugins=()`加入自己喜欢的即可。

所有plugin[列表](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins)

这里列出一些我个人比较喜欢的

```shell
archlinux # arch linux pacman, yay 包管理器alias
autojump # 需要使用apt或pacman安装
docker # docker 命令alias
docker-compose # docker-compose 命令alias
extract # 解压命令 使用`x`即可解压任何形式压缩文件
git # git 命令alias
```

另外还有其他非自带的好用plugins：

### zsh-autosuggestions 自动提示

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

### zsh-syntax-highlighting 语法高亮

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

## powerlevel10k 灵活可配置的oh-my-zsh 主题

纯属个人爱好，要支持此主题还需要安装特定字体才能支持那些图标

### 安装字体和图标
效果图上第一行各种图标就是基于此主题，而且支持自行调整图标

推荐两款字体。可以随便选一个下载，解压后打开点击安装即可。 然后在windows terminal和vsvode上修改字体即可完美显示。

其他字体可以去自行寻找 ： [nerd fonts](https://www.nerdfonts.com/), 可以在[programmingfonts](https://www.programmingfonts.org/)中查看各种字体的效果

这里给几个推荐
* [Hack](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/Hack.zip)
* [Meslo](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/Meslo.zip)
* [JetBrainsMono](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/JetBrainsMono.zip)
建议使用Mono等宽字体的版本，

### 安装theme
```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

## 修改`~/.zshrc` 文件

```
ZSH_THEME="powerlevel10k/powerlevel10k"
plugins=(autojump git extract z zsh-autosuggestions zsh-syntax-highlighting )
```

其中`extract` 可以在终端中直接使用`x` 解压所有格式压缩文件。

`z`为目录快速补全功能，只要之前`cd` 到达过该目录，再次cd 该目录时可以直接输入目标目录，其会自动补全。
`j`其实更好用一些


修改完后执行命令`source ~/.zshrc` 使所有设置生效。

### 重新设置

如果选错了效果想要重新选择

```bash
p10k configure
```

## 进一步设置

打开 ~/.p10k.zsh  可以进一步设置显示`CPU load, disk usage, free ram`等，如果经常需要ssh连接多个机器的话，可以使其显示`ip`，防止在多个终端中跳来跳去忘记到底是哪个机器

我的选择是，显示`CPU,内存，硬盘，ip`，并把`context`即`user@hostname`取消掉 

![Untitled](/images/zsh_custom/p10k_custom.png)

### 我的效果

![Minion](/images/zsh_custom/custom-done.png)

## 参考链接

[oh my zsh 官网](https://ohmyz.sh/)

[所有 oh my zsh 主题](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)

[powerlevel10k](https://github.com/romkatv/powerlevel10k)

[nerd fonts字体下载](https://www.nerdfonts.com/)

