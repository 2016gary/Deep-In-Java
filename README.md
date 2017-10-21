# Java Garbage Collection基础
### <a href="http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html">翻译自Oraclle官方文档</a>
### By Gary

---
# Index：
### 1.概述
### 2.Java技术和JVM
### 3.垃圾回收
### 4.分代垃圾回收的过程
### 5.Java垃圾收集器
### 6.如何监控JVM
### 7.总结

---
# 1.概述
## 目的
### 本教程将介绍垃圾回收如何与Hotspot JVM一起使用的基础知识。一旦了解了垃圾收集器的功能，了解如何使用Visual VM监视垃圾回收过程。最后，了解Java SE 7 Hotspot JVM中哪些垃圾收集器可用。

## 介绍
### 本OBE涵盖了Java中Java虚拟机（JVM）垃圾回收（GC）的基础知识。在OBE的第一部分中，提供了JVM的概述以及对垃圾回收和性能的介绍。接下来为学生提供了一个循序渐进的指导，了解垃圾回收如何在JVM中工作。接下来，为学习者提供了一个交流活动，尝试Java JDK中提供的一些监控工具，并将他们实践于刚刚了解的垃圾回收中的内容。最后，提供了一个部分，涵盖Hotspot JVM中可用的垃圾回收方案选项。

## 硬件和软件要求
### 以下是硬件和软件要求的列表：
- 运行于Windows XP或更高版本，Mac OS X或Linux的PC。请注意，手上已经完成了Windows 7，并没有在所有平台上进行测试。但是，应该在OS X或Linux上正常工作。而且，具有多于一个核的机器也是优选的。
- Java 7 Update 7或更高版本
- 最新的Java 7 demo和样例Zip文件

## 先决条件
### 在开始本教程之前，您应该：
- 如果还没有这样做，请下载并安装最新版本的Java JDK（JDK 7 u7或更高版本）。
- 从同一位置下载并安装Demos和Samples zip文件。 解压文件并将内容放在目录中。 例如：C：\ javademos

---
# 2.Java技术和JVM
## Java概述
### Java是Sun Microsystems于1995年首次发布的编程语言和计算平台。它是为Java程序（包括实用程序，游戏和业务应用程序）提供支持的基础技术。Java遍布全球超过8.5亿台个人计算机，全球数十亿台设备，包括手机和电视设备。Java由许多关键组件组成，整体而言，Java创建了Java平台。

## Java运行时版本
### 当您下载Java时，您将获得Java运行时环境（JRE）。JRE由Java虚拟机（JVM），Java平台核心类和支持的Java平台库组成。所有这三个都需要在您的计算机上运行Java应用程序。使用Java 7，Java应用程序作为桌面应用程序从操作系统运行，作为桌面应用程序运行，但是使用Java Web Start从Web安装，或者作为浏览器中的Web Embedded应用程序（使用JavaFX）运行。

## Java编程语言
### Java是面向对象的编程语言，包括以下功能：
- 平台独立性 - Java应用程序被编译为字节码，存储在类文件中并加载到JVM中。由于应用程序在JVM中运行，因此可以在许多不同的操作系统和设备上运行。
- 面向对象 - Java是面向对象的语言，它采用C和C ++的许多功能，并对其进行了改进。
- 自动垃圾回收 - Java自动分配和释放内存，使程序不负担任务。
- 丰富的标准库 - Java包含大量的内置对象，可用于执行输入/输出，网络和日期操作等任务。

## Java开发工具包
### Java开发工具包（JDK）是开发Java应用程序的工具集合。使用JDK，您可以编译以Java编程语言编写的程序，并在JVM中运行它们。此外，JDK提供了用于打包和分发应用程序的工具。 
### JDK和JRE共享Java应用程序编程接口（Java API）。Java API是开发人员用来创建Java应用程序的预包装库的集合。Java API通过提供工具来完成许多常见的编程任务（包括字符串操作，日期/时间处理，网络和实现数据结构（例如列表，映射，堆栈和队列）），从而使开发变得更简单。

## Java虚拟机
### Java虚拟机（JVM）是一种抽象计算机。JVM是一个程序，看起来像写入要执行的程序的机器。这样，Java程序就被写入同一组接口和库。用于特定操作系统的每个JVM实现将Java编程指令转换为在本地操作系统上运行的指令和命令。这样，Java程序实现平台独立性。
### 在Sun Microsystems，Inc.完成的Java虚拟机的第一个原型实现模拟了由类似于当代个人数字助理（PDA）的手持设备托管的软件中的Java虚拟机指令集。Oracle当前的实现模拟了移动，桌面和服务器设备上的Java虚拟机，但Java虚拟机不承担任何特定的实现技术，主机硬件或主机操作系统。它不是固有的解释，但也可以通过将其指令集编译为硅CPU的指令集来实现。它也可以在微代码中或直接在硅中实现。
### Java虚拟机也不知道什么是Java编程语言，只有特定的二进制格式，类文件格式。类文件包含Java虚拟机指令（或字节码）和符号表以及其他辅助信息。
### 为了安全起见，Java虚拟机对类文件中的代码强加了句法和结构限制。但是，具有可以用有效的类文件表达的功能的任何语言都可以由Java虚拟机托管。受到通常可用的独立于机器的平台的吸引，其他语言的实现者可以将Java虚拟机转换为其语言的传送工具。

## 探索JVM体系结构
### Hotspot结构
### HotSpot JVM具有支持强大的功能和功能基础的架构，并支持实现高性能和大规模可扩展性的能力。例如，HotSpot JVM JIT编译器生成动态优化。换句话说，它们在Java应用程序运行时进行优化决策，并生成针对底层系统架构的高性能本地机器指令。此外，通过其运行时环境和多线程垃圾收集器的成熟演进和持续工程，即使是最大的可用计算机系统，HotSpot JVM也可提供高可扩展性。

<img src="/JVM/hotspot_architecture.png"  alt="图片无法显示" />

### JVM的主要组件包括类加载器，运行时数据区和执行引擎。

### Hotspot主要组件
### 与性能相关的JVM的关键组件在下图中突出显示。

<img src="/JVM/hotspot_key_component.png"  alt="图片无法显示" />

### 当调整性能时，JVM有三个组件。堆是存储对象数据的地方。然后，该区域由启动时选择的垃圾收集器进行管理。大多数调优选项涉及到调整堆的大小，并为您的情况选择最合适的垃圾回收器。JIT编译器也对性能有很大的影响，但很少需要使用较新版本的JVM进行调优。

## 性能的基础
### 通常，在调整Java应用程序时，重点放在两个主要目标之一：响应性或吞吐量。随着教程的进行，我们将回顾这些概念。
### 响应
### 响应性是指应用程序或系统对所请求的数据进行响应的速度。例子包括：
- 桌面UI对事件的响应速度有多快
- 网站返回页面的速度有多快
- 数据库查询返回的速度有多快
### 对于专注于响应性的应用程序，不能接受大的暂停时间。重点是在短时间内做出回应。
### 吞吐量
### 吞吐量专注于在特定时间内最大限度地提高应用程序的工作量。 如何测量吞吐量的示例包括：
- 在特定时间内完成的事物数量。
- 批处理程序可以在一小时内完成的作业数。
- 可在一小时内完成的数据库查询数。
### 对于专注于吞吐量的应用，高暂停时间是可以接受的。由于高吞吐量应用在较长时间内集中于基准测试，因此不需要考虑快速的响应时间。

---
# 3.垃圾回收
## 什么是自动垃圾回收？
### 自动垃圾回收是查看堆内存的过程，识别哪些对象正在使用，哪些对象不在，以及删除未使用的对象。使用对象或引用对象意味着程序的某些部分仍然保留指向该对象的指针。未使用的对象或未引用的对象不再由程序的任何部分引用。因此，可以回收未引用对象使用的内存。
### 在像C这样的编程语言中，分配和释放内存是一个手动过程。在Java中，重新分配内存的进程由垃圾收集器自动处理。基本过程可以描述如下。

## 步骤1：标记
### 该过程的第一步称为标记。这是垃圾收集器识别哪些内存块正在使用的地方，哪些不在。

<img src="/JVM/marking.png"  alt="图片无法显示" />

### 引用的对象以蓝色显示。未引用的对象以金色显示。所有对象在标记阶段进行扫描，以进行此确定。如果必须扫描系统中的所有对象，这可能是非常耗时的过程。

## 步骤2：正常删除
### 正常删除未引用的对象，将引用的对象和指针留给可用空间。

<img src="/JVM/normal_deletion.png"  alt="图片无法显示" />

### 内存分配器保存对可以分配新对象的可用空间块的引用。

## 步骤2a：通过压缩删除
### 为了进一步提高性能，除了删除未引用的对象之外，还可以压缩剩余的引用对象。通过将引用的对象一起移动，这使得新的内存分配更容易和更快。

<img src="/JVM/deletion_compacting.png"  alt="图片无法显示" />

## 为什么要分代垃圾回收？
### 如前所述，标记和压缩JVM中的所有对象是低效的。随着越来越多的对象被分配，对象列表增长并且增长导致更长和更长的垃圾回收时间。然而，应用程序的实证分析表明，大多数对象生命周期都是短暂的。
### 以下是这些数据的示例。Y轴显示分配的字节数，X访问显示随时间改变分配的字节数。

<img src="/JVM/ObjectLifetime.gif"  alt="图片无法显示" />

### 如您所见，随着时间的推移，对象数量越来越少。事实上，大多数对象的寿命非常短，如图左侧的较高值所示。

## JVM代
### 从对象分配行为学到的信息可以用来增强JVM的性能。 因此，堆被分解成较小的部分或几代。堆分为：年轻代，老年代，永久代

<img src="/JVM/hotspot_heap_structure.png"  alt="图片无法显示" />

### 年轻代是所有新对象被分配和老化的地方。当年轻代填满时，这会导致少量的垃圾回收。小样本可以优化，假设对象死亡率高。一个充满死亡对象的年轻代很快就被回收起来了。一些幸存的对象被老化，最终转移到老年代。
### Stop the World Event-所有小垃圾回收都是“Stop the World”事件。这意味着所有应用程序线程都将停止，直到操作完成。
### 老年代用于储存长寿命的物品。通常，对于年轻代对象设置一个阈值，当满足该年龄时，该对象将被移动到老年代。最终需要回收老年代。这个事件被称为主要的垃圾回收。
### 主要垃圾回收也是停止世界事件。通常，一个主要的回收要慢得多，因为它涉及所有存活的对象。因此，对于响应性应用程序，应尽量减少主要的垃圾回收。另请注意，主要垃圾回收的Stop World事件的时间受到老年代空间所使用的垃圾回收器的影响。
### 永久代包含JVM所需的元数据来描述应用程序中使用的类和方法。永久代在运行时由基于应用程序使用的类填充JVM。此外，java SE库类和方法可以被存储在这里。
### 如果JVM发现不再需要类，并且可能需要其他类的空间，则可以回收（卸载）类。永久代被包含在一个完整的垃圾回收中。

---
# 4.分代垃圾回收的过程
### 现在你明白了为什么堆被分成不同的代，现在是时候看这些空间是如何相互作用的。随后的图片将遍历JVM中的对象分配和老化过程。
### 1.首先，将任何新对象分配给eden空间。 两个幸存空间开始空。

<img src="/JVM/object_allocation.png"  alt="图片无法显示" />

### 2.当eden空间填满时，会触发小的垃圾回收。

<img src="/JVM/filling_eden_space.png"  alt="图片无法显示" />

### 3.被引用的对象被移动到第一幸存空间。当eden空间被清除时，未引用的对象被删除。

<img src="/JVM/copying_referenced_object.png"  alt="图片无法显示" />

### 4.在下一个小的GC中，同样的事情发生在eden空间。未引用的对象被删除，引用对象被移动到幸存空间。然而，在这种情况下，它们被移动到第二幸存空间（S1）。另外，来自第一幸存空间（S0）的最后一个小的GC的对象的年龄增加并被移动到S1。一旦所有幸存的对象都移动到S1，S0和eden都被清除。请注意，我们现在在幸存空间中具有不同的老化对象。

<img src="/JVM/object_aging.png"  alt="图片无法显示" />

### 5.在下一个小的GC中，重复相同的过程。但这次幸存空间转换。被引用的对象被移动到S0。幸存的对象是老化的。Eden和S1被清除。

<img src="/JVM/additional_aging.png"  alt="图片无法显示" />

### 6.经过小的GC，当老年对象达到一定的年龄阈值（在这个例子中8），他们从年轻代变为老年代。

<img src="/JVM/promotion.png"  alt="图片无法显示" />

### 7.年轻代继续给新的对象分配内存将继续推动对象往老年代空间的变迁。

<img src="/JVM/promotion2.png"  alt="图片无法显示" />

### 8.所以这几乎涵盖了与年轻代的整个过程。最终，一个主要的GC将在老年代进行，清理和压缩这个空间。

<img src="/JVM/gc_summary.png"  alt="图片无法显示" />

---
# 7.总结
### 在本OBE中，您已经了解了Java JVM上垃圾回收系统的概述。首先，您了解了堆和垃圾收集器是任何Java JVM的关键部分。使用分代数垃圾回收方法实现自动垃圾回收。一旦你学到了这个过程，你就用Visual VM工具观察过了。最后，您回顾了Java Hospot JVM中可用的垃圾回收器。
### 在本教程中，您了解到：
- Java JVM的组件
- 自动垃圾回收如何工作
- 分代垃圾回收过程
- 如何使用Visual VM监控JVM
- JVM上可用的垃圾收集器类型
