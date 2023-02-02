## Reids的特点

Redis本质上是一个Key-Value类型的内存数据库，很像memcached，整个数据库统统加载在内存当中进行操作，定期通过异步操作把数据库数据flush到硬盘上进行保存。

因为是纯内存操作，Redis的性能非常出色，每秒可以处理超过 10万次读写操作，是已知性能最快的Key-Value DB。

Redis的出色之处不仅仅是性能，Redis最大的魅力是支持保存多种数据结构，此外单个value的最大限制是1GB，不像 memcached只能保存1MB的数据，因此Redis可以用来实现很多有用的功能。

比方说用他的List来做FIFO双向链表，实现一个轻量级的高性 能消息队列服务，用他的Set可以做高性能的tag系统等等。另外Redis也可以对存入的Key-Value设置expire时间，因此也可以被当作一 个功能加强版的memcached来用。

Redis的主要缺点是数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此Redis适合的场景主要局限在较小数据量的高性能操作和运算上。

## 日志

mysql强一致性，要通过日志保证数据的正确性。主要通过redo和bin，两阶段提交日志。
redis后写是因为redis的日志不是强一致性的，redis的操作还要程序先判断是否正确，正确执行之后再写日志。而且内存操作很快，所以redis使用后写日志的策略。

## 限流器

比如一秒内进行某个操作50次，这种行为应该进行限制。

滑动窗口就是记录一个滑动的时间窗口内的操作次数，操作次数超过阈值则进行限流。

在redis中可以用zset数据结构来实现这个功能：
用唯一的id作为zset的key，可以是user_id + action_key ，value是当前操作的时间戳。每次新的操作请求进来时，先判断当前时间窗口内记录的操作次数 count，小于阈值max则允许进行操作，超过阈值则进行限流。同时对时间窗口之外的数据进行清理，节省内存。
简单代码实现：

```

def can_pass_slide_window(user,time_zone=60, times=30):
    """
    :param user: 用户唯一标识
    :param action: 用户访问的接口标识(即用户在客户端进行的动作)
    :param time_zone: 接口限制的时间段
    :param time_zone: 限制的时间段内允许多少请求通过
    """
    key = user
    now_ts = time.time() * 1000
    # value是什么在这里并不重要，只要保证value的唯一性即可，这里使用毫秒时间戳作为唯一值
    value = now_ts
    # 时间窗口左边界
    old_ts = now_ts - (time_zone * 1000)
    # 记录行为
    redis_conn.zadd(key,{value:now_ts})
    # 删除时间窗口之前的数据
    redis_conn.zremrangebyscore(key, 0, old_ts)
    # 获取窗口内的行为数量
    count = redis_conn.zcard(key)
    # 设置一个过期时间免得占空间
    redis_conn.expire(key, time_zone + 1)
    if not count or count < times:
        return True
    return False

can_pass_slide_window(3)

```

zset中的value没有特殊含义，只是用来保证每次操作都是唯一的能够被zset记录。

滑动窗口法，简单来说就是随着时间的推移，时间窗口也会持续移动，有一个计数器不断维护着窗口内的请求数量，这样就可以保证任意时间段内，都不会超过最大允许的请求数。例如当前时间窗口是0s~60s，请求数是40，10s后时间窗口就变成了10s~70s，请求数是60。

时间窗口的滑动和计数器可以使用redis的有序集合(sorted set)来实现。score的值用毫秒时间戳来表示，可以利用 当前时间戳 - 时间窗口的大小 来计算出窗口的边界，然后根据score的值做一个范围筛选就可以圈出一个窗口；value的值仅作为用户行为的唯一标识，也用毫秒时间戳就好。最后统计一下窗口内的请求数再做判断即可。

##使用redis有哪些好处？

1.速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)

2.支持丰富数据类型，支持string，list，set，sorted set，hash

3.支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行

4.丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除

##为什么redis需要把所有数据放到内存中?

Redis为了达到最快的读写速度将数据都读到内存中，并通过异步的方式将数据写入磁盘。所以redis具有快速和数据持久化的特征。如果不将数据放在内存中，磁盘I/O速度为严重影响redis的性能。在内存越来越便宜的今天，redis将会越来越受欢迎。

如果设置了最大使用的内存，则数据已有记录数达到内存限值后不能继续插入新值。

##Redis是单进程单线程的

redis利用队列技术将并发访问变为串行访问，消除了传统数据库串行控制的开销

##单线程的redis为什么这么快

回答:主要是以下三点

(一)纯内存操作

(二)单线程操作，避免了频繁的上下文切换

(三)采用了非阻塞I/O多路复用机制

## 怎么多路复用

Redis 会优先选择时间复杂度为 𝑂(1)
 的 I/O 多路复用函数作为底层实现，包括 Solaries 10 中的 evport、Linux 中的 epoll 和 macOS/FreeBSD 中的 kqueue，上述的这些函数都使用了内核内部的结构，并且能够服务几十万的文件描述符。
 
## 内存淘汰

1.1 定期删除 1.redis每过100ms，从设置了过期时间的key中随机取出20个缓存key. 2.清除其中的过期key. 3.若过期key占比超过1/4，则重复步骤1. ...
1.2 惰性删除 某个缓存key在查询时，若此时已过期，则会从缓存中删除，同时返回空 这种方式是被动触发的，

##redis持久化的几种方式

1、快照（snapshots）

缺省情况情况下，Redis把数据快照存放在磁盘上的二进制文件中，文件名为dump.rdb。你可以配置Redis的持久化策略，例如数据集中每N秒钟有超过M次更新，就将数据写入磁盘；或者你可以手工调用命令SAVE或BGSAVE。

工作原理

Redis forks.

子进程开始将数据写到临时RDB文件中。

当子进程完成写RDB文件，用新文件替换老文件。

这种方式可以使Redis使用copy-on-write技术。

2、AOF

快照模式并不十分健壮，当系统停止，或者无意中Redis被kill掉，最后写入Redis的数据就会丢失。

这对某些应用也许不是大问题，但对于要求高可靠性的应用来说，Redis就不是一个合适的选择。Append-only文件模式是另一种选择。你可以在配置文件中打开AOF模式

3、虚拟内存方式

当你的key很小而value很大时,使用VM的效果会比较好.因为这样节约的内存比较大.

当你的key不小时,可以考虑使用一些非常方法将很大的key变成很大的value,比如你可以考虑将key,value组合成一个新的value.

vm-max-threads这个参数,可以设置访问swap文件的线程数,设置最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的.可能会造成比较长时间的延迟,但是对数据完整性有很好的保证.

自己测试的时候发现用虚拟内存性能也不错。如果数据量很大，可以考虑分布式或者其他数据库。



##使用过Redis分布式锁么，

它是什么回事？先拿setnx来争抢锁，抢到之后，再用expire给锁加一个过期时间防止锁忘记了释放。这时候对方会告诉你说你回答得不错，然后接着问如果在setnx之后执行expire之前进程意外crash或者要重启维护了，那会怎么样？这时候你要给予惊讶的反馈：唉，是喔，这个锁就永远得不到释放了。紧接着你需要抓一抓自己得脑袋，故作思考片刻，好像接下来的结果是你主动思考出来的，然后回答：我记得set指令有非常复杂的参数，这个应该是可以同时把setnx和expire合成一条指令来用的！对方这时会显露笑容，心里开始默念：摁，这小子还不错。

## 分布式锁 生命周期

通过expire命令设置过期时间，防止死锁
EXPIRE key seconds:设置key的生存时间，当key过期时（生存时间为0），会被自动删除

## 设置锁 原子操作

通过2.6.8之后版本的reids的set命令实现，可以解决先SETNX再EXPIRE 非原子操作，可能造成死锁的问题

## 解锁需要判断客户端

//判断加锁与解锁是不是同一个客户端
if (requestId.equals(jedis.get(lockKey))) {
     //若在此时，这把锁突然不是这个客户端的，则会误解锁（所以使用lua脚本方式）
    jedis.del(lockKey);
}

简言之，就是谁加的锁，谁释放，别人不能释放

## 假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？
使用keys指令可以扫出指定模式的key列表。对方接着追问：如果这个redis正在给线上的业务提供服务，那使用keys指令会有什么问题？这个时候你要回答redis关键的一个特性：redis的单线程的。keys指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用scan指令，scan指令可以无阻塞的提取出指定模式的key列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用keys指令长。

##使用过Redis做异步队列么，你是怎么用的？

一般使用list结构作为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会再重试。如果对方追问可不可以不用sleep呢？list还有个指令叫blpop，在没有消息的时候，它会阻塞住直到消息到来。 如果对方追问能不能生产一次消费多次呢？使用pub/sub主题订阅者模式，可以实现1:N的消息队列。如果对方追问pub/sub有什么缺点？在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如rabbitmq等。如果对方追问redis如何实现延时队列？我估计现在你很想把面试官一棒打死如果你手上有一根棒球棍的话，怎么问的这么详细。但是你很克制，然后神态自若的回答道：使用sortedset，拿时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用zrangebyscore指令获取N秒之前的数据轮询进行处理。到这里，面试官暗地里已经对你竖起了大拇指。但是他不知道的是此刻你却竖起了中指，在椅子背后。

##如果有大量的key需要设置同一时间过期，一般需要注意什么？

如果大量的key过期时间设置的过于集中，到过期的那个时间点，redis可能会出现短暂的卡顿现象。一般需要在时间上加一个随机值，使得过期时间分散一些。

##Pipeline有什么好处，为什么要用pipeline？

可以将多次IO往返的时间缩减为一次，前提是pipeline执行的指令之间没有因果相关性。使用redis-benchmark进行压测的时候可以发现影响redis的QPS峰值的一个重要因素是pipeline批次指令的数目。


##Redis的同步机制了解么？

Redis可以使用主从同步，从从同步。第一次同步时，主节点做一次bgsave，并同时将后续修改操作记录到内存buffer，待完成后将rdb文件全量同步到复制节点，复制节点接受完成后将rdb镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。

##是否使用过Redis集群，集群的原理是什么？ 

Redis Sentinal着眼于高可用，在master宕机时会自动将slave提升为master，继续提供服务。Redis Cluster着眼于扩展性，在单个redis内存不足时，使用Cluster进行分片存储。



##什么是Redis的并发竞争问题


Redis的并发竞争问题，主要是发生在并发写竞争。

考虑到redis没有像db中的sql语句，update val = val + 10 where ...，无法使用这种方式进行对数据的更新。

假如有某个key = "price"，  value值为10，现在想把value值进行+10操作。正常逻辑下，就是先把数据key为price的值读回来，加上10，再把值给设置回去。如果只有一个连接的情况下，这种方式没有问题，可以工作得很好，但如果有两个连接时，两个连接同时想对还price进行+10操作，就可能会出现问题了。

例如：两个连接同时对price进行写操作，同时加10，最终结果我们知道，应该为30才是正确。

考虑到一种情况：

T1时刻，连接1将price读出，目标设置的数据为10+10 = 20。

T2时刻，连接2也将数据读出，也是为10，目标设置为20。

T3时刻，连接1将price设置为20。

T4时刻，连接2也将price设置为20，则最终结果是一个错误值20。


###解决方案

使用乐观锁的方式进行解决（成本较低，非阻塞，性能较高）

如何用乐观锁方式进行解决？

本质上是假设不会进行冲突，使用redis的命令watch进行构造条件。伪代码如下：

```
watch price

get price $price

$price = $price + 10

multi

set price $price

exec
```

watch这里表示监控该key值，后面的事务是有条件的执行，如果从watch的exec语句执行时，watch的key对应的value值被修改了，则事务不会执行。

方案2
这个是针对客户端来的，在代码里要对redis操作的时候，针对同一key的资源，就先进行加锁（java里的synchronized或lock）。

方案3
利用redis的setnx实现内置的锁。


##redis和memcached的区别（总结）

1、Redis和Memcache都是将数据存放在内存中，都是内存数据库。不过memcache还可用于缓存其他东西，例如图片、视频等等；

2、Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，hash等数据结构的存储；

3、虚拟内存--Redis当物理内存用完时，可以将一些很久没用到的value 交换到磁盘；

4、过期策略--memcache在set时就指定，例如set key1 0 0 8,即永不过期。Redis可以通过例如expire 设定，例如expire name 10；

5、分布式--设定memcache集群，利用magent做一主多从;redis可以做一主多从。都可以一主一从；

6、存储数据安全--memcache挂掉后，数据没了；redis可以定期保存到磁盘（持久化）；

7、灾难恢复--memcache挂掉后，数据不可恢复; redis数据丢失后可以通过aof恢复；

8、Redis支持数据的备份，即master-slave模式的数据备份；

应用场景

redis：数据量较小的更性能操作和运算上

memcache：用于在动态系统中减少数据库负载，提升性能;做缓存，提高性能（适合读多写少，对于数据量比较大，可以采用sharding）

MongoDB:主要解决海量数据的访问效率问题  


## 缓存穿透

当用户访问的数据，既不在缓存中，也不在数据库中，导致请求在访问缓存时，发现缓存缺失，再去访问数据库时，发现数据库中也没有要访问的数据，没办法构建缓存数据，来服务后续的请求。那么当有大量这样的请求到来时，数据库的压力骤增，这就是缓存穿透的问题。

缓存穿透的发生一般有这两种情况：

业务误操作，缓存中的数据和数据库中的数据都被误删除了，所以导致缓存和数据库中都没有数据；
黑客恶意攻击，故意大量访问某些读取不存在数据的业务；
应对缓存穿透的方案，常见的方案有两种。

第一种方案，非法请求的限制；
第二种方案，设置缓存空值或者默认值；
第三种方案，布隆过滤器

## 缓存击穿

缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力

解决方案：

设置数据永远不过期。（只针对热数据）

## 缓存雪崩

缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
设置数据永远不过期。

## 布隆过滤器

布隆过滤器可以检查值是 “可能在集合中” 还是 “绝对不在集合中”。“可能” 表示有一定的概率，也就是说可能存在一定为误判率。

当我们搜索一个值的时候，若该值经过 K 个哈希函数运算后的任何一个索引位为 ”0“，那么该值肯定不在集合中。但如果所有哈希索引值均为 ”1“，则只能说该搜索的值可能存在集合中。

在实际工作中，布隆过滤器常见的应用场景如下：

网页爬虫对 URL 去重，避免爬取相同的 URL 地址；
反垃圾邮件，从数十亿个垃圾邮件列表中判断某邮箱是否垃圾邮箱；
Google Chrome 使用布隆过滤器识别恶意 URL；
Medium 使用布隆过滤器避免推荐给用户已经读过的文章；
Google BigTable，Apache HBbase 和 Apache Cassandra 使用布隆过滤器减少对不存在的行和列的查找。
除了上述的应用场景之外，布隆过滤器还有一个应用场景就是解决缓存穿透的问题。所谓的缓存穿透就是服务调用方每次都是查询不在缓存中的数据，这样每次服务调用都会到数据库中进行查询，如果这类请求比较多的话，就会导致数据库压力增大，这样缓存就失去了意义。

利用布隆过滤器我们可以预先把数据查询的主键，比如用户 ID 或文章 ID 缓存到过滤器中。当根据 ID 进行数据查询的时候，我们先判断该 ID 是否存在，若存在的话，则进行下一步处理。若不存在的话，直接返回，这样就不会触发后续的数据库查询。需要注意的是缓存穿透不能完全解决，我们只能将其控制在一个可以容忍的范围内。

```
from bitarray import bitarray

import mmh3

class BloomFilter(set):
    def __init__(self,size,hash_count):
        #size:the num of the bitarray
        #hash_count:the num of hash function
        super(BloomFilter,self).__init__()
        self.bit_array = bitarray(size)
        self.bit_array.setall(0) #初始化为0
        self.size = size
        self.hash_count = hash_count

        def __len__(self):
            return self.size

        def __iter__(self):
            return iter(self.bit_array)

        def add(self,item):
            for i in range(self.hash_count):
                index = mmh3.hash(item,i) % self.size
                self.bit_array[index] = 1
            return self

        def __contains__(self,item):
            out = True
            for i in range(self.hash_count):
                index = mmh3.hash(item,i)%self.size
                if bit_array[index] == 0:
                    out = False
                    
            return out

def main():
    bloom = BloomFilter(100,5)
    fd = open("urls.txt")  #有重复的网址 
    bloomfilter = BloomFilter(100,10)    
    while True:    
        url = fd.readline().strip()   
        if (url == 'exit') :  
            print ('complete and exit now')
            break    
        elif url not in bloomfilter:   
            bloomfilter.add(url)
            # print(url)    
        else:    
            print ('url :%s has exist' % url )

if __name__ == '__main__':
        main()
```



