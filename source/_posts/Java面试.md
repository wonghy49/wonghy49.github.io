---
title: Java面试宝典
date: 2019-08-21 16:43:24
categories: 面试
tags: [面试]
comments: true
toc: true
---

# 一、单例模式

1、懒汉式（线程不安全）

```java
    public class Singleton {  
        private static Singleton instance;  
        private Singleton (){}  
      
        public static Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
        }  
    }  
```

2、懒汉式（线程安全）

```java
    public class Singleton {  
        private static Singleton instance;  
        private Singleton (){}  
        public static synchronized Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
        }  
    }  
```

3、饿汉式

```java
    public class Singleton {  
        private static Singleton instance = new Singleton();  
        private Singleton (){}  
        public static Singleton getInstance() {  
        return instance;  
        }  
    }  
```

4、饿汉式（更新）





# 二、线程池的原理及实现



# 三、finally一定会被执行吗？

答案是错的。有两种情况finally不会被执行

1、try代码块之前就结束了方法，比如报异常，return出去了

2、try代码块中，出现System.exit(0)，退出当前Java虚拟机，一旦退出Java虚拟机，任何代码都不会再执行

3、如果当一个线程在执行 try 语句块或者 catch 语句块时被打断（interrupted）或者被终止（killed），与其相对应的 finally 语句块可能不会执行

4、极端的情况是：就是在线程运行 try 语句块或者 catch 语句块时，突然死机或者断电，finally 语句块肯定不会执行了。

如果在try，catch代码块中有**return语句**，try、catch、finally的执行顺序又是怎样的呢？

```java
	public static void main(String[] args) {
 
		System.out.println("main 代码块中的执行结果为：" + myMethod());
	}		
	public static int myMethod() {
		int i = 6;
		try {
			System.out.println("try 代码块被执行！");
			// i = i/0;
			return 1;
		} catch (Exception e) {
			System.out.println("catch 代码块被执行！");
			return 2;
		} finally {
			System.out.println("finally 代码块被执行！");
            //return 3；
		}
    }
//顺序是try  finally  main1
//如果放开注释  i = i / 0 ，执行顺序为try  catch  finally main2
//如果放开注释  return 3 ，注释i = i / 0，执行顺序为 try finally main3
```
**综上所述**：finally块里面的代码也是在return前执行的。此外，如果try-finally或者catch-finally中都有return，那么finally块中的return语句将会覆盖别处的return语句，最终返回调用者那里的是finally中return的值。因为finally会把try或者catch代码块中的**返回值保留**，再来执行finally代码块中的语句，等到finally代码块执行完毕之后，在把之前**保留的返回值给返回出去**。  

```java
public static int myMethod() {
		int i = 1;
		try {
			System.out.println("try 代码块被执行！");
			return i;
            //return num();
		} finally {
			++i;
			System.out.println("finally 代码块被执行！");
			System.out.println("finally 代码块中的i = " + i);
		}
	}
//这种情况下，finally打印出来的是2，但是 try 中return 的值还是1
//但还有一种特殊情况，就是try return的时候是调用一个方法的，我们可以这样子理解
//return num() 等同于 int sum = num(); return sum; 执行顺序是 try  num finally main
```

# 四、ArrayList、LinkedList、Vector的区别是啥

![](https://note.youdao.com/yws/api/personal/file/F67FABE2E7C641BCA5E1EF9A8DE789F6?method=download&shareKey=cf883d57b6e9041be2192213170082fc)

- **ArrayList** 就是**动态数组**（线性表），是Array的复杂版本，动态的增加和减少元素。当更多的元素加入到ArrayList中时，其大小将会动态地增长。它的元素可以通过get/set方法直接访问，因为ArrayList本质上是一个数组。

  但ArrayList随机get/set快  add/remove慢，ArrayList里面维护了一个数组，add时：检查数组的大小是否足够，如果不够将创建一个尺寸扩大一倍新数组，将原数组的数据拷贝到新数组中，原数组丢弃，这里会很慢 
  
- **Vector** 和ArrayList类似, 区别在于Vector是同步类(synchronized).因此,开销就比ArrayList要大。

- **LinkedList** 是一个**链表**，在添加和删除元素时具有比ArrayList更好的性能.但在get与set方面弱于ArrayList.当然,这些对比都是指数据量很大或者操作很频繁的情况下的对比。它还实现了 **Queue** 接口,该接口比List提供了更多的方法,包括 offer(),peek(),poll()等.

> 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
>
> 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据，如果是在末位添加元素，则两者差别不大；如果是在首位添加，ArrayList就要将所有元素后移一位。



# 五、分布式消息队列，挂掉怎么处理没推送成功的信息信息







# 六、微服务通讯协议 、序列化





# 七、为什么公司项目用dubbo，不用springclould

dubbo和spring cloud的优缺点吗，说完后结合自己的场景说明就可以了

|          | Spring Cloud                        | dubbo                                                        |
| -------- | ----------------------------------- | ------------------------------------------------------------ |
| 框架原理 | 基于Http协议+rest接口调用远程过程的 | 使用Netty这样的NIO框架，是基于TCP协议传输的，配合以Hession序列化完成RPC |
|          |                                     |                                                              |
|          |                                     |                                                              |



# 八、hashmap冲突的解决方法以及原理分析

HashMap 采用一种所谓的“Hash 算法”来决定每个元素的存储位置。当程序执行 
map.put(String,Obect)方法 时，系统将调用String的 hashCode() 方法得到其 hashCode 值——每个 
Java 对象都有 hashCode() 方法，都可通过该方法获得它的 hashCode 值。得到这个对象的 hashCode 
值之后，系统会根据该 hashCode 值来决定该元素的存储位置。

[hashmap冲突的解决方法以及原理分析](https://www.cnblogs.com/peizhe123/p/5790252.html)

# 九、hashmap的死循环





# 十、框架问题

## 10.1、hibernate跟Mybatis 的区别

**来源：**Hibernate 是当前最流行的O/R mapping框架，它出身于sf.net，现在已经成为Jboss的一部分。 Mybatis 是另外一种优秀的O/R mapping框架。目前属于apache的一个子项目。

区别：

|            | Hibernate                                                    | Mybatis                                                      |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
|            | POJO 到数据库表的映射关系                                    | POJO 与SQL之间的映射关系                                     |
| 开发速度   | 简单CRUD，效率较快，基本sql封装好                            | 复杂语句校多                                                 |
| 开发工作量 | 良好的映射机制，无需关心SQL生成与结果映射                    | 高级查询，手动编写SQL和ResultMap，需要维护SQL和结果映射      |
| sql优化    | 所有字段查出，性能消耗；可用HQL，破坏开发简洁性              | 按需求指定查询的字段                                         |
| 对象管理   | 关注对象的状态，不考虑SQL语句的执行                          | 需要对对象进行详细的管理                                     |
| 缓存       | Hibernate的二级缓存配置在SessionFactory生成的配置文件中进行详细配置，然后再在具体的表-对象映射中配置是那种缓存。 | MyBatis的二级缓存配置都是在每个具体的表-对象映射中进行详细配置，这样针对不同的表可以自定义不同的缓存机制。并且Mybatis可以在命名空间中共享相同的缓存配置和实例，通过Cache-ref来实现。 |
| 移植性     | 数据库无关性好                                               | 需要根据不同数据库，设置不同的SQL语句                        |





# 十一、数据库

## 11.1、乐观锁和悲观锁的实现

**悲观锁**

总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁（**共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程**）。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。Java中`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现。

**乐观锁**

总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制和CAS算法实现。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**，像数据库提供的类似于**write_condition机制**，其实都是提供的乐观锁。在Java中`java.util.concurrent.atomic`包下面的原子变量类就是使用了乐观锁的一种实现方式**CAS**实现的。

**两种锁的使用场景**

从上面对两种锁的介绍，我们知道两种锁各有优缺点，不可认为一种好于另一种，像**乐观锁适用于写比较少的情况下（多读场景）**，即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量。但如果是多写的情况，一般会经常产生冲突，这就会导致上层应用会不断的进行retry，这样反倒是降低了性能，所以**一般多写的场景下用悲观锁就比较合适。**

**乐观锁常见的两种实现方式**

> **乐观锁一般会使用版本号机制或CAS算法实现。**

version方式：

> 一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。
>

CAS操作方式：

> 即**compare and swap（比较与交换）**，是一种有名的**无锁算法**。无锁编程，即不使用锁的情况下实现多线程之间的变量同步，也就是在没有线程被阻塞的情况下实现变量的同步，所以也叫非阻塞同步（Non-blocking Synchronization）。**CAS算法**涉及到三个操作数
>
> - 需要读写的内存值 V
> - 进行比较的值 A
> - 拟写入的新值 B
>
> 当且仅当 V 的值等于 A时，CAS通过原子方式用新值B来更新V的值，否则不会执行任何操作（比较和替换是一个原子操作）。一般情况下是一个**自旋操作**，即**不断的重试**。

## 11.2、什么情况索引不会命中，会造成全表扫描

## 11.3、LEFT JOIN关联表中ON,WHERE后面跟条件的区别

> A left B join on   and   where
>
> **join on and 不会过滤结果记录条数，只会根据and后的条件是否显示 B表的记录，A表的记录一定会显示不管and 后面的是A.id=1还是B.id=1,都显示出A表中所有的记录，并关联显示B中对应A表中id为1的记录或者B表中id为1的记录。**
>
> 

1、on条件是在生成临时表时使用的条件，它不管on中的条件是否为真，都会返回左边表中的记录。

2、where条件是在临时表生成好后，再对临时表进行过滤的条件。这时已经没有left join的含义（必须返回左边表的记录）了，条件不为真的就全部过滤掉。

## 11.4、Mysql的B+树索引和Hash索引之间有什么优缺点？

B+树索引：是一种平衡查找树。B+树索引并不能根据键值找到具体的行数据，B+树索引只能找到行数据所在的页，然后通过把页读到内存，再在内存中查找到行数据。B+树索引也是最常用的最为频繁使用的索引。

> B+树，每一个父节点的元素都出现在子节点仲，是子节点的最大（或最小）元素；
>
> 根节点的最大元素，等同于整个B+树的最大元素，始终保持最大元素在根节点中；
>
> 由于父节点的元素都出现在子节点中，因此所有叶子节点包含了全量元素信息
>
> 每个叶子节点都带有指向下一个节点的指针，形成了一个有序链表

Hash索引：底层的数据结构就是哈希表，

# 十二、Java基础

## 12.1、面向对象编程有三大特性有哪些?

面向对象编程有三大特性：封装、继承、多态。

- 封装隐藏了类的内部实现机制，可以在不影响使用的情况下改变类的内部结构，同时也保护了数据。对外界而已它的内部细节是隐藏的，暴露给外界的只是它的访问方法。

- 继承是为了重用父类代码。两个类若存在IS-A的关系就可以使用继承。
- 多态

## 12.2、反射的用途以及实现

​		**JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种<u>动态获取信息</u>以及<u>动态调用对象方法</u>的功能称为java语言的反射机制。**（**程序运行时是根据编译后的 .class 来执行的**）

> 简单来说反射就是解剖一个类，然后获取这个类中的属性和方法，前提是要获取这个类的Class对象

**如何获取 class文件对象**

1、使用类的对象获取

类的对象.getClass()方法

2、使用类的静态属性获取

类名.class

3、使用Class类的静态方法获取

Class.forName(String 类名)

[Java 反射由浅入深 | 进阶必备](https://juejin.im/post/598ea9116fb9a03c335a99a4#heading-7)

**小总结一波**：

- JVM为每个加载的`class`及`interface`创建了对应的`Class`实例来保存`class`及`interface`的所有信息；
- 获取一个`class`对应的`Class`实例后，就可以获取该`class`的所有信息；
- 通过Class实例获取`class`信息的方法称为反射（Reflection）
- JVM总是动态加载`class`，可以在运行期根据条件来控制加载class。

## 12.3、Java线程池解析

**线程池概念**

- 帮助我们管理线程，避免增加创建线程和销毁线程的资源损耗

- 提高响应速度
- 重复利用线程

**线程池的创建**

```java
public ThreadPoolExecutor(
    int corePoolSize, 
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler) 
```

- **corePoolSize：** 线程池核心线程数最大值

- **maximumPoolSize：** 线程池最大线程数大小

- **keepAliveTime：** 线程池中非核心线程空闲的存活时间大小

- **unit：** 线程空闲存活时间单位

- **workQueue：** 存放任务的阻塞队列

- **threadFactory：** 用于设置创建线程的工厂，可以给创建的线程设置有意义的名字，可方便排查问题。

- **handler：**  线城池的饱和策略事件，主要有四种类型。

**线程池的执行流程**



**线程池的工作队列**

- **ArrayBlockingQueue**

  ArrayBlockingQueue（有界队列）是一个用数组实现的有界阻塞队列，按FIFO排序量。

- **LinkedBlockingQueue**

  LinkedBlockingQueue（可设置容量队列）基于链表结构的阻塞队列，按FIFO排序任务，容量可以选择进行设置，不设置的话，将是一个无边界的阻塞队列，最大长度为Integer.MAX_VALUE，吞吐量通常要高于ArrayBlockingQuene；<u>newFixedThreadPool线程池</u>使用了这个队列

- **DelayQueue**

  DelayQueue（延迟队列）是一个任务定时周期的延迟执行的队列。根据指定的执行时间从小到大排序，否则根据插入到队列的先后排序。<u>newScheduledThreadPool线程池</u>使用了这个队列。

- **PriorityBlockingQueue**

  PriorityBlockingQueue（优先级队列）是具有优先级的无界阻塞队列；

- **SynchronousQueue**

  SynchronousQueue（同步队列）一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene，<u>newCachedThreadPool线程池</u>使用了这个队列。

**常用的线程池**

newFixedThreadPool (固定数目线程的线程池)

newCachedThreadPool(可缓存线程的线程池)

newSingleThreadExecutor(单线程的线程池)

newScheduledThreadPool(定时及周期执行的线程池)

