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

