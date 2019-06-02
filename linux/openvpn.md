# 搭建openvpn内网平台

### 1、开启内核转发
```
vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
sysctl -p
```

### 2、生成证书
服务端证书，客户端证书，CA证书
[root@m01 ~]# yum install easy-rsa -y        #安装easy-rsa
[root@m01 ~]# cd /opt/
[root@m01 opt]# mkdir easy-rsa            #创建目录
[root@m01 opt]# cd easy-rsa/
[root@m01 easy-rsa]# cp -a /usr/share/easy-rsa/3.0.3/* .        #拷贝默认的文件到当前目录下
[root@m01 easy-rsa]# cp /usr/share/doc/easy-rsa-3.0.3/vars.example vars    #拷贝默认的vars文件并重命名为vars，vim进行编辑
[root@m01 easy-rsa]# vim vars        #去掉以下几处前面的注释，后面内容对部分进行修改，这里只修改了最后一个
```
set_var EASYRSA_DN "cn_only"
set_var EASYRSA_REQ_COUNTRY "US"
set_var EASYRSA_REQ_PROVINCE "California"
set_var EASYRSA_REQ_CITY "San Francisco"
set_var EASYRSA_REQ_ORG "Copyleft Certificate Co"
set_var EASYRSA_REQ_EMAIL "me@example.net"
set_var EASYRSA_REQ_OU "My Organizational Unit"
set_var EASYRSA_NS_SUPPORT "yes"        #此处记得将no改为yes
```
[root@m01 easy-rsa]# ./easyrsa init-pki    #初始化创建pki目录，用于存储一些中间变量及最终生成的证书
```
Your newly created PKI dir is: /opt/easy-rsa/pki        #成功后提示生成的pki目录路径
[root@m01 easy-rsa]# ./easyrsa build-ca    #创建根证书
Verifying - Enter PEM pass phrase:            #提示属于密码123456回车
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:        #提示直接回车，也可手动修改
Your new CA certificate file for publishing is at:
/opt/easy-rsa/pki/ca.crt            #成功后提示生成的ca根证书路径
```
[root@m01 easy-rsa]# ./easyrsa gen-req server nopass    #创建server端证书和private key，nopass表示不加密private key
```
Common Name (eg: your user, host, or server name) [server]:        #提示直接回车，也可手动修改
Keypair and certificate request completed. Your files are:
req: /opt/easy-rsa/pki/reqs/server.req
key: /opt/easy-rsa/pki/private/server.key        #成功后提示生成的server端证书及密钥文件路径
```
[root@m01 easy-rsa]# ./easyrsa sign server server        #给server端证书做签名
```
Type the word 'yes' to continue, or any other input to abort.
Confirm request details: yes        #输入yes回车
Using configuration from ./openssl-1.0.cnf
Enter pass phrase for /opt/easy-rsa/pki/private/ca.key:    #输入密码123456回车
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName :PRINTABLE:'server'
Certificate is to be certified until May 13 07:24:59 2028 GMT (3650 days)
Write out database with 1 new entries
Data Base Updated
Certificate created at: /opt/easy-rsa/pki/issued/server.crt        #成功后提示生成的server端签名文件路径
```
[root@m01 easy-rsa]# ./easyrsa gen-dh    #创建Diffie-Hellman，需要耐心等待较长时间
```
DH parameters of size 2048 created at /opt/easy-rsa/pki/dh.pem    #成功后提示生成的交换文件路径
```
[root@m01 easy-rsa]# ./easyrsa gen-req client nopass    #创建client端证书和private key，nopass表示不加密private key
```
Common Name (eg: your user, host, or server name) [client]:
Keypair and certificate request completed. Your files are:
req: /opt/easy-rsa/pki/reqs/client.req
key: /opt/easy-rsa/pki/private/client.key        #成功后提示生成的客户端证书及密钥文件路径
```
[root@m01 easy-rsa]# ./easyrsa sign client client            #给client端证书做签名
```
Type the word 'yes' to continue, or any other input to abort.
Confirm request details: yes    #输入yes回车
Using configuration from ./openssl-1.0.cnf
Enter pass phrase for /opt/easy-rsa/pki/private/ca.key:    #输入密码123456回车
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName :PRINTABLE:'client'
Certificate is to be certified until May 13 07:30:06 2028 GMT (3650 days)
Write out database with 1 new entries
Data Base Updated
Certificate created at: /opt/easy-rsa/pki/issued/client.crt    #成功后提示生成的client端签名文件路径
```

### 3、配置服务端
#### 3.1、安装openvpn
在安装openvpn之前需要配置epel源
```
[root@m01 ~]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
[root@m01 ~]# yum makecache
[root@m01 ~]# yum -y install openvpn    #安装openvpn
```
3.2、配置
[root@m01 client]# cd /etc/openvpn/
[root@m01 openvpn]# cp /usr/share/doc/openvpn-2.4.6/sample/sample-config-files/server.conf /etc/openvpn/server.conf.bak  #拷贝默认server端配置文件，也可直接进行编辑，不拷贝
[root@m01 openvpn]# grep -Ev '^$|^#|^;' server.conf.bak >server.conf    #最小化配置文件
[root@m01 openvpn]# vim server.conf        #编辑server端配置文件
```
local 172.16.0.100                               # 监听的ip
port 1999                                        # 监听的端口，默认1194，根据需求更改
proto tcp                                        # 使用的协议，默认udp，因为经常要通过vpn传文件对可靠性要求比较高，所以用tcp
dev tun                                          # 使用tun（隧道）模式，openvpn有两种模式，一种是TUN，另一种是TAP
ca   /etc/openvpn/easy-rsa/keys/ca.crt           # 这四条都是指定证书的路径，要确保路径或文件能够访问
cert /etc/openvpn/easy-rsa/keys/vpnserver.crt
key  /etc/openvpn/easy-rsa/keys/vpnserver.key
dh   /etc/openvpn/easy-rsa/keys/dh2048.pem
server 10.8.0.0 255.255.255.0                    # 设置成server模式并给客户端分配的ip段，服务端会用其中.1和.2两个ip，不要和你的内网冲突
ifconfig-pool-persist ipp.txt                    # 当vpn断开或重启后，可以利用该文件重新建立相同的IP地址连接
push "route 172.16.0.0 255.255.255.0"            # 这三条是给客户端推的路由，客户端连上后会根据这个添加路由，vpn服务器后端有几个网段就写几个
push "route 172.16.2.0 255.255.255.0"            # 这些路由的作用是告诉客户端去另一个子网都转发给TUN接口，类似于静态路由
push "route 10.10.10.0 255.255.255.0"
client-config-dir /etc/openvpn/ccd               # 客户端的个性配置目录，比如针对每个客户端推送不同的路由、配置不同的ip
keepalive 10 120                                 # 每10秒ping一次，如果超过120s认为对方已经down了，需要重连
comp-lzo                                         # 在vpn链接上启用压缩，服务端开启客户端也必须开启
max-clients 100                                  # 最多有几个vpn客户端可以连
user nobody                                      # 启动openvpn的用户和组，建议用nobody
group nobody
client-to-client                                 # 允许vpn客户端之间通信
duplicate-cn                                     # 允许多个客户端使用同一个证书登陆，生产环境建议为每个用户都生成自己的证书
persist-key                                      # 通过keepalived检测后重新启动vpn，不重新读取keys，保留第一次使用的keys
persist-tun                                      # 通过keepalived检测后重新启动vpn，一直保持tun或tap设备是linkup
status   /var/log/openvpn/openvpn-status.log     # openvpn的状态日志文件
log      /var/log/openvpn/openvpn.log            # openvpn的日志文件
writepid /var/run/openvpn/server.pid             # pid文件
verb 3                                           # 日志级别
mute 20                                          # 如果连续出现20条相同的日志，只记录一条
# ifconfig-pool-persist ipp.txt        #指定ip地址池，存放每个人使用的ip，不能与duplicate-cn同时使用
```
** 拷贝以下四个文件到服务端目录下 **
[root@m01 openvpn]# cp /opt/easy-rsa/pki/ca.crt .
[root@m01 openvpn]# cp /opt/easy-rsa/pki/issued/server.crt .
[root@m01 openvpn]# cp /opt/easy-rsa/pki/private/server.key .
[root@m01 openvpn]# cp /opt/easy-rsa/pki/dh.pem .
[root@m01 client]# ls /etc/openvpn/
```
ca.crt server.conf server.key dh.pem server.crt
```

** 拷贝以下三个文件到创建的客户端目录下**
[root@m01 client]# cp /opt/easy-rsa/pki/ca.crt .
[root@m01 client]# cp /opt/easy-rsa/pki/issued/client.crt .
[root@m01 client]# cp /opt/easy-rsa/pki/private/client.key .
[root@m01 client]# ls
```
ca.crt client.crt client.key
```

### 4. 启动openvpn
```
systemctl -f enable openvpn@server.service
systemctl start openvpn@server.service
openvpn --genkey --secret /etc/openvpn/ta.key
```
查看配置是否正确
```
openvpn /etc/openvpn/server.conf
```

### 5、配置客户端
下载这三个客户端文件（根证书，客户端证书，客户端密钥）ca.crt  client.crt  client.key

新建名为client.ovpn的客户端配置文件

编辑内容并保存，配置内容与服务端对应，也可以参照服务端默认的客户端配置文件进行编辑，路径为
/usr/share/doc/openvpn-2.4.6/sample/sample-config-files/client.conf
```
client                  # 设置为clinet模式
dev tun                 # 使用tun模式，必须和服务端一致
proto tcp               # 使用的协议，必须和服务端一致
remote 10.0.0.61 1194   # 指定openvpn服务端的ip和端口，有需要可以使用多个做高可用.
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert client.crt
key client.key
ns-cert-type server
comp-lzo              # 压缩数据，服务端有指定客户端也要
verb 3
tls-auth ta.key 1　　 ; 服务器需要设置为0， 客户端需要配置为1.
cipher AES-256-CBC　　; ubuntu和os x上面必须指定，否则会出现错误。
```

** 1. 如何给客户端指定配置，比如不同的客户端不同的ip地址、推送不同路由等**

先在server.conf中指定 client-config-dir目录，能够让openvpn访问的到
```
client-config-dir /etc/openvpn/ccd        #  确保/etc/openvpn/ccd存在
```
在/etc/openvpn/ccd目录下编辑一个与客户端同名的文件，比如想给vpnclient的这个客户端指定配置，那么就是/etc/openvpn/ccd/vpnclient，这些配置可以是
```
ifconfig-push 10.8.0.9 10.8.0.10       # 指定客户端的IP地址，如果想固定客户端的ip可以这样做
push "route 172.16.0.0 255.255.255.0"  # 推送指定的路由，不同的客户端推送不同的路由可以这样做
iroute 172.16.20.0 255.255.255.0       # 忽略路由
push "dhcp-option DNS 202.96.128.166"  # 推送DNS
```

*** 2. openvpn服务端怎么做高可用**

如果vpn服务器只有一台的话就是个单点，如果有需求可以做高可用，做法是准备两台openvpn服务器，它们相关证书、配置都一样（可以做一个然后复制到另一个），然后在客户端的配置文件中指定两个remote
```
remote  第一个vpn服务器ip   端口
remote  第二个vpn服务器ip   端口
```
然后测试下模拟挂掉一个，能不能用、对使用有没有什么影响

***3. 怎么给客户端推送指定的DNS**

可以在服务端server.conf或者ccd下面配置推送给客户端的dns
```
push "dhcp-option DNS 202.96.128.166"
push "dhcp-option DNS 202.96.128.167"
```
这种场景一般是要用到远程内网中的dns，用的不多



