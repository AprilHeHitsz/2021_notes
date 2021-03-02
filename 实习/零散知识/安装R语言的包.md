# 安装R语言的包
2021/01/15
在插件适配过程中，需要安装R语言的包，但是执行过程中出现错误：
> install.packages("languageserver")
将程序包安装入‘/home/april/R/x86_64-pc-linux-gnu-library/4.0’
(因为‘lib’没有被指定)
Warning: 无法在貯藏處https://cloud.r-project.org/src/contrib中读写索引:
  无法打开URL'https://cloud.r-project.org/src/contrib/PACKAGES'
Warning message:
package ‘languageserver’ is not available for this version of R
A version of this package for your version of R might be available elsewhere,
see the ideas at
https://cran.r-project.org/doc/manuals/r-patched/R-admin.html#Installing-packages


原因是需要选择位于中国的镜像
执行：
>chooseCRANmirror()

选择任意中国镜像即可