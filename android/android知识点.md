#### App启动机制
![[Pasted image 20210518104205.png]]
![[Pasted image 20210518104257.png]]
1. 系统启动，创建出SystemServer
系统启动bootloader启动Linux内核和init进程之后，会由init进程fork出Zygote进程，所有的Android系统服务进程以及应用程序进程都是由这个Zygote进程fork出来的(通过socket来监听创建请求)，其中包括SystemServer进程(Zygote在类main方法中调用startSystemServer方法)

 2. 在SystemServer调用createSystemContext
   做了三件事
   - 第一，创建ActivityThread即主线程(UI线程)，ActivityThread会创建一个mainLooper开启消息循环，这也是在主线程中使用Handler不手动创景Looper的原因
   - 第二，获取系统上下文
   - 设置了默认主题

 3. SystemServer启动系统服务
在SystemServer的main方法中会启动系统服务，包括ActivityMS、PackageMS、WindowMS、PowerMS

 4. 启动Launcher(Home进程中)
 ![[Pasted image 20210518110153.png]]
   PackageMS启动后会将系统中的应用安装完成，AMS会将Launcher启动起来(如上图systemReady即为Launcher入口)，Launcher会显示已安装应用的快捷图标
   
  5. 点击应用图标，启动App
   - Launcher接收到Click Event，获取应用信息之后，会通过Binder IPC机制向ActivityManagerService（简称AMS）发起启动应用（startActivity(intent)）的请求。此时AMS会做以下三个内部操作：
   
        1.通过PackageManager的resolveIntent方法收集这个intent对象的指向信息，并将指向信息存储在一个intent对象中；
		
        2.通过grantUriPermissionLocked方法来验证用户是否有足够的权限去调用该intent对象指向的Activity
		
        3.如果有权限，AMS会检查并在新的task中启动目标Activity
  - 待AMS做完上述三个内部操作外，它会检查这个进程的ProcessRecord是否存在。如果ProcessRecord是null，启动进程会调用AMS的startProcessLocked方法，内部调用Process的start方法，通过socket通道传递参数给Zygote进程
  - 之后Zygote进程会通过孵化自身的方式去fork一个新的Linux进程（此处特指UI进程），并调用ZygoteInit.main()方法来实例化ActivityThread对象并最终返回新进程的pid，同时创建ApplicationThread，Looper，Hander对象，调用attach方法进行Binder通信。随后依次调用Looper.prepareLoop()和Looper.loop()来开启消息循环
  - 接下来要做的就是将进程和指定的Application绑定起来。 这个是通过上一步的ActivityThread对象中调用bindApplication()方法完成的，该方法发送一个BIND_APPLICATION的消息到消息队列中，最终通过handleBindApplication()方法处理该消息，然后调用makeApplication()方法来加载App的classes到内存中
  -  后面的调用顺序就是普通的从一个已经存在的进程中启动一个新进程的activity了。实际调用方法是realStartActivity()，它会调用application线程对象中的sheduleLaunchActivity()方法发送一个LAUNCH_ACTIVITY消息到消息队列中，通过 handleLaunchActivity()来处理该消息
 
 #### 详解AMS
 

##### 四大组件之Activity
一，Activity生命周期
![[Pasted image 20210517232457.png]]
1.启动Activity：系统会先调用onCreate方法，然后调用onStart方法，最后调用onResume，Activity进入运行状态。

2.当前Activity被其他Activity覆盖其上或被锁屏：系统会调用onPause方法，暂停当前Activity的执行。

3.当前Activity由被覆盖状态回到前台或解锁屏：系统会调用onResume方法，再次进入运行状态。

4.当前Activity转到新的Activity界面或按Home键回到主屏，自身退居后台：系统会先调用onPause方法，然后调用onStop方法，进入停滞状态。

5.用户后退回到此Activity：系统会先调用onRestart方法，然后调用onStart方法，最后调用onResume方法，再次进入运行状态。

6.当前Activity处于被覆盖状态或者后台不可见状态，即第2步和第4步，系统内存不足，杀死当前Activity，而后用户退回当前Activity：再次调用onCreate方法、onStart方法、onResume方法，进入运行状态。

7.用户退出当前Activity：系统先调用onPause方法，然后调用onStop方法，最后调用onDestory方法，结束当前Activity

二，Activity横竖屏生命周期
-   不设置 Activity 的 `android:configChanges` 时，切屏会销毁当前Activity，然后重新加载调用各个生命周期，切横屏时会执行一 次，切竖屏时会执行两次；`onPause()`→`onStop()`→`onDestory()`→`onCreate()`→`onStart()`→`onResume()`
-   设置 Activity 的 `android:configChanges="orientation"`时，切换横竖屏都只执行一次
-   设置Activity的android:configChanges=”orientation|keyboardHidden”时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法

三，onSaveInstanceState和onRestoreInstanceState
https://www.heqiangfly.com/2018/08/12/android-knowledge-about-activity-save-state/
Activity的 onSaveInstanceState() 和 onRestoreInstanceState()并不是生命周期方法，它们不同于 onCreate()、onPause()等生命周期方法，它们并不一定会被触发。当应用遇到意外情况（如：内存不足、用户直接按Home键）由系统销毁一个Activity时，onSaveInstanceState() 会被调用。但是当用户主动去销毁一个Activity时，例如在应用中按返回键，onSaveInstanceState()就不会被调用。因为在这种情况下，用户的行为决定了不需要保存Activity的状态。通常onSaveInstanceState()只适合用于保存一些临时性的状态，而onPause()适合用于数据的持久化保存。 在activity被杀掉之前调用保存每个实例的状态,以保证该状态可以在onCreate(Bundle)或者onRestoreInstanceState(Bundle) (传入的Bundle参数是由onSaveInstanceState封装好的)中恢复。这个方法在一个activity被杀死前调用，当该activity在将来某个时刻回来时可以恢复其先前状态。 例如，如果activity B启用后位于activity A的前端，在某个时刻activity A因为系统回收资源的问题要被杀掉，A通过onSaveInstanceState将有机会保存其用户界面状态，使得将来用户返回到activity A时能通过onCreate(Bundle)或者onRestoreInstanceState(Bundle)恢复界面的状态

四，Fragment声明周期
![[Pasted image 20210518145428.png]]
Fragment从创建到销毁整个生命周期中涉及到的方法依次为：onAttach()→onCreate()→ onCreateView()→onActivityCreated()→onStart()→onResume()→onPause()→onStop()→onDestroyView()→onDestroy()→onDetach()，其中和Activity有不少名称相同作用相似的方法，而不同的方法有:

-   **onAttach()**：当Fragment和Activity建立关联时调用；
-   **onCreateView()**：当fragment创建视图调用，在onCreate之后；
-   **onActivityCreated()**：当与Fragment相关联的Activity完成onCreate()之后调用；
-   **onDestroyView()**：在Fragment中的布局被移除时调用；
-   **onDetach()**：当Fragment和Activity解除关联时调用；

#### 四大组件之ContentProvider
进行跨进程通信，实现进程间的数据交互和共享。通过Context 中 getContentResolver() 获得实例，通过 Uri匹配进行数据的增删改查。ContentProvider使用表的形式来组织数据，无论数据的来源是什么，ConentProvider 都会认为是一种表，然后把数据组织成表格。

#### 动画
https://www.jianshu.com/p/b5fa7bf4fddd
  - View动画，作用对象是View，有平移、缩放、旋转、透明度四种特效
  - 帧动画，帧动画表示view切换过程中的动画效果，可以是activity之间的切换，也可以是view的切换，是在xml中定义好一系列资源后，使用AnimatonDrawable来播放的动画
  - 属性动画，属性动画一般是用的最多的一类动画，属性动画区别于其它动画的最主要特征是view的属性值的改变；原理是其实就是利用插值器和估值器，来计出各个时刻View的属性，然后通过改变View的属性来实现View的动画效果

#### 屏幕适配
1.首先获取设备PPI
![[Pasted image 20210518164836.png]]
2.获取1dp=多少个像素
1dp\*ppi/160 = px
3.查表获得分辨率
![[Pasted image 20210518164954.png]]
4.美工就可以按照查到的分辨率来设计图，工程师可以通过像素大厨等标注软件来抠图，量尺寸

#### Android个版本特性
 - 5.x，MaterialDesign设计风格+支持64位ART虚拟机(不同于JIT-Just In Time编译，改为了AOT-Ahead Of Time)
 - 6.x，动态权限管理+文件夹拖拽应用+快速充电切换+相机新增专业模式
 - 7.x，多窗口+V2签名+夜间模式
 - 8.x，Notification Channel+画中画模式+后台限制
 - 9.x，室内WIFI定位+刘海屏幕支持+安全增强
 - 10.x，屏幕录制+桌面模式+夜间模式(针对所有应用)+5G网络+折叠屏
 - 11.x，隔空切换歌曲+反向充电+HEIF图像格式

#### xml解析
- DOM解析:将全部xml数据加载到内存来解析
- SAX，PULL解析:事件驱动，(解析到一个元素就是一个事件)，按需加载数据


#### Context相关
-   1、Activity和Service以及Application的Context是不一样的,Activity继承自ContextThemeWraper.其他的继承自ContextWrapper。
    
-   2、每一个Activity和Service以及Application的Context是一个新的ContextImpl对象。
    
-   3、getApplication()用来获取Application实例的，但是这个方法只有在Activity和Service中才能调用的到。那也许在绝大多数情况下我们都是在Activity或者Servic中使用Application的，但是如果在一些其它的场景，比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext()方法，getApplicationContext()比getApplication()方法的作用域会更广一些，任何一个Context的实例，只要调用getApplicationContext()方法都可以拿到我们的Application对象。
    
-   4、创建对话框时不可以用Application的context，只能用Activity的context。
    
-   5、Context的数量等于Activity的个数 + Service的个数 +1，这个1为Application

#### android进程优先级
 - 前台进程
 - 可见进程
   处于暂停状态onPause的Activity或者绑定其上的Service，即被用户可见，但失去了焦点而不能与用户交互
 - 服务进程
   虽然不可见，但运行用户任务的服务Service，被用户关心的进程
 - 后台进程
   执行onStop而停止的进程，不是用户关心的进程
 - 空进程
   最先被回收的进程

##### 动态权限管理机制


##### Handler机制
https://blog.csdn.net/xk7298/article/details/104141352
![[Pasted image 20210518040122.png]]

1.调用Looper.prepare()
   会创建Looper对象，在该对象中又创建了MeeageQueue消息队列；然后又将该Looper对象绑定到当前线程（通过ThreadLocal）

2.创建Handler
构造Handler对象时会拿到当前线程的Looper，通过Looper和MessageQueue

3.开启Looper轮询
Looper.loop()，分发消息，处理消息

3.通过Handler对象发送消息到MessageQueue

总结：

-   **Handler**：负责发送和处理消息；
    
-   **Looper**：负责不断的轮询消息队列，并将消息 分发至Handler处理；
    
-   **MessageQueue**：存储消息的队列；
    
-   **Message**：消息载体

注意：
主线程中不需要调用prepare
  
 #### Parcelable
 1. Parcelable是Android为我们提供的序列化的接口
 2. Parcelable接口的实现类是通过Parcel写入和恢复数据的,并且必须要有一个非空的静态变量 CREATOR
 3. Parcel提供了一套机制，可以将序列化之后的数据写入到一个共享内存中，其他进程通过Parcel可以从这块共享内存中读出字节流，并反序列化成对象
  
#### AsyncTask
https://juejin.cn/post/6844904052552105991
一、 它是一个处理异步任务的抽象类，可以在线程池中执行后台任务，并把执行结果传递给主线程
- 三个泛型参数：Params、Progress、Result
- 四个核心方法：nPreExecute()、doInBackground()、onProgressUpdate()、onPostExecute()

二，源码
 包含一个Handler(InternalHandler)和两个线程池(serial executor，thread pool executor)
 
 1. InternalHandler负责自线程和主线程沟通，子任务执行完或取消通知主线程
 2. serial executor负责任务分发，它内部有个可存储Runnable的ArrayDeque队列，按串行顺序一次分发一个任务
 3. thread pool executor负责执行serial分发来的任务，按串行顺序一次执行一个任务
 4. Callable类似Runnable，是任务类，不同之处是可以返回结果；FutureTask是一个可以取消异步任务的管理类
 
三，注意事项
  1. 在子线程中new AsyncTask会因为Handler没有prepare而报错，execute必须在UI线程中调用
  2. 同一个AsyncTask对象，只能执行一次execute，否则会报再执行错误
  3. AsyncTask在Activity是非静态内部类是，默认持有外部Activity的应用，会导致内存泄漏
  4. Activity退出时，要在onDestory中做AsyncTask做isCancel检查，否则可能会导致Activity已退出，AsyncTask还更新其中的View控件
  5. cancel AsyncTask并不能立刻取消任务，因为不能中断正在执行的Thread，它只是在task上加了个中断标记

#### SQLite
一，源码分析
https://wqyjh.github.io/2016/12/22/Android-SQLite-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/
Android 中 SQLiteDatabase 的 INSERT，UPDATE，DELETE 方法都是由 SQLiteStatment 来执行的，这几个方法仅仅做了拼接 SQL 语句，然后把参数从 ContentValues 里面取出放到一个 Object\[\] 类型的数组里，然后通过 SQL 语句和这个 绑定参数的数组创建一个 SQLiteStatment 的实例，再由 SQLiteStatment 来执行查询。  
SQLiteStatment 的 INSERT，UPDATE，DELETE 也是一个简单的封装，它会获取当前线程的 SQLiteSession 对象，将参数基本原封不动的传给它，再由 SQLiteSession 对象来查询。  
SQLiteSession 代表了一个数据库的会话，会话执行的时候会从数据库连接池中获取一个数据库连接（SQLiteConnection），然后数据库连接可以通过 JNI 来调用 sqlite 驱动，从而真正的执行数据库方法。  
因此 Android 系统提供的 SQLiteDatabase 仅仅提供了 SQL 拼接，参数绑定，并发控制的功能。

二，事务
事务（Transaction）有两种，隐式事务和显式事务。任何一个非显式事务的数据库操作，都会创建一个隐式事务，它会在操作成功后马上提交（commit）。显式事务通过`beginTransaction()`方法创建，一旦一个显式事务被创建，一系列的子操作会被当做这个事务的一部分来执行。结束一个事务的时候，如果事务成功，首先应该调用`setTransactionSuccessful()`，然后再调用`endTransaction()`。如果事务被标记为成功，它就会被提交，否则回滚（rolled back）。显式事务也可以嵌套，如果任何一个子事务没有被标记为成功，整个事务都会在最外层事务结束的时候回滚。

三，面试题
1. SQLite事务是如何操作的
SQLite在做CRDU操作时都默认开启了事务，然后把SQL语句翻译成对应的SQLiteStatement并调用其相应的CRUD方法，此时整个操作还是在rollback journal这个临时文件上进行，只有操作顺利完成才会更新db数据库，否则会被回滚。

2. SQLite如何做批量操作
使用SQLiteDatabase的beginTransaction方法开启一个事务，将批量操作SQL语句转化为SQLiteStatement并进行批量操作，结束后endTransaction()

3. SQLite优化操作
 - 事务批量操作
 - 及时关闭cursor，避免内存泄漏
 - 数据库操作是耗时操作，建议放到异步线程中处理
 - ContentValue内部用的HashMap结构存储，初始化容量为8，要设计合理容量，避免扩容

4. SQLiteOpenHelper回调方法
- onOpen()，每次打开数据库的时候执行
 - onCreate()，创建数据库时会被执行
 - onUpgrade()，当数据库升级时会被执行，首次创建不会被执行
 
 5. 升级方法有三种
  - onCreate保证最新，onUpgrade中保存就数据，删除旧表，手动调用onCreate，将旧数据插入新表
  - 通过onUpgrade中的oldVersion和newVersion来更新表
  
  6. 常用sql语句
  - 创建table
```java
CREATE TABLE IF NOT EXISTS account " +
                    "(userId VERCHAR PRIMARY KEY,subAccountId VERCHAR,totalAccount INTEGER)
```
   - 新增column
```java
ALTER TABLE account ADD COLUMN subAccountId VARCHAR default 
```

#### 理解Window和WindowManager

#### Window加载DecorView
1. Activity视图
一个Activity包含了一个Window对象，这个对象是由PhoneWindow来实现的。PhoneWindow将DecorView作为整个应用窗口的根View，而这个DecorView又将屏幕划分为两个区域：一个是TitleView，另一个是ContentView，而我们平时所写的就是展示在ContentView中的

2. 加载机制
-   从Activity的startActivity开始，最终调用到ActivityThread的handleLaunchActivity方法来创建Activity，首先，会调用performLaunchActivity方法，内部会执行Activity的onCreate方法，从而完成DecorView和Activity的创建。然后，会调用handleResumeActivity，里面首先会调用performResumeActivity去执行Activity的onResume()方法，执行完后会得到一个ActivityClientRecord对象，然后通过record.window.getDecorView()的方式得到DecorView，然后会通过a.getWindowManager()得到WindowManager，最终调用其addView()方法将DecorView加进去。
-   WindowManager的实现类是WindowManagerImpl，它内部会将addView的逻辑委托给WindowManagerGlobal，可见这里使用了接口隔离和委托模式将实现和抽象充分解耦。在WindowManagerGlobal的addView()方法中不仅会将DecorView添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView通过root.setView()把DecorView加载到Window中。这里的ViewRootImpl是ViewRoot的实现类，是连接WindowManager和DecorView的纽带。View的三大流程均是通过ViewRoot来完成的

#### 视图绘制流程


#### View事件分发机制
1. MotionEvent类型
  -   ACTION\_DOWN
  -   ACTION\_MOVE(移动的距离超过一定的阈值会被判定为ACTION\_MOVE操作)
  -   ACTION\_UP

2. 事件分发流程
事件分发过程由三个方法共同完成：

  **dispatchTouchEvent**：方法返回值为true表示事件被当前视图消费掉；返回为super.dispatchTouchEvent表示继续分发，返回为false表示交给父类的onTouchEvent处理。

  **onInterceptTouchEvent**：方法返回值为true表示拦截这个事件并交由自身的onTouchEvent方法进行消费；返回false表示不拦截，需要继续传递给子视图的dispatch处理。如果return super.onInterceptTouchEvent(ev)， 事件拦截分两种情况: 　
   - 1.如果该View存在子View且点击到了该子View, 则不拦截, 继续分发 给子View 处理, 此时相当于return false。
   - 2.如果该View没有子View或者有子View但是没有点击中子View(此时ViewGroup 相当于普通View), 则交由该View的onTouchEvent响应，此时相当于return true

   **onTouchEvent**：方法返回值为true表示当前视图可以处理对应的事件；返回值为false表示当前视图不处理这个事件，它会被传递给父视图的onTouchEvent方法进行处理。如果return super.onTouchEvent(ev)，事件处理分为两种情况：

   - 1.如果该View是clickable或者longclickable的,则会返回true, 表示消费 了该事件, 与返回true一样;
   - 2.如果该View不是clickable或者longclickable的,则会返回false, 表示不 消费该事件,将会向上传递,与返回false一样

3. 在Android系统中，拥有事件传递处理能力的类有以下三种：
-   Activity：拥有分发和消费两个方法。
-   ViewGroup：拥有分发、拦截和消费三个方法。
-   View：拥有分发、消费两个方法

##### View事件分发结论
1. 事件传递优先级：onTouchListener.onTouch > onTouchEvent > onClickListener.onClick
2. ViewGroup默认不拦截任何事件
3. 通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程


#### ANR机制
https://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649493643&idx=1&sn=34b51d1f61bd2ecaa8fd0a2d39c4d1d1&chksm=8eec9b74b99b126246acc4547597dfe55c836b8f689b2d1a65bdf1ee2054ced2fc070bfa2678&mpshare=1&scene=24&srcid=0116vzNfMMv2dLizhAT8mEYq#rd


#### [[Android安全机制]]

##### Binder机制


##### 自定义View


#### Apk安装流程
 - 复制APK到/data/app目录下，解压并扫描安装包。

 - 资源管理器解析APK里的资源文件。

- 解析AndroidManifest文件，并在/data/data/目录下创建对应的应用数据目录。

- 然后对dex文件进行优化，并保存在dalvik-cache目录下。

- 将AndroidManifest文件解析出的四大组件信息注册到PackageManagerService中。

- 安装完成后，发送广播。


#### 打包流程(即描述清点击 Android Studio 的 build 按钮后发生了什么)
一，打包流程
![[Pasted image 20210519162022.png]]
-   通过AAPT工具进行资源文件（包括AndroidManifest.xml、布局文件、各种xml资源等）的打包，生成R.java文件。
-   通过AIDL工具处理AIDL文件，生成相应的Java文件。
-   通过Java Compiler编译R.java、Java接口文件、Java源文件，生成.class文件。
-   通过dex命令，将.class文件和第三方库中的.class文件处理生成classes.dex，该过程主要完成Java字节码转换成Dalvik字节码，压缩常量池以及清除冗余信息等工作。
-   通过ApkBuilder工具将资源文件、DEX文件打包生成APK文件。
-   通过Jarsigner工具，利用KeyStore对生成的APK文件进行签名。
-   如果是正式版的APK，还会利用ZipAlign工具进行对齐处理，对齐的过程就是将APK文件中所有的资源文件距离文件的起始距位置都偏移4字节的整数倍，这样通过内存映射访问APK文件的速度会更快，并且会减少其在设备上运行时的内存占用

二，apk内部结构
-   dex：最终生成的Dalvik字节码。
-   res：存放资源文件的目录。
-   asserts：额外建立的资源文件夹。
-   lib：如果存在的话，存放的是ndk编出来的so库。
-   META-INF：存放签名信息
-   androidManifest：程序的全局清单配置文件。
-   resources.arsc：编译后的二进制资源文件

#### Webview



#### 热修复、插件化、模块化、组件化、Gradle、编译插桩技术


#### 音视频，图片处理