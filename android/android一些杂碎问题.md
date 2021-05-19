#### jar和aar
  前者只有代码，后者既有代码也有资源
  
#### Android每个应用分配多少内存
和具体机型有关

#### 更新UI的几种方式
 - Activity.runOnUiThread(Runnable)
 - View.post(Runnable)，**view.post() 的内部也是调用了 Handler**
 - Handler
 - AsyncTask
 - Rxjava
 - LiveData

#### Thread、AsyncTask、IntentService使用场景及特点
 - Thread是独立于Activity的，即使Activity被finish掉了，也会一致运行
 - AsyncTash封装了两个线程池和一个Handler(SerialExecutor用于排队，Thread Pool executor执行任务，Handler将工作线程切换到主线程)，必须在UI中创建，execute方法必须在UI线程工执行，一个实例只能执行一次，可用于简单数据处理
 - IntentService，可处理多线程操作，在onHandleIntent中处理耗时操作，多个任务依次执行，执行完自动结束

#### Merge和ViewStub标签
 - merge可以减少视图层级，是结构更扁平
 - ViewStub，按需加载，减少内存使用，加快渲染速度，不支持merge标签
    当我们需要根据某个条件控制某个View的显示或者隐藏的时候，通常是把可能用到的View都写在布局上，然后设置可见性为View.GONE或View.InVisible ，之后在代码中根据条件动态控制可见性。虽然操作简单，但是耗费资源，因为即便该view不可见，仍会被父窗体绘制，仍会创建对象，仍会被实例化，仍会被设置属性。
    而android.view.ViewStub，是一个大小为0 ，默认不可见的控件，只有给他设置成了View.Visible或调用了它的inflate()之后才会填充布局资源，也就是说占用资源少。所以，推荐使用viewStub
	
#### Activity的startActivity和Context的startActivity区别
从非 Activity （例如从其他Context中）启动Activity则必须给intent设置**Flag:FLAG\_ACTIVITY\_NEW\_TASK**
 因为非 Activity环境启动Activity 的时候没有 Activity 栈，所以需要加上这个标志位新建一个栈，例如ContextImpl.java中的实现，会检查有没有设置Flag:FLAG\_ACTIVITY\_NEW\_TASK，否则会报错：
 
 #### 怎么在Service中创建Dialog
 ```
dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT)
```
```
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINOW" />
```
 如果只在Service中写入常在Activity中使用的创建Alter的代码，运行时是会发生错误的，因为Alter的显示需要依附于一个确定的Activity类。而以上做法就是声明我们要弹出的这个提示框是一个系统的提示框，即全局性质的提示框，所以只要手机处于开机状态，无论它现在处于何种界面之下，只要调用alter.show()，就会弹出提示框来
 
 #### Asset和res目录区别
 - res中有R文件，res下面所有资源id会放到R文件中，资源在打包时，如果被用到就会打包到apk，否则不会
 - assets，存在这里的资源会打包到apk，assets下的文件通过文件读写访问
 - res/anim，res/raw同assets下的资源一样打包到apk，同时也在R中有引用id

#### Android怎么加速启动Activity
-   onCreate() 中不执行耗时操作 把页面显示的 View 细分一下，放在 AsyncTask 里逐步显示，用 Handler 更好。这样用户的看到的就是有层次有步骤的一个个的 View 的展示，不会是先看到一个黑屏，然后一下显示所有 View。最好做成动画，效果更自然。
-   利用多线程的目的就是尽可能的减少 onCreate() 和 onReume() 的时间，使得用户能尽快看到页面，操作页面。
-   减少主线程阻塞时间。
-   提高 Adapter 和 AdapterView 的效率。
-   优化布局文件。

#### 程序A能否接收到程序B的广播
能，使用全局的BroadCastRecevier能进行跨进程通信，但是注意它只能被动接收广播。此外，LocalBroadCastRecevier只限于本进程的广播间通信。

#### 数据加载更多涉及到分页，你是怎么实现的？
分页加载就是一页一页加载数据，当滑动到底部、没有更多数据加载的时候，我们可以手动调用接口，重新刷新RecyclerView。

#### 内存泄露，怎样查找，怎么产生的内存泄露？


#### 类的初始化顺序依次是？
（静态变量、静态代码块）>（变量、代码块）>构造方法

#### Serializable和Parcelable区别
1. 两者最大的区别在于 **存储媒介的不同**，`Serializable` 使用 **I/O 读写存储在硬盘上**，而 `Parcelable` 是直接 **在内存中读写**。很明显，内存的读写速度通常大于 IO 读写，所以在 Android 中传递数据优先选择 `Parcelable`
2. `Serializable` 会使用反射，序列化和反序列化过程需要大量 I/O 操作， `Parcelable` 自已实现封送和解封（marshalled &unmarshalled）操作不需要用反射，数据也存放在 共享内存中，效率要快很多

#### Parcelable插件
android parcelable code generator

#### Bitmap使用注意事项
1. 采用合适的图片规格，不同规格每个像素占用字节不同
2. 降低采样率
3. 复用内存
4. 使用recycle()及时回收内存
5. 压缩图片

#### 多进程应用场景
 1. 在新进程中，启动前台Service，播放音乐(为了提高进程优先级，达到进程保活目的)
 2. 利用多进程解决OOM问题(因为android内存限制是针对进程的) 


#### bitmap recycler 相关
在Android中，Bitmap的存储分为两部分，一部分是Bitmap的数据，一部分是Bitmap的引用。 在Android2.3时代，Bitmap的引用是放在栈中的，而Bitmap的数据部分是放在堆中的，需要用户调用recycle方法手动进行内存回收，而在Android2.3之后，整个Bitmap，包括数据和引用，都放在了堆中，这样，整个Bitmap的回收就全部交给GC了，这个recycle方法就再也不需要使用了

#### Intent或广播传输大小限制
Intent在传递数据时是有大小限制的，大约限制在1MB之内，你用Intent传递数据，实际上走的是跨进程通信（IPC），跨进程通信需要把数据从内核copy到进程中，每一个进程有一个接收内核数据的缓冲区，默认是1M；如果一次传递的数据超过限制，就会出现异常。

不同厂商表现不一样有可能是厂商修改了此限制的大小，也可能同样的对象在不同的机器上大小不一样。

传递大数据，不应该用Intent；考虑使用ContentProvider或者直接匿名共享内存。简单情况下可以考虑分段传输

### 硬件加速
https://www.mtyun.com/library/hardware-accelerate
硬件加速的主要原理，就是通过底层软件代码，将CPU不擅长的图形计算转换成GPU专用指令，由GPU完成
1. CPU的控制器较为复杂，而ALU数量较少。因此CPU擅长各种复杂的逻辑运算，但不擅长数学尤其是浮点运算，和CPU不同的是，GPU就是为实现大量数学运算设计的
2. 页面由各种基础元素（DisplayList）构成，渲染时需要进行大量浮点运算
3. 硬件加速条件下，CPU用于控制复杂绘制逻辑、构建或更新DisplayList；GPU用于完成图形计算、渲染DisplayList
4. 实现同样效果，应尽量使用更简单的DisplayList，从而达到更好的性能（Shape代替Bitmap等）

https://juejin.cn/post/6844903893189525512
在 Android 里，硬件加速专指把View中绘制的计算工作交给 GPU来处理。进一步地明确一下，这个绘制的计算工作指的就是把绘制方法中的那些 Canvas.drawXXX() 变成实际的像素

5. 4.0后硬件加速默认是开启的
6. 可以在AndroidManifest.xml中开启或关闭硬件加速，如下application：
```java
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:hardwareAccelerated="true"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
</application>
```

#### ContentProvider的权限管理(读写分离，权限控制-精确到表级，URL控制)。

对于ContentProvider而言，有很多权限控制，可以在AndroidManifest.xml文件中对节点的属性进行配置，一般使用如下一些属性设置：

-   android:grantUriPermssions:临时许可标志。
-   android:permission:Provider读写权限。
-   android:readPermission:Provider的读权限。
-   android:writePermission:Provider的写权限。
-   android:enabled:标记允许系统启动Provider。
-   android:exported:标记允许其他应用程序使用这个Provider。
-   android:multiProcess:标记允许系统启动Provider相同的进程中调用客户端
  
  #### Fragment状态保存

Fragment状态保存入口:

1、Activity的状态保存, 在Activity的onSaveInstanceState()里, 调用了FragmentManger的saveAllState()方法, 其中会对mActive中各个Fragment的实例状态和View状态分别进行保存.

2、FragmentManager还提供了public方法: saveFragmentInstanceState(), 可以对单个Fragment进行状态保存, 这是提供给我们用的。

3、FragmentManager的moveToState()方法中, 当状态回退到ACTIVITY\_CREATED, 会调用saveFragmentViewState()方法, 保存View的状态.

#### 直接在Activity中创建一个thread跟在service中创建一个thread之间的区别？

在Activity中被创建：该Thread的就是为这个Activity服务的，完成这个特定的Activity交代的任务，主动通知该Activity一些消息和事件，Activity销毁后，该Thread也没有存活的意义了。

在Service中被创建：所以，在Service中创建的Thread，适合长期执行一些独立于APP的后台任务，一般在Service的onCreate()中创建，在onDestroy()中销毁，比较常见的就是：在Service中保持与服务器端的长连接

#### 如何计算一个Bitmap占用内存的大小，怎么保证加载Bitmap不产生内存溢出？
```
Bitamp 占用内存大小 = 宽度像素 x （inTargetDensity / inDensity） x 高度像素 x （inTargetDensity / inDensity）x 一个像素所占的内存
```
1. 这里inDensity表示图片的dpi（放在哪个资源文件夹下），inTargetDensity表示屏幕的dpi，所以你可以发现inDensity和inTargetDensity会对Bitmap的宽高进行拉伸，进而改变Bitmap占用内存的大小

2. 为了保证在加载Bitmap的时候不产生内存溢出，可以使用BitmapFactory进行图片压缩

#### apk为啥要签名
1. 防止相同package name的应用冒名顶替
2. 防止apk被篡改

#### 为什么bindService可以跟Activity生命周期联动？
1、bindService 方法执行时，LoadedApk 会记录 ServiceConnection 信息。
2、Activity 执行 finish 方法时，会通过 LoadedApk 检查 Activity 是否存在未注销/解绑的 BroadcastReceiver 和 ServiceConnection，如果有，那么会通知 AMS 注销/解绑对应的 BroadcastReceiver 和 Service，并打印异常信息，告诉用户应该主动执行注销/解绑的操作

注：**LoadedApk对象是APK文件在内存中的表示**。 Apk文件的相关信息，诸如Apk文件的代码和资源，甚至代码里面的Activity，Service等组件的信息我们都可以通过此对象获取

#### 如何通过Gradle配置多渠道包？
在productFlavors中配置不同渠道

```
android {  
    productFlavors {
        xiaomi {}
        baidu {}
        wandoujia {}
        _360 {}        // 或“"360"{}”，数字需下划线开头或加上双引号
    }
}
```

执行./gradlew assembleRelease ，将会打出所有渠道的release包；

执行./gradlew assembleWandoujia，将会打出豌豆荚渠道的release和debug版的包；

执行./gradlew assembleWandoujiaRelease将生成豌豆荚的release包。

因此，可以结合buildType和productFlavor生成不同的Build Variants，即类型与渠道不同的组合

#### activty和Fragmengt之间怎么通信，Fragmengt和Fragmengt怎么通信？
1. Handler
2. 广播
3. 事件总线：EventBus、RxBus、Otto
4. 接口回调
5. Bundle和setArguments(bundle)

#### 自定义view效率高于xml定义吗？说明理由
1、少了解析xml。
2.、自定义View 减少了ViewGroup与View之间的测量,包括父量子,子量自身,子在父中位置摆放,当子view变化时,父的某些属性都会跟着变化


#### 广播注册一般有几种，各有什么优缺点？
第一种是静态注册：当应用程序关闭后如果有信息广播来，程序也会被系统调用，自己运行；第二种不常驻动态注册：广播会跟随程序的生命周期。

1. 动态注册
优点： 在android的广播机制中，动态注册优先级高于静态注册优先级，因此在必要情况下，是需要动态注册广播接收者的。
缺点： 当用来注册的 Activity 关掉后，广播也就失效了。

2. 静态注册
优点： 无需担忧广播接收器是否被关闭，只要设备是开启状态，广播接收器就是打开着的

#### 服务启动一般有几种，服务和activty之间怎么通信，服务和服务之间怎么通信
1、startService：
onCreate()--->onStartCommand() ---> onDestory()

如果服务已经开启，不会重复的执行onCreate()， 而是会调用onStartCommand()。一旦服务开启跟调用者(开启者)就没有任何关系了。 开启者退出了，开启者挂了，服务还在后台长期的运行。 开启者不能调用服务里面的方法。

2、bindService：
onCreate() --->onBind()--->onunbind()--->onDestory()

bind的方式开启服务，绑定服务，调用者挂了，服务也会跟着挂掉。 绑定者可以调用服务里面的方法。

通信：
1、通过Binder对象。
2、通过broadcast(广播)。


#### ddms 和 traceView 的区别？
ddms 原意是：davik debug monitor service。简单的说 ddms 是一个程序执行查看器，在里面可以看见线程和堆栈等信息，traceView 是程序性能分析器。traceview 是 ddms 中的一部分内容。

Traceview 是 Android 平台特有的数据采集和分析工具，它主要用于分析 Android 中应用程序的 hotspot（瓶颈）。Traceview 本身只是一个数据分析工具，而数据的采集则需要使用 Android SDK 中的 Debug 类或者利用DDMS 工具。二者的用法如下：开发者在一些关键代码段开始前调用 Android SDK 中 Debug 类的 startMethodTracing 函数，并在关键代码段结束前调用 stopMethodTracing 函数。这两个函数运行过程中将采集运行时间内该应用所有线程（注意，只能是 Java线程） 的函数执行情况， 并将采集数据保存到/mnt/sdcard/下的一个文件中。 开发者然后需要利用 SDK 中的 Traceview工具来分析这些数据


#### ListView卡顿原因
1. 单元格复用：Adapter的getView方法里面convertView没有使用setTag和getTag方式

2. getView里面有耗时操作：在getView方法里面ViewHolder初始化后的赋值或者是多个控件的显示状态和背景的显示没有优化好，或是里面含有复杂的计算和耗时操作；

3. 布局文件复杂，嵌套深：在getView方法里面 inflate的row 嵌套太深（布局过于复杂）或者是布局里面有大图片或者背景所致；

4. 频繁刷新数据和view：Adapter多余或者不合理的notifySetDataChanged；

#### AndroidManifest的作用与理解
AndroidManifest.xml文件，也叫清单文件，来获知应用中是否包含该组件，如果有会直接启动该组件。可以理解是一个应用的配置文件。

作用：
-   为应用的 Java 软件包命名。软件包名称充当应用的唯一标识符。
-   描述应用的各个组件，包括构成应用的 Activity、服务、广播接收器和内容提供程序。它还为实现每个组件的类命名并发布其功能，例如它们可以处理的 Intent - 消息。这些声明向 Android 系统告知有关组件以及可以启动这些组件的条件的信息。
-   确定托管应用组件的进程。
-   声明应用必须具备哪些权限才能访问 API 中受保护的部分并与其他应用交互。还声明其他应用与该应用组件交互所需具备的权限
-   列出 Instrumentation类，这些类可在应用运行时提供分析和其他信息。这些声明只会在应用处于开发阶段时出现在清单中，在应用发布之前将移除。
-   声明应用所需的最低 Android API 级别
-   列出应用必须链接到的库

#### singleInstance应用场景
闹铃的响铃界面。 你以前设置了一个闹铃：上午6点。在上午5点58分，你启动了闹铃设置界面，并按 Home 键回桌面；在上午5点59分时，你在微信和朋友聊天；在6点时，闹铃响了，并且弹出了一个对话框形式的 Activity(名为 AlarmAlertActivity) 提示你到6点了(这个 Activity 就是以 SingleInstance 加载模式打开的)，你按返回键，回到的是微信的聊天界面，这是因为 AlarmAlertActivity 所在的 Task 的栈只有他一个元素， 因此退出之后这个 Task 的栈空了。如果是以 SingleTask 打开 AlarmAlertActivity，那么当闹铃响了的时候，按返回键应该进入闹铃设置界面

#### 说说Activity、Intent、Service 是什么关系
他们都是 Android 开发中使用频率最高的类。其中 Activity 和 Service 都是 Android 四大组件之一。他俩都是 Context 类的子类 ContextWrapper 的子类，因此他俩可以算是兄弟关系吧。不过兄弟俩各有各自的本领，Activity 负责用户界面的显示和交互，Service 负责后台任务的处理。Activity 和 Service 之间可以通过 Intent 传递数据，因此 可以把 Intent 看作是通信使者


#### ApplicationContext和ActivityContext的区别
这是两种不同的context，也是最常见的两种.第一种中context的生命周期与Application的生命周期相关的，context随着Application的销毁而销毁，伴随application的一生，与activity的生命周期无关.第二种中的context跟Activity的生命周期是相关的，但是对一个Application来说，Activity可以销毁几次，那么属于Activity的context就会销毁多次.至于用哪种context，得看应用场景。还有就是，在使用context的时候，小心内存泄露，防止内存泄露，注意一下几个方面：

-   不要让生命周期长的对象引用activity context，即保证引用activity的对象要与activity本身生命周期是一样的。
-   对于生命周期长的对象，可以使用application context。
-   避免非静态的内部类，尽量使用静态类，避免生命周期问题，注意内部类对外部对象引用导致的生命周期变化。


#### Handler、Thread和HandlerThread的差别
1、Handler：在android中负责发送和处理消息，通过它可以实现其他支线线程与主线程之间的消息通讯。

2、Thread：Java进程中执行运算的最小单位，亦即执行处理机调度的基本单位。某一进程中一路单独运行的程序。

3、HandlerThread：一个继承自Thread的类HandlerThread，Android中没有对Java中的Thread进行任何封装，而是提供了一个继承自Thread的类HandlerThread类，这个类对Java的Thread做了很多便利的封装。HandlerThread继承于Thread，所以它本质就是个Thread。与普通Thread的差别就在于，它在内部直接实现了Looper的实现，这是Handler消息机制必不可少的。有了自己的looper，可以让我们在自己的线程中分发和处理消息。如果不用HandlerThread的话，需要手动去调用Looper.prepare()和Looper.loop()这些方法

#### ThreadLocal的原理
ThreadLocal是一个关于创建线程局部变量的类。使用场景如下所示：

-   实现单个线程单例以及单个线程上下文信息存储，比如交易id等。
    
-   实现线程安全，非线程安全的对象使用ThreadLocal之后就会变得线程安全，因为每个线程都会有一个对应的实例。 承载一些线程相关的数据，避免在方法中来回传递参数。
    
当需要使用多线程时，有个变量恰巧不需要共享，此时就不必使用synchronized这么麻烦的关键字来锁住，每个线程都相当于在堆内存中开辟一个空间，线程中带有对共享变量的缓冲区，通过缓冲区将堆内存中的共享变量进行读取和操作，ThreadLocal相当于线程内的内存，一个局部变量。每次可以对线程自身的数据读取和操作，并不需要通过缓冲区与 主内存中的变量进行交互。并不会像synchronized那样修改主内存的数据，再将主内存的数据复制到线程内的工作内存。ThreadLocal可以让线程独占资源，存储于线程内部，避免线程堵塞造成CPU吞吐下降。

在每个Thread中包含一个ThreadLocalMap，ThreadLocalMap的key是ThreadLocal的对象，value是独享数据

#### 计算一个view的嵌套层级
```
private int getParents(ViewParents view){
    if(view.getParents() == null) 
        return 0;
    } else {
    return (1 + getParents(view.getParents));
   }
}
```

#### MVP，MVVM，MVC解释和实践
##### MVC:
-   视图层(View) 对应于xml布局文件和java代码动态view部分
-   控制层(Controller) MVC中Android的控制层是由Activity来承担的，Activity本来主要是作为初始化页面，展示数据的操作，但是因为XML视图功能太弱，所以Activity既要负责视图的显示又要加入控制逻辑，承担的功能过多。
-   模型层(Model) 针对业务模型，建立数据结构和相关的类，它主要负责网络请求，数据库处理，I/O的操作。

##### 总结
具有一定的分层，model彻底解耦，controller和view并没有解耦

##### MVP
通过引入接口BaseView，让相应的视图组件如Activity，Fragment去实现BaseView，实现了视图层的独立，通过中间层Preseter实现了Model和View的完全解耦。MVP彻底解决了MVC中View和Controller傻傻分不清楚的问题，但是随着业务逻辑的增加，一个页面可能会非常复杂，UI的改变是非常多，会有非常多的case，这样就会造成View的接口会很庞大

##### MVVM
MVP中我们说过随着业务逻辑的增加，UI的改变多的情况下，会有非常多的跟UI相关的case，这样就会造成View的接口会很庞大。而MVVM就解决了这个问题，通过双向绑定的机制，实现数据和UI内容，只要想改其中一方，另一方都能够及时更新的一种设计理念，这样就省去了很多在View层中写很多case的情况，只需要改变数据就行

##### MVVM与DataBinding的关系？
MVVM是一种思想，DataBinding是谷歌推出的方便实现MVVM的工具。

看起来MVVM很好的解决了MVC和MVP的不足，但是由于数据和视图的双向绑定，导致出现问题时不太好定位来源，有可能数据问题导致，也有可能业务逻辑中对视图属性的修改导致。如果项目中打算用MVVM的话可以考虑使用官方的架构组件ViewModel、LiveData、DataBinding去实现MVVM


#### SharedPrefrences的apply和commit有什么区别？
这两个方法的区别在于：

1.  apply没有返回值而commit返回boolean表明修改是否提交成功。
    
2.  apply是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘, 而commit是同步的提交到硬件磁盘，因此，在多个并发的提交commit的时候，他们会等待正在处理的commit保存到磁盘后在操作，从而降低了效率。
    
3.  apply方法不会提示任何失败的提示。 由于在一个进程中，sharedPreference是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的

#### Base64、MD5是加密方法么
1. Base64是用文本表示二进制的编码方式，它使用4个字节的文本来表示3个字节的原始二进制数据。 它将二进制数据转换成一个由64个可打印的字符组成的序列：A-Za-z0-9+/
2. MD5是哈希算法的一种，可以将任意数据产生出一个128位（16字节）的散列值，用于确保信息传输完整一致。我们常在注册登录模块使用MD5，用户密码可以使用MD5加密的方式进行存储。

加密，指的是对数据进行转换以后，数据变成了另一种格式，并且除了拿到解密方法的人，没人能把数据转换回来。 MD5是一种信息摘要算法，它是不可逆的，不可以解密。所以它只能算的上是一种单向加密算法。 Base64也不是加密算法，它是一种数据编码方式，虽然是可逆的，但是它的编码方式是公开的，无所谓加密

#### HttpClient和HttpConnection的区别
Http Client适用于web浏览器，拥有大量灵活的API，实现起来比较稳定，且其功能比较丰富，提供了很多工具，封装了http的请求头，参数，内容体，响应，还有一些高级功能，代理、COOKIE、鉴权、压缩、连接池的处理。 　　但是，正因此，在不破坏兼容性的前提下，其庞大的API也使人难以改进，因此Android团队对于修改优化Apache Http Client并不积极。(并在Android 6.0中抛弃了Http Client，替换成OkHttp)

在Android 2.2版本之前，HttpClient拥有较少的bug，因此使用它是最好的选择。 而在Android 2.3版本及以后，HttpURLConnection则是最佳的选择。它的API简单，体积较小，因而非常适用于Android项目。

#### ActivityA跳转ActivityB然后B按back返回A，各自的生命周期顺序，A与B均不透明。

##### ActivityA跳转到ActivityB：
```
Activity A：onPause
Activity B：onCreate
Activity B：onStart
Activity B：onResume
Activity A：onStop
```

##### ActivityB返回ActivityA：
```
Activity B：onPause
Activity A：onRestart
Activity A：onStart
Activity A：onResume
Activity B：onStop
Activity B：onDestroy
```


#### 如何通过广播拦截和abort一条短信？
可以监听这条信号，在传递给真正的接收程序时，我们将自定义的广播接收程序的优先级大于它，并且取消广播的传播，这样就可以实现拦截短信的功能了


#### BroadcastReceiver，LocalBroadcastReceiver 区别？
##### 1、应用场景
   1、BroadcastReceiver用于应用之间的传递消息；

   2、而LocalBroadcastReceiver用于应用内部传递消息，比broadcastReceiver更加高效。

##### 2、安全
   1、BroadcastReceiver使用的Content API，所以本质上它是跨应用的，所以在使用它时必须要考虑到不要被别的应用滥用；

   2、LocalBroadcastManager不需要考虑安全问题，因为它只在应用内部有效
   
##### 原理方面
(1) 与BroadcastReceiver是以 Binder 通讯方式为底层实现的机制不同，LocalBroadcastReceiver 的核心实现实际还是 Handler，只是利用到了 IntentFilter 的 match 功能，至于 BroadcastReceiver 换成其他接口也无所谓，顺便利用了现成的类和概念而已。

(2) LocalBroadcastReceiver因为是 Handler 实现的应用内的通信，自然安全性更好，效率更高


#### 如何选择第三方，从那些方面考虑？

##### 大方向：从软件环境做判断
- 性能是开源软件第一解决的问题。
- 一个好的生态，是一个优秀的开源库必备的，取决标准就是观察它是否一直在持续更新迭代，是否能及时处理github上用户提出来的问题。大家在社区针对这个开源库是否有比较活跃的探讨。
- 背书，该开源库由谁推出，由哪个公司推出来的。
- 用户数和有哪些知名的企业落地使用

##### 小方向：从软件开发者的角度做判断
- 是否解决了我们现有问题或长期来看带来的维护成本。
- 公司有多少人会。
- 学习成本

#### 单例实现线程的同步的要求
1.单例类确保自己只有一个实例(构造函数私有:不被外部实例化,也不被继承)。

2.单例类必须自己创建自己的实例。

3.单例类必须为其他对象提供唯一的实例


#### 如何保证Service不被杀死？
Android 进程不死从3个层面入手：

A.提供进程优先级，降低进程被杀死的概率

方法一：监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的 Activity，在用户解锁时将 Activity 销毁掉。

方法二：启动前台service。

方法三：提升service优先级：

在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播。

B. 在进程被杀死后，进行拉活

方法一：注册高频率广播接收器，唤起进程。如网络变化，解锁屏幕，开机等

方法二：双进程相互唤起。

方法三：依靠系统唤起。

方法四：onDestroy方法里重启service：service + broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service；

C. 依靠第三方

根据终端不同，在小米手机（包括 MIUI）接入小米推送、华为手机接入华为推送；其他手机可以考虑接入腾讯信鸽或极光推送与小米推送做 A/B Test


#### 如何导入外部数据库?
1. 将数据库文件放到在项目源码的 res/raw目录下
2. android系统下数据库应该存放在 /data/data/com.（package name）/ 目录下，所以我们需要做的是把已有的数据库传入那个目录下。操作方法是用FileInputStream读取原数据库，再用FileOutputStream把读取到的东西写入到那个目录

#### LinearLayout、FrameLayout、RelativeLayout性能对比，为什么？
RelativeLayout会让子View调用2次onMeasure，LinearLayout 在有weight时，也会调用子 View 2次onMeasure

RelativeLayout的子View如果高度和RelativeLayout不同，则会引发效率问题，当子View很复杂时，这个问题会更加严重。如果可以，尽量使用padding代替margin。

在不影响层级深度的情况下,使用LinearLayout和FrameLayout而不是RelativeLayout


#### scheme跳转协议
Android中的scheme是一种页面内跳转协议，通过定义自己的scheme协议，可以跳转到app中的各个页面

服务器可以定制化告诉app跳转哪个页面

App可以通过跳转到另一个App页面

可以通过H5页面跳转页面


#### 如何将一个Activity设置成窗口的样式。
中配置：

```
android:theme="@android:style/Theme.Dialog"
```

另外

```
android:theme="@android:style/Theme.Translucnt"
```

#### Android中跨进程通讯的几种方式
1：访问其他应用程序的Activity 如调用系统通话应用

```
Intent callIntent = new Intent(Intent.ACTION_CALL,Uri.parse("tel:12345678");
startActivity(callIntent);
```

2：Content Provider 如访问系统相册

3：广播（Broadcast） 如显示系统时间

4：AIDL服务


#### 显示Intent与隐式Intent的区别
对明确指出了目标组件名称的Intent，我们称之为“显式Intent”。
对于没有明确指出目标组件名称的Intent，则称之为“隐式 Intent”


#### 如何让程序自动启动？
定义一个Braodcastreceiver，action为BOOT COMPLETE，接受到广播后启动程序


#### 断点续传实现

##### 基础知识：
-   Http基础：在Http请求中，可以加入请求头Range，下载指定区间的文件数。
-   RandomAccessFile：支持文件随机位置访问，可以从指定位置对文件进行读写。

有了这个基础以后，思路就清晰了：
-   通过HttpUrlConnection获取文件长度。
-   自己分配好线程进行制定区间的文件数据的下载。
-   获取到数据流以后，使用RandomAccessFile进行指定位置的读写


#### SparseArray和ArrayMap代替HashMap
https://blog.csdn.net/u010687392/article/details/47809295
1. SparseArray比HashMap更省内存，在某些条件下性能更好，主要是因为它避免了对key的自动装箱（int转为Integer类型），它内部则是通过两个数组来进行数据存储的，一个存储key，另外一个存储value，为了优化性能，它内部对数据还采取了压缩的方式来表示稀疏数组的数据，从而节约内存空间

2. ArrayMap是一个<key,value>映射的数据结构，它设计上更多的是考虑内存的优化，内部是使用两个数组进行数据存储，一个数组记录key的hash值，另外一个数组记录Value值，它和SparseArray一样，也会对key使用二分法进行从小到大排序，在添加、删除、查找数据的时候都是先使用二分查找法得到相应的index，然后通过index来进行添加、查找、删除等操作，所以，应用场景和SparseArray的一样，如果在数据量比较大的情况下，那么它的性能将退化至少50%

SparseArray和ArrayMap如何选择：
1、如果key的类型已经确定为int类型，那么使用SparseArray，因为它避免了自动装箱的过程，如果key为long类型，它还提供了一个LongSparseArray来确保key为long类型时的使用
2、如果key类型为其它的类型，则使用ArrayMap

#### 关于< include >< merge >< viewstub >三者的使用场景
- include使用场景：
include标签常用于将布局中的公共部分提取出来供其他layout共用，以实现布局模块化

- merge标签使用场景：  
1. 布局顶结点是FrameLayout且不需要设置background或padding等属性，可以用merge代替，因为Activity内容试图的parent view就是个FrameLayout，所以可以用merge消除只剩一个。  
2. 某布局作为子布局被其他布局include时，使用merge当作该布局的顶节点，这样在被引入时顶结点会自动被忽略，而将其子节点全部合并到主布局中

- viewstub使用场景：
viewstub常用来引入那些默认不会显示，只在特殊情况下显示的布局，如进度布局、网络失败显示的刷新布局、信息出错出现的提示布局等

#### 系统如何知道当前 WiFi 有问题？
如果一个 WiFi 发送过数据包，但是没有收到任何的 ACK 回包，这个时候就可以初步判断当前的 WiFi 是有问题的

#### 谈谈Android的安全机制
-   1.  Android 是基于Linux内核的，因此 Linux 对文件权限的控制同样适用于 Android。在 Android 中每个应用都有自己的/data/data/包名 文件夹，该文件夹只能该应用访问，而其他应用则无权访问。
-   2.  Android 的权限机制保护了用户的合法权益。如果我们的代码想拨打电话、发送短信、访问通信录、定位、访问、sdcard 等所有可能侵犯用于权益的行为都是必须要在 AndroidManifest.xml 中进行声明的，这样就给了用户一个知情权。
-   3.  Android 的代码混淆保护了开发者的劳动成果


#### 如何开启多进程？应用是否可以开启N个进程？
在AndroidManifest中给四大组件指定属性android:process开启多进程模式，在内存允许的条件下可以开启N个进程

#### 为何需要IPC？多进程通信可能会出现的问题？
所有运行在不同进程的四大组件（Activity、Service、Receiver、ContentProvider）共享数据都会失败，这是由于Android为每个应用分配了独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，这会导致在不同的虚拟机中访问同一个类的对象会产生多份副本。比如常用例子（通过开启多进程获取更大内存空间、两个或者多个应用之间共享数据、微信全家桶）。

一般来说，使用多进程通信会造成如下几方面的问题:

-   静态成员和单例模式完全失效：独立的虚拟机造成。
-   线程同步机制完全失效：独立的虚拟机造成。
-   SharedPreferences的可靠性下降：这是因为Sp不支持两个进程并发进行读写，有一定几率导致数据丢失。
-   Application会多次创建：Android系统在创建新的进程时会分配独立的虚拟机，所以这个过程其实就是启动一个应用的过程，自然也会创建新的Application


#### JVM、Dalvik和ART区别