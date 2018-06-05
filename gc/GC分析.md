 1、分析gc日志，对应时间点的gc操作，定位是否是gc问题
 ```
   cat gc.log | awk '{print $1 $(NF-1)}' | awk -F '=' '{if($2>1){print;}}'
   grep -C20 '<grep-info>'
 ```

 2、对于gc问题，复现，制作dump，分析具体的线程问题
   1）定位消耗cpu最大进程和改进程内消耗cpu最大的线程，jstack其dump信息：
 ```
   #/bin/bash
   PID=$(top -b -n1 | grep java | sort -n -k9 | tail -n1 | awk '{print $1}')
   echo ${PID}
   top -H -b -n1 -p ${PID} | grep java | sort -n -k9 |tail -n1 > dump.${PID}
   cat dump.${PID}            #打印出最大线程的cpu消耗百分比
   NID=$(printf "nid=0x%x" $(awk '{print $1}' dump.${PID}))
   echo ${NID}
   jstack ${PID} | grep -A500 ${NID} | grep -m1 "^$" -B500
 ```  
    
   2）定位消耗cpu最大进程和改进程内消耗cpu最大的线程，jstack其dump信息（简单版）：
 ```  
  #/bin/bash
  PID=$(top -b -n1 | grep java | sort -n -k9 | tail -n1 | awk '{print $1}')
  NID=$(printf "nid=0x%x" $(top -H -b -n1 -p ${PID} | grep java | sort -n -k9 |tail -n1 | awk '{print $1}'))
  echo ${PID}:${NID}
  jstack ${PID} | grep -A500 ${NID} | grep -m1 "^$" -B500
 ```
 
   3）总体分析：GCEasy （http://gceasy.io/）
        下载日志文件到本地
        打包并upload到gceasy
        分析：
           . 持续总时间：
           . full gc的次数和时间
                                           
 3、查看java进程的内存状态信息jstat:
 ```
    jstat -gc <pid> -h20 1000 10  
    jstat -gcutil <pid> 1000 10
    jstat -gccapacity <pid> 
    jstat -gcnew <pid>
    jstat -gcnewcapacity<pid>
    jstat -gcold <pid>
    jstat -gcoldcapacity <pid>
    jstat -gcpermcapacity<pid>
 ```
 
 4、调优
 
| jvm_args    | desc | 
| ------------- |:-------------:|
| -Xmx4096m | 最大堆大小 |
| -Xms4096m | 初始堆大小 |
| -Xmn2048m | 新生代大小 |
| -XX:CMSInitiatingOccupancyFraction=80 | 使用cms作为垃圾回收，且使用80％后开始CMS收集（可避免promotion failed） |
| -XX:+UseCMSCompactAtFullCollection | 在FULL GC的时候， 对年老代的压缩， 可减少碎片 |
| -XX:+UseCMSInitiatingOccupancyOnly | 使用手动定义初始化定义开始CMS收集（禁止hostspot自行触发CMS GC） |
| -XX:MaxTenuringThreshold=15 | 垃圾最大年龄（如果设置为0的话,则年轻代对象不经过Survivor区,直接进入年老代。对于年老代比较多的应用,可以提高效率.如果将此值设  置为一个较大值,则年轻代对象会在Survivor区进行多次复制,这样可以增加对象再年轻代的存活 时间,增加在年轻代即被回收的概率） |
| -XX:-UseAdaptiveSizePolicy | 自动选择年轻代区大小和相应的Survivor区比例（并行收集器会自动选择年轻代区大小和相应的Survivor区比例,以达到目标系统规定的最低相应时间或者收集频率等） |
| -XX:PermSize=128M | 设置持久代(perm gen)初始值 |
| -XX:MaxPermSize=256M | 设置持久代(perm gen)最大值 |
| -XX:SurvivorRatio=3 | Eden区与Survivor区的大小比值（3  2:3） |
| -XX:+UseParNewGC | 设置年轻代为并行收集(可与CMS收集同时使用) |
| -XX:+UseConcMarkSweepGC | 使用CMS内存收集 |
| -XX:+AlwaysPreTouch | ... |

 5、日志分析：
    . gc time
```
    sys|user|real      (https://blog.gceasy.io/2016/12/08/real-time-greater-than-user-and-sys-time/)          
    user:    gc event在用户模式下的cpu时间总和
    sys:      gc  event在内核模式下(用户空间)的cpu时间总和
    real:     gc event实际执行的时间
        一般  user+sys  >> real  OR  (user+sys)/核数 ≈ real
        #1  io等待
        #2  cpu竞争资源 
```
   . CMS日志：

```
(1) [GC [1 CMS-initial-mark: 1677998K(2097152K)] 1729010K(3774912K), 0.5388790 secs] [Times: user=0.02 sys=0.03, real=0.54 secs]
(2) [CMS-concurrent-mark-start]
(3) [CMS-concurrent-mark: 0.448/0.449 secs] [Times: user=0.90 sys=0.40, real=0.45 secs]
(4) [CMS-concurrent-preclean-start]
(5) [CMS-concurrent-preclean: 0.012/0.012 secs] [Times: user=0.01 sys=0.01, real=0.02 secs]
(6) [CMS-concurrent-abortable-preclean-start]
 CMS: abort preclean due to time 2017-04-27T22:38:22.536+0800: 537936.656: [CMS-concurrent-abortable-preclean: 5.109/5.113 secs] [Times: user=7.39 sys=0.92, real=5.11 secs]
(7) [GC[YG occupancy: 705218 K (1677760 K)]2017-04-27T22:38:22.540+0800: 537936.660: [Rescan (parallel) , 4.1705560 secs]2017-04-27T22:38:26.711+0800: 537940.831: [weak refs processing, 0.1304200 secs]2017-04-27T22:38:26.841+0800: 537940.961: [scrub string table, 0.0006920 secs] [1 CMS-remark: 1677998K(2097152K)] 2383217K(3774912K), 4.3038590 secs] [Times: user=16.67 sys=0.00, real=4.31 secs]
(8) [CMS-concurrent-sweep-start]
(9) [GC2017-04-27T22:38:29.348+0800: 537943.468: [ParNew: 1309376K->61534K(1677760K), 0.0760790 secs] 1474898K->227299K(3774912K), 0.0768230 secs] [Times: user=0.72 sys=0.00, real=0.08 secs]
(10) [CMS-concurrent-sweep: 2.586/2.736 secs] [Times: user=5.34 sys=1.32, real=2.73 secs]
(11) [CMS-concurrent-reset-start]
(12) [CMS-concurrent-reset: 0.066/0.066 secs] [Times: user=0.00 sys=0.00, real=0.07 secs]
 两个SWT阶段：
    . CMS-initial-mark
    . CMS-remark

```
      
