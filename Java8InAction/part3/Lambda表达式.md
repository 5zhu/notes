# Lambda表达式
> 可以把Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式：它没有名称，但它
有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。

先前：
    
     Comparator<Apple> byWeight = new Comparator<Apple>() { 
      public int compare(Apple a1, Apple a2){ 
      return a1.getWeight().compareTo(a2.getWeight()); 
       } 
     }; 
     
之后（用了Lambda表达式）：
    
     Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()); 


Lambda表达式有三个部分
- 参数列表——这里它采用了Comparator中compare方法的参数，两个Apple。
- 箭头——箭头   ->  把参数列表与Lambda主体分隔开。
- Lambda主体——比较两个Apple的重量。表达式就是Lambda的返回值了。
 
Lambda
的基本语法是
(parameters) -> expression 
或（请注意语句的花括号）
(parameters) -> { statements; }

- 一些Lambda的例子和使用案例

| 使用案例        | Lambda示例       |  
| -------------   |:-------------:|  
| 布尔表达式      | (List<String> list) -> list.isEmpty()  |  
| 创建对象        | () -> new Apple(10)       |   
| 消费一个对象    | (Apple a) -> { System.out.println(a.getWeight()); }       |  
|从一个对象中选择/抽取|(String s) -> s.length() |
|组合两个值|(int a, int b) -> a * b |

- Lambda及其等效方法引用的例子

| Lambda                   | 等效的方法引用 |
|------------------------  |:---------------:|
|(Apple a) -> a.getWeight()| Apple::getWeight| 
|() -> Thread.currentThread().dumpStack()| Thread.currentThread()::dumpStack |
|(str, i) -> str.substring(i) |String::substring |
|(String s) -> System.out.println(s) |System.out::println| 

## 如何构造方法引用
- (1) 指向静态方法的方法引用（例如Integer的parseInt方法，写作Integer::parseInt）
- 指 向 任意类型实例方法 的方法引用（例如 String 的 length 方法，写作
String::length）
-(3) 指向现有对象的实例方法的方法引用（假设你有一个局部变量expensiveTransaction
用于存放Transaction类型的对象，它支持实例方法getValue，那么你就可以写expensiveTransaction::getValue）

![](https://s1.ax2x.com/2017/12/28/0MboG.png)
