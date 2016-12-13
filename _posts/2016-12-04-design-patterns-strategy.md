---
layout: post
title: 设计模式之策略模式
tags:
- 设计模式
- java
- 策略模式
categories: 设计模式
---
　　设计模式是程序员必备的知识和技能，平时开发过程中，用到一些的设计模式，但是没有很好的进行归类和总结，所以准备通过对设计模式系统的学习和归纳总结，希望能够在以后的开发过程中更好和合理的使用设计模式。
>　　策略模式是：定义了算法族, 分别封装起来, 让它们之间可以相互替换, 此模式让算法的变化独立于使用算法的客户。

<!-- more -->
　　以一个鸭子的游戏为例，游戏中的鸭子既能游泳又能呱呱叫。
####鸭子类
　　首先要设计一个鸭子的类，鸭子有外观、游泳和叫的属性和行为：
```java
/**
 * 鸭子
 * @author jianxw
 * @date 2016/12/3
 */
public class Duck {
    void swim() {
        System.out.println("鸭子浮水");
    }

    void display() {
        System.out.println("普通鸭子外观");
    }

    void quack() {
        System.out.println("鸭子呱呱叫");
    }
}
```

然后鸭子有野鸭子和红头鸭：
```java

/**
 * 野鸭
 * @author jianxw
 * @date 2016/12/3
 */
public class MallardDuck extends Duck {
    /**
     * 野鸭是绿头的
     */
    @Override
    void display() {
        System.out.println("外观是绿头");
    }
}

```

```java

/**
 * 红头鸭
 * @author jianxw
 * @date 2016/12/3
 */
public class ReadHeadDuck extends Duck {

    @Override
    public void display() {
        System.out.println("外观是红头");
    }

}

```
现在有个新的需求需要引入，在以前的鸭子的基础上增加一种可以在屏幕上飞行的橡皮鸭，为了满足这个需求，在上面的鸭子类的的基础上增加一个fly的行为，修改后鸭子类为：
```java
/**
 * 鸭子
 * @author jianxw
 * @date 2016/12/3
 */
public class Duck {
    void swim() {
        System.out.println("鸭子浮水");
    }

    void display() {
        System.out.println("普通鸭子外观");
    }

    void quack() {
        System.out.println("鸭子呱呱叫");
    }

    void fly() {
        System.out.println("鸭子飞");
    }

}
```
然后定义一个橡皮鸭：
```java

/**
 * 橡皮鸭
 *
 * @author jianxw
 * @date 2016/12/3
 */
public class RubberDuck extends Duck {
    @Override
    void quack() {
        System.out.println("橡皮鸭吱吱叫");
    }
}

```
这样就能够满足橡皮鸭在屏幕上飞行，现在问题出现了，野鸭和红头鸭也莫名其妙的飞起来了，为了满足新的需求导致不该飞的鸭子也飞起来了。
>设计模式的第一个原则：封装变化-找出需要变化的部分，独立出来，不需要和那些需要变化的部分放在一起。

####优化
　　根据第一个原则，需要把不同鸭子的需要变化的行为单独的取出来，所以设计了两个鸭子行为的接口，一个飞行行为FlyBehavior和叫声行为QuackBehavior，
>设计模式的第二个设计原则：针对接口编程而不是针对实现编程

```java

/**
 * 飞行行为接口
 * @author jianxw
 * @date 2016/12/3
 */
public interface FlyBehavior {
    void fly();
}

```

```java

/**
 * 叫行为的接口
 * @author jianxw
 * @date 2016/12/3
 */
public interface QuackBehavior {
    void quack();
}

```

设计飞行行为的实现，一个是用翅膀飞，一个是不具备飞行能力:
```java

/**
 * 用翅膀飞
 * @author jianxw
 * @date 2016/12/3
 */
public class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("用翅膀飞");
    }
}

```

```java

/**
 * 不具有飞行能力
 * @author jianxw
 * @date 2016/12/3
 */
public class FlyNoWay implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("不具有飞行能力");
    }
}

```

叫声也有两个行为试下一个普通的呱呱叫和一个不能够叫：
```java

/**
 * 普通呱呱叫
 * @author jianxw
 * @date 2016/12/3
 */
public class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("呱呱叫");
    }
}

```

```java

/**
 * 不能够叫
 * @author jianxw
 * @date 2016/12/3
 */
public class MuteQuack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("不能够叫");
    }
}

```
然后重新的来设计鸭子类，鸭子的飞行行为和叫声行为由子类来决定：
```java

/**
 * 优化后的鸭子类
 * @author jianxw
 * @date 2016/12/3
 */
public abstract class Duck {
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;

    public Duck() {

    }
    public abstract void display();

    public void performFly() {
        flyBehavior.fly();
    }

    public void perfromQuack() {
        quackBehavior.quack();
    }

    public void setFlyBehavior(FlyBehavior flyBehavior) {
        this.flyBehavior = flyBehavior;
    }

    public void setQuackBehavior(QuackBehavior quackBehavior) {
        this.quackBehavior = quackBehavior;
    }
}

```

重新设计橡皮鸭的类：
```java

/**
 * 橡皮鸭
 * @author jianxw
 * @date 2016/12/3
 */
public class RubberDuck extends Duck {
    @Override
    public void display() {
        System.out.println("橡皮鸭为黄色");
    }
}

```
新建橡皮鸭的测试类：
```java

/**
 * @author jianxw
 * @date 2016/12/3
 */
public class DuckTest {
    public static void main(String[] args) {
        Duck duck = new RubberDuck();
        duck.setFlyBehavior(new FlyWithWings());
        duck.setQuackBehavior(new SqueakQuack());
        duck.display();
        duck.performFly();
        duck.perfromQuack();
    }
}
```
这样设计让鸭子有飞行行为和呱呱叫的行为，程序能够动态的给橡皮鸭添加行为和叫声，同时又新的鸭子的需求也能够很好的扩展，而且扩展的过程中也不影响其他鸭子的行为。
>设计模式的另外一个原则：多用组合，少用继承

####类图
　　上面优化的鸭子类对应的类图为：
![](/img/design_patterns/strategy_duck.png)
<br/>
####总结
　　第一个设计模式学习完了，主要涉及到3个设计原则和一个设计模式，在开发过程中这个设计模式还是经常会用到，希望这个分享能够让大家在以后的开发设计过程中更多的将设计模式和一些原则运用到其中。