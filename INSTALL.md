基于Light平台的应用程序部署说明

### 目录
----
1. 服务器配置要求
2. 系统及中间件要求

    2.1. 操作系统
    
    2.2. Container要求
    
    2.3. Runtime要求
    
3. 安装步骤

    3.1. 确认系统内核环境, 确保Linux系统内核版本大于3.10
    
    3.2. 安装Docker, Light平台的所有中间件及应用程序都运行在容器中
    
    3.3. 安装GIT工具, 用于下载启动用配置文件及代码
    
    3.4. 安装并启动 DB (数据库)
    
    3.5. 安装并启动 AP (应用程序)
    
    3.6. 安装并启动 CACHE (缓存)
    
    3.7. 安装并启动 LB (负载均衡)
    
4. 备份 & 恢复

    4.1. 备份
    
    4.2. 恢复

### 1. 服务器配置要求(最低)
---

| No. | 类别 | CPU | MEMORY | HDD | NETWORK | 说明 |
|---|---|---|---|---|---|---|
| 1 | AP | 2Core | 4G | 20GB | 1M | 应用程序服务器 |
| 2 | LB | 2Core | 4G | 20GB | 5M | 负载均衡服务器 |
| 3 | Cache | 2Core | 4G | 20GB | 1M | 缓存服务器 |
| 4 | DB | 4Core | 8G | 80GB | 1M | 数据库服务器 |

注: 以上是使用公有云时(UCloud为例)的标准服务器配置要求。
准确的服务器配置需要根据用户应用程序的要件做负荷测试才能得出。

### 2. 系统及中间件要求
----
#### 2.1. 操作系统

| 名称 | 版本 | 内核 | 说明 |
|---|---|---|---|
| ContOS | 7.x | > 3.10 | 64-bit |
| Red Hat Enterprise Linux | 7.x | > 3.10 | 64-bit |

#### 2.2. Container要求

| 名称 | Tag | 描述 | 使用的中间件 |
|---|---|---|---|
| docker.alphabets.cn/data | 0.0.1 | 数据卷 |  |
| docker.alphabets.cn/light | 1.3.1 | Light平台内核 | light-core 0.2.6 |
| docker.alphabets.cn/mongo-alone | 0.0.1 | 数据库 单服务器版 | MongoDB 3.0.4 |
| docker.alphabets.cn/nginx | 0.0.1 | 负载均衡 | Nginx 1.8.0 |
| docker.alphabets.cn/varnish | 0.0.1 | 缓存 | Varnish 4.1 |

#### 2.3. Runtime要求

| 名称 | Tag | 描述 |
|---|---|---|
| docker.alphabets.cn/shoteyesxibei | latest | 应用程序 - 巡店系统 |
| docker.alphabets.cn/admin1.3.1 | latest | 应用程序 - 管理界面 |

### 3. 安装步骤
----

#### 3.1. 确认系统内核环境, 确保Linux系统内核版本大于3.10
```
$ uname -r
```

#### 3.2. 安装Docker, Light平台的所有中间件及应用程序都运行在容器中

- 登陆到服务器, 确保可以使用 ```sudo``` 命令, 或拥有root账户权限。

- 请确保现有的yum包是最新的。

```
# yum update
```

- 安装 docker-engine。

```
# yum -y install http://qiniu.alphabets.cn/lib/docker-engine-1.11.1-1.el7.centos.x86_64.rpm
```

- 安装容器管理工具 docker-compose。

```
# curl -L http://qiniu.alphabets.cn/lib/docker-compose-1.6.2 > /usr/local/bin/docker-compose & chmod +x /usr/local/bin/docker-compose
```

- 启动 docker 服务, 并设置docker为自动启动服务(系统启动时, 可以自动启动)。

```
# systemctl start docker
# systemctl enable docker
```

- 修改数据盘 (如果存储数据用的磁盘与系统磁盘是分别被挂载的, 那么需要将docker的运行路径指向容量更大的数据磁盘)

```
# vi /usr/lib/systemd/system/docker.service
  ExecStart=/usr/bin/docker daemon -g /data/docker -H tcp://0.0.0.0:2375 -H fd://
```

上述例子是, 假设数据盘被 mount 到 ```/data```, 我们把docker的工作目录指向了/data/docker
 
#### 3.3. 安装GIT工具, 用于下载启动用配置文件及代码

```
# yum -y install git
```

#### 3.4. 安装并启动 DB (数据库)

- DB使用的系统资源

    | 项目 | 值 |
    |---|---|
    | 端口 | 57017 |
    | 日志目录 | /data/mongodb/log/mongod.log |
    | 数据文件路径 | /data/mongodb/data |

- 安装数据库服务器

    1.1. 下载部署文件
    
    ```
    # git clone http://git.alphabets.cn/LightCompose.git
    ```

    1.2. 启动DB容器
    
    ```
    # docker-compose -f /opt/LightDeploy/compose/mongo_alone.yml up -d
    ```
    
    1.3. 查看DB容器是否启动正常
    ```
    # docker ps
    ```

- 数据导入

    1.1. 下载导入工具
    ```
    # curl -O http://qiniu.alphabets.cn/lib/mongodb-linux-x86_64-rhel70-3.2.6.tgz
    # tar -zxf mongodb-linux-x86_64-rhel70-3.2.6.tgz
    ```
    
    1.2. 上传数据
    ```
    # bin/mongorestore --host 127.0.0.1 --port 57017 --db LightDB --drop dump/LightDB
    # bin/mongorestore --host 127.0.0.1 --port 57017 --db LightDB --drop dump/ApplicationID
    ```
    
    1.3. 启用数据库密码 (authorization 从 disabled 改为 enabled)
    ```
    # docker exec -it mongo-alone bash
    # vi /etc/mongod.conf
      #authorization: disabled
      authorization: enabled
    # docker restart mongo-alone
    ```

#### 3.5. 安装并启动 AP (应用程序)

- AP使用的系统资源

    | 项目 | 值 | |
    |---|---|---|
    | 端口 | 20000 | 管理界面 实例1 |
    | 端口 | 20001 | 管理界面 实例2 |
    | 端口 | 20100 | 应用程序 实例1 |
    | 端口 | 20101 | 应用程序 实例2 |
    | 日志目录 | /data/ApplicationID/log/ | |
    | 代码目录 | /data/ApplicationID/ | |
    
- 安装应用程序

    1.1. 下载部署文件
    
    ```
    # git clone http://git.alphabets.cn/LightCompose.git
    ```

    1.2. 启动AP容器
    
    ```
    # docker-compose -f /opt/LightDeploy/compose/ApplicationID.yml up -d
    ```

#### 3.6. 安装并启动 CACHE (缓存)

- CACHE使用的系统资源

    | 项目 | 值 |
    |---|---|
    | 端口 | 6081 |
    | 配置文件目录 | /data/varnish |

- 安装应用程序

    1.1. 下载部署文件
    
    ```
    # git clone http://git.alphabets.cn/LightCompose.git
    ```

    1.2. 启动AP容器
    
    ```
    # docker-compose -f /opt/LightDeploy/compose/cache.yml up -d
    ```

#### 3.7. 安装并启动 LB (负载均衡)

- 应用程序使用的系统资源

    | 项目 | 值 |
    |---|---|
    | 端口 | 80 |
    | 端口 | 443 |
    | 配置文件目录 | /etc/nginx/conf.d |
    | 日志文件目录 | /var/log/nginx |

- 安装应用程序

    1.1. 下载部署文件
    
    ```
    # git clone http://git.alphabets.cn/LightCompose.git
    ```

    1.2. 启动AP容器
    
    ```
    # docker-compose -f /opt/LightDeploy/compose/lb.yml up -d
    ```

### 备份 & 恢复
----

#### 4.1. 备份

- 设定定时执行, 该脚本会在每天凌晨1点开始执行 
```
# crontab -e
  00 1 * * * /opt/LightDeploy/backup/dump.sh
```

- 备份策略及内容

    - 每天, 凌晨1点实施全数据的完整备份
    - 数据最多保存28天, 超过28天的数据将会被清除
    - 备份的文件保存在 /data/backup/history/data_yyyymmdd_hhmmss.tar.gz 位置
    - 备份需要的时间取决于数据的大小, 网络带宽, 磁盘性能

#### 4.2. 恢复

- 解压缩备份的数据
```
# tar -zxf /data/backup/history/data_yyyymmdd_hhmmss.tar.gz
```

- 恢复数据
```
# bin/mongorestore --host 127.0.0.1 --port 57017 --drop data_yyyymmdd_hhmmss
```

注: 恢复操作会把所有的数据恢复到备份时间点的状态, 备份时间点以后的数据将丢失, 请谨慎操作。

----
最终更新 2016/06/02 字符科技
