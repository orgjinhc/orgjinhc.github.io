# 简介

**根据进入阻塞状态的方式不同，阻塞状态也会有细微的差异**

## 阻塞状态分类

- sleeping
- on object monitor
- parking

**实际上这几种“阻塞”状态，恰恰就是Java中不同的锁机制实现**

### Monitor

**Java中对悲观锁思想的实现就是我们最常使用的synchronized关键字，前面的文章有针对synchronize原理做解释[`Synchronized原理分析`](http://orgjinhc.github.io/2020/01/28/Synchronized/) ，Monitor机制就是synchronized锁机制升级为“重量级锁”后的工作机制，也就是JVM对操作系统级别的互斥锁(Mutex Lock)的管理过程**

