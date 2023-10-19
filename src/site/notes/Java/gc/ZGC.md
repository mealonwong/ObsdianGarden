---
{"dg-publish":true,"permalink":"/java/gc/zgc/"}
---


## 概述

ZGC（Z Garbage Collector），在 jdk 11 中引入的一种可扩展的低延迟垃圾收集器，在 jdk 15 中发布稳定版。

设计目标：

-   max gc pause time（最大停顿时间） < 1ms（JDK16及以后）。
    
-   停顿时间不会随着堆、或者活跃对象的大小而增加。
    
-   适用内存大小从 8MB 到16TB 的堆。
    

## ZGC特征

ZGC收集器是一款**基于Region内存布局**的，（暂时）**不设分代**的，使用了**读屏障（Load Barriers）**、**染色指针（Reference Coloring）**和**内存多重映射（Multi-Mapping）**等技术来实现**可并发**的标记-整理算法，以低延迟为首要目标的一款垃圾回收器。

## 内存布局

ZGC内存布局与G1一样，均采用了基于Region的堆内存布局，但ZGC的Region具有动态性（动态创建和销毁，以及动态的区域容量大小）。在x64的硬件平台下，ZGC的Region可以具有大、中、小三类容量。

-   小型Region（Small Region）：容量固定为2MB，存放小于256K的对象
    
-   中型Region（Medium Region）：容量固定为32MB，存放大于等于256K但小于4MB的对象
    
-   大型Region（Large Region）：容量不固定，可以动态变化，但容量必须为2MB的整数倍，用于放置4MB或以上的大对象，并且不会被**重分配**。
![image.png](https://pic.imgdb.cn/item/6530fd8dc458853aef4f8565.png)

## 染色指针（Colored Pointer）

Colored Pointer，即染色指针，是ZGC的核心设计之一，如下图所示。以前的垃圾回收器的GC信息都保存在对象头中，而ZGC的GC信息保存在指针中（直接将标记信息记录在对象的引用指针上）。


![image.png](https://pic.imgdb.cn/item/6530fdb0c458853aef4fdb6e.png)

每个对象有一个64位指针，这64位被分为以下：

-   18 bit：预留位，留给以后使用。
    
-   1 bit：Finalizable标识，此位与并发处理有关，它表示这个对象只能通过finalize()才能访问（finalize()：Object基类的一个空方法，如果被重写则会在GC前调用该方法，该方法会且仅会被调用一次）。
    
-   1 bit：Remapped标识，判断应用是否已指向新的地址。
    
-   1 bit：Marked1标识
    
-   1 bit：Marked0标识，与Marked1标识均用来标记可达对象。
    
-   42 bit：使用42位表示对象的地址，引用可寻址2^42=4TB的内存空间。
    

## 内存多重映射

ZGC使用了多重映射技术，将多个不同的虚拟内存地址映射到同一个物理内存地址上，是一种多对一映射。
![image.png](https://pic.imgdb.cn/item/6530fdc8c458853aef5016ab.png)

当应用程序创建对象时，会在堆上申请一个虚拟地址，这时ZGC会为这个对象在M0，M1和Remapped这三个视图空间分别申请一个虚拟地址，这三个虚拟地址映射到同一个物理地址。

M0、M1和Remapped视图都可以映射到实际的物理地址，但在ZGC中，同一时间仅有一个视图空间有效。由ZGC并发过程演示可以知道，这三个视图空间的切换是由垃圾回收的不同阶段触发的，通过限定三个空间在同一时间仅有一个空间有效，利用虚拟空间换时间，高效地完成GC过程中的并发操作。

## 读屏障(Load Barriers)

ZGC为了解决内存碎片化的问题引入了Relocation，但对于一个很大的堆来说，Relocation的过程相当缓慢，因为ZGC并不希望有较大的延时，因此会将大多数的Relocation过程与应用程序并发执行。但是这就引入了另外一个问题，例如当前有一个线程的引用，然后ZGC Relocation了这个对象的引用，紧接着发生了线程的上下文切换，用户线程尝试获取这个对象的旧内存地址，那么此时肯定是无法获取到的。

ZGC引入了读屏障（Load Barriers）来解决这个问题，Load Barriers是线程从堆中获取一个对象引用时加入的一小段代码（仅“从堆中读取对象引用”才会触发这段代码）。

读屏障作用：在程序需要并行获取对象的引用时，ZGC就会对该对象的指针进行读取，判断Remapped标识，如果标识为该对象位于本次需要清理的Region区中，该对象则会有内存地址变化，会在指针中将新的引用地址替换掉原有对象的引用地址，然后再进行返回。

```
Object O = obj.fieldA;      //从堆中获取引用，需要加入读屏障
<load barriers needed here>
Object p = o;               //无需加入读屏障，因为不是从堆中读取引用
o.doSomething();						//无需加入读屏障，因为不是从堆中读取引用
int i = obj.fieldB;         //无需加入读屏障，因为不是对象引用
```

## ZGC工作原理

下图展示了ZGC三个暂停阶段（STW）：**Pause Mark Start(初始标记)、Pause Mark End（再次标记）、Pause Relocate Start（初始转移）**。

**Pause Mark Start(初始标记)和Pause Relocate Start（初始转移）**都只需要扫描所有GC Roots，其处理时间和GC Roots的数量是成正比的，一般耗时较短，**Pause Mark End（再次标记）**阶段重新标记并发标记阶段发生变化的对象，还会对非强引用（软引用，虚引用等）进行并行标记，该阶段标记的对象少，耗时很短，超过1ms则再次进入并发标记阶段。
![image.png](https://pic.imgdb.cn/item/6530fddfc458853aef50522c.png)

ZGC的主要运作过程可以分为以下四个阶段

-   **并发标记（Concurrent Mark）:**与G1一样，并发标记阶段是遍历对象图做可达性分析的阶段，前后也需要经过类似于G1的初始标记、最终标记的短暂停顿，而且这些停顿阶段所做的事情在目标上也是相类似的。与G1和Shenandoah不同的是，ZGC的标记是在指针上而不是在对象上进行的，标记计算会更新染色指针中的Marked0、Marked1标志位。
    
-   **并发预备重分配（Concurrent Prepare for Relocate）**：这个阶段需要根据特定的查询条件统计出本次收集过程中要清理哪些Region，并将这些Region组成重分配集（Relocation Set）。ZGC每次回收都会扫描所有的Region，且ZGC的重分配集只是决定了里面的存活对象会被重新复制到其他Region中，里面的Region会被释放，而并不能说回收行为就只是针对这个集合里的Region进行，因为标记过程是针对全堆的。
    
-   **并发重分配（Concurrent Relocate）**：重分配是ZGC执行过程中的核心阶段，这个过程要把重分配集中的存活对象复制到新的Region上，并未重分配集中的每个Region维护一个转发表（Forward Table），记录从就对象到新对象的转向关系。得益于染色指针的支持，ZGC收集器能即从引用上就明确得知一个对象是否处于重分配及中，如果用户线程此时并发访问了位于重分配集中的对象，这次访问将会被预置的内存屏障所截获，然后立即根据Region上的转发表记录将访问转发到复制的新对象上，并同时修正更新改引用的值，使其直接指向新对象，ZGC将这种行为称为指针的“自愈”（Self-Healing）能力。
    
-   **并发重映射（Concurrent Remap）**：重映射做的就是修正整个队中指向重分配集中就对象的所有引用，但对于ZGC来说并发重映射并不是很迫切的任务，因为上面提到过ZGC拥有指针的“自愈”能力，即使是旧引用，最多只是第一次使用时需要多一次转发和修正工作。因此ZGC将并发重映射要做的工作合并到下一次垃圾回收循环中的并发标记阶段里去执行了，因为它们都需要遍历所有对象，合并后还可以节省一次遍历所有对象的开销。一旦所有指针都被修正之后，原来记录新旧对象关系的转发表就可以释放掉了。
    

## ZGC并发处理演示

-   **初始化**：ZGC初始化之后，整个内存空间的地址视图被置为Remapped。程序正常运行，在内存中分配对象，满足一定条件后垃圾回收启动，此时进入标记阶段。
    
-   **并发标记阶段：**第一次进入并发标记阶段，如果对象被GC标记线程或应用线程访问过，则代表是活跃对象，那就将该对象的地址视图从Remapped调整为Marked0。因此在标记阶段结束后，对象的地址要么是Marked0，要么是Remapped，如果是Marked0则代表是活跃对象，否则就是非活跃对象。
    
-   **并发转移阶段：**标记结束后进行转移阶段，此时地址视图被重置为Remapped。如果对象被GC转移线程或者应用线程访问过，那么就将对象的地址视图从Marked0调整为Remapped。
    

![](https://pic.imgdb.cn/item/6530fe1bc458853aef50f17e.png)

上面提到过并发重映射是合并在下一次标记阶段的，因此在标记阶段其实存在两个地址视图M0和M1，之所以存在两个地址视图，就是为了区别前一次标记和当前标记，也就是说，当第二次进入到并发标记阶段后，地址视图调整为M1，而非M0。

第二次GC时M0的视图的对象代表的是上一次GC中被标记为活跃的对象，但并未被转移，且本次GC过程中被未被标记为活跃的对象。M1视图的对象代表的是本次GC过程中被标记为活跃的对象。Remapped视图是上次垃圾回收发生转移或者被Java应用程序访问过的对象，本次垃圾回收中被未被标记为活跃的对象。

## ZGC核心参数

参数

使用样例

说明

-XX:+UseZGC

-XX:+UnlockExperimentalVMOptions -XX:+UseZGC

启用ZGC

-Xmx

-Xms10G -Xmx10G

设置最大堆内存

-Xlog:gc

打印GC日志

-Xlog:gc*

打印GC详细日志

-XX:ReservedCodeCacheSize -XX:InitialCodeCacheSize

-XX:ReservedCodeCacheSize=64m

-XX:InitialCodeCacheSize=128m

设置CodeCache的大小， JIT编译的代码都放在CodeCache中，一般服务64m或128m就已经足够

-XX:ConcGCThreads

-XX:ConcGCThreads=2

并发垃圾回收时线程数，默认为总核数的12.5%

-XX:ParallelGCThreads

-XX:ParallelGCThreads=6

STW阶段使用的线程数，默认是总核数的60%

-XX:ZCollectionInterval

-XX:ZCollectionInterval=120

ZGC发生的最小时间间隔，单位秒

-XX:ZAllocationSpikeTolerance

-XX:ZAllocationSpikeTolerance=5

ZGC触发自适应算法的修正系数，默认为2，数值越大越早触发ZGC

-XX:+UnlockDiagnosticVMOptions

-XX:-ZProactive

-XX:+UnlockDiagnosticVMOptions

-XX:-ZProactive

是否启用主动回收

## 触发时机

-   **定时触发：**默认为不使用，可以通过ZCollectionInterval参数配置。GC日志中的关键字"Timer"。
    
-   **预热触发：**最多三次，在堆内存空间达到10%、20%、30%时机触发，主要是通过GC的时间为其他的GC触发做准备。GC日志关键字"Warmup"。
    
-   **分配速率：**基于正态分布统计，计算内存99%可能的最大分配速率，以及此速率下内存将要耗尽的时间点，在耗尽之前触发GC（耗尽时间，一次GC最大持续时间-一次GC检测周期时间）。GC日志关键字"Allocation Rate"。
    
-   **主动触发：**默认开启，可以通过ZProactictive参数配置，距上一次GC堆内存增长10%，超过5分钟时，对比上次GC的间隔时间限（一次GC最大持续时间），超过则触发。GC日志关键字"Proactive"。
    
-   **元数据分配触发：**元数据区不足导致，GC日志中的关键字"Metadata GC Threshold"。
    
-   **直接触发：**代码中显示调用System.gc()触发，GC日志关键字是"System.gc()"。
    
-   **阻塞内存分配请求触发：**垃圾对象来不及回收，占满整个堆空间，导致部分线程阻塞，GC日志关键字是"Allocation Stall"。
    

## ZGC日志分析

```
public class Main {
    /**
     * VM options:-XX:+UseZGC -Xmx16m -Xlog:gc*
     * @param args
     */
    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<>();
        while (true) {
            list.add(new byte[2048]);
        }
    }
}
```

GC日志中每一行都注明了GC过程中的信息，关键信息如下：

-   **Start**：开始GC，并标明的GC触发的原因。上图中触发原因是自适应算法。
    
-   **Phase-Pause Mark Start**：初始标记，会STW。
    
-   **Phase-Pause Mark End**：再次标记，会STW。
    
-   **Phase-Pause Relocate Start**：初始转移，会STW。
    
-   **Heap信息**：记录了GC过程中Mark、Relocate前后的堆大小变化状况。High和Low记录了其中的最大值和最小值，我们一般关注High中Used的值，如果达到100%，在GC过程中一定存在内存分配不足的情况，需要调整GC的触发时机，更早或者更快地进行GC。
    
-   **GC信息统计**：可以定时的打印垃圾收集信息，观察10秒内、10分钟内、10个小时内，从启动到现在的所有统计信息。利用这些统计信息，可以排查定位一些异常点。
    

## 总结

内存多重映射和染色指针的引入，使 ZGC 的并发性能大幅度提升。

ZGC 只有 3 个需要 STW 的阶段，其中初始标记和初始转移只需要扫描所有 GC Roots，STW 时间 GC Roots 的数量成正比，不会耗费太多时间。再标记过程主要处理并发标记引用地址发生变化的对象，这些对象数量比较少，耗时非常短。可见整个 ZGC 的 STW 时间几乎只跟 GC Roots 数量有关系，不会随着堆大小和对象数量的变化而变化。

ZGC 也有一个缺点，就是浮动垃圾。因为 ZGC 没有分代概念，虽然 ZGC 的 STW 时间在 1ms 以内，但是 ZGC 的整个执行过程耗时还是挺长的。在这个过程中 Java 线程可能会创建大量的新对象，这些对象会成为浮动垃圾，只能等下次 GC 的时候进行回收。

## 参考文献

[新一代垃圾回收器ZGC的探索与实践](https://km.sankuai.com/page/394182506)

[ZGC垃圾收集器](https://www.51cto.com/article/706899.html)

[ZGC原理与实现分析](https://www.jianshu.com/p/4e4fd0dd5d25)

[mmap原理简介](https://juejin.cn/post/6956031662916534279)

[https://www.usenix.org/legacy/events/vee05/full_papers/p46-click.pdf](https://www.usenix.org/legacy/events/vee05/full_papers/p46-click.pdf)