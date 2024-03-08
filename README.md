# cockroachai-oauth

- 指路[cockroachai](https://github.com/cockroachai/cockroachai)，本项目可分布部署，与cockroachai部署在不同的服务器上，可对接~~企业微信机器人~~，tg机器人

## 对接cockroachai的第三方账户系统

- 本项目采用oauth的方式对接cockroachai的第三方账户系统

## 部署方法

### 拉取本项目或者手动创建docker-compose.yml

- 拉取指令

```bash
git clone https://github.com/lyy0709/cockroachai-oauth.git
```

- 手动创建``` docker-compose.yml ```
  
```bash
version: '3.8'
services:
  cockroachai-oauth:
    image: lyy0709/cockroachai-oauth:latest
    ports:
      - "8999:8999" #左侧为暴露的端口
    environment:
      - SECRET_KEY=xxxxxxxxxxxxxx #修改此处进行session保护，应使用复杂的随机值
      - CORRECT_PASSWORD=xxxxx #修改此处改为登陆密码
    volumes:
      - ./data:/app/data
```

### 创建密码本

- 密码本路径为本项目地址下的``` /data/usertoken.txt ```
  
### 使用caddy进行反代

- 安装caddy指令（debian，ubuntu系统）

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

- 进入caddy目录并填写反代

```bash
cd /etc/caddy
vim Caddyfile
```

- 复制并粘贴

```bash
xxx.com { #将xxx.com修改为自己的域名
    encode gzip
    reverse_proxy localhost:8999
}
```

### 配置cockroachai

- 进入cockroachai的config文件夹并打开config.yaml

- 添加以下代码

```bash
OAUTH_URL: https://xxx.com/oauth #填入上一步反代的域名
```

- 重启cockroachai

## 对接~~企业微信机器人~~

## 对接tg机器人

- 填入tg机器人的token，点击保存配置

- tg机器人命令为

```bash
/add token #添加某个token到列表中
/delete token #从列表中删除某个token
/list #查看列表中的所有tokens
```

## 运行图片

![IMG_0334](https://img.lyy0709.xyz/i/2024/03/08/114136.webp)
![IMG_0333](https://img.lyy0709.xyz/i/2024/03/08/114138.webp)
![ba8a61de40018c216fb56c0395084544](https://img.lyy0709.xyz/i/2024/03/08/114135.webp)

