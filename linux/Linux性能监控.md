## 一、影响Linux服务器性能的因素 
1. 操作系统级
 
Ø       CPU
Ø       内存
Ø       磁盘I/O带宽
Ø       网络I/O带宽
 
2.        程序应用级
 
## 二、系统性能评估标准
                                     		     评判标准
影响性能因素       好                                   坏                                糟糕
CPU           user% + sys%< 70%                    user% + sys%= 85%              user% + sys% >=90%

内存          Swap In（si）＝0 Swap Out（so）＝0    Per CPU with 10 page/s          More Swap In & Swap Out

磁盘          iowait % < 20%                       iowait % =35%                  iowait % >= 50%

 
其中：
       %user：表示CPU处在用户模式下的时间百分比。
       %sys：表示CPU处在系统模式下的时间百分比。
       %iowait：表示CPU等待输入输出完成时间的百分比。
       swap in：即si，表示虚拟内存的页导入，即从SWAP DISK交换到RAM
       swap out：即so，表示虚拟内存的页导出，即从RAM交换到SWAP DISK。
 
## 三、系统性能分析工具
 
1.常用系统命令 
Vmstat、sar、iostat、netstat、free、ps、top等
 
2.常用组合方式 
•           用vmstat、sar、iostat检测是否是CPU瓶颈
•           用free、vmstat检测是否是内存瓶颈
•           用iostat检测是否是磁盘I/O瓶颈
•           用netstat检测是否是网络带宽瓶颈
 
## 四、Linux性能评估与优化
 
1. 系统整体性能评估（uptime命令）
 
[root@server ~]# uptime
16:38:00 up 118 days,  3:01,  5 users,  load average: 1.22, 1.02, 0.91
这里需要注意的是：load average这个输出值，这三个值的大小一般不能大于系统CPU的个数，例如，本输出中系统有8个CPU,如果load average的三个值长期大于8时，说明CPU很繁忙，负载很高，可能会影响系统性能，但是偶尔大于8时，倒不用担心，一般不会影响系统性能。相反，如果load average的输出值小于CPU的个数，则表示CPU还有空闲的时间片，比如本例中的输出，CPU是非常空闲的。
 
2. CPU性能评估
 
（1）利用vmstat命令监控系统CPU
   该命令可以显示关于系统各种资源之间相关性能的简要信息，这里我们主要用它来看CPU一个负载情况。
   下面是vmstat命令在某个系统的输出结果：
 
[root@server ~]# vmstat 2 3
procs -----------memory----------  ---swap--  -----io---- --system--  -----cpu------
 r  b   swpd   free      buff  cache   si   so    bi    bo       in     cs     us sy  id   wa st
 0  0    0    162240   8304  67032   0    0    13    21   1007   23     0  1   98   0   0
 0  0    0    162240   8304  67032   0    0     1     0     1010   20     0  1   100 0   0
 0  0    0    162240   8304  67032   0    0     1     1     1009   18     0  1    99  0   0

  Procs
     r列表示运行和等待cpu时间片的进程数，这个值如果长期大于系统CPU的个数，说明CPU不足，需要增加CPU。
     b列表示在等待资源的进程数，比如正在等待I/O、或者内存交换等。
  CPU
    us列显示了用户进程消耗的CPU 时间百分比。us的值比较高时，说明用户进程消耗的cpu时间多，但是如果长期大于50%，就需要考虑优化程序或算法。
     sy列显示了内核进程消耗的CPU时间百分比。Sy的值较高时，说明内核消耗的CPU资源很多。
    根据经验，us+sy的参考值为80%，如果us+sy大于 80%说明可能存在CPU资源不足。
 
（2）利用sar命令监控系统CPU
 
sar功能很强大，可以对系统的每个方面进行单独的统计，但是使用sar命令会增加系统开销，不过这些开销是可以评估的，对系统的统计结果不会有很大影响。
 下面是sar命令对某个系统的CPU统计输出：
[root@server ~]# sar -u 3 5
Linux 2.6.9-42.ELsmp (webserver)        11/28/2008      _i686_  (8 CPU)
11:41:24 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
11:41:27 AM     all      0.88      0.00      0.29      0.00      0.00     98.83
11:41:30 AM     all      0.13      0.00      0.17      0.21      0.00     99.50
11:41:33 AM     all      0.04      0.00      0.04      0.00      0.00     99.92
11:41:36 AM     all      90.08     0.00      0.13      0.16      0.00     9.63
11:41:39 AM     all      0.38      0.00      0.17      0.04      0.00     99.41
Average:        all      0.34      0.00      0.16      0.05      0.00     99.45
                           
对上面每项的输出解释如下：
l        %user列显示了用户进程消耗的CPU 时间百分比。
l        %nice列显示了运行正常进程所消耗的CPU 时间百分比。
l        %system列显示了系统进程消耗的CPU时间百分比。
l        %iowait列显示了IO等待所占用的CPU时间百分比
l        %steal列显示了在内存相对紧张的环境下pagein强制对不同的页面进行的steal操作 。
l        %idle列显示了CPU处在空闲状态的时间百分比。
 
问题
1.你是否遇到过系统CPU整体利用率不高，而应用缓慢的现象？
       在一个多CPU的系统中，如果程序使用了单线程，会出现这么一个现象，CPU的整体使用率不高，但是系统应用却响应缓慢，这可能是由于程序使用单线程的原因，单线程只使用一个CPU，导致这个CPU占用率为100%，无法处理其它请求，而其它的CPU却闲置，这就导致了整体CPU使用率不高，而应用缓慢现象的发生。
 
3. 内存性能评估
（1）利用free指令监控内存
free是监控linux内存使用状况最常用的指令，看下面的一个输出：
[root@server ~]# free  -m
                total         used       free     shared    buffers     cached
Mem:       8111       7185        926          0        243           6299
-/+ buffers/cache:     643       7468
Swap:       8189          0         8189
     一般有这样一个经验公式：应用程序可用内存/系统物理内存>70%时，表示系统内存资源非常充足，不影响系统性能，应用程序可用内存/系统物理内存<20%时，表示系统内存资源紧缺，需要增加系统内存，20%<应用程序可用内存/系统物理内存<70%时，表示系统内存资源基本能满足应用需求，暂时不影响系统性能。
3.内存性能评估
 
（1）利用free指令监控内存
free是监控linux内存使用状况最常用的指令，看下面的一个输出：
[root@server ~]# free  -m
                total         used       free     shared    buffers     cached
Mem:       8111       7185        926          0        243           6299
-/+ buffers/cache:     643       7468
Swap:       8189          0         8189
     一般有这样一个经验公式：应用程序可用内存/系统物理内存>70%时，表示系统内存资源非常充足，不影响系统性能，应用程序可用内存/系统物理内存<20%时，表示系统内存资源紧缺，需要增加系统内存，20%<应用程序可用内存/系统物理内存<70%时，表示系统内存资源基本能满足应用需求，暂时不影响系统性能。
  
（2）利用vmstat命令监控内存
 
[root@server ~]# vmstat 2 3
procs -----------memory----------  ---swap--  -----io---- --system--  -----cpu------
 r  b   swpd   free      buff  cache   si   so    bi    bo       in     cs     us sy  id  wa st
 0  0    0    162240   8304  67032   0    0    13    21   1007   23     0  1  98   0  0
 0  0    0    162240   8304  67032   0    0     1     0     1010   20     0  1  100 0  0
 0  0    0    162240   8304  67032   0    0     1     1     1009   18     0  1  99   0  0
l        memory
         swpd列表示切换到内存交换区的内存数量（以k为单位）。如果swpd的值不为0，或者比较大，只要si、so的值长期为0，这种情况下一般不用担心，不会影响系统性能。
         free列表示当前空闲的物理内存数量（以k为单位）
         buff列表示buffers cache的内存数量，一般对块设备的读写才需要缓冲。
         cache列表示page cached的内存数量，一般作为文件系统cached，频繁访问的文件都会被cached，如果cache值较大，说明cached的文件数较多，如果此时IO中bi比较小，说明文件系统效率比较好。
l        swap
si列表示由磁盘调入内存，也就是内存进入内存交换区的数量。
so列表示由内存调入磁盘，也就是内存交换区进入内存的数量。
一般情况下，si、so的值都为0，如果si、so的值长期不为0，则表示系统内存不足。需要增加系统内存。
 
4.磁盘I/O性能评估 
（1）磁盘存储基础
l             熟悉RAID存储方式，可以根据应用的不同，选择不同的RAID方式。
l             尽可能用内存的读写代替直接磁盘I/O，使频繁访问的文件或数据放入内存中进行操作处理，因为内存读写操作比直接磁盘读写的效率要高千倍。
l             将经常进行读写的文件与长期不变的文件独立出来，分别放置到不同的磁盘设备上。
l              对于写操作频繁的数据，可以考虑使用裸设备代替文件系统。
        
       使用裸设备的优点有：
ü           数据可以直接读写，不需要经过操作系统级的缓存，节省了内存资源，避免了内存资源争用。
ü           避免了文件系统级的维护开销，比如文件系统需要维护超级块、I-node等。
ü           避免了操作系统的cache预读功能，减少了I/O请求。
       使用裸设备的缺点是：
ü            数据管理、空间管理不灵活，需要很专业的人来操作。
 
（2）利用iostat评估磁盘性能
[root@server ~]#   iostat -d 2 3
Linux 2.6.9-42.ELsmp (webserver)        12/01/2008      _i686_  (8 CPU)
 
Device:         tps   Blk_read/s   Blk_wrtn/s   Blk_read      Blk_wrtn
sda               1.87         2.58       114.12        6479462     286537372
 
Device:         tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
sda               0.00         0.00         0.00              0                0
 
Device:         tps   Blk_read/s   Blk_wrtn/s   Blk_read    Blk_wrtn
sda               1.00         0.00        12.00             0                24
对上面每项的输出解释如下：
Blk_read/s表示每秒读取的数据块数。
Blk_wrtn/s表示每秒写入的数据块数。
Blk_read表示读取的所有块数。
Blk_wrtn表示写入的所有块数。
Ø            可以通过Blk_read/s和Blk_wrtn/s的值对磁盘的读写性能有一个基本的了解，如果Blk_wrtn/s值很大，表示磁盘的写操作很频繁，可以考虑优化磁盘或者优化程序，如果Blk_read/s值很大，表示磁盘直接读取操作很多，可以将读取的数据放入内存中进行操作。
Ø            对于这两个选项的值没有一个固定的大小，根据系统应用的不同，会有不同的值，但是有一个规则还是可以遵循的：长期的、超大的数据读写，肯定是不正常的，这种情况一定会影响系统性能。
 
 
（3）利用sar评估磁盘性能
         通过“sar –d”组合，可以对系统的磁盘IO做一个基本的统计，请看下面的一个输出：
[root@server ~]# sar -d 2 3
Linux 2.6.9-42.ELsmp (webserver)        11/30/2008      _i686_  (8 CPU)
 
11:09:33 PM  DEV     tps   rd_sec/s   wr_sec/s  avgrq-sz  avgqu-sz   await  svctm   %util
11:09:35 PM dev8-0  0.00  0.00            0.00        0.00          0.00         0.00   0.00     0.00
 
11:09:35 PM  DEV     tps  rd_sec/s    wr_sec/s  avgrq-sz  avgqu-sz  await   svctm   %util
11:09:37 PM dev8-0  1.00  0.00         12.00        12.00         0.00        0.00    0.00     0.00
 
11:09:37 PM   DEV    tps    rd_sec/s  wr_sec/s   avgrq-sz  avgqu-sz  await  svctm   %util
11:09:39 PM dev8-0  1.99   0.00         47.76         24.00       0.00        0.50    0.25     0.05
 
Average:  DEV          tps    rd_sec/s   wr_sec/s  avgrq-sz  avgqu-sz    await  svctm   %util
Average:  dev8-0      1.00   0.00          19.97         20.00       0.00         0.33    0.17     0.02
      需要关注的几个参数含义：
     await表示平均每次设备I/O操作的等待时间（以毫秒为单位）。
     svctm表示平均每次设备I/O操作的服务时间（以毫秒为单位）。
     %util表示一秒中有百分之几的时间用于I/O操作。
 
 
对以磁盘IO性能，一般有如下评判标准：
     正常情况下svctm应该是小于await值的，而svctm的大小和磁盘性能有关，CPU、内存的负荷也会对svctm值造成影响，过多的请求也会间接的导致svctm值的增加。
     await值的大小一般取决与svctm的值和I/O队列长度以及I/O请求模式，如果svctm的值与await很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，系统上运行的应用程序将变慢，此时可以通过更换更快的硬盘来解决问题。
     %util项的值也是衡量磁盘I/O的一个重要指标，如果%util接近100%，表示磁盘产生的I/O请求太多，I/O系统已经满负荷的在工作，该磁盘可能存在瓶颈。长期下去，势必影响系统的性能，可以通过优化程序或者通过更换更高、更快的磁盘来解决此问题。
 
5. 网络性能评估
 
（1）通过ping命令检测网络的连通性
（2）通过netstat –i组合检测网络接口状况
（3）通过netstat –r组合检测系统的路由表信息
（4）通过sar –n组合显示系统的网络运行状态
 
## 五、Oracle在Linux下的性能优化
 
Oracle数据库内存参数的优化
Ø       与oracle相关的系统内核参数
Ø       SGA、PGA参数设置
 
Oracle下磁盘存储性能优化
Ø       文件系统的选择（ext2/ext3、xfs、ocfs2）
Ø       Oracle  ASM存储
 1.优化oracle性能参数之前要了解的情况
1）物理内存有多大
2）操作系统估计要使用多大内存
3）数据库是使用文件系统还是裸设备
4）有多少并发连接
5）应用是OLTP类型还是OLAP类型
 
2.oracle数据库内存参数的优化
 
（1）系统内核参数
修改 /etc/sysctl.conf 这个文件，加入以下的语句：
kernel.shmmax = 2147483648
kernel.shmmni = 4096
kernel.shmall = 2097152
kernel.sem = 250 32000 100 128
fs.file-max = 65536
net.ipv4.ip_local_port_range = 1024 65000
参数依次为：
Kernel.shmmax:共享内存段的最大尺寸（以字节为单位）。
Kernel.shmmni：系统中共享内存段的最大数量。
Kernel.shmall：共享内存总量，以页为单位。
fs.file-max：文件句柄数，表示在Linux系统中可以打开的文件数量。
net.ipv4.ip_local_port_range：应用程序可使用的IPv4端口范围。
 
需要注意的几个问题
关于Kernel.shmmax
     Oracle SGA 由共享内存组成，如果错误设置 SHMMAX可能会限制SGA 的大小，SHMMAX设置不足可能会导致以下问题：ORA-27123:unable to attach to shared memory segment，如果该参数设置小于Oracle SGA设置，那么SGA就会被分配多个共享内存段。这在繁忙的系统中可能成为性能负担，带来系统问题。
     Oracle建议Kernel.shmmax最好大于sga，以让oracle共享内存区SGA在一个共享内存段中，从而提高性能。
关于Kernel.shmall
     表示系统共享内存总大小，以页为单位。
     一个32位的Linux系统，8G的内存，可以设置kernel.shmall = 2097152，即为： 2097152*4k/1024/1024 = 8G就是说可用共享内存一共8G，这里的4K是32位操作系统一页的大小，即4096字节。
关于Kernel.shmmni
     表示系统中共享内存段的最大数量。系统默认是4096，一般无需修改，在SUN OS下还有Kernel.shmmin参数，表示共享内存段最小尺寸，勿要混肴！ 
（2）SGA、PAG参数的设置
 
A Oracle在内存管理方面的改进
     Oracle 9i通过参数PGA_AGGREGATE_TARGET参数实现PGA自动管理  Oracle 10g通过参数SGA_TARGET参数实现了SGA的自动管理，
     Oracle 11g实现了数据库所有内存块的全自动化管理，使得动态管理SGA和PGA成为现实。
 
自动内存管理的两个参数：
     MEMORY_TARGET：表示整个ORACLE实例所能使用的内存大小，包括PGA和SGA的整体大小，即这个参数是动态的，可以动态控制SGA和PGA的大小。
     MEMORY_MAX_TARGET：这个参数定义了MEMORY_TARGET最大可以达到而不用重启实例的值，如果没有设置MEMORY_MAX_TARGET值，默认等于MEMORY_TARGET的值。
     使用动态内存管理时，SGA_TARGET和PGA_AGGREGATE_TARGET代表它们各自内存区域的最小设置，要让Oracle完全控制内存管理，这两个参数应该设置为0。
B Oracle五种内存管理方式
Ø         自动内存管理,即AMM (Automatic Memory Management）
Ø         自动共享内存管理，即ASMM（Automatic Shared Memory Management）
Ø         手动共享内存管理
Ø         自动PGA管理
Ø         手动PGA管理
 
自动内存管理（AMM）
默认安装oracle11g的实例就是AMM方式。通过如下查看：
示例如下：
SQL> show parameters target 
NAME                                       TYPE                  VALUE
------------ ---------------------      ------------------    ---------------------- archive_lag_target                     integer                      0
db_flashback_retention_target   integer                    1860
fast_start_io_target                    integer                      0
fast_start_mttr_target                 integer                      0
memory_max_target                  big integer              1400M
memory_target                          big integer              1400M
pga_aggregate_target                big integer                0
sga_target                                  big integer                0
注意：如果初始化参数 LOCK_SGA ＝ true ，则 AMM 是不可用的。
自动共享内存管理
自动共享内存管理是oracle10g引进的，如果要使用自动共享内存管理，只需设置MEMORY_TARGET=0，然后显式指定SGA_TARGET即可。
示例如下：
SQL> alter system set memory_target=0 scope=both;
System altered.
SQL> alter system set sga_target=1024m scope=both;
System altered.
SQL>
 
手工共享内存管理
Oracle9i以及以前版本，只能手工设置共享内存管理，如果要使用手动共享内存管理，首先需要设置SGA_TARGET 与 MEMORY_TARGET为0。
SGA包含主要参数有：
share_pool_size：共享池大小，建议300-500M之间。
Log_buffer：日志缓冲区大小，建议1-3M之间。
Large_pool_size：大缓冲池大小，非MTS系统，建议在20-30M之间。
Java_pool_size：java池大小，没有java应用时，建议10-20M之间。
db_cache_size：数据缓冲区大小，根据可使用内存大小，尽可能大。
 
自动PAG管理
Oracle9i版本引入了自动PGA管理，如果使用的是AMM管理方式，则无需担心PGA的配置，但是如果对对AMM管理不放心的话，可以设置自动PGA管理，设置
     WORKAREA_SIZE_POLICY ＝ AUTO
然后指定PGA_AGGREGATE_TARGET大小即可。，
 
手工PAG管理
 
如果要做到精确的控制PGA，还可以设置手动管理PGA，设置
WORKAREA_SIZE_POLICY = manual
然后分别指定PGA相关参数即可：
PGA相关参数有：
SORT_AREA_SIZE
SORT_AREA_RETAINED_SIZE，
 
3.Oracle下磁盘存储性能优化
 
  ①选择文件系统存取数据
文件系统的选择
     单一文件系统（ext2、ext3、xfs等）
     集群文件系统（gfs、ocfs2）
文件系统存储优缺点：
     优点：管理维护方便。
     缺点：数据读写要经过操作系统级的缓存，效率不是很高。 
  ②      ASM（Automatic Storage Management）
ASM优点：
     数据可直接读写，无需经过操作系统存取效率很高，读写效率与直接的原始设备基本相同。
     Oracle提供了专门的管理和维护工具
 
 
