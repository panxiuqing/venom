---
title: 函数模式匹配
date: 2021-07-22
tags: Elixir, 离散数学
---

# 函数名匹配

一部分语言调用函数时，只能通过函数名进行匹配，比如 Javascript、Python、C 等。在 Javascript 中：

```javascript
// 两个数之和
function sum(a, b) {
  return a + b;
}

// Javascript 中同一个命名空间中定义下面这个方法会覆盖之前的两个数之和的同名方法
// 三个数之和
function sum(a, b, c) {
  return a + b + c;
}

// 所以在 Javascript 中要实现一个方法支持不同参数，需要写成类似下方的代码
function sum(a, b, c) {
  return a + b + (c ?? 0);
}
// or
function sum(a, b, c = 0) {
  return a + b + c;
}
```

## 重载

某些语言更进一步，支持函数名+参数类型/参数数量 进行匹配，比如 Java，这在 Java 中被称为 [函数重载](https://zh.wikipedia.org/wiki/函数重载)，取这个名称只是相对于 _函数名匹配_ 而言（同一个名称不同的参数可以定义不同的方法了）。

```java
class A {
    public Integer sum(Integer a, Integer b) {
        return a + b;
    }

    public Double sum(Double a, Double b) {
        return a + b;
    }

    public Integer sum(Integer a, Integer b, Integer c) {
        return a + b + c;
    }
}
```

静态类型语言的函数重载，在编译期即可知道确切的调用方法。

## 函数签名

Objective C 和 Java 类似，只不过是通过参数标签来区分。

Swift 在 Objective C 的基础上增加了参数类型的支持，所以可以视为 Java + Objective C 的结合。

其中 Objective C 和 Swift 称之为函数签名。

函数签名的优势在于函数的意义不再需要一个很长的函数名（在 Java 中经常用来表示各个参数的含义），使用过 JPA 的朋友应该有写过类似代码：

```java
interface A {
    findByIdAndNameAndAge(Integer id, String name, Integer age);
}
```

当然这个举例并不严谨，其中的 `And` 是有查询语义的， JPA 可以据此生成对应的数据库查询。但是如果可以类似这样表达:

```java
// 调用时： A.find(id: 1, name: "test", age: 18)
interface A {
    find(Integer id: id, Integer name: name, Integer age: age);
}
```

私以为是更好的表达方式。

接下来就是最丧心病狂的 **函数模式匹配** 了。

## 模式匹配

比如 Elixir，支持任意数据结构的匹配：map、tuple、list、struct、binary 等：

```elixir
defmodule A do
    # 数值参数
    # A.sum(1, 2)
    def sum(a, b) do
        a + b
    end

    # map
    # A.sum(%{a: 1, b: 2})
    def sum(%{a: a, b: b}) do
        a + b
    end

    # tuple
    # A.sum({1, 2})
    def sum({a, b}) do
        a + b
    end

    # list
    # A.sum([1, 2])
    def sum([a, b]) do
        a + b
    end
end

```

还可以支持值的匹配:

```elixir
defmodule A do
    # 只会在第二个参数为 1 的时候被调用
    def sum(a, 1) do
        a + 1
    end

    # 只会在 b 为 1 的时候被调用
    def sum(%{a: a, b: 1}) do
        a + 1
    end
end
```

在此基础上还支持 guard，带条件的匹配，即子集匹配:

```elixir
defmodule A do
    def sum(a, b) when b < 5 do
        a + b
    end

    def sum(a, b) when b > 10 do
        a + b
    end
end
```

上面的表达就像中学时的分段函数。

从离散数学的角度来看，把函数视为一个集合 P（参数）到另一个集合 R（返回值）的映射关系，
那么 Elixir 的函数模式匹配就是 **声明式地描述 P 的任意子集 Pn**，并单独编写 Pn 到 R 的映射关系。

在没有模式匹配的语言中，**Pn** 可以通过控制逻辑（比如条件判断语句）来分离：

```java
class A {
    public int sum(int a, int b) {
        if (b < 5) {
            //
        } else if (b < 10) {
            //
        } else {
            //
        }
    }
}
```

这样存在的一个问题是当逻辑复杂的时候，不同条件下的代码块如果需要拆分出不同的函数，就有了取名困难症：

```java
class A {
    public int sum(int a, int b) {
        if (b < 5) {
            sumWhenBLessThanFive(a, b);
        } else if (b > 10) {
            sumWhenBMoreThanTen(a, b);
        } else {
            //
        }
    }

    // 不知道如何取合适的函数名
    private int sumWhenBLessThanFive(int a, int b) {

    }

    private int sumWhenBMoreThanTen(int a, int b) {

    }
}
```

## 总结

函数模式匹配很强大，因为其基于数学理论进行设计。其他语言中的 重载、函数签名 等概念都是对先前语言设计的一种修补。

函数模式匹配的弊端是不可避免的需要在运行时进行匹配；
当然回过头来看，需要在运行时进行的匹配，其他语言也得在运行时判断，只不过可以更方便地明确哪些路径是编译期确定，哪些是在运行时。
所以说还是那句话：所有技术都是 trade-off，我们要权衡哪些留给语言实现者，哪些留给语言使用者。
