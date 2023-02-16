---
layout:     post
title:      GSettings
subtitle:   Linux下GSettings使用
date:       2022-05-21
author:     Key
header-img: img/post-gnome.jpg
catalog: true
tags:
    - Linux
---

## 前言
在linux上，系统中经常需要存储一些的配置和数据，一般情况下有几种方式可以保存

- 环境变量，可以保证每次启动都是相同的环境变量，不易被改变
- 文件保存，适合长时间保存，安全，且可以用做进程间通讯
- GSettings，容易使用

其他的方式这篇文章暂时不介绍，主要介绍一下GSettings，GSettings固然不是最有效和最好的保存方式，但是在一些场景下，但是GSettings也有很多优缺点，

```
优点有以下几点：
1. 方便读取，修改，没有权限要求，当然这也是一个缺点
2. 其他进程对其修改，有相应的信号发出，同步较快
3. 代码中无法直接创建一个GSettings的path、id、键值对，只能提前写好并修改配置
4. 数据保存在当前用户态中，每个用户都有自己的一份GSettings配置
5. 数据类型丰富，对QT和C++开发较为友好

缺点有：
1. 无法确定修改者
2. 修改方式过多
3. root下无法使用
```



## 正文

​	以下就来介绍一下怎么去使用GSettings，以及一些GSettings相关知识。

### GSettings命令<a id="shell">

​	GSettings常用命令基本可以用来读写，编译安装GSettings相关文件，以下命令中返回值自于下面[示例的xml文件](#xml),以下只介绍了一些常用的命令，另外如果觉得麻烦，可以安装**dconf-editor** (gsettings可视化工具)。

```shell
// 编译安装xml文件和override文件: 安装/usr/share/glib-2.0/schemas路径下的匹配的文件
$ glib-compile-schemas /usr/share/glib-2.0/schemas

// 获取键值：获取id为com.deepin.controlcenter.test中的enabled的值
$ gsettings get com.deepin.controlcenter.test enabled
	true

// 设置键值：设置id为com.deepin.controlcenter.test中的enabled的值
$ gsettings set com.deepin.controlcenter.test false

// 恢复某个key的默认值,恢复后可用get查看
$ gsettings reset com.deepin.controlcenter.test enabled

// 显示已经安装的gsettings的id
$ gsettings list-schemas
	com.deepin.controlcenter.test
	com.deepin.controlcenter.test1
	com.deepin.controlcenter.test2
	......
	
// 获取id下的keys列表
$ gsettings list-keys com.deepin.controlcenter.test
	enabled
	brightness

// 获取某个id下的键值，可以无参数，则获取当前系统中全部id的键值
$ gsettings list-recursively com.deepin.controlcenter.test
	com.deepin.controlcenter.test enabled true
	com.deepin.controlcenter.test brightness 30

// 获取某个key的取值范围（部分自定义的值需要提前在xml中配置）
$ gsettings range com.deepin.controlcenter.test

// 恢复某个id下的全部值为默认值（不建议使用）
$ gsettings reset-recursively com.deepin.controlcenter.test

```

​		

### GSettings相关文件保存位置

​	在linux中，GSettings一般由四个部分组成，

```
1. xml文件，提供基础的键值对，告知相应的路径和默认值
2. override文件，提供默认值的覆盖，做定制化使用，**可以不存在override**
3. bin目录下的一些命令。主要是GSettings和glib-compile-schemas命令
4. 当前用户下，存在的配置，其中包括已经被修改的值。
```

##### 1. xml文件

​	xml文件的作用是提供给Gsettings的Path，id和键值对的，其中采用了schema的规范创建的，其中文件内容基本如下<a id="xml">：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<schemalist>
    <schema path="/com/deepin/controlcenter/" id="com.deepin.controlcenter.test">
        <key type="b" name="enabled">
            <default>true</default>
            <summary>Enable Test</summary>
            <description>'false' disables the test function of the control center</description>
        </key>
        <key type="i" name="brightness">
            <default>30</default>
            <summary>Global brightness</summary>
            <description>System global brightness.</description>
        </key>
    </schema>
</schemalist>
```

​	对于schema必须保证以下几点：

1. path两头必须都有/，否则会验证失败;
2. schema文件的命名规则必须是**＊.gschema.xml**(例如：com.deepin.controlcenter.gschema.xml)，否则这个规则文件将无法被正确编译安装，最终无法被gsettings使用;
3. 如果想让gsettings能被dconf-editor所读取，则必须指定path属性。

​	当有了这个文件后，需要将文件放入到系统中的一个目录中以便管理，目前linux上基本上是将文件放入到**/usr/share/glib-2.0/schemas**和**/usr/local/share/glib-2.0/schemas**下，当然，直接把文件放入目录中并没有什么用，因为这个就相当于你的源代码，还没有进行编译，无法被gsettings给识别。[GSettings命令](#shell)中已经介绍如何安装。



##### 2. override文件

​	override(覆盖)，很好理解，就是用来覆盖默认配置的文件，可以看到上述的文件中，有**default**的默认值，但是有些时候，默认值并不能在不同的情况下显示不同的内容，举个例子，有个app，我想在中国地区的时候显示中文，英文地区显示英文，那我可以在便衣英文版本的时候加上相应的配置，保证在默认情况下是中文，有override文件时替换为英文，这样就可以达到目的（只是举个例子，实现方式一般不这么麻烦）。

​	override文件也有一定的格式要求，格式如下：

```ini
[com.deepin.controlcenter.test]
enabled=false
brightness=50
[com.deepin.controlcenter.test1]
enabled=true
```
​	override文件采用的是ini的格式，方括号中填写的是**schemas文件**中的**id**，下面的键值对同**schemas文件**中的键值对，要保证数据类型一致，部分情况下，建议在**value**前后加上**引号**(单双引号皆可，但是要英文引号)。

​	override文件也有一些要注意的点：

1. 格式、数据类型和键值、id都要注意(本人有两三次写错的经历)；

2. 同一个键值对可以被两次覆盖，同一文件中下方的会覆盖上方的值，**不同文件中，需要比较文件名的ASII码大小，ASII码大文件会后读取，则会覆盖先读取到的内容**;

3. 文件命名规则必须为**＊.gschema.override**(例如：com.deepin.controlcenter.gschema.override)。

   ```ini
   #例1，这种情况下，enabled的默认值就会变成true
   [com.deepin.controlcenter.test]
   enabled=false
   brightness=50
   enabled=true
   
   #例2,这种情况下，02的文件中的值会替换文件1中的值，则enabled为false，brightness为50
   # 文件1: 02_controlcenter.gschema.override
   [com.deepin.controlcenter.test]
   enabled=false
   brightness=50
   # 文件2: 01_controlcenter.gschema.override
   [com.deepin.controlcenter.test]
   enabled=true
   brightness=70
   
   ```

​	override文件存储位置也同xml一样，一般存放在**/usr/share/glib-2.0/schemas**和**/usr/local/share/glib-2.0/schemas**下，也需要通过[GSetting相关命令](#shell)编译安装后才会生效。



##### 3. 一些可执行程序

​	这个其实没什么好介绍的，由于手边目前没有linux机器，也无法补充deb包信息，所以后期再补充。



##### 4. 配置后的数据

​	由于GSettings是用户级的，所以数据是保存在用户目录下的，目前没有linux机器，所以这里也后续补充。

​	