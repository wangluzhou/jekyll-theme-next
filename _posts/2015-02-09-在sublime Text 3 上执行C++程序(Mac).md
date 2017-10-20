---
title: 在sublime Text 3 上执行C++程序(Mac)
date: 2015-02-09 20:57:45
categories:
- 技术
tags:
- C++
---
Sublime是一款非常不错的代码编辑器，对于我而言，sublime最大的好处就是C++代码和自己的读书笔记能在一个界面下进行编写，非常方便。但是sublime默认的C++编译方式有两个很大的缺点，一个是无法编译多个源文件，另一个就是无法在程序运行之后输入参数(也就是说，我们在sublime中无法使用std::cin).

<!-- more -->

#利用makefile执行含多个源文件的C++程序#

其实我对makefile并不是十分了解，只是用其去执行过几个小程序罢了，不过对于目前的我来说也足够用了。要使用sublime执行makefile，直接添加一个building tool就可以了。
到~/Library/Application Support/Sublime Text 3/Packages/User/目录下建立makefile.sublime-build文件。并在里面写入：

```
	{
		"file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
		"working_dir": "${file_path}",
		"selector": "source.makefile",
		"cmd": ["bash", "-c", "make clean && make -f '${file}'"],
		"variants":
		[
			{
				"name": "Clean",
				"cmd": ["make", "clean"]
			},
			{
				"name": "Run",
				"cmd": ["make", "run"]
			}
		]
	}
```
这是一个sublime-build文件的基本框架，其实只要前四行就可以了，指定文件名，工作路径（即文件所在路径），文件类型以及命令就行了。
不知道为什么，如果不删除`*.o`文件(以o结尾的文件)的话，即使你修改了源代码，再执行makefile文件还是修改前的情况，似乎压根就没有编译。
我想应该是makefile的执行逻辑是先看是否有`*.o`文件。如果有，就直接执行，没有的话在执行下面生成`*.o`文件的命令。所以每次执行前，最好删除掉所有的`*.o`这些目标文件。
很简单，在makefile里面加一个clean的目标:
```makefile
	all:SimpleMCMain1.out
		./SimpleMCMain1.out #&& open -a Terminal.app SimpleMCMain1.out
	SimpleMCMain1.out:SimpleMCMain1.o Random1.o
		g++ -std=c++11 SimpleMCMain1.o Random1.o -o SimpleMCMain1.out
	SimpleMCMain1.o:
		g++ -std=c++11 -c SimpleMCMain1.cpp
	Random1.o:
		g++ -std=c++11 -c Random1.cpp

	clean:
		rm -rf *.o
```

然后在makefile.sublime-build里面再加入 `make clean` 的命令同时用&&符号连接原来的命令。


#利用终端解决`cin`无法运行的问题#
解决这个问题很简单，只要在`g++`的命令之后添加`open -a Terminal.app`并用`&&`符号连接这两个命令就可以了，这样每次执行程序，sublime就会打开终端，你就可以在终端输入参数。

```
	{
		"file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
		"working_dir": "${file_path}",
		"selector": "source.c, source.c++",    
        "cmd": ["bash", "-c", "g++ -std=c++11 '${file}' -o '${file_path}/${file_base_name}.out' && open -a Terminal.app '${file_path}/${file_base_name}.out'"],
		"variants":
		[
			{
				"name": "Run",
				"cmd": ["bash", "-c", "g++ '${file}' -o '${file_path}/${file_base_name}' && '${file_path}/${file_base_name}'"]
			}
		]
	}
```

#补充#
`variants`里面的配置我也不太理解，但是目前为止也还用不上，我也就懒得去管了。
我在makefile里面也尝试过去执行`open -a Terminal.app` 但是总是报错，不知道为什么。不过即使不能用`cin`也是没关系的，直接建立一个`data.h`头文件，在里面加入所有你需要输入的全局变量和其对应的值，就可以了。
```C++
double expiry = 2;
double strike = 2;
double spot = 1;
double vol = 0.5;
double r = 0.1;
unsigned long numberOfPaths = 1000000;
```
