#### Android系统框架
![[android系统框架.jpeg]]
Android底层以Linux Kernel作为基石，上层用户空间有Native系统库、虚拟机运行环境、框架层组成，通过系统调用(Syscall)连通内核空间和用户空间。用户空间主要有Java和C++代码编写，可以通过JNI打通Java层和Native层。

- Linux 内核（Linux Kernel），特点如下：
  1. 强大的内存管理和进程管理
  2. 基于权限的安全模式
  3. 支持共享库
  4. 经过认证的驱动模型
  5. 开源项目
  
- 硬件抽象层（HAL）
  1. HAL位于操作系统和驱动程序之上，运行在用户空间中的服务程序。其目的是为上层的应用提供一个统一的查询硬件设备的接口。有了HAL接口，就可以将硬件开发和上层的应用开发分离开，上层的应用开发不必关系具体实现是什么硬件，同样地，如果硬件厂家需要改变硬件设备，只需要按照HAL接口的规范和标准提供对应的硬件驱动，而不必更改应用。（HAL层并不提供对硬件的实际操作，对硬件的实际操作仍然由具体的驱动程序来完成。）
   2. HAL层的优势我们在前面已经提到，这是其中之一，另一个重要的原因就是为了保障在Android 平台基于Linux开发的硬件驱动和应用程序不必遵循GPL(General Public License)许可而保持封闭，保证硬件厂家的利益。我们都知道， Linux Kernel和Android的许可证不一样，Linux Kernel遵循的是GPL许可，Android遵循的是ASL(Apache SoftWare License)许可，ASL许可规定，可以随意使用源码，不必开源，所以建立在Android之上的硬件驱动和应用程序都可以保持封闭。也就是说，只要把 关键的驱动处理相关的主要逻辑转移到Android平台内，在Linux Kernel中国仅保留基础的通信功能，原本应该包括在Linux Kernel中的某些驱动的关键处理逻辑，被转移到了HAL层，保证了硬件厂家的利益的同时，也方便了软件的开发

- Android Runtime(系统运行库)
   1. Android 4.4前是Dalvik虚拟机，之后引入了ART虚拟机，5.0之后Dalvik被彻底丢弃
   2. ART不同于Dalvik虚拟机采用的JIT(Just-In-Time)编译模式，而是采用了AOT(Ahead-Of-Time)模式，应用在第一次安装时，字节码就会预先编译成机器码存储在本地；所以ART占用空间比Dalvik大，以空间换时间
   3. 由于ART减少了编译，也就减少了CPU使用频率，改善了电池续航，也使启动更快，运行更快，体验更流畅
   4. ART优化了垃圾回收机制
   
- 原生C/C++库(系统运行库)
  1. android系统组件和服务都是用C/C++编写的，为了方便开发者调用这些原生库功能，Android java framework提供了相应的API，例如可以通过Java OpenGL API访问OpenGL ES
  2. NDK（Native Develop Kit）工具包，是Android SDK的扩展，可以帮助开发者快速的开发C/C++库，并将so和java打包成apk；同时NDK还提供了交叉编译器，根据mk文件针对不同的CPI，ABI生成不同的so库
  
- Java API Framework
  1. UI组件
  2. 资源管理器
  3. 通知管理器
  4. Activity管理器
  5. Content Provider管理器

#### Linux进程和线程
1. 进程定义
  狭义：就是一段程序的执行过程
  广义：是一个具有独立功能的程序关于某个数据集合的一次运行过程，它是操作系统分配资源和调度的独立单元

2. 进程组成
  控制块，程序段，数据段三部分组成
  
3. 线程定义
  线程是操作系统能够调度执行的最小单元，被包含在进程中，是进程中的实际运作单位。
  
4. 进程和线程区别
  - 一个线程只能属于一个进程，而一个进程最少有一个线程
  - 线程是进程工作单位
  - 多个线程共享父进程地址空间
  - 一个线程的崩溃可能会影响到整个程序稳定性

5. 进程状态图
![[Pasted image 20210516174554.png]]


#### Linux文件系统


#### Android SDK，Android系统和Android SDK Manager关系
https://www.zhihu.com/question/415958735


#### [[Android安全机制]]


