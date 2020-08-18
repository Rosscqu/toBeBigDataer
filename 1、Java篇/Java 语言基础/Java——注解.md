## Java——注解

注解就是给程序添加一些信息，用字符`@`开头，这些信息用于修饰它后面紧挨着的其他代码元素，例如**类、接口、字段、方法、方法中的参数、构造方法**等。注解可以被编译器、程序运行时和其他工具使用，用于增强或修改程序行为等。

### 1 元注解

元注解：@Target和@Retention

- @Target：表示注解的目标，值是ElementType枚举类，枚举值有：
  - ​	TYPE：表示类、接口，或者枚举声明；
  - ​	FIELD：字段，包括枚举常量；
  - ​	METHOD：方法；
  - ​	PARAMETER：方法中的参数；
  - ​	CONSTRUCTOR：构造方法；
  - ​	LOCAL_VARIABLE：本地变量；
  - ​	MODULE：模块

- @Retention表示注解信息保留到什么时候，取值只能有一个，类型为RetentionPolicy，枚举值有：
  - ​	SOURCE：只在源代码中保留，编译器将代码编译为字节码文件后就会丢掉；
  - ​	CLASS：保留到字节码文件中，但虚拟机将class文件加载到内存时不一定会在内存中保留；
  - ​	RUNTIME：一直保留到运行时

### 2 Java内置注解

1）@Override

作用：表示该方法不是当前类首先声明的，而是在某个父类或实现的接口中声明的，当前类重写这个方法。

修饰范围：方法

```java
/**
 * @Description: 重写toString()方法
 * @Author: ross
 **/
public class OverrideDemo {
    
    private Integer type;
    
    private String info;
    
    public OverrideDemo(Integer type, String info) {
        this.type = type;
        this.info = info;
    }

    @Override
    public String toString() {
        return "type:" + this.type + " info: " + this.info;
    }
}
```

注意：如果方法有Override注解，但没有任何父类或实现的接口声明该方法，则编译器会报错，强制程序员修复该问题；如果没有加Override注解，编译器不会报任何错误。

2）@Deprecated

作用：表示对应的代码已经过时，程序不应该使用它；

修饰范围：类、方法、字段、参数等

例如Date类中存在的过时方法：

```java
@Deprecated
public Date(int year, int month, int date, int hrs, int min) {
    this(year, month, date, hrs, min, 0);
}
```

注意：从Java9开始，@Deprecated多了2个属性：since和forRemoval。

since是一个字符串，表示从哪个版本开始过时；

forRemoval是一个boolean值，表示将来是否会删除

3）@SuppressWarnings

作用：表示压制Java的编译告警，它有一个必填参数表示压制哪种类型的警告。

修饰范围：可以修饰大部分代码元素



### 3 创建注解

注解定义：使用关键字@interface，并添加元注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

从源码可以看出，@Override是给编译器使用的，主要用来修饰方法。

注解内可以定义一些参数，定义的方式是在注解内定义一些方法。注解内参数的类型有基本类型、String、Class、枚举、注解，以及这些类型的数组。

使用`@Inherited`注解的类，则这个类的子类都能继承这个注解。



### 4 利用反射查看注解信息





### 5 Spring常用注解