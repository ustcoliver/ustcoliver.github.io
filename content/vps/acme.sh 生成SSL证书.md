本文主要记录为自己的域名申请`*.example.com `证书的主要流程，
默认使用 `Cloudflare` 做 DNS 解析，使用 `Nginx` 在云服务器上做代理

# 前期准备

## nginx

### 安装

```shell
sudo apt install nginx
```

### 启动

```shell
sudo systemctl enable --now nginx
```

# acme.sh

>  **下面所有操作推荐在 root 用户下进行**

## 安装

```shell
curl https://get.acme.sh | sh -s email=my@example.com
```

随后执行`source .bashrc`将`acme.sh`添加到`PATH`中

## 使用 Cloudflare API 生成证书

登录 Cloudflare，进入 `My Profile`-> `API Tokens` -> `Create Token`
选择 `Edit zone DNS`, `Use template`

`Permissions`
不用修改，保持`Zone， DNS， Edit` 不变

`Zone Resources`
选择你要申请的域名

`Client IP Address Filtering` 部分，为了安全，建议将其指定为你自己云服务器的 ip

随后 `Continue to summary`

立刻记录下你的包含 40 个字符的 Token，
根据**Test this token**给出的命令，在云服务器上执行，查看返回是否有报错
正常应该是：

```json
{
	"result": { "id": "xxxx", "status": "active" },
	"success": true,
	"errors": [],
	"messages": [
		{
			"code": 10000,
			"message": "This API Token is valid and active",
			"type": null
		}
	]
}
```

### 设置环境变量

```shell
export CF_Token="xxxx"
```

此处为前面生成的 Token 值

退出 My Profile, 进入所需设置的域名，在页面右下角分别展示了 Zone ID 和 Account ID

```shell
export CF_Zone_ID="xxx"
```

```shell
export CF_Account_ID="xxx"
```

### 生成证书

```shell
acme.sh --issue --dns dns_cf -d example.com -d '*.example.com'
```

### 安装证书到 Nginx

```shell
acme.sh --install-cert -d example.com \
--key-file       /etc/nginx/ssl/example.com/key.pem  \
--fullchain-file /etc/nginx/ssl/example.com/cert.pem \
--reloadcmd     "service nginx force-reload"
```

添加一个 nginx 配置：

```nginx
server {
        listen 443 ssl;
        server_name media.example.com;
        ssl_certificate /etc/nginx/ssl/example.com/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/example.com/key.pem;

        location / {
                proxy_pass http://localhost:8096;
        }
}
```
