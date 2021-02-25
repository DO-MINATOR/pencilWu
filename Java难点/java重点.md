### 枚举类Enum

当定义有限个变量，且变量之间有一定次序关系时，使用枚举类创建final对象较为合适。

JDK1.5之前，使用自定义类来实现枚举类，如下所示：

```java
class Season {
    private final String name;
    private final int id;

    private Season(String name, int id) {
        this.name = name;
        this.id = id;
    }

    public static final Season SPRING = new Season("Spring", 1);
    public static final Season SUMMER = new Season("Summer", 2);
    public static final Season AUTUMN = new Season("Autumn", 3);
    public static final Season WINNTER = new Season("Winter", 4);

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Season{" +
                "name='" + name + '\'' +
                ", id=" + id +
                '}';
    }
}
```

通过将属性、构造器私有化，通过类静态变量初始化的方式创建实例（类似单例对象创建），进而实现枚举对象的创建，其余访问方法和普通对象相同。

JDK1.5之后可通过Enum关键字进行枚举对象的创建，如下：

```java
enum Season {
    SPRING("Spring", 1),
    SUMMER("Summer", 2),
    AUTUMN("Autumn", 3),
    WINNTER("Winter", 4);

    private final String name;
    private final int id;

    private Season(String name, int id) {
        this.name = name;
        this.id = id;
    }

    public String getName() {
        return name;
    }
}
```

修改方式是将`public static final Season`关键字删除，`new Season`语句删除，当Season enum加载时，四个对象在类初始化阶段创建，属于饿汉式。

另外，推荐使用enum创建单例对象，线程安全的同时也不会像双重检查锁、内部类那样复杂，同时也不会被反序列化、反射等方式破坏单例规则。

### 注解Annotation

JDK1.5增加的特殊标记，在类被加载、运行时读取，执行相应的处理逻辑。可以说框架=注解+反射+设计模式

使用场景：

- 文档注解，如@author，@version，@see
- 编译时检查，如@Override，@Deprecated
- 功能补充，如@WebServlet，@Controller，@Transaction，@Test

**自定义注解**

1. 注解声明为@interface

2. 内部定义成员，通常使用value表示

3. 可以指定默认值，default

4. 如果没有成员，则只起到标识作用，否则使用注解时需要显式指明成员值

   自定义注解需要使用注解处理流程（使用反射）才有意义

```java
public @interface Myannotation {
    String value() default "test";
}

@Myannotation("test_good")
class test {
}
```

**元注解**

注解的注解，对定义的注解进行补充说明。

- Retention（生命周期注解）：指定所修饰的Annotation的生命周期，SOURCE\CLASS，以及RUNTIME，只有声明为RUNTIME的注解才能通过反射调用。
- Target（修饰对象）：TYPE、FILED、CONSTRUCT...
- Documented（javadoc保留）：声明该注解时，javadoc后会显示该注解
- Inherited（继承性）：声明该注解的注解会自动继承给子类

