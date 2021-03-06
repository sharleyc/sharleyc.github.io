---
layout:     post
title:      基于Python的后台自动化测试方法
keywords:   博客
categories: [Python]
tags:	    [Python, 后台自动化]
---

如果现有的后台自动化测试框架无法满足你的个性化需求，如果你熟悉Python的使用，那么本文介绍的基于Python的后台自动化测试方法会非常适合你。


# 准备工作


* 安装Python工作环境，考虑版本兼容性，推荐安装2.7版本。
* 了解协议类型和结构，最好是有详细的设计文档，这是后续组包和解包的基础。（笔者的项目是基于UDP的自定义协议格式）
* 获得相关的接口定义脚本，以及接口所在服务器的IP和端口，包括测试环境以及现网的。
* 弄清楚服务器的环境和访问限制。


# 基础知识

* 后台接口测试，简单说包括三个步骤：发包，收包，校验结果，发包前要按照协议结构组包，收包后要按照协议结果解包。
* 用Python发包和收包是非常简单的事情，以UDP协议为例:
![](/images/images_2017/socket_udp.jpg)
* Python2中没有字节(Byte)类型的数据类型，我们可以使用Python的struct模块来处理字节流数据(即二进制数据)。其中fmt见下图说明，这里的使用有一个坑，后面会分享。

>struct.pack_into(fmt, buffer, offset, v1, v2, ...)
Pack the values v1, v2, ... according to the format string fmt and write the packed bytes into the writable buffer starting at position offset. Note that offset is a required argument.
按照指定的格式fmt，将v1,v2...打包到buffer中，其中偏移位置为offset 

>struct.unpack_from(fmt, buffer, offset=0)
Unpack from buffer starting at position offset, according to the format string fmt. The result is a tuple even if it contains exactly one item. The buffer’s size in bytes, minus offset, must be at least the size required by the format, as reflected by calcsize().
按照指定的格式fmt，从偏移位置offset开始解包，返回数据格式是一个元组(v1,v2...)

![](/images/images_2017/fmt.jpg)

* Protocol Buffers，是一种语言无关、平台无关、扩展性好的用于通信协议、 数据存储的结构化数据串行化方法。在程序中使用PB需要先编译PB 描述文件(.proto后缀名的文件)。下载安装protobuf for python，然后cmd进入:\protobuf-2.7\src目录下执行以下命令，生成private_message _pb2.py文件。
>protoc -I=./ --python_out=./ private_message.proto

* PB message的序列号和反序列号。SerializeToString()：将message序列化并返回str类型的结果（str类型只是二进制数据的一个容器而已，而不是文本内容）。ParseFromString(data)：从给定的二进制str解析得到message对象。其使用的简单实例如下：
![](/images/images_2017/python_pb1.jpg) 
![](/images/images_2017/python_pb2.jpg) 
![](/images/images_2017/python_pb3.jpg) 

# 进阶

利用以上基础知识，就能完成基于Python的后台自动化测试了。如果还想更进一步，往后台自动化测试框架发展的话，以笔者经验建议如下：

* 分层设计。最基本的可以抽象成三层，底层包含自动化测试框架的基本功能，比如收发包支持，校验规则的设计，任务的调度，日志的管理，测试报告的生成和发送等，这些都是与业务功能无关的。中间层包含协议结构的封装，组包和解包功能的支持，服务接口的配置信息，公共函数等，这些与业务相关，但不涉及具体的接口用例。最上层是接口用例，接口的参数设定以及对应的校验规则，都在这层完成。
* 在用例中避免硬编码，比如帐号。比较推荐的做法是做一个帐号资源池，脚本申请符合条件的帐号完成测试后，释放资源。这种设计的好处是不需要频繁修改脚本，维护帐号资源池即可，而且还能支持并发模式的脚本执行，进一步提升测试效率。
