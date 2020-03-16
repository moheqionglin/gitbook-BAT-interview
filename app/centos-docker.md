```
yum update
#安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
yum install -y yum-utils device-mapper-persistent-data lvm2
#设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

```

```
yum list docker-ce --showduplicates | sort -r
#可以查看所有仓库中所有docker版本，并选择特定版本安装
Loading mirror speeds from cached hostfile
docker-ce.x86_64            3:19.03.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64            18.06.3.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.2.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.3.ce-1.el7                    docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable

```

```
# yum install -y docker-ce-18.03.1.ce
 yum install 18.03.1.ce-1.el7.centos
 
 systemctl start docker
 # docker login
 #开机启动
 systemctl enable docker
 
systemctl daemon-reload

systemctl restart docker.service


 
 # 下载到 /usr/local 
wget https://www.python.org/ftp/python/3.5.1/Python-3.5.1.tgz
cd /usr/local 
tar -zxvf Python-3.5.1.tgz
ln -s /usr/local/Python-3.5.1 /usr/local/python3
cd /usr/local/python3
./configure --with-ssl  
sudo make && make install

vi /etc/profile

alias python2=/usr/bin/python
alias python3=/usr/local/python3/python
alias python=/usr/local/python3/python

source /etc/profile
pip install docker-compose

vi /usr/bin/yum
把#!/usr/bin/python 修改为 /usr/local/python3/python。
如果修改完以后用yum命令报错，可以修改回来。


```



### 安装jenkins（docker方式安装）
```
#安装拉取jenkins最新版本
docker pull jenkins/jenkins:lts

#查看当前的docker的images
docker images;
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jenkins/jenkins     lts                 d9abe6b78d13        4 weeks ago         570MB

# 确定安装的docker是否在最新版本
docker inspect d9abe6b78d13

Env": [
...
 "JENKINS_VERSION=2.204.2",
 ...
]        
        
```

```
#运行环境初始化

mkdir -p /data/jenkins_home;
chmod -R 766 /data/jenkins_home
# 如果自己当前账户是root账户还要执行如下命令
chown -R 1000:1000 /data/jenkins_home

docker run -d --name jenkins -p 8081:8080 -m 300m --memory-swap=300m -v /data/jenkins_home:/var/jenkins_home jenkins/jenkins:lts 

# 看启动情况
docker ps

CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                               NAMES
3137e1f07029        jenkins/jenkins:lts   "/sbin/tini -- /usr/…"   3 seconds ago       Up 2 seconds        50000/tcp, 0.0.0.0:8081->8080/tcp   jenkins

#看日志
docker logs 3137e1f07029 

# 修改jenkins插件的下载源，加速下载插件速度
cd {你的Jenkins工作目录}/updates
vim default.json 

:1,$s/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g
:1,$s/http:\/\/www.google.com/https:\/\/www.baidu.com/g

# 重启jenkins
```


docker run -u root -d -p 9999:8080 -p 50000:50000 
-v jenkins-data:/var/jenkins_home 
-v /var/run/docker.sock:/var/run/docker.sock 
-v /usr/java/maven:/usr/local/maven 
-e "JAVA_OPTS=-server -Xms300m -Xmx600m -Xss512k -Duser.timezone=Asia/Shanghai"
--name jenkins 
jenkinsci/blueocean
