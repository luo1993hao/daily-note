### jps
进程状况


#### 命令格式
jps -
- q 只输出id
- m 传递给main的参数
- l 主类全名，如果是jar包，输出路径
- v 启动jvm参数

### jstat
 显示进程中的类加载，内存，垃圾收集，jit编译数据
 
 #### 命令格式
jstat[option vmid[interval[s|ms][count]]]
参数interval和count代表查询间隔和次数，如果省略这两个参数，说明只查询一次。 
example
```
jstat -gc id time count
```
 - class 监视类装载，卸载数量，总空间以及装载时间
```
Loaded  Bytes  Unloaded  Bytes     Time
 19410 37277.7      421   606.6      19.28
```
- gc 汇总信息，包括eden区，2个survivor区，老年代，永久代（1.8摸得了），等信息
```
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
2560.0 2560.0  0.0   1152.2 1355264.0 117364.5  795136.0   737472.1  127488.0 118775.7 14848.0 13410.9  42319  308.689   7      1.910  310.598
```
- gccapacity 与-gc基本相同，但是主要关注堆每个区域使用到的最大，最小空间
```
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
 84992.0 1360384.0 1360384.0 2048.0 2048.0 1356288.0   171008.0  2721280.0   795136.0   795136.0      0.0 1161216.0 127488.0      0.0 1048576.0  14848.0  42328     7
```
- gcutil 与-gc基本相同，使用百分比
```
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00  23.44  17.36  92.88  93.17  90.32  42335  308.824     7    1.910  310.733
```
- gccause 与-gc一样，但是会额外输出导致上一次gc的原因
```
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
  0.00  68.75  97.48  92.91  93.17  90.32  42349  308.949     7    1.910  310.859 Allocation Failure   No GC
```
- gcnew 监控新生产gc状况
```
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
1024.0 1024.0  480.0    0.0 15  15 1024.0 1358336.0 704824.9  42358  309.032
```
- gcnewcapacity -gcnew基本相同，关注新生代每个区域使用的最大，最小空间
- gcold/gcoldcapacity
### jmap/jhat
 - jmap
   - -dump:[live/all]format=b,file=<filename>:生成heap dump,配合jhat分析。
   - heap：打印jvm heap的情况
   --histo：打印jvm heap的直方图。其输出信息包括类名，对象数量，对象占用大小。
### jsrack
线程跟踪工具，用于打印指定Java进程的线程堆栈信息。一般用于解决cpu过高问题
### 问题以及回答
1. 什么情况下执行fgc
  - old空间不足（20%），perm空间不足
#### cpu过高
  1. top查看cpu状况
  2. top -H -p(-H指定线程 -p指定进程) pid 得到线程编号
  3. 执行 printf "%x\n"  线程编号 得到十六进制地址
  4. jstack pid |grep 十六进制地址 或者jstack -l 线程编号 > tempfile.txt
  5. 一半来说就会定位到代码行