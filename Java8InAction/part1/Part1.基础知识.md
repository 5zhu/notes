# 本章内容
## Java怎么又变了

  Java 8所做的改变，在许多方面比Java历史上任何一次改变深远。而且好
消息是，这些改变会让你编起程来更容易，用不着再写类似下面这种啰嗦的程序了（对inventory
中的苹果按照重量进行排序）：
``` java  
    Collections.sort(inventory, new Comparator<Apple>() { 
      public int compare(Apple a1, Apple a2){ 
        return a1.getWeight().compareTo(a2.getWeight()); 
     } 
    });
```
在Java 8里面，你可以编写更为简洁的代码，这些代码读起来更接近问题的描述：
     
     inventory.sort(comparing(Apple::getWeight));


## Java中的函数
Java 8中新增了函数——值的一种新形式。 借助于Stream API，Java 8可以进行多核处理器上的并行编程。

第一个例子中介绍了Java8的新功能方法引用,比方说，你想要筛选一个目录中的所有隐藏文件。
    
    File[] hiddenFiles = new File(".").listFiles(new FileFilter() { 
         public boolean accept(File file) { 
         return file.isHidden(); 
        } 
    }); 

在Java 8里，你可以把代码重写成这个样子：哇！酷不酷？你已经有了函数isHidden，因此只需用Java 8的方法引用::语法（即“把这
个方法作为值”）将其传给listFiles方法；
     
     File[] hiddenFiles = new File(".").listFiles(File::isHidden);

## 流
 Java8之前操作集合
比方说，你需要
从一个列表中筛选金额较高的交易，然后按货币分组。

    Map<Currency, List<Transaction>>     transactionsByCurrencies = 
     new HashMap<>(); 
        for (Transaction transaction : transactions) { 
          if(transaction.getPrice() > 1000){ 
           Currency currency = transaction.getCurrency(); 
            List<Transaction> transactionsForCurrency = 
                     transactionsByCurrencies.get(currency); 
        if (transactionsForCurrency == null) { 
        transactionsForCurrency = new ArrayList<>(); 
        transactionsByCurrencies.put(currency, 
         transactionsForCurrency); 
        } 
     transactionsForCurrency.add(transaction); 
     } 
    } 
    
 有了Stream API

    import static java.util.stream.Collectors.toList; 
      Map<Currency, List<Transaction>> transactionsByCurrencies = 
        transactions.stream() 
         .filter((Transaction t) -> t.getPrice() > 1000) 
            .collect(groupingBy(Transaction::getCurrency)   );
## 默认方法
接口中可以有方法的实现了，在Java 8里，你现在可以直接对List调用sort方法。它是用Java 8 List接口中如下所
示的默认方法实现的，它会调用Collections.sort静态方法：
    
    default void sort(Comparator<? super E> c) { 
     Collections.sort(this, c); 
    } 

