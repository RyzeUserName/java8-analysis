# 浅析java8 Stream

Classes to support functional-style operations on streams of elements, such as map-reduce transformations on collections.

## 1.代码

```java
List<String> strList = new ArrayList<>();
strList.add("1");
strList.add("2");
strList.add("3");
strList.stream().filter(Objects::nonNull).map(Integer::valueOf).forEach(System.out::println);
```

## 2.解析

以上代码一个简单的 stream 的应用, 分成  stream ->filter+ map->forEach，stream一次消费流 

### 1.Stream 

![image-20200624180454421](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624180454421.png)

![image-20200624181806962](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624181806962.png)

​						最终返回的对象是 ReferencePipeline.Head

### 2.filter

无状态 诸如map或者filter会从输入流中获取每一个元素，并且在输出流中得到一个结果，不依赖于上个操作，这些操作没有内部状态，称为无状态操作。

![image-20200624182513237](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624182513237.png)

### 3.skip

有状态  但是像reduce、sum、max这些操作都需要内部状态来累计计算结果，依赖上个操作，所以称为有状态操作。

![image-20200624182735540](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624182735540.png)

![image-20200624183933679](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624183933679.png)

### 4.forEach

![image-20200624183609870](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624183609870.png)

综上：我们看到一个stream 的流水线 包含 head+ n * StatelessOp /  n * StatefulOp... +  evaluate

下面我们详细查看 ReferencePipeline 详细结构是这样的

![image-20200624182220973](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624182220973.png)

​						看其实内部就是个一个双端链表

![image-20200627153115540](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200627153115540.png)

​							是怎么组装起来的？在哪里组装的？

​							![image-20200629200911086](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200629200911086.png)

​							也就是消费流操作前返回的Stream是个双端链表，那么在  evaluate 方法里，做了什么操作呢？

​							![image-20200627162137802](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200627162137802.png)

​							接着往下看

​							![image-20200627163046339](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200627163046339.png)

​							查看里面的 wrapSink方法：

![image-20200627163248255](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200627163248255.png)

​						也就是从foreach 之前的中间操作操作往前遍历，opWrapSink 方法顾名思义  包装成Sink，从这个双端链表末尾往前遍历，回看之前的代码

​						每个中间操作都覆盖了opWrapSink 方法返回	Sink.ChainedReference  

​						![image-20200629205312263](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200629205312263.png)

​						就是短路操作的要实现的方法：cancellationRequested()							

​						也就是将一个双端链表 ReferencePipeline 变成一个责任链 Sink.ChainedReference  ,为什么包装成sink？其实就是为了把各种中间操作包装成相同						的无差异的操作，Sink的方法，begin accept end   cancellationRequested( 是否不再接受数据)

​						往回看AbstractPipeline # copyInto  

​						![image-20200629212348900](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200629212348900.png)

​							注：短路操作  anyMatch() allMatch() noneMatch() findFirst() findAny()  ，短路操作是指不用处理全部元素就可以返回结果

### 						5.并行

​						![image-20200702202037232](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200702202037232.png)

​						其实都是创建了 ForkJoinTask  源自框架    ForkJoinPool