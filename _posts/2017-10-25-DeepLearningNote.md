---
title: 深度学习
date: 2017-10-25 23:42:21
categories:
- 技术
tags:
- 深度学习
---

# Docker和TensorFlow环境

## 1. Docker学习
## 2 环境配置
### 2.1 Docker安装
到docker官网下载指定操作系统的docker容器。
### 2.2 TensorFLow安装

原本的安装命令如下
```
docker run -it -p 8888:8888 tensorflow/tensorflow
```

参数解释：
- `run`：以某个image为模板构建自己的容器
- `-p 8888:8888`:容器8888端口映射主机的8888端口
- `-it`:用命令行和docker容器里面的程序进行交互

这个命令第一次运行会检查本地是否有docker镜像，如果没有就会从docker hub上下载docker镜像，但是由于国内网络的问题，下载的速度非常慢。因此我们需要使用DAOCLOUD加速器进行加速配置。([参考博客](http://www.jianshu.com/p/d896ec46db66))。

首先登录[DAOCLOUD官网](https://www.daocloud.io/mirror#accelerator-doc)进行注册,注册完成之后进入到自己的管理界面，点击右上角的加速符号。
![image](https://github.com/wangluzhou/DeepLearning101-002/blob/master/ch0/note/Screenshot%20at%20Oct%2016%2023-09-26.png)
在配置docker加速器的栏目下选择linux,mac或者windows对应的镜像地址，然后依次点击mac最上方的`docker->preferences->Daemon->Registry` Mirrors中添加刚才的镜像地址即可:
![image](https://github.com/wangluzhou/DeepLearning101-002/blob/master/ch0/note/Screenshot%20at%20Oct%2016%2023-16-21.png)
此时下载速度就会灰常快！

### 2.3 docker的一些常用命令
1. 查看容器状态
```
docker ps [-a]
```
其中
`-a`:可以用于查看 已经关闭的容器

- 查看docker的版本
```
docker version
```

2. 列出机器上的镜像
**注意**：镜像不是容器，镜像应该叫做模板，一个镜像可以创建很多容器。
```
docker images
```

3. 容器重命名
修改docker容器的name对应的值
```
docker rename xxx yyy
```

4. 启动docker容器
用`start`命令：
```
docker start -i (name or id)
```
> 其中
id只需要写前几位就可以（docker可以识别唯一即可）
`-i`：表示输出
详细的命令可以参考[docs.docker.com](https://docs.docker.com/engine/reference/commandline)

5. docker映射本地路径
将本地的课程目录挂载到docker容器中：
```
docker run -it -p 8888:8888 -v ~/github/DeepLearning101-002/:/DeepLearning101-002 -w /DeepLearning101-002 tensorflow/tensorflow
```

> 其中
`-w`:设置运行的命令的工作目录。在这个示例中，工作目录更改为挂载的卷。
如果是windows，则只能映射到`c:\Users\your-name\`目录及其子目录下（即`~/`目录下）。

，这样创建学习环境的容器的命令是：
```
docker run --name dl -it -p 8888:8888 -v /c/Users/Public/researcher/Doing-Projects/workspace/DeepLearning101-002:/DeepLearning101-002 -w /DeepLearning101-002 tensorflow/tensorflow
```
6. 命令行连接Docker容器
```
docker exec -it dl /bin/bash
```
只是不知道最后那个`/bin/bash`是什么意思？

7. Docker 切换Python环境
tensorflow镜像支持用tag`-py3`的方式设置Python版本
```
docker run -it -p 8888:8888 -v ~/WorkStation/DeepLearning101-002/:/WorkStation/DeepLearning101-002 -w /WorkStation/DeepLearning101-002 tensorflow/tensorflow:latest-py3
```


8. 脚本执行方法：
  1. 使用`!python`来执行本地脚本
  2. 在jupyter目录页面new一个terminal
  3. 执行`!pip`安装新的包

## 3. 神经网络
1. Top-to-Bottom
2. bottom-to-Top

神经网络的结构：
1. 输入层
2. 隐含层
3. 输出层

### 4. 深度学习在自然语言处理上的优势
- End-to-end learning vs step-by-step learning
 - 微博自动回复
- 相比一般机器学习算法，通用性、迁移性强
- 效果直逼或赶上传统的算法

## 5. 神经网络参考教程
[Foundations of Deep Learning (Hugo Larochelle, Twitter)](https://www.youtube.com/watch?v=zij_FTbJHsk&feature=youtu.be)

## 参考笔记
[clearboy:Windows下搭建docker环境](https://github.com/AIHackers/DeepLearning101-002/issues/50)


# Ch1：基础语言模型

> Ch1 课程安排
> - 任务1： 参数个数的计算
> - 任务2：实现Naive的语言模型
> - 任务3：实现N-gram语言模型

## 语言模型
语言模型是自然语言处理的基础，任何模型都无法绕过语言模型。

语言模型的三个功能：

通过语言模型和文本数据（语料），我们可以得到句子的概率：
- 错别句子检验
- 新短语发现
- 用该模型**生成**不同的句子：
[像汪峰老师一样去写歌](https://github.com/phunterlau/wangfeng-rnn)

## 机器理解自然语言的方式
### 语法
词性(名词，动词，形容词)，句子成分(主谓宾)
- 词性的功能：
  - 句子的压缩
  - 中文分词：N和V之间的切分
- 语法不能保证语义：火星追杀绿色的梦

### 概率统计
- [齐夫定律](http://www.wikiwand.com/zh-hans/%E9%BD%8A%E5%A4%AB%E5%AE%9A%E5%BE%8B)

$$ {}+1=1 $$

- 香农：信息论

# Ch2: Naive Bayes和Gradient Descent
> 任务一：运用贝叶斯公式
  任务二：实现 Navive Bayes 方法
  任务三：实现 Gradient Descent 算法

## 情绪的理解
情绪的分析最基本的判断就是正面和负面的划分。
[snwnlp语料库](https://github.com/isnowfy/snownlp)
snowNLP是一个Python写的类库，可以方便地处理中文文本内容，一个方便用于处理中文的类库，并且没有NLTK（自然语言处理包），所有的算法都是自己实现的，并且自带了一些训练好了的字典。本程序都是处理unicode编码的，需要自行decode和unicode。

推荐英文论文:Speech and Language Processing - ch6 - naive bayes and sentiment classification
