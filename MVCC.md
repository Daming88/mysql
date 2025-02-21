### MVCC　多版本并发控制
* 当前读:  
读取的时记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如: select...lock share mode(共享锁)等都是一种当前读。
* 快照读:  
简单的select(不加锁)就是快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。  
    - read committed(读已提交)：每次select，都生成一个快照读。  
    - repeatable read(可重复读): 开启事务后第一个select语句才是快照读的地方。
    - Serializable(串行化): 快照读会退化为当前读。
* MVCC  
全称Multi-Version Concurrency Control,多版本并发控制。指维护一个数据的多个版本，使得读写操作没有冲突。快照读为mysql实现Mvcc提供了一个非阻塞读功能。MVCC的具体实现，还需要依赖于数据库记录中的三个隐式字段、undo 日志、readView。
* 