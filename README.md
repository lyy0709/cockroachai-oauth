# cockroachai-oauth

- 指路[cockroachai](https://github.com/cockroachai/cockroachai)，本项目可分布部署，与cockroachai部署在不同的服务器上，可对接企业微信机器人，tg机器人

## 对接cockroachai的第三方账户系统

- 本项目采用oauth的方式对接cockroachai的第三方账户系统

## 项目更新

- 新增对每个用户进行独立限速以及是否启用plus，Token列表独立出来

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

![微信截图_20240310180719](https://img.lyy0709.xyz/i/2024/03/10/180817.webp)
![微信截图_20240310180755](https://img.lyy0709.xyz/i/2024/03/10/180817_1.webp)
![微信截图_20240310180905](https://img.lyy0709.xyz/i/2024/03/10/180936.webp)
![微信截图_20240310180926](https://img.lyy0709.xyz/i/2024/03/10/180936_1.webp)

