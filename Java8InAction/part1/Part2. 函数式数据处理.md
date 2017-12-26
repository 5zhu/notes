# 流的概念，与集合的异同

## 流是什么
- 流是Java API的新成员，它允许你以声明性方式处理数据集合（通过查询语句来表达，而不
是临时编写一个实现）。可以把它们看成遍历数据集的高级迭代器。

- 作者在第一章中举了一个实际的例子：Unix的cat命令会把两个文件连接起来创建一个流，tr会转换流中的
字符，sort会对流中的行进行排序，而tail -3则给出流的最后三行。Unix命令行允许这些程
序通过管道（|）连接在一起，比如

    cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3

   会（假设file1和file2中每行都只有一个词）先把字母转换成小写字母，然后打印出按照词典
排序出现在最后的三个单词。我们说sort把一个行流①作为输入，产生了另一个行流（进行排
序）作为输出。
![](https://s1.ax2x.com/2017/12/26/YM8eu.png)

- Stream API就是基于这种思想设计的。
- 使用流的好处：
  下面两段代码都是用来返回低热量的菜肴名称的，
并按照卡路里排序。
- Java7：用了一个“垃圾变量”lowCaloricDishes。它唯一的作用就是作为一次
性的中间容器。
 ``` java 
     List<Dish> lowCaloricDishes = new ArrayList<>(); 
        for(Dish d: menu){ 
        if(d.getCalories() < 400){ 
        lowCaloricDishes.add(d); 
        } 
      } 
        Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
            public int compare(Dish d1, Dish d2){ 
            return Integer.compare(d1.getCalories(), d2.getCalories()); 
            } 
     }); 
      List<String> lowCaloricDishesName = new ArrayList<>(); 
      for(Dish d: lowCaloricDishes){ 
       lowCaloricDishesName.add(d.getName()); 
      } 
``` 
- Java8:
``` java 
    import static java.util.Comparator.comparing; 
    import static java.util.stream.Collectors.toList; 
    List<String> lowCaloricDishesName = 
     menu.stream()  //从menu获得流（菜肴列表）
     .filter(d -> d.getCalories() < 400) //首先选出低热量的菜肴
     .sorted(comparing(Dish::getCalories)) //排序
     .map(Dish::getName) //获取菜名
     .collect(toList());
 ``` 
-将流操作链接起来构成流的流水线
![](https://s1.ax2x.com/2017/12/26/YO3ei.png)

-那么，流到底是什么呢？简短的定义就是“**从支持数据处理操作的源生成的元素序列**”。

### 流只能消费一次
> 迭代器类似，流只能遍历一次。遍历完之后，我们就说这个流已经被消费掉了。你可以从原始数据源那里再获得一个新的流来重新遍历一遍

    List<String> title = Arrays.asList("Java8", "In", "Action"); 
    Stream<String> s = title.stream(); 
     s.forEach(System.out::println); //打印标题中的每个单词
     s.forEach(System.out::println);
        //java.lang.IllegalStateException:流已被操作或关闭

### 外部迭代与内部迭代
> 使用Collection接口需要用户去做迭代（比如用for-each），这称为外部迭代。 相反，Streams库使用内部迭代。

### 流的操作
- **中间操作** 和  **终端操作**
    

     List<String> names = menu.stream() 
     .filter(d -> d.getCalories() > 300)
     .map(Dish::getName) 
     .limit(3) 
     .collect(toList());

- filter、map和limit可以连成一条流水线  （中间操作）
- collect触发流水线执行并关闭它  （终端操作）

> 总而言之，流的使用一般包括三件事：
  一个数据源（如集合）来执行一个查询；
  一个中间操作链，形成一条流的流水线；
  一个终端操作，执行流水线，并能生成结果。


# 使用流实现达复杂数据处理查询

## 筛选和切片
- filter方法（你现在应该很熟悉了）。该操作会接受一个谓词（一个返回
boolean的函数）作为参数，并返回一个包括所有符合谓词的元素的流
        
        List<Dish> vegetarianMenu = menu.stream() 
        .filter(Dish::isVegetarian) 
        .collect(toList());

- 流还支持一个叫作**distinct**的方法，它会返回一个元素各异（根据流所生成元素的
hashCode和equals方法实现）的流。也就是去重。

- 流支持**limit(n)**方法，该方法会返回一个不超过给定长度的流。所需的长度作为参数传递
给limit。如果流是有序的，则最多会返回前n个元素。

-流还支持**skip(n)**方法，返回一个扔掉了前n个元素的流。如果流中元素不足n个，则返回一
个空流。

# 收集器

# 并行数据处理
