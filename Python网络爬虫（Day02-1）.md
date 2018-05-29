# Python网络爬虫（Day02-1）

---

### 基于阿里云服务器的```CentOS7.3```的```Redis```的安装及测试
阿里云服务器的Redis的安装及测试
* 1.```wget```联网获取网络资源 
```
wget http://download.redis.io/releases/redis-3.2.11.tar.gz 
```
* 2.解压缩
```
gunzip redis-3.2.11.tar.gz
```
* 3.解归档
```
tar -xvf redis-3.2.11.tar 
```
* 4.修改```redis.conf```配置文件
```
# 将redis.conf拷贝一份到 root目录下，然后修改
cp redis-3.2.11/redis.conf redis.conf
```
* 5.cd进入```redis-3.2.11```目录
```
cd redis-3.2.11
```
* 6.对```gcc```进行编译器
```
make && make install
```
* 7.绑定阿里云真实```ip```地址（内网地址）,如果想让别人访问
```
# 根据内容4，通过vim redis.conf进入配置文件
# 查看
ipconfig
# 61行
bind 172.18.41.200
# 84行 强烈建议不使用默认端口，自己设置的端口7726
port 7266
# 设置口令
# requirepass code(密码)
requirepass wangmomo-0000
# 启服务器（在后台运行）
redis-server redis.conf &
# 连接redis服务器(redis客户端)，要指定自己的主机在哪，连公网ip
redis-cli -h 112.74.172.000 -p 7266
# 这里需要在阿里云开一个7266的端口
授权密码
auth wangmomo-0000
# 测试---输入ping， 返回 pong，表示 连接走通
# 如果连不上，查看 netstat -ntlp，然后 kill -9 
# 进程号，杀掉进程继续尝试

# python的pycharm测试

C:\python\spider-crawl\day02>python
Python 3.6.5 (v3.6.5:f59c0932b4, Mar 28 2018, 17:00:18) [MSC v.1900 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import redis
>>> client = redis.Redis(host='112.74.172.237', port=7266, password='wangmomo-0726')
>>> client.ping()
True
# redis中的命令。就是这里 的方法 

在python下的pycharm中输入，同时在阿里云服务器中接收。
>>> client.set('username', 'wangmomo')
True
阿里云服务器中接收的内容
112.74.172.000:7266> get username
"wangmomo"
# 默认情况下，redis设置了16个数据库。          
```



