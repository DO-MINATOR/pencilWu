[toc]

## 使用Typora+PicGo+阿里云OSS实现markdown图床保存功能

总所周知，在Typecho内置的markdown编辑器中上传一幅图片的步骤是较为繁琐的，尤其是对于那些图文并茂的文章，typecho实现图片显示的过程是，首先需要作者将图片上传到公网环境中，然后按照markdown语法将图片链接写在文章中需要呈现图片的位置。下图显示的是typecho上传图片的过程。这样上传图片的方式是在让人难以接受。

![image-20200518213341379](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518213341379.png)

那，有没有什么方法可以像word那样直接ctrl+c复制图片，然后直接ctrl+v粘贴图片，并且该过程还要在markdown笔记中实现？方法其实是有的，接下来将要介绍的Typora就是一款本地在线编辑markdown的工具。

## Typora

关于Typora，经常使用markdown写作的，应该都了解，应该说是目前市面上最好用的markdown编辑器了，只此一家可以将markdown的书写和渲染无缝的有机结合。其他大多数都是左右分屏，且通过仿word的快捷键，使得稍微熟悉word的童鞋也可以很快写出优美的markdown。

## PicGo

好了，关于Typora的介绍就先讲到这里，今天的主角其实是它的子模块，在偏好设置中，关于图像设置中，进行如下设置，可以看到在进行插入图片时，我们选择的是上传图片（由于之后Typora写好的笔记需要上传到我们的博客，因此图片地址只能是公网环境下的url），其他选项都只能保存在本地磁盘中，这样就无法无缝对接在博客平台上了。

![image-20200518214023360](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518214023360.png)

![image-20200518214104735](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518214104735.png)

之后在上传服务中，这里选择的是PicGo-Core(command line)方式，与app方式的不同之处在于，app是一个独立的客户端，在上传图片过程中，客户端需要常驻，这种方式较为消耗资源，而command-line方式仅是上传图片功能所需要的最基本服务，占用资源极小，不过该方式配置起来较为麻烦，网上大多数教程也都是基于app方式进行的，今天，我就将带领一步一步实现基于command-line方式下的图片上传图床功能。

首先，选择下载或更新，这里会根据你选择方式，下载不同组件，如果是app方式，就会引导去下载一个Picgo客户端，否则只会下载一个大小约17Mb的组件。

![image-20200518215235245](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518215235245.png)

下载完成后，在C:\Users\用户名\AppData\Roaming\Typora\picgo\win64下会出现一个pigco客户端，不过只能以命令行方式运行。输入".\picgo.exe -v"后出现当前版本，表示安装成功。

![image-20200518215608324](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518215608324.png)

继续通过".\picgo.exe -h"命令查看使用说明，在下方选择使用模块进行插件配置

![image-20200518215810741](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518215810741.png)

不知道如何使用没关系，通过查阅官方文档，得知use使用说明。

![image-20200518220020201](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518220020201.png)

输入".\picgo.exe use"出现如下提示，按照顺序依次选择图床所在地（aliyun）、传输过程（path）、插件（无，本教程无需插件）选择完后，出现successfully字样，代表初始化成功。

![image-20200518220153853](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518220153853.png)

![image-20200518220253827](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518220253827.png)

![image-20200518220543260](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518220543260.png)

最后一步，还需要配置config，路径位于C:\Users\用户名\\.picgo下，对于config文件，官网也有说明。

![image-20200518221010151](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518221010151.png)

这里我们使用的是阿里云的图床，因此，关注的内容如下。

![image-20200518221114990](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518221114990.png)

以我的配置文件为例。

```json
{
  "picBed": {
    "current": "aliyun",//当前图床，选择aliyun
    "uploader": "aliyun",//上传者，选择aliyun
    "transformer": "path",//方式，path
    "aliyun": {//阿里云图床具体配置，参数会在阿里云oss章节中进行展开
      "accessKeyId": "*********************",
      "accessKeySecret": "*********************",
      "bucket": "imagebag",
      "area": "oss-cn-chengdu",
      "path": "img/"
    }
  },
  "picgoPlugins": {}
}
```

这里需要关注的主要内容就是"accessKeyId"、"accessKeySecret"、"bucket"、"area"和"path"了，具体参数在阿里云oss搭建中再具体展开。在配置完成后，Pigco的command-line方式就算配置完成了。接下来搭建阿里云图床。

## 阿里云OSS

所谓图床，就是一个专门用于存储图片的云服务器，经常进行剪辑、PS的小伙伴肯定对其不陌生，什么又拍云、七牛云...这里借助阿里云OSS实现一个图传环境。关于OSS服务的购买这里不再赘述，需要的小伙伴，可以看看别的专门讲解OSS选购与配置的博客。

![image-20200518222130044](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518222130044.png)

这里可以看到，我配置了一个名为"imagebag"的图床仓库，别问为什么不叫iamgebed，哈哈，当然是因为别人在用，bucket名称是全阿里云OSS唯一的，通过该名称定位到你的仓库的。然后存储目录自己创建一个img目录。

![image-20200518222440645](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518222440645.png)

在权限管理中一定要选择公共读，否则会导致上传图片失败。至此，基于阿里云OSS的图床环境就搭建完成了。接下来，就是将OSS的系统参数写在Piggo下的config.json中。还记得之前的几个参数吗，现在只需要配置"accessKeyId"、"accessKeySecret"和、"area"了。

"accessKeyId"和"accessKeySecret"的参数在你的个人控制台中。右上角，选择AccessKey管理。之后按照系统引导就会得到属于你的Id和密钥，注意不要泄露，阿里云的很多产品服务都需要用到这两个参数。

![](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/diawoncs.JPG)

关于"area"参数的配置，在阿里云提供的[配置手册][配置手册]中可以查看到，需要依据你购买的OSS所在地理位置进行选择。

一切配置OK，现在回到Typora，在图像设置中，选择验证图片上传选项。如果没有问题的话，验证就会成功。

![image-20200518223125323](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518223125323.png)

![image-20200518223219215](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518223219215.png)

好了，现在在回到Typora的编辑区，试试复制、拖动或者插入图片，一旦插入后，系统便会自动上传到我们创建的阿里云图床。编辑完成之后，直接复制全文到Typecho，现在文章的图片链接就是可用的了。

![image-20200518223634933](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200518223634933.png)

## 写在最后

其实，本文暂时还没将Typora的很多优点展示出来，今天提到的图片自动上传功能只是冰山一角，除此之外，还有word转markdown，导出多种形式，智能代码行等，最方便的要数快捷插入了，什么表格呀、引用呀、链接呀、注解呀，这些用markdown写起来真是要死人惹，而这些在Typora中只需一个快捷键。

通过Typora书写markdown，妈妈再也不用担心我的博客产量低了。

[配置手册]: https://www.alibabacloud.com/help/zh/doc-detail/31837.htm?spm=a2c63.p38356.a3.3.179112f0PBtYui

