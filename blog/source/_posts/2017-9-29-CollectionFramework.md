---
title: java集合框架
date: 2017-09-29 14:00:01
categories: java集合
tags:
  - java
  - 集合
---

# 背景

网上有着很多关于java集合框架的的优秀文章，我在这个写并不是想突出自己有多么特殊，只是想将自己学到的东西归类总结一下，便于自己以后的工作和学习，不要
在重复做相同的事情；如果文章能帮助到大家，我也深感万分荣幸。

# java集合框架

本文大部分内容翻译自官方文档，如果有不对的地方欢迎大家指正。[[官方文档](https://docs.oracle.com/javase/tutorial/collections/intro/index.html)]

## 什么是集合?

集合也称为容器，是一个简单的对象，把多个元素组成一个单元。集合可以用于存储，检索,操作和通信，通常情况下，它代表一个自然组中的数据项，比如一副手牌(牌集合)，邮件文件夹(邮件集合)，或者一个通讯录(一个姓名到电话号码的映射)。如果你已经使用过java编程语言或者其他编程语言，你应该已经很熟悉集合了.

## 什么是集合框架?

Collections Framework 是一个表示和操作的统一框架，集合框架包括如下：

- **Interfaces:** 这些都是表示集合的抽象数据类型，接口允许集合完成操作，独立它们的实现细节。在面向对象语言中，接口构成体系结构  
- **Implementations:** 这些都是接口的具体实现，其实质就是可复用的数据结构  
- **Algorithms:** 这些方法可以对实现集合接口的对象进行有用的计算，例如查找和排序，这些算法是多态的，也就是说同样的方法可以用来在不同集合接口的实现中，本质上，算法是可复用的方法  

除了java集合框架，还有一些其他优秀的集合框架实例，例如C++的标准模板库(STL)和Smalltalk集合结构.从历史上来看，集合框架是相当复杂的，
有着陡峭学习曲线的声誉，我们相信java集合框架可以中断这种传统，就如同你可以自己学习这节内容一样。


## java集合框架有那些优点？

java集合框架提供了如下的好处：

- 减少编程工作量：提供了有用的数据结构和算法，集合框架能让开发更专注于自己程序的核心功能，而不去做一个底层的“管道工”，通过促进无关API之间的相互作用，java集合框架让你摆脱写适配器或转换代码连接的API。  
- 增加编程速度和质量：通过有用的算法和数据机构，java集合框架实现了高性能和高质量.不同接口的实现可以相互转换，所以程序可以通过不同接口的实现来进行转换.
.正因为你从写自己数据结构的苦差事中解放出来，你就有了更多的时间去改善程序的质量和性能。  
- 允许无关API的互操作：集合接口是API之间传递集合的一个“方言”，例如我的网络管理API提供了一个节点名称的集合，你的GUI工具包期望一个列标题的集合，即使是分开实现它们，我们的API可以无缝的互操作。  
- 省力的学习和使用新的API：许多API自然的将集合作为输入和提供他作为输出。在过去，每一个这样的API都有一个子API专门操作其集合。这些集合的子API之间缺乏一致性，所以你必须学会每一个从无到有，并且在使用他们的时候很容易犯错误。随着标准集合接口的到来，这些问题都将离我们远去。  
- 减少新API的设计：这是上个优点的另一方面，设计者和实现者在创建依赖于集合的API，不需要重复设计；它们可以用标准的集合接口来代替。  
- 促进软件的重复使用：符合标准集合接口的新数据结构本质上是可复用的，对于操作这些新数据结构的算法也是可复用的。

## java集合框架中的接口

核心集合接口封装了不同类型的集合[[官方文档](http://docs.oracle.com/javase/tutorial/collections/interfaces/index.html)]，如下图所示。这些接口允许集合对他们所表示的细节进行独立的操作。核心集合接口是java集合框架的基础。正如你在下图中看到的一样，核心集合接口形成了一个分层结构  

{% asset_img http://docs.oracle.com/javase/tutorial/figures/collections/colls-coreInterfaces.gif 效果图 %}

一个 **Set** 是一种特殊的 **Collection** ,**SortedSet** 有是一种特殊的 **Set** ,等等如是。还请注意，在分层结构种包含两种不同的树， **Map** 不是一个真正的集合  
注意：核心集合接口是泛型的。例如，如下是 **Collection** 接口的声明.

{% codeblock Collection声明 lang:java %}
public interface Collection<E> ...
{% endcodeblock %}

**<E>** 的语法告诉你这个接口是一个泛型。当你声明一个 **Collection** 接口时，你可以指定集合所包含对象的类型。指定类型允许在编译时期检查放入到集合中的对象类型是否真确，因此减少了运行时的错误。关于泛型的信息可以查看[Generics](http://docs.oracle.com/javase/tutorial/java/generics/index.html)这节内容  
当你知道了如何使用这些接口，你就知道了java集合框架中的大部分东西，下面我们学习下如何有效使用这些接口的指引路线，包括当什么时候使用那些接口，并学习这些接口的程序设计语法帮助我们学到更多。  
保持核心接口的数量管理，java平台不为每个集合类型的变体提供单独的接口（这些变体可以包括不可变的，固定大小的，并且仅追加的），相反，每个接口的修改操作被指定为可选的，给定的实现可以选择不支持所有操作。如果一个不支持的操作被调用，集合就抛出一个[ **UnsupportedOperationException** ](https://docs.oracle.com/javase/8/docs/api/java/lang/UnsupportedOperationException.html)异常。实现负责它们所支持操作的文档化。java平台通常实现所有支持的可选操作。  

如下列表描述了集合核心接口：

- **Collection** -集合分层结构的根，一个collection是已知的一组对象。**Collection** 接口是所有集合的实现，用来传递周围集合的最大范围，通常希望操作他们的最小公分母。一些集合允许重复的元素，一些则不允许。一些有序，一些则无序。java平台不提供这些接口的具体实现，但提供了更具体的子接口。例如 **Set** 和 **List**
另请参阅[Collection](http://docs.oracle.com/javase/tutorial/collections/interfaces/collection.html)部分
- **Set** -一个不包含重复元素的集合，这个接口模型的数学集抽象和用于表示组。例如包括一副手牌，课程中的学生日程安排表，或一台计算机中的进程。另请参考[Set](http://docs.oracle.com/javase/tutorial/collections/interfaces/set.html)课程
- **List** -一个有序的集合（有时叫序列）**List**可以包含相同的元素，一个List用户可以精确的插入元素到List中，并通过int索引来访问它们。如果你使用**Vector**有使用List相同的味道，可以参阅[List](http://docs.oracle.com/javase/tutorial/collections/interfaces/list.html)接口课程
- **Queue** -一个用于保存处理之前的多个元素的集合，除了基本的**Collection**操作，**Queue**还提供了额外的添加，移除和检查操作,队列通常提供元素一种FIFO（先进先出）方式，还有一种例外的优先级队列，根据提供的比较器或元素的自然顺序对元素进行排序，无论使用怎样的排序，所述队列的头将通过调用移除或查询，在一个FIFO队列中，所有新元素都被插入在队列的尾部。每个队列的实现必须指定其顺序性，请参阅[Queue](http://docs.oracle.com/javase/tutorial/collections/interfaces/queue.html)课程,Deques即可用FIFO（先进先出）也可用LIFO(后进先出)，在Deque中所有的新元素被插入，取出和移除在队列的尾部，请参阅[Deque](http://docs.oracle.com/javase/tutorial/collections/interfaces/deque.html)课程
- **Map** -一个键到值映射对象，一个Map不能包含相同的键；每一个键对应一个值，如果你使用**Hashtable**,你将有和使用Map一样的感觉，请参阅[Map](http://docs.oracle.com/javase/tutorial/collections/interfaces/map.html)课程

最后两个核心集合接口是有序版本的 **Set** 和 **Map**：

- **SortedSet** -扩展了**Set**接口，并声明了以升序进行排序的组行为，提供了几个附加的操作便于排序。用于自然有序集，如单词表和会员登记的有序集合，请参阅[SortedSet](http://docs.oracle.com/javase/tutorial/collections/interfaces/sorted-set.html)课程
- **SortedMap** -声明了以键升序进行排序的Map映射.这是一个和SortedSet类似的Map,有序映射用于自然有序键值对集合，例如字典和电话目录.请参阅[SortedMap](http://docs.oracle.com/javase/tutorial/collections/interfaces/sorted-map.html)课程

之所以没有将 **Iterator** 列入到集合框架的核心接口，个人认为其真正意义上并不是一个集合，只是提供了集合一种统一的、公共的访问方式而已，所以它应该是集合框架中的一个分支子集。

## 集合框架设计目标有哪些？

- 首先，框架必须是高性能的。基本集合（动态数组、链表、树以及哈希表）的实现是高效率的。很少需要手动编写这些“数据引擎”中的某个。其次，框架必须允许不同类型的集合以类似的方式进行工作，并且具有高度的互操作性。再次，扩展或改造集合必须易于实现。为了满足 这些目标，整个集合框架基于一套标准接口进行改造，提供了这些接口的一些可以直接使用的标准实现（例如LinkedList、HashSet和TreeSet）。作为一种选择，也可以实现自己的集合。为了方便，提供了各种特定目的的实现，并且提供了一些部分实现，从而使我们可以更容易的创建自己的集合类。最后必须添加可以将标准数组集成到集合框架中的机制。
- 算法是集合机制的另外一个重要部分。算法操作集合，并且定义为[Collections](http://docs.oracle.com/javase/7/docs/api/java/util/Collections.html)类中的静态方法。因此，所有集合都可以使用它们。每个集合类都不需要实现特定于自己的版本，算法为集合提供了标准的方式。
- 和集合框架密切相关的另一个内容是**Iterator**接口。迭代器为访问集合中的元素提供了通用、标准的方式，每次访问一个元素。因为每个集合都提供了迭代器，所以可以通过**Iterator**定义的方法访问所有集合类的元素。因此，对循环遍历集合的代码进行很小的修改，就可以将其用于遍历列表

JDK5之后的集合框架进行了一些根本性修改，这些修改显著的增强了集合框架的功能并简化了它的使用。这些修改包括增加了泛型，自动拆装箱和for-each风格的for循环。泛型从根本上改变了集合框架，添加了集合一直缺失的一个特性：类安全性。自动装箱使得使用基本类型更加容易。可以使用for-each的for循环替代Iterator的循环。

## 集合框架通用实现表

| Interfaces | Hash table Implementations | Resizable array Implementations | Tree Implementations | Linked list Implementations|Hash table + Linked list Implementations |
| :---: | :---: | :---: | :---: | :---: | :---: |
| Set | HashSet | | TreeSet | | LinkedHashSet |
| List | | [ArrayList](http://thinkdevos.net/blog/20160918/java-collections-framework-arraylist/) | | LinkedList | |
| Queue | | | | | |
| Deque | | ArrayDeque | | LinkedList | |
| Map | HashMap | | TreeMap | | LinkedHashMap |

**后记:** ***一直再用java集合框架，但对于该框架的设计思想一直不清楚，也看过框架中类的源码，却因为理解不透彻陷入在重复的学习过程，于是决定从官方最基础的文档中去理解和感悟集合框架中的核心内容，加深记忆，减少重复学习的工作,后续将继续分析集合框架中的核心类库.***

---
参考资料
[集合官方文档](http://docs.oracle.com/javase/tutorial/collections/interfaces/index.html)
