# 悲观锁
概念：对于并发数据，悲观锁使用数据的时候认为一定会有其他线程来修改数据，在修改前会先给数据添加锁，然后再进行数据修改。

优点：很好保证数据的同步性。

缺点：资源消耗较多，每次都会进行加解锁操作。

使用场景：适用于写较多场景。

# 乐观锁
概念：对于并发数据，乐观锁任务在自己修改数据的时候，不会有其他线程对数据进行修改。不加锁直接对数据进行更新，进行写入时再进行判断选择更新策略。

优点：通过相关算法（CAS）进行值更新，资源消耗少，性能较好。

缺点：数据同步可能存在问题。

使用场景：适用于读较多场景。

# 自旋锁
概念：当被加锁操作代码消耗时间较短时，这时多线程进行锁竞争时会造成线程切换的资源浪费（挂起和唤醒都需要消耗资源），这时就不让线程进行阻塞，而是进行无意义循环，
等待锁的获得，这就是自旋锁。

优点：减少线程切换资源消耗。

缺点：导致线程等待时间过长或者自旋线程较多，占用过多的CPU时间。

使用场景：被加锁代码等待时间较短。

# 自适应自旋锁
概念：是对自旋锁的优化，自旋的次数和时间不定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获
得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。如果对于某个锁，自旋很
少成功获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。

# 偏向锁、轻量级锁，重量级锁

[synchronized锁升级](9-Java多线程#相关锁介绍)

# 公平锁
概念：公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁。

优点：等待锁的线程不会饿死。

缺点：整体吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大。

使用场景：

# 非公平锁
概念：非公平锁是多个线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾等待。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁，所以非公平锁有可能出现后申请锁的线程先获取锁的场景。

优点：可以减少唤起线程的开销，整体的吞吐效率高。

缺点：处于等待队列中的线程可能会饿死，或者等很久才会获得锁。

使用场景：

# 重入锁（递归锁）
概念：当前线程获得重入锁，当线程再执行同对象同步方法时，如果对象相同并且是同一个线程，无需再次获得锁直接执行，这样也避免出现死锁出现。

优点：一定程度避免死锁。

缺点：

使用场景：

# 非重入锁
概念：当前线程调用向对象内同步方法时，因为当前线程已经获得当前类锁未释放，导致无法再获得。就会出现死锁现象。

优点：

缺点：当前线程出现死锁。

使用场景：

# 独享锁（排它锁）
概念：独享锁也叫排他锁，是指该锁一次只能被一个线程所持有。如果线程T对数据A加上排它锁后，则其他线程不能再对A加任何类型的锁。

优点：获得排它锁的线程即能读数据又能修改数据。

缺点：

使用场景：

# 共享锁
概念：共享锁是指该锁可被多个线程所持有。如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排它锁。

优点：

缺点：获得共享锁的线程只能读数据，不能修改数据。

使用场景：

# 分布式锁
单机多线程，在共享资源访问上面直接使用JDK自带的本地锁即可实现同步。对于分布式系统，就需要分布式锁。

锁满足条件：
- 互斥：任意时刻只能一个线程访问
- 高可用：锁服务高可用，就算客户端释放锁逻辑出现问题，锁任然会被释放，不影响其他线程使用共享资源。

实现条件：Redis和ZooKeeper

## Redis实现分布式锁

### 版本一
使用SETNX设置值，如果不存在则设置成功，存在的话什么都不在。
```shell
SETNX local value
 (integer) 1
```
释放锁：`DEL local`删除key即可

防止误删其他线程锁，需要进行值判断，这时需要Lua脚本保证一系列操作的原子性。
```lua
// 释放锁时，先比较锁对应的 value 值是否相等，避免锁的误释放
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

**问题：**_当执行释放锁操作的一个服务出错并无法释放锁，进而造成资源无法被其他服务获取。_

### 版本二
通过设置锁过期时间来完成锁的释放，保证锁一定会被释放。
```shell
SET key value EX|PX number NX
```
key：锁的名称
value：锁的值
EX|PX：过期时间秒|毫秒
NX：保证唯一，库中只存在一个key。

一定要保证设置指定 key 的值和过期时间是一个原子操作。

### Redisson实现Redis分布式锁

# 相关问题
## 死锁
死锁描述的是这样一种情况：多个进程/线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于进程/线程被无限期地阻塞，因此程序不可能正常终止。

## 死锁产生条件
- 互斥：资源必须处于非共享模式，即一次只有一个进程可以使用。如果另一进程申请该资源，那么必须等待直到该资源被释放为止。
- 占用并等待：一个进程至少占用一个资源，并等待另一个资源，而且该资源被另一个进程占用。
- 非抢占：资源被一个线程占用的时候，只能等完成一系列操作后，资源才被释放。
- 循环等待：有一组等待线程，p0,p1...pn,p0等待p1占用资源，p1等待...,pn等待p0占用资源。

**注意，只有四个条件同时成立时，死锁才会出现。**

## 避免死锁
1. 避免一个线程获得多锁。
2. 避免一个线程占用多个资源。
3. 尝试使用定时锁。

## 解决死锁方法

[解决死锁方法](https://javaguide.cn/cs-basics/operating-system/operating-system-basic-questions-01.html#%E8%A7%A3%E5%86%B3%E6%AD%BB%E9%94%81%E7%9A%84%E6%96%B9%E6%B3%95)


