#### 变量
---

##### Playbook中使用的变量

在Playbook中使用，需要用{{ }}引用以来即可：
```yaml
---
- hosts: webserver
  vars:
    apache_config: labs.conf
  tasks:
    - name: deploy haproxy config
      template: src={{ apache_config }} dest=/etc/httpd/conf.d/{{ apache_config }}
```

在Playbook中使用变量文件定义变量

```shell
# 变量文件vars/server_vars.yml的内容为：
cat vars/server_vars.yml

apache_config: labs.conf

```

```yaml
- hosts: webserver
  vars_files:
      - vars/server_vars.yml
  tasks:
      - name: deploy haproxy config
        template: src={{ apache_config }} dest=/etc/httpd/conf.d/{{ apache_config }}
```

####### YAML的陷阱

YAML的陷阱是YAML和Ansible Playbook的变量语法不能在一起好好工作了。这里特指冒号后面的值不能以"{ "开头。

```yaml
- hosts: app_servers
  vars:
    app_path: "{{ base_path }}/22"
```

##### 主机的系统变量(facts)

ansible会通过module setup来收集主机的系统信息，这些收集到的系统信息叫做facts，这些facts信息可以直接以变量的形式使用。

```shell
ansible all -m setup -u root
```

怎样在playbook中使用facts变量呢，答案是直接使用：

```
---
- hosts: webserver
  user: root
  tasks:
    - name: echo system
      shell: echo {{ ansible_os_family }}
    - name: debug debain
      debug: msg="This system is Debain"
      when: ansible_os_family == "Debain"
    - name: debug redhat
      debug: msg="This system is Redhat"
      when: ansible_os_family == "RedHat"
```

使用复杂facts变量:  1)中括号 {{ ansible_ens3["ipv4"]["address"] }} 2) 点号 {{ ansible_ens3.ipv4.address }}

####### 关闭facts

在Playbook中,如果写gather_facts来控制是否收集远程系统的信息.如果不收集系统信息,那么上面的变量就不能在该playybook中使用了

```yaml
- hosts: webserver
  gather_facts: no
```

##### 把运行结果当做变量使用-注册变量

把task的执行结果当作是一个变量的值也是可以的。这个时候就需要用到注册变量，将执行结果注册到一个变量中，待后面的action使用：

```yaml
---
- hosts: webserver
  tasks:
     - shell: ls
       register: result
       ignore_errors: True

     - shell: echo "{{ result.stdout }}"
       when: result.rc == 5

     - debug: msg="{{ result.stdout }}"
```

##### 文件模板中使用变量

在playbook中定义的变量，可以直接在template中使用。

```yaml
---
- hosts: web
  vars:
    http_port: 80
    defined_name: "Hello My name is Jingjng"
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: pkg=httpd state=latest

  - name: Write the configuration file
    template: src=templates/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
    notify:
    - restart apache

  - name: Write the default index.html file
    template: src=templates/index2.html.j2 dest=/var/www/html/index.html

  - name: ensure apache is running
    service: name=httpd state=started
  - name: insert firewalld rule for httpd
    firewalld: port={{ http_port }}/tcp permanent=true state=enabled immediate=yes

  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```

在template index.html.j2中可以直接使用系统变量和用户自定义的变量:
> * 系统变量 {{ ansible_hostname }} , {{ ansible_default_ipv4.address }}
> * 用户自定义的变量 {{ defined_name }}

```html
<html>
<title>#46 Demo</title>

<!--
http://stackoverflow.com/questions/22223270/vertically-and-horizontally-center-a-div-with-css
http://css-tricks.com/centering-in-the-unknown/
http://jsfiddle.net/6PaXB/
-->

<style>.block {text-align: center;margin-bottom:10px;}.block:before {content: '';display: inline-block;height: 100%;vertical-align: middle;margin-right: -0.25em;}.centered {display: inline-block;vertical-align: middle;width: 300px;}</style>

<body>
<div class="block" style="height: 99%;">
    <div class="centered">
        <h1>#46 Demo {{ defined_name }}</h1>
        <p>Served by {{ ansible_hostname }} ({{ ansible_default_ipv4.address }}).</p>
    </div>
</div>
</body>
</html>
```

##### 用命令行传递参数

在release.yml文件里，hosts和user都定义为变量，需要从命令行传递变量值。
```yaml
---
- hosts: '{{ hosts }}'
  remote_user: '{{ user }}'

  tasks:
     - ...
```

使用命令行变量:
```shell
# 在命令行里面传值得的方法：
ansible-playbook 在release.yml --extra-vars "hosts=web user=root"

# 还可以用json格式传递参数：
ansible-playbook 在release.yml --extra-vars "{'hosts':'vm-rhel7-1', 'user':'root'}"

# 还可以将参数放在文件里面：
ansible-playbook 在release.yml --extra-vars "@vars.json"
```
