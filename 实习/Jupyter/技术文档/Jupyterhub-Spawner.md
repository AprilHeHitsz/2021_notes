

# 问题描述

参考官方代码及技术文档，对jupyter中的spawner进行较为详细的了解。

# 技术背景

## JupyterHub介绍

JupyterHub 是一个用于管理Jupyter Notebook的多用户服务器。它通过生成、管理和代理许多单一的Jupyter Notebook服务器来支持许多用户。

## JupyterHub中Spawner的作用





# 解决方案

本文首先描述Spawner在JupyterHub中的使用场景，使用流程图梳理其工作流程，然后对Spawner的各个状态及其转换条件结合代码进行分析，最后对除生成用户server的其他的spawner进行简单分析。

# 研究结果

## 使用场景

用户登录后，JupyterHub需要为用户启动一个 single-user notebook server。启动server的工作就是由spawner来完成。

启动一个 single-user notebook server有多种方式，例如在本机创建新的notebook server进程、在Docker容器中生成 server、使用SSH在远程服务器上生成server等等。

## 实现

Spawner是可以自定义的，它的主要操作是启动server（.start）、查看spawner的运行状态（.poll）、终止进程（stop）。可以参考[Spawner的开发文档](https://jupyterhub.readthedocs.io/en/stable/reference/spawners.html)。

官方也提供了多种[Spawner的实现](https://github.com/jupyterhub/jupyterhub/wiki/Spawners)，这些实现都是可配置的。

## 源码调研

### 流程

![SpawnerHandler流程图](D:\2020_Sources\2020秋\实习\截图\SpawnerHandler流程图.png)





## Spawner的主要操作

### Spawner.start

为指定用户启动single-user notebook server。

可从self.user获取用户的name、authentication、和server的信息。用self.get_env()获取环境变量等信息。

返回server的（ip，port），此时server应该已经开始运行了。

### Spawner.poll

检查spawner是否还在运行。

返回Spawner进程的退出状态，若还在运行则返回None。

### Spawner.stop

终止进程（server）。必为协程。

当进程结束时返回。



## Spawner 状态

用于保存一些hub重启之后的恢复信息，以实现hub重启、不影响server。比如说，在本地启动server进程的spawner中，state保存服务器进程的ID，（即state.pid保存self.pid）。

对state的操作有如下几种：

- get_state(): 将当前状态保存到state中，返回当前的状态。比如创建了server进程之后，jiangserver的pid，即self.pid赋值给state.pid。
- load_state(): 从数据库中加载当前state，并存储到对应的属性中。比如从数据库中加载state，并将state的pid字段赋值给self.pid。
- clear_state(): (server?)shutdown之后清除所有的状态。比如当server shutdown了的时候，将self.pid赋值为0。

## Spawner表示server状态的属性

- **active**：从一个server被请求直到到它完全终止的时间段的状态。server的pending、ready、running状态都是active的。

  可以配置c.JupyterHub.active_server_limit 以限制active的server的最大个数，默认为0，无限制。

- **ready**：pending完成之后转换到ready，表示server可用了。

- **pending**：当server正在生成（spawn_pending）、server正在终止（stop_pending）、server正在（check？）时都属于pending状态。

## Spawner的选项 

Spawner.options_form是启动server的一些用户选项，（如果定义options_form的话）在spawner启动之前，用户通过填写表单页面进行配置。未定义则直接生成用户的服务器，不显示表单页面。

![image-20210226105507112](C:\Users\hyp19\AppData\Roaming\Typora\typora-user-images\image-20210226105507112.png)





