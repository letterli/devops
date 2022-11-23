### Ansible命令行模块
---
命令格式：ansible <组名> -m <模块> -a <参数列表>

#### Command模块
command: ansible的默认模块，不指定-m参数的时候，使用的就是command模块。在远程主机执行命令，不支持管道，重定向等shell的特性。

```shell
ansible 192.168.11.108 -m command -a "ifconfig"

ansible webservers -m command -a "uptime"
```

command常用参数:
> * chdir：在远程主机上运行命令前提前进入目录 
> * creates：判断指定文件是否存在，如果存在，不执行后面的操作
> * removes：判断指定文件是否存在，如果存在，执行后面的操作

```shell
ansible dbservers -a "chdir=/home ls -a"
ansible dbservers -a "creates=/opt/test.txt touch text.txt"
ansible dbservers -a "removes=/opt/test.txt rm -f test.txt"
```

#### Shell模块
在远程主机执行命令，相当于调用远程主机的shell进程，然后在该shell下打开一个子shell运行命令。

```shell
ansible dbservers -m shell -a 'ifconfig ens333| awk "NR==2 {print \$2}"'

ansible dbservers -m shell -a "cat /var/nginx/access.log|awk '{print $1}'|sort|uniq -c|sort -nr|head -n 10"
```

#### Cron模块
在远程主机定义任务计划。其中有两种状态（state）：present表示添加（可以省略），absent表示移除。

```shell
minute hour day month weekday: 分 时 日 月 周
job: 任务计划要执行的命令
name: 任务计划名称

ansible webservers -m cron -a 'minute="10" hour="10,22" day="10" month="*/2" job="/usr/bin/cp -f /var/log/messages /opt" name="cp messages"'
ansible webservers -m shell -a "crontab -l"

#Ansible: cp messages
10 10,22 10 */2 * /usr/bin/cp -f /var/log/messages /opt

```

#### User模块
用户管理模块

```shell
常见的参数：
name：用户名，必选参数
state=present|absent：创建账号或者删除账号，present表示创建，absent表示删除
system=yes|no：是否为系统账号
uid：用户uid
group：用户基本组
shell：默认使用的shell
move_home=yes|no：如果设置的家目录已经存在，是否将已经存在的家目录进行移动
password：用户的密码，建议使用加密后的字符串
comment：用户的注释信息
remove=yes|no：当state=absent时，是否删除用户的家目录

ansible webservers -m user -a 'name="admin"'   # 创建用户admin
ansible webservers -m shell -a 'tail -n 10 /etc/passwd'
snsible webservers -m user -a 'name="admin" state=absent'  # 删除用户admin
```

#### Group模块
用户组管理模块

```shell
常用参数：
name参数：必须参数，用于指定组名称。
state参数：用于指定组的状态，两个值可选，present，absent，默认为 present，设置为absent 表示删除组。
gid参数：用于指定组的gid。如果不指定为随机
system参数：如果是yes为系统组。–可选

ansible dbservers -m group -a 'name="mysql" gid=3306 system=yes'
ansible dbservers -m shell -a 'tail /etc/group'
ansible dbservers -m user -a 'name=ts uid=306 group=mysql system=yes'
ansible dbservers -m shell -a 'id ts'
```

#### Copy模块
用于复制指定主机文件到远程主机。

```shell
常用参数：
dest：指出复制文件的目标及位置，使用绝对路径，如果是源目录，指目标也要是目录，如果目标文件已经存在会覆盖原有的内容
src：指出源文件的路径，可以使用相对路径或绝对路径，支持直接指定目录，如果源是目录则目标也要是目录
mode：指出复制时，目标文件的权限 
owner：指出复制时，目标文件的属主
group：指出复制时，目标文件的属组
content：指出复制到目标主机上的内容，不能与src一起使用

ansible dbservers -m copy -a 'src=/etc/fstab dest=/opt/fstab.bak owner=root mode=600'
ansible dbservers -m copy -a 'src=./test.txt dest=/opt/test1.txt owner=root mode=777 group=root'
ansible dbservers -m copy -a 'content="devops" dest=/opt/ct.txt'
```

#### File模块
设置文件属性。

```shell
常用参数：
owner：修改属主
group：修改属组
mode：修改权限
path=：要修改文件的路径
recurse：递归的设置文件的属性，只对目录有效
yes:表示使用递归设置
state：
touch：创建一个新的空文件
directory：创建一个新的目录，当目录存在时不会进行修改

ansible dbservers -m file -a 'owner=test01 group=mysql mode=644 path=/opt/fstab.bak' #修改属主 属组
ansible dbservers -m file -a 'path=/opt/ansible.txt state=touch'  # 指定路径创建文件
ansible dbservers -m file -a "path=/opt/ct.txt state=absent"  #删除文件 
```

#### Hostname模块
用于管理远程主机上的主机名。

```shell
ansible dbservers -m hostname -a 'name=dbserver'  # 修改远程主机上的主机名
```

#### Ping模块
检测远程主机的连通性。

```shell
ansible dbserver -m ping
```

#### Yum模块
在远程主机上安装与卸载软件包。

```shell
ansible webservers -m yum -a 'name=httpd'
ansible webservers -m yum -a 'name=httpd state=absent'
```

#### Service/Systemd模块
用于管理远程主机上的管理服务的运行状态。

```shell
常用的参数：
name：被管理的服务名称
state=started|stopped|restarted：动作包含启动关闭或者重启
enabled=yes|no：表示是否设置该服务开机自启
runlevel：如果设定了enabled开机自启去，则要定义在哪些运行目标下自启动
 
ansible webservers -m shell -a 'systemctl status httpd'  # 查看web服务器httpd运行状态
ansible webservers -m service -a 'enabled=true name=httpd state=started'    # 启动httpd服务
```

#### Script模块
实现远程批量运行本地的 shell 脚本。

```shell
ansible -m script -a '/path/test.sh'
```

#### Setup模块
facts 组件是用来收集被管理节点信息的，使用 setup 模块可以获取这些信息。

```shell
ansible webservers -m setup  # 获取主机所有facts信息
ansible webservers -m setup -a 'filter=*ipv4'  # 使用filter可以筛选出指定的facts信息
```
