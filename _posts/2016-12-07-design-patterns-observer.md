---
layout: post
title: 设计模式之观察者模式
tags:
- 设计模式
- java
- 观察者模式
categories: 设计模式
---

　　第一个设计模式已经学习完了，相信你和我一样意犹未尽，本文准备学习第二个设计模式-观察者模式，首先看一下定义：

>　　观察者模式：在对象之间定义一对多的依赖, 这样一来, 当一个对象改变状态, 依赖它的对象都会收到通知, 并自动更新。

<!-- more -->

　　以订阅天气通知为例，设计一个布告板，当收到气象站有关温度变化的时候，通知所有订阅了天气变化信息的用户进行通知。

#### 普通思路

　　首先设计一个布告板的接口，用来显示温度的信息：

```java
/**
 * 布告板，通知订阅用户温度
 * @author jianxw
 * @date 2016/12/8
 */
public interface IBillBoard {
    void display(String tem);
}

```

布告板的实现：

```java

/**
 * 布告板实现
 *
 * @author jianxw
 * @date 2016/12/8
 */
public class BillBoardImpl implements IBillBoard {
    private String name;

    public BillBoardImpl(String name) {
        this.name = name;
    }

    @Override
    public void display(String tem) {
        System.out.println("订阅用户：" + name + "，温度为：" + tem);
    }
}

```

设计一个气象站接口：

```java

/**
 * 气象站接口
 * @author jianxw
 * @date 2016/12/8
 */
public interface WeatherData {
    void change(String tem);
}

```

然后是气象站的实现：

```java

/**
 * 气象站实现
 * @author jianxw
 * @date 2016/12/8
 */
public class WeatherDataImpl implements WeatherData {

    private List<IBillBoard> iBillBoards;

    public WeatherDataImpl(List<IBillBoard> iBillBoards) {
        this.iBillBoards = iBillBoards;
    }

    @Override
    public void change(String tem) {
        for(IBillBoard iBillBoard: iBillBoards){
            iBillBoard.display(tem);
        }
    }
}

```

需要给用户“wjx”和“jianxw”订阅天气通知，当天气改变的时候，能够通知到两个用户：

```java
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;

/**
 * 订阅类
 *
 * @author jianxw
 * @date 2016/12/8
 */
public class Client {
    @Test
    public void test() {
        IBillBoard billBoardWjx = new BillBoardImpl("wjx");
        IBillBoard billBoardJianxw = new BillBoardImpl("jianxw");
        List<IBillBoard> iBillBoards = new ArrayList<>();
        iBillBoards.add(billBoardWjx);
        iBillBoards.add(billBoardJianxw);
        WeatherData temperatureChange = new WeatherDataImpl(iBillBoards);
        temperatureChange.change("10度");
    }
}


```

这样也能满足通知所有订阅的用户，但是每次新增加、删除和修改订阅者的时候需要重新new WeatherData。

#### 观察者模式

　　优化后的气象站接口为：

```java
/**
 * 气象站接口
 * @author jianxw
 * @date 2016/12/8
 */
public interface WeatherData {
    void addBillBoard(IBillBoard iBillBoard);
    void deleteBillBoard(IBillBoard iBillBoard);
    void notifyBillBoards(String tem);
    void change(String tem);
}

```

优化后的气象站实现：

```java
import java.util.ArrayList;
import java.util.List;

/**
 * 气象站实现
 * @author jianxw
 * @date 2016/12/8
 */
public class WeatherDataImpl implements WeatherData {

    private List<IBillBoard> iBillBoards;


    @Override
    public void addBillBoard(IBillBoard iBillBoard) {
        if (iBillBoards == null) {
            iBillBoards = new ArrayList<>();
        }
        iBillBoards.add(iBillBoard);
    }

    @Override
    public void deleteBillBoard(IBillBoard iBillBoard) {
        if (iBillBoards == null) {
            return;
        }
        if (!iBillBoards.contains(iBillBoard)) {
            iBillBoards.remove(iBillBoard);
        }
    }

    @Override
    public void notifyBillBoards(String tem) {
        if (iBillBoards == null) {
            return;
        }
        for (IBillBoard iBillBoard : iBillBoards) {
            iBillBoard.display(tem);
        }
    }

    @Override
    public void change(String tem) {
        notifyBillBoards(tem);
    }
}
```

然后订阅类来模拟用户订阅天气预报：

```java

import org.junit.Test;

/**
 * 订阅类
 *
 * @author jianxw
 * @date 2016/12/8
 */
public class client {

    @Test
    public void test(){
        WeatherData weatherData = new WeatherDataImpl();
        IBillBoard iBillBoard = new BillBoardImpl("jianxw");
        IBillBoard iBillBoardJohn = new BillBoardImpl("wjx");
        weatherData.addBillBoard(iBillBoard);
        weatherData.addBillBoard(iBillBoardJohn);
        weatherData.change("10度");
    }

}

```

#### 类图

　　优化后的类图为：

![](/img/design_patterns/observer.png)
　　
#### 总结

　　java也自带了观察者模式，文章就不做介绍了，有很多文章进行介绍，就不在做介绍了。