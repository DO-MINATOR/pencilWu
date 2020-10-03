### 内核版本&发行版本

内核作为基础，各大厂商在此编写软件、服务作为其发行版

### 应用

主要作为服务器操作系统，稳定、可靠、开源

### 网络设置

桥接模式：直接与本机共用同一ip。NAT：利用vmware设置的虚拟网卡，可以联网，默认无法访问本机所在的局域网（需手工设置同一网段），只能和本机进行通信。

### 基本命令

`mkdir -p /a/b/` 递归创建目录

### 软硬链接

![image-20200806091125855](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806091125855.png)

![image-20200806091055022](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806091055022.png)

![image-20200806091706688](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806091706688.png)

注意不加s，为创建硬链接，加s，创建软链接（与windows的快捷方式一致）。通过`ls -i`查看硬链接id号相同，软链接不同。软链接权限最终还是要取决于原文件权限。软链接原文件要写绝对路径

### 搜索命令

1. `locate 文件名`在数据库中搜索，`updatedb`更新数据库，搜索速度极快，搜索规则遵循updatedb.conf

2. `whereis ls`搜索命令所在位置，`which ls`查找出该命令的别名

3. find命令![image-20200806201300657](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806201300657.png)

   ![image-20200806201402143](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806201402143.png)

   注意，使用通配符需要加' " '

   ![image-20200806201628923](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806201628923.png)

4. grep命令![image-20200806202634771](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806202634771.png)

   注意，find搜索文件，而grep搜索字符串

### 压缩命令

![image-20200806203206182](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806203206182.png)

![image-20200806203252965](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806203252965.png)

![image-20200806203641670](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806203641670.png)

![image-20200806203750174](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806203750174.png)

![image-20200806203848054](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806203848054.png)

![image-20200806204104948](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806204104948.png)

输入-C 路径指定解压位置，ztvf/jtvf查看压缩包内容

### 快捷键

![image-20200806212901730](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806212901730.png)

### 通配符

![image-20200806215740343](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200806215740343.png)