title: Android-Technology-应用维持活性，后台存活维活
description: Android开发技术：应用后台维持活性
date: 2016/07/06 18:50:30 
categories: Android
tags: [Android,Technology,后台,维活,性能]
---
近期开发的项目属于IM即时通讯类型，需要应用在最低能耗的情况下，长时间存活在系统后台，用于保证能够实时接收其他用户发来的消息。看起来很简单的问题，但由于Android手机碎片化、各种第三方定制ROM等限制，以至于实现起来一波三折，遂将过程记录下来~

<!--more-->

# 特别感谢 #
文章参考：
> http://www.cnblogs.com/lzl-sml/p/5222310.html
> https://github.com/drinkWaterForThirsty/androidKeepAlive
> http://blog.csdn.net/think_soft/article/details/7299438
> http://www.jianshu.com/p/2889a69a89c6

如有侵权，请与我联系~我会尽快处理并删除侵权内容。

# 正文#

## 引言 ##

为什么我们的后台Service会被结束掉，目前能想到的有如下情况：

1.Android系统内存回收机制

2.各厂商对后台程序的一个管理制度（就是允许程序后台运行那个）

3.第三方软件的清理(360什么的)

4.其他

**测试结果：不同的手机，不同的Android版本保活效果各有差异~。最难绕过的是个厂商对“后台程序保活”管理。**

## 通过互拉维活 ##

互拉的方式目前有如下9种：

**1、可以通过监听系统广播来把自己拉起来。**
通过监听开机广播、网络变更广播、锁屏解锁屏广播等，将服务拉起来。

**2、可以多个app相互拉。**
目前市面上，很多应用都汇集成第三方SDK，任何一款应用启动，都会拉起集成同样SDK的其他应用，比如集成了极光推送、个推推送SDK的应用，一个被唤醒，则拉起所有。

**3、可以把自己的服务搞成前台服务**
前台服务是哪些被认为用户知道的并且在内存低的时候不允许系统杀死的服务。前台服务必须给状态栏提供一个通知，他被放到了“正在进行中（Ongoing）”标题之下，这就意味着直到这个服务被终止或从前台删除通知才能被解除。
例如，一个播放音乐的音乐播放器服务应该被设置在前台运行，因为用户明确的知道它们的操作。状态栏中的通知可能指明了当前的歌曲，并且用户启动一个跟这个音乐播放器交互的Activity。

要让你的服务在前台运行，需要调用startForeground()方法，这个方法需要两个参数：一个唯一标识通知的整数和给状态栏的通知，如：

	Notification notification = new Notification(R.drawable.icon, getText(R.string.ticker_text),
	        System.currentTimeMillis());
	Intent notificationIntent = new Intent(this, ExampleActivity.class);
	PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent, 0);
	notification.setLatestEventInfo(this, getText(R.string.notification_title),
	        getText(R.string.notification_message), pendingIntent);
	startForeground(ONGOING_NOTIFICATION, notification);

要从前台删除服务，需要调用stopForeground()方法，这个方法需要一个布尔型参数，指示是否删除状态栏通知。这个方法不终止服务。但是，如果你终止了正在运行的前台服务，那么通知也会被删除。

注意：startForeground()和stopForeground()方法是在Android2.0(API Level 5)中引入的。为了在比较旧的平台版本中运行你的服务，你必须使用以前的setForeground()方法---关于如何提供向后的兼容性，请看startForeground()方法文档。

**4、在service的onstart方法里返回 STATR_STICK**

当服务被异常终止时，是否重启服务?

有些文章里面在用这个做保活时，修改的是flag，在我实际测试中是无效。有效的做法是直接返回参数。另外默认的flags值为0，是START_STICKY_COMPATIBILITY。如下：

	@Override
	public int onStartCommand(Intent intent,intflags,intstartId) {
	// TODO Auto-generated method stub
	return START_STICKY;
	//return super.onStartCommand(intent, flags, startId);
	}

测试结果：

魅族的机子：无效，不管默认还是修改参数，在DDMS里面直接结束进程后都不会重启服务。

其它三台机子（9100没测）：默认参数的情况下就会重启服务，return START_STICKY 会重启，return START_NOT_STICKY 不会重启。

其它：1.用360一键清理，或者360超级ROOT的手机优化，会杀死进程，过会儿还是会重启，只是会慢很多，大概是在排队重启服务。

一次测试完后确保服务重启后，执行了onStartCommand函数。

**5、添加Manifest文件属性值为android:persistent=“true”**

主要是针对第一种kill服务的情况，内存回收机制。由于这个测试比较难搭建。360清理什么把后台的进程都杀的，体现不出优先级这样的概念。我的建议是能提高就提高。

若有root权限：

android:persistent="true",并放入system/app中

测试结果：效果一般，三星9100上用360等清理工具杀不掉进程，在华为G730上没什么效果.

**6、覆写Service的onDestroy方法**

这个在所有能触发onDestory的情况下都是有效的。4台测试机都测试过。直接startService 或者发送广播重启都可以 。

但能触发onDestory的情况，我不知道内存回收会不会触发。。我的测试方法是在“设置”-》应用管理-》正在运行-》停止服务。（这个是正常停止服务，会触发onDestory，所以上面的onStartC

**7、服务互相绑定**

同时开启多个服务，互相监测，如果一方被干掉，则重启。
双服务没有native守护进程来的好，虽然360，微信什么的都有几个进程服务，但如果不添加到后台保活的话，效果一样不能保活，也会进入停止状态。

**8、设置闹钟，定时唤醒**

**9、自己的app在native层fork一个子进程来与主进程互拉**
Android通过JNI的方式，在 C端 fork()出一组子进程作为守护进程，监听进程事件（kill），或者检测Service是否存活，若Service已被杀死，则adb shell进行重启Service。  至于检测方式，可以轮询获取子进程Pid，若为1， 则说明子进程被Init进程所领养，已经成为了孤儿进程。    但是这种方式比较消耗电量，并且由于不同手机系统定制的改变，当应用被强制停止时，父进程并不一定被真正杀死，因此在一些特定机型上是无法通过此方式进行判断。 这里推荐使用liunx socket的方式进行类似心跳包的检测，并且当触发检测Service是否被杀死之前，需要判断应用是否已经被卸载，如果应用已经被卸载，则不再进行检测Service行为，直接调用exit(0)退出子进程，避免浪费系统资源和消耗电量。

注意: 目前在Android 5。0系统上会把fork出来的进程放到一个进程组里， 当程序主进程挂掉后，也会把整个进程组杀掉，因此用fork的方式也无法在Android5。0及以上系统实现守护进程。 这个是系统层面的限制，当然也是为了优化整个的系统环境，守护进程给手机带来的体验并不好。

Is All~


