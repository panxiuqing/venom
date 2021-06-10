---
title: 依赖注入/缘起和实践
date: 2021-05-29
tags: Java, OOP
---

依赖注入（dependency injection）在现代 OOP 语言框架中是一种常用的设计模式实现，本文尝试解释使用依赖注入的原由和最佳实践。

为了一段逻辑能够被复用，我们会定义一个函数:

```java
class SumClass {
    // 对两个整数求和
    sum(Integer a, Integer b) {
        return a + b;
    }
}
```

如果想要把这个函数应用在不同的数据（有共性）上，或者和其他一段逻辑进行交互，我们提出了面向接口编程，只要实现了一组接口，使用这组接口的逻辑就能得到复用:

```java
interface Param {
    Integer getA();
    Integer getB();
}

class SumClass {
    // 实现了 Param 接口的对象可以传给 sum 复用逻辑
    sum(Param param) {
        return param.getA() + param.getB();
    }
}
```

虽然是面向接口，实际中传递给复用逻辑的依旧是一个实现了该接口的类实例：

```java

class ParamClass implements Param {
    private Integer a;
    private Integer b;

    public ParamClass(Integer a, Integer b) {
        this.a = a;
        this.b = b;
    }

    public Integer getA() {
        return a;
    }

    public Integer getB() {
        return b;
    }
}

class SumClass {
    sum(Param param) {
        return param.getA() + param.getB();
    }

    main() {
        sum(new ParamClass(1, 2));
    }
}
```

到这里，我们已经在使用依赖注入了，但是这里就引出了一个新的问题，**实例化树**，简单举例：

```java

interface Calculator {
    integer run(Param param);
}

class SumClass implements Calculator {
    run(Param param) {
        return param.getA() + param.getB();
    }
}

class MulClass implements Calculator {
    run(Param param) {
        return param.getA() * param.getB();
    }
}

class Demo {
    run(Calculator calculator, Param param) {
        calculator.run(param);
    }

    main() {
        Param param = new ParamClass(1, 2);
        this.run(new SumClass(), param);
        this.run(new MulClass(), param);
    }
}

```

调用 `Demo::run` 之前需要先实例化所有相关的类，可以想象，如果这是一个庞大的系统，每一个接口依赖其他接口，调用一个方法前，需要实例化一颗依赖树，所以系统设计是美好了，但是实际使用相关接口时的问题还没有解决。

同时实例本身也可能依赖其他接口，如果在构造函数中自行初始化依赖，这里的依赖均没法面向接口，因为初始化时只能使用具体的类。把依赖通过构造函数的参数传入可以解决这个问题。到目前为止，局部来看代码都已经面向接口了。

现在要面对的问题是：调用一个方法前或者实例化一个类前，要手动进行它的依赖树的实例化。

容器和代理，通过注解自动实例化类，然后通过注解注入依赖。

到了这一步，会有人疑惑那为什么不用静态方法呢？因为所谓的面向对象的类基本沦为代码命名空间。
一方面是因为单元测试，替换某一个接口的实现方便进行单元测试。
另一方面在不同的情况下可以选择多例或单例。
