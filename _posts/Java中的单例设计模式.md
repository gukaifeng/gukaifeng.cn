
**设计模式**是在大量的实践中总结和理论化之后优选的代码结构、编程风格、以及解决问题的思考方式。

**单例设计模式**就是采取一定的方法保证在整个的软件系统中，对某个类智能存在一个对象实例，并且该类只提供一个取得其对象实例的方法。

如果我们要让类在一个虚拟机中只能产生一个对象，我们首先必须将类构造方法的访问权限设置为 private，这样，就不能用 new 操作符在类的外部产生类的对象了，只能调用该类的某个静态方法以返回内部创建的对象，静态方法只能访问类中的静态成员变量，所以指向类内部产生的该类对象的变量也必须定义成静态的。

单例设计模式 的实现方式主要有饿汉式和懒汉式。

<!--more-->

1. 饿汉式

    ```java
    class Single {
        // private 构造器，不能在类的外部创建该类的对象
        private Single() {}
        // 私有的，只能在类的内部访问
        private static Single onlyone = new Single();
        // getSingle() 为 static，不用创建对象即可访问
        public static Single getSingle() {
            return onlyone;
      }
    }
    ```

2. 懒汉式

    ```java
    class Singleton {
        // private 构造器
        private Singleton() {}
        // 用 static 修饰，声明此类的引用
        private static Singleton instance = null;
        // 设置公共的方法来访问类的实例
        // 若实例已存在，返回实例；否贼新建实例并返回
        public static Singleton getSingleton() {
            if (instance == null)
              instance = new Singleton();
            return instance;
        }
    }
    ```

    暂时懒汉式存在线程安全问题，讲到多线程时，可修复。