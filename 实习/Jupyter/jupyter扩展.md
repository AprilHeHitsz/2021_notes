### **Jupyter扩展方式**

- 模块化、扩展性

- 上图中的JupyterLab、Notebook Server、IPython、JupyterHub都是可扩展的。

**JupyterLab扩展（labextension）**

JupyterLab是Jupyter的前端项目，它的一大特点就是便于扩展。通过开发JupyterLab扩展，可以为前端界面增加新功能，例如新的文件类型打开/编辑支持、Notebook工具栏增加新的按钮、菜单栏增加新的菜单项等等。

JupyterLab扩展通常采用TypeScript开发，可参考[开发文档](https://jupyterlab.readthedocs.io/en/stable/developer/extension_dev.html)。

**Notebook Server扩展（serverextension）**

Notebook Server是用Python写的一个基于[Tornado](https://www.tornadoweb.org/en/stable/)的Web服务。

通过Notebook Server扩展，可以为这个Web服务增加新的Handler。增加新的Handler通常有两种用途：

1. 为JupyterLab扩展提供对应的后端接口，用于响应一些需要由服务端处理的事件。例如调度任务的注册需要通过JupyterLab扩展发起请求，由Notebook Server扩展执行。
2. 提供一个前端界面以及对应的后端处理服务。例如jupyter-rsession-proxy，用于在JupyterHub中使用RStudio。

Notebook Server扩展开发文档可参考[开发文档](https://jupyter-notebook.readthedocs.io/en/stable/extending/handlers.html)。

**Jupyter Kernels**

Jupyter用于执行代码的模块叫Kernel，除了默认的ipykernel以外，还可以有其他的Kernel用于支持其他编程语言。例如支持Scala语言的[almond](https://almond.sh/)、支持R语言的[irkernel](https://irkernel.github.io/)，更多详见[语言支持列表](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels)。

**IPython Magics**

IPython Magics就是那些%、%%开头的命令。常见的Magics有 %matplotlib inline，设置Notebook中调用matplotlib的绘图函数时，直接展示图表在Notebook中。执行Magics时，事实上是调用了该Magics定义的一个函数。对于Line Magics（一个%），传入函数的是当前行的代码；对于Cell Magics（两个%），传入的是整个Cell的内容。定义一个新的IPython Magics仅需定义一个函数，这个函数的入参有两个，一个是当前会话实例，可以用来遍历当前会话的所有变量，可以为当前会话增加新的变量；另一个是用户输入，对于Line Magics是当前行，对于Cell Magcis是当前Cell。

IPython Magics在简化代码方面非常有效，我们开发了%%spark、%%sql用于创建Spark会话以及SQL查询。另外很多第三方的Magics可以用来提高我们的开发效率，例如在开发Word2Vec变种时，使用%%cython来进行Cython和Python混合编程，省去编译加载模块的工作。

IPython Magics开发文档可参考：

https://ipython.readthedocs.io/en/stable/config/custommagics.html。

**IPython Widgets（ipywidgets）**

IPython Widgets是一种基于Jupyter Notebook和IPython的可交互控件。与普通可视化不同的是，在控件上的交互会触发和Python的通信并执行相应的代码，Python上相应的动作也会触发界面实时变化。

IPython Widgets在提供工具类型的功能增强上非常有用，基于它，我们实现了一个线上排序服务的调试和复现工具，用于展示排序结果以及指定房源在排序过程中的各种特征以及中间变量的值。IPython Widgets的开发可以通过组合现有的Widgets实现，也可以完全自定义一个，IPython Widgets开发文档可参考：

[https://ipywidgets.readthedocs.io/en/stable/examples/Widget%20Custom.html](https://ipywidgets.readthedocs.io/en/stable/examples/Widget Custom.html)。

**扩展JupyterHub**

**Authenticators**

JupyterHub是一个多用户系统，登录模块可替换，通过实现新的Authenticator类并在配置文件中指定即可。通过这个扩展点，我们实现了使用内部SSO系统登录JupyterHub。Authenticator开发文档可参考：

https://jupyterhub.readthedocs.io/en/stable/reference/authenticators.html。

**Spawners**

当用户登录时，JupyterHub需要为用户启动一个用户专用Notebook Server。启动这个Notebook Server有多种方式：本机新的Notebook Server进程、本机启动Docker实例、K8s系统中启动新的Pod、YARN中启动新的实例等等。每一种启动方式都对应一个Spawner，官方提供了多种Spawner的实现，这些实现本身是可配置的。如果不符合需求，也可以自己开发全新的Spawner。由于我们需要实现Spark接入，对K8s的Pod有新的要求，所以基于KubeSpawner定制了一个Spawner来解决Spark连接集群的网络问题。Spawner开发文档可参考：

https://jupyterhub.readthedocs.io/en/stable/reference/spawners.html。