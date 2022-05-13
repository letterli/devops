#### Playbook逻辑控制
---

> * when： 条件判断语句，类似于变成语言中的if;
> * loop： 循环语句，类似于编程语言的中的while;
> * block： 把几个tasks组成一块代码，便于针对一组操作的异常处理等操作;


##### 条件语句When

有时候用户有可能需满足特定条件才执行某一个特定的步骤。
在某一个特定版本的系统上装包，或者只在磁盘空间满了的文件系统上执行清理操作一样。这些操作在Ansible上，使用when语句都非常简单。

主机为Debian Linux立刻关机:
```yaml
---
tasks:
  - name: "shutdown Debian flavored systems"
    command: /sbin/shutdown -t now
    when: ansible_os_family == "Debian"
```

根据action的执行结果，来决定接下来执行的action:
```yaml
tasks:
  - command: /bin/false
    register: result
    ignore_errors: True
  - command: /bin/something
    when: result|failed
  - command: /bin/something_else
    when: result|success
  - command: /bin/still/something_else
    when: result|skipped
```

远程中的系统变量facts变量作为when的条件，用“|int”还可以转换返回值的类型：
```yaml
---
- hosts: web
  tasks:
    - debug: msg="only on Red Hat 7, derivatives, and later"
      when: ansible_os_family == "RedHat" and ansible_lsb.major_release|int >= 6
```

条件表达式:
```yaml
vars:
  epic: true
```

基本款:
```yaml
tasks:
    - shell: echo "This certainly is epic!"
      when: epic
```

否定款：
```yaml
tasks:
    - shell: echo "This certainly isn't epic!"
      when: not epic
```

变量定义款:
```yaml
tasks:
    - shell: echo "I've got '{{ foo }}' and am not afraid to use it!"
      when: foo is defined

    - fail: msg="Bailing out. this play requires 'bar'"
      when: bar is not defined
```

数值表达款:
```yaml
tasks:
    - command: echo {{ item }}
      with_items: [ 0, 2, 4, 6, 8, 10 ]
      when: item > 5
```

##### 循环语句Loop

####### 标准循环

为了保持简洁,重复的任务可以用以下简写的方式:
```yaml
- name: add several users
  user: name={{ item }} state=present groups=wheel
  with_items:
     - testuser1
     - testuser2
```

如果你在变量文件中或者 ‘vars’ 区域定义了一组YAML列表,你也可以这样做:
```yaml
vars:
  somelist: ["testuser1", "testuser2"]
tasks:
  -name: add several user
   user: name={{ item }} state=present groups=wheel
   with_items: "{{somelist}}"
```

使用 ‘with_items’ 用于迭代的条目类型不仅仅支持简单的字符串列表.如果你有一个哈希列表,那么你可以用以下方式来引用子项:
```yaml
- name: add several users
  user: name={{ item.name }} state=present groups={{ item.groups }}
  with_items:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }
```

注意：如果同时使用 when 和 with_items （或其它循环声明）,when声明会为每个条目单独执行.

####### 嵌套循环

循环也可以嵌套:
```yaml
- name: give users access to multiple databases
  mysql_user: name={{ item[0] }} priv={{ item[1] }}.*:ALL append_privs=yes password=foo
  with_nested:
    - [ 'alice', 'bob' ]
    - [ 'clientdb', 'employeedb', 'providerd']
```

或者

```yaml
- name: give users access to multiple databases
  mysql_user: name={{ item.0 }} priv={{ item.1 }}.*:ALL append_privs=yes password=foo
  with_nested:
    - [ 'alice', 'bob' ]
    - [ 'clientdb', 'employeedb', 'providerd']
```

##### 块语句Block

多个action组装成块，可以根据不同条件执行一段语句 ：
```yaml
tasks:
 - block:
     - yum: name={{ item }} state=installed
       with_items:
         - httpd
         - memcached

     - template: src=templates/src.j2 dest=/etc/foo.conf

     - service: name=bar state=started enabled=True

   when: ansible_distribution == 'CentOS'
   become: true
   become_user: root

组装成块处理异常更方便：
```yaml
tasks:
  - block:
      - debug: msg='i execute normally'
      - command: /bin/false
      - debug: msg='i never execute, cause ERROR!'
    rescue:
      - debug: msg='I caught an error'
      - command: /bin/false
      - debug: msg='I also never execute :-('
    always:
      - debug: msg="this always executes"
```
