# Java Garbage Collection基础
### <a href="http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html">翻译自Oraclle官方文档</a>
### By Gary

---
# Index：
### 1.概述
### 2.Java技术和JVM
### 3.垃圾回收
### 4.分代垃圾回收的过程
### 5.Java垃圾回收器

---
# 1.概述
## 目的
### 本教程将介绍垃圾回收如何与Hotspot JVM一起使用的基础知识。一旦了解了垃圾回收器的功能，了解如何使用Visual VM监视垃圾回收过程。最后，了解Java SE 7 Hotspot JVM中哪些垃圾回收器可用。

## 介绍
### 本OBE涵盖了Java中Java虚拟机（JVM）垃圾回收（GC）的基础知识。在OBE的第一部分中，提供了JVM的概述以及对垃圾回收和性能的介绍。接下来为学生提供了一个循序渐进的指导，了解垃圾回收如何在JVM中工作。接下来，为学习者提供了一个交流活动，尝试Java JDK中提供的一些监控工具，并将他们实践于刚刚了解的垃圾回收中的内容。最后，提供了一个部分，涵盖Hotspot JVM中可用的垃圾回收方案选项。

## 硬件和软件要求
### 以下是硬件和软件要求的列表：
- 运行于Windows XP或更高版本，Mac OS X或Linux的PC。请注意，手上已经完成了Windows 7，并没有在所有平台上进行测试。但是，应该在OS X或Linux上正常工作。而且，具有多于一个核的机器也是优选的。
- Java 7 Update 7或更高版本

## 先决条件
### 在开始本教程之前，您应该：
- 如果还没有这样做，请下载并安装最新版本的Java JDK（JDK 7 u7或更高版本）。

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
### HotSpot JVM具有支持强大的功能和功能基础的架构，并支持实现高性能和大规模可扩展性的能力。例如，HotSpot JVM JIT编译器生成动态优化。换句话说，它们在Java应用程序运行时进行优化决策，并生成针对底层系统架构的高性能本地机器指令。此外，通过其运行时环境和多线程垃圾回收器的成熟演进和持续工程，即使是最大的可用计算机系统，HotSpot JVM也可提供高可扩展性。

<img src="./Images/hotspot_architecture.png"  alt="图片无法显示" />

### JVM的主要组件包括类加载器，运行时数据区和执行引擎。

### Hotspot主要组件
### 与性能相关的JVM的关键组件在下图中突出显示。

<img src="./Images/hotspot_key_component.png"  alt="图片无法显示" />

### 当调整性能时，JVM有三个组件。堆是存储对象数据的地方。然后，该区域由启动时选择的垃圾回收器进行管理。大多数调优选项涉及到调整堆的大小，并为您的情况选择最合适的垃圾回收器。JIT编译器也对性能有很大的影响，但很少需要使用较新版本的JVM进行调优。

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
### 在像C这样的编程语言中，分配和释放内存是一个手动过程。在Java中，重新分配内存的进程由垃圾回收器自动处理。基本过程可以描述如下。

## 步骤1：标记
### 该过程的第一步称为标记。这是垃圾回收器识别哪些内存块正在使用的地方，哪些不在。

<img src="./Images/marking.png"  alt="图片无法显示" />

### 引用的对象以蓝色显示。未引用的对象以金色显示。所有对象在标记阶段进行扫描，以进行此确定。如果必须扫描系统中的所有对象，这可能是非常耗时的过程。

## 步骤2：正常删除
### 正常删除未引用的对象，将引用的对象和指针留给可用空间。

<img src="./Images/normal_deletion.png"  alt="图片无法显示" />

### 内存分配器保存对可以分配新对象的可用空间块的引用。

## 步骤2a：通过压缩删除
### 为了进一步提高性能，除了删除未引用的对象之外，还可以压缩剩余的引用对象。通过将引用的对象一起移动，这使得新的内存分配更容易和更快。

<img src="./Images/deletion_compacting.png"  alt="图片无法显示" />

## 为什么要分代垃圾回收？
### 如前所述，标记和压缩JVM中的所有对象是低效的。随着越来越多的对象被分配，对象列表增长并且增长导致更长和更长的垃圾回收时间。然而，应用程序的实证分析表明，大多数对象生命周期都是短暂的。
### 以下是这些数据的示例。Y轴显示分配的字节数，X访问显示随时间改变分配的字节数。

<img src="./Images/ObjectLifetime.gif"  alt="图片无法显示" />

### 如您所见，随着时间的推移，对象数量越来越少。事实上，大多数对象的寿命非常短，如图左侧的较高值所示。

## JVM代
### 从对象分配行为学到的信息可以用来增强JVM的性能。 因此，堆被分解成较小的部分或几代。堆分为：年轻代，老年代，永久代

<img src="./Images/hotspot_heap_structure.png"  alt="图片无法显示" />

### 年轻代是所有新对象被分配和老化的地方。当年轻代填满时，这会导致少量的垃圾回收。小样本可以优化，假设对象死亡率高。一个充满死亡对象的年轻代很快就被回收起来了。一些幸存的对象被老化，最终转移到老年代。
### Stop the World Event-所有小垃圾回收都是“Stop the World”事件。这意味着所有应用程序线程都将停止，直到操作完成。
### 老年代用于储存长寿命的物品。通常，对于年轻代对象设置一个阈值，当满足该年龄时，该对象将被移动到老年代。最终需要回收老年代。这个事件被称为主要的垃圾回收。
### 主要垃圾回收也是停止世界事件。通常，一个主要的回收要慢得多，因为它涉及所有活动的对象。因此，对于响应性应用程序，应尽量减少主要的垃圾回收。另请注意，主要垃圾回收的Stop World事件的时间受到老年代空间所使用的垃圾回收器的影响。
### 永久代包含JVM所需的元数据来描述应用程序中使用的类和方法。永久代在运行时由基于应用程序使用的类填充JVM。此外，java SE库类和方法可以被存储在这里。
### 如果JVM发现不再需要类，并且可能需要其他类的空间，则可以回收（卸载）类。永久代被包含在一个完整的垃圾回收中。

---
# 4.分代垃圾回收的过程
### 现在你明白了为什么堆被分成不同的代，现在是时候看这些空间是如何相互作用的。随后的图片将遍历JVM中的对象分配和老化过程。
### 1.首先，将任何新对象分配给eden空间。 两个幸存空间开始空。

<img src="./Images/object_allocation.png"  alt="图片无法显示" />

### 2.当eden空间填满时，会触发小的垃圾回收。

<img src="./Images/filling_eden_space.png"  alt="图片无法显示" />

### 3.被引用的对象被移动到第一幸存空间。当eden空间被清除时，未引用的对象被删除。

<img src="./Images/copying_referenced_object.png"  alt="图片无法显示" />

### 4.在下一个小的GC中，同样的事情发生在eden空间。未引用的对象被删除，引用对象被移动到幸存空间。然而，在这种情况下，它们被移动到第二幸存空间（S1）。另外，来自第一幸存空间（S0）的最后一个小的GC的对象的年龄增加并被移动到S1。一旦所有幸存的对象都移动到S1，S0和eden都被清除。请注意，我们现在在幸存空间中具有不同的老化对象。

<img src="./Images/object_aging.png"  alt="图片无法显示" />

### 5.在下一个小的GC中，重复相同的过程。但这次幸存空间转换。被引用的对象被移动到S0。幸存的对象是老化的。Eden和S1被清除。

<img src="./Images/additional_aging.png"  alt="图片无法显示" />

### 6.经过小的GC，当老年对象达到一定的年龄阈值（在这个例子中8），他们从年轻代变为老年代。

<img src="./Images/promotion.png"  alt="图片无法显示" />

### 7.年轻代继续给新的对象分配内存将继续推动对象往老年代空间的变迁。

<img src="./Images/promotion2.png"  alt="图片无法显示" />

### 8.所以这几乎涵盖了与年轻代的整个过程。最终，一个主要的GC将在老年代进行，清理和压缩这个空间。

<img src="./Images/gc_summary.png"  alt="图片无法显示" />

---
# 5.Java垃圾回收器
### 在本节中，您将了解可用于Java的垃圾回收器以及您需要选择它们的命令行开关。

## 常用的堆相关的开关
### 有许多不同的命令行开关可以与Java一起使用。 本节介绍一些本OBE中使用的更常用的开关。

<table><thead><tr><td>Switch</td><td>Description</td></tr></thead><tbody><tr><td>-Xms</td><td>设置JVM启动时的初始堆大小。</td></tr><tr><td>-Xmx</td><td>设置最大堆大小。</td></tr><tr><td>-Xmn</td><td>设置年轻代的大小。</td></tr><tr><td>-XX:PermSize</td><td>设置永久代的起始大小。</td></tr><tr><td>-XX:MaxPermSize</td><td>设置永久代的最大大小。</td></tr></tbody></table>

## 串行GC
### 串行回收器是Java SE 5和6中客户端风格计算机的默认设置。对于串行回收器，次要和主要垃圾回收都是连续完成的（使用单个虚拟CPU）。 此外，它使用标记压缩的回收方法。此方法将较老的内存移动到堆的开头，以便在堆的末尾将新的内存分配制作为单个连续的内存块。这种内存压缩使得将更多的内存分配给堆很快。

## 用例
### 串行GC是大多数应用程序的首选垃圾回收器，它们没有低暂停时间要求并在客户端机器上运行。它仅利用单个虚拟处理器进行垃圾回收工作（因此，它的名称）。然而，在今天的硬件上，串行GC可以通过几百MB的Java堆有效地管理大量非平凡的应用程序，而相对较短的最坏情况暂停（完全垃圾回收约需要几秒钟）。
### 串行GC的另一个常见用途是在同一台机器上运行大量JVM（在某些情况下比可用处理器更多的JVM）的环境中。在JVM执行垃圾回收的这种环境中，最好只使用一个处理器来尽可能减少对其余JVM的干扰，即使垃圾回收可能持续更长时间。而串行GC适合这种权衡。
### 最后，随着嵌入式硬件的扩展，内存最少，内核少，串行GC可能会复出。

## 命令行开关
### 要使用串行回收器：
### -XX:+UseSerialGC

## 并行GC
### 并行垃圾回收器使用多个线程来执行年轻代垃圾回收。默认情况下，在具有N个CPU的主机上，并行垃圾回收器在回收中使用N个垃圾回收器线程。垃圾回收器线程的数量可以通过命令行选项进行控制：
### -XX:ParallelGCThreads=desired number
### 在具有单个CPU的主机上，即使已经请求并行垃圾回收器，也使用默认垃圾回收器。在具有两个CPU的主机上，并行垃圾回收器通常与默认垃圾回收器一样执行，并且可以预期在具有两个以上CPU的主机上可以减少年轻代回收器暂停时间。

## 用例
### 并行回收器也称为吞吐量回收器。因为它可以使用多个CPU来加快应用程序吞吐量。当需要做大量工作并且可以长时间停顿时，应该使用这个回收器。例如，批处理，或执行大量数据库查询。
### -XX:+UseParallelGC
### 通过这个命令行选项，您可以得到一个具有单线程老年代回收器的多线程年轻代回收器。该选项也是老年代的单线程压缩。
### -XX:+UseParallelOldGC
### 使用-XX：+ UseParallelOldGC选项，GC既是多线程的年轻代回收器又是多线程的老年代回收器。它也是一个多线程压缩回收器。HotSpot只能在老年代中进行压缩。HotSpot的年轻代被认为是一个复制回收器; 因此，不需要压缩。
### 压缩描述了以对象之间没有缝隙的方式移动对象的动作。垃圾回收扫描后，对象之间可能有缝隙。压缩移动对象，使得没有剩余的缝隙。垃圾回收器可能是非压缩回收器。因此，并行回收器和并行压缩回收器之间的差异可能是垃圾回收扫描之后的空间压缩。前者不会。

## 并发标记扫描（CMS）回收器
### 并发标记扫描（CMS）回收器（也称为并发低暂停回收器）回收老年代。它尝试通过与应用程序线程同时执行大部分垃圾回收工作来最小化由于垃圾回收引起的暂停。通常，并发的低暂停回收器不会复制或压缩活动对象。在不移动活动对象的情况下完成垃圾回收。如果碎片成为问题，请分配更大的堆。
### 注意：年轻代的CMS回收器使用与并行回收器相同的算法。

## 用例
### CMS回收器应用于需要较短暂停时间的应用程序，并可与垃圾回收器共享资源。用于包括响应事件的桌面UI应用程序，响应请求的Web服务器或响应查询的数据库。

## 命令行开关
### 使用CMS回收器：
### -XX:+UseConcMarkSweepGC
### 并设置线程使用次数：
### -XX:ParallelCMSThreads=N

## G1垃圾回收器
### G1垃圾回收器在Java 7中可用，旨在成为CMS回收器的长期替代品。G1回收器是一个并行，并发和增量式压缩的低暂停垃圾回收器，与之前描述的其他垃圾回收器具有完全不同的布局。但是，详细的讨论超出了本OBE的范围。

## 命令行开关
### 使用G1回收器：
### -XX:+UseG1GC

---
# G1垃圾回收器入门
### <a href="http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html">翻译自Oraclle官方文档</a>
### By Gary

---
## Index:
### 1.概述
### 2.G1垃圾回收器
### 3.使用CMS查看GC
### 4.一步一步G1垃圾回收器
### 5.命令行选项和最佳实践
### 6.用G1记录GC

---
# 1.概述
## 目的
### 本教程介绍了如何使用G1垃圾回收器以及如何与Hotspot JVM一起使用的基础知识。您将了解G1回收器在内部的功能，使用G1的关键命令行开关以及用于记录其操作的选项。

## 介绍
### 本OBE涵盖了Java虚拟机（JVM）G1垃圾回收（GC）的基础知识。在OBE的第一部分中，提供了JVM的概述以及对垃圾回收和性能的介绍。接下来学生将获得有关CMS回收器如何与Hotspot JVM一起使用的评论。接下来，一步一步地指导在使用带有Hotspot JVM的G1垃圾回收时垃圾回收的工作原理。接下来，提供了一个部分，涵盖G1垃圾回收器提供的Garbage Collection命令行选项。最后，您将了解与G1回收器一起使用的日志记录选项。

---
# 2.G1垃圾回收器
### 垃圾回收器（G1）是一种服务器式垃圾回收器，针对具有大量内存的多处理器机器。它以高概率满足垃圾回收（GC）暂停时间目标，同时实现高吞吐量。Oracle JDK 7更新4及更高版本完全支持G1垃圾回收器。G1回收器专为以下应用而设计：
- 可以与CMS回收器等应用程序线程同时运行。
- 压缩空闲空间，无长时间的GC停顿时间。
- 需要更可预测的GC暂停持续时间。
- 不想牺牲大量的吞吐量性能。
- 不需要更大的Java堆。
### G1计划作为并行标记扫描回收器（CMS）的长期替代品。将G1与CMS进行比较，存在差异，使G1成为更好的解决方案。一个区别是G1是一个压缩回收器。G1压缩，足以完全避免使用细粒度的自由列表进行分配，而是依赖于区域。这大大简化了回收器的部分，并且大部分消除了潜在的碎片问题。此外，G1提供比CMS回收器更可预测的垃圾回收暂停，并允许用户指定所需的暂停目标。

## G1操作概述
### 老的垃圾回收器（串行，并行，CMS）都将堆结构分为三个部分：年轻代，老年代和固定内存大小的永久代。

<img src="./Images/HeapStructure.png"  alt="图片无法显示" />

### 所有内存对象最后都在这三个部分之一中。G1回收器采取不同的方法。

<img src="./Images/g1_heap.png"  alt="图片无法显示" />

### 堆被分成一组相等大小的堆区，每个区都是连续的虚拟内存范围。某些区域集与老的回收器分配的角色相同（eden，幸存区，老年代），但是它们的大小不一样。这提供了更大的内存使用灵活性。
### 执行垃圾回收时，G1以类似于CMS回收器的方式运行。G1执行并发的全局标记阶段来确定整个堆中对象的活动性。标记阶段完成后，G1知道哪些区域大部分是空的。它首先在这些地区回收，这通常产生大量的可用空间。这就是为什么垃圾回收方法叫做G1。顾名思义，G1将其回收和压缩活动区中在可能充满可回收对象的堆的区域，也就是垃圾。G1使用暂停预测模型来满足用户定义的暂停时间目标，并根据指定的暂停时间目标选择要回收的区域数量。
### 由G1认定为成熟的区域是通过疏散内存回收垃圾。G1将对象从堆的一个或多个区域复制到堆上的单个区域，并且在此过程中，压缩和释放内存。这种疏散内存在多处理器上并行执行，以减少暂停时间并提高吞吐量。因此，对于每个垃圾回收，G1连续地减少碎片，在用户定义的暂停时间内工作。这超出了以前的两种方法的能力。CMS（并发标记扫描）垃圾回收器不执行压缩。ParallelOld垃圾回收仅执行整堆压缩，这导致相当多的暂停时间。
### 重要的是要注意，G1不是一个实时的回收器。它以高概率符合设定的暂停时间目标，但不能绝对确定。根据以前收集的数据，G1估计可以在用户指定的目标时间内回收多少个区域。因此，回收器具有回收区域成本的相当准确的模型，并且使用该模型来确定在暂停时间目标期间回收哪些和多少区域。
### 注意：G1同时（与应用程序线程一起运行，例如，细化，标记，清除）和并行（多线程，例如停止世界）阶段。完整的垃圾回收仍然是单线程的，但是如果正确调整，您的应用程序应避免使用full GC。

## G1踪迹
### 如果从ParallelOldGC或CMS回器迁移到G1，则可能会看到更大的JVM进程大小。这主要与“accounting”数据结构相关，如Remembered Sets和Collection Sets。
### Remembered Sets或RSets跟踪对象引用到给定的区域。堆中每个区域有一个RSet。RSet可以并行和独立地回收一个区域。RSets的整体踪迹影响小于5％。
### Collection Sets或CSets将在GC中回收的一组区域。GC中的所有实时数据在GC期间疏散（复制/移动）。一组地区可以是Eden，幸存区和/或老年代。CSets对JVM的大小影响不到1％。

## 推荐用于G1的用例
### G1的第一个重点是为运行应用程序的用户提供一个解决方案，这些应用程序需要大量的具有有限GC延迟的大堆 这意味着大约6GB或更大的堆大小，以及稳定和可预测的暂停时间低于0.5秒。
### 如果应用程序具有一个或多个以下特征，则使用CMS或ParallelOldGC垃圾回收器运行的应用程序将有利于切换到G1。
- Full GC持续时间太长或太频繁。
- 对象分配率或变迁率差异很大。
- 不期望的长期回收或压缩暂停（超过0.5至1秒）
### 注意：如果您正在使用CMS或ParallelOldGC，并且您的应用程序没有经历长时间的垃圾回收中止，则保持与您的当前回收器是很好的。更改为G1回收器不是使用最新JDK的要求。

---
# 3.使用CMS查看GC
## 回顾分代GC和CMS
### 并发标记扫描（CMS）回收器（也称为并发低暂停回收器）回收老年代。它尝试通过与应用程序线程同时执行大部分垃圾回收工作来最小化由于垃圾回收引起的暂停。通常，并发的低暂停回收器不会复制或压缩活动对象。在不移动活动对象的情况下完成垃圾回收。如果碎片成为问题，请分配更大的堆。
## CMS回收阶段
### CMS回收器在旧版本的堆上执行以下阶段：

<table><thead><tr><td>Phase</td><td>Description</td></tr></thead><tbody><tr><td>（1）初始标记
（停止世界事件）</td><td>老年代的对象被“标记”为可触及，包括可能从年轻代到达的对象。暂停时间通常相对于次要回收暂停时间短</td></tr><tr><td>（2）并发标记</td><td>在Java应用程序线程正在执行的同时遍历可达对象的老年代对象图。从标记的对象开始扫描，并将从根部到达的所有对象进行传递。突发器在并发阶段2,3和5期间执行，并且在这些阶段（包括升级对象）期间在CMS代中分配的任何对象都将立即标记为存活。</td></tr><tr><td>（3）备注
（停止世界事件）</td><td>查找并发标记阶段遗漏的对象，因为在并发回收器完成跟踪该对象后，Java应用程序线程会更新到对象。</td></tr><tr><td>（4）并发扫描</td><td>在标记阶段回收标识为不可达的对象。死对象的集合将对象的空间添加到空闲列表中以供以后分配。此时可能会发生死亡对象的聚结。请注意，活动对象不会移动。</td></tr><tr><td>（5）复位</td><td>通过清除数据结构准备下一个并发回收。</td></tr></tbody></table>

## 回顾垃圾回收步骤
### 接下来，我们一步一步查看CMS回收器操作。
### 1.CMS回收器堆结构
### 堆被分成三个空间。

<img src="./Images/cms_structure.png"  alt="图片无法显示" />

### 年轻代分裂成Eden和两个幸存空间。老年代是一个连续的空间。对象集合完成。除非有full GC，否则不进行压缩。

### 2.年轻代GC如何在CMS中工作
### 年轻代是浅绿色，老年代是蓝色的。如果您的应用程序运行了一段时间，这可能是CMS的表现。对象分散在老年代。

<img src="./Images/young_gc.png"  alt="图片无法显示" />

### 使用CMS，老年代对象被释放到位。他们没有移动。空间不紧凑，除非有完整的GC。

### 3.年轻代回收
### 活动对象从Eden和幸存空间复制到另一个幸存空间。任何已达到老化阈值的老对象都会升为老年代。

<img src="./Images/young_collection.png"  alt="图片无法显示" />

### 4.年轻代GC后
### 在年轻代GC之后，Eden空间被清除，其中一个幸存空间被清除。

<img src="./Images/after_gc.png"  alt="图片无法显示" />

### 新提升的对象在图中以深蓝色显示。绿色的对象是幸存下来的一代尚未晋升为老年代的年轻代对象。

### 5.CMS老年代回收
### 两次停止世界事件发生：初始标记和备注。当老年代达到一定的入住率时，CMS被启动。

<img src="./Images/old_collection.png"  alt="图片无法显示" />

### （1）初始标记是一个短暂的暂停阶段，其中存在（可达到的）对象被标记。（2）并发标记在应用程序继续执行时查找活动对象。最后，在（3）备注阶段，发现在上一阶段（2）并发标记期间遗漏的对象。

### 6.老年代回收 - 并列扫描
### 在前一阶段没有标记的对象被释放到位。没有压缩。

<img src="./Images/concurrent_sweep.png"  alt="图片无法显示" />

### 注意：未标记的对象==死对象

### 7.老年代回收 - 清扫后
### 在（4）扫描阶段之后，你可以看到很多内存被释放。您还会注意到没有压缩已经完成。

<img src="./Images/after_sweeping.png"  alt="图片无法显示" />

### 最后，CMS回收器将通过（5）复位阶段，等待下一次达到GC阈值。

---
# 4.一步一步G1垃圾回收器
### G1回收器采用不同的方法分配堆。后续的图片逐步回顾了G1系统。

### 1.G1堆结构
### 堆是一个分为许多固定大小区域的内存区域。

<img src="./Images/g1_structure.png"  alt="图片无法显示" />

### 区域大小由JVM在启动时选择。JVM通常针对2000个区域，大小从1到32Mb不等。

### 2.G1堆分配
### 实际上，这些区域被映射为Eden，幸存区和老年代空间的逻辑表示。

<img src="./Images/g1_allocation.png"  alt="图片无法显示" />

### 图片中的颜色显示哪个区域与哪个角色相关联。活动对象从一个区域撤出（即复制或移动）到另一个区域。区域被设计为与或不停止所有其他应用程序线程并行回收。
### 如图所示，这些区域可以分配到Eden，幸存区和老年代区域。另外还有第四类被称为Humongous区域的对象。 这些区域被设计为保持标准区域或更大尺寸的50％的对象。它们被存储为一组连续的区域。最后，最后一种类型的区域将是堆的未使用区域。
### 注意：在撰写本文时，回收humongous对象尚未被优化。因此，您应避免创建此大小的对象。

### 3.G1中的年轻代
### 堆被分为大约2000个区域。最小尺寸为1Mb，最大尺寸为32Mb。蓝色区域拥有老年代的对象，绿色区域拥有年轻代的对象。

<img src="./Images/young_g1.png"  alt="图片无法显示" />

### 请注意，这些区域不像老的垃圾回收器那样连续。

### 4.G1中年轻代的GC
### 活动对象被撤离（即复制或移动）到一个或多个幸存区域。如果满足老化阈值，某些对象将被升级到老年代区域。

<img src="./Images/young_gc_g1.png"  alt="图片无法显示" />

### 这是一个停止世界（STW）的暂停。对于下一个年轻代的GC计算Eden大小和幸存区大小。保存Accounting信息以帮助计算大小。像暂停时间目标这样的事情被考虑在内。
### 这种方法使得非常容易调整区域大小，使其根据需要变得更大或更小。

### 5.G1年轻代GC的结束
### 活动对象已被撤离到幸存区或老年代。

<img src="./Images/end_young_gc.png"  alt="图片无法显示" />

### 最近升级的对象以深蓝色显示。幸存区用绿色。
### 总之，关于G1的年轻代可以说如下：
- 堆是单个内存空间分割成区域。
- 年轻代内存由一组非连续的区域组成。这使得在需要时可以轻松调整大小。
- 年轻代的垃圾回收，或年轻代GC是停止世界事件。所有应用程序线程都停止运行。
- 年轻代GC是使用多个线程并行完成的。
- 活动对象被复制到新的幸存区或老年代区。

## G1的老年代回收
### 像CMS回收器一样，G1回收器被设计为用于老年代对象的低暂停回收器。下表介绍了老年代的G1回收阶段。

## G1回收阶段 - 并发标记循环阶段
### G1回收器在老年代的堆上执行以下阶段。请注意，一些阶段是年轻代的回收部分。

<table><thead><tr><td>Phase</td><td>Description</td></tr></thead><tbody><tr><td>（1）初始标记
（停止世界事件）</td><td>这是停止世界事件。使用G1，它搭载在正常的年轻代GC上。标记幸存区域（根区域），它们可能引用了老年代中的对象。</td></tr><tr><td>（2）根区域扫描</td><td>扫描幸存区参考老年代。这在应用程序继续运行时发生。必须在发生年轻代的GC之前完成。</td></tr><tr><td>（3）并发标记</td><td>在整个堆中查找活动对象。这在应用程序运行时发生。这个阶段可以被年轻代垃圾回收中断。</td></tr><tr><td>（4）备注（停止世界事件）</td><td>在堆中完成活动对象的标记。使用一种名为snapshot-at-the-begin（SATB）的算法，该算法比CMS回收器中使用的算法快得多。</td></tr><tr><td>（5）清理（停止世界事件和并发）</td><td>
- 执行活动对象和完全自由区域的统计。（停止世界）
- 清理记忆集。（停止世界）
- 重置空区域并将其返回到空闲列表。（并发）</td></tr><tr><td>（*）复制（停止世界事件）</td><td>这些是停止世界暂停将活动对象撤离或复制到新的未使用区域。这可以用记录为[GC暂停（年轻）]的年轻代区域完成。或者作为[GC暂停（混合）]记录的年轻和老年代区域。</td></tr></tbody></table>

### G1老年代一步一步回收
### 通过这些阶段的定义，我们来看看它们如何与G1回收器中的老年代进行交互。
### 6.初始标记阶段
### 活动对象的初始标记搭载在年轻代的垃圾回收中。在日志中，这被称为GC暂停（年轻代）（初始标记）。

<img src="./Images/g1_phase.png"  alt="图片无法显示" />

### 7.并行标记阶段
### 如果找到空区域（如“X”所示），则在备注阶段立即删除它们。此外，计算出确定活跃度的“accounting”信息。

<img src="./Images/g1_marking_phase.png"  alt="图片无法显示" />

### 8.备注阶段
### 空区域被删除和回收。现在，对所有区域计算区域活跃度。

<img src="./Images/g1_remark.png"  alt="图片无法显示" />

### 9.复制/清理阶段
### G1选择“活跃度”最低的区域，可以最快回收的区域。那么这些区域就是和年轻代的GC同时回收。这在日志中被表示为[GC pause（mixed）]。 所以同时回收年轻代和老年代。

<img src="./Images/g1_clean_phase.png"  alt="图片无法显示" />

### 10.复制/清理阶段后
### 所选择的区域已被回收并压缩成如图所示的深蓝色区域和深绿色区域。

<img src="./Images/g1_after_clean_phase.png"  alt="图片无法显示" />

### 老年代GC概要
### 总而言之，我们可以提到关于老年代的G1垃圾回收的几个要点。
- 并行标记阶段
	> 在应用程序运行的同时计算活动信息。
	
	> 这种活动信息确定在撤离暂停期间哪些区域最适合回收。

	> 没有像CMS这样的扫荡阶段。
- 备注阶段
	> 使用快照开始（SATB）算法，这比CMS使用的快得多。
	
	> 完全空的区域被回收。
- 复制/清理阶段
	> 年轻代和老年代同时被回收。
	
	> 根据活动信息选择老年代。

---
# 5.命令行选项和最佳实践
### 在本节中，我们来看看G1的各种命令行选项。
## 基本命令行
### 使用G1回收器：-XX:+UseG1GC

## 基本的命令行开关
### -XX:+UseG1GC - 告诉JVM使用G1垃圾回收器。
### XX:MaxGCPauseMillis=200 - 设置最大GC暂停时间的目标。这是一个软目标，JVM将尽力实现。因此暂停时间的目标有时候不会得到满足。 默认值为200毫秒。
### -XX:InitiateHeapOccupancyPercent=45 - 开始并发GC循环的（整个）堆占用的百分比。G1使用它来触发并发的GC循环，基于整个堆的占用，而不仅仅是某一代。值为0表示“做恒定GC循环”。默认值为45（即45％满或占用）。

## 最佳实践
### 使用G1时，应遵循一些最佳实践。
### 不要设置年轻代的大小
### 使用G1回收器的默认行为，通过-Xmn插入显式设置年轻代的大小。
- G1将不再尊重回收的暂停时间目标。所以在本质上，设置年轻代的大小会阻止暂停时间的目标。
- G1不再能够根据需要扩大和收缩年轻代的空间。由于大小是固定的，所以不能改变大小。

## 响应时间指标
### 而不是使用平均响应时间（ART）作为度量来设置XX：MaxGCPauseMillis=N，请考虑设置满足90％或更长时间的目标的值。 这意味着90％的用户提出请求将不会超过目标的响应时间。请记住，暂停时间是一个目标，不能保证始终得到满足。

## 什么是疏散失败？
### 当幸存区GC和升级对象的GC期间，JVM用尽堆区时，会发生升级失败。堆不能扩展，因为它已经在最大。当通过空间溢出使用-XX:+PrintGCDetails时，这在GC日志中显示。这很昂贵！
- GC仍然必须继续，所以空间必须被释放。
- 未成功复制的对象必须升级到老年代。
- 必须重新生成CSet中区域的RSets的任何更新。
- 所有这些步骤都是昂贵的。

## 如何避免疏散失败
### 为避免疏散故障，请考虑以下选项。
- 增加堆大小
- 增加-XX:G1ReservePercent=n，默认值为10。
- 如果需要更多的“空间”，则G1将通过尝试离开备用内存来创建虚假的上限。
- 开始标记循环
- 使用-XX:ConcGCThreads=n选项增加标记线程的数量。

## G1 GC开关的完整列表
### 这是G1 GC开关的完整列表。请记住使用上述最佳做法。

<table><thead><tr><td>Option and Default Value</td><td>Description</td></tr></thead><tbody><tr><td>-XX:+UseG1GC</td><td>使用垃圾回收（G1）回收器</td></tr><tr><td>-XX:MaxGCPauseMillis=n</td><td>设置最大GC暂停时间的目标。这是一个软目标，JVM将尽力实现。</td></tr><tr><td>-XX:InitiatingHeapOccupancyPercent=n</td><td>启动并发GC循环的（整个）堆占用的百分比。它被GC使用，基于整个堆的占用而不仅仅是一代（例如，G1）触发并发GC循环。值为0表示“做恒定GC循环”。默认值为45。</td></tr><tr><td>-XX:NewRatio=n</td><td>年轻代/老年代大小比例。默认值为2。</td></tr><tr><td>-XX:SurvivorRatio=n</td><td>Eden/幸存空间大小的比例。默认值为8。</td></tr><tr><td>-XX:MaxTenuringThreshold=n</td><td>固定阈值的最大值。默认值为15。</td></tr><tr><td>-XX:ParallelGCThreads=n</td><td>设置在垃圾回收器的并行阶段使用的线程数。默认值随运行JVM的平台而异。</td></tr><tr><td>-XX:ConcGCThreads=n</td><td>并发垃圾回收器将使用的线程数。默认值随运行JVM的平台而异。</td></tr><tr><td>-XX:G1ReservePercent=n</td><td>设置保留为假上限的堆的数量，以减少升级失败的可能性。默认值是10。</td></tr><tr><td>-XX:G1HeapRegionSize=n</td><td>使用G1，Java堆被细分为均匀大小的区域。这设定了各个部分的大小。此参数的默认值根据堆大小的人体工程学确定。最小值为1Mb，最大值为32Mb。</td></tr></tbody></table>

---
# 6.用G1记录GC
### 我们需要解决的最后一个主题是使用日志信息来分析G1回收器的性能。本节简要介绍可用于回收数据的开关以及日志中打印的信息。

## 设置日志详细信息
### 您可以将细节设置为三个不同级别的细节。
### （1）-verbosegc（相当于-XX:+PrintGC）设置日志的细节级别。
### 示例：
    [GC pause (G1 Humongous Allocation) (young) (initial-mark) 24M- >21M(64M), 0.2349730 secs]
	[GC pause (G1 Evacuation Pause) (mixed) 66M->21M(236M), 0.1625268 secs]   
### （2）-XX:+PrintGCDetails将详细级别设置为更精细。选项显示以下信息：
- 每个阶段显示平均，最小和最大时间。
- 根扫描，RSet更新（已处理缓冲区信息），RSet扫描，对象复制，终止（尝试次数）。
- 还显示“其他”时间，例如花费在选择CSet，参考处理，引用排队和释放CSet的时间。
- 显示Eden，幸存区和总堆占用。
### 示例：
	[Ext Root Scanning (ms): Avg: 1.7 Min: 0.0 Max: 3.7 Diff: 3.7]
	[Eden: 818M(818M)->0B(714M) Survivors: 0B->104M Heap: 836M(4096M)->409M(4096M)]
### （3）-XX:+UnlockExperimentalVMOptions -XX:G1LogLevel=最好将细节级别设置为最佳。喜欢更细，但包括个人工作者的线程信息。
### 示例：
	[Ext Root Scanning (ms): 2.1 2.4 2.0 0.0
           Avg: 1.6 Min: 0.0 Max: 2.4 Diff: 2.3]
       [Update RS (ms):  0.4  0.2  0.4  0.0
           Avg: 0.2 Min: 0.0 Max: 0.4 Diff: 0.4]
           [Processed Buffers : 5 1 10 0
           Sum: 16, Avg: 4, Min: 0, Max: 10, Diff: 10]

## 确定时间
### 几个开关决定了GC日志中显示的时间。
### （1）-XX:+PrintGCTimeStamps - 显示自JVM启动以来的经过时间。
### 示例：
	 1.729: [GC pause (young) 46M->35M(1332M), 0.0310029 secs]
### （2）-XX:+PrintGCDateStamps - 为每个条目添加时间前缀。
### 示例：
	2012-05-02T11:16:32.057+0200: [GC pause (young) 46M->35M(1332M), 0.0317225 secs]

## 了解G1日志
### 要了解日志，本节使用实际GC日志输出定义了一些术语。以下示例显示了日志中的输出，以及您将在其中找到的术语和值的解释。
## G1日志术语索引
### 1.Parallel Time并行时间
### 示例：
	414.557: [GC pause (young), 0.03039600 secs] [Parallel Time: 22.9 ms]
	[GC Worker Start (ms): 7096.0 7096.0 7096.1 7096.1 706.1 7096.1 7096.1 7096.1 7096.2 7096.2 7096.2 7096.2
       Avg: 7096.1, Min: 7096.0, Max: 7096.2, Diff: 0.2]
### Parallel Time – 暂停的主要并行部分的总经过时间
### Worker Start – worker开始的时间戳
### 注意：日志在线程ID上排序，并且在每个条目上是一致的

### 2.External Root Scanning扫描外部根部
### 示例：
	[Ext Root Scanning (ms): 3.1 3.4 3.4 3.0 4.2 2.0 3.6 3.2 3.4 7.7 3.7 4.4
     Avg: 3.8, Min: 2.0, Max: 7.7, Diff: 5.7]
### External root scanning - 扫描外部根部所花费的时间（例如，指向堆的系统字典）

### 3.Update Remembered Set更新记忆集
### 示例：
	[Update RS (ms): 0.1 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 Avg: 0.0, Min: 0.0, Max: 0.1, Diff: 0.1]
   	[Processed Buffers : 26 0 0 0 0 0 0 0 0 0 0 0
    Sum: 26, Avg: 2, Min: 0, Max: 26, Diff: 26]
### Update Remembered Set - 必须更新在开始暂停之前完成但尚未被并发优化线程处理的任何缓冲区。时间取决于卡的密度。卡越多，需要的时间越长。

### 4.Scanning Remembered Sets扫描记忆集
### 示例：
	[Scan RS (ms): 0.4 0.2 0.1 0.3 0.0 0.0 0.1 0.2 0.0 0.1 0.0 0.0 Avg: 0.1, Min: 0.0, Max: 0.4, Diff: 0.3]F
### Scanning Remembered Sets - 查找指向回收集的指针。

### 5.Object Copy对象复制
### 示例：
	[Object Copy (ms): 16.7 16.7 16.7 16.9 16.0 18.1 16.5 16.8 16.7 12.3 16.4 15.7 Avg: 16.3, Min: 12.3, Max:  18.1, Diff: 5.8]
### Object copy – 每个线程用于复制和疏散对象的时间。

### 6.Termination Time终止时间
### 示例：
	[Termination (ms): 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
	0.0 Avg: 0.0, Min: 0.0, Max: 0.0, Diff: 0.0] [Termination Attempts : 1 1 1 1 1 1 1 1 1 1 1 1 Sum: 12, Avg: 1, Min: 1, Max: 1, Diff: 0]
### Termination time - 当工作线程完成其特定的一组对象进行复制和扫描时，它将进入终止协议。它寻找工作窃取，一旦完成了这项工作，它再次进入终止协议。终止企图计算所有企图窃取工作

### 7.GC Worker End GC工作结束
### 示例：
	[GC Worker End (ms): 7116.4 7116.3 7116.4 7116.3 7116.4 7116.3 7116.4 7116.4 7116.4 7116.4 7116.3 7116.3
    Avg: 7116.4, Min: 7116.3, Max: 7116.4, Diff:   0.1]
	[GC Worker (ms): 20.4 20.3 20.3 20.2 20.3 20.2 20.2 20.2 20.3 20.2 20.1 20.1
     Avg: 20.2, Min: 20.1, Max: 20.4, Diff: 0.3]
### GC worker end time – 单个GC工作停止时的时间戳。
### GC worker time – 单个GC工作线程花费的时间。

### 8.GC Worker Other GC其他工作
### 示例：
	[GC Worker Other (ms): 2.6 2.6 2.7 2.7 2.7 2.7 2.7 2.8 2.8 2.8 2.8 2.8
    Avg: 2.7, Min: 2.6, Max: 2.8, Diff: 0.2]
### GC worker other – 不能归因于前面列出的工作阶段的时间（对于每个GC线程）。应该相当低。在过去，我们已经看到了过高的价值，它们被归因于JVM其他部分的瓶颈（例如，代码缓存占用的增加）。

### 9.Clear CT
### 示例：
	[Clear CT: 0.6 ms]
### 用于清除RSet扫描元数据的卡表的时间

### 10.Other
### 示例：
	[Other: 6.8 ms]
### GC暂停的各种其他顺序阶段所花费的时间。

### 11.CSet
### 示例：
	[Choose CSet: 0.1 ms]
### 最后确定要回收的区域集合的时间。通常非常小；稍长时必须选择old。

### 12.Ref Proc
### 示例：
	[Ref Proc: 4.4 ms]
### 从GC的前几个阶段延迟处理软、弱等引用的时间。

### 13.Ref Enq
### 示例：
	[Ref Enq: 0.1 ms]
### 在待定列表中放置软、弱等引用的时间。

### 14.Free CSet
### 示例：
	[Free CSet: 2.0 ms]
### 时间花费了刚刚回收的一些区域，包括他们的记忆集。
