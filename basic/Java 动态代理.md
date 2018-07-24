# Java 动态代理

+ 概述
+ 代理
+ 实现

## 概述

代理本质上就是通过一个中间类（代理类）去调用类（我们需要调用的方法所在类）的方法。*** 通过代理类我们能在调用方法前后添加一些需要的代码,这就是 AOC 的实现 ***

## 代理

代理分为静态代理与动态代理。

静态代理 ： 在编译阶段就生成了代理类的课执行文件
动态代理 ： 在运行时创建代理类

## 实现

InvocationHandler 接口 、 Proxy 类。

接口 Test.java

```java
public interface Test {
    public void test();
}
```

实现类 TestIpm.java

```java
public class TestIpm implements Test {

    @Override
    public void test() {
        System.out.println("TestIpm test()");
    }
}
```

代理类 ProxyTest.java

```java
public class ProxyTest implements Test,InvocationHandler {
    Test target;

    public ProxyTest(Test target){
        this.target = target;
    }

    @Override
    public void test() {
        System.out.println("ProxyTest Test");
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("ProxyTest started");
        Object result = method.invoke(target,args);
        System.out.println("proxy end");
        return result;
    }
}
```

main.java

```java
public static void main(String[] args) {
        Test test = new TestIpm();
        InvocationHandler proxy = new ProxyTest(test);

        Test testP = (Test) Proxy.newProxyInstance(Test.class.getClassLoader(),new Class<?>[]{Test.class},proxy);

        testP.test();

    }
```

运行结果:

```
ProxyTest started
SubTest test()
proxy end
```