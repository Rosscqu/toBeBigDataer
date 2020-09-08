## Java ——泛型从入门到放弃

泛型实现参数化类型，这样编写的组件（通常是集合）可以适用于多种类型，泛型的初衷是解耦类或方法与所使用的类型，而是只关注类或方法的能力。促成泛型出现的最主要动机就是为了创建集合类，所以在集合类中大量使用泛型。

### 1、定义泛型类

在了解如何定义泛型之前，我们看一下java1.5之前是如何定义的：

```java
public class Pair_1 {
    Object first;

    Object second;

    public Pair_1(Object first, Object second) {
        this.first = first;
        this.second = second;
    }

    public Object getFirst() {
        return first;
    }

    public Object getSecond() {
        return second;
    }
}
```

这种方式有个缺点：没有对字段类型进行约束，这两个字段可以是任意类型，加入定义一个List<Pair_1>是列表，那么这个列表每个元素的first和second的类型都不相同。所以泛型的目的是为了约束类型，并且通过编译器确保规约得以满足。

泛型使用字符T、U、V、E等来表示类型，并将其放入尖括号<>中，具体使用如下：

```java
public class Pair_2<U, V> {

    U first;

    V second;

    public Pair_2(U first, V second) {
        this.first = first;
        this.second = second;
    }

    public U getFirst() {
        return first;
    }

    public V getSecond() {
        return second;
    }
}
```

在使用该类时，用实际的类型替换尖括号的符号即可：

```java
Pair_2<String, String> pair2 = new Pair_2<>("我是", "泛型");
System.out.println(pair2.getFirst() + pair2.getSecond());
```

泛型的核心概念是：只需要告诉编译器要使用什么类型，剩下的细节交给他处理。但是Java内部实现泛型是什么原理呢？答案就是**类型擦除**。

我们将使用javap命令编译Pair_2.class文件，节选部分：

```
U first;
    descriptor: Ljava/lang/Object;
    flags:
    Signature: #9                           // TU;

  V second;
    descriptor: Ljava/lang/Object;
    flags:
    Signature: #11                          // TV;
```

从编译的字节码可以看出，first和second编译后都是Object类，这其实和Pair_1类一样，将类型参数U、V擦除，替换为Object，并插入必要的强制类型转换。因为泛型是Java 5以后才出现，所以Java这样设计也是为了兼容性。



### 2、定义泛型接口

既然可以定义泛型类，当然也可以定义泛型接口，也是将泛型符号放在接口名后。例如Comparable和Comparator接口都是泛型的，

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

String类就继承了Comparable接口，表示只能用于String类型的比较。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
    
    ...
 }
```

### 3、定义泛型方法

方法是类的主要组成部分，除了泛型类，方法也可以是泛型的。但是一个方法是不是泛型的与其类是不是泛型没有关系。

```java
public class GenericMethod {
    
    public static <T> int indexOf(T[] arr, T elm) {
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] == elm) {
                return i;
            }
        }
        
        return -1;
    }
}
```

从代码中可以看出，并没有把类定义为泛型类，但是indexOf()方法却是泛型的。泛型方法将类型参数T放在返回值前面即可。

在使用上，泛型方法与泛型类不同，调用方法时一般不需要特意指定类型参数的实际类型，编译器会自动推断出参数类型。例如：

```java
String[] arr = {"a", "b", "c"};
int index = indexOf(arr, "b");
System.out.println(index);   // 输出1
```

《Java编程思想》建议尽可能使用泛型方法，

 

### 4、类型参数的限定



类型上界通过extends关键字表示，即参数必须为给定的上界类型或其子类型。这个上界可以是

1）某个具体的类

```java
public class NumberPair<U extends Number, V extends Number>{

    U first;
    V second;

    public NumberPair(U first, V second) {
        this.first = first;
        this.second = second;
    }

    public U getFirst() {
        return first;
    }

    public V getSecond() {
        return second;
    }
}
```

对该类编译以后：

```
  U first;
    descriptor: Ljava/lang/Number;
    flags:
    Signature: #9                           // TU;

  V second;
    descriptor: Ljava/lang/Number;
    flags:
    Signature: #11                          // TV;
```

对比Pair_2与NumberPair编译的结果可以看到，Pair_2字段的descriptor为Object，NumberPair类字段的descriptor为Number，说明限定边界后，类型擦除时就不会转换为Object，而是转换为它的边界类型。

2）某个具体的接口

3）其他类型参数

写法：

```
<T extends E>：用于定义类型参数，声明一个类型参数T，可以放在泛型类定义中类名后面、泛型方法返回值前面；

<? extends E>：用于实例化类型参数，用于实例化泛型变量中的类型参数，只是这个具体类型是未知的，只知道它是E或E的某个子类型。
```

在上面用到的`?`表示通配符。下一节将详细解释通配符。

### 5、解析通配符

形如`<? extends E>`称为有限通配符，如果没有限定上界称为无限定通配符，例如`List<?>`。无限定通配符形式也可以改为使用类型参数，例如：

```java
public void indexOf(ArrayList<?> list, Object elm)

public <T> void indexOf(ArrayList<T> list, Object elm)
```

使用通配符的优点是形式简洁，但是有一个重要限制：**只能读不能写**。因为问号表示类型安全无知，`? Extends Number`表示是Number的某个子类型，但不知道具体子类型，如果允许写入则无法保证类型安全性，所以干脆禁止。那么泛型方法到底应该使用通配符还是类型参数，结论如下：

- 通配符形式都可以用类型参数的形式替代；
- 通配符形式可以减少类型参数，形式上更简单，可读性也更好，所以能用通配符就用通配符；
- 如果类型参数之间有依赖关系，或者返回值依赖类型关系，或者需要写操作，则只能用类型参数；
- 通配符形式和类型参数往往配合使用

还有一种通配符：`<? super E>`，称为超类型通配符，表示E的某个父类型。这种类型可以更灵活的写入。但是这种类型无法用类型参数代替，即不能写成`<T super E>`.

总结来说：

- `<? super E>` 用于灵活写入或比较，使得对象可以写入父类型的容器，使得父类型的比较方法可以应用于子类对象，且不能被类型参数替代；
- `<? extends E>`用于灵活读取，使得方法可以读取E或E的仁义子类型的容器对象，可以用类型参数的形式替代，但通配符形式更简洁。