### 1.Enum

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

推荐使用enum创建单例对象，线程安全的同时也不会像双重检查锁、内部类那样复杂，同时也不会被反序列化、反射等方式破坏单例规则。

### 2.String

- 声明为final，不可被继承
- 实现了Serializable接口，支持序列化
- 实现Comparable接口，可比较大小
- 内部定义final char[] value用于存储字符串，不可改变指向，也没有方法修改内部数据，此为“不可变性”，体现在
  1. 当对字符串重新赋值时，需要重新改变指向内存区域（方法区的字符串常量池），不能使用原有区域进行改写
  2. 当对字符串执行拼接操作时，也需要重新指定内存区域。
  3. 调用replace方法替换个别字符串时，也需要重新指定内存区域。
- 区别于new一个对象，直接显式赋值，字符串声明在字符串常量池中。
- 字符串常量池不会存储相同字符串。

![image-20210315094448164](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210315094448164.png)

**new构造器：**

```java
String s = "java";//字符串常量池
s = new String("java");//堆空间地址值,同时也会在stringtable中创建一个该字符串
```

![image-20210315094812330](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210315094812330.png)

![image-20210315094938222](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210315094938222.png)

**字符串+操作：**

- 二者都是显式字面量相加，因此编译期间替换，替换为字符串常量池
- 如果其中之一是变量形式出现，则通过new stringbuilder进行append最终进行tostring，tostring类似new string，但不完全一样，区别在于会不会在stringtable中也创建一个相同字符串。

![image-20210315095521252](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210315095521252.png)

**Stringbuffer线程安全**

接下来两个内部是char value[]修饰，但没有final修饰，可变。

初始化时，如果传入的字符串一开始为""，则内部value数组长度为16，之后append字符串先判断长度是否够。长度超出时，则新创建一个数组，并复制过去。

stringbuffer任何关于修改字符串的方法都是在原有value基础上进行修改，时空效率上比String较高。

方法链设计，如调用append方法后返回this对象。

**Stringbuilder非安全-效率高**

效率对比：StringBuilder>StringBuffer>String

### 3.泛型

集合容器类在设计阶段没有指明将来所要存储的对象具体是什么类型的，在JDK5.0之前就只能设计为Object，在5.0之后改用泛型，当指明泛型类型时，对该类、接口的具体实现就能够明确要求属性、方法返回类型。其实这是一个编译期行为，旨在检查编码时所填入的类型是否满足泛型标准，在运行期间仍然看作Object，因此可通过反射破坏泛型约束。

**注意点：**

1. 尽管编译时ArrayList\<Integer> 和ArrayList\<String>是两个不同类型，但其实内部仍然看作Object，因此还是创建了同一个类的两个不同对象，JVM中只加载了这一个类。
2. 泛型内部仍看作Object，但不完全等价于Object。
3. 静态方法、属性无法使用泛型，因为泛型类是在创建实例时确定的。
4. 子类可继承泛型父类，但遵循“富二代”原子，即必须指明或保留父类的泛型，同时还可以增加自己的泛型。
5. 泛型方法只与调用方法时传入的参数或接受类型有关，和泛型类无关，静态方法可以使用泛型方法。

**泛型在多态性的体现**

```java
List<Object> list1 = new ArrayList<>();
List<String> list2 = new ArrayList<>();
list1 = list2;//×，由于list2本来规定只能存放String类型变量，但强制赋值给list1，导致变量类型范围被扩大，就失去泛型约束的作用了

List<String> list1 = new ArrayList<>();
ArrayList<String> list2 = new ArrayList<>();
list1 = list2;//√，这种写法可以，因为List是ArrayList的父类，肯定包含子类所有的属性和方法。
```

**通配符"?"**

```java
ArrayList<Integer> list1 = new ArrayList<>();
ArrayList<String> list2 = new ArrayList<>();
ArrayList<?> list3 = null;
//虽然list1和list2是不同泛型，但公共父类是泛型为?的list3
//对于list3来说，只能读取(且转换为Object)，但无法添加，因为无法规定添加什么类型
list3 = list1;
list3 = list2;
list2.add("hello");
list2.add("world");
Object o = list3.get(0);
```

**通配符"?"的extends和super**

```java
List<Person> list1 = new ArrayList<>();
List<Student> list2 = new ArrayList<>();
List<? extends Person> list3 = null;//<=Person
List<? super Student> list4 = null;//>=Student
list3 = list1;
list3 = list2;
Person person = list3.get(0);
list4 = list1;
list4 = list2;
Object object = list4.get(0);
list4.add(new Student());
```

### 4.IO流

**流的分类：**

- 处理单位不同：字节流、字符流
- 流向不同：输入流、输出流
- 处理方式不同：节点流、处理流

**IO流体系：**

![image-20210317095751852](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210317095751852.png)

| 抽象基类     | 节点流                                     | 缓冲流                      | 转换流            | 数据流          | 对象流             |
| ------------ | ------------------------------------------ | --------------------------- | ----------------- | --------------- | ------------------ |
| InputStream  | FileInputStream(read(byte[]))              | BufferInputStream           | InputStreamReader | DataInputStream | ObjectInputStream  |
| OutputStream | FileOutputStream(write(byte[],offset,len)) | BufferOutputStream          | OuputStreamWriter | DataOuputStream | ObjectOutputStream |
| Reader       | FileReader(read(char[]))                   | BufferReader(readline())    |                   |                 |                    |
| Writer       | FileWriter(write(char[],offset,len))       | BufferWriter(write(String)) |                   |                 |                    |

**FileReader：**

```java
File file = new File("hello.txt");
Reader reader = new FileReader(file);
int r;
char buf[]=new char[10];
String s=null;
try {
    while (((r = reader.read(buf)) != -1)) {
        s=new String(buf,0,r);
        System.out.print(s);
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        if (reader != null)
            reader.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

注意IO流中的try-catch-finally处理，防止资源未被关闭。（JVM垃圾回收可以针对没有引用的对象自动的进行回收，但像IO、Socket和数据库连接这类对象无法自动进行回收，一定注意手动close）

**FileWriter：**

1. 如果文件不存在，则自动创建
2. 如果文件已经存在：
   - new FileWirter(file, append=true)追加
   - new FileWirter(file, append=false)覆盖

**FileInputStream和FileOutputStream实现文件复制**

```java
File srcFile = new File("src.jpg");
File destFile = new File("src2.jpg");
FileInputStream fileInputStream = null;
FileOutputStream fileOutputStream = null;
try {
    fileInputStream = new FileInputStream(srcFile);
    fileOutputStream = new FileOutputStream(destFile);
    byte buf[] = new byte[1024];
    int r;
    while ((r = fileInputStream.read(buf)) != -1) {//批量读取
        fileOutputStream.write(buf, 0, r);//批量写入
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {//关闭资源
    try {
        if (fileInputStream != null)
            fileInputStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
    try {
        if (fileOutputStream != null)
            fileOutputStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

**BufferedInputStream和BufferedOutputStream复制速度提升**

此为处理流，相较节点流，又在外包了一层，原来的FileInputStream和FileOutputStream中间的byte数组一旦填满，就立即传输，而BufferStream在中间额外创建了一个内存buffer，只有当该空间被填满以后，才会进行传输，相当于减少了IO次数。

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/20180930222711588)

注意，在资源进行关闭后，只需关闭外层流即可，内层流也会被关闭，使用了装饰者模式。

**转换流**

- InputStreamReader：将磁盘中的字节流转换为内存中的字符流（解码）
- OutputStreamWriter：将内存中的字符流再次转换为磁盘中的字节流（编码）

![image-20210317120303811](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210317120303811.png)

```java
InputStreamReader inputStreamReader=new InputStreamReader(new FileInputStream(new File("")));
```

**System.in和System.out**

System.in本质也是InputStream，其默认输入位置是键盘，out默认是控制台。

```java
//从标准输入读取字符串，转换成大写输出，如果是e或者exit则退出
InputStreamReader inputStreamReader = new InputStreamReader(System.in);
BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
try {
    while (true) {
        String data = bufferedReader.readLine();
        if ("e".equalsIgnoreCase(data) || "exit".equalsIgnoreCase(data)) {
            break;
        }
        System.out.println(data.toUpperCase());
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        if (bufferedReader != null)
            bufferedReader.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
    try {
        if (inputStreamReader != null)
            inputStreamReader.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

**数据流**

将java内存中的基本数据类型转换为磁盘中的二进制数据。

```java
public static void main(String[] args) throws IOException {
    DataOutputStream dataOutputStream = new DataOutputStream(new FileOutputStream(new File("")));
    dataOutputStream.writeDouble(12.3);
    dataOutputStream.writeBoolean(true);
    dataOutputStream.flush();
    dataOutputStream.close();
}
```

**对象流**

将内存中的对象以二进制流的方式写入到磁盘文件中。

```java
public static void main(String[] args) throws IOException {
    ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(new File("")));
    objectOutputStream.writeObject(new Person());
}
static class Person implements Serializable {
    private static final long serialVersionUID = 3213123213L;
    String name;
    int age;
}
```

serialVersionUID的作用是：如果没有显式指明该值，如果当类被修改时，该值会自动更改，此时如果还原原来就已经写入过的对象，则会报错，提示无法反序列化。因此它的作用时，确保当类结构发生变化时，反序列化仍可正常进行。

总结，要想序列化成功，需满足：

- 对象实现serializable接口，属性递归实现serializable接口
- 对象提供一个serialVersionUID类静态常量
- 如果不想某些属性被序列化，除了static修饰的静态变量以外，用transient修饰的变量也不会被序列化。

现在的应用传输对象数据一般都使用json。

### 5.注解

JDK1.5增加的特殊标记，在类被加载、运行时读取，执行相应的处理逻辑。可以说框架=注解+反射+设计模式

使用场景：

- 文档注解，如@author，@version，@see
- 编译时检查，如@Override，@Deprecated
- 功能补充，如@WebServlet，@Controller，@Transaction，@Test

**自定义注解**

1. 注解声明为@interface

2. 内部定义成员，通常使用value表示

3. 可以指定默认值，default

4. 如果没有成员，则只起到标识作用，否则使用注解时需要显式指明成员值，自定义注解需要使用注解处理流程（使用反射）才有意义


```java
public @interface Myannotation {
    String value() default "test";
}

@Myannotation(value = "test_good")
class test {
}
```

**元注解**

注解的注解，对定义的注解进行补充说明。

- Retention（生命周期注解）：指定所修饰的Annotation的生命周期，SOURCE\CLASS，以及RUNTIME，只有声明为RUNTIME的注解才能通过反射调用。
- Target（修饰对象）：TYPE、FILED、CONSTRUCT...
- Documented（javadoc保留）：声明该元注解时，javadoc后会显示该注解
- Inherited（继承性）：声明该元注解的注解会自动继承给子类

注解的变量值通过反射获取，并形成一套逻辑。

### 6.反射

反射是动态语言的关键，该机制允许在程序运行期间借助于ReflectionAPI获取类的任何信息，包括属性、方法、构造器和注解等，通过运行时类加载以及反射的机制，我们可以在方法区中产生任意class类的类型。

用途：

- 判断任意对象所属的类
- 构造任意类的对象
- 获取任意类所具有的成员变量和方法
- 获取类的泛型信息
- 调用任意一个对象的成员变量和方法
- 处理注解
- 生成代理对象，类的动态性（动态加载、动态生成）

获取类的类型四种方式：

```java
public class Person {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<Person> personClass = Person.class;

        Person person = new Person();
        personClass = (Class<Person>) person.getClass();

        personClass = (Class<Person>) Class.forName("Person");//主动加载

        ClassLoader classLoader = Person.class.getClassLoader();
        personClass = (Class<Person>) classLoader.loadClass("Person");//被动加载
    }
}
```

通过反射创建运行时对象

```java
Class<Person> personClass = Person.class;
Person person = personClass.newInstance();
```

本质上仍调用的是空参构造器，一般不会去获取指定的构造器方法（getconstructor）去创建一个对象。

注意，通过setAccessible可以强制获取到私有属性、方法，并且关闭jvm对其访问权限的检查，以提高效率。

```java
Class<test> cl = (Class<test>) Class.forName("test");
Method method = cl.getMethods()[1];
method.setAccessible(true);
```

**获取类内部结构**

```java
public static void main(String[] args) throws ClassNotFoundException {
    Class<test> cl = (Class<test>) Class.forName("test");
    Field[] fields = cl.getFields();
    for (Field field : fields) {
        System.out.println(field);
    }//公有属性

    fields = cl.getDeclaredFields();
    for (Field field : fields) {
        System.out.println(field);
    }//全部属性

    Method[] methods = cl.getMethods();
    for (Method method : methods) {
        System.out.println(method);
    }//子类及其父类public方法

    methods = cl.getDeclaredMethods();
    for (Method method : methods) {
        System.out.println(method);
    }//本类全部方法
}
```

**反射处理注解**

```java
public class Test {
    public static void main(String[] args) throws NoSuchFieldException {
        Class<Person> personClass = Person.class;
        clAnnotation annotation = personClass.getAnnotation(clAnnotation.class);
        System.out.println(annotation.value());

        Field age = personClass.getDeclaredField("age");
        filedAnnotation annotation1 = age.getAnnotation(filedAnnotation.class);
        System.out.println(annotation1.name());
        System.out.println(annotation1.type());
        System.out.println(annotation1.length());
    }
}

@clAnnotation("clAnnotation_test")
class Person {
    @filedAnnotation(name = "age", type = "int", length = 3)
    int age;
    @filedAnnotation(name = "name", type = "varchar", length = 20)
    String name;

    public Person(int age, String name) {
        this.age = age;
        this.name = name;
    }
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface clAnnotation {
    String value();
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface filedAnnotation {
    String name();

    String type();

    int length();
}
```

### 7.代理

**静态代理**

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
        this.cloth = cloth;
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

**动态代理**

没有显式写明代理类对象，而是通过Proxy.newProxyInstance()方法返回一个被代理对象，该对象实现同名接口，用于之后调用代理类对象的方法。

```java
public class reflect {
    public static void main(String[] args) {
        Clothfactory nike = new Nike();
        Clothfactory instance = (Clothfactory) getInstance(nike);
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
/*
通用方法1
produce nike-cloth
通用方法2*/
```

扩充的实现逻辑就在handler对象的invoke方法中，每次调用接口中的方法时，都会执行该invoke方法，不同方法可通过判断method名称进行逻辑扩充。动态代理的动态性体现在没有显示定义代理类，可通过传入不同的Handler类进行不同方面的增强。

### 8.JDK8新特性

**Lambda表达式与方法引用**

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

**函数接口**

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
        //同为类::实例方法的还有String::toUpperCase
    }
}
```

**注意：**类::实例方法相当于将第一个参数作为对象，当成该类的实例对象，进而能够执行该方法。构造器引用、数组引用也是可以看作类::静态方法的补充，`类名::new`，在创建对象/数组时可以考虑该种写法。

**Stream**

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
    return list;
}
```

**map算子：**

```java
List<String> strings = Arrays.asList("aa", "bb", "cc");
Stream<String> stream = strings.stream();
stream.map(String::toUpperCase).forEach(System.out::println);//类::实例方法
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

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4);
Stream<Integer> stream = list.stream();
List<Integer> collectlist = stream.collect(Collectors.toList());
collectlist.forEach(System.out::println);//此时的forEach是collections的外部迭代
```

**Optional**

为了在程序中避免出现空指针异常而创建的一种“容器”。常用方法：

```java
Person person = new Person(12);
person = null;
Optional<Person> optionalPerson = Optional.ofNullable(person);
Person one = optionalPerson.orElse(new Person(15));
System.out.println(one.age);//15
//Option.of创建一个非空容器
//Optional.ofNullable创建可以为空的容器
//Optional.ofElse如果容器内部为空，则返回指定对象
```