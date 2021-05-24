##  为什么使用Synchronized

线程安全是并发编程中的至关重要的，造成线程安全问题的主要原因：

1. 临界资源，存在共享数据
2. 多线程共同操作共享数据

Java中的synchronized关键字是为多线程场景下防止临界资源访问冲突提供支持，可以保证在同一刻，只有同一个线程可以执行某个方法或某个方法块操作共享数据

即当要执行代码使用synchronized关键字时，它将检查锁是否可用，然后获取锁，执行代码，最后在释放锁。

## Synchronized的三种使用方式

1. synchronized方法：synchronized当前实例对象，进入同步代码前要获得当前实例的锁
2. synchronized静态方法：syncharonized当前类的class对象，进入同步代码前当前类对象的锁

3. synchronized代码块：syncharonized括号里面的对象，给特定的对象加锁，进入代码块库前要获得给特定对象的锁

## Synchronized的原理

synchronized是基于JVM实现的一种锁，其中锁的获取和释放分别是monitorenter和monitorexit指令。

加入sychronized关键字的代码，生成的字节码文件会多出monitorenter和monitorexit两条指令，并且会多一个ACC_SYNCHRONIZED标志位。

当方法被调用时，调用指令将会检查方法的ACC_SYNCHRONIZED标志位是否被设置，如果设置了，执行线程将会获取monitor，获取成功之后才能执行方法体，方法执行后再释放monitor。

在方法执行期间，其他任何线程都无法再获得同意monitor对象，其实本质上没有区别，只是方法的同步是一种隐式的方法来实现，无需通过字节码来完成。

在Java1.6之后，synchronized在实现上分为偏向锁、轻量级锁、重量级锁，其他中偏向锁在Java1.6中默认开启的，轻量级锁在多线程竞争下会膨胀为重量级锁，有关锁的数据都保存在对象头中。

* 偏向锁：在只有一个线程访问同步代码块的时，通过CAS操作获取锁
* 轻量级锁：当存在多个线程交替访问同步块，偏向锁就会升级为轻量级锁，当线程获取轻量级锁失败，说明存在着竞争，轻量级锁会膨胀为重量级锁，当前线程会通过自旋（通过CAS操作不断获取锁），后面其他获取锁的进程直接进入阻塞状态。
* 重量级锁：锁获取失败则线程阻塞，因此会有线程上下文切换，性能最差

## 锁优化

### 适应性自旋（Adaptive Spinning）

从轻量级锁获取的流程中可以看到，当线程在获取轻量级锁的过程执行CAS操作失败时，是要通过自旋来获取重量级锁的，问题在于，自旋是需要消耗CPU的，如果一直获取不到锁，那该线程就会一直处在自旋状态，白白浪费CPU资源。

其中解决这个问题最简单的办法就是指定自旋次数，例如让其循环10次，如果还没有获取到锁就进入阻塞状态。但是Jdk采用了更聪明的方法──适应性自旋，简单来说就是如果获取锁成功了，则下次的自旋次数就增加，如果获取锁失败了，那自旋的次数就会减少。

### 锁粗化（Lock Coarsening）

锁粗化的概念应该比较好理解，就是将多次连接在一起的加锁、解锁操作合并为一次，将多个连续的锁扩展成一个范围更大的锁。

```java
public class StringBufferTest {
     StringBuffer stringBuffer = new StringBuffer();
     public void append(){
       stringBuffer.append("a");
			 stringBuffer.append("b");
			 stringBuffer.append("c");
     } 
}
```

这里每次调用stringBuffer.append方法都需要加锁和解锁，如果虚拟机检测到有一系列连串的对同一 个对象加锁和解锁操作，就会将其合并成一次范围更大的加锁和解锁操作，即在第一次append方法时 进行加锁，最后一次append方法结束后进行解锁。

### 锁消除（Lock Elimination）

锁消除即删除不必要的加锁操作，根据代码逃逸技术，如果判断一段代码中，堆上的数据不会逃逸出当前线程，那么可以认为这段代码是线程安全的，则不必要加锁

```java
public class SynchronizedTest02 {
     public static void main(String[] args) {
         SynchronizedTest02 test02 = new SynchronizedTest02();
         for (int i = 0; i < 10000; i++) {
							i++; 
         }
         long start = System.currentTimeMillis();
         for (int i = 0; i < 100000000; i++) {
             test02.append("abc", "def");
         }
         System.out.println("Time=" + (System.currentTimeMillis() - start));
     }
     public void append(String str1, String str2) {
         StringBuffer sb = new StringBuffer();
         sb.append(str1).append(str2);
} }
```

虽然StringBuffer的append是一个同步方法，但是这段程序中的StringBuffer属于一个局部变量，并且 不会从该方法中逃逸出去，所以其实这过程是线程安全的，可以将锁消除。

## Synchronized的缺点

synchronized会让没有得到锁的资源进入block状态，争夺到资源之后又会转为Running状态，这个过程涉及到操作系统用户模式和内核模式的切换，代价比较高。

Java1.6为synchronized做了优化，增加了偏向锁到轻量级锁到重量级锁的过渡，但是在最终转变为重量锁之后，性能仍然很低。