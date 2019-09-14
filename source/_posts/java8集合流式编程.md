---
title: java8集合流式编程
date: 2018-07-12 11:43:08
tags: 
categories: java8
---
#### java8集合流式编程

##### 获取流对象

- 对于集合来说,直接通过stream()方法即可获取流对象

```
List<Person> list = new ArrayList<Person>(); 
Stream<Person> stream = list.stream();
```

- 对于数组来说,通过Arrays类提供的静态函数stream()获取数组的流对象

```
String[] names = {"chaimm","peter","john"};
Stream<String> stream = Arrays.stream(names);
```

- 直接将几个普通的数值变成流对象

```
Stream<String> stream = Stream.of("chaimm","peter","john");
```



##### 中间操作：

- filter()： 对元素进行过滤

- sorted()：对元素排序

- map()：用于操作单个集合,比如List<String>

- distinct()：去除重复的元素

- limit(): 截取流的前N个元素

- skip(): 跳过流的前N个元素

- flatMap():用于操集合嵌套集合List<List<String>>

  ```java
  String[] s={"hello","world"};
  List<String> collect = Arrays.asList(s).stream().flatMap(str -> Arrays.stream(str.split(""))).distinct().collect(Collectors.toList());
  System.out.println(collect);
  输出
  collect---------[h, e, l, o, w, r, d]
  ```

##### 最终操作：

- forEach()：遍历每个元素。
- reduce()：把Stream 元素组合起来。例如，字符串拼接，数值的 sum，min，max ，average 都是特殊的 reduce。
- collect()：将集合元素集合起来,具体返回什么看里面参数Collectors的操作。
- min()：找到最小值。
- max()：找到最大值。



**Collectors用于对新集合的操作,常用方法asList()返回新的集合,joining()拼接起来返回字符串,joining("_")用短横线拼接起来返回字符串**

用于对集合的操作,

collection.stream.中间操作.中间操作...最终操作

如:

```Java
ArrayList<Integer> integers = Lists.newArrayList();
integers.add(5);
integers.add(10);
integers.add(20);
integers.add(19);
String collect = integers.stream().filter(i -> i > 10).map(i -> i + "").collect(Collectors.joining());
System.out.println(collect);2019
当->只有一条语句不需要返回值,当多条语句,需要{}且要有返回值
```

##### Collectors.toMap用法

Collectors.toMap(v1,v2);v1方法返回key,v2方法返回value

Collectors.toMap(v1,v2,v);v1方法返回key,v2方法返回value,v方法返回当key值相同时,value的处理,参数为(oldkey,newkey)

例如:

```Java
collect.(Collectors.toMap(t -> t[0], t -> Lists.newArrayList(t[1]),
                    (List<String> oldList, List<String> newList) -> {
                        oldList.addAll(newList);
                        return oldList;
                    })
t为流中的一个元素,key值为:t[0],value值为:t[1]转为集合,当key值相同,value值的处理,将新集合的元素全部添加到旧集合中,再将旧集合返回.                    
```

例:

```Java 
//颜色=[银], 尺寸=[小号, 中号, 大号]
String s = "颜色:银;尺寸:小号;颜色:银;尺寸:中号;颜色:银;尺寸:大号";
List<String> collect1 = Arrays.stream(s.split(";")).distinct().collect(Collectors.toList());
System.out.println(collect1);//[颜色:银, 尺寸:小号, 尺寸:中号, 尺寸:大号]
List<String[]> collect2 = collect1.stream().map(t -> t.split(":")).collect(Collectors.toList());
System.out.println(collect2);//[[颜色,银],[尺寸,小号],[尺寸,中号],[尺寸,大号]]
Map<String, List<String>> collect = collect2.stream().collect(Collectors.toMap(t -> t[0], t -> {
    List<String> list = new ArrayList<>();
    list.add(t[1]);
    return list;
},(List<String> oldList,List<String> newList)->{
    oldList.addAll(newList);
    return oldList;
}));
System.out.println(collect);//{颜色=[银], 尺寸=[小号, 中号, 大号]}

--------------------------------------------------------------------
链式处理

Map<String, List<String>> collect3 = Arrays.stream(s.split(";")).distinct().map(t -> t.split(":")).collect(Collectors.toMap(t -> t[0], t -> {
            List<String> list = new ArrayList<>();
            list.add(t[1]);
            return list;
        }, (List<String> oldList, List<String> newList) -> {
            oldList.addAll(newList);
            return oldList;
        }));
   
  System.out.println(collect3);//{颜色=[银], 尺寸=[小号, 中号, 大号]}  
    
```