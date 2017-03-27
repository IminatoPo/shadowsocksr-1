XsAdmin ShadowsocksR端部署教程
=========================
本ss服务端支持多用户流量控制、支持SSR新特性，需配合[xsadmin](https://github.com/alishtory/xsadmin)面板项目Restfull接口使用

# 1. 安装相关依赖
## 1.1 安装Requests/Git等
CentOS:
```
yum install -y git python-setuptools && easy_install pip
```
ubuntu/debian：
```
apt-get install -y python-pip git
```

## 1.2 安装libsodium

如果要使用 salsa20 或 chacha20 或 chacha20-ietf 算法，请安装 [libsodium](https://github.com/jedisct1/libsodium) :

centos：

```
yum install -y epel-release libsodium
```
如果想自己编译，那么可以用以下的命令
```
yum -y groupinstall "Development Tools"
wget https://github.com/jedisct1/libsodium/releases/download/1.0.10/libsodium-1.0.10.tar.gz
tar xf libsodium-1.0.10.tar.gz && cd libsodium-1.0.10
./configure && make -j2 && make install
echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf
ldconfig
```

ubuntu/debian：

```
apt-get install -y build-essential
wget https://github.com/jedisct1/libsodium/releases/download/1.0.10/libsodium-1.0.10.tar.gz
tar xf libsodium-1.0.10.tar.gz && cd libsodium-1.0.10
./configure && make -j2 && make install
ldconfig
```

如果曾经安装过旧版本，亦可重复用以上步骤更新到最新版，仅1.0.4或以上版本支持chacha20-ietf

## 1.3 安装Supervisor
安装supervisor很简单，通过easy_install就可以安装
```
yum -y install python-setuptools
easy_install supervisor
```
安装完成之后，就可以用`echo_supervisord_conf`命令来生成配置文件
```
echo_supervisord_conf > /etc/supervisord.conf
```
supervisor开机脚本
```
wget https://github.com/Supervisor/initscripts/raw/master/redhat-init-mingalevme
mv redhat-init-mingalevme /etc/init.d/supervisord
chmod +x /etc/init.d/supervisord
chkconfig supervisord on  #开机自启动
service supervisord restart  #启动
```

## 1.4 安装依赖
```
pip install requests
```

# 2. 获取项目源码
```
git clone -b manyuser https://github.com/alishtory/shadowsocksr.git
```
执行完毕后此目录会新建一个shadowsocksr目录，其中根目录的（./shadowsocksr）是多用户版

进入根目录初始化配置(假设根目录在~/shadowsocksr，如果不是，命令需要适当调整)：
```
cd ~/shadowsocksr
```

# 3. 配置客户端
## 3.1 配置API
进入xsadmin项目管理员后台，添加节点，然后点击进入编辑节点页面，右上角有一个[API配置信息](:void(0))，点击进入API配置信息页面

复制`user-config.json`配置内容，并在`~/shadowsocksr`文件夹下建立并保存成`user-config.json`文件

同理，复制`config_xsadmin.py`配置内容，并在`~/shadowsocksr`文件夹下建立并保存成`config_xsadmin.py`文件

## 3.2 配置supervisor进程管理
执行以下命令：
```
cat<< EOF >> /etc/supervisord.conf
[program:ssserver]
command = python /root/shadowsocksr/xsadmin_server.py
directory = /root/shadowsocksr
user = root
autostart = true
autorestart = true
redirect_stderr=true
stdout_logfile = /root/shadowsocksr/ssserver.log
EOF
```
重启supervisor：
```
service supervisord restart
```
查看ss的运行log：
```
supervisorctl tail -f ssserver
```
重启ssserver
```
supervisorctl restart ssserver
```



