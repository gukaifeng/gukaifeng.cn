
枚举类和普通类的区别：

* 使用 enum 定义的枚举类默认继承了 java.lang.Enum 类
* 枚举类的构造器只能使用 private 访问控制符
* 枚举类的所有实例必须在枚举类中显式列出（用 ',' 分隔，';' 结尾 ），系统会自动为列出的实例添加 public static final 修饰；
* 所有枚举类都提供一个 values 方法，该方法可以很方便地遍历所有的枚举值；
* 枚举类中的每个枚举，都是单例模式的。

JDK 1.5 中可以在 switch 表达式中使用枚举类的对象作为表达式，case 子句可以直接使用枚举值的名字，无需添加枚举类作为限定。

若枚举类只有一个成员，则可以作为一种单子模式的实现方式。

<!--more-->

```java
enum Season {
    SPRING("春天", "春暖花开"),
    SUMMER("夏天", "炎炎夏日"),
    AUTUMN("秋天", "秋高气爽"),
    WINTER("冬天", "寒风凛冽");
  
    private final String name;
    private final String desc;
  
    private Season(String name, String desc) {
        this.name = name;
        this.desc = desc;
    }
  
    public void showInfo() {
        System.out.println(this.name + ": " + this.decs);
    }
}
```

```java
public class untitled {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
        sprint.showInfo();
    }
}

// 输出
// 春天: 春暖花开
```

枚举类也可以实现接口，若需要每个枚举值实现接口的方法不同，可以让每个枚举值分别来实现该方法。