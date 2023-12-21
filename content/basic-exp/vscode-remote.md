# Vscode-ssh实现远程开发

# 生成 ssh 密钥

首先，切换目录到.ssh 文件夹，如果没有就创建一个

```
cd ~/.ssh
```

```shell
ssh-keygen -t ed25519 -C "test@test" -f test_key
```

- `-t`指定密码学算法, `ed25519`是一种高强度的非对称加密算法，相比于 ras，ecdsa 等密钥最短。
- `-C`是为公钥添加备注，可以自己随便写
- `-f`指定密钥文件名，这里指定为`test_key`

后续会让你设置`passphrase`，直接回车即可
随后会在你当前文件夹，即`~/.ssh`中生成两个文件，`test_key`和`test_key.pub`
其中`test_key`为私钥，自己妥善保管，`test_key.pub`是公钥，可以分享给别人

# 写入到远程主机

通过密码登录远程主机

```shell
ssh user@ip
```

切换目录到`~/.ssh`, 同样，如果没有此文件夹可以创建一个

```
cd ~/.ssh
```

编辑 authorized_keys 文件, 将你的 test_key.pub 的文件内容写入到此文件中

```
vim ./authorized_keys
```

`:wq`保存退出

退出远程主机，回到本机，可以使用`-i`指定密钥来进行登录

```
ssh user@ip -i /path/to/test_key.pub
```

# 编写 ssh config 文件

```shell
cd ~/.ssh
```

创建新文件 `config`, 在新文件中写入以下内容

```
Host server
	HostName xxx.xxx.xxx.xxx
	User user
	IdentityFile ~/.ssh/test_key
```

- Host 项对应的`server`可以随便设置，如果经常使用此主机，可以设置的短一些，少打几个字母
- HostName 写远程主机的 ip 或域名
- User 即远程主机用户名
- `IdentityFile`即所需的私钥路径

保存上述内容到 config 文件，使用命令

```shell
ssh server
```

就可以直接连接到远程主机

# vscode 配置

打开 vscode，安装`remote ssh`插件
随后就可以看到 `server` 选项了
![](https://assets.oliverustc.top/cfoliver/2023/11/2fb1de2e61993c48ac65e7d43eed41a9.png)
点击右侧即可打开，等待其在远程主机设置好 vscode 服务器

设置好后，点击右侧文件图标->打开文件夹，选择开发所使用文件夹即可

![](https://assets.oliverustc.top/cfoliver/2023/11/bf66305402ff9fe0adf220c80bb466ba.png)
