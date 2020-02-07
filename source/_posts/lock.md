## 锁

## 作用：多线程同步

## 实现

- 内部实现
- - JVM实现
  - - synchronize1.6之前
    - synchronize1.6之后
  - 内置类实现（需要通过OS实现同步非常消耗系统资源，改为JVM内实现线程同步）
  - - juc下
    - - ReentrantLock
- 外部实现











## 多线程相关方法拓展

### wait()

#### 作用：使当前线程阻塞，用于多线程之间通信，与notify、notifyAll组合构建生产消费模型

### sleep()

### yield()

### park()

#### 作用：