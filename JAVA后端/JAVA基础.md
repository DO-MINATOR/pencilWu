1、java是介于解释型和编译型的语言，先统一编译为字节码文件，然后不同平台的jvm虚拟机负责加载字节码并执行。java分三大版本。Java SE：Standard Edition、Java EE：Enterprise Edition、Java ME：Micro Edition

![image-20200530093837521](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200530093837521.png)

通俗来说，普通编程用se，企业版ee只是在se的基础上多了大量api和库，以方便开发web应用、数据库、消息服务等，他俩使用的JVM虚拟机完全一致。而ME是针对嵌入式设备的“瘦身版”，虚拟机也有所不同。

2、首先要学习Java SE，掌握Java语言本身、Java核心开发技术以及Java标准库的使用；

如果继续学习Java EE，那么Spring框架、数据库开发、分布式架构就是需要学习的；

如果要学习大数据开发，那么Hadoop、Spark、Flink这些大数据平台就是需要学习的，他们都基于Java或Scala开发；

如果想要学习移动开发，那么就深入Android平台，掌握Android App开发。

3、JRE是java运行环境，能够加载字节码并运行，而JDK除了包含JRE，还包含一系列编译器、调试器等，负责将java源码编译成字节码。

4、java基本程序结构，每一个java源文件包含一个public class单元，含有一个public static void修饰的main函数，参数为string的数组

```java
public class Hello{
    public static void main(String[] args) {
        System.out.println("hello world");
    }
} 
```

5、变量分为基本变量和引用变量，整型，浮点型，布尔型，字符型；常量使用final初始化，无法更改。

6、自增减操作   <<、>>二进制移位运算 &、|、~、^位运算

强制类型转换使用（short）i，类型会自动进行提升。

7、浮点数无法精确存储，1 - 9.0 / 10和0.1直接比较不相等，应使用math.abs判断之差是否小于一个阈值。

8、布尔关系运算包含&&、||、！，以及大小和等于判断

9、println代表输出一行（带换行），printf不带换行，且可以进行格式化输出。

10、==用于判断引用变量指向是否相同，如果判断内容是否相同，使用equal。

11、switch语句中的case具有“穿透性”，因此不要忘记break

12、for(int n : ns)意味直接循环获取可迭代数据类型的所有元素

13、多维数组不同维度长度可不同，直接使用deeptostring打印。

14、面向对象中的this始终指向当前实例，如果没有命名冲突，可以省略this。

15、可变参数String... names，可以接受0个参数，会自动转换为数组形式。

16、先执行类中属性的初始化，再执行构造方法。

17、方法名称相同，参数不同，返回值可以不同，称之为方法**重载（单类）**。 

18、public修饰的属性和方法可被自身和外部调用，private只可以被自身调用（子类无法访问），protected修饰，则可以将访问范围扩展到继承数内部，符合正常逻辑。

19、super表示引用父类属性和方法，大部分情况，有没有都是一样的，但是当父类声明了构造方法时（不带参数除外）或者使用了覆写，则需要用到super。

20、在继承中，向上转型一定会成功，因为父类的属性和方法子类一定有，向下转型不一定成功，需要instanceof判断。

21、继承是is关系，组合是has(implements extend)关系。

22、子类与父类方法名、返回值、参数相同，称为方法**覆写(父子类)**，如果不希望覆写，父类可在对应方法上前加final。（字段被修饰final，则只能静态/构造初始化，无法修改；用final修饰的类本身无法被继承），

23、一个实际类型为Student，引用类型为Person的变量，调用其run()方法，调用的是Person还是Student的run()方法？答案是Student的方法，Java的实例方法调用是基于运行时的实际类型的动态调用，称之为多态。（多态是指，针对某个类型的方法调用，其真正执行的方法取决于运行时期的实际类型）类似于python中的read方法，只要对象包含read方法，就可以运行read，其取决于运行read时的 实际类型。（鸭子类型）

24、abstract用于修饰抽象方法，其必须位于抽象类(abstract class)中，抽象类用于定制规范，子类必须实现其抽象方法。由此引申出面向抽象编程的概念。

25、如果抽象类没有属性，只有抽象方法，则可以定义为接口(interface)，实现该接口需要使用implements，无法多继承，但可以多实现。接口之间可以extends，相当于扩展了规范。接口还可以拥有静态字段，只能static final修饰。

```java
interface Person {
    void run();
    String getName();
}
```

26、抽象类比类抽象，而接口比抽象类更抽象，因此，实例化对象时，总是用接口去引用它，面向接口编程工厂模式。

27、default用于在接口中定义默认方法，这样该方法就不需要手动编写实现。

28、普通实例字段只在自己内存空间下有用，静态字段共享一个独立空间，任意通过一个实例修改静态字段效果都一样。推荐使用类.静态字段的方式访问。

29、静态方法内部只能访问静态字段，没有this指向。实例对象访问静态字段和方法会有警告。java内置了许多工具类，本质上就是类中的静态方法。

30、Java内建的访问权限包括public、protected、private和package（默认）权限；
Java在方法内部定义的变量是局部变量，局部变量的作用域从变量声明开始，到一个块结束；final修饰符不是访问权限，它可以修饰class（不继承）、field（不可变）和method（不覆写）；一个.java文件只能包含一个public类，但可以包含多个非public类。

31、jar包相当于包class文件组织起来了，方便下载和使用，MANIFEST.MF定义主入口函数，可直接运行jar包。

32、包装类型类似于将基本类型引申为引用类型class，可以进行自动包装，包装类内部提供了大量实用方法。

33、静态工厂方法即是以静态方法返回一个以缓存的实例，例如两次Integer.valueOf(100);的返回结果其实指向的是同一个对象。包装类型的比较需要使用equal方法。

34、如果class内部定义若干private修饰的字段，并通过getter、setter操纵，则这种模式称之为javabean。

35、enum枚举类型适合进行查询比较，可以使用==，因为每个enum只被实例化一次。

36、异常通过try、catch捕获并处理，方法定义跟上throws...表示调用该方法可能会抛出异常，因此调用方必须被包裹在try语句块中。main函数是最后一次可以处理异常的地方。

37、反射：JVM为每个加载的class及interface创建了对应的class实例来保存它们相关信息，通过class获取实例信息的方法称之为反射。另外，jvm采用动态加载的方式加载class，及如果主函数中没有相关调用函数，则import语句不执行。

38、反射的应用不符合封装这一特性，它更多的是给工具或者底层框架来使用，目的是在不知道目标实例任何信息的情况下，获取特定信息（字段），甚至修改。getField，并利用get、set读取、设置字段值。

39、反射也可以获取实例的方法信息，通过getMethod获取，并利用invoke执行方法。

40、反射获取构造方法为getConstructor

41、泛型是一种模板，可以适用于多种不同的对象，比如可以定义

ArrayList<String> strList = new ArrayList<String>();也可以定义

ArrayList<Person> personList = new ArrayList<Person>();

42、泛型可以向上转型ArrayList<Integer>->List<Integer>，但不允许ArrayList<Integer>->ArrayList<Number>。

43、泛型除了可以当成数组以外，还可以用于实现接口，比如类对象想要实现比较功能，就必须要实现Comparable<T>这个接口。

44、集合可在内部持有若干其他同类型对象，并对外提供访问接口，相比数组的固定大小和按索引访问，集合可实现可变大小。集合collection包含了三大类，List（有序列表集合）、Set（无重复元素集合）、Map（键值映射表集合）

45、集合的访问总是通过迭代器实现的，Iterator，无需知道内部采用什么方式存储的。

46、list的实现分为ArrayList和LinkedList，由于iterator访问效率最高，因此for each循环默认实现的是iterator遍历。

47、map是以键值对的形式存放数据对象的，通过在map中put('key',object)，便可通过key高效的查询到目标对象，本质是hashmap。同样迭代器使用for each循环，遍历map.ketSet()即可获取到所有key。另外，key值存放是无序的。

48、set是存放无重复元素的集合，hashset（无序）、sortedset（有序）.

49、单例模式，该模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一对象的方式，可以直接访问，不需要实例化该类的对象。优点是解决了频繁创建和销毁的问题，避免对资源的多重占用，比如文件读写，只需要有一个file对象即可。

50、在hashmap中，若要判断两个key对象是否相同，首先需要重写key下的equals方法，这样即可判断引用不同，但内容是否相同了。内容相同之后，由于map中key的无重复性，因此equals下判断相同的两个对象，也需要得到同一个hashcode码才行，所以还需再重写hashcode方法。

**java反射：**

1、java中一切皆对象，除了一般的Object类可以new 出一个对象外，类本身也是java.lang.class的实例对象

```java
public class test {
    private int age;
    public test() {

    }
    public static void main(String[] args) {
        test t1 = new test();
        System.out.println(t1);
        System.out.println(test.class);
        System.out.println(t1.getClass());
    }
}
```

通过类名.class或者实例.getclass()的方式获取class类的实例对象。此外，还有使用class.forname(类全名)的方式获取该类类型。

2、使用类类型可以实例化一个该类的对象，需要注意类必须有无参构造方法。

3、静态加载是指编译时刻就加载了所有可能会用到的Java类，包括new关键字实例化一个对象都是静态加载。

4、动态加载是指运行时刻去加载某些类，通过forname以及newinstance实例化动态加载类的实例对象。动态加载类可以不必重新编译没有修改过的java源文件，一般功能性的接口逻辑可以适宜用动态加载的方式加载。

5、java语言是一种介于编译和解释型的语言，编译是因为需要编译成字节码class文件，而解释是由于运行时是class字节码一条一条读取并执行的，因此可以实现动态加载的功能。

6、利用反射可以获取类的所有方法(getmethods java.lang.reflect.Method)，以及对每个方法有getReturnType，getName，getParameterTypes获取返回值类型、方法名和参数类型。

7、利用反射可以获取类的所有属性（getFileds java.lang.reflect.Filed)还有利用反射获取类的构造方法（类似上一点的方法）。

8、方法的反射操作，可以根据类类型获取指定的方法，通过指定方法名称和参数列表（数组）获取指定方法，并利用invoke执行该函数。可变参数...（既可以通过构造数组传入，也可以直接,隔开多个参数传入）

9、集合中的泛型只是规定了编译时刻下对类型的检查，并没有检查运行时刻添加类型的正确性，因此，可以利用反射invoke对集合进行不同类型变量的add操作。

**java注解：**

1、注解可以提升代码阅读性，简化（优化）代码。

2、注解分为JDK自带注解，比如@Override（覆写），@Deprecated（方法过时），@SuppressWarnings（忽略方法过时）；第三方注解，包括Spring、Mybatis的注解等；还可以自定义注解。

3、按照运行机制分为源码注解、编译时注解、运行时注解





**java复习：**

2、Enum枚举类型类似于静态常量，但是它富有明确意义，通过.name获取输出字符串

.ordinal()获取索引下标。

3、Math类的方法有：abs、max、min、pow（x,y）x的y次方、sqrt、exp(x)e的x次方、log(x)、logx(y)、PI、E、random（0<=x<1）、StrictMath会保证所有平台浮点数计算一致，而math保证优化速度。

4、Random和math.random都是伪随机数，后者调用前者，前者可以指定种子。

5、SecureRandom是真随机数，使用安全的随机种子，这个种子是通过CPU的热噪声、读写磁盘的字节、网络流量等各种随机事件产生的“熵”。伪随机数在密码学中容易被公攻破。

6、try catch语句可以捕获异常并自行处理，后期代码也可执行，若不捕获异常，则编译器捕获，程序立刻退出。

7、泛型，一种数据类型的模板，一次编写，万能匹配，比如List<T>，初始化时，T为String就是String类型的数组，泛型只在编译阶段有效，用于类型检查，class文件执行时不起作用，可以通过反射的例子验证。

8、集合，一般Array数组大小固定，且增删元素不方便，因此提供了List接口，实现类有ArrayList、LinkedList，实际上封装了底层实现代码，又可以保证数组大小可变。

10、引用类型的比较，a.equals(b)，a如果为空，则报空指针异常，需要先排除，但使用Object.equals(a,b)可以避免这种情况。

11、Map中的get(key)实际上是通过key.hashcode方法获取value的，key必须正确实现hashcode方法，如果key的equals方法相同，则hashcode应当返回相同大小。自定义类型的两个方法必须同时覆写。Map的实现类HashMap

12、Set类，只存储Key，保证不重复，需要key实现equals和hashcode方法，实现类Hashset无序，Treeset保证有序，需要key实现comparable<T>接口。

13、字节流为inputstream和outputstream，字符流有reader和writer（自动解编码）。

14、文件读写流需要在finally中及时关闭，简洁的写法为try(source)类似python的with语句。Fileoutputstram和Fileinputstream是两个stream的实现类，都是阻塞式IO处理。

15、Filter模式，类似

InputStream file = new FileInputStream("test.gz");

InputStream buffered = new BufferedInputStream(file);

InputStream gzip = new GZIPInputStream(buffered);

不断的包装，使之具有新的功能，本质上都是Inputstream，这种filter模式称为装饰器模式。

16、注解是放在java源码前的一种标注性元数据，一共分为三类，第一类是给编译器看的解释性注解，如@Override、@SuppressWarnings，不会保存在class文件，第二类是处理class文件的注解，会保存在class中，但不会加载于内存中，第三类是运行时注解，常驻与JVM，进行代码逻辑上的功能扩充。

17、上述三类注解类型即是：

仅编译期：RetentionPolicy.SOURCE；

仅class文件：RetentionPolicy.CLASS；

运行期：RetentionPolicy.RUNTIME。

18、通常定义注解需要使用到@interface字段，而对注解进行修饰的注解称为元注解，常用的元注解有@Target，定义注解可以修饰在什么地方，ElementType.METHOD、ElementType.FIELD等；@Retention，定义注解的生命周期，如上述三个。这两个元注解通常必写。

19、自定义注解的步骤：编写annotation，填写字段属性（以及默认值），配置@Target和@Retention，在目标代码打上注解，但此时无法执行自定义注解的功能，需要手动实现注解的功能。例如，对于字段检查型注解，需要通过反射获取实例的类的字段，通过该字段获取注解，如果注解不为空，再获取实例的字段，并根据注解约束进行验证。

20、静态加载是指编译时刻就需要确定下来的加载类，诸如new Object()，这样的编码方式需要编译成class的时候就需要确定加载类。而动态加载使用class.forName在运行时刻加载需要用到的类，类似数据库底层驱动就使用动态加载的方式。

21、关于反射的操作都是在编译之后执行的，即对编译后的class字节码进行操作的。

22、Thread线程是java启动多线程的方法，通过继承Thread并覆写run方法，在main中通过thread.start来启动并执行。

23、通过使用synchronized（lock)实现加解锁，进而实现同步互斥，lock需要为对象类型，另外，一些原子操作无需锁执行，通过转换一些原子操作可以提高程序执行效率。

24、普通方法使用synchronized相当于该方法执行时需要锁定该对象实例的锁，如果是静态方法使用synchronized修饰，则需要锁定住类的锁。

25、java允许可重入锁，即一个线程内可以对已获得的的锁进行重复加锁，但此时会进行锁数量的+1.

26、wait和notify都只能以锁对象来执行，wait执行后，会将该线程暂时释放锁，notify执行，会随机唤醒其中一个wait的线程，该线程重新获取锁后，执行下一条语句。

27、守护线程是为前台普通线程服务的，当所有用户线程执行完毕后，守护线程会随JVM关闭而一起终止运行。

28、枚举类型将常量进行统一管理，且具有范围定义和意义明确。

枚举通常定义如下：

```java
enum Weekday {
    SUN(1,"sun"), MON(2,"mon"), TUE(3,"tue");
    public final int dayvalue;
    public final String name;

    private Weekday(int dayvalue,String name) {
        this.dayvalue = dayvalue;
        this.name=name;
    }
}
```

29、序列化时将java中的对象信息持久化到存储器中，对象需实现Serializable接口，且在序列化和反序列化时不能改变class信息，ObjectOutputStream.writeObject和ObjectInputStream.readObject（强制类型转换）进行序列化和反序列化操作。