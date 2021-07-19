### 双亲委派机制

- 沙箱安全机制：保证JDK核心类库只能由bootstrapClassLoader加载器加载，防止核心类篡改和注入。
- 类的唯一性：向上递归加载时，如果已经有加载器加载过该类，则无法再次加载相同的类，除非使用其他加载器，并破坏双亲委派模型。

### Tomcat打破双亲委派

Tomcat是一个web容器，不同应用可能需要依赖到相同第三方类库的不同版本，而JDK提供的加载器由于类的唯一性，导致无法实现。

1. 提供多个加载器负责加载不同context下的类库，并隔离开来。
2. 同一个web容器下相同的第三方且版本也相同的依赖可以由相同加载器加载，这样可以减小元空间占用。
3. web容器也有自己依赖的类库，不能与应用类库相混淆，因此单独抽离出一个类加载器。
4. 需要支持热加载，无论是第三方类库还是servlet的修改，如果发现文件变动，需要启动tomcat的热加载功能，关闭当前context对应的contextClassLoader，重新启动contextClassLoader，加载对应目录下的class文件。

![image-20210715100911845](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210715100911845.png)

### Tomcat核心加载器

- commonClassLoader：Tomcat基本的类加载器，加载的类可被Tomcat容器本身和各个Webapp访问。
- CatalinaClassLoader：Tomcat容器私有加载器，其加载的类对web应用不可见。
- SharedClassLoader：web应用公共加载器，负责加载一些公共且相同版本的类库。
- WebAppClassLoader：每个应用下的每个类库独有的加载器，负责应用间的隔离、热加载。每个class对应一个classloader就是为了方便热加载，这样不会影响到其他类及其对应加载器。

同一个JVM下可存在相同全限定名的class，只要加载器不同，则类也不同，这也是tomcat实现应用类库隔离的原因。

### 热部署和热加载

![image-20210715104920729](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20210715104920729.png)

热加载仅会重启个别类加载器，而热部署是将整个context及该容器下的资源全部释放再重启，当然该webapp对应的所有类加载器都会重启。