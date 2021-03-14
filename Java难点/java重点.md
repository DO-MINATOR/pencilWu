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

### String类的理解451



### 泛型565



### IO流584



### 注解与反射【狂神】

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

### 动态代理

#### 静态代理

接口规范了代理类和被代理类的调用方法名，被代理类实现具体细节，代理类通过将被代理类赋值自身属性，并扩充实现方法，此后通过代理类调用方法，以实现静态代理。实现如下：

```java
public class reflect {
    public static void main(String[] args) {
        Clothfactory nike = new Nike();
        nike.produce();
        System.out.println("************************");
        Clothfactory proxy = new Proxy(nike);
        proxy.produce();

    }
}

interface Clothfactory {
    void produce();
}

class Nike implements Clothfactory {

    @Override
    public void produce() {
        System.out.println("produce nike-cloth");
    }
}

class Proxy implements Clothfactory {
    Clothfactory cloth;

    public Proxy(Clothfactory cloth) {
        this.cloth = cloth;java
    }

    @Override
    public void produce() {
        System.out.println("before");
        cloth.produce();
        System.out.println("after");
    }
}
```

静态代理的实现特点是，编译期间就要确定所有代理类和被代理类，无法再在运行期间动态生成指定代理类。

#### 动态代理

没有显式写明代理类对象，而是通过Proxy.newProxyInstance()方法返回一个被代理对象，该对象实现同名接口，用于之后调用代理类对象的方法。

```java
public class reflect {
    public static void main(String[] args) {
        Clothfactory nike = new Nike();
        Clothfactory instance = (Clothfactory) getInstance(nike);
        instance.produce();
        Clothfactory lining = new Lining();
        instance = (Clothfactory) getInstance(lining);
        instance.produce();
    }

    public static Object getInstance(Object obj) {
        Handler handler = new Handler();
        handler.bind(obj);
        Object o = Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), handler);
        return o;
    }
}

class Handler implements InvocationHandler {
    Object obj;

    void bind(Object obj) {
        this.obj = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("通用方法1");
        Object returnvalue = method.invoke(obj, args);
        System.out.println("通用方法2");
        return returnvalue;
    }
}

interface Clothfactory {
    void produce();
}

class Nike implements Clothfactory {
    @Override
    public void produce() {
        System.out.println("produce nike-cloth");
    }
}

class Lining implements Clothfactory {
    @Override
    public void produce() {
        System.out.println("produce lining-cloth");
    }
}
/*
通用方法1
produce nike-cloth
通用方法2
通用方法1
produce lining-cloth
通用方法2*/
```

扩充的实现逻辑就在handler对象的invoke方法中，由于通用方法固定，因此动态性就体现在传入的object。

无论是静态代理还是动态代理，被代理对象和代理对象都实现同样的接口，最终通过代理类调用原来的同名方法。

### JDK8新特性

#### Lambda表达式与方法引用

当接口只有一个方法需要实现时，可以采用lambda表达式或方法引用。由于接口只有一个方法，相当于被实现的方法是固定的，因此lambda表达式左边的参数列表固定（方法的形参列表），`->`右侧为执行体。lambda其本质就是接口的一个实例化对象，只不过是匿名的。

```java
public static void main(String[] args) {
    Comparator<Integer> comparator = new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return Integer.compare(o1, o2);
        }
    };
    System.out.println(comparator.compare(1, 2));
    //lambda
    Comparator<Integer> comparator1 = (o1, o2) -> Integer.compare(o1, o2);
    System.out.println(comparator1.compare(3, 2));
    //方法引用
    Comparator<Integer> comparator2 = Integer::compare;
    System.out.println(comparator2.compare(3, 4));
}
```

#### 函数接口

注解@FunctionnalInterface用于校验接口是否满足lambda函数式接口，只有一个抽象方法时，校验通过。

| 函数接口      | 参数类型 | 返回类型 | 用途              |
| ------------- | -------- | -------- | ----------------- |
| Consumer\<T>  | T        | void     | accept()接受参数  |
| Supplier \<T> | null     | T        | get()返回结果     |
| Function<T,R> | T        | R        | R apply(T t)      |
| Predicate\<T> | T        | boolean  | boolean test(T t) |

```java
public static ArrayList<String> filter(ArrayList<String> list, Predicate<String> predicate) {
    ArrayList<String> list2 = new ArrayList<>();
    for (String s : list) {
        if (predicate.test(s)) {
            list2.add(s);
        }
    }
    return list2;
}
ArrayList<String> list = new ArrayList<>();
list.add("1k");list.add("3k");list.add("4k");list.add("1w");

ArrayList<String> list2 = filter(list, (String s) -> s.contains("k"););
```

只有当前已经存在一个方法，且它的参数列表、返回类型与接口方法完全一致时，则lambda表达式可以替换为对该方法的引用，本质上还是接口的实例化对象，只不过内部进行了二次调用。

方法引用一共有三种模式：

```java
public class lambda {
    String hello() {
        return "hello world";
    }

    public static void main(String[] args) {
        lambda lambda = new lambda();
        Supplier<String> supplier = lambda::hello;//对象::实例方法
        System.out.println(supplier.get());

        Comparator<Integer> com1 = Integer::compare;//类::静态方法
        System.out.println(com1.compare(1, 2));//-1

        Comparator<Integer> com2 = Integer::compareTo;//类::实例方法
        System.out.println(com2.compare(1, 2));//-1
    }
}
```

**注意：**类::实例方法相当于将第一个参数作为对象，当成该类的实例对象，进而能够执行该方法。构造器引用、数组引用也是可以看作类::静态方法的补充，`类名::new`，在创建对象/数组时可以考虑该种写法。

#### Stream

对集合进行操作，执行查找、过滤、映射数据等操作，相较于在数据库中执行计算的SQL，它是在java层面中进行数据计算。Collection面向数据，Stream面向计算。

- 不会存储数据
- 不改变原有对象，返回结果为新的stream
- 操作是延迟执行的，等到需要调用终止操作时才会执行任务

stream的使用以函数式接口居多，通过lambda表达式和方法引用实现。

**筛选和切片算子：**

```java
public static void main(Strijavang[] args) {
    List<Person> list = getlist();
    Stream<Person> stream = list.stream();
    stream.filter(person -> person.age > 22).forEach(System.out::println);//过滤
    stream = list.stream();
    stream.limit(2).forEach(System.out::println);//截断流
    stream = list.stream();
    stream.skip(3);//跳过
    stream = list.stream();
    stream.distinct();//依据hashcode()、equals()自动去重。
}
static List<Person> getlist() {
    List<Person> list = new ArrayList<>();
    list.add(new Person("wsp", 22));
    list.add(new Person("linda", 21));
    list.add(new Person("jack", 23));
    list.add(new Person("dsc", 25));
    return list;java
}
```

**map算子：**

```java
List<String> strings = Arrays.asList("aa", "bb", "cc");
Stream<String> stream = strings.stream();
stream.map(String::toUpperCase).forEach(System.out::println);
```

**flatmap算子：**

```java
public static void main(String[] args) {
    List<String> strings = Arrays.asList("aa", "bb", "cc");
    Stream<String> stream = strings.stream();
    Stream<Stream<Character>> streamStream = stream.map(lambda::string2stream);
    streamStream.forEach(s -> {
        s.forEach(System.out::println);
    });//map
    
    stream = strings.stream();//reopen
    Stream<Character> characterStream = stream.flatMap(lambda::string2stream);//flatmap
    characterStream.forEach(System.out::println);
}
static Stream<Character> string2stream(String s) {
    List<Character> list = new ArrayList<>();
    for (Character c : s.toCharArray()) {
        list.add(c);
    }
    return list.stream();
}
```

map操作可以将stream中的元素又转转为stream，形成stream<stream\<T>>嵌套，而flatmap算子在转换过程中直接进行扁平化处理，防止嵌套。

**sort算子：**

进行排序用

**终止操作**

1、匹配与查找

| 方法                   | 返回值   | 描述                                         |
| ---------------------- | -------- | -------------------------------------------- |
| allMatch(predicate p)  | boolean  | 检查是否匹配所有元素                         |
| anyMatch(predicate p)  | boolean  | 检查是否有一个元素匹配                       |
| noneMatch(predicate p) | boolean  | 检查是否都不匹配                             |
| findFirst()            | Optional | 返回stream中第一个元素                       |
| findAny()              | Optional | 返回stream中任意一个元素                     |
| count()                | int      | 返回stream总元素个数                         |
| max(Comparator c)      | Optional | 返回流中最大元素                             |
| min(Comparator c)      | Optional | 返回流中最小元素                             |
| forEach(Consumer c)    | void     | 内部迭代。相较于Collection的iterator外部迭代 |

2、规约

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4);
Stream<Integer> stream = list.stream();
Integer reduce = stream.reduce(10, Integer::sum);//reduce第二参数为一个接口，待实现方法是T applay(T,R)，因此直接用方法引用
System.out.println(reduce);//20
```

3、收集(转成collection集合)

```
List<Integer> list = Arrays.asList(1, 2, 3, 4);
Stream<Integer> stream = list.stream();
List<Integer> collectlist = stream.collect(Collectors.toList());
collectlist.forEach(System.out::println);//此时的forEach是collections的外部迭代
```

#### Optional

为了在程序中避免出现空指针异常而创建的一种“容器”。常用方法：

```java
Person person = new lambda().new Person(12);
person = null;
Optional<Person> optionalPerson = Optional.ofNullable(person);
Person one = optionalPerson.orElse(new lambda().new Person(15));
System.out.println(one.age);//15
//Option.of创建一个非空容器
//Optional.ofNullable创建可以为空的容器
//Optional.ofElse如果容器内部为空，则返回指定对象
```