title: Android-Technology-NDK-实现JNI
description: Android开发技术：通过NDK-JNI调用本地方法
date: 2016/03/06 18:50:30 
categories: Android
tags: [Android,Technology,NDK,JNI,C,C++,本地方法,AndroidStudio]
---
甚是惭愧，搞了两年Android，居然没研究过JNI，对NDK也是一知半解。近期因为项目需要，抽时间研究了一下如何在AndroidStudio平台上，开发能够执行本地C、C++方法的Android程序，也就是所说的SDK-JNI。
<!--more-->

# 特别感谢 #
文章参考：
> http://blog.chinaunix.net/uid-26000296-id-5296362.html
> http://blog.csdn.net/sodino/article/details/41946607

如有侵权，请与我联系~我会尽快处理并删除侵权内容。

# 关于NDK #
**Android NDK 是在SDK前面又加上了“原生”二字，即Native Development Kit，因此又被Google称为“NDK”**

## NDK产生的背景 ##

Android平台从诞生起，就已经支持C、C++开发。
众所周知，Android的SDK基于Java实现，这意味着基于Android SDK进行开发的第三方应用都必须使用Java语言。
但这并不等同于“第三方应用只能使用Java”。
在Android SDK首次发布时，Google就宣称其虚拟机Dalvik支持JNI编程方式，
也就是第三方应用完全可以通过JNI调用自己的C动态库，
即在Android平台上，“Java+C”的编程方式是一直都可以实现的。

不过，Google也表示，使用原生SDK编程相比Dalvik虚拟机也有一些劣势，Android SDK文档里，找不到任何JNI方面的帮助。
即使第三方应用开发者使用JNI完成了自己的C动态链接库（so）开发，但是so如何和应用程序一起打包成apk并发布？
这里面也存在技术障碍。比如程序更加复杂，兼容性难以保障，无法访问Framework API，Debug难度更大等。
开发者需要自行斟酌使用。

于是NDK就应运而生了。NDK全称是Native Development Kit。
NDK的发布，使“Java+C”的开发方式终于转正，成为官方支持的开发方式。
NDK将是Android平台支持C开发的开端。

## 为什么使用NDK ##

1.代码的保护。由于apk的java层代码很容易被反编译，而C/C++库反编译难度较大。

2.可以方便地使用现存的开源库。大部分现存的开源库都是用C/C++代码编写的。

3.提高程序的执行效率。将要求高性能的应用逻辑使用C开发，从而提高应用程序的执行效率。

4.便于移植。用C/C++写得库可以方便在其他的嵌入式平台上再次使用。

## NDK简介 ##

1.NDK是一系列工具的集合
NDK提供了一系列的工具，帮助开发者快速开发C（或C++）的动态库，并能自动将so和java应用一起打包成apk。
NDK集成了交叉编译器，并提供了相应的mk文件隔离CPU、平台、ABI等差异，开发人员只需要简单修改mk文件
（指出“哪些文件需要编译”、“编译特性要求”等），就可以创建出so。

2.NDK提供了一份稳定、功能有限的API头文件声明
Google明确声明该API是稳定的，在后续所有版本中都稳定支持当前发布的API。
从该版本的NDK中看出，这些API支持的功能非常有限，
包含有：C标准库（libc）、标准数学库（libm）、压缩库（libz）、Log库（liblog）。

## NDK的目录结构说明 ##

build：    该目录存放的使用NDK的mk脚本，mk脚本指定了编译参数

docs：     该目录存放的是NDK的使用帮助文档

platforms：这里面存放的是与各个Android版本相关的平台（x86，arm，mips）相关C语言库和头文件

prebuilt： 预编译工作目录

samples：  存放的是演示程序

sources：  存放的是NDK工具链的C语言源码

tests：    测试相关的文件

toolchains：工具链，存放了三种架构的静态库等文件

ndk-build.cmd：Window平台使用NDK的命令

ndk-build：Linux平台使用NDK的命令

## NDK安装及配置 ##
安装：

NDK安装和SDK安装相同，[下载NDK包，如android-ndk-r10e-windows-x86_64.exe，](http://www.cnblogs.com/tc310/p/3938353.html)解压到指定路径即可（我的路径为：C:\android-ndk-r10e）。**注意：路径不可含有中文和空格。**

环境变量配置：

进入：计算机-高级系统设置-环境变量，在系统变量栏目中，

增加“NDK_ROOT”属性，值为NDK解压路径“C:\android-ndk-r10e”，

修改“Path”属性的值，增加值“%NDK_ROOT%”

# 开发JNI工程 #
使用Android Sutdio创建一个新的工程后，接下来记录创建NDK工程的基本步骤。

**Step: 1. 添加native接口**

注意写好native接口和System.loadLibrary()即可了，并无特别之处。


    public class MainActivity extends Activity {
    private static final boolean DEBUG = true;
    private static final String TAG = "MainActivity";

    static {
        System.loadLibrary("JniTest");
    }

    // 新定义的native方法，意思是该方法的具体实现交给c语言实现
    public native String getStringFromNative();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Log.i("", "调用C方法:" + getStringFromNative());
    }
	}


**Step: 2.执行Build->Make Project**
![](http://7xrjck.com1.z0.glb.clouddn.com/blog%2Fimage%2FQQ%E6%88%AA%E5%9B%BE20160306215801.png-blog)
	
这一步骤执行一下，验证工程中并无其它错误，并对工程进行了编译，生成了.class文件.
.class文件的生成路径是在 app_path/build/intermediates/classes/debug下的

**Step: 3.javah生成c头文件**

点击"View->Tool Windows->Terminal"，即在Studio中进行终端命令行工具.执行如下命令生成c语言头文件。

这里需要注意的是要进入 <Project>\app\src\main的目录下执行javah命令，为的是生成的 .h 文件同样是在<Project>\app\src\main路径下，可以在Studio的工程结构中直接看到。

操作命令：
javah -d jni -classpath <SDK_android.jar>;<APP_classes> lab.sodino.jnitest.MainActivity

    javah -d jni -classpath C:\android-sdk-windows\platforms\android-17\android.jar;..\..\build\intermediates\classes\debug rat.test.MainActivity

最后会在<Project>\app\src\main\jni\目录下生成.h文件，注意生成的**文件名格式**。

![](http://7xrjck.com1.z0.glb.clouddn.com/blog%2Fimage%2FQQ%E6%88%AA%E5%9B%BE20160306215950.png-blog)

**Step: 4.编辑c文件**

创建并编辑**rat_test_MainActivity.c**文件

实现头文件中的方法，具体功能为直接return回一个String，并且使用android_log打印出相关日志。
代码如下：

	#include <jni.h>
	#include <android/log.h>
	
	#ifndef LOG_TAG
	#define LOG_TAG "ANDROID_LAB"
	#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG, __VA_ARGS__)
	#endif
	
	#ifndef _Included_rat_test_MainActivity
	#define _Included_rat_test_MainActivity
	#ifdef __cplusplus
	extern "C" {
	#endif
	
	JNIEXPORT jstring JNICALL Java_rat_test_MainActivity_getStringFromNative
	  (JNIEnv * env, jobject jObj){
	      return (*env)->NewStringUTF(env,"Hello From JNI!");
	  }
	
	#ifdef __cplusplus
	}
	#endif
	#endif

到这里后，我们再执行一个"Build->Make Project"，发现"Messages Gradle Build"会给出提示如下：

	Error:Execution failed for task ':app:compileDebugNdk'. 
	> NDK not configured. 
	Download the NDK from http://developer.android.com/tools/sdk/ndk/.Then add ndk.dir=path/to/ndk in local.properties. 
	(On Windows, make sure you escape backslashes, e.g. C:\\ndk rather than C:\ndk)

这里提示了NDK未配置，并且需要在工程中的local.properties文件中配置NDK路径。好了，提示很清楚了，那我们就进入下一步吧。

**Step: 5.配置NDK**

这一步包括两个动作：

1.指明ndk路径
![](http://7xrjck.com1.z0.glb.clouddn.com/blog%2Fimage%2FQQ%E6%88%AA%E5%9B%BE20160306220735.png-blog)

2.修改build.gradle配置

工程中共有两个build.gradle配置文件，我们要修改的是在<Project>\app\build.gradle这个文件。为其在defaultConfig分支中增加上**ndk**参数

	android {
    compileSdkVersion 17
    buildToolsVersion "22.0.1"

    defaultConfig {
        applicationId "rat.test"
        minSdkVersion 9
        targetSdkVersion 17
        versionCode 1
        versionName "1.0"

        ndk {
            moduleName "JniTest"
            ldLibs "log", "z", "m"
            abiFilters "armeabi", "armeabi-v7a", "x86"
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
	}

以上配置代码指定的so库名称为JniTest，链接时使用到的库，对应android.mk文件中的LOCAL_LDLIBS，及最终输出指定三种abi体系结构下的so库。

这时，再执行"Build->Rebuild Project"，就可以编译出so文件了。

编译出来的库文件被Studio输出到了下图的路径中
![](http://7xrjck.com1.z0.glb.clouddn.com/blog%2Fimage%2F20160306221243.png-blog)

**Step: 6.安装运行**

打印出Log:

调用C方法：Hello From JNI!

大功告成！


