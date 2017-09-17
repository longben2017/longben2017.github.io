---
title: linux常用命令与操作
date: 2017-06-17 12:00:00
categories:
- 技术分享
tags:
- linux
---

初用linux的人可能深有体会，linux的命令实在是太多了，要记的话肯定是记不全的。所以，只能从实际操作中去练习了，下面就给大家介绍一下工作中经常会用到的一些操作。

<!-- more -->

## 一、常用命令
ls　　        显示文件或目录
&emsp;-l           列出文件详细信息l(list)
&emsp;-a          列出当前目录下所有文件及目录，包括隐藏的a(all)
mkdir         创建目录
&emsp;-p           创建目录，若无父目录，则创建p(parent)
cd               切换目录
touch          创建空文件
echo            创建带有内容的文件。
cat              查看文件内容
cp                拷贝
mv               移动或重命名
rm               删除文件
&emsp;-r            递归删除，可删除子目录及文件
&emsp;-f            强制删除
find              在文件系统中搜索某文件
wc                统计文本中行数、字数、字符数
grep             在文本文件中查找某个字符串
rmdir           删除空目录
tree             树形结构显示目录，需要安装tree包
pwd              显示当前目录
ln                  创建链接文件
more、less  分页显示文本文件内容
head、tail    显示文件头、尾内容

## 二、常用操作
### 1.查找文件
```bash
$ find ./ -name xxx
```
"./"表示查找的路径(此处为当前文件夹)，-name后面接文件名，注意此查询为准确查询，也就是说有一个字符不匹配都会造成查找失败，find命令无需加上-r去遍历子文件

### 2.查找字符串
```bash
$ grep -r "xxx" ./
```
此命令是查询文件中是否存在"xxx"字符串，是模糊匹配。

### 3.zip压缩包中删除文件
```bash
$ zip -d myfile.zip 1.txt
```
表示把1.txt文件从myfile.zip压缩文件中删除

### 4.zip压缩包中添加文件
```bash
$ zip -m myfile.zip 1.txt
```
表示把1.txt文件添加到myfile.zip压缩文件

### 5.批量修改文件内容
```bash
$ sed -i "s/aaa/bbb/g" ccc.sh
```
表示把ccc.sh文件里的所有aaa字符串修改为bbb字符串

### 6.删除文件指定行
```bash
$ sed -20d ccc.sh
```
表示把ccc.sh文件的20行删除

### 7.vi打开文本后搜索后文本会高亮显示，取消高亮
```bash
$ nohl
```
取消高亮的命令，意为no high light

### 8.批量修改文件后缀
```bash
$ rename .c .h *.c ./
```
表示把当前文件夹所有.c后缀的文件修改为.h后缀，此处第一个.c表示需要修改的字段，.h表示修改后的字段，*.c表示目标文件
```bash
$ rename .a '' *.a
```
此命令表示删除所有.a的后缀

### 9.批量添加文件后缀名
```bash
$ ls |xargs -i mv {} {}.txt
```
表示把当前所有文件增加.txt后缀

### 10.把文件内容置空
```bash
$ > test.txt
```
表示把test.txt的内容置空





