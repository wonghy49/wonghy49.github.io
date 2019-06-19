---
title: Java面试
date: 2017-03-07 13:25:24
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

## 11.2、什么情况索引不会命中，会造成全表扫描

# 十二、Java基础

## 12.1、面向对象编程有三大特性有哪些?

面向对象编程有三大特性：封装、继承、多态。

- 封装隐藏了类的内部实现机制，可以在不影响使用的情况下改变类的内部结构，同时也保护了数据。对外界而已它的内部细节是隐藏的，暴露给外界的只是它的访问方法。

- 继承是为了重用父类代码。两个类若存在IS-A的关系就可以使用继承。
- 多态