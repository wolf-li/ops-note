# Redis 忧患

## 内存管理
### 自动碎片整理defrag
当前 内存分配器查看 用 info memory，中 mem_allocator 项有内容
info memory 参数详解
```
used_memory:1107928   
used_memory_human:1.06M
used_memory_rss:10059776
used_memory_rss_human:9.59M
used_memory_peak:1528088
used_memory_peak_human:1.46M
used_memory_peak_perc:72.50%
used_memory_overhead:954056
used_memory_startup:810072
used_memory_dataset:153872
used_memory_dataset_perc:51.66%
allocator_allocated:1102200
allocator_active:1429504
allocator_resident:4849664
total_system_memory:16632262656
total_system_memory_human:15.49G
used_memory_lua:45056
used_memory_lua_human:44.00K
used_memory_scripts:216
used_memory_scripts_human:216B
number_of_cached_scripts:1
maxmemory:8589934592
maxmemory_human:8.00G
maxmemory_policy:noeviction
allocator_frag_ratio:1.30
allocator_frag_bytes:327304
allocator_rss_ratio:3.39
allocator_rss_bytes:3420160
rss_overhead_ratio:2.07
rss_overhead_bytes:5210112
mem_fragmentation_ratio:9.43
mem_fragmentation_bytes:8992864
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:143512
mem_aof_buffer:0
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0
lazyfreed_objects:0
```




https://juejin.cn/post/7031847977782083621/


https://blog.csdn.net/smartbetter/article/details/97971151


