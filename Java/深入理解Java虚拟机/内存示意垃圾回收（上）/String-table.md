### String概述

1. 可通过直接""（双引号）方式声明，也可以new创建，前者在stringtable中，后者在堆中。
2. String为final修饰，无法继承。
3. 实现了Serializable接口，支持序列化，实现Comparable接口，支持比较大小。
4. JDK8及以前，使用final char[]存储字符，JDK9开始使用final byte[]，原因，char占用两个字节，但大部分字符串都为拉丁文，即一个字节，因此改为byte[]+编码方式的存储方式保存字符串。

### String的不可变性

当新声明一个字符串时，是在新的内存空间中进行赋值，无法对已有字符串进行修改。replace方法同上。

```java
public void test1() {
    String s1 = "abc";//字面量定义的方式，"abc"存储在字符串常量池中
    String s2 = "abc";
    s1 = "hello";
    
    System.out.println(s1 == s2);//判断地址：false
    System.out.println(s1);//hello
    System.out.println(s2);//abc
}
```

对现有字符串进行拼接操作时，也是重新指定内存区域，不对原有value修改。

```java
public void test2() {
    String s1 = "abc";
    String s2 = "abc";
    s2 += "def";
    System.out.println(s2);//abcdef
    System.out.println(s1);//abc
}
```

当调用string的replace()方法修改指定字符或字符串时，重新指定内存区域赋值，不能使用原有的value进行赋值。

```java
ublic void test3() {
    String s1 = "abc";
    String s2 = s1.replace('a', 'm');
    System.out.println(s1);//abc
    System.out.println(s2);//mbc
}
```

### String底层结构HashTable

StringTable是不会存储相同内容的字符串，是一个固定大小的Hashtable，默认值大小长度是1009。如果放进String Pool的String非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后造成的影响就是当调用String.intern()方法时性能会大幅下降。

```java
public class StringTest2 {
    public static void main(String[] args) {       
        BufferedReader br = null;
        try {
            br = new BufferedReader(new FileReader("words.txt"));
            long start = System.currentTimeMillis();
            String data;
            while ((data = br.readLine()) != null) {
                //如果字符串常量池中没有对应data的字符串的话，则在常量池中生成
                data.intern();
            }

            long end = System.currentTimeMillis();

            System.out.println("花费的时间为：" + (end - start));//1009:143ms  100009:47ms
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }
}
```

- -XX:StringTableSize=1009 ：程序耗时 143ms
- -XX:StringTableSize=100009 ：程序耗时 47ms

### String的内存分配过程

Java提供的8中基本类型和Sring类型都提供了缓存概念（通常用常量池实现），使其访问迅速，避免频繁创建、销毁。String的常量池实现较为特殊：

1. 直接用" "声明的字符串字面量会存放在常量池StringTable中。
2. 其他String对象可通过intern方法将其保存在StringTable中。

JDK7开始，字符串常量从永久代移到堆空间，使得字符串的GC更加方法。因为永久代空间小，GC较少，而堆空间足够大，字符串可被及时回收。

### 字符串拼接(+操作)

1. 常量和常量拼接（如"hello "+"world")，直接在stringtable生成字符串，利用了编译器优化。
2. 常量池不会存放相同内容。
3. 拼接前后，只要有一个是变量（final修饰除外），就会new StringBulider，append()，toString()。（tostring类似new string，因此结果存放在堆中）（**注意：**这个new string的对象不会将字符串保存在stringtable中，而人为的new string会保存）
4. 拼接后的结果通过intern方法可以将字符串保存在stringtable中，并返回该对象地址。

```java
public void test1() {
    String s1 = "a" + "b" + "c";//编译期优化：等同于"abc"
    String s2 = "abc"; //"abc"一定是放在字符串常量池中，将此地址赋给s2
    /*
     * 最终.java编译成.class,再执行.class
     * String s1 = "abc";
     * String s2 = "abc"
     */
    System.out.println(s1 == s2); //true
    System.out.println(s1.equals(s2)); //true
}
```

```c
 0 ldc #2 <abc>	//字节码发现编译器已经将a b c合并为abc
 2 astore_1
 3 ldc #2 <abc>
 5 astore_2
 6 getstatic #3 <java/lang/System.out>
 9 aload_1
10 aload_2
11 if_acmpne 18 (+7)
14 iconst_1
15 goto 19 (+4)
18 iconst_0
19 invokevirtual #4 <java/io/PrintStream.println>
22 getstatic #3 <java/lang/System.out>
25 aload_1
26 aload_2
27 invokevirtual #5 <java/lang/String.equals>
30 invokevirtual #4 <java/io/PrintStream.println>
33 return
```

```java
public void test2(){
    String s1 = "javaEE";
    String s2 = "hadoop";

    String s3 = "javaEEhadoop";
    String s4 = "javaEE" + "hadoop";//编译期优化

    //如果拼接符号的前后出现了变量，则相当于在堆空间中new String()，具体的内容为拼接的结果：javaEEhadoop
    String s5 = s1 + "hadoop";
    String s6 = "javaEE" + s2;
    String s7 = s1 + s2;

    System.out.println(s3 == s4);//true
    System.out.println(s3 == s5);//false
    System.out.println(s3 == s6);//false
    System.out.println(s3 == s7);//false
    System.out.println(s5 == s6);//false
    System.out.println(s5 == s7);//false
    System.out.println(s6 == s7);//false

    //intern():判断字符串常量池中是否存在javaEEhadoop值，如果存在，则返回常量池中javaEEhadoop的地址；
    //如果字符串常量池中不存在javaEEhadoop，则在常量池中加载一份javaEEhadoop，并返回此对象的地址。
    String s8 = s6.intern();
    System.out.println(s3 == s8);//true
}
```

```java
/*
体会执行效率：通过StringBuilder的append()的方式添加字符串的效率要远高于使用String的字符串拼接方式！
 改进的空间：
    在实际开发中，如果基本确定要前前后后添加的字符串长度不高于某个限定值highLevel的情况下,建议使用构造器实例化：
    StringBuilder s = new StringBuilder(highLevel);//new char[highLevel]
 */
@Test
public void test6(){

    long start = System.currentTimeMillis();

    // method1(100000);//4014
    method2(100000);//7

    long end = System.currentTimeMillis();

    System.out.println("花费的时间为：" + (end - start));
}

public void method1(int highLevel){
    String src = "";
    for(int i = 0;i < highLevel;i++){
        src = src + "a";//每次循环都会创建一个StringBuilder、String
    }
}

public void method2(int highLevel){
    //只需要创建一个StringBuilder
    StringBuilder src = new StringBuilder();
    for (int i = 0; i < highLevel; i++) {
        src.append("a");
    }
}
```

append方式效率高于普通string的+操作，分析原因：

1. StringBuilder的append()的方式：
   - 自始至终中只创建过一个StringBuilder的对象
   - 使用String的字符串拼接方式：创建过多个StringBuilder和String的对象
2. 使用String的字符串拼接方式：
   - 内存中由于创建了较多的StringBuilder和String的对象，内存占用更大；
   - 如果进行GC，需要花费额外的时间。
3. 改进的空间：
   - 在实际开发中，如果基本确定要前前后后添加的字符串长度不高于某个限定值highLevel的情况下，建议使用构造器实例化：`StringBuilder s = new StringBuilder(highLevel);`内部有char[]数组防止频繁扩容

### new String()操作

- new String(“ab”)会创建几个对象？

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("ab");
    }
}
```

```c
 0 new #2 <java/lang/String>	  //在堆中创建了一个 String 对象
 3 dup
 4 ldc #3 <ab>					//在字符串常量池中放入 “ab”（如果之前字符串常量池中没有 “ab” 的话）
 6 invokespecial #4 <java/lang/String.<init>>
 9 astore_1
10 return
```

- new String(“a”) + new String(“b”) 会创建几个对象？

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("a") + new String("b");
    }
}
```

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/a5eb5d8adac93afb8fc8071b12864fd0.png)

1. 变量相加，先创建StringBuilder()，1次
2. 两此new String()，共4次
3. append之后tostring中有个new string，但这个new string不会保存string table，因此只有1次

### JDK6、7关于intern()到底放没放字符串到stringtable中？

```java
public class StringIntern {
    public static void main(String[] args) {
        String s = new String("1");
        s.intern();//这方法其实没啥用，调用此方法之前，字符串常量池中已经存在"1"
        String s2 = "1";
        /*
            jdk6：false   jdk7/8：false
            因为 s 指向堆空间中的 "1" ，s2 指向字符创常量池中的 "1"
         */
        System.out.println(s == s2);

        // 执行完下一行代码以后，字符串常量池中，是否存在"11"呢？答案：不存在！！
        String s3 = new String("1") + new String("1");//s3变量记录的地址为：new String("11")
        /*
            如何理解：jdk6:创建了一个新的对象"11",也就有新的地址。
            jdk7:此时常量中并没有创建"11",而是在常量池中记录了指向堆空间中new String("11")的地址（节省空间）
         */
        s3.intern(); // 在字符串常量池中生成"11"。
        String s4 = "11";//s4变量记录的地址：使用的是上一行代码代码执行时，在常量池中生成的"11"的地址

        // jdk6：false  jdk7/8：true
        System.out.println(s3 == s4);
    }
}
```

```java
public class StringIntern1 {
    public static void main(String[] args) {
        //执行完下一行代码以后，字符串常量池中，是否存在"11"呢？答案：不存在！！
        String s3 = new String("1") + new String("1");//new String("11")
        //在字符串常量池中生成对象"11"
        String s4 = "11";
        String s5 = s3.intern();

        // s3 是堆中的 "ab" ，s4 是字符串常量池中的 "ab"
        System.out.println(s3 == s4);//false

        // s5 是从字符串常量池中取回来的引用，当然和 s4 相等
        System.out.println(s5 == s4);//true
    }
}
```

```java
public class StringExer1 {
    public static void main(String[] args) {
        //在下一行代码执行完以后，字符串常量池中并没有"ab"
        String s = new String("a") + new String("b");//new String("ab")
        /*
            jdk6中：在串池中创建一个字符串"ab"
            jdk8中：串池中没有创建字符串"ab",而是创建一个引用，指向new String("ab")，将此引用返回
         */
        String s2 = s.intern();
        System.out.println(s2 == "ab");//jdk6:true  jdk8:true
        System.out.println(s == "ab");//jdk6:false  jdk8:true
    }
}
```

**注意：**在JDK7及以后，当执行完`String s = new String("a") + new String("b")`后，如果先执行s.intern()，则ab常量永远指向该堆中对象，即使再创建对象="ab"也会指向该对象，如果先执行="ab"，则在stringtable中创建该副本，此后s.intern()也会指向stringtable中的字符串常量。占山为王。