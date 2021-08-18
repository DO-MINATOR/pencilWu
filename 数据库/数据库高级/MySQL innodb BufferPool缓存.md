![sda](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/99001)

1. server层在连接器、分析器、优化器后就开始真正执行sql语句，这里以innodb引擎为主。不同引擎底层执行机理不同。
2. 将磁盘中数据所在的那一页load入内存
3. 更新内存中的数据，记录undo日志，用于配合read view实现mvcc。
4. 记录redo日志，该日志保存的是sql逻辑语句，可用于数据恢复。
5. 记录binlog日志，属于server层公共的日志记录行为，可用于数据恢复和主从备份。

通过buffer pool缓存机制，使得MySQL的并发量可以支撑k/s，此时性能瓶颈在redo日志的记录上，但由于是顺序IO读写，因此比随机的idx数据页存储快很多。MySQL的IO线程会不定期将buffer pool中的数据刷新到磁盘中，如果此时宕机，可以通过redo日志来恢复内存中的数据。
