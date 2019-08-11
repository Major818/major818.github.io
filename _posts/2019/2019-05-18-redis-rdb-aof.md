---
layout: post
title: Redis持久化方式
category: database
tags: [database]
keywords: redis、RDB、AOF、持久化
---

## 什么是Redis持久化
Redis 作为一个键值对内存数据库(NoSQL)，数据都存储在内存当中，在处理客户端请求时，所有操作都在内存当中进行，如下所示：

![](http://www.major818.com/assets/images/2019/database/redis数据存储在内存中.png)

这样做有什么问题呢?其实，只要稍微有点计算机基础知识的人都知道，存储在内存当中的数据，只要服务器关机(各种原因引起的)，内存中的数据就会消失了。

不仅服务器关机会造成数据消失，Redis 服务器守护进程退出，内存中的数据也一样会消失。

![](http://www.major818.com/assets/images/2019/database/redis内存数据丢失.png)

对于只把 Redis 当缓存来用的项目来说，数据消失或许问题不大，重新从数据源把数据加载进来就可以了。

但如果直接把用户提交的业务数据存储在 Redis 当中，把 Redis 作为数据库来使用，在其放存储重要业务数据，那么 Redis 的内存数据丢失所造成的影响也许是毁灭性。

为了避免内存中数据丢失，Redis 提供了对持久化的支持，我们可以选择不同的方式将数据从内存中保存到硬盘当中，使数据可以持久化保存。

![](http://www.major818.com/assets/images/2019/database/redis将数据持久化到硬盘.png)

Redis 提供了 RDB 和 AOF 两种不同的数据持久化方式，下面我们就来详细介绍一下这种不同的持久化方式吧。

## RDB

RDB 是一种快照存储持久化方式，具体就是将 Redis 某一时刻的内存数据保存到硬盘的文件当中，默认保存的文件名为 dump.rdb，而在 Redis 服务器启动时，会重新加载 dump.rdb 文件的数据到内存当中恢复数据。

**1. 开启 RDB 持久化方式**

开启 RDB 持久化方式很简单，客户端可以通过向 Redis 服务器发送 Save 或 Bgsave 命令让服务器生成 RDB 文件，或者通过服务器配置文件指定触发 RDB 条件。

save 命令：是一个同步操作。


      # 同步数据到磁盘上 
       >save 

![](http://www.major818.com/assets/images/2019/database/redis-save.png)

当客户端向服务器发送 Save 命令请求进行持久化时，服务器会阻塞 Save 命令之后的其他客户端的请求，直到数据同步完成。

如果数据量太大，同步数据会执行很久，而这期间 Redis 服务器也无法接收其他请求，所以，最好不要在生产环境使用 Save 命令。

Bgsave：与 Save 命令不同，Bgsave 命令是一个异步操作。

      # 异步保存数据集到磁盘上 
      > bgsave 

![](http://www.major818.com/assets/images/2019/database/redis-bgsave.png)

当客户端发服务发出 Bgsave 命令时，Redis 服务器主进程会 Forks 一个子进程来数据同步问题，在将数据保存到 RDB 文件之后，子进程会退出。

所以，与 Save 命令相比，Redis 服务器在处理 Bgsave 采用子线程进行 IO 写入。

而主进程仍然可以接收其他请求，但 Forks 子进程是同步的，所以 Forks 子进程时，一样不能接收其他请求。

这意味着，如果 Forks 一个子进程花费的时间太久(一般是很快的)，Bgsave 命令仍然有阻塞其他客户的请求的情况发生。

服务器配置自动触发：除了通过客户端发送命令外，还有一种方式，就是在 Redis 配置文件中的 Save 指定到达触发 RDB 持久化的条件，比如【多少秒内至少达到多少写操作】就开启 RDB 数据同步。

例如我们可以在配置文件 redis.conf 指定如下的选项：

      # 900s内至少达到一条写命令 
      save 900 1 
      # 300s内至少达至10条写命令 
      save 300 10 
      # 60s内至少达到10000条写命令 
      save 60 10000 

之后在启动服务器时加载配置文件。

      # 启动服务器加载配置文件 
      redis-server redis.conf 

这种通过服务器配置文件触发 RDB 的方式，与 Bgsave 命令类似，达到触发条件时，会 Forks 一个子进程进行数据同步。

不过最好不要通过这方式来触发 RDB 持久化，因为设置触发的时间太短，则容易频繁写入 RDB 文件，影响服务器性能，时间设置太长则会造成数据丢失。

**2. RDB 文件**

前面介绍了三种让服务器生成 RDB 文件的方式，无论是由主进程生成还是子进程来生成，其过程如下：

+ 生成临时 RDB 文件，并写入数据。
+ 完成数据写入，用临时文代替代正式 RDB 文件。
+ 删除原来的 DB 文件。

RDB 默认生成的文件名为 dump.rdb，当然，我可以通过配置文件进行更加详细配置。

比如在单机下启动多个 Redis 服务器进程时，可以通过端口号配置不同的 RDB 名称，如下所示：

      # 是否压缩rdb文件 
	  rdbcompression yes 
 
	  # rdb文件的名称 
      dbfilename redis-6379.rdb 
 
      # rdb文件保存目录 
      dir ~/redis/ 

**RDB的几个优点：**

+ 与 AOF 方式相比，通过 RDB 文件恢复数据比较快。
+ RDB 文件非常紧凑，适合于数据备份。
+ 通过 RDB 进行数据备份，由于使用子进程生成，所以对 Redis 服务器性能影响较小。


**RDB 的几个缺点：**

+ 如果服务器宕机的话，采用 RDB 的方式会造成某个时段内数据的丢失，比如我们设置 10 分钟同步一次或 5 分钟达到 1000 次写入就同步一次，那么如果还没达到触发条件服务器就死机了，那么这个时间段的数据会丢失。
+ 使用 Save 命令会造成服务器阻塞，直接数据同步完成才能接收后续请求。
+ 使用 Bgsave 命令在 Forks 子进程时，如果数据量太大，Forks 的过程也会发生阻塞，另外，Forks 子进程会耗费内存。

## AOF

聊完了 RDB，来聊聊 Redis 的另外一个持久化方式：AOF(Append-only file)。

与 RDB 存储某个时刻的快照不同，AOF 持久化方式会记录客户端对服务器的每一次写操作命令，并将这些写操作以 Redis 协议追加保存到以后缀为 AOF 文件末尾。

在 Redis 服务器重启时，会加载并运行 AOF 文件的命令，以达到恢复数据的目的。

![](http://www.major818.com/assets/images/2019/database/redis-aof.png)

**1.开启 AOF 持久化方式**

Redis 默认不开启 AOF 持久化方式，我们可以在配置文件中开启并进行更加详细的配置，如下面的 redis.conf 文件：

      # 开启aof机制 
      appendonly yes 
 
      # aof文件名 
      appendfilename "appendonly.aof" 
 
      # 写入策略,always表示每个写操作都保存到aof文件中,也可以是everysec或no 
      appendfsync always 
 
     # 默认不重写aof文件 
	 no-appendfsync-on-rewrite no 
 
	 # 保存目录 
	 dir ~/redis/ 

**2.三种写入策略**

在上面的配置文件中，我们可以通过 appendfsync 选项指定写入策略，有三个选项：

      appendfsync always 
	  # appendfsync everysec 
      # appendfsync no 

always：客户端的每一个写操作都保存到 AOF 文件当中，这种策略很安全，但是每个写操作都有 IO 操作，所以也很慢。

everysec：appendfsync 的默认写入策略，每秒写入一次 AOF 文件，因此，最多可能会丢失 1s 的数据。

no：Redis 服务器不负责写入 AOF，而是交由操作系统来处理什么时候写入 AOF 文件。更快，但也是最不安全的选择，不推荐使用。

**3.AOF 文件重写**

AOF 将客户端的每一个写操作都追加到 AOF 文件末尾，比如对一个 Key 多次执行 Incr 命令，这时候，AOF 保存每一次命令到 AOF 文件中，AOF 文件会变得非常大。

      incr num 1 
	  incr num 2 
	  incr num 3 
	  incr num 4 
	  incr num 5 
	  incr num 6 
	  ... 
	  incr num 100000 

AOF 文件太大，加载 AOF 文件恢复数据时，就会非常慢，为了解决这个问题，Redis 支持 AOF 文件重写。

通过重写 AOF，可以生成一个恢复当前数据的最少命令集，比如上面的例子中那么多条命令，可以重写为：

      set num 100000 

AOF 文件是一个二进制文件，并不是像上面的例子一样，直接保存每个命令，而使用 Redis 自己的格式，上面只是方便演示。

两种重写方式：通过在 redis.conf 配置文件中的选项 no-appendfsync-on-rewrite 可以设置是否开启重写。

这种方式会在每次 Fsync 时都重写，影响服务器性能，因此默认值为 no，不推荐使用。

      # 默认不重写aof文件 
	  no-appendfsync-on-rewrite no 

客户端向服务器发送 bgrewriteaof 命令，也可以让服务器进行 AOF 重写。

	  # 让服务器异步重写追加aof文件命令 
	  > bgrewriteaof 
	
AOF 重写方式也是异步操作，即如果要写入 AOF 文件，则 Redis 主进程会 Forks 一个子进程来处理，如下所示：

![](http://www.major818.com/assets/images/2019/database/redis-aof重写.png)

**重写 AOF 文件的好处：**

+ 压缩 AOF 文件，减少磁盘占用量。
+ 将 AOF 的命令压缩为最小命令集，加快了数据恢复的速度。

**3.AOF 文件损坏**

在写入 AOF 日志文件时，如果 Redis 服务器宕机，则 AOF 日志文件文件会出格式错误。

在重启 Redis 服务器时，Redis 服务器会拒绝载入这个 AOF 文件，可以通过以下步骤修复 AOF 并恢复数据：

+ 备份现在 AOF 文件，以防万一。
+ 使用 redis-check-aof 命令修复 AOF 文件，该命令格式如下：

      	# 修复aof日志文件 
		$ redis-check-aof -fix file.aof 

+ 重启 Redis 服务器，加载已经修复的 AOF 文件，恢复数据。

**AOF 的优点：**

+ AOF 只是追加日志文件，因此对服务器性能影响较小，速度比 RDB 要快，消耗的内存较少。

**AOF 的缺点：**

+ AOF 方式生成的日志文件太大，即使通过 AFO 重写，文件体积仍然很大。
+ 恢复数据的速度比 RDB 慢。 

**选择 RDB 还是 AOF 呢?**

通过上面的介绍，我们了解了 RDB 与 AOF 各自的优点与缺点，到底要如何选择呢?

通过下面的表示，我们可以从几个方面对比一下 RDB 与 AOF，在应用时，要根据自己的实际需求，选择 RDB 或者 AOF。

其实，如果想要数据足够安全，可以两种方式都开启，但两种持久化方式同时进行 IO 操作，会严重影响服务器性能，因此有时候不得不做出选择。

|方式|RDB|AOF|
|:------------:|:------------:|:------------:|
|启动优化及|低|高|
|体积|小|大|
|恢复速度|快|慢|
|数据安全性|会丢数据|由策略决定|
|轻重|重|轻|

当 RDB 与 AOF 两种方式都开启时，Redis 会优先使用 AOF 日志来恢复数据，因为 AOF 保存的文件比 RDB 文件更完整。

小结：上面讲了一大堆 Redis 的持久化机制的知识，其实，如果你只是单纯把 Redis 作为缓存服务器，那么可以完全不用考虑持久化。

但是，在如今的大多数服务器架构中，Redis 不单单只是扮演一个缓存服务器的角色，还可以作为数据库，保存我们的业务数据，此时，我们则需要好好了解有关 Redis 持久化策略的区别与选择。




