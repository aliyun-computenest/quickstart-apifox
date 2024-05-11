# Apifox 服务实例部署文档

## 概述

`
Apifox 所做的 API 相关一体化协作流程，围绕 API 文档先行理念，从 API 的全生命周期、全角色工具切入，通过一体化的协作平台实现研发团队各角色的全面协同，为开发者和私有化部署的客户提升软件开发的整体效率。Apifox 功能直击开发工程师的痛点，产品体验简单易用，大幅提升了开发团队的工作效率，迅速赢得了一大批忠实的用户。
本文向您介绍如何在线下服务器部署私有化版 Apifox。
`

## 部署架构

环境资源准备

### 1. 硬件

| 设备推荐配置        | 设备数量 | 部署对象                                                 |
| ------------------- | -------- | -------------------------------------------------------- |
| 4核8G，100G磁盘空间 | 1        | Apifox 服务端                                            |
| 4核8G，100G磁盘空间 | 1        | Mysql 数据库（版本 8 以上）、Redis 数据库（版本 6 以上） |

设备为推荐配置，根据具体的使用人数，不超过 300 人也可以选用单个服务器进行部署，**单机器部署推荐使用 docker-compose 方式部署** 。

### 2. 系统

推荐使用 Linux，发行版不限制，只要能安装 docker 即可。

- Docker Engine 版本：可以通过命令 `docker --version` 查看版本号，推荐使用 2022 年的 [20.10.13](https://docs.docker.com/engine/release-notes/#201013) 的版本，最低须使用 2020 年的 [20.10.0](https://docs.docker.com/engine/release-notes/#20100) 的版本
- 【可选】Docker Compose 版本：可以通过命令 `docker compose --version` 查看版本号，推荐使用 `2.3.3`，最低须使用 `>= 2.x` 的版本

### 3. 服务器资源相关Q&A

1.Apifox 支持非 K8s 的集群部署吗？ 
支持的，将镜像导入到这个集群内的镜像源里，参考 docker run 去编写或界面化配置启动即可

2.访问 Redis 支持集群访问吗？
有相关需求去适配支持，如果是主从类型的话，指定主节点即可，要可写，如果是哨兵则暂时不支持

3.是否支持其他的数据库
如果是与 Mysql8 同语法的数据库或分支数据库可以支持

## 二、获取安装文件

请联系 Apifox 工作人员获取`安装文件下载地址` 和 `授权 License 序列号`
解压后的安装文件包含如下：


**1. 以下文件为必须下载：**
`apifox-ee-xxx.docker.zip`：apifox 服务端 docker 镜像压缩包，需 zip 解压（文件名中的 xxx 表示版本号）


**2. 以下为公开的 docker 镜像，仅服务器无法直连外网时，才需要下载：**

链接: [https://pan.baidu.com/s/1MjXi1Ekzd_9TRONv_j9uuA](https://pan.baidu.com/s/1MjXi1Ekzd_9TRONv_j9uuA) 提取码: f5t6 

`mysql-8.0.docker.zip`：mysql 官方镜像（mysql:8.0）压缩包，需 zip 解压
`redis-6.0.5-alpine.docker.zip`：redis 官方镜像（redis:6.0.5-alpine）压缩包，需 zip 解压

## 部署流程

### 部署步骤

#### 一、获取安装文件

请联系 Apifox 工作人员获取`安装文件下载地址` 和 `授权 License 序列号`
解压后的安装文件包含如下：


**1. 以下文件为必须下载：**
`apifox-ee-xxx.docker.zip`：apifox 服务端 docker 镜像压缩包，需 zip 解压（文件名中的 xxx 表示版本号）


**2. 以下为公开的 docker 镜像，仅服务器无法直连外网时，才需要下载：**

链接: [https://pan.baidu.com/s/1MjXi1Ekzd_9TRONv_j9uuA](https://pan.baidu.com/s/1MjXi1Ekzd_9TRONv_j9uuA) 提取码: f5t6 

`mysql-8.0.docker.zip`：mysql 官方镜像（mysql:8.0）压缩包，需 zip 解压
`redis-6.0.5-alpine.docker.zip`：redis 官方镜像（redis:6.0.5-alpine）压缩包，需 zip 解压


#### 二、安装

##### 1. 安装 mysql

###### 1.1 运行 mysql 容器

如果已有 mysql 数据库，可直接跳过这一步。

**1.1.1 加载 mysql 离线镜像**
提示：如服务器可以直连外网，可跳过本步骤。

将`mysql-8.0.docker.zip`解压（解压命令：`unzip mysql-8.0.docker.zip`）后得到文件`mysql-8.0.docker`，然后运行以下命令。

```shell
docker load -i mysql-8.0.docker
```

如得到的是`mysql-8.0.tar`，则运行以下命令

```shell
docker load -i mysql-8.0.tar
```

**1.1.2 启动 mysql 容器**

启动容器前需要先确保**挂载目录**存在且有写入权限，可以执行 `mkdir -p ~/data/var/data/mysql8/db`

```shell
docker run \
    --restart always \
    --name=mysql8 \
    -e MYSQL_ROOT_PASSWORD=这里改成root账号的密码 \
    -v ~/data/var/data/mysql8/db:/var/lib/mysql \
    -p 3306:3306 \
    -d mysql:8.0 --default-authentication-plugin=mysql_native_password --character-set-server=UTF8MB4
```

##### 2. 安装 redis


如果已有 redis，则可以直接跳过这一步

###### 2.1 加载 redis 离线镜像

提示：如服务器可以直连外网，可跳过本步骤。

将`redis-6.0.5-alpine.docker.zip`解压（解压命令：`unzip redis-6.0.5-alpine.docker.zip`）后得到文件`redis-6.0.5-alpine.docker`，然后运行以下命令。

```shell
docker load -i redis-6.0.5-alpine.docker
```

如得到的是`redis-6.0.5-alpine.tar`，则运行以下命令

```shell
docker load -i redis-6.0.5-alpine.tar
```

###### 2.2 启动 redis 容器

```shell
docker run \
    --restart always \
    --name=redis6 \
    -p 6379:6379 \
    -d redis:6.0.5-alpine
```

##### 3. 初始化 apifox 数据库

###### 3.1. 进入 mysql 容器

```
docker exec -it mysql8 /bin/sh
```

###### 3.2. 进入容器后，连接数据库并按提示输入密码

```
mysql -uroot -p
```

###### 3.3. 执行创建数据库命令

MySQL 里创建数据库**

```sql
CREATE DATABASE IF NOT EXISTS apifox DEFAULT CHARACTER SET utf8mb4 DEFAULT COLLATE utf8mb4_general_ci;
```

##### 4. 安装 Apifox 服务端

###### 4.1 运行 Apifox 容器

###### 加载 Apifox 离线镜像

将`apifox-ee-xxx.docker.zip`解压（解压命令：`unzip apifox-ee-xxx.docker.zip`)后得到文件`apifox-ee.docker.docker`，然后运行以下命令。

```shell
docker load -i apifox-ee.docker
```

###### 运行 Apifox 容器

以下示例使用原生 Docker 方式安装，也 k8s 或 Docker Compose 方式安装

- [k8s 方式安装指南](doc-2355687)
- [Docker Compose 方式安装](doc-2355110)

```shell
docker run \
    --restart always \
    --name=apifox \
    -e MYSQL_HOST=这里改成mysql的ip地址（注意：如果mysql和也安装在本机，不能使用 127.0.0.1，要使用本机内网IP） \
    -e MYSQL_PORT=3306 \
    -e MYSQL_DATABASE=apifox \
    -e MYSQL_USER_NAME=这里改成有apifox库的建表权限的mysql账号名 \
    -e MYSQL_PASSWORD=这里改成mysql账号的密码 \
    -e REDIS_HOST=这里改成redis的ip地址（注意：如果redis和也安装在本机，不能使用 127.0.0.1，要使用本机内网IP） \
    -e REDIS_PORT=6379 \
    -e REDIS_PASSWORD=redis的密码，可为空 \
    -e REDIS_DB=0 \
    -e JWT_SECRET='这里改成20~50位随机字符（登录态等 token 加密秘钥），请务必妥善记录并保管好该值，后续更新升级的时候会用到' \
    -e MAILER_HOST=smtp.exmail.qq.com \
    -e MAILER_PORT=465 \
    -e MAILER_SECURE=true \
    -e MAILER_USER=系统邮件发件邮箱地址 \
    -e MAILER_PASSWORD=系统邮件发件邮箱密码 \
    -e LICENSE='这里改成服务端授权License' \
    -e ADMIN_USERNAME=这里改成管理后台的账号\
    -e ADMIN_PASSWORD=这里改成管理后台的密码\
    -v ~/data/docker/apifox-api/appdata/logs:/usr/src/app/logs \
    -v ~/data/docker/apifox-api/appdata/static-upload:/usr/src/app/app/public/static-upload \
    -e BASE_URL='license文件中的IP地址或域名地址（如：http://192.168.2.28 ），用于头像、重置密码的服务端地址' \
    -p 80:80 \
    -d apifox-ee
```

###### 4.2 挂载目录说明

> 集群环境部署推荐使用云存储不建议使用挂载存储的方式，常见的云存储方案都支持的：阿里云OSS、腾讯云COS、七牛云、AWS S3 都支持，参考下面的环境变量配置即可

- static_upload 目录，用于存储用户上传的头像、自动化测试的团队报告和文档内上传的图片等（**云存储方案**不需要配置）
- logs 目录，用于存储服务器运行产生的访问日志、错误日志等，主要用于系统接口无法访问时辅助排查（可选）


###### 4.3 环境变量说明

###### 基础配置

| 环境变量名       | 含义                                                         | 默认值 |
| ---------------- | ------------------------------------------------------------ | ------ |
| BASE_URL         | 服务端访问地址（如：http://192.168.2.28 ），用于头像、重置密码的服务端地址。 |        |
| SERVER_BASE_PATH | 服务端基础路径（如：/apifox），v2.1.24 开始支持，需要 LICENSE 授权，请在申请 LICENSE 时将该路径一并提供给 apifox 工作人员 |        |
| MYSQL_HOST       | MySQL 服务器地址                                             |        |
| MYSQL_PORT       | MySQL 服务器端口                                             | 3306   |
| MYSQL_DATABASE   | 数据库名                                                     |        |
| MYSQL_USER_NAME  | MySQL 用户名                                                 |        |
| MYSQL_PASSWORD   | MySQL 用户密码                                               |        |
| REDIS_HOST       | Redis 服务器地址                                             |        |
| REDIS_PORT       | Redis 服务器端口                                             | 6379   |
| REDIS_PASSWORD   | Redis 密码，可为空                                           |        |
| REDIS_DB         | Redis DB，一般填 0                                           | 0      |
| JWT_SECRET       | 登录态等 token 加密秘钥，10~50位随机字符，建议修改           |        |
| LICENSE          | 服务器端授权 License，联系 Apifox 工作人员获取               |        |
| ADMIN_USERNAME   | 后台管理系统的管理员账号                                     |        |
| ADMIN_PASSWORD   | 后台管理系统的管理员密码                                     |        |

###### 系统邮件配置（非必须）

系统邮件，用在“找回密码”时发送重置密码邮件

| 环境变量名      | 含义                              | 默认值 |
| --------------- | --------------------------------- | ------ |
| MAILER_HOST     | smtp 服务器地址                   |        |
| MAILER_PORT     | smtp 端口                         |        |
| MAILER_SECURE   | 是否使用 SSL，可选值：true、false |        |
| MAILER_USER     | 发件人邮箱地址                    |        |
| MAILER_PASSWORD | 发件人邮箱密码                    |        |

###### 功能配置（非必须）

| 环境变量名         | 含义                                                         | 默认值           | 其他 |
| ------------------ | ------------------------------------------------------------ | ---------------- | ---- |
| ENABLE_WEB         | 是否启用 WEB 版，填 true 启用，空字符串或不设置为关闭，并且需要授权 License 支持才会生效 |                  |      |
| NOT_FOUND_PAGE_URL | 设置后端服务的 404 的跳转地址，需使用相对路径，以 / 开头     | /help/index.html | 选填 |

###### 实时协同配置（非必须）

:::tip
Apifox 私有化版本需`>= 2.5.1`才能使用实时协同功能
:::

| 环境变量名         | 含义                              | 默认值 | 其他               |
| ------------------ | --------------------------------- | ------ | ------------------ |
| RTM_QUEUE_ENABLE   | 是否开启实时协同                  | false  | 开启实时同步时必填 |
| RTS_ENABLE         | 是否开启实时协同                  | false  | 开启实时同步时必填 |
| RTM_REDIS_HOST     | 实时协同 Redis 服务器地址         |        | 开启实时同步时必填 |
| RTM_REDIS_PORT     | 实时协同 Redis 服务器端口         |        | 开启实时同步时必填 |
| RTM_REDIS_PASSWORD | 实时协同 Redis 密码，无密码可为空 |        | 开启实时同步时必填 |
| RTM_REDIS_DB       | 实时协同 Redis DB                 |        | 开启实时同步时必填 |

##### 静态资源（图片等）存储配置（非必须）

非必须，配置将静态资源的存储方式，默认以本地文件方式存储。

| 环境变量名     | 含义                                                         | 默认值 | 其他 |
| -------------- | ------------------------------------------------------------ | ------ | ---- |
| STORAGE_DRIVER | 文件存储选项，默认为 file, 即本地文件存储。可选云存储: oss = 阿里云OSS, cos = 腾讯云COS, s3 = AWS S3, qiniu = 七牛云 | file   |      |

默认使用本地文件存储，如需使用其他云存储，请参考以下方式设置：

- [AWS S3 接入 (也支持私有 S3 协议)](doc-2355040)
- [阿里云 OSS 接入](doc-2355042)

##### HTTPS 支持配置（非必须）

如需要让客户端以 https 方式访问 Apifox 服务器，则可增加如下配置。

| 环境变量名                      | 含义                                                         | 默认值 | 其他 |
| ------------------------------- | ------------------------------------------------------------ | ------ | ---- |
| SERVER_SSL_CERTIFICATE_FILE     | https 证书文件的容器内路径（挂载证书文件的的方式配置 http，需要挂载证书文件到容器内） |        | 选填 |
| SERVER_SSL_CERTIFICATE_KEY_FILE | https 证书密钥文件的容器内路径（挂载证书文件的的方式配置 https） |        | 选填 |
| SERVER_SSL_CERTIFICATE          | https 证书内容（不挂载的方式配置 https）                     |        | 选填 |
| SERVER_SSL_CERTIFICATE_KEY      | https 证书密钥内容（不挂载的方式配置 https）                 |        | 选填 |

##### 安全性配置（非必须）

| 环境变量名                         | 含义                                                  | 默认值 | 其他                                                         |
| ---------------------------------- | ----------------------------------------------------- | ------ | ------------------------------------------------------------ |
| DISABLE_REGISTER                   | 是否禁止注册功能                                      |        | 可选填：true、false，填 true 表示禁止，不设置表示允许注册；开启禁止注册功能后，只允许手动在后台管理添加账号（SSO 登录也会被禁止创建新账号） |
| PASSWORD_ERROR_RATE_LIMIT_DURATION | 登录错误次数限制【时间间隔】，单位秒                  | 1800   |                                                              |
| PASSWORD_ERROR_RATE_LIMIT_MAX      | 【时间间隔】内允许最多错误次数                        | 15     | 0 表示不限制次数。                                           |
| PASSWORD_TRANSFER_EXTRA_ENCRYPTION | 是否开启密码传输加密                                  |        | 选填，填 true 表示启用                                       |
| SECRET_KEY_FOR_COMMON_CASE         | rsa 加密键值对                                        |        | 例如：'[{"public":"base64编码后的公钥","private":"base64 编码后的私钥"}]' |
| APP_DOMAIN_WHITE_LIST              | CORS 同源策略相关，设置允许访问跨域访问后端服务的域名 |        | 从 2.3.12 开始的服务端版本才支持。示例值：http://apifox.com,https://apifox.com,http://app.apifox.com |

##### 静态文件资源存储（非必须）

主要用来存放客户端安装包和升级包。
:::caution
该功能还在开发中，Coming soon
:::

| 环境变量名      | 含义                                               | 默认值             | 其他 |
| --------------- | -------------------------------------------------- | ------------------ | ---- |
| ASSETS_BASE_URL | 静态大文件（客户端升级包、视频教程等）存储路径前缀 | {BASE_URL}/assets/ |      |

:::tip
请将存放在如下位置

1. 客户端安装包/升级包存放位置：{ASSETS_BASE_URL}/clients/
2. 教程视频文件存放位置：{ASSETS_BASE_URL}/videos/
   :::

#### 4.4 【可选】通过挂载配置文件的方式设置容器内需要的变量

> 此项是可选配置，多数情况推荐用变量配置，如果环境有限制变量值长度才建议启用配置文件方式

自定义配置文件。如下示例，可使用「覆盖模式」，也可使用「补充模式」。

```shell
# 覆盖模式（覆盖启动命令中的 env 配置）
export SERVER_BASE_PATH="/456"

# 补充模式（不覆盖启动命令中的 env 配置）
if [ "$BASE_URL" == "" ]; then
  export BASE_URL="/abc"
fi
```

在容器的启动参数中配置，将自定义的配置文件，挂载到容器内。
 容器内路径，须指定为 **/usr/src/app/startup.conf**

```shell
......
    -v ~/data/docker/apifox-api/startup.conf:/usr/src/app/startup.conf \
......
```


#### 三、系统管理后台

开启系统管理后台，需要启动服务端容器时配置相关的环境变量，如下：

- 系统管理后台账号的环境变量名为：ADMIN_USERNAME
- 系统管理后台密码的环境变量名为：ADMIN_PASSWORD

使用浏览器访问，访问地址为为 `http://服务端访问地址 + /admin`（如：http://192.168.2.28/admin ）

> 其中 http://xxx 为 BASE_URL 变量配置的地址，服务端容器镜像的启动变量里还需要增加设置管理后台的账号和密码环境变量才能登录使用。


#### 四、数据备份

为了保障数据安全，请务必设置每日定时备份数据库，只要备份 mysql 数据库即可。

#### 五、客户端自动更新

1. 自 2.5.1 版本后支持检测更新功能，需要运维部署人员在 docker 镜像启动时新增一个挂载目录，挂载到容器 /usr/src/app/app/public/client-packages 目录内，例如 ：

```shell
-v /data/xxx:/usr/src/app/app/public/client-packages \
```

2. 运维人员将所有客户端产物（包括yml文件）存放至服务器该目录即可，比如上面的 /data/xxx 下

### 验证结果

#### 检查是否运行成功

```shell
docker logs apifox -f
```

提示：Ctrl + C 可以退出查看界面

如果显示如下信息，表示运行成功，否则表示没有启动成功。

```shell
...
...
[master] egg started on http://127.0.0.1:xxxx
```

到此，安装已完成。

如果遇到问题，可以参考 [常见问题](#user-content-%E5%AE%B9%E5%99%A8%E6%9C%8D%E5%8A%A1%E5%90%AF%E5%8A%A8%E5%BC%82%E5%B8%B8)

### 使用Demo

`(服务使用说明内容)`

```
eg:

请访问Demo官网了解如何使用：[使用文档](https://www.aliyun.com)
```

## 问题排查

### 容器服务启动异常

使用命令`docker exec -it apifox sh` 进入容器后执行：

```
nc -vz $MYSQL_HOST $MYSQL_PORT
nc -vz $REDIS_HOST $REDIS_PORT
node env-test.js 
```

通过 docker logs -f apifox 查看容器日志截图或拍照，然后将启动命令（注意模糊部分隐私密钥信息）也截图一下，一起提供到相应微信群咨询即可

一般常见的问题有：

- MYSQL 版本没有跟文档一致，需要 8.0 以上
- Redis 版本没有跟文档一致，需要 6 以上
- MYSQL 账号权限问题，无法创建表等
- Redis 权限问题，连接的为从节点且从节点没有配置允许写，没有写入权限
- MYSQL_HOST 或 REDIS_HOST 配置不正确，改为相应 IP 或 `docker inspect apifox | grep 172` 改为 Gateway IP 即可
- Docker 版本较旧，最低须使用 `>=20.10.0` 以上

### 容器 MYSQL 或 REDIS 连不通

这种情况常见表现是启动出现卡在 `npx sequelize db:migrate` 然后就出现 `existed Node.js` 或 redis 相关关键字。

- 如果不是在同一台机器上的数据库，则需要定位一下是否在不同网段连通不了，或有防火墙、安全组限制引起的。
- 如果是在同一台机器的话，可以检查一下防火墙配置

以下是 CentOS firewall 防火墙的示例：

```bash
# 查看防火墙状态，如果是 running 才需要执行下面的命令
firewall-cmd --state
# 查看现有的规则，看看 MYSQL、REDIS 有没有开放相关的 MYSQL、REDIS 端口允许 docker 访问
iptables -nL
firewall-cmd --zone=public --list-ports
# 针对 docker 网段开放
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="172.17.0.0/24" accept"
```

更多防火墙相关命令：[firewalld](http://www.dowhere.com/2021/06/07/centos7%E7%9A%84firewall-cmd%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4/)、[ufw](https://luciaca.cn/posts/ufw-common-commands/)


### 接口错误定位

- 首先确保能访问服务，通过浏览器访问一下这个接口看看 `/api/v1/configs/client` ，前面的地址是这边部署的入口，一般为服务容器启动的 BASE_URL 对象的值。 

如果浏览器上这个接口也不能访问的话，最有可能是防火墙、云服务器的安全组、负载均衡、本地代理软件等拦截了，提供到相应微信群咨询即可。

- 如果能访问，但客户端特定接口不行的话，则需要查看运行日志，在启动容器的命令里有类似这一行 `-v ~/data/docker/apifox-api/appdata/logs:/usr/src/app/logs \` 。可以通过下面的命令查看这个挂载的目录里的错误日志，查看运行日志截图或拍照，提供到相应微信群咨询即可
- 如果能访问，返回错误码是 0 则为授权 LICENSE 已过期，1 则是授权 LICENSE 不合法

```
tail -1000 ~/data/docker/apifox-api/appdata/logs/apifox-api/common-error.log
```

### 浏览器访问 `/api/v1/configs/client`  发生错误 ERR_CONNECTION_TIME_OUT

最有可能是防火墙、云服务器的安全组、负载均衡、本地代理软件等拦截，可以检查一下机器的防火墙、云服务器的安全组等配置是否有开放相关端口。

- 如果还是不能访问，可以提供 `cat /etc/os-release` 和 `uname -a` 截图到相应微信群咨询即可。

### 配置了云存储，图片上传失败

常见的云存储方案，阿里云OSS、腾讯云COS、七牛云、AWS S3 都已支持。
不同云存储服务上传失败，可能基于多种原因，如账号权限、bucket 权限、跨区访问、网络问题等等。云存储服务的错误信息，可以在当天的服务日志中查询，并进行调试：

```
tail -1000 ~/data/docker/apifox-api/appdata/logs/apifox-api/apifox-api-web.log
```

### 关于客户端替换 license 

1. 客户端版本 >= 2.1.27，客户端界面已经直接支持在设置或网络请求错误的界面选择 `更改授权码` ，直接点击设置即可
2. 客户端版本 < 2.1.27，需要清理一下缓存，可以尝试删除一下缓存，重新录入 license

- windows 的话删除一下这个目录 `C:\Users\Administrator\AppData\Roaming\apifox-pdv`
- mac 的话删除一下这个目录 `~/Library/Application\ Support/apifox-pdv/`

### 如果 Apifox 启动后 80 端口不行

通过运行下列命令
vi /usr/local/openresty/nginx/conf/nginx.conf
找到 client_max_body_size 这一行，前增加一个下面两行内容:

```
map_hash_max_size 26214;
map_hash_bucket_size 26214;
```

重启容器 docker restart apifox

### 服务端报错

#### bind(): /tmp/apifox-backend.sock (Address in use)

报错原因：由于服务器容器实例异常关闭导致，如服务器断电等；
进入到容器实例中，执行一下步骤

1. rm -f /tmp/apifox-backend.sock
2. 手动重启一下容器

### 生成业务代码下载安装生成插件提示错误

1. 通过 https://cdn.apifox.cn/app/static/openapi-generator-cli.jar 手动下载生成代码插件；
2. 将插件手动放到 Apifox 安装目录中即可；


如有更多问题请查看文档 https://apifox-ee-setup.apifox.cn/?pwd=FhUcxj0y

## 联系我们

欢迎访问Apifox官网（[https://apifox.com/](https://apifox.com)）了解更多信息。

联系邮箱：[alextian@apifox.com](alextian@apifox.com)

联系电话：[400-0518-030](400-0518-030)

![Apifox 客户经理](./202312051047964.png)
