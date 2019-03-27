#### java内存区域
- 方法区
- jvm堆
- 程序计数器
- 虚拟机栈
- 本地方法区
#### java内存模型
- 抽象概念，描述一组规范
- 主内存

### jvm参数说明
1. -xmx 堆最大值
2. -Xms 堆初始值
3. -XX:NewRatio=[ratio] 指定年轻一代的大小相对于老一代的大小
4. -Xmn[memoryVal] 直接指定年轻一代大小= -XX:NewSize and -XX:MaxNewSize 相同值
5. If your heap is larger than 4GB you should always use G1
6. -Xss=[memoryValue] 分配给新堆栈的内存
7. -XX:PermSize=[memoryValue] and -XX:MaxPermSize=[memoryValue] 非堆区（永久代），过小会导致 Java.lang.OutOfMemoryError: PermGen space
8. -XX:SurvivorRatio=8:表示新生代的eden区:from区：to区:8:1:1

  ```$xslt
https://github.com/FoxxMD/intellij-jvm-options-explained
```

### gc算法
#### g1
- G1 GC由Young Generation和Old Generation组成。G1将Java堆空间分割成了若干个Region，即年轻代/老年代是一系列Region的集合，这就意味着在分配空间时不需要一个连续的内存区间，即不需要在JVM启动时决定哪些Region属于老年代，哪些属于年轻代
- G1年轻代收集器是并行Stop-the-world收集器，和其他的HotSpot GC一样，当一个年轻代GC发生时，整个年轻代被回收。G1的老年代收集器有所不同，它在老年代不需要整个老年代回收，只有一部分Region被调用。