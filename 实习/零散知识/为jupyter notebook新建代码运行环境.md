# 为jupyter notebook新建代码运行环境
首先要打开anaconda prompt为Anaconda安装nb_conda插件：
```
conda install nb_conda
```
然后使用命令：
```
conda create -n <环境名称> python=3
```
然后cd进入想要写代码的文件夹
```
cd <文件夹路径>
```
激活刚才创建的环境
```
source activate <环境名称> 
```
安装需要的包之后，打开jupyter notebook
```
jupyter notebook
```