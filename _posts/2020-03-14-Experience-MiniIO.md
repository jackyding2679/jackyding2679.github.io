---
layout: post
title: 'MinIO初体验'
date: 2020-03-13
author: Jacky
categories: 存储
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: MinIO 对象存储
---

体验MinIO

### 介绍
MinIO是一款高性能对象存储软件，基于Apache License v2.0协议，其API与AWS S3兼容。
可以用MinIO为机器学习，大数据分析以及应用负载构造高性能的基础设施。
github:https://github.com/minio
website:https://min.io/

### 安装
官网介绍：https://docs.min.io/docs/minio-quickstart-guide.html
可以支持容器安装，macOS，GNU/linux，Microsoft Windows和FreeBSD。
安装非常的方便，以linux为例：
```clike
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
./minio server /data
```
就可以启动对象存储服务器，默认监听9000端口，如果系统开启了iptables或者
firewalld，需要配置防火墙放开9000端口。

### 浏览器操作
MinIO默认的access_key和secrete_key为minioadmin
我的测试环境的URL为http://192.168.56.11:9000，用浏览器可以登录系统
![](/assets/img/MinIO_MainPage.jpg)

我创建了一个叫video的桶，并且上传了3个视频文件，可以对桶或者对象做一些
简单的操作，比如设置桶Policy，还可以生成对象的PreSigned URL。
![](/assets/img/MinIO_Browser1.jpg)

### MC命令行操作
另外，还可以通过MinIO自带的工具或者s3cmd来操作桶或对象。
```clike
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
mc config host add myminio http://192.168.56.11:9000   minioadmin minioadmin
```
mc命令行支持一些基本的操作
```clike
# mc -h
NAME:
  mc - MinIO Client for cloud storage and filesystems.

USAGE:
  mc [FLAGS] COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

COMMANDS:
  ls         list buckets and objects
  mb         make a bucket
  rb         remove a bucket
  cp         copy objects
  mirror     synchronize object(s) to a remote site
  cat        display object contents
  head       display first 'n' lines of an object
  pipe       stream STDIN to an object
  share      generate URL for temporary access to an object
  find       search for objects
  sql        run sql queries on objects
  stat       show object metadata
  tree       list buckets and objects in a tree format
  du         summarize disk usage recursively
  lock       set and get object lock configuration
  retention  set object retention for objects with a given prefix
  diff       list differences in object name, size, and date between two buckets
  rm         remove objects
  event      configure object notifications
  watch      listen for object notification events
  policy     manage anonymous access to buckets and objects
  admin      manage MinIO servers
  config     configure MinIO client
  update     update mc to latest release

GLOBAL FLAGS:
  --autocompletion              install auto-completion for your shell
  --config-dir value, -C value  path to configuration folder (default: "/root/.mc")
  --quiet, -q                   disable progress bar display
  --no-color                    disable color theme
  --json                        enable JSON formatted output
  --debug                       enable debug output
  --insecure                    disable SSL certificate verification
  --help, -h                    show help
  --version, -v                 print the version

TIP:
  Use 'mc --autocompletion' to enable shell autocompletion

VERSION:
  RELEASE.2020-03-14T01-23-37Z
```

比如列出桶，列出桶的对象，查看对象信息等等
```clike
# mc ls myminio
[2020-03-14 19:35:32 CST]      0B video/

# mc ls myminio/video
[2020-03-14 19:35:31 CST]   11MiB 2019-09-09-181105120.mp4
[2020-03-14 19:35:31 CST]   12MiB 2019-09-10-102655444.mp4
[2020-03-14 19:35:32 CST]   29MiB 2019-09-20-230421873.mp4

# mc stat myminio/video/2019-09-09-181105120.mp4
Name      : 2019-09-09-181105120.mp4
Date      : 2020-03-14 19:35:31 CST
Size      : 11 MiB
ETag      : 95f25d316bc08cabdad8a0180a5be867-1
Type      : file
Metadata  :
  Content-Type        : video/mp4
  X-Amz-Mp-Parts-Count: 1

```

### s3cmd命令行操作
```clike
# cat /root/.s3cfg
[default]
access_key = minioadmin
secret_key = minioadmin
host_base = 192.168.56.11:9000
host_bucket = 192.168.56.11:9000
signature_v2 = True
signurl_use_https = False
skip_existing = False
socket_timeout = 300
stats = False
stop_on_error = False
throttle_max = 100
urlencoding_mode = normal
use_http_expect = False
use_https = False
use_mime_magic = True
verbosity = WARNING

# s3cmd  ls
2020-03-14 11:35  s3://video

# s3cmd  ls s3://video
2020-03-14 11:35  11440620   s3://video/2019-09-09-181105120.mp4
2020-03-14 11:35  12826990   s3://video/2019-09-10-102655444.mp4
2020-03-14 11:35  30875180   s3://video/2019-09-20-230421873.mp4

# s3cmd  info s3://video/2019-09-09-181105120.mp4
s3://video/2019-09-09-181105120.mp4 (object):
   File size: 11440620
   Last mod:  Sat, 14 Mar 2020 11:35:31 GMT
   MIME type: video/mp4
   Storage:   STANDARD
   MD5 sum:   95f25d316bc08cabdad8a0180a5be867-1
   SSE:       none
   Policy:    none
```

### 纠删码（Erasure Coding）
官网介绍：
https://min.io/product/overview
https://docs.minio.io/docs/minio-erasure-code-quickstart-guide.html

MinIO以每个对象的粒度为数据提供保护，使用了Reed-Solomon算法将
对象拆分成条带，有N/2个数据对象和N/2个校验对象，当然，这些MiniIO
也可以配置成任意的冗余级别。假如配置了12块盘用来存储数据，则一个对象
被切分成6个数据块，并且计算出6个校验块，即使丢失5（N/2 - 1）个任意的数据块
或者校验块，都可以由剩下的数据重建原始数据。MinIO能保证在有多个设备丢失
或者不可用的情况下，对象依然可读或者写新对象。最后，MinIO的纠删码
是对象级别的，一次可以修复一个对象。
![](/assets/img/MiniIO-Erasure-Code.jpg)

###比特反转保护（Bitrot Protection）

###加密（Encryption）

###一写多读（WORM,Write Once Read Many）

###身份管理（Identity Management）

###持续复制（Continous Replication）

###全局联邦（Global Federation）

###多云网关（Multi-Cloud Gateway）
