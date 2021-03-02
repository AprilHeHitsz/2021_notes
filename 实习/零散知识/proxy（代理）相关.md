#跟设置代理有关的一些问题
2020/01/20
##设置git，npm，pip（系统）代理
## git:

- 设置代理
```
git config --global http.proxy http://IP:Port
git config --global https.proxy http://IP:Port
```

- 取消设置代理
```
git config --global --unset http.proxy
```
```
git config --global --unset http.proxy
```

- 查看git配置
```
git config --global -l
```

注意，git配置有三个级别，--system(系统级别)、--global（当前用户）、--local（当前环境）.这三个层级是逐层覆盖的.

## npm

- 设置代理
```
npm config set proxy=http://IP:Port
```

- 取消设置代理
```
npm config delete proxy
```
- 设置镜像
```
npm config set registry <镜像，例如https://registry.npm.taobao.org>
```

- 查看当前镜像
```
npm config get registry
```

- 查看当前配置
```
npm config list
```

## pip


- 设置当前环境代理
```
export http_proxy="IP:Port"  
export https_proxy="IP:Port"  
export ftp_proxy="IP:Port"
```

- 也可以在一条pip命令中设置代理
```
pip install --proxy=http://username:password@proxy:port package
```

- 取消设置当前环境下的代理
```
unset http_proxy  
unset https_proxy  
unset ftp_proxy
```

- 查看当前环境下的代理
```
env|grep -i proxy
```



- 设置镜像
```
pip install jieba -i https://pypi.douban.com/simple
```

## conda
- 设置清华源
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud//pytorch/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --set show_channel_urls yes
```

- 显示镜像配置
```
conda config --show channels
```

- 移除清华源
```
conda config --remove channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/

```



## windows查看代理
```
Netsh winhttp show proxy
```


