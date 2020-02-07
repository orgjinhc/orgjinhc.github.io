---
title: synchronize
author: 靳宏财
top: true
cover: true
toc: true
summary: synchronize、CAS、Monitor相关实现
categories: java并发
tags:
  - java基础
  - 多线程	
date: 2020年01月30日19:08:13


---

# Synchronized同步锁的底层实现

## synchronize是什么

> Java语言的关键字，可用来给对象和方法或者代码块加锁

## synchronize作用

`先看段代码`

```java
public class Synchronized {
    private static final CountDownLatch COUNT_DOWN_LATCH = new CountDownLatch(3);
    private static final Object LOCK_OBJ = new Object();
    public static void main(String[] args) throws Exception {
        new Thread(new Runnable() {
            public void run() {
                print();
            }
        }, "Thread1").start();

        new Thread(new Runnable() {
            public void run() {
                print();
            }
        }, "Thread2").start();

        new Thread(new Runnable() {
            public void run() {
                print();
            }
        }, "Thread3").start();
        COUNT_DOWN_LATCH.await();
    }

    private static void lockedPrint() {
        synchronized (LOCK_OBJ) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("currentThread : [{}]", Thread.currentThread().getName());
            COUNT_DOWN_LATCH.countDown();
        }
    }
   
    private static void unLockedPrint() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("currentThread : [{}]", Thread.currentThread().getName());
            COUNT_DOWN_LATCH.countDown();
    }
}
```

`对应的输出结果`

> **unLockedPrint**
>
> > 12:06:32.738 [Thread1] INFO lock.Synchronized - currentThread : [Thread1]
> > 12:06:32.738 [Thread3] INFO lock.Synchronized - currentThread : [Thread3]
> > 12:06:32.738 [Thread2] INFO lock.Synchronized - currentThread : [Thread2]
>
> **lockedPrint**
>
> > 12:11:31.173 [Thread1] INFO lock.Synchronized - currentThread : [Thread1]
> > 12:11:32.179 [Thread3] INFO lock.Synchronized - currentThread : [Thread3]
> > 12:11:33.179 [Thread2] INFO lock.Synchronized - currentThread : [Thread2]
>
> **结论**：lockedPrint方法可以按照我们期望的流程间隔1s输出结果，通过synchronize关键字修饰类和对象可以`确保多个线程同一时刻，只能有一个线程处于方法或同步块中`

## synchronize的目的

> 多线程并发环境下保证同步和数据安全

## synchronize的原理

**synchronize加、释放锁流程图**

![synchronize整体流程](http://lion-heart.online/blog/2020-01-28-043438.png)

**流程：多线程同时访问同步方法或代码块时，线程必须先获取对象监视器才有方法执行权。获取失败进入同步队列，线程状态变为BLOCKED状态。只有当持有锁的线程释放锁后，唤醒阻塞在同步队列中的线程，使其重新尝试获取监视器锁，往复此流程，如果持有锁的线程调用wait()方法，当前线程释放monitor，线程状态变成WAITING，等待其他线程唤醒，如若唤醒则从等待队列进入到同步队列继续抢夺monitor持有权**

#### Monitor

> ​	**Monitor是一个`同步工具`，也可以说是一种`同步机制`，java所提供的同步机制、互斥锁机制，这个机制的保障来源于监视锁`Monitor`，它内置于每一个Object对象中，任何一个对象都有自己的监视器**
>
> ​	**<font color = red size = 5>疑问</font>：代码里添加synchronize关键字可以实现线程安全是因为synchronize关键字是通过monitor同步机制实现的，那监视器对象或锁信息在哪？**
>
> ​	**为了解决上面的疑问，我们还需要了解对象在内存中存储的布局，一图看懂对象存储布局**
>
> ![对象存储布局](http://lion-heart.online/blog/2020-01-29-012949.png)
>
> ##### 一、`对象存储布局`
>
> 1. **Java对象头**：下面重点介绍
> 2. **实例数据**：各种类型的字段内容（父类 + 子类）
> 3. **对齐补充**：占位符（没有其他作用），虚拟机内存管理系统要求对象起始位置必须是 8 的整数倍，实例数据部分没有对齐，则填充（1.是方便CPU进行计算，2.GC更高效的回收）
>
> ##### 二、Java对象头主要包括两部分数据
>
> - **类型指针（Klass Pointer）**：是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例（数组无法制定大小，还需要存储数组长度）
> - **标记字段(Mark Word)**：用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等,它是实现轻量级锁和偏向锁的关
>
> ##### 三、原理
>
> **`32`位`HotSpot`虚拟机Java对象头存储结构**
>
> ![Java对象头存储结构](http://lion-heart.online/blog/2020-01-29-021240.png)
>
> **再来看下虚拟机源码中对象头的数据结构，位置：markOop.hpp**
>
> ![Java对象头数据结构](http://lion-heart.online/blog/2020-01-30-030826.png)
>
> **对比对象头数据存储图和数据结构图可以得知**
>
> **39行指定的是`无锁态`**
>
> **40行指定的是`偏向态`**
>
> **42行指定的是`清量级锁`、`重量级锁`、`GC标记`**
>
> **epoch：保存偏向时间戳**
>
> **lock：锁标志位（枚举如下图）**
>
> ![lock枚举](http://lion-heart.online/blog/2020-01-29-022643.png)
>
> **锁标识位注解**
>
> ![锁标志位解释](http://lion-heart.online/blog/2020-01-29-022850.png)
>
> **<font color =red size = 5 >结论</font>：在hotspot虚拟机中，不管是32/64位JVM，都是1bit偏向锁+2bit锁标志位（两个锁位用于描述三种状态:锁定/解锁/监视），同时根据枚举得知，只有锁标识位为10才触发监视器锁，也就是我们常说的重量级锁**
>
> **回答开篇的两个问题**
>
> 1. **锁信息在哪**
>
>    > **synchronized的底层实现是完全依赖JVM虚拟机Java对象header中的Mark Word来标识对象加锁状态**
>
> 2. **监视器锁在哪**
>
>    > **Hotspot，Monitor是ObjectMonitor.class提供服务，基于C++里ObjectMonitor.hpp实现**
>    >
>    > ![openJDK,ObjectMonitor实现](http://lion-heart.online/blog/2020-01-29-094610.png)
>    >
>    > **Steps流程图**
>    >
>    > ![Steps](http://lion-heart.online/blog/2020-01-29-092753.png)
>    >
>    > **ObjectMonitor方法如下**
>    >
>    > ![ObjectMonitor方法列表](http://lion-heart.online/blog/2020-01-29-095205.png)
>    >
>    > <font color = red size = 5>结论</font>：**结合开篇synchronize加锁、释放锁流程图可知，加锁通过enter方法，解锁通过exit方法**

#### 锁的升级

##### JDK1.6之前加锁和解锁流程

**通过ObjectMonitor的enter()方法获取锁，通过exit()方法释放锁，这也是导致我们经常会说`synchronize`是`重量级锁的`原因，因为`java的线程`是映射到`操作系统的原生线程`，如果要wait或notify一个线程就需要OS帮助，需要从`用户态`切换到`内核态`，这种转换需要花费很多`处理器时间`**

`enter()`

```c++
void ATTR ObjectMonitor::enter(TRAPS) {
  Thread * const Self = THREAD ;
  void * cur ;
  //	通过CAS尝试把monitor的`_owner`字段设置为当前线程
  cur = Atomic::cmpxchg_ptr (Self, &_owner, NULL) ;
  if (cur == NULL) {
     assert (_recursions == 0   , "invariant") ;
     assert (_owner      == Self, "invariant") ;
     return ;
  }
  //	如果旧值和当前线程一样，说明当前线程已经持有锁，此次为重入，_recursions自增，并获得锁。
  if (cur == Self) {
     // TODO-FIXME: check for integer overflow!  BUGID 6557169.
     _recursions ++ ;
     return ;
  }

  // 如果当前线程是第一次进入该monitor，设置_recursions为1，_owner为当前线程
  if (Self->is_lock_owned ((address)cur)) {
    assert (_recursions == 0, "internal state error");
    _recursions = 1 ;
    _owner = Self ;
    OwnerIsThread = 1 ;
    return ;
  }
    // 更改java线程状态以指示在监视器上阻塞
    JavaThreadBlockedOnMonitorEnterState jtbmes(jt, this);

    DTRACE_MONITOR_PROBE(contended__enter, this, object(), jt);
    if (JvmtiExport::should_post_monitor_contended_enter()) {
      JvmtiExport::post_monitor_contended_enter(jt, this);
    }

    OSThreadContendState osts(Self->osthread());
    ThreadBlockInVM tbivm(jt);

    Self->set_current_pending_monitor(this);

		//	通过自旋执行ObjectMonitor::EnterI方法等待锁的释放
    for (;;) {
      jt->set_suspend_equivalent();
      //	当前线程被封装成ObjectWaiter对象node，通过CAS把node节点push到_cxq列表中，通过自旋尝试获取锁，如果还是没有获取到锁，则通过park将当前线程挂起，等待被唤醒*******enter重的原因********
      EnterI (THREAD) ;
      if (!ExitSuspendEquivalent(jt)) break ;
          _recursions = 0 ;
      _succ = NULL ;
      exit (false, Self) ;
      jt->java_suspend_self();
    }
    Self->set_current_pending_monitor(NULL);
}
```

`exit():`通过退出monitor的方式实现锁的释放，并通知被阻塞的线程

```c++
void ATTR ObjectMonitor::exit(bool not_suspended, TRAPS) {
   Thread * Self = THREAD ;
   if (THREAD != _owner) {
     if (THREAD->is_lock_owned((address) _owner)) {
       // Transmute _owner from a BasicLock pointer to a Thread address.
       // We don't need to hold _mutex for this transition.
       // Non-null to Non-null is safe as long as all readers can
       // tolerate either flavor.
       assert (_recursions == 0, "invariant") ;
       _owner = THREAD ;
       _recursions = 0 ;
       OwnerIsThread = 1 ;
     } else {
       // NOTE: we need to handle unbalanced monitor enter/exit
       // in native code by throwing an exception.
       // TODO: Throw an IllegalMonitorStateException ?
       TEVENT (Exit - Throw IMSX) ;
       assert(false, "Non-balanced monitor enter/exit!");
       if (false) {
          THROW(vmSymbols::java_lang_IllegalMonitorStateException());
       }
       return;
     }
   }
  //	根据不同的策略（由QMode指定），从cxq或EntryList中获取头节点，通过ObjectMonitor::ExitEpilog方法唤醒该节点封装的线程，唤醒操作最终由unpark完成 *********exit()*********
  int QMode = Knob_QMode ;
  //	....
}
```

##### JDK1.6对加锁的实现引入大量的优化来减少锁操作开销

还有我们经常说的对synchronize的升级也是1.6之后对锁的`实现`和`类型`进行了优化，目的是降低加锁、释放锁的开销，出现了`自旋锁`、`偏向锁`、`轻量级锁`、`重量级`锁等等，下面我们会一起讨论

###### `偏向锁`（Biased Locking）：目的是消除共享资源在无多线程竞争情况下的同步原语。使用CAS记录获取它的线程。下一次同一个线程进入则偏向该线程，无需任何同步操作

###### `轻量级锁`（Lightweight Locking）：目的是在没有多线程竞争的情况下避免重量级互斥锁，只需要依靠一条CAS原子指令就可以完成锁的获取及释放

###### `锁粗化`（Lock Coarsening）：将多个连续的锁扩展成一个范围更大的锁，用以减少频繁互斥同步导致的性能损耗

###### `锁消除`（Lock Elimination）：JIT在运行时，通过逃逸分析，如果判断一段代码中，堆上的所有数据不会逃逸出去从来被其他线程访问到，就可以去除这些锁

###### `适应性自旋`（Adaptive Spinning）：为了避免线程频繁挂起、恢复的状态切换消耗。产生了忙循环（循环时间固定），即自旋。JDK1.6引入了自适应自旋。自旋时间根据之前锁自旋时间和线程状态，动态变化，用以期望能减少阻塞的时间

 ######`流程`

> ![整体流程](http://lion-heart.online/blog/2020-01-30-085923.png)
>
> `引用`：[Java多线程：锁的底层实现](https://blog.csdn.net/qq_29753285/article/details/81299509?utm_source=blogxgwz0)

synchronized关键字修饰的代码段，在JVM被编译为monitorenter、monitorexit指令来获取和释放互斥锁，解释器执行monitorenter时会进入到`InterpreterRuntime.cpp`的`InterpreterRuntime::monitorenter`函数，具体实现如下：

```c++
//  JavaThread* thread:java线程
//  BasicObjectLock:包含一个BasicLock和一个指向Object对象的指针oop
IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorenter(JavaThread* thread, BasicObjectLock* elem))
  //	UseBiasedLocking:开启偏向锁
  if (UseBiasedLocking) {
		//	尝试获取偏向锁
    ObjectSynchronizer::fast_enter(h_obj, elem->lock(), true, CHECK);
  } else {
    //	轻量级锁的获取
    ObjectSynchronizer::slow_enter(h_obj, elem->lock(), CHECK);
  }
}

//	fast_enter实现
void ObjectSynchronizer::fast_enter(Handle obj, BasicLock* lock, bool attempt_rebias, TRAPS) {
 //	如果开启了偏向锁
 if (UseBiasedLocking) {
  	//	如果不在安全点
    if (!SafepointSynchronize::is_at_safepoint()) {
      //	.............
    } else {
      //	当前处于安全点，调用revoke_at_safepoint
      BiasedLocking::revoke_at_safepoint(obj);
    }
 }
 //	如果未开启偏向锁，或竞争偏向锁失败
 slow_enter (obj, lock, THREAD) ;
}
  


//	revoke_at_safepoint实现
BiasedLocking::Condition BiasedLocking::revoke_and_rebias(Handle obj, bool attempt_rebias, TRAPS) {
	1. 判断当前对象是否为可偏向（101），且偏向时间戳已过期（没有其他线程在占用该对象），如果是，则进入步骤2，否则进入步骤3 
	2. 执行CAS操作将markword中的线程ID替换为本线程ID。如果成功则进入步骤4，否则进入步骤3 
	3. 存在竞争，当达到全局安全点（safepoint），获得偏向锁的线程被挂起，撤销偏向锁，并升级为轻量级，升级完成后被阻塞在安全点的线程继续执行同步代码块； 
	4. 执行同步代码
}
```





## 拓展

### CAS

>  CAS（Compare And Swap）比较并替换。在Java中，主要使用在`Atomic`包下，都是基于`Unsafe`类相关方法
>
>  涉及三个值：**内存中**真正存的值（可能被其他线程改变）、逻辑上的**原值**、（当前线程）要写入的**新值**。通过循环检查内存中的值是不是原来的值，以此来判断是不是正有其他线程在改变它。判断成功立即写入新值，这一步是原子的
>
>  **Unsafe：提供了一些绕开JVM的更底层功能，基于它的实现可以提高效率。它所分配的内存不被GC回收，同时提供了JNI某些功能的简单替代**
>
>  **结论：利用CPU的CAS指令，同时借助JNI来完成Java的非阻塞算法。其它原子操作都是利用类似的特性完成的。而整个J.U.C都是建立在CAS之上的，因此对于synchronized阻塞算法，J.U.C在性能上有了很大的提升。**

#### 案例

##### AtomicInteger

```java
java.util.concurrent.atomic.AtomicInteger#compareAndSet(原值，新值)
  
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // 设置为使用Unsafe.compareAndSwapInt进行更新
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
          //	初始化获取内存偏移值
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    } 
  
  /**
   *	expect：期望值（原值）
   *	update：修改值（目标值）
   */
	public final boolean compareAndSet(int expect, int update) {
    	//	this代表当前AtomicInteger对象
      //	valueOffset:value值得内存地址偏移量
    	return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
	}
}
```

##### Unsafe

```java
public final class Unsafe {
    private Unsafe() {}
    private static final Unsafe theUnsafe = new Unsafe();
    
    //	拿到调用它的那个Class的ClassLoader判断是否是SystemDomainLoader,如果不是则抛安全异常
    //	简而言之就是判断是否可以信任调用者返给他实例
    @CallerSensitive
    public static Unsafe getUnsafe() {
        Class<?> caller = Reflection.getCallerClass();
        if (!VM.isSystemDomainLoader(caller.getClassLoader()))
            throw new SecurityException("Unsafe");
        return theUnsafe;
    }

  //	java实现:`unsafe.compareAndSwapInt(this, valueOffset, expect, update)`
  public final native boolean compareAndSwapInt(java.lang.Object o, long l, int i, int i1);
}
```

###### 自定义Unsafe类相关方法

```java
public class PersonalUnsafe {
    private static int changeValue = 88;
    private static long ageOffsetByCustom = 00;
    private static String AGE = "age";
    public static void main(String[] args) throws Exception {
        //  通过反射实例化Unsafe
        Field f = Unsafe.class.getDeclaredField("theUnsafe");

        //  设置暴力访问
        f.setAccessible(true);

        //  获取私有方法
        Unsafe unsafe = (Unsafe) f.get(null);

        //实例化Person
        long ageOffsetByActual = 0;
        Person person = (Person) unsafe.allocateInstance(Person.class);
        person.setAge(40);
        person.setName("zhangsan");
        for (Field field : Person.class.getDeclaredFields()) {
            if (field.getName().equals(AGE)) {
                ageOffsetByActual = unsafe.objectFieldOffset(field);
            }
            log.info("{}:对应的内存偏移地址:{}", field.getName(), unsafe.objectFieldOffset(field));
        }

        //  通过内存偏移地址修改age的值
        //  错误的内存地址
        log.info("custom offset result:{}", unsafe.compareAndSwapInt(person, ageOffsetByCustom, person.getAge(), changeValue));

        //  内存中真正存的值和逻辑上的原值不相等
        int illegalAgeExpectValue = 10;
        log.info("custom expect result:{}", unsafe.compareAndSwapInt(person, ageOffsetByActual, illegalAgeExpectValue, changeValue));

        /**
         * ageOffsetByActual:内存中真正存的值
         * person.getAge():逻辑上的原值
         * 100:要写入的新值
         * 通过循环检查内存中的值是不是原来的值，以此来判断是不是正有其他线程在改变它
         *
         */
        log.info("actual result:{}", unsafe.compareAndSwapInt(person, ageOffsetByActual, person.getAge(), changeValue));
        log.info("age修改后的值:{},地址:{}", person.getAge(), unsafe.objectFieldOffset(Person.class.getDeclaredField(AGE)));
    }
}

@Data
class Person {
    private int age;
    private String name;
}
```

`compareAndSwapInt`，是一个native方法，该方法的实现位于`unsafe.cpp`中

![Unsafe.CompareAndSwapInt](http://lion-heart.online/blog/2020-01-28-093824.png)

> 1. **JNIHandles::resolve就是把obj（Persion）转换为oop**
>
> 2. **然后调用index_oop_from_field_offset_long(p, offset)方法，其中offset为修改的字段在对象所占内存中的偏移位置，最终得到的addr就是该字段在内存中的位置，这里可以简单理解为对象地址加上offset**
>
> 3. **得到字段地址addr之后就会调用核心的Atomic::cmpxchg方法，该方法定义在atomic.hpp中**
>
>    > ```java
>    > inline jint Atomic::cmpxchg (jint exchange_value, volatile jint* dest, jint compare_value) {
>    >   	//	判断是否是多处理器
>    >     int mp = os::isMP();
>    >     _asm {
>    >       	//	变量内存位置放edx
>    >         mov edx, dest  
>    >         //	要更新的值放ecx
>    >         mov ecx, exchange_value  
>    >         //	原内存值放eax  
>    >         mov eax, compare_value 
>    >         //	原内存值放eax  
>    >         LOCK_IF_MP(mp) ≈
>    >         // 	这句是真正的CAS操作
>    >         cmpxchg dword ptr [edx], ecx
>    >         //	dword ptr 将 [edx] 强制类型转换成双字
>    >         //	cmpxchg 将 eax 里 内存原值 与（转换后的）对象值 比较
>    >         //	如果相等，就是没别的线程在改变这个对象，那么这个线程就可以改了，将ecx值更新到这个对象。
>    >     }
>    > }
>    > ```
>    >
>    >  **LOCK_IF_MP:会根据当前处理器的类型来决定是否为cmpxchg指令添加lock前缀。如果程序是在多处理器上运行，就为cmpxchg指令加上lock前缀（lock cmpxchg）。反之，如果程序是在单处理器上运行，就省略lock前缀（单处理器自身会维护单处理器内的顺序一致性，不需要lock前缀提供的内存屏障效果）**
>
> 4. **最后用返回的内存中的值和期望值做比较直接返回**

结论：

- Intel使用缓存锁定来保证指令执行的原子性
- **禁止**该指令与前面和后面的指令**重排序**
- 把**写缓冲**区的所有数据**刷新**到内存中

