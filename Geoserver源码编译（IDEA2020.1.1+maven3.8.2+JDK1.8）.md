﻿# 一、编译环境准备
&#8195;&#8195;记录下Geoserver源码编译部署整个流程，编译环境使用的IDEA2020.1.1+maven3.8.2+JDK1.8，之前使用JDK14在编译时会出现问题。geoserver源代码建议从github上拉取[github:geoserver源码](https://github.com/geoserver/geoserver)，第一次从官网上下载的源代码没有编译成功。下面简单介绍下相关环境配置。
## 1、IDEA2020.1.1
&#8195;&#8195;IDEA2020.1.1安装教程：[IDEA安装教程](https://www.bilibili.com/video/BV1NK4y1P76F?share_source=copy_web)
## 2、maven3.8.2
&#8195;&#8195;从官网上下载：[maven官网](https://maven.apache.org/)。点击左侧列表中download，选择	apache-maven-3.8.2-bin.zip下载后解压（记录时版本已经到了3.8.3）
![在这里插入图片描述](https://img-blog.csdnimg.cn/24c152d6c9634806a98e897abccc1cee.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
配置环境变量：
* M2_HOME：E:\apache-maven-3.8.2-bin\apache-maven-3.8.2\bin（maven目录下的bin目录）
* MAVEN_HOME：E:\apache-maven-3.8.2-bin\apache-maven-3.8.2（maven目录）
* Path：%MAVEN_HOME%\bin

配置阿里云镜像：
&#8195;&#8195;打开conf文件夹下的settings.xml文件，在mirrors部分添加如下代码：

```xml
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
</mirrors>
```
建立本地仓库：
&#8195;&#8195;在根目录下新建本地仓库目录文件夹maven-repo
![在这里插入图片描述](https://img-blog.csdnimg.cn/87a6c7b7b20a40fd86495373fb142cfb.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
&#8195;&#8195;打开conf文件夹下的settings.xml文件，添加如下代码：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c29cd9063d9f476b98cd93ea8d8c333f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
```xml
<localRepository>E:\apache-maven-3.8.2-bin\apache-maven-3.8.2\maven-repo</localRepository>
```
&#8195;&#8195;配置完成后，在IDEA中创建maven项目后会自动在maven仓库下拉取jar包。
## 3、JDK1.8
&#8195;&#8195;JDK1.8安装参考：[JDK安装](https://blog.csdn.net/weixin_37601546/article/details/88623530)
# 二、源代码编译及项目启动
## 1、项目构建
* 进入geoserver源代码文件夹，进入.\geoserver-2.14.0\src目录下，输入cmd进入命令行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/593a8f3a2a1042c6a1d4e4b00a852924.png)
* 在命令行下使用maven命令进行项目构建，下载相关依赖文件，注意此处使用跳过测试功能，否则会报错。
```bash
mvn -DskipTests install
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d8ec83d835c47a194714410d30c8cf8.png)
* 等待一段时间后出现BUILD SUCCESS说明项目构建成功。
![在这里插入图片描述](https://img-blog.csdnimg.cn/31edf40f9605484180933978ae24cfcc.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* 编译时可能会出现FileSystemResourceStore.java文件异常报错，找到该文件对应路径.\src\platform\src\main\java\org\geoserver\platform\resource可以用NotePad直接打开，将第240行报错的代码段super.finalize();注释后重新编译。
## 2、项目配置
* 使用IDEA打开geoserver源码，点击File->New->Project from Existing Sources
![在这里插入图片描述](https://img-blog.csdnimg.cn/f3667a7a021c4fb09cfb955a864634b1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_18,color_FFFFFF,t_70,g_se,x_16)
* 选择src目录下的pom.xml配置文件进行项目导入
![image-20211014085031577](C:\Users\Asus\AppData\Roaming\Typora\typora-user-images\image-20211014085031577.png)
* 项目导入完成后点击Setting进行项目配置，将编译器更改为Eclipse
![在这里插入图片描述](https://img-blog.csdnimg.cn/d80d9135542e4f1194061fe3a5ca8504.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* maven配置，分别设置maven所在路径，maven的setting.xml配置文件路径，maven的本地仓库路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/eb4a041b71db4f4394b1d2a83fad286b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* 最后点击右侧maven，再点击一次刷新，否则有可能会报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/9981f85144ed43e09701168044a8b4da.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_16,color_FFFFFF,t_70,g_se,x_16)

## 3、项目启动
* 点击启动项配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/92d2d8c5b2194bd9acb74e32f1569916.png)
* 启动项配置，Main class配置为org.geoserver.web.Start，VM options配置用于指定geoserver数据目录，添加-DGEOSERVER_DATA_DIR=F:\geoserver-2.14.0\geoserver-2.14.0\data，工作路径Working directory设置为F:\geoserver-2.14.0\geoserver-2.14.0\src\web\app，启动项模块为gs-web-app，JRE为安装的JRE路径。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f5352b7740734999b24800b5dc3226ac.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* 运行项目，打开web模块中的Start类，点击运行
![在这里插入图片描述](https://img-blog.csdnimg.cn/d2c5970deab4475fbb7366e1c8e8ca96.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_13,color_FFFFFF,t_70,g_se,x_16)
* 运行可能出现Command line is too long错误
![在这里插入图片描述](https://img-blog.csdnimg.cn/01a2513c80ae4fb9973024e9bae7e62e.png)
* 点开项目文件夹里的.idea
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9886f9b5f5e4b97a0c137a931f4be8f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_11,color_FFFFFF,t_70,g_se,x_16)
* 在标签下添加此标签component name="PropertiesComponent"内加入如下标签
```xml
<property name="dynamic.classpath" value="true" />
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d643fc88989545c7a32a066a08132788.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* 出现如下图所示，说明运行成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/2e322e4cce91435fa37fe8829c08b8da.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* 在浏览器中输入http://localhost:8080/geoserver/web/就可以实现应用访问
![在这里插入图片描述](https://img-blog.csdnimg.cn/7be1a4f5280149f6a873c3c8ad4fd1c6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* 访问后若出现页面404报错，在官网下载geoserver的安装包进行解压
![在这里插入图片描述](https://img-blog.csdnimg.cn/3c2f8db36b7946e5a06f97806b9f9358.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_15,color_FFFFFF,t_70,g_se,x_16)
* 解压后的文件夹中有data_dir文件夹，将其拷入源码的F:\geoserver-2.14.0\geoserver-2.14.0\data路径下
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ef474d4966841cd8e4806e77107fabc.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* 启动项更改VM option配置为-DGEOSERVER_DATA_DIR=F:\geoserver-2.14.0\geoserver-2.14.0\data\data_dir![在这里插入图片描述](https://img-blog.csdnimg.cn/5b4c29e1d0c64622a0a2fa69679a982c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGFwZXJUaWdlcjIzMw==,size_20,color_FFFFFF,t_70,g_se,x_16)
* 至此完成geoserver源码的部署
## 参考文章
* [开源地图服务geoserver源代码研究实践（IntelliJ IDEA2017导入工程、环境搭建）](https://blog.csdn.net/u010608964/article/details/83719105)
* [GeoServer二级开发-环境配置 IDEA](https://blog.csdn.net/weixin_38670190/article/details/116695214)
* [Maven环境配置](https://www.bilibili.com/video/BV12J411M7Sj?share_source=copy_web)
* [IDEA2020.1.1安装](https://www.bilibili.com/video/BV1NK4y1P76F?share_source=copy_web)
* [Intellij Quickstart](https://docs.geoserver.org/latest/en/developer/quickstart/intellij.html)

