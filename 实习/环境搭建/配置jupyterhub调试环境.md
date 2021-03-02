# 配置Jupyterhub调试环境

## 问题1：找不到proxy

配置好var.py中的两个变量之后，运行bootstrap.py，报错找不到proxy，信息提示需要安装configurable-http-proxy

1. 复制粘贴，执行 npm -g install configurable-http-proxy。
2. 结果仍然报错。
3. 一番google之后，发现是npm环境变量没有设置好的缘故，导致找不到已经安装好的命令。于是配置环境变量。
4. 配置好环境变量之后再次运行，还是不行，但此时已经能用terminal执行configurable-http-proxy命令了。
5. 学长建议直接使用terminal运行bootstrap.py。

**这里是因为pycharm的环境变量设置不正确，解决办法是打开bootstrap的run configuration，然后在terminal中使用export|grep PATH命令的到的环境变量设置到congfiguration的PATH环境变量里面，重新运行就不再报错了。**

1. 然后遇到了端口被占用的情况。解决该问题后成功运行。

**问题应该在于pycharm的环境变量没有配置好**

## 问题2：登录之后无法进入页面

理解错了，应该进入30443和30444...

## 记录

30444登录：

账号：april

密码：1906067657hyp_HYP