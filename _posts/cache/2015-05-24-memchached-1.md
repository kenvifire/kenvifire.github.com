---
layout: post
title: "memcached协议"
description: "简单讲解memcached的协议"
category: 
tags: [cache,memcached]
---
{% include JB/setup %}

## 概述

Memcached是一个开源的，高性能的分布式对象缓存系统。它是一个基于内存的key-value存储。

## 功能

Memchached支持的功能有：

- 保存数据
- 读取数据
- 删除数据
- 增减操作
- 修改失效时间
- Slab重分配
- Slab自动移动
- 设置LRU_Crawler
- 统计数据

## 协议

Memchached支持TCP和UDP协议进行通信，但是应用层协议都是一致的，下面来看看各功能在协议中是如何得到支持的。

### 保存数据

首先，客户端发送一个如下格式的命令：

{% highlight java %}
<command name> <key> <flags> <exptime> <bytes> [noreply]\r\n
cas <key> <flags> <exptime> <bytes> <cas unique> [noreply]\n\n
{% endhighlight %}

- `<command>` 主要包含：set，replace，append和prepend

	- set 表示保存数据
	- add表示保存数据，但是只是在服务端没有对应key的情况下
	- replace表示保存数据，但是只是在服务端已有该key的情况下
	- append表示在已有数据的后面添加数据
	- prepend表示在已有数据的前面添加数据

 prepend和append命令不会有flags和exptime字段

- `<key>` 表示客户端想要保存数据所对应的key

- `<flags>` 是一个任意的16位整数，memcached服务器会把它和数据保存在一起，读取的时候也会返回给客户端，这个值对于服务端而言是透明的。

- `<exptime>` 表示失效时间，0表示永不失效（但是可能会因为server需要给其他数据腾出空间而被删除）

- `<bytes>` 表示数据的长度

- `<cas unique>` 表示客户端用来进行cas更新时从服务端取到的关联到当前取到值一个64位的值。

- “noreplay”是个可选的参数，它表示不需要server返回信息。

在这行结束后，客户端就可以把数据发送给服务端：

{% highlight java %}
<data block>\r\n
{% endhighlight %}

- `<data block>`是一个由任意`<byte>`长度的8位数据块。

发送完命令后，客服端就等待服务端的返回结果，结果可能是：

- “STORED\r\n”，表示成功
- “NOT_STORED\r\n”表示数据没有保存，不过不是因为错误。一般而言，是因为“add”和“replace”命令的条件没达到
- “EXISTS\r\n”表示通过cas要保存的数据项在上次获取之后被修改过
- “NOT_FOUND\r\n“表示通过cas命令要保存的数据项不存在

### 读取数据

获取数据的命令“get”和“gets”命令的形式如下：

{% highlight java %}
get <key>*\r\n
gets <key>*\r\n
{% endhighlight %}

- `<key>*` 表示通过空格区分的多个key

发送完这条命令后，客户端会等待服务器返回0条或者多条数据项。每条数据项都是一个文本行，后面是数据块。在最后面是字符串：

“END\r\n”

用来表示相应结束。

每个服务端返回的数据项都如下所示：

{% highlight java %}
VALUE <key> <flags> <bytes> [<cas unique>]\r\n
<data block>\r\n
{% endhighlight %}

- <key> 对应的是数据项的key

- <flags> 是存储命令设置的flags值

- <bytes> 是数据块的长度，不包含`\r\n`

- <cas unique> 是一个用来唯一表示指定数据项的一个64位整数

- <data block>表示当前数据项对应的数据

### 删除数据

`delete`命令让我们可以显示地去删除数据：

{% highlight java %}
delete <key> [noreply]\r\n
{% endhighlight %}

- `<key>`是客户端想要删除的数据项的key

- “noreply”是可选的参数，旨在告诉服务器不用发送相应信息。

对于这个命令的相应有：

- “DELETED\r\n”表示删除成功
- “NOT_FOUND\r\n“表示和这个key对应的数据项不存在

### 增加/减少

`incr`和`decr`命令都是用来立即修改数据的，分表表示增加和减少。对应数据项的值会被按照64位的无符号整数进行处理。如果当前的数据值不能当做数字进行处理的话，服务器会返回错误。

客户端的命令如下：

{% highlight java %}

incr <key> <value> [noreply]\r\n
decr <key> <value> [noreply]\r\n

{% endhighlight %}

- `<value>`表示想要增加或者减少的值。

服务端的相应有：

- “NOT_FOUND\r\n” 表示更当前key对应的值不存在

- “<value>\r\n” 表示修改后的新值

### Touch命令

`touch`命令主要是用来更新已有数据项的时效时间。

{% highlight java %}
touch <key> <exptime> [noreply]\r\n
{% endhighlight %}

- `<key>`表示需要更新的数据项的key

- `<exptime>`表示失效时间

服务端的相应有：

- “TOUCHED\r\n” 表示成功
- “NOT_FOUND\r\n”表示该key对应的数据项不存在








