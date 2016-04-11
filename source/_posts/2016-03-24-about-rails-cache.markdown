---
layout: post
title: "about rails cache"
date: 2016-03-24 10:05:39 +0800
comments: true
categories: 
---
在项目中经常会存储一些短时候内比较重要的数据，或者一些临时数据，这种数据以前一般我都把它
存到redis中，后来想想如果不是很重要的临时数据就没必要往redis里放了，直接用cache可能更方便

###ActiveSupport::Cache::Store
它是一个抽象类，有很多cache store的实现
rails中的这个类提供了和缓存交互的很多基本的方法，包括(read,write,delete,exist?,fetch)，下面说下具体实现


###ActiveSupport::Cache::FileStore
先说一下FileStore,这种存储方式Rails默认的存储方式，他是将数据存储到系统文件,默认将文件存储到项目的/tmp/cache下面，你可以修改它的存储路径

```ruby
config.cache_store = :file_store, "/path/to/cache/directory"
```
上面说FileStore是Rails默认的存储方式，看下面这句代码就知道了
```ruby
Rails.cache.class  => ActiveSupport::Cache::FileStore 
```
<!-- more -->

cache有几种常用的方法
```ruby
Rails.cache.write("name","peter")
Rails.cache.read("name")  => "peter"
Rails.cache.read("age")  => nil
```
cache中的key都是string型，而且区分大小写的，Hash和数据也或以当作key,上面的代码的key也可以用hash表示
```ruby
Rails.cache.read(:name)  => "peter"
Rails.cache.read(:name) == Rails.cache.read("name")  => true
```
如果你的cache是和其它程序公用的存储，你可以给他定义一个namespace，一旦namespace定义，就会在key前面加了个前缀
其它一些参数
:compress 是否压缩缓存，
:compress_threshold 和compress一起使用，设定一个阈值，低于这个值就不压缩缓存。默认为 16 KB。
:expires_in 为缓存记录设定一个过期时间，单位为秒，过期后把记录从缓存中删除。

###ActiveSupport::Cache::MemoryStore
这种存储方式是把数据存储在内存中，而且存储空间的大小由:size决定，默认为32M，如果超出分配的大小，系统会清理缓存，把最不常使用的记录删除。
```ruby
cache = ActiveSupport::Cache::MemoryStore.new
cache.write("name","peter")
cache.read("name")  => "peter"
```

###ActiveSupport::Cache::MemCacheStore
这种存储方式使用 Danga 开发的 memcached 服务器，为程序提供一个中心化的缓存存储。Rails 默认使用附带安装的 dalli gem 实现这种存储方式。这是目前在生产环境中使用最广泛的缓存存储方式，可以提供单个缓存存储，或者共享的缓存集群，性能高，冗余度低。

初始化时要指定集群中所有 memcached 服务器的地址。如果没有指定地址，默认运行在本地主机的默认端口上，这对大型网站来说不是个好主意。

在这种缓存存储中使用 write 和 fetch 方法还可指定两个额外的选项，充分利用 memcached 的特有功能。指定 :raw 选项可以直接把没有序列化的数据传给 memcached 服务器。在这种类型的数据上可以使用 memcached 的原生操作，例如 increment 和 decrement。如果不想让 memcached 覆盖已经存在的记录，可以指定 :unless_exist 选项。
上面的这种我也没用过，只是摘了别人一段话来补补。。。
