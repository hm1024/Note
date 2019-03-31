## 第一章 走进Java
## Java技术体系
世界上没有完美的程序，但我们并不因此而沮丧，因为写程序本来就是一个不断追求完美的过程。
java语言第一次提出：Write Once, Run Anywhere. => 一次编写，到处运行。
Java的**优点**：

* 是一门结构严谨、面向对象的编程语言。
* 摆脱了平台的束缚，实现“一次编写，到处运行”的理想；
* 提供了一个相对安全的内存管理和访问机制，避免了绝大多数的内存泄漏和指针越界问题。
* 实现了热点代码检测和运行时编译优化，使的Java应用程序能随着运行时间的增加而获得更高的性能。
* 有一套完善的应用程序接口，还有无数来自商业机构和开源社区的第三方类库来帮助它实现各种各样的功能。
  Java所带来的这些好处使程序的开发效率得到了很大的提升。

认识技术运作的本质，是自己思考**程序这样写好不好**的基础和前提。当我们在使用一种技术时，如果不在依赖书本和他人就能得到这些问题的答案，那才算上升到了**不惑**的境界。

**Java中最重要的技术特性的实现原理**

**句柄（handle）:**
有多种意义，其中第一种是指程序设计，第二种是指Windows编程。现在大部分都是指程序设计/程序开发这类。

* 第一种解释：句柄是一种特殊的智能指针 。当一个应用程序要引用其他系统（如数据库、操作系统）所管理的内存块或对象时，就要使用句柄。
* 第二种解释：整个Windows编程的基础。一个句柄是指使用的一个唯一的整数值，即一个4字节(64位程序中为8字节)长的数值，来标识应用程序中的不同对象和同类中的不同的实例，诸如，一个窗口，按钮，图标，滚动条，输出设备，控件或者文件等。应用程序能够通过句柄访问相应的对象的信息，但是句柄不是指针，程序不能利用句柄来直接阅读文件中的信息。如果句柄不在I/O文件中，它是毫无用处的。 句柄是Windows用来标志应用程序中建立的或是使用的唯一整数，Windows大量使用了句柄来标识对象。

## 1.2 Java技术体系   
Sun官方所定义的Java技术体系：
* Java程序设计语言
* 各种硬件平台上的Java虚拟机
* Class文件格式
* Java API类库
* 来自商业机构和开源社区的第三方Java类库

通常把Java程序设计语言、Java虚拟机、JavaAPI类库这三部分统称为JDK（Java Development Kit），JDK是用于支持Java程序开发的最小环境。另外把Java API类库中的Java SE API子集和Java虚拟机这两部分统称为JRE（Java Runtime Environment),JRE是支持Java程序运行的标准环境。

**Java体系所包含的内容**
![Snipaste_2019-03-31_20-14-39.jpg](https://i.loli.net/2019/03/31/5ca0af56b47ba.jpg)

以上是根据各个组成部分的功能来进行划分的，如果按照技术所服务的领域来划分，或者说按照Java技术关注的重点业务领域来划分，Java技术体系可以分为4个平台，分别为：
* Java Card：支持一些Java小程序（Applets）运行在小内存设备（如智能卡）上的平台。
* Java ME（Micro Edition）：支持Java程序运行在移动终端（手机、PDA）上的平台，对Java API有所精简，并加入了针对移动终端的支持，这个版本以前称为J2ME。
* Java SE（Standard Edition）：支持面向桌面级应用（如Windows下的应用程序）的Java平台，提供了完整的Java核心API，这个版本以前称为J2SE。
* Java EE（Enterprise Edition）：支持使用多层架构的企业应用（如ERP、CRM应用）的Java平台，除了提供Java SE API外，还对其做了大量的扩充[3]并提供了相关的部署支持，这个版本以前称为J2EE。

## 1.3 Java发展史
Java技术发展的时间线
![Snipaste_2019-03-31_20-21-04.jpg](https://i.loli.net/2019/03/31/5ca0b0bc49510.jpg)

1996年1月23日，JDK 1.0发布，Java语言有了第一个正式版本的运行环境。JDK 1.0提供了一个纯解释执行的Java虚拟机实现（Sun Classic VM）。JDK 1.0版本的代表技术包括：Java虚拟机、Applet、AWT等。

1997年2月19日，Sun公司发布了JDK 1.1，Java技术的一些最基础的支撑点（如JDBC 等）都是在JDK 1.1版本中发布的，JDK 1.1版的技术代表有：JAR文件格式、JDBC、 JavaBeans、RMI。Java语法也有了一定的发展，如内部类（Inner Class）和反射 （Reflection）都是在这个时候出现的。

1998年12月4日，JDK迎来了一个里程碑式的版本JDK 1.2，工程代号为Playground（竞技 场），Sun在这个版本中把Java技术体系拆分为3个方向，分别是面向桌面应用开发的 J2SE（Java 2 Platform,Standard Edition）、面向企业级开发的J2EE（Java 2 Platform,Enterprise Edition）和面向手机等移动终端开发的J2ME（Java 2 Platform,Micro Edition）。在这个版本 中出现的代表性技术非常多，如EJB、Java Plug-in、Java IDL、Swing等，并且这个版本中 Java虚拟机第一次内置了JIT（Just In Time）编译器（JDK 1.2中曾并存过3个虚拟机，Classic VM、HotSpot VM和Exact VM，其中Exact VM只在Solaris平台出现过；后面两个虚拟机都是 内置JIT编译器的，而之前版本所带的Classic VM只能以外挂的形式使用JIT编译器）。在语 言和API级别上，Java添加了strictfp关键字与现在Java编码之中极为常用的一系列Collections 集合类。在1999年3月和7月，分别有JDK 1.2.1和JDK 1.2.2两个小版本发布。 

1999年4月27日，HotSpot虚拟机发布，HotSpot最初由一家名为“Longview Technologies”的小公司开发，因为HotSpot的优异表现，这家公司在1997年被Sun公司收购 了。HotSpot虚拟机发布时是作为JDK 1.2的附加程序提供的，后来它成为了JDK 1.3及之后所 有版本的Sun JDK的默认虚拟机。 

2000年5月8日，工程代号为Kestrel（美洲红隼）的JDK 1.3发布，JDK 1.3相对于JDK 1.2 的改进主要表现在一些类库上（如数学运算和新的Timer API等），JNDI服务从JDK 1.3开始 被作为一项平台级服务提供（以前JNDI仅仅是一项扩展），使用CORBA IIOP来实现RMI的 通信协议，等等。这个版本还对Java 2D做了很多改进，提供了大量新的Java 2D API，并且 新添加了JavaSound类库。JDK 1.3有1个修正版本JDK 1.3.1，工程代号为Ladybird（瓢虫）， 于2001年5月17日发布。 

自从JDK 1.3开始，Sun维持了一个习惯：大约每隔两年发布一个JDK的主版本，以动物 命名，期间发布的各个修正版本则以昆虫作为工程名称。

2002年2月13日，JDK 1.4发布，工程代号为Merlin（灰背隼）。JDK 1.4是Java真正走向 成熟的一个版本，Compaq、Fujitsu、SAS、Symbian、IBM等著名公司都有参与甚至实现自己 独立JDK1.4。JDK 1.4同样发布 了很多新的技术特性，如正则表达式、异常链、NIO、日志类、XML解析器和XSLT转换器 等。JDK 1.4有两个后续修正版：2002年9月16日发布的工程代号为Grasshopper（蚱蜢）的 JDK 1.4.1与2003年6月26日发布的工程代号为Mantis（螳螂）的JDK 1.4.2。

2004年9月30日，JDK 1.5[1]发布，工程代号Tiger（老虎）。从JDK 1.2以来，Java在语法 层面上的变换一直很小，而JDK 1.5在Java语法易用性上做出了非常大的改进。例如，自动装箱、泛型、动态注解、枚举、可变长参数、遍历循环（foreach循环）等语法特性都是在JDK 1.5中加入的。在虚拟机和API层面上，这个版本改进了Java的内存模型（Java Memory Model,JMM）、提供了java.util.concurrent并发包等。

2006年12月11日，JDK 1.6发布，工程代号Mustang（野马）。在这个版本中，Sun终结了 从JDK 1.2开始已经有8年历史的J2EE、J2SE、J2ME的命名方式，启用Java SE 6、Java EE 6、Java ME 6的命名方式。JDK 1.6的改进包括：提供动态语言支持（通过内置Mozilla JavaScript Rhino引擎实现）、提供编译API和微型HTTP服务器API等。同时，这个版本对Java 虚拟机内部做了大量改进，包括锁与同步、垃圾收集、类加载等方面的算法都有相当多的改 动。 

在2006年11月13日的JavaOne大会上，Sun公司宣布最终会将Java开源，并在随后的一年 多时间内，陆续将JDK的各个部分在GPL v2（GNU General Public License v2）协议下公开了 源码，并建立了OpenJDK组织对这些源码进行独立管理。，OpenJDK的质量主管曾经表示，在JDK 1.7中，Sun JDK和 OpenJDK除了代码文件头的版权注释之外，代码基本上完全一样，所以OpenJDK 7与Sun JDK 1.7本质上就是同一套代码库开发的产品。 

2009年2月19日，工程代号为Dolphin（海豚）的JDK 1.7完成了其第一个里程碑版本。2009年4月20日，Oracle公司宣布正式以74亿美元的价格收购Sun公司，Java商标从此正 式归Oracle所有（Java语言本身并不属于哪间公司所有，它由JCP组织进行管理，尽管JCP主 要是由Sun公司或者说Oracle公司所领导的）。

根据Oracle官方提供的信息，JDK 1.8的第一个正式版本将于2013年9月发布，JDK 1.8将 会提供在JDK 1.7中规划过，但最终未能在JDK 1.7中发布的特性，即Lambda表达式、 Jigsaw（很不幸，随后Oracle公司又宣布Jigsaw在JDK 1.8中依然无法完成，需要延至JDK 1.9）和JDK 1.7中未实现的一部分Coin等。

在2011年的JavaOne大会上，Oracle公司还提到了JDK 1.9的长远规划，希望未来的Java虚 拟机能够管理数以GB计的Java堆，能够更高效地与本地代码集成，并且令Java虚拟机运行时 尽可能少人工干预，能够自动调节。 

## 1.4 Java虚拟机发展史
###　1.4.1　Sun Classic/Exact VM
以今天的视角来看，Sun Classic VM的技术可能很原始，这款虚拟机的使命也早已终 结。但仅凭它“世界上第一款商用Java虚拟机”的头衔，就足够有让历史记住它的理由。 

**Sun Classic** 只能使用纯解释器方式来执行Java代码， 如果要使用JIT编译器，就必须进行外挂。但是假如外挂了JIT编译器，JIT编译器就完全接管 了虚拟机的执行系统，解释器便不再工作了。由于解释器和编译器不能配合工作，这就意味着如果要使用编译器执行，编译器 就不得不对每一个方法、每一行代码都进行编译，而无论它们执行的频率是否具有编译的价 值。基于程序响应时间的压力，这些编译器根本不敢应用编译耗时稍高的优化技术，因此这 个阶段的虚拟机即使用了JIT编译器输出本地代码，执行效率也和传统的C/C++程序有很大差 距，“**Java语言很慢**”的形象就是在这时候开始在用户心中树立起来的。 

Sun的虚拟机团队努力去解决Classic VM所面临的各种问题，提升运行效率。在JDK 1.2 时，曾在Solaris平台上发布过一款名为Exact VM的虚拟机，它的执行系统已经具备现代高性 能虚拟机的雏形：如两级即时编译器、编译器与解释器混合工作模式等。Exact VM因它使用 准确式内存管理（Exact Memory Management，也可以叫Non-Conservative/Accurate Memory Management）而得名，即虚拟机可以知道内存中某个位置的数据具体是什么类型。譬如内存 中有一个32位的整数123456，它到底是一个reference类型指向123456的内存地址还是一个数 值为123456的整数，虚拟机将有能力分辨出来，这样才能在GC（垃圾收集）的时候准确判 断堆上的数据是否还可能被使用。

虽然Exact VM的技术相对Classic VM来说先进了许多，但是在商业应用上只存在了很短 暂的时间就被更为优秀的HotSpot VM所取代，甚至还没有来得及发布Windows和Linux平台下
的商用版本。而Classic VM的生命周期则相对长了许多，它在JDK 1.2之前是Sun JDK中唯一 的虚拟机，在JDK 1.2时，它与HotSpot VM并存，但默认使用的是Classic VM（用户可用javahotspot参数切换至HotSpot VM），而在JDK 1.3时，HotSpot VM成为默认虚拟机，但Classic VM仍作为虚拟机的“备用选择”发布（使用java-classic参数切换），直到JDK 1.4的时 候，Classic VM才完全退出商用虚拟机的历史舞台，与Exact VM一起进入了Sun Labs Research VM之中。

### 1.4.2　Sun HotSpot VM
Sun HotSpot VM是Sun JDK和OpenJDK中所带的虚拟 机，也是目前使用范围最广的Java虚拟机。HotSpot VM既继承了Sun之前两款商用虚拟机的优点（如前面提到的准确式内存管 理），也有许多自己新的技术优势，如它名称中的HotSpot指的就是它的**热点代码探测技术** 。

### 1.4.3　Sun Mobile-Embedded VM/Meta-Circular VM
### 1.4.4　BEA JRockit/IBM J9 V
### 1.4.5　Azul VM/BEA Liquid VM
### 1.4.6　Apache Harmony/Google Android Dalvik VM
### 1.4.7　Microsoft JVM及其他
## 1.5 展望Java技术的未来
### 1.5.1　模块化
模块化是解决应用系统与技术平台**越来越复杂、越来越庞**大问题的一个重要途径。无论 是开发人员还是产品最终用户，都不希望为了系统中一小块的功能而不得不下载、安装、部 署及维护整套庞大的系统。站在整个软件工业化的高度来看，模块化是建立各种功能的标准 件的前提。
###　1.5.2　混合语言
当单一的Java开发已经无法满足当前软件的复杂需求时，越来越多基于Java虚拟机的语 言开发被应用到软件项目中，Java平台上的多语言混合编程正成为主流，每种语言都可以针 对自己擅长的方面更好地解决问题。

通过特定领域的语言去解决特定领域的 问题是当前软件开发应对日趋复杂的项目需求的一个方向。

![Snipaste_2019-03-31_21-32-58.jpg](https://i.loli.net/2019/03/31/5ca0c1964c9b8.jpg)

对这些运行于Java虚拟机之上、Java之外的语言，来自系统级的、底层的支持正在迅速 增强，以JSR-292为核心的一系列项目和功能改进（如Da Vinci Machine项目、Nashorn引擎、 InvokeDynamic指令、java.lang.invoke包等），推动Java虚拟机从“**Java语言的虚拟机**”向“**多语 言虚拟机**”的方向发展。 
### 1.5.3  多核并行
CPU硬件的发展方向已经从高频率转变为多核心，随着多核时代的来临，软件开 发越来越关注并行编程的领域。早在JDK 1.5就已经引入java.util.concurrent包实现了一个粗粒 度的并发框架。而JDK 1.7中加入的java.util.concurrent.forkjoin包则是对这个框架的一次重要 扩充。Fork/Join模式是处理并行编程的一个经典方法，如图1-5所示。虽然不能解决所有的问 题，但是在此模式的适用范围之内，能够轻松地利用多个CPU核心提供的计算资源来协作完 成一个复杂的计算任务。通过利用Fork/Join模式，我们能够更加顺畅地过渡到多核时代。
![Snipaste_2019-03-31_21-37-57.jpg](https://i.loli.net/2019/03/31/5ca0c2be5a2ed.jpg)

在Java 8中，将会提供Lambda支持，这将会极大改善目前Java语言不适合函数式编程的 现状（目前Java语言使用函数式编程并不是不可以，只是会显得很臃肿），函数式编程的一 个重要优点就是这样的程序天然地适合并行运行，这对Java语言在多核时代继续保持主流语 言的地位有很大帮助。

另外，在并行计算中必须提及的还有OpenJDK的子项目Sumatra[2]，目前显卡的算术运算 能力、并行能力已经远远超过了CPU，在图形领域以外发掘显卡的潜力是近几年计算机发展 的方向之一，例如C语言的CUDA。Sumatra项目就是为Java提供使用GPU（Graphics Processing Units）和APU（Accelerated Processing Units）运算能力的工具，以后它将会直接提 供Java语言层面的API，或者为Lambda和其他JVM语言提供底层的并行运算支持。 

在JDK外围，也出现了专为满足并行计算需求的计算框架，如Apache的Hadoop Map/Reduce，这是一个简单易懂的并行框架，能够运行在由上千个商用机器组成的大型集群 上，并且能以一种可靠的容错方式并行处理TB级别的数据集。另外，还出现了诸如Scala、 Clojure及Erlang等天生就具备并行计算能力的语言。
### 1.5.4　进一步丰富语法
Java 5曾经对Java语法进行了一次扩充，这次扩充加入了自动装箱、泛型、动态注解、 枚举、可变长参数、遍历循环等语法，使得Java语言的精确性和易用性有了很大的进步。在 Java 7（由于进度压力，许多改进已推迟至Java 8）中，对Java语法进行了另一次大规模的扩 充。Sun（已被Oracle收购）专门为改进Java语法在OpenJDK中建立了[Coin子项目](http://wikis.sun.com/display/ProjectCoin/Home)来统一处 理对Java语法的细节修改，如二进制数的原生支持、在switch语句中支持字符串、“＜＞”操 作符、异常处理的改进、简化变长参数方法调用、面向资源的try-catch-finally语句等都是在 Coin项目之中提交的内容。 

除了Coin项目之外，在JSR-335（Lambda Expressions for the Java TM Programming Language）中定义的[Lambda表达式](http://openjdk.java.net/projects/lambda/)也将对Java的语法和语言习惯产生很大的影响，面向函数 方式的编程可能会成为主流。 

### 1.5.5 64位虚拟机