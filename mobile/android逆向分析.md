# Android逆向分析
## 1. 版本历史
|版本号|日期|修订人|描述|
|:----|:----|:----|:---|
|0.01|2016.01.04|黄文博|初稿|

## 2. 文档目的
- 深入理解android软件安全与逆向分析
- 整合android相关技术文档
- 开发人员在看雪论坛学习理解相关软件安全方面知识[看雪论坛](http://bbs.pediy.com)

## 3. 目标读者
- 平台部android开发人员
- 二次打包相关成员
 
## 4. 主要内容
### 4.1 apk生成过程
请参考:《[android打包过程](apk-你从哪里来.md)》

### 4.2 apk反编译之旅
#### 4.2.1 Android环境搭建
- 安装JDK
- 安装Android SDK
- 安装Android NDK
- 集成开发环境

Android 开发环境搭建相关资料很多，不在此过多描述。
#### 4.2.2 Android Dalvik虚拟机与指令集
- Dalvik虚拟机与java虚拟机的区别
	
	(1)java虚拟机运行的是java字节码，Dalvik虚拟机运行的是Dalvik字节码
	
	(2)Dalvik可执行文件体积更小
	
	(3)Java虚拟机与Dalvik虚拟机架构不同

- Dalvik指令格式

	(1)每16位的字采用空格分隔开来
	
	(2)每个字母表示四位，每个字母按照顺序从高字节开始，排列到低字节，每四位之间可能使用竖线“｜”来表示不同内容
	
	(3)顺序采用A～Z的单个大写字母作为一个4位的操作码，op表示一个8位的操作码
	
	(4)指令格式标识大多由三个字符组成，前两个是数字，最后一个是字母

- Dalvik指令集
	
	(1)指令特点
	Dalvik 指令在调用格式上模仿了C语言的调用约定，Dalvik指令的语法与助词符有如下特点：
	
	参数采用从目标（destination）到源（source）的方式。
	
	根据字节码的大小与类型不同，一些字节码添加了名称后缀以消除歧义。32位常规类型的字节码未添加任何后缀。64位常规类型的字节码添加－wide后缀。特殊类型的字节码根据具体类型添加后缀，他们可以是-boolean,-byte,-char,-short,-int,-long,-float,-double,-object,-string,-class,-void之一。根据字节码的布局与选项不同，一些字节码添加了字节码后缀以消除歧义，这些后缀通过在字节码主名称后添加斜杠"/"来分隔开
	
	(2)空操作指令
	
	(3)数据操作指令
	
	(4)返回指令
	
	(5)数据定义指令
	
	(6)锁指令
	
	(7)实例操作指令
	
	(8)数组操作指令
	
	(9)异常指令
	
	(10)跳转指令
	
	(11)比较指令
	
	(12)字段操作指令
	
	(13)方法调用指令
	
	(14)数据转换指令
	
	(15)数据运算指令
			
		
#### 4.2.3 android常用反编译方法
- jar反编译
	
	在分析大型软件时，为了弄清楚程序的结构框架，需要花费大量的时间与精力来阅读smali代码，这无疑是分析成本的一大开销，然而，Android程序大多数情况下采用java语言开发，传统意义上的Java反汇编工具依然可用派上用场
	
	我们在Android程序生成步骤中其中有一个环节时将Java语言的字节码转换成Dalvik虚拟机的字节码，同样，这个过程也是可逆的，使用dex2jar工具即可。

	示例：

		d2j-dex2jar.sh ~/Desktop/dex2jar/herodefence.apk
	这个步骤便是将apk文件转化为jar，接着我们便可以使用jd-gui工具查看改apk的java代码了。
	
	下载链接：
	
	1、[dex2jar官网](http://code.google.com/p/dex3jar/)	2、[jd-gui官网](http://java.decompiler.free.fr/)		

	dex2jar是一个工具包，除了提供dex文件转换成jar文件外，还提供了一些其他的功能：
	
	d2j-apk-sign用来为apk文件签名。命名格式：d2j-apk-sign xxx.apk
	
	d2j-asm-verify用来验证jar文件。命令格式：d2j-asm-verify xxx.apk
	
	d2j-dex2jar用来将dex文件转换成jar文件(可直接转换apk文件)。命令格式：d2j-dex2jar xxx.apk
	
	d2j-dex-asmifier用来验证dex文件。命令格式：d2j-dex-asmifier xxx.apk
	
	d2j-dex-dump用来转存dex文件的信息。命令格式：d2j-dex-dump xxx.apk out.jar
	
	d2j-init-deobf用来生成反混淆jar文件时的初始化配置文件。
	
	d2j-jar2dex用来将jar文件转换成dex文件。命令格式：d2j-dex2jar xxx.apk
	
	d2j-jar2jasmin用来将jar文件转换成jasmin格式的文件。命令格式：d2j-jar2jasmin xxx.jar
	
	d2j-jar-access用来修改jar中的类、方法已经字段的访问权限。
	
	d2j-jar-remap用来重命名jar文件中的包、类、方法以及字段的名称。
	
	d2j-jarmin2jar用来将jar文件转换成jasmin格式的文件。命令格式：d2j-jarmin2jar dir
	
	dex2jar为d2j-dex2jar的副本。
	
	dex-dump为d2j-dex-dump的副本。
- dex反编译

	(1)我们当然可以使用上述dex2jar工具将dex文件转化成jar，然后通过jd-gui来查看java级的源码

	(2)我们知道dex文件是Davlik虚拟机的可执行文件，我们在上面介绍了Dalvik虚拟机指令集，那我们可以将apk代码转换致汇编级语言来进行反编译，获取apk的代码逻辑
	
	目前dex可执行文件主流的反汇编工具有BakSmalil与Dedexer，两者但反汇编效果都不错，在语法上也有很多相似处。

- so反编译

#### 4.2.4 Android静态调试与动态调试
- 静态分析

	静态分析是指在不运行代码的情况下，采用此法分析，语法分析等各种技术手动对程序中的文件进行扫描从而生成程序的反汇编代码，然后阅读反汇编代码来掌握程序功能的一种技术
	
	静态分析的两种方法：
	
	(1)阅读反汇编生成的Dalvik字节码，可以使用IDA Pro分析dex文件，或者使用文本编辑器阅读baksmali反编译生成的smali文件
	
	(2)阅读反汇编生成的java源码，可以使用dex2jar生成jar文件，然后再使用jd－gui阅读jar文件的代码
	
	IDA Pro是目前全世界最强大的静态反汇编分析工具，它具备可交互、可编程、可扩展、多处理器支持等众多特点，但是
	
	`IDA Pro是商业收费软件，而且价格不菲。`
	
- 动态调试

	软件调试可分为源码级调试与汇编级调试，源码级调试多用于软件开发阶段，开发人员拥有软件的源码，可以用过集成开发环境中的调试器跟踪运行自己的软件，解决软件中的错误；汇编级调试就是动态调试，多用于软件的逆向工程，分析人员没有软件源代码，调试程序时只能跟踪与分析汇编代码，查看寄存器的值，这些数据远远没有源码级调试展示的信息那么直观，但动态调试程序同样能跟踪软件的执行流程，反馈程序执行时的中间结果，在静态分析程序难以取得突破时，动态调试也是一种行之有效的逆向手段

### 4.3 apk二次打包
### 4.3.1 二次打包原理
我们将编译apk的过程称为一次打包，二次打包其实就是将apk反编译之后按照相应的操作重新编译生成apk的过程

### 4.3.2 实战应用
- 修改smali文件

	阅读反编译的smali代码：

	(1)循环语句

	(2)switch分支语句

	(3)try/catch语句	

- 在破解的apk目录中加入资源文件或smali文件

我们把破解后的apk加入资源文件或smali文件的目的是什么呢，我们可以按照我们的需求，在破解后的apk中添加我们需要的功能

- apk二次打包

这个过程是将smali文件与资源文件重新打包生成apk的过程，我们可以使用apktool工具来生成apk，但是这个apk是未被签名的apk，我们可以使用signapk对未签名的apk文件进行签名。

	apktool b outdir

## 5. 遗留问题
### 5.1 如何把已知jar转换为相应的smail文件
 