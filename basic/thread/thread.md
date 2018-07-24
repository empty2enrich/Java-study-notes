# Thread

 + [1、synchronized 的实现原理以及锁优化？](#1)
 + [2、锁：偏向锁、轻量级锁、重量级锁](#2)
 + [3、对象头](#3)
 + [4、自旋锁(spin lock) 和 互斥锁(mutual exclusion lock)](#4)
 + [5、CAS 和 ABA](#5)
 + [6、synchronized 用法](#6)
 + [7、synchronized 内部实现原理](#7)
 + [8、volatile 的实现原理](#8)
 + [9、thread 的 wait、sleep、yield、join](#9)
 + [10、Java 信号灯(Semaphore)](#10)
 + 

## <p id=1>1、synchronized 的实现原理以及锁优化？</p>

synchronized 关键字: 1、对于同一 scope 的方法、代码块(eg:同一个类的普通方法之间或者类的静态方法之间)之间会竞争同一资源,谁获取到资源谁执行,对于普通方法只有同一个实例调用才会发生同步,静态方法是同一个类调用就会方法同步

synchronized 用法：

```java
// synchronized 关键字修饰方法

public static synchronized void method(){}
public synchronized void method(){}

//代码块

public void method(){
    synchronized(this){

    }
}

```

monitorenter:每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权,步骤如下：

 + 1、entry count = 0,进入锁,entry count = 1,该线程为 monitor 所有者
 + 2、持有 monitor 线程重新进入,entry count++
 + 3、如果其他线程占用了 monitor ,进入阻塞状态,直到 monitor entry count =0 , 重新尝试获取 monitor 所有权

monitorexit:执行 monitorexit 的必须是 monitor 持有线程,没执行一次 entry count--。如果 entry count =0,退出 monitor

monitor:

对于静态变量、方法加上 synchronized 关键字后，反编译出来常量池中多了ACC_SYNCHRONIZED 标志符,

JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。

### Java 对象头：

Monitor record:

## <p id=2>2、锁：偏向锁、轻量级锁、重量级锁</p>

 + 偏向锁：偏向锁认为获取锁的总是同一个线程

偏向锁流程：在对象头中有字段 threadId ,第一次获取锁的时候就把自己 id 写入对象头 threadId 中,同时`把锁内是否偏向锁状态设置为 1` ,下次获取锁时，查看 threadId 是否与自身线程 id 一致,如果一致表示当前线程已经获取到锁无需再次获取，略过了轻量级、重量级锁的加锁阶段,提高了效率。

### jvm开启/关闭偏向锁

开启偏向锁：-XX:+UseBiasedLocking -XX:BiasedLockingStartupDelay=0;

关闭偏向锁：-XX:-UseBiasedLocking

 + 轻量级锁：轻量锁认为大多数情况不会出现锁竞争,即使出现竞争,获取锁的线程也会很快释放锁,获取不到锁的线程可以自旋等待一会,不会陷入阻塞状态

轻量级锁流程：线程在`执行同步代码块之前`，jvm 会在当初线程的栈帧中`创建用于存储锁记录的空间`，并把对象头的 Mask Word 复制到锁记录空间,官方称为 Displayed Mark Word。然后线程尝试持有(CAS, Compare and Swap) 把头对象中的 Mark Word 替换为指向锁记录的指针，如果成功,当前线程获得锁,如果失败表示有其他线程竞争,当前线程用自旋来获取锁,获取失败升级成重量级锁。

## <p id=3>3、对象头</p>

<table>
    <tr>
        <th rowspan="2">锁状态</th>
        <th colspan="2">25 bit</th>
        <th rowspan="2">4 bit</th>    
        <th>1 bit</th>
        <th>2 bit</th>
    </tr>
    <tr>
        <th>23 bit</th>
        <th>2 bit</th>
        <th>是否偏向锁</th>
        <th>锁标志位</th>
    </tr>
    <tr>
        <td>轻量级锁</td>
        <td colspan="4">指向栈中锁记录的指针</td>
        <td>00</td>
    </tr>
    <tr>
        <td>重量级锁</td>
        <td colspan="4">指向互斥量(重量级锁)的指针</td>
        <td>10</td>
    </tr>
    <tr>
        <td>GC 标记</td>
        <td colspan="4">空</td>
        <td>11</td>
    </tr>
    <tr>
        <td>偏向锁</td>
        <td>线程 id</td>
        <td>Epoch</td>
        <td>对象分代年龄</td>
        <td>1</td>
        <td>01</td>
    </tr>
</table>

## <p id=4>4、自旋锁(spin lock) 和 互斥锁(mutual exclusion lock)</p>

### 自旋锁开启

JDK1.6中-XX:+UseSpinning开启； 
-XX:PreBlockSpin=10 为自旋次数； 
JDK1.7后，去掉此参数，由jvm控制；

## <p id=5>5、CAS 和 ABA</p>

CAS(Compare and Swap,比较交换): CAS(old,new) , 将 old 值替换为 new,但在替换之前会先检查 old 值是否修改(),未修改执行替换操作。

ABA problem ： old 值修改为 A,再修改回 old , CAS 也能执行替换操作(检查 old 为原值,但其实已经发生变化了)，解决：各种乐观锁的实现中通常都会用版本戳version来对记录或对象标记，避免并发操作带来的问题，在Java中，AtomicStampedReference<E>也实现了这个作用，它通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题

## <p id=6>6、synchronized 用法</p>

```java
// synchronized 关键字修饰方法

public static synchronized void method(){}
public synchronized void method(){}

//代码块

public void method(){
    synchronized(this){
        
    }
}
```

## <p id=7>7、synchronized 内部实现原理</p>

Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象

## <p id=8>8、volatile 的实现原理</p>

volatile 保证了可见性、禁止指令重排序(保证在使用 volatile 变量前的代码一定在之前执行,且执行完了才到 volatile 变量,变量后面的得在之后执行),不能保证原子性；

## <p id=9>9、thread 的 wait、sleep、yield、join</p>

## <p id=10>10、Java 信号灯(Semaphore)</p>




