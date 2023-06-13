
Java Reflection：Reflection（反射）是被视为动态语言的关键，反射机制允许程序在执行期借助于 Reflection API 取得任何类的内部信息，并能直接操作任何对象的内部属性及方法。



## 1. Java 反射机制提供的功能

* 在运行时判断任意一个对象所属的类；
* 在运行时构造任意一个类的对象；
* 在运行时判断任意一个类所具有的成员变量和方法；
* 在运行时调用任意一个对象的成员变量和方法；
* **生成动态代理**。



## 2. Java 反射相关的主要 API

* java.lang.Class：代表一个类；
* java.lang.reflect.Method：代表类的方法；
* java.lang.reflect.Field：代表类的成员变量；
* java.lang.reflect.Constructor：代表类的构造方法。
* ...



<!--more-->

## 3. Class 类

在 Object 类中定义了以下的方法，此方法将被所有子类继承；

```java
public final Class getClass()
```

以上方法返回值的类型是一个 Class 类，此类是 Java 反射的源头。

```java
// 假设有一个类
public class Person() {
    String name;
    int age;
}
```

```java
public class Test {
    public static void main(String[] args) {
        Person p = new Person();
        Class c = p.getClass(); // c 对象中就包含了 p 对象所属类的全部信息
    }
}
```

注：

1. 对于每个类而言，JRE 都为其保留了一个不变的 Class 类型的对象；

2. 一个 Class 对象包含了特定某个类的有关信息；

3. Class 对象只能由系统建立；
4. 一个类在 JVM 中只会有一个 Class 实例；
5. 一个 Class 对象对应的是一个加载到 JVM 中的一个 .class 文件；
6. 每个类的实例都会记得自己是由哪个 Class 实例生成的；
7. 通过 Class 可以完整地得到一个类中的完整结构。



### 3.1. Class类的常用方法



| 方法名                                              | 作用                                       |
| --------------------------------------------------- | ------------------------------------------ |
| static Class forName(String name)                   | 根据类的全名（包名+类名）获取 Class 类对象 |
| Object newInstance()                                | 创建目标类的对象                           |
| String getName()                                    | 获取类全名（包+类名）                      |
| Class getSuperclass()                               | 获取父类的 Class 对象                      |
| Class[] getInterface()                              | 获取所有实现的接口 Class 对象              |
| ClassLoader getClassLoader()                        | 获取类的加载器                             |
| Constructor[] getConstructors()                     | 获取所有的 public 构造器                   |
| Consturctor[] getDeclaredConstructors()             | 获取由程序员声明的构造器                   |
| Field[] getDeclaredFields()                         | 获取所有的属性                             |
| Method getMethod(String name, Class ... paramTypes) | 获取相应的方法                             |



### 3.2. 实例化 Class 类对象的方法

1. 前提：已知具体的类，通过类的 `class` 属性获取，该方法最安全可靠，程序性能最高。

    ```java
    Class c = String.class;
    ```

2. 前提：已知某个类的实例，调用该实例的 getClass() 方法。

    ```java
    Class c = "123xyz".getClass();
    ```

3. （常用）前提：已知一个类的全名（包+类名），且该类在类路径下，可通过 Class 类的静态方法 forName() 获取，可能抛出 ClassNotFoundException。

    ```java
    Class c = Class.forName("java.lang.String");
    ```

4. 其他方式（不做要求）

    ```java
    ClassLoader cl = this.getClass().getClassLoader();
    Class c = cl.loadClass("类的全类名");
    ```

## 4. 通过反射调用类的完整结构

通过反射调用类的完整结构。

如 Field, Method, Constructor, Superclass, Interface, Annoation。

详细见代码与输出：

代码

```java
// 父类
package Test;

public class Person {
    public String name;
    public int age;
}

```

```java
// 接口 Move
package Test;

public interface Move {
    void moveType();
}
```

```java
// 接口 Study
package Test;

public interface Study {
    void studyInfo();
}

```

```java
// Student 类
package Test;

public class Student extends Person implements  Move, Study {

    // public 无参构造器
    public Student() {};
    // public 有参构造器
    public Student(String school) {
        this.school = school;
    }
    // private 有参构造器
    private Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String school;
    private int JJlength = 12;

    void showInfo() {
        System.out.println("school is: " + this.school);
    }

    public void setSchool(String school) {
        this.school = school;
    }
    private void setAge(int age) {
        this.age = age;
    }

    @Override
    public void moveType() {
        System.out.println("Car");
    }
    @Override
    public void studyInfo() {
        System.out.println("English");
    }
}

```

```JAVA
// Test 代码
package Test;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class Test {
    public static void main(String[] args) {
        try {
            Class c = Class.forName("Test.Student");

            Class superC = c.getSuperclass();   // 获取父类名
            Class[] interfacesC = c.getInterfaces();    // 获取接口类名
            Constructor[] ct1 = c.getConstructors();    // 获取 public 构造器
            Constructor[] ct2 = c.getDeclaredConstructors();    // 获取程序员声明的构造器（全部构造器）
            Method[] mt1 = c.getMethods();    // 获取 public 方法
            Method[] mt2 = c.getDeclaredMethods();    // 获取程序员声明的方法（不包括父类继承下来的）
            Field[] f1 = c.getFields();    // 获取 public 属性
            Field[] f2 = c.getDeclaredFields();    // 获取程序员声明的方法（不包括父类继承下来的）
            Package p = c.getPackage();    // 获取类所属的包

            // 打印父类名
            System.out.println(superC);

            System.out.println();

            // 打印接口类名
            for(Class cc: interfacesC)
                System.out.println(cc);

            System.out.println();

            // 打印 public 构造器
            for(Constructor cc: ct1)
                System.out.println(cc);

            System.out.println();

            // 打印全部构造器，以 修饰符 + 名字+ 参数类型 方式
            // 修饰符类型 {0: default, 1: public, 2: private, 4: protected} 等等
            for(Constructor cc: ct2) {
                System.out.print(cc.getModifiers() + " " + cc.getName() + " ");
                Class[] tp = cc.getParameterTypes();
                for(Class cc2: tp) {
                    System.out.print("argument: " + cc2 + " ");
                }
                System.out.println();
            }

            System.out.println();

            // 通过 Class 构造新对象，getConstructor 会根据其参数选择对应的构造器返回
            Constructor ctn1 = c.getConstructor(String.class);
            Object obj1 = ctn1.newInstance("the fifth middle school");
            Student stu1 = (Student)obj1;
            System.out.println(stu1.school);
            System.out.println();

            // 用过反射机制强制调用私有构造方法
            Constructor ctn2 = c.getDeclaredConstructor(String.class, int.class);
            ctn2.setAccessible(true);    // 设置为 true 为解除封装
            Object obj2 = ctn2.newInstance("gukaifeng", 22);
            Student stu2 = (Student)obj2;
            System.out.println(stu2.name + " " + stu2.age);
            System.out.println();

            // 打印 public 方法
            for(Method m: mt1)
                System.out.println(m);
            System.out.println();

            // 打印声明的方法
            // 换一种方式打印，以 修饰符 + 返回类型 + 名字 方式
            // 获取方法参数类型方法与获取构造器的一样，这里没写
            for(Method m: mt2)
                System.out.println(m.getModifiers() + " " + m.getReturnType() + " " + m.getName());
            System.out.println();

            // 打印所有属性
            for(Field f: f1)
                System.out.println(f);
            System.out.println();

            // 打印声明的属性
            // 换个方式，以 修饰符 + 类型 + 名字 打印
            for(Field f: f2)
                System.out.println(f.getModifiers() + " " + f.getType() + " " + f.getName());
            System.out.println();

            // 打印类所属的包
            System.out.println(p);
            System.out.println();

            // 方法的调用类似构造器的调用
            // getMethod() 和 getDeclaredMethod() 返回类型为 Method
            // 调用函数为 m.invoke(obj, args) 第一个参数是对象，第二个参数是方法的参数列表
            // 这里使用上面构造好的 stu1 对象
            Method m1 = c.getMethod("setSchool", String.class);    // 得到铭文 setSchool，参数列表为 [String] 的方法
            m1.invoke(stu1, "Hebei Normal University"); // 如果方法有返回值，要有变量接收
            System.out.println(stu1.school);

            Method m2 = c.getDeclaredMethod("setAge", int.class);
            m2.setAccessible(true); // 解除封装
            m2.invoke(obj1, 23); // 也可以通过父类调用，这里的 obj1 其实就是 student 类
            System.out.println(stu1.age);
            System.out.println();

            // 调用方法
            Field f = c.getField("school");
            Field f3 = c.getField("age");
            String s = (String)f.get(stu1);
            Integer a1 = (Integer)f3.get(stu1);      // 调用
            System.out.println(a1);
            f3.set(stu1, 22);   // 修改值
            Integer b2 = (Integer)f3.get(stu1);
            System.out.println(b2);
            System.out.println();

            // 访问 private 属性也要解除权限
            Field f4 = c.getDeclaredField("JJlength");
            f4.setAccessible(true);
            Integer a2 = (Integer)f4.get(stu1);
            System.out.println(a2);




        } catch(Exception e) {
            e.printStackTrace();
        }


    }
}

```

输出

```shell
class Test.Person

interface Test.Move
interface Test.Study

public Test.Student(java.lang.String)
public Test.Student()

2 Test.Student argument: class java.lang.String argument: int 
1 Test.Student argument: class java.lang.String 
1 Test.Student 

the fifth middle school

gukaifeng 22

public void Test.Student.setSchool(java.lang.String)
public void Test.Student.moveType()
public void Test.Student.studyInfo()
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public java.lang.String java.lang.Object.toString()
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()

1 void setSchool
2 void setAge
0 void showInfo
1 void moveType
1 void studyInfo

public java.lang.String Test.Student.school
public java.lang.String Test.Person.name
public int Test.Person.age

1 class java.lang.String school
2 int JJlength

package Test

Hebei Normal University
23

23
22

12
```

1. 