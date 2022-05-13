### Ansible简介
---

#### Ansible是什么？
ansible自动化运维工具，基于Python开发，集合了众多运维工具（puppet、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。

ansible是基于paramiko开发的,并且基于模块化工作，本身没有批量部署的能力。真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架。
ansible不需要在远程主机上安装client/agents，因为它们是基于ssh来和远程主机通讯的。
ansible目前已经已经被红帽官方收购，是自动化运维工具中大家认可度最高的，并且上手容易，学习简单。

#### Ansible特点
１、部署简单，只需在主控端部署Ansible环境，被控端无需做任何操作；
２、默认使用SSH协议对设备进行管理；
３、有大量常规运维操作模块，可实现日常绝大部分操作；
４、配置简单、功能强大、扩展性强；
５、支持API及自定义模块，可通过Python轻松扩展；
６、通过Playbooks来定制强大的配置、状态管理；
７、轻量级，无需在客户端安装agent，更新时，只需在操作机上进行一次更新即可；
８、提供一个功能强大、操作性强的Web管理界面和REST API接口——AWX平台。

#### Ansible架构图

！[ansible架构图](ansible架构图.pnd)

> * Ansible: Ansible核心程序。
> * Host Inventory: 记录有Ansible管理的主机信息，包括ip、端口、密码等。
> * PlayBooks: “剧本“YAML格式文件，多个任务定义在一个文件中，定义主机需要调用哪些模块来完成的功能。
> * Core Modules:　核心模块，主要操作是通过调用核心模块来完成管理任务。
> * Costom Modeuls:　自定义模块，完成核心模块无法完成的功能，支持多种语言。
> * Plugins: 完成模块功能的补充。
> * Connect Plugins: 连接插件，Ansible和Host通信使用。
