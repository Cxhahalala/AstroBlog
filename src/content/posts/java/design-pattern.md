---
title: "设计模式"
published: 2025-07-24
description: "设计模式学习笔记：工厂模式、单例模式等常用创建型与结构型设计模式的核心思想与实战代码解析。"
image: "https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260817162903558.png"
tags: ["设计模式","java"]
category: "Java"
draft: false
---

# 设计模式学习笔记

## 一、工厂模式 (Factory Pattern)

工厂模式属于 **创建型模式 (Creational Pattern)**，它的核心思想是**将对象的创建过程封装起来**。使用者（客户端）不需要关心对象是如何被创建的，只需要向工厂索要即可。

### 1. 生活比喻
想象一下去快餐店点餐。你只需要走到柜台说“我想要一个汉堡”，然后柜台后面的“工厂”（厨房）就会为你制作好一个汉堡并交给你。

*   **你（客户端）**：不需要知道汉堡的面包是哪个牌子的，肉饼是怎么煎的，蔬菜是怎么切的。
*   **柜台（工厂接口）**：是你和厨房沟通的唯一途径。
*   **厨房（具体工厂）**：负责汉堡（对象）的实际创建过程。

如果你想喝可乐，厨房这个“工厂”同样能生产出来。这个“工厂”封装了所有产品的创建细节。

#### 2. 工厂模式的分类与详解

工厂模式通常分为三种：简单工厂、工厂方法和抽象工厂。

##### (1) 简单工厂模式 (Simple Factory Pattern) - “万能厨房”

*   **定义**：一个工厂类根据传入的参数，动态决定应该创建哪一种产品类的实例。
*   **角色**：
    *   **Factory (工厂类)**：负责创建所有产品实例的核心类。
    *   **Product (抽象产品)**：所有产品需要实现的接口或继承的父类。
    *   **ConcreteProduct (具体产品)**：工厂类创建的目标，是抽象产品的具体实现。
*   **优点**：简单，客户端免除了直接创建对象的责任。
*   **缺点**：工厂类职责过重，如果要增加新产品，就需要修改工厂类的判断逻辑，违反了**开闭原则**（对扩展开放，对修改关闭）。

**代码示例 (Java):**
```java
// 1. 抽象产品：车
interface Car {
    void run();
}

// 2. 具体产品：特斯拉、宝马
class Tesla implements Car {
    @Override
    public void run() {
        System.out.println("特斯拉在跑...");
    }
}

class Bmw implements Car {
    @Override
    public void run() {
        System.out.println("宝马在跑...");
    }
}

// 3. 工厂类
class CarFactory {
    public static Car createCar(String type) {
        if ("tesla".equalsIgnoreCase(type)) {
            return new Tesla();
        } else if ("bmw".equalsIgnoreCase(type)) {
            return new Bmw();
        }
        return null;
    }
}

// 客户端使用
public class Client {
    public static void main(String[] args) {
        Car tesla = CarFactory.createCar("tesla");
        if (tesla != null) {
            tesla.run(); // 输出: 特斯拉在跑...
        }
    }
}
```

##### (2) 工厂方法模式 (Factory Method Pattern) - “专卖店”

*   **定义**：定义一个用于创建对象的接口，但让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。
*   **核心**：将简单工厂中集中的创建逻辑分散到各个子工厂中，每个子工厂只负责生产一种特定的产品。
*   **角色**：
    *   **AbstractFactory (抽象工厂)**：声明创建产品的工厂方法。
    *   **ConcreteFactory (具体工厂)**：实现工厂方法，创建具体的产品。
    *   **Product (抽象产品)**：同上。
    *   **ConcreteProduct (具体产品)**：同上。
*   **优点**：完美遵循开闭原则。增加新产品时，只需增加一个具体产品类和一个对应的具体工厂类，无需修改原有代码。
*   **缺点**：每增加一个产品，就需要增加一个工厂类，导致类的数量成倍增加。

**代码示例 (Java):**
```java
// Product 接口和 ConcreteProduct 类保持不变

// 1. 抽象工厂
interface ICarFactory {
    Car createCar();
}

// 2. 具体工厂：特斯拉工厂、宝马工厂
class TeslaFactory implements ICarFactory {
    @Override
    public Car createCar() {
        return new Tesla();
    }
}

class BmwFactory implements ICarFactory {
    @Override
    public Car createCar() {
        return new Bmw();
    }
}

// 客户端使用
public class Client {
    public static void main(String[] args) {
        // 我想买特斯拉，就去找特斯拉工厂
        ICarFactory teslaFactory = new TeslaFactory();
        Car tesla = teslaFactory.createCar();
        tesla.run(); // 输出: 特斯拉在跑...

        // 我想买宝马，就去找宝马工厂
        ICarFactory bmwFactory = new BmwFactory();
        Car bmw = bmwFactory.createCar();
        bmw.run(); // 输出: 宝马在跑...
    }
}
```

##### (3) 抽象工厂模式 (Abstract Factory Pattern) - “品牌旗舰店”
*   **定义**：提供一个接口，用于创建**一系列相关或相互依赖的对象**，而无需指定它们具体的类。
*   **核心**：它生产的不是单一产品，而是一个**产品族（family）**。
*   **比喻**：一个“小米”旗舰店（抽象工厂）可以生产“小米手机”、“小米耳机”、“小米充电器”（一个产品族）。一个“苹果”旗舰店（另一个抽象工厂）可以生产“iPhone”、“AirPods”、“MagSafe充电器”（另一个产品族）。
*   **优点**：非常适合用于创建产品族。客户端与具体的产品实现解耦。
*   **缺点**：最复杂。如果产品族需要增加一个新种类的产品（例如增加“智能手表”），那么所有的工厂类都需要修改，违反了开闭原则。

---

## 二、策略模式 (Strategy Pattern)

策略模式属于 **行为型模式 (Behavioral Pattern)**，它的核心思想是**定义一系列算法，将每一个算法封装起来，并使它们可以相互替换**。

### 1. 生活比喻
想象一下你去商场结账。你可以选择多种支付方式：
*   **策略A**：使用信用卡支付。
*   **策略B**：使用支付宝扫码支付。
*   **策略C**：使用微信扫码支付。

对于“支付”这个行为，你有多种不同的实现（策略）。你可以根据情况（比如哪个有优惠）在结账时自由选择其中一种策略。**你的选择不会影响商品本身，只会改变“支付”这个行为的具体实现方式。**

#### 2. 定义与目的
*   **定义**：它定义了算法家族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。
*   **目的**：将算法的**定义**和算法的**使用**分离开来，避免在代码中使用大量的 `if-else` 或 `switch-case` 来选择不同的行为。

### 3. 结构和代码示例

*   **角色**：
    *   **Context (上下文)**：持有一个 `Strategy` 对象的引用。它不关心具体是哪个策略，只负责在需要时调用策略的方法。通常是使用策略的那个类。
    *   **Strategy (策略接口)**：定义所有支持的算法的公共接口。
    *   **ConcreteStrategy (具体策略)**：实现了 `Strategy` 接口，封装了具体的算法或行为。

**代码示例 (Java):**
```java
// 1. 策略接口：支付策略
interface PaymentStrategy {
    void pay(int amount);
}

// 2. 具体策略：信用卡支付、支付宝支付
class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;

    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }

    @Override
    public void pay(int amount) {
        System.out.println("使用信用卡 " + cardNumber + " 支付了 " + amount + " 元。");
    }
}

class AlipayPayment implements PaymentStrategy {
    private String email;

    public AlipayPayment(String email) {
        this.email = email;
    }

    @Override
    public void pay(int amount) {
        System.out.println("使用支付宝账号 " + email + " 支付了 " + amount + " 元。");
    }
}

// 3. 上下文：购物车
class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    // 允许客户端在运行时设置策略
    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(int amount) {
        // 调用当前策略的支付方法
        if (paymentStrategy != null) {
            paymentStrategy.pay(amount);
        } else {
            System.out.println("请选择支付方式！");
        }
    }
}

// 客户端使用
public class Client {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        
        // 场景1：选择信用卡支付
        cart.setPaymentStrategy(new CreditCardPayment("1234-5678-9012-3456"));
        cart.checkout(100); // 输出: 使用信用卡 1234-5678-9012-3456 支付了 100 元。

        // 场景2：更换为支付宝支付
        cart.setPaymentStrategy(new AlipayPayment("user@example.com"));
        cart.checkout(250); // 输出: 使用支付宝账号 user@example.com 支付了 250 元。
    }
}
```

---

## 三、工厂模式 vs. 策略模式

很多人容易混淆这两个模式，因为它们都涉及接口和多个实现。但它们的**意图（Intent）**完全不同。

| 特性 | 工厂模式 (Factory Pattern) | 策略模式 (Strategy Pattern) |
| :--- | :--- | :--- |
| **模式类型** | 创建型 (Creational) | 行为型 (Behavioral) |
| **核心目的** | **创建对象**。隐藏 `new` 的过程，让客户端无需关心具体类的实例化。 | **选择行为**。封装可互换的算法，让客户端在运行时动态选择行为。 |
| **关注点** | **“谁”来创建“什么” (What to create)**。关注的是对象的诞生过程。 | **“如何”做一件事 (How to do something)**。关注的是一个行为的不同实现方式。 |
| **解决问题** | 解决了**对象创建的耦合**问题。 | 解决了**算法选择的耦合**问题，避免了 `if-else` 地狱。 |
| **最终结果** | 客户端最终**得到一个对象实例**。 | 客户端（上下文）**完成一个行为**，这个行为由内部持有的策略对象来执行。 |
| **使用方式** | 通常是调用一个工厂方法 `createObject()` 来获取一个对象。 | 通常是将一个策略对象注入到上下文对象中 `context.setStrategy(strategy)`，然后调用上下文的方法 `context.execute()`。 |
| **比喻总结** | **造车厂**：你告诉工厂你要什么车，它就造给你。 | **出行方式**：你决定了目的地，然后选择是开车、坐火车还是坐飞机去。 |

## 四、总结

*   **什么时候用工厂模式？**
    *   当你需要创建的对象有共同的父类或接口。
    *   当你不希望客户端代码和具体要创建的对象类耦合在一起时。
    *   你想把创建对象的复杂逻辑封装起来。

*   **什么时候用策略模式？**
    *   当你有很多相关的类，它们只有行为上的区别。
    *   当一个对象有多种行为，并且这些行为可以在运行时动态切换时。
    *   当你需要消除大量的条件判断语句（`if-else`, `switch`）时。

**核心记忆口诀：要 new 对象时，想想工厂；要换 `if-else` 时，想想策略。**