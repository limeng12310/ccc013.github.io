---
title: Android官方文档Guide阅读系列(1)--Application Fundamentals
date: 2016-04-20 14:10:54
tags: 
 - Android
 - 阅读笔记
---
最近打算好好看看android的官方文档，`Training`的已经看过了，打算看下`Guide`指导文档，这个比`Training`会介绍得更加详细。

谷歌的官网因为被墙，所以这里介绍一个国内的镜像[Android 6.0官网](http://android.xsoftlab.net/index.html)，本篇文章是第一篇文章，地址是[Application Fundamentals](http://android.xsoftlab.net/guide/components/fundamentals.html)

主要是翻译`Guide`文档的内容，但不一定会逐句翻译，因为英语水平问题，可能会有选择的翻译一些自己认为比较重要的内容，但是介绍的知识点会尽可能保证都翻译到。

## 应用基础(Application Fundamentals)
Android的app是使用Java语言编写的。Android SDK工具是用来将你写的代码以及一些数据和资源文件，编译成一个APK，APK是一个android的包，是一个后缀名为`.apk`的归档文件(archive file)。一个APK文件包含了一个android app的所有内容，也是使用android系统的设备用来安装app的文件。

一旦在一个设备上安装了，每个 android app 都是运行在自己的安全沙箱中：

 * android的操作系统是一个多用户的Linux系统，每个app是一个不同的用户
 * 默认情况下，系统会给每个app分配一个独立的Linux用户ID(这个ID只是系统使用，app是不知道这个ID的)。系统会给一个app中所有文件都设置了权限，只有分配给这个app的用户ID才可以使用这些文件
 * 每个进程都有自己的虚拟机(virtual machine,VM),因此每个app都是相互隔离地运行自己的代码的
 * 默认情况，每个app都是运行在自己的Linux进程。当app的组件需要被执行时，Android会开始其进程，而在其不用执行或者系统必须给其他应用分配内存(recover memory)时会关掉这个进程。

通过上述方式，Android系统实现了最少特权原则(the principle of least privilege)。也就是对于每个app，默认情况，它只可以使用用于完成其工作的组件。这就创造了一个非常安全的环境，每个app都无法获取它没有得到权限的系统功能或服务。

然而，对每个app，还是有方法可以跟其他app分享数据以及使用系统的服务的：

 * 给两个app分享同一个Linux的用户ID是有可能的，在这种情况下它们是可以使用对方的文件。出于节省系统资源的考虑，拥有一样用户ID的apps会分配到同一个Linux进程中，并且共享同一个VM(这些apps都必须签署同样的证书)
 * 一个应用是可以要求获取使用设备数据的权限，如用户的联系人，短信，SD卡，照相机，蓝牙等等。所有的应用权限都必须在安装的时候得到用户的同意。
 
以上内容包含了基本的关于一个android应用如何存在于系统中，本文接下来的将介绍以下内容：

 * 定义app中的核心框架组件
 * `mainfest`文件，用于声明组件以及要求设备功能
 * 跟app代码分开的资源文件，用于让你的应用可以在面对多种设备配置时优雅地优化其行为

***
### 1.应用组件(App Components) 
应用组件是一个Android应用的基本建筑基石。每个组件是一个系统可以进入应用的不同点。但不是所有的组件是用户实际的入口点，其中一些需要相互依赖，但每一个都是作为一个实体存在并且扮演一个特殊的角色——每一个组件都是一个帮助开发者定义其应用整体行为的基石。

总共有4种不同类型的应用组件。每一种类型都是用于一个明显的用途，并且都有一个清楚的定义了其如何创建和摧毁的生命周期。

##### Activities
一个活动表示用户界面上一个单独的屏幕。例如，在一个电子邮件的应用中会有一个展示新邮件列表的活动，一个活动是用于写信，以及一个读取邮件内容的活动。尽管在这个应用中这些活动一起运作形成了一个有凝聚力的用户体验，但每个活动都是相互独立的。同样地，另一个不同的应用是可以打开这些活动中的任何一个(在获得该电子邮件应用的允许的前提下)。例如，一个照相应用可以打开这个电子邮件应用中的写信的活动，以便于用户可以分享一张图片。

##### Services
一个服务是一个运行在后台，执行长时间运算或者执行远程进程的工作的组件。服务是没有用户界面的。例如，当用户在使用其他应用的时候，一个服务正在后台播放音乐，或者它是在通过网络获取一些数据但没有阻碍用户与一个活动界面进行交互。其他的组件，如一个活动，可以开启服务并让它运行，或者绑定这个服务来与它进行交互。

##### Content providers
一个内容提供器是管理一个用于分享的应用数据集。开发者可以通过文件系统，一个SQLite数据库，网络，或其他固定的应用可以访问的存储位置来保存数据。通过内容提供器，其他的应用可以查询甚至修改数据(在这个内容提供器允许前提下)。例如，Android系统提供了一个管理用户联系人信息的内容提供器。正因为如此，任何一个应用在有合法的权限下都可以查询这个内容提供器(如`ContactsContract.Data`)来读写一个特定的人的信息。

##### Broadcast receivers
一个广播接收器是一个用于响应系统全局广播通知的组件。很多广播都是起源于系统——例如，一个广播通知屏幕要关闭，电量低，或者截屏。应用也可以初始化广播——例如，让其他应用知道有些数据被下载到设备中并且可以提供给它们使用。尽管广播接收器没有显示一个用户界面，但是它们可以创建一条状态栏的通知来提醒用户有一个广播事件发生了。但更普遍的情况是，一个广播接收器仅仅是一个通向其他组件的“门(gateway)”，并且一般只做很少量的工作，例如，广播接收器可能就是开启一个基于广播事件的执行一些任务的服务。
***
### 2.Activating Components
在4种组件类型中有三种--活动、服务和广播接收器--都是通过一个叫做`intent`的异步信息启动的。Intents可以在运行时将独立的组件绑定在一起(可以将它们看做是需要从其他组件中要求一个`action`的信使)，无论这个组件是否属于你的应用。

通过使用一个`Intent`对象可以创建一个`intent`，它可以使用一个信息来启动一个特定的组件或者一类组件，即`intent`是分为显式的和隐式的。

对于活动和服务，一个intent定义了需要执行的`action`(如发送或者查看)，有可能还会特定数据的`URI`，比如需要打开一个网页的时候，这个`URI`就是网页的网址了。在某些情况下，活动在使用一个intent后，还可以接受返回的结果，这个结果也是通过`Intent`返回的。

对于广播接收器，intent只是简单定义了正在广播的通知。

对于内容提供器，它是由`ContentResolver`来启动的。它可以处理跟内容提供器所有的直接事务。

对不同组件有不同的开启方法：

* 对于活动，可以传递一个`Intent`给`startActivity()`或者`startActivityForResult()`(当需要开启的活动返回一个结果时);
* 对于服务，传递一个`Intent`给`startService()`方法来开启服务，绑定服务则是传递给`bindService()`;
* 初始化一个广播可以通过传递`Intent`给如`sendBroadcast()`,`sendOrderedBroadcast()`,或者`sendStickyBroadcast()`;
* 通过调用`ContentResolver`的`query()`可以实现对内容提供器的查询。
***
### 3. The Manifest File
所有的组件都必须在一个文件——`AndroidManifest.xml`中声明，Android 系统才能使用它，这个文件必须是在应用项目文件的根目录中。

除了用于声明组件，它还有以下这些功能：

* 声明应用所需要的用户权限，比如网络连接或者是读取用户的联系人；
* 声明该应用要求的最低 API Level，(在Android Studio中这个功能是放在`build.gradle`中了)
* 声明应用需要用到或者要求的硬件和软件的功能特性，比如照相机，蓝牙，或者是一个多点触摸的屏幕
* 应用需要链接到的 API 库，比如 Google Maps library
* 等等

##### 声明组件
`manifest`的主要任务是声明组件，比如，一个声明活动的例子如下：

    <?xml version="1.0" encoding="utf-8"?>
    <manifest ... >
        <application android:icon="@drawable/app_icon.png" ... >
            <activity android:name="com.example.project.ExampleActivity"
                      android:label="@string/example_label" ... >
            </activity>
            ...
        </application>
    </manifest>
没有在这个文件声明的组件是不可以使用的，除了广播接收器外。

广播接收器即可以在该文件中声明，也可以在代码中动态地创建，并使用`registerReceiver()`方法在进行注册。

##### 声明组件的功能
`Intent`的真正作用体现在使用隐式intent上，这种intent简单描述了需要执行的行为类型，有时还会说明需要使用的数据，然后系统就会寻找可以执行这种行为的组件并启动它，而如果有多种组件，那么用户可以选择使用哪一个。

通过在`AndroidManifest.xml`声明组件时，使用一个`<intent-filter>`可以声明该组件的功能，也就是其能执行的行为。

下面是一个邮件应用中用户写信的一个活动的声明例子：

    <manifest ... >
        ...
        <application ... >
            <activity android:name="com.example.project.ComposeEmailActivity">
                <intent-filter>
                    <action android:name="android.intent.action.SEND" />
                    <data android:type="*/*" />
                    <category android:name="android.intent.category.DEFAULT" />
                </intent-filter>
            </activity>
        </application>
    </manifest>
这个例子中的活动可以响应需要`ACTION_SEND`的intent

##### 声明应用的必要条件
由于Android的设备非常多，但不是所有的设备都拥有同样的功能和特性，在`AndroidManifest.xml`文件中声明应用所需要的硬件和软件功能和特性是有必要的。

不过大部分这些声明都只是信息，系统也不会读取它们，但对于如 Google Play 会读取它们然后给用户进行过滤。

例如，你的应用需要使用照相功能并且使用 Android 2.1(API Level 7)的 API，应该如下所示：

    <manifest ... >
        <uses-feature android:name="android.hardware.camera.any"
                      android:required="true" />
        <uses-sdk android:minSdkVersion="7" android:targetSdkVersion="19" />
        ...
    </manifest>
这样，当不具备以上要求的设备是无法从 Google Play 上安装的。当然上述的`android:required`也可以设置为`false`，这表示你的应用使用照相功能，但并不要求设备拥有这个功能，所以在运行的时候会检查设备是否有照相机，并且会在适当的时候关闭这个功能。
***
### 4. 应用资源
一个android应用除了代码外，还有一些其他的资源，比如图片，音频文件，以及任何与应用的视觉展示有关的东西。使用应用资源可以在不修改代码的情况下升级应用的不同功能特性，并且可以使应用适配不同的设备配置(如不同的语言和屏幕尺寸)。

对每个在项目中使用到的资源，SDK 编译工具都会定义一个独一无二的整型 ID，这个可以在代码中或者 XML 中用来引用这个资源。

使用应用资源可以有利于适配不同的设备配置，比如对于在 XML 中使用字符串，可以将字符串翻译成其他语言并保存在不同的文件，并且对该文件名字使用一个限定符(如`res/values-fr/`是用于法语的字符串)。

Android提供了很多不同的限定符，它是一个短字符串用于放在资源文件名字中。例如，对于布局，通常需要创建不同的布局文件，因为设备的屏幕方向和尺寸。当屏幕是垂直的(`portrait`)或者是水平的(`landscape`),可以定义两个不同的布局文件来对应这两种情况，而系统会根据当前设备的方向来自动使用合适的布局。
***
### 5. 总结
  第一篇简单介绍了 android 的开发语言是 `Java` ，这个是最基础的，也是学习android首先需要掌握的基础语言，而且我也认为想要在android这条道路上走得更远，Java还不能只是了解，还必须要好好掌握好，毕竟到时如果要阅读源码，读的可以都是用java写的代码，所以重新好好系统学学java也是一个近期的计划。当然，最近看到有说可以使用其他语言来写android，比如`kotlin`这门语言，还有就是说`swift`要支持 Android了，不过这个目前只是 `NDK` 支持而已。

其次还介绍了 android 的四大组件， 目前用得比较多的当然还是 `Activity` 了，其他三个组件用得比较少，甚至还没怎么使用过，不过这也是目前开发的经验比较少，做过的项目不多。

然后就是`AndroidManifest`文件，用来注册四大组件，声明应用需要的权限和硬件必须具备的功能等，还有就是说明了 Android 中除了纯 Java 代码外，还有其他资源文件，包括图片，布局文件等，这里需要根据设备的屏幕大小，分辨率，屏幕方向等分别设置对应的资源文件。

第一次写文章，主要是翻译自官方文档，因为英语水平和表达水平不是很好，所以有翻译错误和表达不好的地方都欢迎提出来。
