# Fortress Machine

硬件配置:
- A部门: 200台机器;
- B部门: 300台机器;
- C部门: 400台机器;

用户管理：
- method1: 每一个机器开一个账户, 机器太多会很麻烦
- method2: 一个账号登录某部门的大量机器。在windows上叫做`ad域`, linux上叫做ldap(集中式认证)

ldap的风险: 一个用户可以登录其他部门的机器，一般局域网才会使用ldap

需求:
- 权限可控: 比如，一个root用户给不同的人使用，能够确定是哪个人使用了root
- 用户行为审计: 记录用户行为、禁止用户行为

跳板机 vs 堡垒机:
> ![](res/jump01.png)  
> 堡垒机也是一个ssh服务器，只是在跳板机上添加了审计功能
> ![](res/jump02.png)

运维人员ssh连接堡垒机服务器，然后选择对应的机器或者部门，堡垒机然后ssh进入开发区；堡垒机记录运维人员的所有操作行为，上传的文件，以及返回的数据；
> 管理堡垒机的只有一个人  
> 如果某运维人员离职了，直接让他无法登录堡垒机即可  
> 如果某运维人员，通过堡垒机上传恶意文件到开发区某些机器，该恶意文件扫描端口xx，伪造成xx端口服务，定时创建socket，那么该运维人员可以在外部不通过堡垒机，直接连接这个socket，操作开发区的这些机器；这种防不住，只能是查看堡垒机上可疑上传的文件，并且堡垒机到开发区的ssh只能是单向的，否则危害更大；

堡垒机:
- 商业堡垒机: 齐治堡垒机(PKU用的), ...
- 开源堡垒机: 一款好用的都没有, jumpserver垃圾，jumpserver底层用的就是paramiko的长链接执行类似ssh命令，除非自己从底层修改ssh代码

example: ssh by paramiko

[paramiko example](https://github.com/paramiko/paramiko/tree/master/demos), then `python demo.py`报错，修改源代码`interactive.py`中为`sys.stdout.write(data.decode())`, 然后`python demo.py`

在xshell中使用不能随着窗口变化而变化, 各种效果支持不好

example: DIY 堡垒机
> DIY堡垒机就是基于paramiko的，封装从堡垒机到开发机的那一段ssh;   
> 在paramiko发送命令之前将该命令记录进入数据库

```python
# 找到输入的命令位置，interactive.py修改为
# add here
cmd = []

while True:
    r, w, e = select.select([chan, sys.stdin], [], [])
    if chan in r:
        try:
            x = u(chan.recv(1024))
            if len(x) == 0:
                sys.stdout.write("\r\n*** EOF\r\n")
                break
            sys.stdout.write(x)
            sys.stdout.flush()
        except socket.timeout:
            pass
    if sys.stdin in r:
        x = sys.stdin.read(1)
        if len(x) == 0:
            break

        # add here
        if x == '\r':
            cmd_str = ''.join(cmd)
            print(cmd_str)
            cmd = []
        else:
            cmd.append(x)

        chan.send(x)
```

注意事项：
- 让用户一登录堡垒机服务器就启动堡垒机程序。让用户尽可能少的接触堡垒机服务器；
- 用户退出堡垒机程序就会退出堡垒机服务器；

```bash
# 登录启动堡垒机程序 vim .bashrc最后一行加入
python3 /home/demos/demo.py
source .bashrc # 接下来就会提示输入开发机的ip，用户名，密码
```

如果在机房将开发机网线拔下来，接到内网交换机上，就可以绕过堡垒机服务器执行ssh
> 解决方法：通过堡垒机程序实现权限控制，不让运维知道开发机密码，只让堡垒机服务器管理员(only one)知道root密码；  
> 实现方法：将开发机的用户名密码写入堡垒机服务器的数据库，暴露给运维的只是一堆选项

`pip install sqlalchemy_utils`

