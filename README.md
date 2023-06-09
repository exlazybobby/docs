张脏包的QQ号：476515621

# Docker常用

linux docker服务

```
systemctl disable docker.service
```



```
docker pull mysql/mysql-server:5.7
```

不会中毒的浏览器

http://54.242.117.82:5800/

```shell
docker run \
-d \
--name=firefox \
-p 5800:5800 \
-p 5900:5900 \
--shm-size 4g \
-e DISPLAY_WIDTH=1366 \
-e DISPLAY_HEIGHT=768 \
swr.cn-north-1.myhuaweicloud.com/iivey/firefox:v1.1
```



## 使用镜像创建mysql

**此处密码设置的是root，可以修改**

```
docker run -d --name mydata --restart=always -p 3309:3306 -v ~/MyData:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root123 registry.cn-hangzhou.aliyuncs.com/feapderd/mysql:5.7.29
```



##### 进入容器，修改权限

**不修改无法使用IP登陆**

```
docker exec -it mysql5.7 bash
```



```
# 登陆mysql
mysql -uroot -p 

# 将root用户的Host 由 localhost 修改为 %
update mysql.user set Host = '%' where User = 'root';

# 刷新(刷新可以使用IP登陆，图形化界面登陆)
flush privileges;

```





##### docker配置主从数据库

https://blog.csdn.net/qq_35354529/article/details/114385396

```shell
docker run --name mysql-stock -v /Users/xubin/MyData/mysql:/var/lib/mysql -p 3309:3306 -e MYSQL_ROOT_PASSWORD=root123 -d --restart=always registry.cn-hangzhou.aliyuncs.com/feapderd/mysql:5.7.29
```



## 使用镜像创建jupyterlab服务

默认密码： 1234 

容器默认工作区域 :/root/workerspace

```shell
docker container run -itd -p 9090:8990 -v /data02/home/group_data/xubin:/root/workerspace --restart=always --name jupyterlab exlobin/jupyterlab:1.0.2 bin/bash
```





## 使用镜像创建redis服务

```shell
docker run -itd --restart=always --name spider-redis -p 6379:6379 -v ~/data/spider_redis:/data registry.cn-hangzhou.aliyuncs.com/feapderd/redis:6.0.10
```



# Docker

```sh
# 停止相同镜像容器
docker stop $(docker ps -a | awk '/clash/ {print $1}')
# 删除所有已退出的容器
docker container rm $(docker container ps -qf status=exited)
# 删除所有已创建的容器
docker container rm $(docker container ps -qf status=created)
# 删除无用镜像
docker image prune -a

```

```sh
# 导出镜像成文件
docker image save quay.io/bitnami/nginx -o nginx.image
docker commit 容器id exlobin/data_ubuntu:1.0.0
# 倒入镜像文件
docker image load -i nginx.image
```

```sh
docker run  -itd -e MYSQL_IP=192.168.1.88 -e MYSQL_PORT=3309 -e MYSQL_DB=NcbiDailyUpdate -e MYSQL_USER_NAME=xubin -e MYSQL_USER_PASS=bin.xu@2021 -e REDISDB_IP_PORTS=192.168.1.89:6379 -e REDISDB_DB=0 --restart=always -v /backup/share/group_data/ncbi_rar:/data -v /data/share/xubin/code:/code --privileged=true --name=ncbi-service  node:latest
```



# Pandas

### py文件导包

```python
import os
import sys

project_path = os.path.abspath(os.path.dirname(os.getcwd()))
sys.path.append(project_path)
```

### 连接mysql数据库

```python
import os
from sqlalchemy import create_engine

database = 'Article'
mysql_engine = create_engine(
    f'mysql+pymysql://{os.environ.get("MYSQL_USER_NAME")}'
    f':{os.environ.get("MYSQL_USER_PASS").replace("@", "%40")}'
    f'@{os.environ.get("MYSQL_IP")}:{int(os.getenv("MYSQL_PORT"))}/{database}')
```

### to_sql()方法参数解析

- name:指定的是将输入接入数据库当做的哪个表
- con：与数据库链接的方式，推荐使用sqlalchemy的engine类型
- [schema](https://so.csdn.net/so/search?q=schema&spm=1001.2101.3001.7020): 相应数据库的引擎，不设置则使用数据库的默认引擎，如mysql中的innodb引擎
- if_exists: 当数据库中已经存在数据表时对数据表的操作，有replace替换、append追加，fail则当表存在时提示*ValueError*。
- index：对[DataFrame](https://so.csdn.net/so/search?q=DataFrame&spm=1001.2101.3001.7020)的index索引的处理，为True时索引也将作为数据写入数据表
- index_label:当上一个参数index为True时，设置写入数据表时index的列名称

# Mongo

```shell
# mongoexport -h IP -d 库名 -c 集合名 --type=导出文件类型（csv/json） -field=指定字段 -o 导出文件名
mongoexport -h 192.168.1.88 -d Accessory -c Figure --type=csv -field=_id -o _id.csv
```



# Jupyter

```
docker container run -itd -p 18990:8990 --restart=always --name jupyterlab -e MYSQL_USER_NAME=xubin -e MYSQL_USER_PASS=bin.xu@2021 -e MYSQL_PORT=3309 -e MONGO_IP=192.168.1.88 -e MONGO_PORT=27017 -e MONGO_DB=SourceCode -v /data/share/xubin:/root/workerspace exlobin/jupyterlab:1.0.2 bin/bash


docker container run -itd -p 18990:8990 --restart=always --name jupyterlab -e MYSQL_USER_NAME=xubin -e MYSQL_USER_PASS=bin.xu@2021 -e MYSQL_PORT=3309 -e MONGO_IP=192.168.1.88 -e MONGO_PORT=27017 -e MONGO_DB=SourceCode -v /:/root/workerspace exlobin/jupyterlab:1.0.2 bin/bash



```

显示内存插件

```sh
pip install jupyterlab-system-monitor
```



# mac 下定时任务

```
crontab [-u user] file crontab [-u user] [ -e | -l | -r ]


参数:
* -u user：用来设定某个用户的crontab服务；
* file：file是命令文件的名字,表示将file做为crontab的任务列表文件并载入crontab。如果在命令行中没有指定这个文件，crontab命令将接受标准输入（键盘）上键入的命令，并将它们载入crontab。
*  -e：编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件。
*  -l：显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容。
*  -r：从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件。
*  -i：在删除用户的crontab文件时给确认提示。
```

crontab的文件格式

```
 * * *  * * 运行的命令
 │ │ │ │  │
 │ │ │ │  └─── 星期几 (0 - 6) (0到6 0代表周日 1周一)
 │ │ │ └──────── 月份 (1 - 12)
 │ │ └───────────── 每月几号 (1 - 31)
 │ └────────────────── 小时 (0 - 23)
 └─────────────────────── 分钟 (0 - 59)
```

创建任务

```
sudo crontab -e//回车后输入密码
//进入VI编辑，输入
* * * * * say hello//这个地方可以放脚本的路径
//保存即可。
//这样每分钟都会听到hello了
```

五个星星依次表示:

```
minute — 分钟，从 0 到 59 之间的任何整数 
hour — 小时，从 0 到 23之间的任何整数 
day — 日期，从 1 到 31之间的任何整数（如果指定了月份，必须是该月份的有效日期） 
month — 月份，从 1 到 12 之间的任何整数（或使用月份的英文简写如 jan、feb等等） 
dayofweek — 星期，从 0 到 7 之间的任何整数，这里的 0 或 7 代表星期日（或使用星期的英文简写如sun、mon 等等）
```

命令语句:

```
crontab -l显示目前所有的任务
crontab -r删除所有的任务
crontab -e编辑任务
```

ps:`上述创建的任务是在root账户下创建的,每次查询删除,编辑都需加上sudo ,如果不加上则表示在当前账户下创建`



# 工具

https://github.com/kingToolbox/WindTerm



# DeepbioGroupData

#### 爬虫管理平台使用

长期维护项目用**regular**账号进行管理

**user:**regular

**Password**:regular@2022



管理员账号

**user:**groupdata

**Password**:groupdata@2022



#### **环境变量配置**

```json
{
  "MYSQL_IP": "192.168.1.88",
  "MYSQL_USER_NAME": "groupdata",
  "MYSQL_USER_PASS": "data@2022",
  "MYSQL_PORT": 3309,
  "MONGO_IP": "192.168.1.88",
  "MONGO_PORT": 27017,
  "MONGO_DB": "SourceCode",
  "WORK_MODE": "INFO",
  "REMOTE_IP": "192.168.1.88",
  "REMOTE_USER": "group_data",
  "REMOTE_USER_PASS": "groupdata",
  "REMOTE_DIR_BASE": "/backup/share/group_data"
}
```

#### groupdata-ftp

http://192.168.1.88:9091/

```sh
# 目录切换
cd /backup/share/group_data
# 服务启动
python3 -m http.server 9091
```





# 自用代码

重置aid

```python
def reset_aid_sql_list(table):
    return [
        f"ALTER TABLE {table} MODIFY aid int(11) ;"
        f"ALTER TABLE {table} MODIFY aid int(11) AUTO_INCREMENT;"
    ]
```

# DeepLearning

```shell
# 环境安装
conda install -c pytorch pytorch torchvision
```

# NPS

nps index : 121.4.80.136:8080



https://www.bilibili.com/video/BV1v5411V7dV/?spm_id_from=333.337.search-card.all.click&vd_source=42fea5fba9883234244c8328c8eb4a67

```
docker run -itd --name nps --net=host --restart=always -v /home/xubin/linux_amd64_server/conf:/conf ffdfgdfg/nps

```

start.bat

```shell
@echo off


::去除黑窗口

if "%1" == "h" goto begin 

mshta vbscript:createobject("wscript.shell").run("%~nx0 h",0)(window.close)&&exit 

:begin

::启动内网穿透，并自动最小化运行




::进去nps文件夹

cd C:\Users\Administrator\Desktop\windows_386_client
::复制nps客户端命令

npc.exe -server=121.4.80.136:8024 -vkey=6acz4qbg1ga9m6jc -type=tcp

pause>nul

::按任意键退出


```

# 收藏网址

网址导航

https://www.mjjloc.com/

apple美国账号

[购买商品 - APP兑换码 专卖店 (appdhm.org)](https://appdhm.org/)

# 账号购买链接

- https://www.feijiji.com/#14
- EDU邮箱 https://www.buyedu.gq/



# ubuntu

## 添加新用户

```shell
#  -r：建立系统账号；-m：自动建立用户的登入目录；-s：指定用户的默认使用shell
sudo useradd -r -m -s /bin/bash userName
# 根据提示输入新用户的密码即可
sudo passwd userName  
# ls /home 出现新用户的目录说明创建成功

# 改变组
usermod -g root username
# 确定是否在root组
id username
uid=1001(username) gid=0(root) groups=0(root)``
```

## 换源update error

```
gpg --keyserver keyserver.ubuntu.com --recv-keys 467B942D3A79BD29
gpg --export --armor 3B4FE6ACC0B21F32 | apt-key add -
```



## 笔记本设置关闭盖子不休眠

```sh
sudo vi /etc/systemd/logind.conf
# 将其中的 #HandleLidSwitch=suspend 去掉 # 号，并改成 HandleLidSwitch=ignore
HandleLidSwitch=ignore
```



## Wi-Fi连接

```sh
# 开启wifi

sudo nmcli r wifi on

# 查看网络设备列表

sudo nmcli dev

# 扫描附近的 WiFi 热点

sudo nmcli dev wifi
# 连接到指定的 WiFi 热点
sudo nmcli device wifi connect "@PHICOMM_12_5G" password "jlc15000860018" ifname wlp6s0
```



# K8S学习

https://www.sealos.io/zh-Hans/

# PM2

https://zhuanlan.zhihu.com/p/58787876

```
# --interpreter指定解释器
pm2 start main.py -x --interpreter python3

```

# GitHub收藏项目

- 大数据相关

  https://github.com/heibaiying/BigData-Notes

- hello算法

  https://github.com/krahets/hello-algo

- 算法

  https://github.com/labuladong/fucking-algorithm

- 计算机基础

  https://github.com/xiaolincoder/CS-Base

- linux相关

  https://github.com/jaywcjlove/linux-command

- python学习

  https://github.com/jackfrued/Python-100-Days

- 人工智能

  https://github.com/tangyudi/Ai-Learn/tree/master

搜索技巧	

in:readme 量化 stars:>1000 language:python



# docker 安装kafka

[文章地址](https://zhuanlan.zhihu.com/p/403783402#:~:text=docker%20run%20-d%20--name%20zookeeper%20-p%202181%3A2181%20-t,3%E3%80%81%E5%90%AF%E5%8A%A8kafka%E5%AE%B9%E5%99%A8%20%28%E4%B8%AD%E9%97%B4%E4%B8%A4%E4%B8%AA%E5%8F%82%E6%95%B0%E7%9A%84172.16.60.81%E6%94%B9%E4%B8%BA%E5%AE%BF%E4%B8%BB%E6%9C%BA%E5%99%A8%E7%9A%84IP%E5%9C%B0%E5%9D%80%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%8D%E8%BF%99%E4%B9%88%E8%AE%BE%E7%BD%AE%EF%BC%8C%E5%8F%AF%E8%83%BD%E4%BC%9A%E5%AF%BC%E8%87%B4%E5%9C%A8%E5%88%AB%E7%9A%84%E6%9C%BA%E5%99%A8%E4%B8%8A%E8%AE%BF%E9%97%AE%E4%B8%8D%E5%88%B0kafka%29%20%23%20%E9%9B%86%E7%BE%A4%EF%BC%9A%E6%90%AD%E5%BB%BA%E5%A4%9A%E4%B8%AAkafka%EF%BC%8C%E5%8F%AA%E9%9C%80%E6%94%B9%E5%8F%98%20%E5%AE%B9%E5%99%A8%E5%90%8D%E3%80%81KAFKA_BROKER_ID%20%E5%92%8C%20%E7%AB%AF%E5%8F%A3%20%EF%BC%88%E5%85%B16%E5%A4%84%EF%BC%89)

- 拉取相关镜像

  ```
  docker pull wurstmeister/zookeeper  
  docker pull wurstmeister/kafka  
  ```

- 启动zookeeper容器

  ```
  docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
  ```

- 启动kafka容器(中间两个参数的172.16.60.81改为宿主机器的IP地址，如果不这么设置，可能会导致在别的机器上访问不到kafka)

  ```
  docker run  -d --name kafka -p 19092:19092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.1.77:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.77:19092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:19092 -t wurstmeister/kafka
  ```

- d

  ```
  ./bin/kafka-console-producer.sh --broker-list localhost:19092 --topic mykafka
  
  ./bin/kafka-console-consumer.sh --bootstrap-server localhost:19092 --topic mykafka --from-beginning
  
  ```

  

[docker-compose一键搭建](https://cloud.tencent.com/developer/article/1623469)

docker-compose.yaml

带管理ui,89服务器在运行

```yaml
version: '3.1'
networks:
  kafka-net: # 网络名
    driver: bridge
services:
  # zookeeper集群
  zoo1:
    image: zookeeper:3.8.0
    container_name: zoo1   # 容器名称
    restart: always       # 开机自启
    hostname: zoo1        # 主机名
    ports:
      - 2181:2181         # 端口号
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
    networks:
      - kafka-net
    deploy:
      placement:
        constraints:
          - node.role == worker
  zoo2:
    image: zookeeper:3.8.0
    container_name: zoo2
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
    networks:
      - kafka-net
    deploy:
      placement:
        constraints:
          - node.role == worker
  zoo3:
    image: zookeeper:3.8.0
    container_name: zoo3
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
    networks:
      - kafka-net
    deploy:
      placement:
        constraints:
          - node.role == worker
  # kafka集群
  kafka1:
    image: 'bitnami/kafka:3.0.0'
    container_name: kafka1
    restart: always       # 开机自启
    hostname: kafka1
    networks:
      - kafka-net
    ports:
      - '19092:9092'
    #    volumes:
    #      - ~/data/kafka1/data:/bitnami/kafka/data
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zoo1:2181,zoo2:2182,zoo3:2183/kafka
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.89:19092
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zoo1
      - zoo2
      - zoo3
    deploy:
      placement:
        constraints:
          - node.role == worker
  kafka2:
    image: 'bitnami/kafka:3.0.0'
    container_name: kafka2
    restart: always       # 开机自启
    hostname: kafka2
    networks:
      - kafka-net
    ports:
      - '19093:9093'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zoo1:2181,zoo2:2182,zoo3:2183/kafka
      - KAFKA_BROKER_ID=2
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9093
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.89:19093
      - ALLOW_PLAINTEXT_LISTENER=yes
    #    volumes:
    #      - ~/data/kafka2/data:/bitnami/kafka/data
    depends_on:
      - zoo1
      - zoo2
      - zoo3
    deploy:
      placement:
        constraints:
          - node.role == worker
  kafka3:
    image: 'bitnami/kafka:3.0.0'
    container_name: kafka3
    restart: always       # 开机自启
    hostname: kafka3
    networks:
      - kafka-net
    ports:
      - '19094:9094'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zoo1:2181,zoo2:2182,zoo3:2183/kafka
      - KAFKA_BROKER_ID=3
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.89:19094
      - ALLOW_PLAINTEXT_LISTENER=yes
    #    volumes:
    #      - ~/data/kafka2/data:/bitnami/kafka/data
    depends_on:
      - zoo1
      - zoo2
      - zoo3
    deploy:
      placement:
        constraints:
          - node.role == worker
  kafka4:
    image: 'bitnami/kafka:3.0.0'
    container_name: kafka4
    restart: always       # 开机自启
    hostname: kafka4
    networks:
      - kafka-net
    ports:
      - '19095:9094'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zoo1:2181,zoo2:2182,zoo3:2183/kafka
      - KAFKA_BROKER_ID=4
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.89:19095
      - ALLOW_PLAINTEXT_LISTENER=yes
    #    volumes:
    #      - ~/data/kafka2/data:/bitnami/kafka/data
    depends_on:
      - zoo1
      - zoo2
      - zoo3
    deploy:
      placement:
        constraints:
          - node.role == worker
  kafka-manager:
    image: sheepkiller/kafka-manager
    restart: always
    container_name: kafka-manager
    hostname: kafka-manager
    ports:
      - "9000:9000"
    depends_on:
      - zoo1
      - zoo2
      - zoo3
      - kafka1
      - kafka2
      - kafka3
      - kafka4
    environment:
      ZK_HOSTS: zoo1:2181,zoo2:2182,zoo3:2183/kafka
      KAFKA_BROKERS: kafka1:19092,kafka2:19093,kafka3:19094,kafka4:19095
      APPLICATION_SECRET: letmein
      KM_ARGS: -Djava.net.preferIPv4Stack=true
    networks:
      - kafka-net

```

```yaml
# 路径/opt/kafka/
# 查看topic状态
./bin/kafka-topics.sh --describe --zookeeper 192.168.1.77:2181 --topic mytopic

# 创建分区Replication为2，Partition为2的topic
# ./bin/kafka-topics.sh --create --zookeeper 192.168.1.77:2181 --replication-factor 1 --partitions 2 --topic mytopic

# 生产
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic newOrder2
# 消费
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic newOrder2 --from-beginning
```

## kafka安装心得，[参考](https://zhuanlan.zhihu.com/p/473289553)

- 需要java环境
- 启动zookeeper
- 设置配置文件server.properties的监听端口为服务器IP（否则无法远程连接）
- 启动kafak

kafka[使用基础](https://zhuanlan.zhihu.com/p/38330574)



# docker 运行桌面ubuntu

### 启动桌面ubuntu

```sh
docker run -itd -p 6080:80 -p 5900:5900 -p 33:22 -p 17000-18000:17000-18000 -e RESOLUTION=1920x1080 -e MYSQL_USER_NAME=xubin -e MYSQL_USER_PASS=bin.xu@2021 -e MYSQL_PORT=3309 -e MONGO_IP=192.168.1.88 -e MONGO_PORT=27017 -e REDISDB_IP_PORTS=192.168.1.88:6379 -e REDISDB_DB=1 -e MONGO_DB=SourceCode -e LANG=C.UTF-8 -v /dev/shm:/dev/shm -v /data/share/xubin:/data/share/xubin -v /backup/share:/backup/share exlobin/data_ubuntu:1.0.2
```

```sh
docker run -itd --shm-size=512m --restart=always -p 6901:6901 -p 33:22 -e VNC_PW=bin.xu@2021 -e MYSQL_USER_NAME=xubin -e MYSQL_USER_PASS=bin.xu@2021 -e MYSQL_PORT=3309 -e MONGO_IP=192.168.1.88 -e MONGO_PORT=27017 -e REDISDB_IP_PORTS=192.168.1.88:6379 -e REDISDB_DB=1 -e MONGO_DB=SourceCode -v /dev/shm:/dev/shm -v /data/share/xubin:/data/share/xubin kasmweb/desktop-deluxe:1.12.0
```

Https://192.168.1.77:6901

User_name = kasm_user

Pass_word = bin.xu@2021

## conda

```sh
sudo -s
apt date
apt install git
wget https://mirrors.bfsu.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh
# 统一条款之后指定目录
/usr/lib/miniconda
cd /usr/lib/miniconda/bin
./conda init


```

## ssh服务

```sh
# 安装ssh服务
sudo apt-get install openssh-server

# 启动ssh服务
sudo /etc/init.d/ssh start

# 开机自动启动ssh命令
sudo systemctl enable ssh
 
# 关闭ssh开机自动启动命令
sudo systemctl disable ssh
 
# 单次开启ssh
sudo systemctl start ssh
 
# 单次关闭ssh
sudo systemctl stop ssh
 
# 设置好后重启系统
reboot
 
#查看ssh是否启动，看到Active: active (running)即表示成功
sudo systemctl status ssh
 

```

ubuntu安装docker

```sh
# 首先，更新软件包索引，并且安装必要的依赖软件，来添加一个新的 HTTPS 软件源
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
# 使用下面的 curl 导入源仓库的 GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
#将 Docker APT 软件源添加到你的系统
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# 想要安装 Docker 最新版本，运行下面的命令。如果你想安装指定版本，跳过这个步骤，并且跳到下一步。
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
# 查看docker状态
sudo systemctl status docker
# 非root用户使用docker
sudo usermod -aG docker $USER

# 卸载docker
docker container stop $(docker container ls -aq)
docker system prune -a --volumes
# 使用apt卸载
sudo apt purge docker-ce
sudo apt autoremove
```

# ubuntu安装pm2

1. 更新apt源

   ```sh
   # 使用清华大学 Ubuntu 软件镜像
   sed -i 's/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
   apt update
   ```

2. 安装npm

   ```sh
   # 安装npm
   apt install npm
   
   # 验证安装
   npm -version
   
   # 更换淘宝源
   npm config set registry https://registry.npm.taobao.org
   
   # 验证源是否更换成功
   npm config get registry 
   
   ```

3. 安装nodejs

   ```sh
   # 在全局安装n模块，用n模块管理nodejs
   npm install -g n
   
   # 下载指定版本的nodejs
   n v14.19.0
   
   # 刷新bash, zsh, ash, dash, ksh
   hash -r
   # 如果用的是tcsh请使用下面的命令
   # rehash
   
   # 验证安装
   node -v
   
   ```

4. 安装pm2

   ```sh
   npm install -g pm2
   ```




## 库

- pyecharts



## 收藏网页

https://baijiahao.baidu.com/s?id=1739742102343446593&wfr=spider&for=pc





# 工作中用到的代码

1. pandas读取mongo数据

   ```python
   import pandas  as pd
   import pymongo
   
   client = pymongo.MongoClient(host='192.168.1.88',port=27017)
   db = client['SourceCode']
   collection = db['BenchSci']
   df = pd.DataFrame(list(collection.find()))
   ```



# Git 详细操作命令大全

![图片](https://img-blog.csdnimg.cn/447096e083ba4a4c83794addd2b102fd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZmxvcmFjaHk=,size_20,color_FFFFFF,t_70,g_se,x_16)



下面是常用 的Git 命令清单。几个专用名词的译名如下：

Workspace：工作区
Index / Stage：暂存区
Repository：仓库区（或本地仓库）
Remote：远程仓库

### 使用代理

```shell
git clone https://github.com/zabbix/zabbix-docker.git --config "http.proxy=192.168.1.77:12343"
```

#### 新建代码库

```shell
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]

```

### 配置

Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

```shell
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

### 增加/删除文件

```shell
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]

```

### 代码提交

```shell
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...

```

### 分支

```shell
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
	
```

### 标签

```shell
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]

```

### 查看信息

```shell
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示当前分支的最近几次提交
$ git reflog

```

### 远程同步

```shell
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all

```

### 撤销

```shell
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop
```

### 其他

```shell
# 生成一个可供发布的压缩包
$ git archive
```

