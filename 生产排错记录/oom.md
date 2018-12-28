
### 原因
- 确实不够
- 内存
### 方式
- jmap -heap pid
- jmap -histo:pid 10765 | more
查看内存最大对象，如果出现异常实例，则
jmap -dump:format=b,file=name pid。倒成dump文件，使用工具分析（最原始的jhat,visualVm）


### 总结
 java.lang.OutOfMemoryError:Java heap space
  - 确实不够->xmx:更大
  - 内存溢出->dump->分析工具查看是否有持续增长的对象
  
 java.lang.OutOfMemoryError:Permgen space
持久代所在区域的内存已被耗尽（方法区）
 - 初始化出现，例如开发环境->-XX:MaxPermSize=增大
 - 运行时，一般来说，也是增大参数。除非能定位到类加载器（只有类加载器被卸载，方法区重的类才可能被卸载）
 
 java.lang.OutOfMemoryError:Metaspace
 - 1.8以后Permgen->Metaspace，一般来说也是增大参数：-XX：MaxMetaspaceSize = 512m
 
 java.lang.OutOfMemoryError:Unable to create new native thread
 Java应用程序已达到其可以启动线程数量的极限了
 
 - 通过工具分析，是真的不够还是持续增加，如果是真的不够。ulimit -u 
 - 用线程池