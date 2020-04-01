# pypi server使用

## pypi-server

### 安装

```console
pip install pypiserver
```

### 常规启动

```console
nohup pypi-server -p 3141 -a admin -P 123456  /opt/pypi/packages
```

### http密码启动

1. 安装httpd

```console
yum install httpd
```

2. 生成用户名和密码

```console
htpasswd -c ~/.pypipasswd admin
```

3. 启动服务

```console
nohup pypi-server -p 3141 -P root/.pypipasswd  /opt/pypi/packages
```

### 本地配置

在用户主目录编辑pypirc文件： `vi /home/wjl/.pypirc`

```md
[distutils]
index-servers =
        ppypi

[ppypi]
repository:http://192.168.21.115:3141
username:admin
password:123456
```

### 打包并上传包

```console
python setup.py sdist upload -r ppypi
```
