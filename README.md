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

​						最终返回的对象是 ReferencePipeline.Head，查看下其实现是这样的：

![image-20200624182220973](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624182220973.png)

### 2.filter

无状态 诸如map或者filter会从输入流中获取每一个元素，并且在输出流中得到一个结果，不依赖于上个操作，这些操作没有内部状态，称为无状态操作。

![image-20200624182513237](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624182513237.png)

### 3.skip

有状态  但是像reduce、sum、max这些操作都需要内部状态来累计计算结果，依赖上个操作，所以称为有状态操作。

![image-20200624182735540](https://raw.githubusercontent.com/RyzeUserName/image-upload/master/img/image-20200624182735540.png)

### 4.forEach

根据中间状态存储的要求，其区分 “有界”（例如：limit）、“无界”（例如：sorted）
