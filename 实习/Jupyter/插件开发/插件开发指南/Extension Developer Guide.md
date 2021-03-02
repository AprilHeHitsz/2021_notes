# 简介

JupyterLab app由一个core application对象和一组extensions组成。其中，extension几乎提供了所有功能，包括notebooks、document editors、code consoles、terminals、 themes、menu system、status bar甚至与服务器的底层通信。

一个jupyter extension包括很多个jupyterlab plugins，一个plugin是扩展的基本单位。

# 概览

## 扩展的类型

- 源扩展（Soure Extension）：JavaScript (npm) package。
  - 安装时需要rebuild JupyterLab，占用大量时间和内存，需要先装Node.js。
  - 但是和Prebuit Extension相比 ，代码总量更少。
- 预构建扩展（Prebuilt Extension）：从源扩展预构建的一组JavaScript 代码。
  - 预构建：使用jupyterlab提供的工具将源扩展编译成一组不依赖JupyterLab JavaScript的Javascript代码，然后打包成一个pip包或conda包等。

## 扩展包的发布

- 在NPM上作为源扩展发布
- 作为预构建的扩展，以python包的形式发布



