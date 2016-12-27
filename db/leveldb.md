## 主要结构

- Slice   存放一个外部data的指针+size,  用于临时的二进制流操作比较方便. 

- WriteBatch  批量插入
     http://luodw.cc/2015/10/30/leveldb-14/

- SequenceNumber  是个一条记录的自增序列号, 每次Put会自增1
     对于snapshot来说，这个实际上是个序列号，用来判断同一个key插入的先后

### db/filename 相关的文件名管理

     log binlog文件
     ldb  Table文件
     sst   SSTable文件
     MANIFEST-xx 文件
     CURRENT   当前文件
     LOCK   锁文件
     LOG     infolog文件
     dbtmp   临时文件


### SkipList 跳表
     按概率来的，leveldb是1/4的概率height++, 就是预期上一层是下一层节点数的1/4, 最多12层
     查找复杂度实际上是 O(logN)


### Skiplist的Insert操作多线程原子性
          http://duanple.blog.163.com/blog/static/70971767201242491334104/
          http://luodw.cc/2015/10/16/leveldb-05/
我们来分析下如果在Insert中的prev[i]->SetNext(i, x);不使用Barrier会有什么问题？如下两条语句：
x->NoBarrier_SetNext(i, prev[i]->NoBarrier_Next(i));
prev[i]->SetNext(i, x);
实际上等效于如下三句：
x->next=prev[i]->next
MemoryBarrier()
prev[i]->next=x

因为读是不加锁的，所以需要内存屏障保证不能乱序
写的话必定需要加锁，这里是否用内存屏障无所谓
假设如果没有内存Barrier，可能会出现如下一种情况，另一个并发线程，可能会看到不正确的内存更新顺序，比如它可能看到prev[i]->next已经是x了，但是x的next还未被正确更新，因为x有可能是存在高速缓存中的，还未刷到内存，这样就会有问题。通过Barrier就可以保证，如果用户看到了prev[i]->next=x那么x->next也肯定已经被正确的设置了。

### Memtable
     内部有个Table table_ 变量(实际上是typedef SkipList<const char*, KeyComparator>Table)
     所以内存部分，实际上是个SkipList (注意这个Table与sstable中的table不是一回事，这个纯粹是个skiplist)

### Put操作
     1. 先AddRecord插入到log (log_writter.cc)
     2. 如果log插入ok, 插入到当前memtable

     WriteLevel0Table()  level 0是最新的一层sstable
     BuildTable() 从memtable写入到sstable 

### Get操作

     现在Memtable及imm(只读memtable)里面Seek, 如果找不到，再找sstable
     Version::Get()
              每个sstable的metadata有个key的范围，找到所在范围的sstable (对于level0, 可能有多个文件)
              之后调用TableCache::Get()
                    FindTable() 读取对应的文件信息, 插入到cache中
                    Table::InternalGet  (table.cc)
                         根据index信息，调用index_block的seek, 找到对应的Data Block. 并读出data block
                         注意：Seek()是二分查找找到 >= 的数据，具体key要严格相等，还要做一次顺序Compare, 这个不同类型block做法不一样。对于index, 只需要找到落入范围的就可以下一步Read DataBlock了。
                         在data block中通过block::Seek()， 二分查找来找restart_point(相同前缀), 然后在一段内顺序compare找到key


### 代码组织和规范
- 单元测试写得很齐全  
     XX_test.cc  为对应模块的单元测试, 放在同级目录下
- 大部分工具类都禁用了copy constructor和赋值
- LRU cache的释放，可以传入一个deleter来决定remove的时候是否要释放value. 
   virtual Handle* Insert(const Slice& key, void* value, size_t charge,  void (*deleter)(const Slice& key, void* value)) = 0;
- 用Logger对象抽象log,  Logger对象保存到需要的类对象中. 
- Cache的handle(是个空结构体) 用了指针转换了实现不同的implementation. (handle转成LRUHandle). 
- ShardedLRUCache实际上是个二级hash，第一级把低位做多个hash table的sharding. 第二级尽可能保持链表长度为1, 如果满了就rehash
- 有个Iterator基类，用于数据查找。每个类型的私有成员都有一个继承Iterator的iterator实现类

db/db_bench.cc
     benchmark用了多线程，Wait(), SignalAll()同步各个线程的状态，让大家等待到某个状态

### C++用法
   
  善用std::string的append处理文本的格式化
     用类模板函数适配类型，用宏定义来批量生成模板函数(见testharness.h 生成各种ASSERT_XX)
     利用临时对象析构函数做一些事情 (参见~Tester()). 这样就把 if（返回值）这种丑陋的逻辑隐藏起来 (临时Mutex锁也是类似用法，析构时候解锁)
     工具类用类static函数产生对象. (参见Status)

### 算法学习
     一种新型的快速hash算法： murmur hash
     手撸的随机算法： util/random.h

## 源码文件速查
     util/testharness     单元测试框架
               用TEST()宏定义一个类，并定义一个实例，定义_Run()函数，然后注册到全局vector
     util/Options类  定义各种配置参数, 里面有一个Env成员
     util/env    系统File操作和Logger封装
     util/coding   用std::string来做序列化/反序列化(push, pop二进制数据)
     util/cache      一个LRUcache实现

## 参考连接
[leveldb日知录](http://www.cnblogs.com/haippy/archive/2011/12/04/2276064.html)

[LevelDB框架解析](http://duanple.blog.163.com/blog/static/70971767201242491334104/)

