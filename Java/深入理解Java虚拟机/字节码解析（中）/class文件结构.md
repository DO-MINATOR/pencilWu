### class存储内容总览

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/36da8e95cc7e4e538be161c94fea8f03~tplv-k3u1fbpfcp-watermark.image)

字节码是二进制流，但无法直接运行，其对应着类或接口的定义信息，其中包含了大量指令，每个指令由一个字节长度的操作码和后跟0+个操作数构成，如：

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/f11d2215cfe441409cb27f87736fc01e~tplv-k3u1fbpfcp-watermark.image)

查看字节码可以通过binary viewer查看，也可通过IDEA插件byte viewer查看。Class文件结构不规则，没有分隔符，因此它通过结构形式存储，主要为表和无符号数。无符号数u1、u2、u4、u8代表1、2、4、8字节的无符号数，用于描述数字、索引引用、数量值或UTF-8编码的字符串。表是由无符号数或表复合而成的，以_info结尾，表长度由最开始的u2指明。

### class总体结构

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/73ff6db240374d1e910be6c34f86f759~tplv-k3u1fbpfcp-watermark.image)

| 类型           | 名称                | 说明                   | 长度    | 数量                  |
| -------------- | ------------------- | ---------------------- | ------- | --------------------- |
| u4             | magic               | 魔数,识别Class文件格式 | 4个字节 | 1                     |
| u2             | minor_version       | 副版本号(小版本)       | 2个字节 | 1                     |
| u2             | major_version       | 主版本号(大版本)       | 2个字节 | 1                     |
| u2             | constant_pool_count | 常量池计数器           | 2个字节 | 1                     |
| cp_info        | constant_pool       | 常量池表               | n个字节 | constant_pool_count-1 |
| u2             | access_flags        | 访问标识               | 2个字节 | 1                     |
| u2             | this_class          | 类索引                 | 2个字节 | 1                     |
| u2             | super_class         | 父类索引               | 2个字节 | 1                     |
| u2             | interfaces_count    | 接口计数               | 2个字节 | 1                     |
| u2             | interfaces          | 接口索引集合           | 2个字节 | interfaces_count      |
| u2             | fields_count        | 字段计数器             | 2个字节 | 1                     |
| field_info     | fields              | 字段表                 | n个字节 | fields_count          |
| u2             | methods_count       | 方法计数器             | 2个字节 | 1                     |
| method_info    | methods             | 方法表                 | n个字节 | methods_count         |
| u2             | attributes_count    | 属性计数器             | 2个字节 | 1                     |
| attribute_info | attributes          | 属性表                 | n个字节 | attributes_count      |

### 常量池

- 内容最丰富，存放编译期生成的字面量和符号引用，加载到方法区成为运行时常量池，帮助字段和方法的解析。
- 由u2指明常量池容量，后面u2-1个字节是常量池表。
- 常量池长度由前置的u2指明![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/edac6c18e223454399f0e87d7687f7d0~tplv-k3u1fbpfcp-watermark.image)其值为0x0016，也就是22。 需要注意的是，这实际上只有21项常量。索引为范围是1一21。为什么呢？ > 通常我们写代码时都是从0开始的，但是这里的常量池却是从1开始，因为它把第0项常量空出来了。这是为了满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义，这种情况可用索引值0来表示。
- 主要包含两大类
  - 字面量：基本数据类型、UTF-8字符串。
  - 符号引用：类、字段、方法、接口的引用，一般为全限定名、简单名称和描述符

全限定名通过UTF-8编码格式说明，如`com/dsh/test/Demo;`，简单名称如`getName`，描述符用来描述数据类型、参数列表、返回值等信息。

| 标志符 | 含义                                                 |
| ------ | ---------------------------------------------------- |
| B      | 基本数据类型byte                                     |
| C      | 基本数据类型char                                     |
| D      | 基本数据类型double                                   |
| F      | 基本数据类型float                                    |
| I      | 基本数据类型int                                      |
| J      | 基本数据类型long                                     |
| S      | 基本数据类型short                                    |
| Z      | 基本数据类型boolean                                  |
| V      | 代表void类型                                         |
| L      | 对象类型，比如：`Ljava/lang/Object;`                 |
| [      | 数组类型，代表一维数组。比如：`double[][][] is [[[D` |

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/299657b5c6a64de3acccd6a97a8a7b06~tplv-k3u1fbpfcp-watermark.image)

**符号引用和直接引用：**虚拟机在class加载后才进行动态链接，因此class文件中并不保存符号引用所对应的内存地址，当转换为运行池常量池中的数据后，才和内存进行绑定。

- 符号引用：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到了内存中。
- 直接引用：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是与虚拟机实现的内存布局相关的。

**类型和结构细节：**得知了常量池表总长度后，通过解读结构说明得出具体内容。

![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/96c71a88175c41d0bc3cd16505349c97~tplv-k3u1fbpfcp-watermark.image)![img](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/59cd7d34b2ed4dd1abd5367b16db8420~tplv-k3u1fbpfcp-watermark.image)