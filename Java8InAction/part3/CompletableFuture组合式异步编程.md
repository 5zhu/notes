
多任务程序设计的两面
* 分支/合并框架以及并行流是实现
并行处理的宝贵工具；它们将一个操作切分为多个子操作，在多个不同的核、CPU甚至是机器上
并行地执行这些子操作
* 与此相反，如果你的意图是实现并发，而非并行，或者你的主要目标是在同一个CPU上执
行几个松耦合的任务，充分利用CPU的核，让其足够忙碌，从而最大化程序的吞吐量，那么你
其实真正想做的是避免因为等待远程服务的返回，或者对数据库的查询，而阻塞线程的执行，
浪费宝贵的计算资源，因为这种等待的时间很可能相当长。

![](https://s1.ax2x.com/2017/12/29/0xkmp.png)

**Future接口**就是处理这种并发情况
 ``` java
ExecutorService executor = Executors.newCachedThreadPool(); 
Future<Double> future = executor.submit(new Callable<Double>() { 
 public Double call() {
 return doSomeLongComputation(); 
 }}); 
doSomethingElse(); 
try { 
 Double result = future.get(1, TimeUnit.SECONDS); 
} catch (ExecutionException ee) {
 // 计算抛出一个异常
} catch (InterruptedException ie) { 
 // 当前线程在等待过程中被中断
} catch (TimeoutException te) { 
 // 在Future对象完成之前超过已过期
} 
 ``` 
这种编程方式让你的线程可以在ExecutorService以并发方式调
用另一个线程执行耗时操作的同时，去执行一些其他的任务。接着，如果你已经运行到没有异步
操作的结果就无法继续任何有意义的工作时，可以调用它的get方法去获取操作的结果。如果操
作已经完成，该方法会立刻返回操作的结果，否则它会阻塞你的线程，直到操作完成，返回相应
的结果.

![](https://s1.ax2x.com/2017/12/29/0RBQO.png)

> 你能想象这种场景存在怎样的问题吗？如果该长时间运行的操作永远不返回了会怎样？为
了处理这种可能性，虽然Future提供了一个无需任何参数的get方法，我们还是推荐大家使用重
载版本的get方法，它接受一个超时的参数，通过它，你可以定义你的线程等待Future结果的最
长时间，而不是永无止境地等待下去。



## 使用 CompletableFuture 构建异步应用
### CompletableFuture的优势
提供了异步程序执行回调的方式，不需要像future.get()通过阻塞线程来获取异步结果或者通过isDone来检测异步线程是否完成来执行后续程序。
能够管理多个异步流程，并根据需要选择已经结束的异步流程返回结果。

### 实现一个异步API
实现最佳价格查询器应用，首先，商店
应该声明依据指定产品名称返回价格的方法：

    
     public class Shop { 
     public double getPrice(String product) { 
     // 待实现
      } 
     }
     

采用delay方法模拟这些长期运行的方法的执行
    
     public static void delay() { 
      try { 
      Thread.sleep(1000L); 
      } catch (InterruptedException e) { 
      throw new RuntimeException(e); 
      } 
     } 

在getPrice方法中引入一个模拟的延迟
   
     public double getPrice(String product) { 
      return calculatePrice(product); 
     } 
     private double calculatePrice(String product) { 
      delay(); 
      return random.nextDouble() * product.charAt(0) + product.charAt(1); 
     } 
    
很明显，这个API的使用者（这个例子中为最佳价格查询器）调用该方法时，它依旧会被
阻塞。为等待同步事件完成而等待1秒钟，这是无法接受的。

#### 将同步方法转换为异步方法
     public Future<Double> getPriceAsync(String product) { ... }
     
Java 5引入了java.util.concurrent.Future接口表示一个异
步计算（即调用线程可以继续运行，不会因为调用方法而阻塞）的结果。这意味着Future是一
个暂时还不可知值的处理器，这个值在计算完成后，可以通过调用它的get方法取得。
- getPriceAsync方法的实现
   
      public Future<Double> getPriceAsync(String product) {
         //创建CompletableFuture 对象，它会包含计算的结果
         CompletableFuture<Double> futurePrice = new CompletableFuture<>(); 
         new Thread( () -> { 
         //在另一个线程中以异步方式执行计算
         double price = calculatePrice(product); 
         //需长时间计算的任务结束并得出结果时，设置Future的返回值
         futurePrice.complete(price); 
         }).start(); 
          return futurePrice; 
         } 


- 使用异步API
    
    
      Shop shop = new Shop("BestShop"); 
     long start = System.nanoTime(); 
     Future<Double> futurePrice = shop.getPriceAsync("my favorite product"); 
     long invocationTime = ((System.nanoTime() - start) / 1_000_000); 
     System.out.println("Invocation returned after " + invocationTime 
      + " msecs"); 
     // 执行更多任务，比如查询其他商店
     doSomethingElse(); 
     // 在计算商品价格的同时
     try { 
      double price = futurePrice.get(); 
      System.out.printf("Price is %.2f%n", price); 
     } catch (Exception e) { 
      throw new RuntimeException(e);
     } 
     long retrievalTime = ((System.nanoTime() - start) / 1_000_000); 
     System.out.println("Price returned after " + retrievalTime + " msecs"); 

如果没有意外，我们目前开发的代码工作得很正常。但是，如果价格计算过程中产生了错误
会怎样呢？这种情况下你会得到一个相当糟糕的结果：用于提示错误的异常会被限制
在试图计算商品价格的当前线程的范围内，最终会杀死该线程，而这会导致等待get方法返回结
果的客户端永久地被阻塞。

### 使用工厂方法supplyAsync创建CompletableFuture

CompletableFuture类自
身提供了大量精巧的工厂方法，使用这些方法能更容易地完成整个流程，还不用担心实现的细节。
比如，采用supplyAsync方法后，你可以用一行语句重写代码中的getPriceAsync方
法，如下所示
   
     public Future<Double> getPriceAsync(String product) { 
      return CompletableFuture.supplyAsync(() -> calculatePrice(product)); 
     } 
     
- 采用顺序查询所有商店的方式实现的findPrices方法


     public List<String> findPrices(String product) { 
      return shops.stream() 
      .map(shop -> String.format("%s price is %.2f", 
     shop.getName(), shop.getPrice(product))) 
     .collect(toList()); 
     } 
        //Done in 4032 msecs 


> findPrices方法的执行时间仅比4秒钟多了那么几毫秒，因为对这4个商店
的查询是顺序进行的，并且一个查询操作会阻塞另一个，每一个操作都要花费大约1秒左右的时
间计算请求商品的价格。

- 使用并行流对请求进行并行操作
     
   
      public List<String> findPrices(String product)    { 
     return shops.parallelStream() 
     .map(shop -> String.format("%s price is %.2f", 
     shop.getName(), shop.getPrice(product)))
     .collect(toList()); 
     } 
    // Done in 1180 msecs 
 
> 现在对四个不同商店的查询实现了并行，所
以完成所有操作的总耗时只有1秒多一点儿。你能做得更好吗？让我们尝试使用刚学过的
CompletableFuture，将findPrices方法中对不同商店的同步调用替换为异步调用。
    
      List<CompletableFuture<String>> priceFutures = 
     shops.stream() 
     .map(shop -> CompletableFuture.supplyAsync( 
     () -> String.format("%s price is %.2f", 
     shop.getName(), shop.getPrice(product)))) 
     .collect(toList()); 
