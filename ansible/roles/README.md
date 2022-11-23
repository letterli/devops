### role目录结构

```shell

role_name/    --- 角色名称=目录
    files/: 存储一些可以用copy调用的静态文件
    tasks/: 存储任务的目录，此目录中至少有一个名为main.yml的文件，用于定义各task；其它文件需要maim.yml运行"包含"调用
    handlers/: 此目录中至少应该有一个名为main.yml的文件，用于定义各handler；其它文件需要maim.yml运行"包含"调用
    vars/: 此目录中至少应该有一个名为main.yml的文件，用于定义各variable；其它文件需要maim.yml运行"包含"调用
    templates/: 存储由template模块调用的模板文件
    site.yml: 定义哪个主机应用哪个角色

```
