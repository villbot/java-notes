## 什么是ReentrantLock

RenntrantLock是一个典型的独占AQS，同步状态为0时表示空闲。当线程获取到空闲的同步状态时，它会将同步状态+1，将同步状态改为非空闲，于是其他线程等待挂起。在修改同步状态的同时，并记录下自己的线程，作为后续重入的依据，即一个线程持有某个对象的锁时，再次获取这个对象的锁是可以成功的，如果是不可重入的锁的话就会造成死锁。

RenntrantLock会涉及到公平锁和非公平锁，实现关键在于成员变量sync的实现不同，这是锁实现互斥同步的核心。

```java
// 公平锁和非公平锁的变量
private final Sync sync;

//父类
abstract static class Sync extends AbstractQueuedSynchronizer{}

//公平锁子类
static final class FairSync extends Sync {}

//非公平锁子类
static final class NonfairSync extends Sync {}
```

## 公平锁跟非公平锁是什么？ 有什么区别？

公平锁是指当锁可用时，在锁上等待时间最长的线程将获得锁的使用权，即先进先出。而非公平锁则随机分配这种使用权，是一种抢占机制，是随机获得锁，并不是先来的一定能先得到锁。

RenntrantLock 提供了一个构造方法，可以实现公平锁跟非公平锁：

```java
public ReentrantLock (boolean fair ){
  sync = fair ? new FairSync(): new NonfairSync();
}
```

虽然公平锁在公平性得以保障，但因为公平锁没有考虑操作系统对线程的调度因素及其他因素，可能会影响性能。

虽然非公平锁效率比较高，但是非公平模式在申请锁的时候线程足够多，可能会导致某些线程一直得不到锁，这就是非公平锁的"饥饿"问题

但大部分场景下还是采用非公平锁，因为其性能比公平锁好很多，但是公平锁能够避免线程饥饿，某下情况下也使用。

# RenntrantLock::lock 公平锁实现

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_05_24/image-20210524161206035.png" alt="image-20210524161206035" style="zoom:50%;" />