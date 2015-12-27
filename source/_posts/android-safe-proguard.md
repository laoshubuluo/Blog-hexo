title: Android-安全-代码混淆
description: Android应用打包时的代码混淆
date: 2015/11/30 20:35:30 
categories: Android
tags: [Android,代码混淆,打包,安全]
---
应用终于要公测了，公测过程需要将apk打包分发到各个渠道（豌豆荚、应用宝等等），为了防止xx反编译源码，需要在应用打包过程中，对apk进行混淆处理。
<!--more-->
# 关于混淆 #
好吧，引用百度词条的说法：
> 代码混淆(Obfuscated code)亦称花指令，是将计算机程序的代码，转换成一种功能上等价，但是难于阅读和理解的形式的行为。代码混淆可以用于程序源代码，也可以用于程序编译而成的中间代码。执行代码混淆的程序被称作代码混淆器。目前已经存在许多种功能各异的代码混淆器。
将代码中的各种元素，如变量，函数，类的名字改写成无意义的名字。比如改写成单个字母，或是简短的无意义字母组合，甚至改写成“__”这样的符号，使得阅读的人无法根据名字猜测其用途。重写代码中的部分逻辑，将其变成功能上等价，但是更难理解的形式。比如将for循环改写成while循环，将循环改写成递归，精简中间变量，等等。打乱代码的格式。比如删除空格，将多行代码挤到一行中，或者将一行代码断成多行等等。

# 开始混淆 #
## 混淆开启 ##

打开**app/build.gradle**文件，增加如下配置：

	//签名配置
	signingConfigs {
	    release {
	        storeFile file("../sign.jks")
	        storePassword "wmgj.amen"
	        keyAlias "amen"
	        keyPassword "wmgj.amen"
	    }
	}
	//build类型
	buildTypes {
	    release {
	        minifyEnabled true
	        debuggable false
	        jniDebuggable false
	        proguardFile('proguard-rules.pro')
	        //proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        signingConfig signingConfigs.release
	    }
	}
其中buildTypes-release节点中**minifyEnabled**属性，即为是否开启混淆，混淆开启后，编译过程会明显变慢。

**proguardFile('proguard-rules.pro')**语句指定了混淆的配置文件（详见下述）。

## 混淆配置 ##

Android Studio集成环境，在创建项目时，就在**根/app/**目录下自动创建好了混淆配置文件**proguard-rules.pro**、**proguard-android.txt**，将此配置文件进行相应修改即可。
如下为验证成功的配置文件（含有第三方SDK避免混淆），具体命令见注释。

	// Add project specific ProGuard rules here.
	// By default, the flags in this file are appended to flags specified
	// in C:\android-SDK-windows/tools/proguard/proguard-android.txt
	// You can edit the include path and order by changing the proguardFiles
	// directive in build.gradle.
	//
	// For more details, see
	//   http://developer.android.com/guide/developing/tools/proguard.html
	
	// Add any project specific keep options here:
	
	// If your project uses WebView with JS, uncomment the following
	// and specify the fully qualified class name to the JavaScript interface
	// class:
	//-keepclassmembers class fqcn.of.javascript.interface.for.webview {
	//   public *;
	//}
	
	// This is a configuration file for ProGuard.
	// http://proguard.sourceforge.net/index.html//manual/usage.html
	
	-dontusemixedcaseclassnames
	-dontskipnonpubliclibraryclasses
	-verbose
	
	// Optimization is turned off by default. Dex does not like code run
	// through the ProGuard optimize and preverify steps (and performs some
	// of these optimizations on its own).
	-dontoptimize
	-dontpreverify
	// Note that if you want to enable optimization, you cannot just
	// include optimization flags in your own project configuration file;
	// instead you will need to point to the
	// "proguard-android-optimize.txt" file instead of this one from your
	// project.properties file.
	
	-keepattributes *Annotation*
	-keep public class com.google.vending.licensing.ILicensingService
	-keep public class com.android.vending.licensing.ILicensingService
	
	//保持 native 方法不被混淆
	-keepclasseswithmembernames class * {
	    native <methods>;
	}
	
	// keep setters in Views so that animations can still work.
	// see http://proguard.sourceforge.net/manual/examples.html//beans
	-keepclassmembers public class * extends android.view.View {
	   void set*(***);
	   *** get*();
	}
	
	// We want to keep methods in Activity that could be used in the XML attribute onClick
	-keepclassmembers class * extends android.app.Activity {
	   public void *(android.view.View);
	}
	
	// For enumeration classes, see http://proguard.sourceforge.net/manual/examples.html//enumerations
	-keepclassmembers enum * {
	    public static **[] values();
	    public static ** valueOf(java.lang.String);
	}
	
	-keep class * implements android.os.Parcelable {
	  public static final android.os.Parcelable$Creator *;
	}
	
	-keepclassmembers class **.R$* {
	    public static <fields>;
	}
	
	// The support library contains references to newer platform versions.
	// Don't warn about those in case this app is linking against an older
	// platform version.  We know about them, and they are safe.
	//如果引用了v4或者v7包
	-dontwarn android.support.**
	-keep class android.support.** { *; }
	
	------------------------// 自定义 start ------------------------------------
	//指定代码的压缩级别
	-optimizationpasses 5
	//包明不混合大小写
	-dontusemixedcaseclassnames
	//不去忽略非公共的库类
	-dontskipnonpubliclibraryclasses
	 //优化  不优化输入的类文件
	-dontoptimize
	 //预校验
	-dontpreverify
	 //混淆时是否记录日志
	-verbose
	 // 混淆时所采用的算法
	-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*
	
	//jar包(同build.gradle里的声明)
	//-libraryjars libs/volley.jar
	
	//API里的类，避免混淆
	-keep public class * extends android.app.Fragment
	-keep public class * extends android.app.Activity
	-keep public class * extends android.app.Application
	-keep public class * extends android.app.Service
	-keep public class * extends android.content.BroadcastReceiver
	-keep public class * extends android.content.ContentProvider
	-keep public class * extends android.app.backup.BackupAgentHelper
	-keep public class * extends android.preference.Preference
	-keep public class * extends android.support.v4.**
	-keep public class com.android.vending.licensing.ILicensingService
	-keep public class java.util.** { *; }
	-dontwarn java.util.**
	
	//model层，避免混淆
	-keep class com.wmgj.amen.entity.** { *; }
	
	//注解，避免混淆
	-keepattributes *Annotation*
	-keep class * extends java.lang.annotation.Annotation { *; }
	//签名，避免混淆
	-keepattributes Signature
	
	//保持自定义控件类不被混淆
	-keepclasseswithmembers class * {
	    public <init>(android.content.Context, android.util.AttributeSet);
	}
	//保持自定义控件类不被混淆
	-keepclasseswithmembers class * {
	    public <init>(android.content.Context, android.util.AttributeSet, int);
	}
	//保持自定义控件类不被混淆
	-keepclassmembers class * extends android.app.Activity {
	   public void *(android.view.View);
	}
	//保持 Parcelable 不被混淆
	-keep class * implements android.os.Parcelable {
	  public static final android.os.Parcelable$Creator *;
	}
	//保持 Serializable 不被混淆
	-keepnames class * implements java.io.Serializable
	//保持 Serializable 不被混淆并且enum 类也不被混淆
	-keepclassmembers class * implements java.io.Serializable {
	    static final long serialVersionUID;
	    private static final java.io.ObjectStreamField[] serialPersistentFields;
	    !static !transient <fields>;
	    !private <fields>;
	    !private <methods>;
	    private void writeObject(java.io.ObjectOutputStream);
	    private void readObject(java.io.ObjectInputStream);
	    java.lang.Object writeReplace();
	    java.lang.Object readResolve();
	}
	
	//保持枚举 enum 类不被混淆 如果混淆报错，建议直接使用上面的 -keepclassmembers class * implements java.io.Serializable即可
	-keepclassmembers enum * {
	  public static **[] values();
	  public static ** valueOf(java.lang.String);
	}
	
	//不混淆资源类
	-keepclassmembers class **.R$* {
	    public static <fields>;
	}
	
	//文件缓存地址等，不混淆
	-keep class com.wmgj.amen.appmanager.** { *; }
	
	//------------------------ 自定义 end ----------------------------------------
	
	//-------------------- 记录生成的日志数据,gradle build时在本项目根目录输出----------------
	//apk 包内所有 class 的内部结构
	-dump class_files.txt
	//未混淆的类和成员
	-printseeds seeds.txt
	//列出从 apk 中删除的代码
	-printusage unused.txt
	//混淆前后的映射
	-printmapping mapping.txt
	//-------------------- 记录生成的日志数据，gradle build时 在本项目根目录输出-end----------------
	
	//------------------------ 第三方 start ------------------------------------
	//融云
	-keepclassmembers class fqcn.of.javascript.interface.for.webview {
	 public *;
	}
	-keepattributes Exceptions,InnerClasses
	-keep class io.rong.** {*;}
	-keep class * implements io.rong.imlib.model.MessageContent{*;}
	-keepattributes Signature
	-keepattributes *Annotation*
	-keep class sun.misc.Unsafe { *; }
	-keep class com.google.gson.examples.android.model.** { *; }
	-keepclassmembers class * extends com.sea_monster.dao.AbstractDao {
	 public static java.lang.String TABLENAME;
	}
	-keep class **$Properties
	-dontwarn org.eclipse.jdt.annotation.**
	-keep class com.ultrapower.** {*;}
	
	//百度
	-dontwarn org.apache.**
	-keep class org.apache.**{*;}
	-keep class com.baidu.**{*;}
	-keep class vi.com.gdi.bgl.**{*;}
	-dontwarn com.baidu.**
	
	//Volley
	-keep class com.android.volley.** {*;}
	-keep class com.android.volley.toolbox.** {*;}
	-keep class com.android.volley.Response$* { *; }
	-keep class com.android.volley.Request$* { *; }
	-keep class com.android.volley.RequestQueue$* { *; }
	-keep class com.android.volley.toolbox.HurlStack$* { *; }
	-keep class com.android.volley.toolbox.ImageLoader$* { *; }
	
	//友盟
	-dontwarn com.umeng.**
	-keep class com.umeng.**{*;}
	-keepclassmembers class * {
	   public <init>(org.json.JSONObject);
	}
	
	//支付宝
	//-keep class com.alipay.android.app.IAliPay{*;}
	//-keep class com.alipay.android.app.IAlixPay{*;}
	//-keep class com.alipay.android.app.IRemoteServiceCallback{*;}
	//-keep class com.alipay.android.app.lib.ResourceMap{*;}
	
	//腾讯
	-dontwarn com.tencent.**
	-keep class com.tencent.**{*;}
	//-keep class com.tencent.android.tpush.**  {* ;}
	//-keep class com.tencent.mid.**  {* ;}
	-keep class com.jg.**{*;}
	
	//Butterknife
	//-keep class butterknife.** { *; }
	//-dontwarn butterknife.internal.**
	//-keep class **$$ViewInjector { *; }
	//-keepclasseswithmembernames class * {
	//    @butterknife.* <fields>;
	//}
	//-keepclasseswithmembernames class * {
	//    @butterknife.* <methods>;
	//}
	
	//Jackson
	//-dontwarn com.fasterxml.jackson.**
	//-keep class com.fasterxml.jackson.**
	//-keepnames class com.fasterxml.jackson.**
	//
	//-keepattributes Signature
	//-keepattributes *Annotation*
	//-keep class sun.misc.Unsafe { *; }
	
	//------------------------ 第三方 end ----------------------------------------

按照上述配置文件，会将部分不可混淆的内容避免掉，如第三方SDK、用于序列化的领域层对象、枚举对象等，如果混淆后应用crash或功能异常，多半是由于部分不可混淆的代码被混淆导致，需要详细排查。

## 混淆查看 ##
混淆之后打包的apk,反编译查看，会发现很多类名及方法被处理成没有意义的a\b\c\d字母，从而达到混淆的目的。

如果排查异常等情况需要查看真实文件名，可参考混淆过程中生成的映射文件。
另，如果应用将异常日志上传到了第三方SDK，如友盟，则平台支持上传mapping.txt文件，并通过mapping文件的映射关系自动还原真实的类名和方法名。

	//-------------------- 记录生成的日志数据,gradle build时在本项目根目录输出----------------
	//apk 包内所有 class 的内部结构
	-dump class_files.txt
	//未混淆的类和成员
	-printseeds seeds.txt
	//列出从 apk 中删除的代码
	-printusage unused.txt
	//混淆前后的映射
	-printmapping mapping.txt
	//-------------------- 记录生成的日志数据，gradle build时 在本项目根目录输出-end----------------

over~