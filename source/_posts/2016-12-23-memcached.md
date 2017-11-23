---
title: memcached
date: 2016-12-23 10:37:22
tags: cache
category: IT
---
# Ref
[Memcache 内存分配策略和性能(使用)状态检查](http://www.cnblogs.com/zhoujinyi/p/5554083.html)

# Note
nc命令相比telnet要更好
echo  "stats cachedump 29 0" | nc 10.211.55.9 11212 >/home/zhoujy/memcache.log
列出29号slab 里所有的key，value。 命令中的0 表示列出所有。