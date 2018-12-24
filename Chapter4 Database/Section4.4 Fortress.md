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

example: create tables & add data
> ![](res/fortress01.png)

```python
import sqlalchemy
from sqlalchemy import Table, Column, Integer, String, ForeignKey, UniqueConstraint
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy_utils import ChoiceType

Base = declarative_base()


class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    username = Column(String(32), unique=True)
    password = Column(String(64))

    group_id = Column(Integer, ForeignKey('groups.id'))
    group = sqlalchemy.orm.relationship('Group', backref='users')

    def __repr__(self):
        return self.username


group_m2m_remote = Table('group_m2m_remote', Base.metadata,
                         Column('group_id', Integer, ForeignKey('groups.id')),
                         Column('remote_id', Integer, ForeignKey('remotes.id')))


class Group(Base):
    __tablename__ = 'groups'
    id = Column(Integer, primary_key=True)
    groupname = Column(String(32))

    remotes = sqlalchemy.orm.relationship(
        'Remote', secondary=group_m2m_remote, backref='groups')

    def __repr__(self):
        return self.groupname


class Host(Base):
    # remote hosts
    __tablename__ = 'hosts'
    id = Column(Integer, primary_key=True)
    hostname = Column(String(32))
    ip = Column(String(40), default='localhost')
    port = Column(String(5), default=22)

    def __repr__(self):
        return self.hostname


class RemoteUser(Base):
    __tablename__ = 'remote_users'
    # 联合unique
    __table_args__ = (UniqueConstraint(
        'auth_type', 'username', 'password', name='auth_user_passwd_uc'),)

    id = Column(Integer, primary_key=True)
    username = Column(String(32))
    AuthTypes = [('ssh-password', 'SSH/Password'), ('ssh-key', 'SSH/KEY')]
    auth_type = Column(ChoiceType(AuthTypes))
    password = Column(String(64))

    def __repr__(self):
        return f'<RemoteUser={self.username}>'


class Remote(Base):
    __tablename__ = 'remotes'
    id = Column(Integer, primary_key=True)

    host_id = Column(Integer, ForeignKey('hosts.id'))
    remote_user_id = Column(Integer, ForeignKey('remote_users.id'))

    host = sqlalchemy.orm.relationship('Host')
    remote_user = sqlalchemy.orm.relationship('RemoteUser')

    def __repr__(self):
        return f'<{self.remote_user.username}@{self.host.ip}:{self.host.port}>'


engine = sqlalchemy.create_engine(
    "mysql+mysqldb://grey:xxxxxx@localhost/china?charset=utf8", echo=True)
Base.metadata.create_all(engine)

Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()

# add hosts
h0 = Host(hostname='Alibaba', ip='192.168.1.2', port='80')
h1 = Host(hostname='AWS', ip='192.168.1.3', port='5555')
h2 = Host(hostname='Google', ip='192.168.1.5', port='9999')
h3 = Host(hostname='Azure', ip='192.168.1.8', port='7777')
s.add_all([h0, h1, h2, h3])

# add remoteusers
ru0 = RemoteUser(username='root', auth_type='ssh-password', password='666666')
ru1 = RemoteUser(username='nginx', auth_type='ssh-key')
ru2 = RemoteUser(username='mysql', auth_type='ssh-password', password='555555')
ru3 = RemoteUser(username='apache', auth_type='ssh-key')
s.add_all([ru0, ru1, ru2, ru3])

# add remotes
r0 = Remote(host=h0, remote_user=ru0)
r1 = Remote(host=h0, remote_user=ru1)
r2 = Remote(host=h0, remote_user=ru2)
r3 = Remote(host=h1, remote_user=ru2)
r4 = Remote(host=h1, remote_user=ru1)
r5 = Remote(host=h2, remote_user=ru3)
s.add_all([r0, r1, r2, r3, r4, r5])

# add groups
g0 = Group(groupname='beijing', remotes=[r0, r1, r3])
g1 = Group(groupname='shenzhen', remotes=[r1, r2, r4])
g2 = Group(groupname='shagnhai', remotes=[r2, r3, r5])
s.add_all([g0, g1, g2])

# add users
u0 = User(username='Moris', password='666', group=g2)
u1 = User(username='Grey', password='123', group=g2)
u2 = User(username='Chris', password='456', group=g1)
u3 = User(username='Tom', password='789', group=g0)
s.add_all([u0, u1, u2, u3])

s.commit()
```

```python
# query
Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()

u1 = s.query(User).filter(User.username == 'Grey').first()
print(u1, u1.group) # Grey shanghai
print(u1.group.remotes) # [<apache@192.168.1.5:9999>, <mysql@192.168.1.2:80>, <mysql@192.168.1.3:5555>]
for remote in u1.group.remotes:
    print(remote.host, remote.remote_user)

# Google <RemoteUser=apache>
# Alibaba <RemoteUser=mysql>
# AWS <RemoteUser=mysql>
```

> 配合上面记录的方式就可以制作堡垒机了