# 简介

**JAVA中的AQS队列从根本上来讲是基于`CAS`的典型实现，同时也是使用volatile关键字的典型案例。它首先依赖于java中的java.util.concurrent.locks.LockSupport，而LockSupport又是基于Unsafe类来实现同步和操作系统函数的调用，Unsafe又是基于SMT(硬件级别同步多线程技术(CPU))来实现**



## AQS的由来

**java语言实现多线程同步的两种实现方案**

- **[Synchronizer](http://orgjinhc.github.io/2020/01/28/Synchronized/)** 

- **AbstractQueuedSynchronizer**

**1.6之前只有synchronize这一种实现同步方案，由于synchronize属于重量级锁，同步需要操作系统的支持，很多同步逻辑不是很复杂的业务逻辑就变得非常的耗时，但是不同步又会有线程安全问题，所以aqs就孕育而生了，aqs要解决的首要问题就是1.6之前的synchronize同步效率低的问题，大概的思想就是通过CAS非阻塞并发算法+CLH队列+操作系统函数（park、unpark）来实现，具备的特性如下**

- **阻塞等待队列**
- **共享/独占**
- **公平/非公平**
- **可重入**
- **允许中断**

**并发**

- **交替执行=CAS + spin**
- **同步执行=CAS + spin + CLH**

**并行=CAS + spin + CLH + 操作系统函数**



**synchronize1.6之后也针对效率问题进行了相关的改进，因为程序大部分时间或大部分需要同步的逻辑并不存在大量的竞争，所以synchronize针对锁的种类和类型进行了升级，提出了锁升级或叫锁膨胀流程。**



`加锁流程`

> ![Aqs加锁流程](http://lion-heart.online/blog/2020-02-02-070427.png)
>
> 
>
> ReentrantLock：`持有锁的线程不在队列中`
>
> 非公平和公平的区别就是：6-19之间的区别
>
> ```java
> //	第一次进入直接通过state状态获取锁
> //	第二次进入（自旋）通过查看队列状态获取锁
> protected final boolean tryAcquire(int acquires) {
>     final Thread current = Thread.currentThread();
>     int c = getState();
>     if (c == 0) {
>       	//	公平锁实现：队列未初始化或队列已初始化后队列内排队的线程，如果是第一个节点获取锁
>         if (!hasQueuedPredecessors() &&
>             compareAndSetState(0, acquires)) {
>             setExclusiveOwnerThread(current);
>             return true;
>         }
>       
>         //	非公平实现：同时去CAS，谁成功谁持有锁
>       	if (compareAndSetState(0, acquires)) {
>                     setExclusiveOwnerThread(current);
>                     return true;
>         }
>     }
>     else if (current == getExclusiveOwnerThread()) {
>         int nextc = c + acquires;
>         if (nextc < 0)
>             throw new Error("Maximum lock count exceeded");
>         setState(nextc);
>         return true;
>     }
>     return false;
> }
> ```
>
> ​	`公平性流程`
>
> ```java
> public final boolean hasQueuedPredecessors() {
>     // The correctness of this depends on head being initialized
>     // before tail and on head.next being accurate if the current
>     // thread is first in queue.
>     Node t = tail; // Read fields in reverse initialization order
>     Node h = head;
>     Node s;
>     return h != t &&
>       	//	未添加队列的线程和刚从队列醒来的线程同时来争夺锁的情况下，保护公平性
>         ((s = h.next) == null || s.thread != Thread.currentThread());
> }
> ```
>
> `阻塞流程`
>
> ```java
> private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
>     int ws = pred.waitStatus;
>     if (ws == Node.SIGNAL)
>         /*
>          * This node has already set status asking a release
>          * to signal it, so it can safely park.
>          */
>         return true;
>     if (ws > 0) {
>         /*
>          * Predecessor was cancelled. Skip over predecessors and
>          * indicate retry.
>          */
>         do {
>             node.prev = pred = pred.prev;
>         } while (pred.waitStatus > 0);
>         pred.next = node;
>     } else {
>         /*
>          * waitStatus must be 0 or PROPAGATE.  Indicate that we
>          * need a signal, but don't park yet.  Caller will need to
>          * retry to make sure it cannot acquire before parking.
>          */
>         compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
>     }
>     return false;
> }
> ```
>
> 

`解锁流程`

> ```java
> //	释放锁方法
> if (tryRelease(arg)) {
>     Node h = head;
>   	//	队列是否初始化，并且队列内是否有阻塞的线程
>     if (h != null && h.waitStatus != 0)
>       	//	唤醒第一个阻塞的线程
>         unparkSuccessor(h);
>     return true;
> }
> ```
>
> `tryRelease`：更新state、释放独占标识
>
> ```java
> protected final boolean tryRelease(int releases) {
>     int c = getState() - releases;
>     if (Thread.currentThread() != getExclusiveOwnerThread())
>         throw new IllegalMonitorStateException();
>     boolean free = false;
>     if (c == 0) {
>         free = true;
>         setExclusiveOwnerThread(null);
>     }
>     setState(c);
>     return free;
> }
> ```
>
> `唤醒阻塞线程方法`
>
> ```java
> private void unparkSuccessor(Node node) {
>     //	如果状态为负(即(可能需要信号)试着发出信号。如果这个操作失败，或者通过等待线程改变状态，这是可以的。
>   	//	阻塞线程的状态是由下一个节点去修改状态,等价于回复状态也需要通过未阻塞的线程去唤醒和修改状态
>     int ws = node.waitStatus;
>     if (ws < 0)
>         compareAndSetWaitStatus(node, ws, 0);
> 
>     Node s = node.next;
>   	//	队列头存放的是取消或已经释放的线程对象，需要从尾部向前开始寻找需要被唤醒的线程对象
>     if (s == null || s.waitStatus > 0) {
>         s = null;
>         for (Node t = tail; t != null && t != node; t = t.prev)
>             if (t.waitStatus <= 0)
>                 s = t;
>     }
>     if (s != null)
>       	//	唤醒线程对象
>         LockSupport.unpark(s.thread);
> }
> ```
>
> 



## AQS概念

- 队列（双向链表）
  - 元素（Node）
    - prev
    - next
    - Thread
    - waitState（默认值为0）
- AbstractQueuedSynchronizer.state
  - SIGNAL：此节点的后继节点被(或即将被)阻塞(通过park)，因此当前节点在释放或取消时必须取消其后继节点。为了避免争用，获取方法必须首先表明它们需要一个信号，然后重试原子获取，然后，失败时阻塞
  - CANCELLED：由于超时或中断，此节点被取消。节点永远不会离开这个状态。特别是，具有已取消节点的线程将不再阻塞。
  - CONDITION：此节点当前位于条件队列上。它不会被用作同步队列节点
    直到传输，此时状态将被设置为0
  - PROPAGATE：一个被释放的节点应该被传播到其他节点。这是设置(只针对头节点)
    即使其他操作已经介入，doreleased也必须确保传播继续。

- - ![AQS.state](http://lion-heart.online/blog/2020-01-31-103600.png)














## AQS使用方式

