### Ansible任务执行
---

#### Ansible任务执行模式

Ansible 系统由控制主机对被管节点的操作方式可分为两类，即ad-hoc和playbook：

ad-hoc模式(点对点模式)
使用单个模块，支持批量执行单条命令。ad-hoc 命令是一种可以快速输入的命令，而且不需要保存起来的命令。就相当于bash中的一句话shell。

playbook模式(剧本模式)
是Ansible主要管理方式，也是Ansible功能强大的关键所在。playbook通过多个task集合完成一类功能，如Web服务的安装部署、数据库服务器的批量备份等。可以简单地把playbook理解为通过组合多条ad-hoc操作的配置文件。


#### Ansible执行流程

Ansible在运行时，首先读取ansible.cfg中的配置，根据规则获取Inventory中的管理主机列表，并行的在这些主机中执行配置的任务，最后等待执行返回的结果。

#### Ansible命令执行过程

１、加载自己的配置文件，默认/etc/ansible/ansible.cfg；
２、查找对应的主机配置文件，找到要执行的主机或者组；
３、加载自己对应的模块文件，如 command；
４、通过ansible将模块或命令生成对应的临时py文件(python脚本)， 并将该文件传输至远程服务器；
５、对应执行用户的家目录的.ansible/tmp/XXX/XXX.PY文件；
６、给文件 +x 执行权限；
７、执行并返回结果；
８、删除临时py文件，sleep 0退出；




