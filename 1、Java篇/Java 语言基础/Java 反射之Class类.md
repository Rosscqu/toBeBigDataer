## Java——反射

### 1 什么是反射

一般操作数据，都是依赖于数据类型，编译器根据类型进行代码对检查进行编译：

- 根据类型使用new创建对象；
- 根据类型定义变量，类型可能是基本类型、类、接口或数组；
- 将特定类型对对象传递给方法；
- 根据类型访问对象对属性，调用对象对方法。

反射是在运行时，动态获取类型的信息，比如接口信息、成员信息、方法信息、构造方法信息等，根据这些动态获取的信息创建对象、访问/修改成员、调用方法等。反射的入口是Class类。

反射主要提供以下功能：

- 在运行时判断任意一个对象所属的类；
- 在运行时构造任意一个类的对象；
- 在运行时判断任意一个类所具有的成员变量和方法（通过反射设置可以调用 private）；
- 在运行时调用一个对象的方法

### 2 反射有什么作用

反射的核心：是 JVM 在运行时才动态加载的类或调用方法或属性，他不需要事先（写代码的时候或编译期）知道运行对象是谁。

反射的作用是在运行时加载类或调用类的方法或属性，利用这个特性可以开发各种通用框架，例如Spring、Jackson等框架。

### 3 Class类

每个已加载的类在内存都有一份类信息，每个对象都有指向它所属类信息的引用。类信息对应的类就是java.lang.Class。

Class是一个泛型类，有一个类型参数，getClass()并不知道具体的类型，所以返回Class<?>。通过<类名>.class获取Class对象。

```java
// 引用类型获取Class对象
Class<Date> dateClass = Date.class;

// 接口获取Class对象
Class<Comparable> comparableClass = Comparable.class;

// 基本类型获取Class对象
Class<Integer> integerClass = int.class;
Class<Byte> byteClass = byte.class;

// void作为特殊类型，也有对应的class
Class<Void> voidClass = void.class;

// 数组类型对应数组类型的Class对象，每个纬度都有一个
String[] strArr = new String[10];
int[][] intArr = new int[3][2];
Class<? extends String[]> strArrCls = strArr.getClass();
Class<? extends int[][]> intArrCls = intArr.getClass();

// 枚举类型对应的class
Class<Size> cls = Size.class;
```

Class有静态方法`forName`，可以根据类名直接加载Class获取Class对象：

```java
try {
    Class<?> objectCls = Class.forName("java.util.HashMap");
    System.out.println(objectCls.getName());
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```

Class对象包含很多关于对象的信息，包括**名称信息、字段信息、方法信息、创建对象和构造方法、类型信息**等。下面分别介绍这些信息。



#### 3.1、名称信息

Class可以获取与名称有关的信息如下：

```java
/**
 * 获取名称相关信息
 */
public void getName() {
    int[][] ints = new int[3][2];
    Class<? extends int[][]> cls = ints.getClass();
    // 获取Java内部使用的真正的名称，输出[[I
    System.out.println(cls.getName());
    // 返回的外部看到的名称，一般为类名，输出int[][]
    System.out.println(cls.getSimpleName());
    // 相对更完整的名称，包名+类名，输出int[][]
    System.out.println(cls.getCanonicalName());
    // 返回包名，输出null
    System.out.println(cls.getPackage());
}
```

与数组类型的`getName()`返回值，使用前缀`[`表示数组，有几个[表示几维数组；数组的类型用一个字符表示：

I表示int；L表示类或接口；Z表示boolean；B表示byte；C表示char；D表示double；F表示float；J表示long；S表示short

#### 3.2、字段信息

类中定义的静态和实例变量都被称为字段，用类`Field`表示。获取`Field`的方法：

```java
/**
 * 获取字段名
 */
public void getField() throws NoSuchFieldException {
    String str = "test";
    Class<? extends String> strCls = str.getClass();
    Class<String> cls = String.class;
    // 返回所有public字段，包括其父类的，如果没有字段则返回空数组
    Field[] fields = cls.getFields();
    // 返回本类的所有字段，包括非public的，但不包括父类
    Field[] declaredField = cls.getDeclaredFields();
    // 返回本类或父类中指定名称的public字段，找不到抛出异常
    Field fieldByName = cls.getField("hash");
    // 返回本类中指定名称的字段，找不到抛异常
    Field declaredFieldByName = cls.getDeclaredField("hash");
    
}
```

`Field`类也有很多方法，可以获取字段的信息，也可以通过`Field`访问和操作指定对象中该字段的值。

```java
try {
    System.out.println("修改前字符串为：" + str);
    // 获取字段名称
    String fieldName = declaredFieldByName.getName();
    // 判断当前程序是否有访问权限
    boolean isAccessible = declaredFieldByName.isAccessible();
    if (! isAccessible) {
        // 更新为true表示忽略Java的访问检查机制
        declaredFieldByName.setAccessible(true);
    }
    // 获取str对象中该字段的值
    Object obj = declaredFieldByName.get(str);
    char[] newValue = "new test".toCharArray();
    // 将str对象的value修改为newValue
    declaredFieldByName.set(str, newValue);
    System.out.println("修改后的字符串为：" + str);
} catch (IllegalAccessException e) {
    e.printStackTrace();
}
```

对于private字段，如果直接使用get/set方法会抛出非法访问异常，所以应该先调用setAccessible方法关闭Java等检查机制。

Field还有几个比较重要的方法：

```java
// 返回字段的修饰符
public int getModifiers()

// 返回字段的类型
public Class<?> getType()

// 查询字段的注解信息，在注解部分会使用
public AnnotatedType getAnnotatedTy
```

#### 3.3、方法信息

与方法一样，类中定义的静态和实例方法都被称为方法，用`Method`类表示。Class类获取Method的方法有：

```java
Class<StringBuffer> cls = StringBuffer.class;
// 获取所有父类和本类的public方法
Method[] methods = cls.getMethods();
// 获取所有本类的方法，包括非public方法
Method[] declaredMethods = cls.getDeclaredMethods();
// 根据方法名获取本类和父类的public方法
Method methodByName = cls.getMethod("toString");
// 根据方法名获取本类的所有方法
Method declaredMethodByName = cls.getDeclaredMethod("toString");
```

Method类可以获取方法的信息，也可以通过Method调用对象的方法。

```java
// 获取方法名
String methodName = declaredMethodByName.getName();
StringBuffer strs = new StringBuffer("test");
// 设置为true表示忽略Java的访问检查机制
declaredMethodByName.setAccessible(true);
// 在指定对象obj上调用Method代表的方法，传递参数列表为args
Object length = declaredMethodByName.invoke(strs);
System.out.println("字符串的长度为：" + Integer.parseInt(length.toString()));
```

对于invoke方法，如果Method为静态方法，obj被忽略，可以为null，args也可以为null，也可以为一个空的数组;

方法调用的返回值被包装为Object。



#### 3.4、创建对象和构造方法

Class类可以创建对象，创建对象的方法有：

- 调用newInstance()方法直接调用默认构造器创建对象；
- 先获取构造方法，构造方法用类Constructor表示，并通过它调用`newInstance(Object...initargs)`方法创建对象

Class类获取Constructor类的方法有：

```java
Class<StringBuffer> cls = StringBuffer.class;
// 获取所有public的构造方法，返回值可能为长度为0的空数组
Constructor<?>[] defaultConstructor = cls.getConstructors();
// 获取所有的构造方法
Constructor<?>[] declaredConstructor = cls.getDeclaredConstructors();
// 获取指定参数类型的public构造方法
Constructor constructorsByName = cls.getConstructor(new Class[]{String.class});
// 获取指定参数类型的所有构造方法
Constructor declaredConstructByName = cls.getDeclaredConstructor(new Class[]{String.class});
```

创建对象的方法有：

```java
// 调用类的默认构造方法，即无参public方法
StringBuffer str1 = cls.newInstance();
// 使用构造类Constructor方法创建类
StringBuffer str2 = (StringBuffer) declaredConstructByName.newInstance("test");
System.out.println("str2 = " + str2);
```

Constructor类还有其他方法，例如获取包括参数、修饰符、注解等。



#### 3.5、类型检查和转换

`instanceOf`可以用来判断对象的类型，但是类型是在代码中确定的。如果要检查的类型是动态的，可以用`isInstance`方法：

```java
try {
    List list = new ArrayList();
    Class cls = Class.forName("java.util.ArrayList");
    if (cls.isInstance(list)) {
        System.out.println("array list");
    }
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```

同样，编写代码时可以强制类型转换，但是如果是动态的，可以利用`Class`的`cast(Object obj)`方法进行类型转换。

isInstance/cast描述的是对象和类之间的关系，Class还有一个方法`isAssignableFrom(Class<?> cls)`，主要用来检查参数类型cls能否赋给当前Class类型。



#### 3.6、Class的类型信息

Class代表的类型既可以是普通的类，也可以是内部类，还可以是基本类型、数组等。

```java
int[] nums = new int[3];
Class cls = nums.getClass();
// 检查是否是数组
System.out.println(cls.isArray());
// 检查是否是基本类型
System.out.println(cls.isPrimitive());
// 检查是否是接口
System.out.println(cls.isInterface());
// 检查是否是枚举类
System.out.println(cls.isEnum());
// 检查是否是注解
System.out.println(cls.isAnnotation());
// 检查是否是匿名类
System.out.println(cls.isAnonymousClass());
// 检查是否是成员类，定义在方法外
System.out.println(cls.isMemberClass());
// 检查是否是本地类，定义在方法内
System.out.println(cls.isLocalClass());
```

#### 3.7、类的声明信息

Class类可以获取类等声明信息，例如修饰符、父类、接口、注解等。

```java
StringBuilder str = new StringBuilder("str");
Class cls = str.getClass();
// 获取修饰符
System.out.println(cls.getModifiers());
// 获取父类
Class superCls = cls.getSuperclass();
// 获取接口
Class[] interfaceCls = cls.getInterfaces();
// 获取所有注解，包括继承得到的
Annotation[] annotations = cls.getAnnotations();
// 获取用户自己声明的注解
Annotation[] declaredAnnotations = cls.getDeclaredAnnotations();
```



#### 3.8、类的加载

类的加载主要通过Class的`forName()`方法加载。

```java
try {
    // 方法1：参数为全限定类名
    Class cls = Class.forName("java.lang.StringBuilder");
    StringBuilder str = (StringBuilder) cls.newInstance();
    // 方法2：参数为全限定类名，initialize表示加载后是否执行类的初始化代码， ClassLoader表示类加载器
    Class<?> cls01 = Class.forName("java.lang.StringBuilder", true, ClassLoader.getSystemClassLoader());
    
    Constructor constructor = cls.getDeclaredConstructor(new Class[]{String.class});
    StringBuilder str2 = (StringBuilder) constructor.newInstance("test");
    System.out.println(str2.toString());
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (InstantiationException e) {
    e.printStackTrace();
} catch (NoSuchMethodException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```

注意：基本类型不支持forName方式。

#### 3.9、数组的反射

对于数组元素，专门通过getComponentType()方法获取它的元素。

```java
int[] arr = new int[10];
System.out.println(arr.getClass().getComponentType());
```

Java.lang.reflect包中有一个针对数组的专门的类Array，提供对于数组的一些反射支持。

```java
int[] arr = new int[10];
Class componentType = arr.getClass().getComponentType();
// 创建指定元素类型、指定长度的一维数组
int[] otherArr = (int[]) Array.newInstance(componentType, 10);
// 创建多维数组
int[][] multiArr = (int[][]) Array.newInstance(componentType, 2,3);
// 给数组赋值
Array.set(otherArr, 0, 5);
// 获取指定索引的值
System.out.println(Array.get(otherArr, 0));
// 获取数组的长度
System.out.println(Array.getLength(otherArr));
```

#### 3.10、枚举类的反射

枚举类型通过`getEnumConstants()`方法获取所有的枚举常量

### 4 了解java.lang.reflect.*

​                                                                                                                                                                                                                                                                             