
`Proxy` ：专门完成代理的操作类，是所有动态代理类的父类。通过此类为一个或多个接口动态地生成实现类。

## 1. 创建一个动态代理类所对应的 Class 对象

```java
static Object newProxyInstance(ClassLoader loader, Class<?> interfaces, InvocationHandler h) // 直接创建一个动态代理对象
```

## 2. 动态代理的步骤

1. 创建一个实现接口 `InvocationHandler` 的类，实现 invoke() 方法；
2. 创建一个接口，和一个实现了这个接口的，准备被代理的类；
3. 通过 Proxy 的静态方法 `static Object newProxyInstance(ClassLoader loader, Class<?> interfaces, InvocationHandler h)` 创建一个 Subject 接口代理；
4. 通过 Subject 代理调用 RealSubject 实现类的方法。



<!--more-->

### 3. 动态代理代码示例



```java
// 接口
package Test2;

public interface testInterface {
    public void test1();
    public void test2();
}
```

```java
// 接口的实现类
package Test2;

public class Student implements testInterface {
    @Override
    public void test1() {
        System.out.println("this is test1() method.");
    }
    @Override
    public void test2() {
        System.out.println("this is test2() method.");
    }
}
```

```java
// 代理类
package Test2;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class testProxy implements InvocationHandler {   // 代理类要实现 InvocationHandler 接口
    public testProxy(Object obj) {
        this.obj = obj;
    }
    Object obj; // 要代理的类

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method invocation."); // 执行被代理对象方法之前的操作
        Object res = method.invoke(obj); // 执行被代理对象的方法
        System.out.println("After method invocation."); // 执行被代理对象方法之后的操作

        return res; // 返回被代理对象方法的返回值，没有返回值就为 null
    }
}
```

```java
// 主程序
package Test2;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class testMain {
    public static void main(String[] args) {
        testInterface t1 = new Student(); // 用接口类接受 Student 对象
        t1.test1();
        t1.test2();
        System.out.println();

        InvocationHandler handler = new testProxy(t1);  // 创建一个代理对象


        // 参数1：代理对象的类加载器。
        // 参数2：被代理的对象的接口。
        // 参数3：代理对象。
        // 返回值：一个成功被代理后的对象。
        testInterface t2 = (testInterface)Proxy.newProxyInstance(handler.getClass().getClassLoader(), t1.getClass().getInterfaces(), handler);
        t2.test1();
        System.out.println(".................................");
        t2.test2();
    }
}
```

```shell
// 输出
this is test1() method.
this is test2() method.

Before method invocation.
this is test1() method.
After method invocation.
.................................
Before method invocation.
this is test2() method.
After method invocation.
```

注：

一个对象如果想要通过 `Proxy.newProxyInstance()` 方法被代理，那么这个类一定要有实现至少一个的接口（动态代理貌似是通过接口间接实现的）。例如本例中的接口 testInterface 和其实现类 testStudent。