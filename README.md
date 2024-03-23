# cockroachai-oauth

- 指路[cockroachai](https://github.com/cockroachai/cockroachai)，本项目可分布部署，与cockroachai部署在不同的服务器上，可对接企业微信机器人，tg机器人

## ja3项目（过cockroachai的cf盾）指路[ja3-proxy](https://github.com/cockroachai/ja3-proxy)

- 找一台可以直连oai的机器，新建一个文件夹复制以下的代码作为docker-compose.yml文件

```bash
version: '3.8'
services:
  tinyproxy:
    image: docker.io/kalaksi/tinyproxy:latest
    restart: unless-stopped
    cap_drop:
      - ALL
    ports:
      - http端口:8888 #左侧端口暴露在外,可根据需求更改，防火墙放行该端口，http代理端口
    environment:
      DISABLE_VIA_HEADER: 'yes'
      STAT_HOST: tinyproxy.stats
      MAX_CLIENTS: 100
      # A space separated list:
      ALLOWED_NETWORKS: 0.0.0.0/0
      LOG_LEVEL: Notice
      TIMEOUT: 900
      AUTH_USER: 'username' #http代理用户名
      AUTH_PASSWORD: 'password' #http代理密码
    healthcheck:
      test: ["CMD", "curl", "-I", "-H", "Host: tinyproxy.stats", "http://localhost:8888"]
      interval: 5m
      timeout: 10s
      retries: 1
  ja3-proxy:
    image: xyhelper/ja3-proxy:latest
    ports:
      - "ja3端口:9988" #左侧端口暴露在外,可根据需求更改，防火墙放行该端口，ja3端口
    environment:
      WEBSITE_URL: "https://chat.openai.com/auth/login" # 要过盾的目标网站
      PROXY: http://username:password@服务器ip:http端口  # 代理服务器信息，填入上方的http代理用户名和密码，端口为上方tinyproxy的端口，必须暴露
      CLIENTKEY: "48bxxxxx" # yescaptcha 的 clientKey
      LOCALPROXYUSER: "" # ja3代理服务用户名，只能为小写字母和数字
      LOCALPROXYPASS: "" # ja3代理服务密码，只能为小写字母和数字
```
或者也可以这样，不过这样http代理以及ja3代理的用户密码则变为一致
```bash
version: '3.8'
services:
  ja3-proxy:
    image: xyhelper/ja3-proxy:latest
    ports:
      - "http端口:3128" #左侧端口暴露在外,可根据需求更改，防火墙放行该端口，http端口
      - "ja3端口:9988" #左侧端口暴露在外,可根据需求更改，防火墙放行该端口，ja3端口
    environment:
      WEBSITE_URL: "https://chat.openai.com/auth/login" # 要过盾的目标网站
      PROXY: http://username:password@服务器ip:http端口  # 代理服务器信息，填入上方的http代理用户名和密码，端口为上方tinyproxy的端口，必须暴露
      CLIENTKEY: "48bxxxxx" # yescaptcha 的 clientKey
      LOCALPROXYUSER: "" # ja3代理服务用户名，只能为小写字母和数字
      LOCALPROXYPASS: "" # ja3代理服务密码，只能为小写字母和数字
```
- 开放以上code中的http代理端口以及ja3端口

- 使用```docker compose up -d ```运行

- 在cockroachai的config文件夹中的config.yml中的PROXY填入```http://username:password@服务器ip:ja3端口```（username为上方代码的ja3代理服务用户名，password为ja3代理服务密码，端口为ja3端口，若ja3和cockroachai部署在一台机器上服务器ip可以为172.17.0.1，不能为127.0.0.1）

- 重启cockroachai

- 在cockroachai服务器上使用```curl --insecure --proxy http://ja3代理服务用户名:ja3代理服务密码@ja3服务器ip:ja3端口 'https://chat.openai.com/'```

- 一个yescaptcha的clientKey,新用户请使用栋哥的推荐链接 https://yescaptcha.com/i/RLazNv 注册,可以直接获得VIP5等级,注意成功后，联系yescaptcha客服可获取赠送的1500积分。

## 对接cockroachai的第三方账户系统

- 本项目采用oauth的方式对接cockroachai的第三方账户系统

## 项目更新

- 新增对每个模型的统计和用户使用模型的统计，总共循环存放7天数据

- 新增token列表对每个用户的标注（普通用户，plus用户，以及未知用户）

- 新增对每个用户进行独立限速以及是否启用plus（默认为不启用plus，40次3小时），Token列表独立出来

- 添加企业微信机器人的定时消息推送，填写企业微信机器人url后每天早上9点推送节点（url）以及节点下的用户

- 添加审计系统，可以看到各个节点过去时间段内（默认3小时，每个用户40次gpt4请求，可修改限速服务中LIMIT: 40，PER: "3h"更改）各个用户的使用情况，仅统计gpt4相关模型

- 新增对不同节点（url）的独立管理，配置token时需独立填写节点（url）

- 新增对删除的用户的限速，当删除某个用户时该用户即使仍然能进入站点但是会提示token无效

- 更改对接tg机器人的命令适配管理不同的节点

- 改用sqlite存储，弃用usertoken.txt文件，更新后需要重新输入用户信息，备份时备份本项目文件夹下data/tokens.db即可

## 使用方法

- 首次使用输入部署时填写的账户系统登录密码进入账户系统中

- 首次添加某一个节点（url）时填写第一行，url为你cockroachai的域名，例如你的cockroachai项目域名为https://cockroachai.com，则url应该填写cockroachai.com

- 后续使用则选择你添加过的节点（url）添加用户（token）即可，删除同理需要先选择节点再对节点内的用户（token）进行删除

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
      - CORRECT_PASSWORD=xxxxx #修改此处改为账户系统登陆密码
	  - UPDATE_URL=http://auditlimit:8998/update #8998为下方auditlimt服务的端口
    volumes:
      - ./data:/app/data
  auditlimit:
    image: lyy0709/auditlimit:latest
    restart: always
    ports:
      - "8998:8998" #左侧为暴露的端口
    volumes:
      - ./data:/app/data
```

### ~~创建密码本~~

- ~~密码本路径为本项目地址下的``` /data/usertoken.txt ```~~
  
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
xxx.com { #将xxx.com修改为自己的域名为账户系统
    reverse_proxy localhost:8999
}
yyy.com { #将yyy.com修改为自己的域名为限速系统
    reverse_proxy localhost:8998
}
```

### 配置cockroachai

- 进入cockroachai的config文件夹并打开config.yaml

- 添加以下代码

```bash
OAUTH_URL: https://xxx.com/oauth #填入上一步反代的域名
AUDIT_LIMIT_URL: https://yyy.com/audit_limit
```

- 重启cockroachai

## 对接企业微信机器人

- 填入企业微信机器人的url，点击保存配置

## 对接tg机器人

- 填入tg机器人的token，点击保存配置

- tg机器人命令为

```bash
/add url token #添加某个token到节点（url）中
/delete url token #从节点（url）中删除某个token
/list #查看每个节点（url）中的所有tokens
```

## 运行图片


![1](https://img.lyy0709.xyz/i/2024/03/19/154619.webp)
![2](https://img.lyy0709.xyz/i/2024/03/19/154706.webp)
![3](https://img.lyy0709.xyz/i/2024/03/19/154754.webp)
![4](https://img.lyy0709.xyz/i/2024/03/19/154817.webp)
![5](https://img.lyy0709.xyz/i/2024/03/19/154847.webp)
![6](https://img.lyy0709.xyz/i/2024/03/10/180936.webp)
![7](https://img.lyy0709.xyz/i/2024/03/10/180936_1.webp)

