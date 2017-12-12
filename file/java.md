
<!-- vim-markdown-toc GFM -->

* [final](#final)
* [java并发基础](#java并发基础)
* [线程安全的HashMap](#线程安全的hashmap)
* [ConcurrentHashMap原理](#concurrenthashmap原理)

<!-- vim-markdown-toc -->

### final
```
final 关键字可用于修饰类、方法、变量。是Java的一个保留关键字。一旦将引用声明作final，就不能改变这个引用，否则会报编译错误。
```

**final 可修饰本地变量、成员变量。**
```
（本地变量： 在方法中或代码块中的变量）
final变量是只读的。
```

**final 可声明方法**
```
方法前面加上final表示该方法不可被子类的方法重写。一般方法功能比较完善了才会加final。
final方法比非final方法快，因为在编译时就已经静态绑定了，不需要在运行时再动态绑定。
```

**final 可修饰类**
```
final类通常功能完整，不能被继承。Java中很多类是final的，比如String，Integer以及其他包装类。
```

**final 关键字的好处**
```
1. final关键字提高了性能。JVM和Java应用都会缓存final变量。
2. final变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销。
3. 使用final关键字，JVM会对方法、变量、类进行优化。
```

**不可变类**
```
创建不可变类要使用final关键字。不可变类是指它的对象一旦被创建了就不能更改。String是不可变类的代表。
```

**关于final的重要知识点**
```
1. final关键字用于成员变量、本地变量、方法、类。
2. final成员变量必须在声明的时候初始化或在构造器中初始化，否则会报编译错误。
3. 不能对final变量再次赋值。
4. 本地变量必须在声明时赋值。
5. 在匿名类中所有变量都必须是final变量。
6. final方法不能重写。
7. final类不能被继承。
8. final关键字不同于finally关键字，后者用于异常处理。
9. final关键字容易与finalize()方法搞混，后者是在Object类中定义的方法，是在垃圾回收之前被JVM调用的方法。
10. 接口中声明的所有变量本身是final的。
11. final和abstract这两个关键字是反相关的，final类不可能是abstract的。
12. final方法在编译阶段绑定，称为静态绑定。
13. 没有在声明时初始化final变量的称为空白final变量(blank final variable)，它们必须在构造器中初始化，或者调用this()初始化。不这么做的话，编译器会报错“final变量（变量名）需要进行初始化”。
14. 将类、方法、变量声明为final能够提高性能，这样JVM就有机会进行估计，然后优化。
15. 按照Java代码惯例，final变量就是常量，而且通常常量名要大写。
```

对于集合对象声明为final指的是引用不能被更改，但是可以向其中增加、删除或者改变内容。



### java并发基础
```
当一个对象或变量可以被多个线程共享的时候，就有可能使得程序的逻辑出现问题。
```

**Java内存模型**
```
在java memory model中。Memory分为两类，main memory和working memory。
main memory为所有线程共享，working memory中存放的是线程所需要的变量的拷贝。
（线程要对main memory中的内容进行操作的话，首先需要拷贝到自己的working memory。
一般为了速度，working memory一般是在cpu的cache中的）
volatile的变量在被操作的时候不会产生working Memory的拷贝，而是直接操作main memory。
当然volatile虽然解决了变量的可见性问题，但没有解决变量操作的原子性的问题。还需要synchronized或者CAS相关操作配合进行。
```

**可见性**
```
假设一个对象中有一个变量i，那么i是保存在main memory中的。
当某一个线程要操作i的时候，首先需要从main memory中将i加载到这个线程的working memory中。
这个时候working memory中就有了一个i的拷贝，这个时候此线程对i的修改都在其working memory中，直到其将i从working memory写回到main memory中，新的i的值才能被其他线程所读取。
从某个意义上说，可见性保证了各个线程的working memory的数据的一致性。

可见性遵循以下规则：
1. 当一个线程运行结束时，所有写的变量会被flush回main memory中。
2. 当一个线程第一次读取某个变量的时候，会从main memory中读取最新的。
3. volatile的变量会被立刻写到main memory中。在jsr133中，对volatil的语义进行增强。
4. 当一个线程释放锁后，所有的变量的变化都会flush到main memory中。
然后一个使用了这个相同的同步锁的进程，将会重新加载所有的使用到的变量，这样就保证了可见性。
```

**原子性**
```
当某个线程修改i的值的时候，从取出i到将新的i的值写给i之间不能有其他线程对i进行任何操作。
也就是说保证某个线程对i的操作是原子性的，这样就可以避免数据脏读。
通过锁机制或者CAS(Compare And Set需要硬件CPU的支持)操作可以保证操作的原子性。
```

**有序性**
```
假设在main memory中存在两个变量i和j，初始值都为0，在某个线程A的代码中依次对i和j进行自增操作(i,j的操作不互相依赖)
i++;
j++;
所以i,j修改操作的顺序可能会被重新排序。
那么修改后的i,j写到main memory中的时候，顺序可能就不是按照i,j的顺序了，这就是所谓的reordering，
在单线程的情况下，是按照as-if-serial语义的，即使在实际运行过程中，i,j的自增可能被重新排序，
当然计算机也不能帮你乱排序，存在上下逻辑关联的运行顺序肯定还是不会变的。
但是在多线程环境下，问题就不一样了。
这就是reordering产生的不好的后果，所以我们在某些时候为了避免这样的问题需要一些必要的策略，以保证多个线程一起工作时也存在一定的次序。

JMM提供了happens-before的排序策略，这样我们可以得到多线程环境下的as-if-serial语义。
```



### 线程安全的HashMap

```
一个hashMap在运行的时候只有读操作，就不会存在线程安全问题。

但当涉及到同时有改变也有读的时候，就要考虑线程安全问题了。
在不考虑性能问题时，我们可以用HashTable,Collections.synchronizedMap(hashMap)。
这两种方式基本都是对整个hash表结构做锁定操作的，这样在锁表的期间，别的线程就需要等待了，性能低。
```


### ConcurrentHashMap原理
**描述**
```
ConcurrentHashMap是在Java1.5作为HashTable的替代选择新引入的，是concurrent包的重要成员。
在Java1.5之前，如果想要实现一个可以在多线程和并发的程序中安全使用的Map，只能在HashTable和synchronized Map中选择，因为HashMap并不是线程安全的。
ConcurrentHashMap不仅是线程安全的，而且比HashTable和synchronized Map的性能要好。
相对HashTable和synchronized Map锁住了整个Map，ConcurrentHashMap只锁住部分Map。
ConCurrentHashMap允许并发的读操作，同时通过同步锁在写操作时保持数据完整性。
```
**适用场景**
```
1. 读数量超过写数量。
因为当写者数量大于等于读者时，ConcurrentHashMap的性能是低于HashTable和synchronized Map的。
由于当锁住了整个Map时，读操作要等待对同一部分执行写操作的线程结束。
ConcurrentHashMap适用于做cache，在程序启动时初始化，之后可以被多个请求线程访问。
ConcurrentHashMap是HashTable一个很好的替代，但是ConcurrentHashMap比HashTable的同步性弱。
```

**java中concurrentHashMap的实现**

```
数据结构ConcurrentHashMap的目标是实现支持高并发、高吞吐量的线程安全的HashMap。
当然不能直接对整个HashTable枷锁，所以在ConcurrentHashMap中，数据的组织结构和HashMap有所区别。

一个ConcurrentHashMap由多个segment组成。
每一个segment都包含了一个HashEntry数组的HashTable，每一个segment包含了对自己的HashTable的操作。
比如get,put,replace等操作，这些操作发生的时候，对自己的HashTable进行锁定，由于每一个segment写操作只锁定自己的HashTable，所以可能存在多个线程同时写的情况，性能比只有一个HashTable锁定的情况好。
```

***在jdk6, jdk7, jdk8中的实现都不同***
```

concurrentHashMap引入了分割，并提供了HashTable支持的所有的功能。
在concurrentHashMap中，支持多线程对Map做读操作，并且不需要任何blocking。
这得益于concurrentHashMap将Map分割成了不同的部分，在执行更新操作时只锁住了一部分。
根据默认的并发级别(concurrency level),Map被分割成16部分，并且由不同的锁控制。
即同时最多可以有16个写线程操作Map，性能的提升显而易见。
但由于一些更新操作，如put(),remove(),putAll(),clear()只锁住操作的部分，所以在检索操作不能保证返回的是最新的结果。

另外

在迭代遍历concurrentHashMap时，keySet返回的iterator是弱一致和failsafe的，可能不会返回某些最近的改变。
并且在遍历过程中，如果已经遍历的数组上的内容变化了，不会抛出ConcurrentModificationException的异常。
concurrentHashMap的并发级别是16，但可以在创建concurrentHashMap时通过构造函数改变。
并发级别代表着并发执行更新操作的数目，所以如果只有很少的线程会更新Map，那么建议设置一个低的并发级别。
另外，concurrentHashMap还使用了ReentrantLock来对segments加锁。
```


**ConcurrentHashMap中的putIfAbsent()**
```
synchronized(map){
    if(map.get(key) == null){
        return map.put(key, value);
    } else {
        return map.get(key);
    }
  }


上面这段代码在HashMap和HashTable中是好用的，但在ConcurrentHashMap中是有出错的风险的。
因为ConcurrentHashMap在put操作时并没有对整个Map加锁，所以一个线程正在put(k,v)时，另一个线程调用
get(k)会得到null，这就会造成一个线程put的值会被另一个线程put的值所覆盖。
当把代码封装到synchronized代码块中，这样虽然线程安全了，但会使你的代码变成单线程。
ConcurrentHashMap提供的putIfAbsent(key, value)方法原子性的实现了同样的功能，同事避免了上面的线程竞争的风险。

```

** ConcurrentHashMap总结 **
```
1. ConcurrentHashMap允许并发的读和线程安全的更新操作；
2. 在执行写操作时，ConcurrentHashMap只锁住部分的Map；
3. 并发的更新是通过内部根据并发级别将Map分割成小部分实现的；
4. 高的并发级别会造成时间和空间的浪费，低的并发级别在写多线程时会引起线程间的竞争；
5. ConcurrentHashMap的所有操作都是线程安全；
6. ConcurrentHashMap返回的迭代器是弱一致性，fail-safe并且不会抛出ConcurrentModificationException异常；
7. ConcurrentHashMap不允许null的键值；
8. 可以使用ConcurrentHashMap代替HashMap，但ConcurrentHashMap不会锁住整个Map
```
