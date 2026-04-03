###### 背景

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

### 常用命令
#### 1.ls
 - -l，显示更详细信息
 - -a，显示所有文件
 - -h，以可读方式显示文件大小
 - -t、-r、-i 等等
 常用 ls -ltr
#### 2.ln （创建链接文件）
- -s，软链接
![[Pasted image 20260403154125.png]] 

#### 3.cat (查看文件内容)

#### 4.rm （删除文件）
- -r ，递归删除，可用于删除文件夹（文件夹下含有文件）
#### 5.chmod (更改文件权限)
- +r/w/x，添加权限
- u/g/o+r/w/x，给指定对象加权限
- +可以换成-
文件权限：
![[Pasted image 20260403155746.png]]

后面直接接数字可以之间修改权限，例如`chmod 777`、`chmod 644`等
![[Pasted image 20260403160134.png]]

#### 6.touch (更新文件修改时间，不存在则创建)

#### 7.echo (输出)
- 配合`>`重定向符号可以输出到文件中

#### 8.pwd,cd等常见命令

#### 9.cp (复制文件)
```shell
cp source.txt target.txt
```
linux一般都是源文件在前，目标文件在后，其它命令一样
- -r 递归，用于复制目录
#### 10.mv (移动文件/重命名文件)

#### 11.mkdir (创建目录)
- -p，创建多级目录
#### 12.du (查看文件和目录大小，也可以查看目录结构)
- tree，需要使用包管理器下载，更好看目录结构
- -h，人性化查看
