<!-- TOC -->

- [1. cloudencode](#1-cloudencode)
    - [1.1. 如何实现转码分布式并行](#11-%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E8%BD%AC%E7%A0%81%E5%88%86%E5%B8%83%E5%BC%8F%E5%B9%B6%E8%A1%8C)
        - [1.1.1. 传统转码服务存在的问题](#111-%E4%BC%A0%E7%BB%9F%E8%BD%AC%E7%A0%81%E6%9C%8D%E5%8A%A1%E5%AD%98%E5%9C%A8%E7%9A%84%E9%97%AE%E9%A2%98)
        - [1.1.2. 分布式并行转码服务的方法](#112-%E5%88%86%E5%B8%83%E5%BC%8F%E5%B9%B6%E8%A1%8C%E8%BD%AC%E7%A0%81%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%96%B9%E6%B3%95)
    - [1.2. 如何编译和运行](#12-%E5%A6%82%E4%BD%95%E7%BC%96%E8%AF%91%E5%92%8C%E8%BF%90%E8%A1%8C)
    - [1.3. 使用说明](#13-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)

<!-- /TOC -->
# 1. cloudencode
cloudencode是一个分布式并行转码服务

## 1.1. 如何实现转码分布式并行

### 1.1.1. 传统转码服务存在的问题
对于大视频文件转码，都是按照顺序进行。对于一个大视频文件，只能按照文件顺序在一台服务器上单进程进行转码。<br/>
即使有多台高性能服务器，也无法加快进度

### 1.1.2. 分布式并行转码服务的方法
主要利用文件切片技术，将大文件切成小片后，放入存储，多台服务器的服务进程都各自取分片文件转码，所有分片完成转码后，再组成完整视频文件。<br/>
业务架构组网图如下：<br/>

![分布式并行转码服务架构组网](https://github.com/runner365/cloudencode/blob/master/doc/fenbushi.jpg)
<br/>

主要步骤如下：
* 对大文件进行切片：切片后生成xxx.ts上传到oss存储，并lpush到redis消息队列中
* 所有转码进程读取redis消息队列中的消息，rpop出xxx.ts的文件名，对其进行转码
* 对xxx.ts转码完成后，上传消息到redis通知该分片转码完毕
* 如果所有的ts分片都转码完毕，就启动合并程序，将多个ts小文件合成完整mp4。

## 1.2. 如何编译和运行
编译方法: go build encoder.go
运行: ./encoder -c conf/encode.json -f logs/encode.log

## 1.3. 使用说明
使用说明简要文档: [使用说明](https://gitlab.com/xiaoq_bj/cloudencode/blob/master/doc/howtouse.md)
