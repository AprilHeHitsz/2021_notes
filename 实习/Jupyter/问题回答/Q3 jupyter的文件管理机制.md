Q3 jupyter的文件管理机制
# Jupyter的文件管理机制
## document
- 是实现IModel接口的一个model实例.
## model
- model接口十分简单，集中于表示document中的data，并使用信号来通知其data的变更.
- 一个文件路径可以对应多个不同的model.
## context
- 每一个model实例都会有一个context与其关联.
- context是存储在model内部的data与文件元数据及文件操作之间的桥梁（？没懂）.
## document widgets
- 一个document的一个view.
- 一个document可以对应多个document widget.
## document registry
- 注册表，用来注册document的类型和factories.

## document manager
- 使用registry来创建document的model和widgets，并控制document的生存期.

参考链接：
https://jupyterlab.readthedocs.io/en/latest/extension/documents.html
