# Spring学习——IOC&DI学习笔记

## SOLID：面向对象设计的五个基本原则
- S：单一职责原则(Single-Responsibility Principle)
    - 定义：对于一个类而言，应该只有一个能够引起它变化的原因，也就是一个类只做一件事
    - 为什么要遵循SPR：
        - 减少类之间的耦合：当需求变化的时候，只修改一个类，从而隔离了变化
        - 提高类的复用性
- O：开放-封闭原则(Open-Closed Principle)
    - Open for extension：模块的行为是开放的，可扩展的
    - Closed for modification：模块是不可修改的
    - 实现开放封闭原则的核心就是抽象编程，让类依赖于固定的抽象类，所以修改就是封闭的，通过面向对象的继承和多态，可以实现抽象类的继承，所以是可扩展的，开放的
- L：里氏替换原则(Liskov-Substitution Principle)
    - 定义：子类必须能够替换成它们的父类，并且能够出现在父类能够出现的任何地方，但是父类不一定能替换子类
    - 如何遵循里氏替换原则：
        - 父类的方法都要在子类中实现或者重写，派生类只实现其抽象类中声明的方法，而不应当给出多余的方法定义或实现
        - 在程序中应当只出现父类对象，而不是直接使用子类对象，这样可以实现运行期绑定(多态绑定)
    - 违反了里氏替换原则，就必然会违反开放封闭原则
- I：接口隔离原则(Interface-Segregation Principle)
    - 定义：避免使用胖接口，应该把胖接口中的方法分组，用多个接口替代它，每个接口服务于一个子模块，也就是说，一个模块不应该依赖于它不需要的接口方法，一个类对另一个类的依赖性应该是建立在最小的接口上的
    - 如果能够隔离接口，那么就不会发生接口污染
    - 接口污染：比如模块A依赖于方法A，模块B依赖于方法B，模块C依赖于方法C，而此时提供服务的抽象类同时包含ABC三个方法，那么模块ABC就同时依赖于同一个服务提供类，如果模块A的服务方法A需要发生改变，此时BC模块也会受到影响
    - 利用委托和多继承分离接口
- D：依赖倒置原则(Dependency-Inversion Principle)
    - 定义：上层模块不应该依赖于下层模块，它们共同依赖于一个抽象；抽象不能依赖与具体，具体应该依赖于抽象
    - 面向接口编程，而不是面向实现编程
    - 依赖倒置的核心原则就是解耦，SpringIoC容器的核心就是解耦和加载依赖

---

## 工厂方法模式
- 简单工厂模式：负责生产对象的一个类成为工厂类(定义了静态方法创建产品实例)
    - 解决的问题：把类的实例化操作和类的使用操作分离，让使用者不需要知道具体参数就可以实例化出所需要的产品类，避免在客户端中显示指定，实现了解耦
    - 具体实例：
        ```
        package org.Pattern;

        abstract class Product{
            public abstract void productShow();
        }

        class ProductA extends Product{
            @Override
            public void productShow(){
                System.out.println("Product A has been created");
            }
        }

        class ProductB extends Product{
            @Override
            public void productShow(){
                System.out.println("Product B has been created");
            }
        }

        class SimpleFactory{
            public static Product createProduct(String productName){
                switch (productName){
                    case "A":
                        return new ProductA();
                    case "B":
                        return new ProductB();
                    default:
                        return null;
                }
            }
        }

        public class SimpleFactoryPattern {
            public static void main(String[] args) {
                SimpleFactory simpleFactory=new SimpleFactory();
                try{
                    simpleFactory.createProduct("A").productShow();
                }catch (NullPointerException e){
                    System.out.println("There is no product A");
                }

                try{
                    simpleFactory.createProduct("B").productShow();
                }catch (NullPointerException e){
                    System.out.println("There is no product B");
                }

                try{
                    simpleFactory.createProduct("C").productShow();
                }catch (NullPointerException e){
                    System.out.println("There is no product C");
                }
            }
        }

        ```
    - 问题：
        - 工厂类集中了所有实例的创建，一旦不能工作整个系统都会瘫痪
        - 违背了开放关闭原则，一旦添加新产品就要修改工厂类的逻辑
        - 采用静态方法创建实例，无法被继承和重写
- 工厂方法模式

---

## IoC容器
- 容器：为某种特定组件的运行提供必要支持的一个软件环境
- IoC：Inversion of Control 控制反转
