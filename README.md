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

​							其实也就在  evaluate 方法里，打开实现：

​							![image-20200627162137802](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200627162137802.png)

​							接着往下看

​							![image-20200627163046339](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200627163046339.png)

​							查看 wrapSink方法：

![image-20200627163248255](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200627163248255.png)

​						也就是从foreach 往前遍历，opWrapSink 方法顾名思义  包装成Sink，AbstractPipeline # opWrapSink 其实我们在前面都看过，每个阶段操作

​				都要实现了 opWrapSink返回 Sink

​						![image-20200627214439966](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200627214439966.png)

​						几乎都是返回Sink.ChainedReference（ChainedDouble,ChainedInt,ChainedLong）

​						

​					回看之前的代码，返回 

 