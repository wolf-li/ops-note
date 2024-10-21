## 有增量数据，恢复数据
先使用全量恢复所有数据
在使用增量恢复全部数据

查看 binlog 文件查找事故相关时间点前的操作,确认 at 位置恢复
```bash
mysqlbinlog binlog.00001

mysqlbinlog --start-postition=4 --stop-postition=704 binlog.00001 | mysql -p
```