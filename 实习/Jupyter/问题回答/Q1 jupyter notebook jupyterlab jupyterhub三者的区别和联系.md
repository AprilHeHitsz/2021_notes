# jupyter notebook/jupyterlab/jupyterhub三者的区别以及关系
## 区别
 - jupyter notebook:是基于web的用于交互计算的应用程序，可以用来编辑、运行 .ipynb文件。在编辑的过程中可以交替地输入文本、代码以及得到代码块的输出。它可以支持多种语言。
 - jupyterlab：是一个灵活的，可扩展的用于交互计算的用户界面，可以并行地打开和编辑多种格式的文本。它包括了jupyter notebook的所有功能，而且有一些专门为其设计的扩展。
 - jupyterhub：是一个为多用户提供jupyter notebook的集成服务系统，可以生成、管理和代理单用户notebook server的多个实例。

##  关系
jupyterlab包含了所有jupyter notebook的功能，可以取代jupyter botebook且jupyterlab使用与nootbook相同的server；jupyterhub用来提供多用户交互计算环境的管理，可以将这些单用户的计算环境与用户想要访问的infrastructure连接起来，也可提供对jupyterlab和notebook 的远程访问服务。