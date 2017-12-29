# 用Optional取代null

NullPointerException是目前Java程序开发中最典型的异常，它让你的代码充斥着深度嵌套的null检查，代码的可读性糟糕透顶。

## Optional 类入门
Java 8中引入了一个新的类java.util.Optional<T>。这
是一个封装Optional值的类。变量存在时，Optional类只是对类简单封装。变量不存在时，缺失的值会被建模成一个“空”
的Optional对象，由方法Optional.empty()返回。

不使用optional的例子
 ``` java
public class Person { 
    private Car car; 
     public Car getCar() { return car; } 
    } 
    public class Car { 
         private Insurance insurance; 
         public Insurance getInsurance() { return insurance; } 
    } 
public class Insurance { 
 private String name; 
 public String getName() { return name; } 
} 
那么，下面这段代码存在怎样的问题呢？
public String getCarInsuranceName(Person person) { 
 return person.getCar().getInsurance().getName(); 
} 
 ```
 如果person.getCar()==null,这个人没有车，就会抛个NullPointerException，
 
 使用optional
 ```Java
 public class Person { 
 //人可能有车，也可能没有车，因此将这个字段声明为Optional
 private Optional<Car> car; 
 public Optional<Car> getCar() { return car; } 
 }
public class Car { 
//车可能进行了保险，也可能没有保险，所以将这个字段声明为ptional
 private Optional<Insurance> insurance; 
 public Optional<Insurance> getInsurance() { return insurance; } 
} 
public class Insurance { 
 private String name; 
 public String getName() { return name; }
} 
```
> null引用和Optional.empty()
有什么本质的区别吗？从语义上，你可以把它们当作一回事儿，但是实际中它们之间的差别非常
大：如果你尝试解引用一个 null ，一定会触发 NullPointerException ，不过使用
Optional.empty()就完全没事儿

## 应用 Optional 的几种模式
### 创建 Optional 对象
* 1. 声明一个空的Optional
正如前文已经提到，你可以通过静态工厂方法Optional.empty，创建一个空的Optional
对象：


     Optional<Car> optCar = Optional.empty(); 

* 2. 依据一个非空值创建Optional
你还可以使用静态工厂方法Optional.of，依据一个非空值创建一个Optional对象：如果car是一个null，这段代码会立即抛出一个NullPointerException，而不是等到你
试图访问car的属性值时才返回一个错误。
    
    
    Optional<Car> optCar = Optional.of(car); 

* 3. 可接受null的Optional
最后，使用静态工厂方法Optional.ofNullable，你可以创建一个允许null值的Optional
对象：

     
     Optional<Car> optCar = Optional.ofNullable(car); 
如果car是null，那么得到的Optional对象就是个空对象。

### 使用 map 从 Optional 对象中提取和转换值
比如，你可能想要从insurance公司对象中提取
公司的名称。提取名称之前，你需要检查insurance对象是否为null，代码如下所示：
    
     String name = null; 
     if(insurance != null){ 
      name = insurance.getName(); 
     } 
为了支持这种模式，Optional提供了一个map方法。它的工作方式如下：
    
    Optional<Insurance> optInsurance = Optional.ofNullable(insurance); 
    Optional<String> name = optInsurance.map(Insurance::getName); 
    
    public String getCarInsuranceName(Person person) { 
    return person.getCar().getInsurance().getName(); 
    } 
    
### 使用 flatMap 链接 Optional 对象
利用map重写之前的代码

     Optional<Person> optPerson = Optional.of(person); 
    	Optional<String> name = 
    	 optPerson.map(Person::getCar) 
    	 .map(Car::getInsurance) 
    	 .map(Insurance::getName); 
    	 
无法通过编译。为什么呢？optPerson是Optional<Person>类型的
变量， 调用map方法应该没有问题。但getCar返回的是一个Optional<Car>类型的对象（如代
码清单10-4所示），这意味着map操作的结果是一个Optional<Optional<Car>>类型的对象。因
此，它对getInsurance的调用是非法的，因为最外层的optional对象包含了另一个optional
对象的值，而它当然不会支持getInsurance方法。
> latMap方法接受一个函数作为参数，这个函数的返回值是另一个流。
这个方法会应用到流中的每一个元素，最终形成一个新的流的流。但是flagMap会用流的内容替
换每个新生成的流。
    
    public String getCarInsuranceName(Optional<Person>  person) { 
     return person.flatMap(Person::getCar) 
     .flatMap(Car::getInsurance) 
     .map(Insurance::getName) 
     .orElse("Unknown"); 
     } 

由Optional<Person>对象，我们可以结合使用之前介绍的map和flatMap方法，从Person
中解引用出Car，从Car中解引用出Insurance，从Insurance对象中解引用出包含insurance
公司名称的字符串。
![](https://s1.ax2x.com/2017/12/29/0KGFz.png)

### 默认行为及解引用 Optional 对象

Optional类提供了多种方法读取
Optional实例中的变量值

- get()是这些方法中最简单但又最不安全的方法。如果变量存在，它直接返回封装的变量
值，否则就抛出一个NoSuchElementException异常。
- orElse(T other)允许你在Optional对象不包含值时提供一个默认值。
- orElseGet(Supplier<? extends T> other)是orElse方法的延迟调用版，Supplier
方法只有在Optional对象不含值时才执行调用
- orElseThrow(Supplier<? extends X> exceptionSupplier)和get方法非常类似，
它们遭遇Optional对象为空时都会抛出一个异常，但是使用orElseThrow你可以定制希
望抛出的异常类型。
- ifPresent(Consumer<? super T>)让你能在变量值存在时执行一个作为参数传入的
方法，否则就不进行任何操作。

### 使用 filter 剔除特定的值
     
     Optional<Insurance> optInsurance = ...; 
     optInsurance.filter(insurance -> 
      "CambridgeInsurance".equals(insurance.getName())) 
      .ifPresent(x -> System.out.println("ok")); 
    

