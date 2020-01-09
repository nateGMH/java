# 1. 介绍

- **ThreadLocal**并不是一个Thread，而是Thread的局部变量
- **ThreadLocal**并不是用来并发控制访问一个共同对象，而是为了给每个线程分配一个只属于该线程的变量。
  - 它的功用非常简单，就是为每一个使用该变量的线程都提供一个变量值的副本，是每一个线程都可以独立地改变自己的副本，而不会和其它线程的副本冲突，实现线程间的数据隔离。

# 2. 实现原理

- 线程类**Thread**中有一个类型为**ThreadLocalMap**的变量为*threadLocals*

- **ThreadLocalMap**是一个映射表，内部实现是一个数组，每一个元素的类型为**Entry**

- **Entry**就是一个键值对(*Key-Value Pair*)，其 *Key* 就是**ThreadLocal**，其 *Value* 可以是任何对象



可以看出，threadlocal的操作其实就是对treadlocalmap进行操作，**ThreadLocal**的set()和get()方法的主体逻辑算是比较简单了，围绕主体逻辑，还做了一些特殊处理，譬如：线程中的映射表还未初始化时，调用createMap()进行初始化；在映射表中没有获取到*Value*时，通过setInitialValue()设置一个初始值，这种场景下，只需要实现initialValue()函数就可以了，这种**ThreadLocal**的使用方式很常见。



![image-20191029194542548](/Users/nate/Library/Application Support/typora-user-images/image-20191029194542548.png)

**ThreadLocalMap**的Entry是WeakReference的子类，这样能保证线程中的映射表的每一个Entry可以被垃圾回收，而不至于发生内存泄露。

**ThreadLocal**作为映射表的*Key*，需要具备唯一的标识，每创建一个新的**ThreadLocal**，这个标识就变的跟之前不一样了，**ThreadLocal**内部有一个名为threadLocalHashCode的变量，每创建一个新的**ThreadLocal**对象，这个变量的值就会增加0x61c88647。 正是因为有这么一个神奇的数字，它能够保证生成的Hash值可以均匀的分布在0~(2^N-1)之间，N是数组长度。