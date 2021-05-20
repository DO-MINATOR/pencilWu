### 为什么学习JVM

- 项目管理、调优需求
- 了解底层垃圾回收算法、内存结构、字节码执行引擎，有助于理解java执行过程
- 大厂面试

### JVM无关性

- 运行平台无关性：编写的java代码，翻译成class文件后，借助于不同系统的JVM，解释/编译成该平台下对应的机器指令。
- 语言无关性：字节码可以通过不同语言翻译器生成，只需要class文件满足格式规范，字节码和符号表即可。不同语言相同功能对应的字节码是相同的，促进了多语言混合编程，如Spark下的scala和java混编。

![image-20200727121725188](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwNzI3MTIxNzI1MTg4LnBuZw)

### JVM架构

![image-20200727132330682](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwNzI3MTMyMzMwNjgyLnBuZw)

- 类加载子系统：加载、链接（验证、准备、解析）、初始化
- 内存自动管理
- 执行引擎：解释器、即时编译器JIT、垃圾回收器

完整框图

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwNzI3MTQ1MzIxMjIyLnBuZw)

### 指令集架构

**基于栈的指令集架构：**

1. 设计实现简单，适用于资源受限的机器
2. 避开寄存器分配，使用零地址指令
3. 指令集简单，主要是出栈、入栈操作，实现相同功能，需要语句更多，效率低
4. 便于移植（这就是JVM的特性以及为什么选用该架构的原因）

```c
iconst_2 //常量2入栈
istore_1
iconst_3 // 常量3入栈
istore_2
iload_1
iload_2
iadd //常量2/3出栈，执行相加
istore_0 // 结果5入栈
```

**基于寄存器的指令集架构：**

1. 完全依赖硬件，不易移植。
2. 多地址指令，指令集较为复杂，但执行高效。
3. 相同功能，使用更少的语句。

```c
mov eax,2 //将eax寄存器的值设为2
add eax,3 //使eax寄存器的值加3
```