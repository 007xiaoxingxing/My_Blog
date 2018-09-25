---
title: 设计模式学习之工厂模式

date: 2017-5-12 14:28:16
categories:
- Program
tags:
- java
- 设计模式
- 工厂模式
---

继续设计模式的学习，接下来是工厂模式。

### 工厂模式所要解决的问题

如果实例化一个对象需要根据繁杂的条件来创建不同类型的对象，那么随着系统的复杂化，创建对象的过程也会变得十分复杂，每次有新的需求都需要去加上判断条件，造成了设计的复杂，也不方便其他方法调用，所以可以将创建对象的过程放到专门的工厂来创建。工厂类扮演了对象生产者的角色。

### 工厂模式的分类

1. 简单工厂

2. 工厂方法模式

3. 抽象工厂模式

<!-- more -->

接下来分别说明工厂模式的以上三种的类型的内容和它们之间的区别。

#### 简单工厂

最开始我们要实例化一个对象的时候是根据需要直接去new一个来用就行了，例如我要购买小米手机:

```java
public class XiaoMi2s{
  
  public XiaoMi2s{
    System.out.println("create Xiaomi 2s");
  }
  
  public void sell(){
        System.out.println("MI2s selled You need pay 1999￥");
  }
}
public class XiaoMi5s{
  
  public XiaoMi5s{
    System.out.println("create Xiaomi 5s");
  }
  public void sell(){
        System.out.println("MI5s selled 1999￥");
  }
}

public class PhoneShop{
  Phone phone;
  public static void main(String[] args){
    
    buyPhone("Mi2s");
    
  }
  public void buyPhone(String model){
    if(model.equals("MI2s")){
      phone = new new XiaoMi2s();
      phone.sell();
    }
    if(model.equals("MI5s")){
      phone = new new XiaoMi5s();
      phone.sell();
    }
    
  }
}

```

可以看到，如果我要购买不同型号的手机，手机店需要进行很多次的判断才能拿到我想要的手机并卖给我。这项就显得十分繁琐了。

简单工厂模式可以解决这个问题，把创建对象的过程放到专门的工厂中去做，像这样：

```java
public class SimplePhoneFactory{
  public Phone createPhone(String model){
    
    Phone phone = null;
    if(model.equals("MI2s")){
      phone = new new XiaoMi2s();
      phone.sell();
    }
    if(model.equals("MI5s")){
      phone = new new XiaoMi5s();
      phone.sell();
    }
    return phone;
    
  }
}
```

简单工厂其实并不算是一种设计模式，反而比较像是一种编程习惯。这个简单工厂为需要的对象创建不同的“产品”。简单工厂也可以被定义为一个静态的方法，这样就不需要使用创建对象的方法来实例化对象，但是这样就不能通过继承来改变创建方法的行为。

