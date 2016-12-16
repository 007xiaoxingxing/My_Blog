---
title: 设计模式学习之观察者模式
date: 2016-12-16 15:57:06
categories:
- Program
tags:
- java
- 设计模式
- 观察者模式
---

之前在学习java的过程中，总提到总会遇到“设计模式”，诸如工厂模式了，观察者模式了，单例模式了....看起来云里雾里的。趁最近空闲时间多了些，来把这些设计模式都熟悉一下。我看的书的《Head First 设计模式 》，这个系列的书看起来还蛮有意思的。不闲扯了，这篇博文是我学习“观察者模式”的笔记。

<!-- more -->

观察者模式有时也被称作“发布-订阅”模式。这种模式有点像我们去订阅报纸，我们是订阅者，报社是发布者，每当报社出版了新的报纸，我们就能够收到最新的报纸了。  在GUI编程中，控件的事件监听算是观察者模式的具体应用了吧。  

> 观察者模式定义了对象之间的一对多依赖，这样一来，当一个对象发生改变时，它的所有依赖者都会收到通知并自动更新。

-----摘自《Head First设计模式》  

那么我们具体来实现一种观察者模式的应用，这样看起来就更加直观了。

观察者模式的最基本的类图如下：  

![image](/blogimg/ObserverUML.bmp)

> Subject接口：定义的主题接口，对象使用此接口注册为观察者，或者把自己从观察者之中删除

> Observer接口：所有潜在的观察者必须实现观察者接口，这个接口只有update方法，当主题状态改变时调用

> ConcreteSubject类：一个具体的主题，实现了Subject接口，除了注册和撤销方法，该主题还实现了notifyObserver()方法，用来在状态改变时，通知所有的观察者

> ConcreteObserver类：具体的观察者，它实现了Observer接口，观察者必须注册具体的Subject，才能够就收到更新事件的通知

  接着来具体实现一下书上提供的气象站的例子：

  Subject.java

```java
  public interface Subject{
      public void registerObserver(Observer o); //注册观察者
      public void removeObserver(Observer o); //移除观察者
      public void notifyObservers(); //当有更新事件发生时，通知观察者
  }
```

  Observer.java

```java
  public interface Observer{
      public void update(float temp, float humidity, float pressure);
      //所有的观察者都必须实现update()方法，以实现观察者接口
  }
```

  WetherData.java

```java
  public class WetherData implements Subject { //需要实现Subject接口

      private ArrayList observers; //用于存放注册的所有观察者对象
      private float tempature;
      private float humidity;
      private float pressure;

      public WetherData(){
          observers = new ArrayList();
      }
      @Override
      public void registerObserver(Observer o) {
          this.observers.add(o);
      }

      @Override
      public void removeObserver(Observer o) {
              int i = observers.indexOf(o);
              if(i > 0){
                  observers.remove(i);
              }
      }

      @Override
      public void notifyObservers() {
        /*当有更新事件发生时，调用此方法。在这里，它会去挨个通知注册的观察者，让他们执行update()方法更新数据*/
          for(int i = 0; i < observers.size(); i++){
              Observer observer=(Observer)observers.get(i);
              observer.update(tempature, humidity, pressure);
          }
      }

      public void measurementsChanged(){
        //接收更新事件通知，然后告诉所有的注册的观察者
          notifyObservers();
      }

      public void setMeasurements(float tempature, float humidity, float pressure){
  		//当有的气象观测数据到来，调用此方法
          this.tempature = tempature;
          this.humidity = humidity;
          this.pressure = pressure;
       
          measurementsChanged(); //发出通知
      }
  }

```

  CurrentConditionDisplay.java

```java
  //这是气象数据显示的公告板，它实现了Observer接口，成为了观察者
  public class CurrentConditionDisplay implements Observer, DisplayElement {

      private float temprature;
      private float humidity;
      private Subject weatherData;

      public CurrentConditionDisplay(Subject weatherData){

          this.weatherData = weatherData;
        //注册成为wetherData的观察者，随时准备接收新数据
          weatherData.registerObserver(this);
      }
      @Override
      public void update(float temp, float humidity, float pressure) {
  			//取得来自Subject的数据，并暂存起来用于稍后的显示
              this.temprature = temp;
              this.humidity = humidity;
              display();
      }

      @Override
      public void display() {
        //显示数据
          System.out.println("Current conditions:" + temprature + "F degree and" + humidity + "% humidity");
      }
  }

```

  DisplayElement.java

```java
  public interface DisplayElement{
      public void display();
  }
```

  ObserverMain.java

```java
  public class ObserverMain {

     public static void main(String[] args){

         WetherData wetherData = new WetherData();

  	CurrentConditionDisplay currentConditionDisplay = new currentConditionDisplay(wetherData);

         wetherData.setMeasurements(80,65,30.1f);
         wetherData.setMeasurements(40,30,10.9f);
         wetherData.setMeasurements(10,20,28.3f);

     }

  }
```

  编译，执行看下结果呢。

  ```
  F:/java ObserverMain
  Current conditions:80.0F degree and65.0% humidity
  Current conditions:40.0F degree and30.0% humidity
  Current conditions:10.0F degree and20.0% humidity
  ```

  Nice，咱们的气象站一收到新的数据，公告板就及时的将数据显示了出来。

