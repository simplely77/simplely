### 背景

>_Linus Torvalds_ 即是linux内核的作者，也是git的作者，只要遵循GPL(General Public License)任何人和机构都可以使用和修改linux，所以才有了那么多发行版

### 基础知识

![[Drawing 2026-04-03 14.17.13.excalidraw]]
几种常见的使用linux的方式

1. 虚拟机

windows常见的虚拟机就不说了，Mac的 __multipass__ 轻量，但是只支持ubantu

multipass常用命令：
![[Pasted image 20260403143828.png]]

2. docker

```shell
docker pull alpine
docker run -it alpine sh
```
