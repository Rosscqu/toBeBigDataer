## JVM篇——字节码、类加载器和JVM内存模型



### 1、字节码技术



#### 1.1 字节码介绍

Java字节码由单字节的指令组成，理论上最多支持256个操作码。实际只使用了200左右的操作码，还有一些操作码保留调试使用。

Java字节码组成：





编译指令：

- javac xxx.java

查看字节码指令：

- 简单字节码：Javap -c xxx.class
- 详细的字节码：javap -c -verbose xxx.class

 

字节码的运行时结构：

JVM是一台基于栈的计算机器。每个线程都有一个属于自己的线程栈（JVM stack），用于存储栈帧。每一个方法调用，JVM都会自动创建一个栈帧。栈帧由操作数栈、局部变量数组以及一个Class引用组成。Class引用指向当前方法在运行时常量池中对应的Class。



#### 1.2 常见字节码

##### 1.2.1 数值处理相关字节码

- load：将本地变量表写到栈上

- store：将栈上的值写到本地变量表中

- 常量：const

- 类型转换：i2d（int类型转换为double）

  ![image-20210110220842673](/Users/chengxi/Library/Application Support/typora-user-images/image-20210110220842673.png)

- 算数操作

  ![image-20210110220828676](/Users/chengxi/Library/Application Support/typora-user-images/image-20210110220828676.png)

  



##### 1.2.2 程序运行相关字节码

循环控制：

- if_icmpge：比较
- iinc:自增
- goto：跳转

方法调用指令

- invokestatic:调用静态方法
- invokespecial:调用构造函数，也可以调用同一个类中的private方法以及一个可见的超类方法；
- invokevirtual:如果是具体类型的目标对象，用于调用public、protect和package级的私有方法；
- invokeinterface:通过接口引用调用方法；
- invokdynamic：JDK7新增指令，实现动态类型语言，也是JDK8支持lambda表达式的实现基础。



### 2、类加载器

#### 2.1 类的生命周期

1）加载：找Class文件

类加载的时机有：

- 当虚拟机启动时，初始化用户指定的主类；
- new一个类时要初始化；
- 遇到静态方法、静态字段时，需要初始化静态方法、静态字段所在的类；
- 子类初始化会触发父类的加载；
- 当使用反射API加载类时；
- 当初次调用MethodHandle实例时，会初始化该MethodHandle指向的方法所在的类



2）链接

- 验证：验证格式、依赖
- 准备：解析字段、方法表
- 解析：符号解析为引用

3）初始化：构造器、静态变量赋值、静态代码块

不会初始化：

- 通过子类引用父类的静态字段，只会触发父类的初始化，不会触发子类的初始化；
- 定义对象数组，不会触发该类的初始化；
- 常量在编译期间会存入调用类的常量池，本质上没有直接引用定义常量的类，不会触发定义常量所在的类；
- 通过类名获取Class对象，不会触发类的初始化：Hello.class不会让Hello类初始化；
- 通过Class.forName加载指定类时，如果指定参数initialize为false，不会触发类初始化；
- 通过ClassLoader默认的loadClass方法，也不会触发类的初始化

4）使用

5）卸载



#### 2.2 类加载器

三类类加载器：

- 启动类加载器（BootstrapClassLoader）
- 扩展类加载器（ExtClassLoader）
- 应用类加载器（AppClassLoader）



类加载器如何避免类被重复加载：

- 双亲委托：
  - 加载类时会首先查看扩展类加载器是否加载该类，如果没有则查看启动类加载器是否加载该类；
  - 如果都没有加载该类，则启动类加载器尝试加载，如果加载失败则扩展类尝试加载；
  - 如果启动类加载器和扩展类加载器都没有加载该类，则应用类加载器加载该类。
- 负责依赖：加载一个类，负责加载这个类所依赖的类
- 缓存加载：加载后的类会进行缓存



2.2.1 查看启动类加载器下的加载的类



```java
public class ShowClassLoader {

    public static void main(String[] args) {
        URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();
        System.out.println("启动类加载器");
        for (URL url : urls) {
            System.out.println("==>" + url.toExternalForm());
        }
        printClassLoader("扩展类加载器", ShowClassLoader.class.getClassLoader().getParent());

        printClassLoader("应用类加载器", ShowClassLoader.class.getClassLoader());


    }

    private static void printClassLoader(String name, ClassLoader classLoader) {
        if (classLoader != null) {
            System.out.println(name + "classloader==>" + classLoader.toString());
            printURLForClassLoader(classLoader);
        } else {
            System.out.println(name + "classloader==>" + classLoader.toString());
        }
    }

    private static void printURLForClassLoader(ClassLoader classLoader) {
        Object ucp = insightField(classLoader, "ucp");
        Object path = insightField(ucp, "path");
        ArrayList ps = (ArrayList) path;
        for (Object o : ps) {
            System.out.println(" ==>" + o.toString());
        }
    }

    private static Object insightField(Object obj, String fName) {
        try {
            Field field = null;
            if (obj instanceof URLClassLoader) {
                field = URLClassLoader.class.getDeclaredField(fName);
            } else {
                field = obj.getClass().getDeclaredField(fName);
            }
            field.setAccessible(true);
            return field.get(obj);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
            return null;
        }

    }
}
```



2.2.2 自定义类加载器



2.2.3 添加引用类

- 放在JDK的lib/ext下，或者-Djava.ext.dirs
- java -cp/classpath或者class文件放在当前路径
- 自定义ClassLoader加载
- 拿到当前执行类的ClassLoader，反射调用addUrl方法添加Jar或路径（JDK9无效）





### 3、JVM内存模型



- 