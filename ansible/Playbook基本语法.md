### Playbook基本语法
---

#### 执行Playbook语法

```python
ansible-playbook deploy.yml

# 查看输出细节
ansible-playbook deploy.yml --verbose

# 查看脚本影响哪些hosts
ansible-playbook deploy.yml --list-hosts

# 并发执行脚本
ansible-playbook deploy.yml -f 10
```

#### 完整的playbook脚本示例

最基本的playbook分为三部分：
1) 在什么机器上以什么身份执行； hosts、users
2) 执行的任务都有什么； tasks
3) 善后的任务有什么；  handlers

```yaml
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  user: root
  tasks:
    - name: ensure apache is at the latest version
      yum: pkg=httpd state=latest
    - name: write the apache config file
      template: src=/srv/httpd.j2 dest=/etc/httpd.conf
      notify:
        - restart apache
    - name: ensure apache is running
      service: name=httpd state=started

  handlers:
    - name: restart apache
      service: name=httpd state=restarted

```


#### 主机和用户

hosts: 为主机的IP，或者主机组名，或者关键字all;
user: 在远程以哪个用户身份执行;
become: 切换成其它用户身份执行，值为yes或者no;
become_method: 与became一起用，指可以为'sudo'/'su'/'pbrun'/'pfexec'/'doas';
become_user: 与bacome一起用，可以是root或者其它用户名;

脚本里用became的时候，执行的playbook的时候可以加参数--ask-become-pass

```python
ansible-playbook deploy.yaml --ask-become-pass
```

#### 执行的任务 Tasks

> * tasks是从上到下顺序执行，如果中间发生错误，那么整个playbook会中止。你可以改修文件后，再重新执行。
> * 每一个task的对module的一次调用。使用不同的参数和变量而已。
> * 每一个task最好有name属性，这个是供人读的，没有实际的操作。然后会在命令行里面输出，提示用户执行情况。

####### 语法

task基本语法：
```yaml
tasks:
  - name: make sure apache is running
    service: name=httpd state=running
```

其中name是可选的，也可以简写成下面的例子。
```yaml
tasks:
  - service: name=httpd state=runing
```

写name的task在playbook执行时，会显示对应的名字，信息更友好、丰富。写name是个好习惯！
```shell
TASK: [make sure apache is running] *************************************************************
changed: [yourhost]
```

没有写name的task在playbook执行时，直接显示对应的task语法。在调用同样的module多次后，不同意分辨执行到哪步了。
```shell
TASK: [service name=httpd state=running] **************************************
changed: [yourhost]
```

####### 参数的不同写法

最基本的传入module的参数的方法 key=value
```yaml
tasks:
  - name: make sure apache is running
    service: name=httpd state=running
```

当需要传入参数列表太长时，可以分隔到多行：
```yaml
tasks:
  - name: Copy ansible inventory file to client
    copy: src=/etc/ansible/hosts dest=/etc/ansible/hosts
            owner=root group=root mode=0644
```

或者用yml的字典传入参数
```yaml
tasks:
  - name: Copy ansible inventory file to client
  copy:
    src: /etc/ansible/hosts
    dest: /etc/ansible/hosts
    owner: root
    group: root
    mode: 0664
```

##### Task的执行状态

task中每个action会调用一个module，在module中会去检查当前系统状态是否需要重新执行。
> * 如果本次执行了，那么action会得到返回值changed;
> * 如果不需要执行，那么action得到返回值ok

#### 响应事件 handler

Handlers里面的每一个handler，也是对module的一次调用。而handlers与tasks不同，tasks会默认的按定义顺序执行每一个task，handlers则不会，它需要在tasks中被调用，才有可能被执行。

Tasks中的任务都是有状态的，changed或者ok。 在Ansible中，只在task的执行状态为changed的时候，才会执行该task调用的handler，这也是handler与普通的event机制不同的地方。

##### 应用场景

如果你在tasks中修改了apache的配置文件。需要重起apache。此外还安装了apache的插件。那么还需要重起apache。像这样的应该场景中，重起apache就可以设计成一个handler。


##### 一个handler最多只执行一次
在所有的任务里表执行之后执行，如果有多个task notify同一个handler,那么只执行一次。

```yaml
---
- hosts: webserver
  user: root
  tasks:
    - name: copy file
      copy:
        src: /data/github/booknotes/devops/ansible/test.txt
        dest: /root/ts.txt
      notify:
        - call handler
    - name: copy other file
      copy:
        src: /data/github/booknotes/devops/ansible/tests.txt
        dest: /root/tst.txt
      notify:
        - call handler
  handlers:
    - name: call handler
      debug:
        msg: 'call handler'
```

###### action是Changed ,才会执行handler

只有当TASKS种的action的执行状态是changed时，才会触发notify handler的执行。

下面的脚本执行两次,执行结果是不同的:
> * 第一次执行是，tasks的状态都是changed，会触发两个handler
> *第二次执行是
  第一个task的状态是OK，那么不会触发handler "call first handler",
  第二个task的状态是changed，触发了handler "call second handler"

```yaml
- hosts: webserver
  user: root
  tasks:
    - name: copy file
      copy:
        src: /data/github/booknotes/devops/ansible/test.txt
        dest: /root/ts.txt
      notify:
        - call first handler
    - name: copy other file
      copy:
        src: /data/github/booknotes/devops/ansible/tests.txt
        dest: /root/tsdt.txt
      notify:
        - call second handler
  handlers:
    - name: call first handler
      debug: msg="call first handler"
    - name: call second handler
      debug: msg="call second handler"

```

##### 按Handler的定义顺序执行

handlers是按照在handlers中定义个顺序执行的，而不是安装notify的顺序执行的。
下面的例子定义的顺序是1>2，notify的顺序是2>1，实际执行顺序：1>2。

```yaml
---
- hosts: webserver
  user: root
  tasks:
    - name: copy file
      copy:
        src: /data/github/booknotes/devops/ansible/test.txt
        dest: /root/ts.txt
      notify:
        - call second handler
    - name: copy other file
      copy:
        src: /data/github/booknotes/devops/ansible/tests.txt
        dest: /root/tsdt.txt
      notify:
        - call first handler
  handlers:
    - name: call first handler
      debug: msg="call first handler"
    - name: call second handler
      debug: msg="call second handler"

```
