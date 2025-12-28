## 适用配置

多核处理器，大内存机器。G1 致力于 平衡 吞吐量和延迟。

## 特点

1. heap size 可以达到数十GB,甚至更大。
2. heap 中存在 大量碎片
3. 可预测的暂停时间不超过几百毫秒，**避免长时间的垃圾回收暂停**。虽然 G1 收集器的**垃圾回收暂停通常更短**，但应用吞吐量也往往略低。

## 基础理论认知

1. G1 是一种分代式、增量式、并行式、大部分并发式、暂停式且具有内存整理功能的垃圾收集器，它会在每次暂停应用程序执行的阶段监控暂停时间目标。
2. 空间回收工作主要集中在年轻一代，因为这样做效率最高，同时也会偶尔对老一代进行空间回收。
3. G1 以逐步并行的方式进行空间回收。G1 通过跟踪之前应用行为和垃圾回收暂停的信息来实现可预测性，从而构建相关成本的模型。它利用这些信息来评估暂停期间的工作量。例如，G1 优先回收最有效率的区域（也就是那些大部分被垃圾填满的区域，因此得名）
4. **G1 会在系统停止运行期间执行垃圾回收和空间回收。**通常会将存活对象从源区域复制到堆中的一个或多个目标区域，并调整对这些已移动对象的现有引用。年轻一代（伊甸园和幸存者区域）的对象会根据其年龄被复制到幸存者区域或老旧区域。旧区域中的对象被复制到其他旧区域中。对于位于庞大区域内的物体，G1 的处理方式有所不同。G1 只负责判断这些物体是否存活，如果它们不存活，则收回它们所占据的空间。G1 只会在万不得已的情况下，以极其缓慢的速度进行收集，才会移动这些庞大的物体。



## 堆内存布局

![image-20251228165531028](D:\note\LIngcheng-zeng.github.io\content\note\assets\image-20251228165531028.png)

年轻一代包含 eden regions（红色）和 survivor regions（红色带“S”）。

这些区域的功能与其他收集器中相应的连续空间相同，区别在于 G1 中的这些区域在内存中通常以非连续模式排列。

老一代由老区域（浅蓝色）构成。对于跨越多个区域的对象，老一代区域可能非常庞大（浅蓝色带“H”）。

应用程序总是将资源分配到新生代（ eden regions ），但超大型对象除外，它们会被直接分配到老代。



## 垃圾回收周期

![image-20251228170127415](./Garbage%20Collector%20G1.assets/image-20251228170127415-1766912495326-3.png)

1. Young-only phase: This phase starts with a few Normal young collections that promote objects into the old generation. The transition between the young-only phase and the space-reclamation phase starts when the old generation occupancy reaches a certain threshold, the Initiating Heap Occupancy threshold. At this time, G1 schedules a Concurrent Start young collection instead of a Normal young collection.

2. Space-reclamation phase: This phase consists of multiple young collections that in addition to young generation regions, also evacuate live objects of sets of old generation regions. These collections are also called Mixed collections. The space-reclamation phase ends when G1 determines that evacuating more old generation regions wouldn't yield enough free space worth the effort.

3. After space-reclamation, the collection cycle restarts with another young-only phase. As backup, if the application runs out of memory while gathering liveness information, G1 performs an in-place stop-the-world full heap compaction (Full GC) like other collectors.

   

   

   