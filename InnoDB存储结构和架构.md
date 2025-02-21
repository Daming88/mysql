### InnoDB逻辑存储结构
* 表空间(idb文件): 一个mysql实例可以对应多个表空间，用于存储记录、索引等数据。  
* 段，分为数据段(Leaf node segment)、索引段(Non-leaf node segment)、回滚段(Rollback segment)，  
InnoDB是索引组织表，数据段就是B+树的叶子节点，索引段即为B+树的非叶子节点。段用来管理多个Extent(区)。
* 区，表空间的单元结构，每个区的大小为1M。默认情况下，InnoDB存储引擎页大小为16K，即一个区中共有64个连续的页。
* 页，是InnoDB存储引擎磁盘管理的最小单元，每页的大小默认为16KB。为了保证页的连续性，InnoDB存储引擎每次从磁盘申请4~5个区。
* 行，InnoDB存储引擎数据是按行进行存放的。
    - Trs_id: 每次对某条记录进行改动时，都会把对应的事务id赋值给trx_id隐藏列。
    - Roll_pointer: 每次对某条记录进行改动时，都会把旧的版本写入到undo日志，然后隐藏列就相当于一个指针，可以通过他来找到该记录修改前的信息。
### 架构  
mysql5.5版本开始，默认使用InnoDB存储引擎，他擅长处理事务，具有崩溃恢复特性，在日常开发环境中使用非常广泛。
> 1、内存结构图  
![内存结构图](image/img_13.png)  
Buffer Pool: <font color="purple">缓冲池，是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据。在执行增删改查操作时，先操作缓冲池的数据(若缓冲池没有数据，则从磁盘加载并缓存)，然后再以一定频率刷新到磁盘，从而减少磁盘IO，加快处理速度。</font>
>> <font color="">缓冲池以Page页为单位，底层采用链表数据结构管理Page。根据状态，将Page分为三种类型</font>  
free page: <font color="purple">空闲页，没有被使用。</font>  
clean page:<font color="purple">被使用page,数据没有被修改过。</font>  
dirty page:<font color="purple">脏页，被使用page，数据被修改过，页中数据与从磁盘数据产生了不一致。</font>
