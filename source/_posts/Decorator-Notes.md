---
title: 设计模式学习之装饰者模式
date: 2017-5-12 2:16:06
categories:
- Program
tags:
- java
- 设计模式
- 装饰者模式
---

这学期前几个月为了写Mango Cloud的网站和微信公众号程序，耽搁了不少时间，已经很长时间没有学习新的知识了。现在终于可以拿出点时间学点新东西了，在之前的网站开发过程中也深刻感受到了面向对象程序编写中存在很多问题，许多业务代码只是简单的调用方法来完成业务逻辑，其中有不少代码存在重复使用，总之代码看起来不是那么的漂亮，对于强迫症的我是不可容忍的。  
这次继续设计模式的学习--------装饰者模式  
<!-- more --> 

其实装饰者模式在学习JAVA IO部分的时候就接触到了，例如以下代码：

```java
BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("test.txt")));
```

其中stream就经过了stream->streamreader->bufferedreader的包装过程。下面对装饰者模式进行拆分理解，搞清楚它是怎么实现的。

### 装饰者模式要解决的问题

通常一个类的功能拓展是通过继承来实现的，而往往一个子类相较于父类只增添了一点属性或更改了一个方法的具体实现内容，随着功能需求的增多，会生成很多子类，增大了程序的维护难度。使用继承也意味着这些功能在编译的时候就已经确定了，是静态的。

### 装饰者模式的定义

装饰者模式动态地将责任附加到对象上。装饰者模式允许向一个现有的对象添加新的功能，同时又不改变其原有的结构。装饰者模式创建了一个装饰类，将原来的类进行包装，并在保持原来的类方法签名完整性的前提下，提供了额外的功能。OO原则之开放-关闭原则------对外扩展开放，对修改关闭，即不修改现有代码的情况下，扩充原有类的行为。

### 代码实例（摘自HeadFirst设计模式上的例子）

```java
//定义一个抽象类，为之后的被装饰对象提供一致的数据类型，这里也可以定义一个接口
public abstract class Beverage{
  String description = "Unknown Beverage";
  
  public String getDescription(){
    return description;
  }
  
  public abstract double cost();
  
}
```



```java
//装饰者抽象类，需要通过继承获得与被装饰者一致的属性和方法，也就是保持原有类的完整性
public abstract class CondimentDecorator extends Beverage{
  
  public abstract String getDescription();
}
```

```java
//定义一个被装饰对象，这里的description是从父类继承得到的
public class Espresso extends Beverage{
  
  public Espresso(){
    
    description = "Espresso";
  }
  
  public double cost(){
    
    return 1.99;
  }
}
```

```java
//定义一个装饰者类，通过这个装饰者类可以为被装饰对象提供新的功能，比如这里的为咖啡加上摩卡，并计算新的价格
public class Mocha extends CondimentDecorator{
  
  Beverage beverage;
  
  public Mocha(Beverage beverage){
    this.beverage = beverage;
  }
  
  public String getDescription(){
    return beverage.getDescription() + ", Mocha";
  }
  
  public double cost(){
    
    return .20 + beverage.cost();
  }
  
}
```

```java
//代码测试
public class DecoratorTest{
  
  public static void main(String args[]){
    
    	Bdverage beverage = new Espresso();
    	beverage = new Mocha(beverage);
    	System.out.println("The coffee is"+mocha.getDescription()+" ,The price is "+mocha.cost());
  }
}
```

我这里举的代码例子只使用了一个装饰类，当然还可以继续定义新的装饰类来继续对mocha咖啡进行扩充，例如是否加牛奶，设定容量大小等等。

利用装饰者模式可以更灵活的对某个类的功能进行扩充，用不着派生出无数的子类，对于代码维护挺有好处的。但是装饰者模式会导致在使用过程中出现许多小对象，如果过度使用，会让程序变得很复杂。