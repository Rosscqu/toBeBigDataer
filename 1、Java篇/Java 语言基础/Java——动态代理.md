## Java——动态代理

动态代理可以在运行时动态创建一个类，实现一个或多个接口，可以在不修改原有类的基础上动态为通过该类获取对象添加方法、修改行为。

动态代理是实现面向切面的编程AOP的基础。切面的例子有日志、性能监控、权限检查、数据库事务等。动态代理有2种实现方式：1）Java SDK实现；2）第三方库（cglib）提供。

了解动态代理之前，首先看一下什么是静态代理。

### 1 静态代理

代理与《设计模式》中的代理模式概念类似，代理背后至少有一个实际对象，代理的外部功能和实际对象一般是一样的，用户与代理打交道，不直接接触实际对象。

静态代理是指在代码中创建代理类，它的代码是在写程序时固定的。

```java
/**
 * @Description: 静态代理demo
 * @Author: ross
 **/
public class StaticProxyDemo {

    static interface IService {

        public void sayHello();
    }

    static class RealService implements IService {

        public void sayHello() {
            System.out.println("hello");
        }
    }

    static class ProxyService implements IService {

        private IService service;

        public ProxyService(IService service) {
            this.service = service;
        }

        public void sayHello() {
            System.out.println("Entering sayHello");
            service.sayHello();
            System.out.println("Leaving sayHello");
        }
    }

    public static void main(String[] args) {
        System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true"); //设置系统属性
        IService service = new RealService();
        IService proxyService = new ProxyService(service);
        proxyService.sayHello();
    }
}
```

代理模式的特点：

1）代理与实际对象一般有相同的接口interface；

2）代理类内部有个interface的成员变量，指向实际对象，并且在构造方法中被初始化。

静态代理的缺点：

代理类中实现的是通用功能，如果多个接口都要实现这个通用功能，就需要为每个类创建代理，实现所有的接口。这样工作比较繁琐。如果又新增其他通用功能，又需要重新实现一遍。



### 2 动态代理



#### 2.1 什么是动态代理



#### 2.2 动态代理与反射的关系



#### 2.3 动态代理的实现方式



##### 2.3.1 Java SDK动态代理

在动态代理中，代理类是动态生成的。

```java
/**
 * @Description:使用Java SDK实现动态代理
 * @Author: ross
 **/
public class JDKDynamicProxyDemo {

    static interface IService {
        public void sayHello(String message);
    }

    static class RealService implements IService {

        public void sayHello(String message) {
            System.out.println("hello " + message);
        }
    }

    static class SimpleInvocationHandler implements InvocationHandler {

        private Object realObj;

        public SimpleInvocationHandler(Object realObj) {
            this.realObj = realObj;
        }

        /**
         *
         * @param proxy 表示代理对象本身，一般不用这个参数
         * @param method 表示正在被调用对方法
         * @param args 方法对参数
         * @return
         * @throws Throwable
         */
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("Entering " + method.getName());
            Object result = method.invoke(realObj, args);
            System.out.println("Leaving " + method.getName());
            return result;
        }
    }

    public static void main(String[] args) {
        IService realService = new RealService();
        /**
         * 1) loader表示类加载器
         * 2）代理类要实现的接口列表
         * 3）InvocationHandler接口，只定义类一个方法invoke，对代理接口所有对调用方法都会转给该方法
         */
        IService proxyService = (IService) Proxy.newProxyInstance(IService.class.getClassLoader(),
                new Class<?>[]{IService.class}, new SimpleInvocationHandler(realService));
        proxyService.sayHello("动态代理");
    }
}
```



##### 2.3.2 cglib动态代理

添加maven依赖：

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

基于cglib的简单实现：

```java
public class CGLibProxy implements MethodInterceptor {
    private Object obj;

    public CGLibProxy(Object obj) {
        this.obj = obj;
    }

    public Object createProxy() {
        Object realObj = Enhancer.create(obj.getClass(), this);
        return realObj;
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("代理前");
        Object res = methodProxy.invokeSuper(o, objects);
        System.out.println("代理后");
        return res;
    }

    public static void main(String[] args) {
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "java_workapace/");
        UserDaoImpl userDao = new UserDaoImpl();
        UserDaoImpl proxy = (UserDaoImpl) new CGLibProxy(userDao).createProxy();
        proxy.getAge("a");
    }
}
```



### 3 利用动态代理实现AOP

