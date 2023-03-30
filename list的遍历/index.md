# List的遍历


这篇文章提供了java中list的一些遍历方法

<!--more-->


## 一 java中Collection集合体系

![Minion](https://github.com/April-1202/April-1202.github.io/blob/master/images/collection.png)




## 二 list的常见便利方式
### 1 普通for循环
```
System.out.println("-----普通for循环-----");
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}
```

### 2 增强for循环
```
 System.out.println("-----增强for循环-----");
for (String s : list) {
    System.out.println(s);
}
```

### 3 lambda
```
System.out.println("-----lambda-----");
list.stream().forEach( x -> System.out.println(x));

list.stream().forEach(System.out::println);
        
list.forEach(x -> System.out.println(x));
```

### 4 迭代器遍历
```
System.out.println("-----迭代器遍历-----");
Iterator<String> iterator = list.iterator();
    while (iterator.hasNext()){
        System.out.println(iterator.next());
    }
```

### 5 list集合自带的迭代器遍历
```
System.out.println("-----List集合自带迭代器-----");
    ListIterator<String> listIterator = list.listIterator();
    while(listIterator.hasNext()){
        System.out.println(listIterator.next());
    }
```




