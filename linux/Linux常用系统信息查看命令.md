## Linux常用系统信息查看命令

### 系统

    uname -a               # 查看内核/操作系统/CPU信息
  
    head -n 1 /etc/issue   # 查看操作系统版本

    cat /proc/cpuinfo      # 查看CPU信息

    hostname               # 查看计算机名

    lspci -tv              # 列出所有PCI设备

    lsusb -tv              # 列出所有USB设备

    lsmod                  # 列出加载的内核模块

    env                    # 查看环境变量

### 资源

    free -m                # 查看内存使用量和交换区使用量
   
    df -h                  # 查看各分区使用情况

    du -sh &lt;目录名&gt;        # 查看指定目录的大小

    grep MemTotal /proc/meminfo   # 查看内存总量

    grep MemFree /proc/meminfo    # 查看空闲内存量

    uptime                 # 查看系统运行时间、用户数、负载

    cat /proc/loadavg      # 查看系统负载

### 磁盘和分区

    mount | column -t      # 查看挂接的分区状态
   
    fdisk -l               # 查看所有分区

    swapon -s              # 查看所有交换分区

    hdparm -i /dev/hda     # 查看磁盘参数(仅适用于IDE设备)

    dmesg | grep IDE       # 查看启动时IDE设备检测状况

### 网络

    ifconfig               # 查看所有网络接口的属性
   
    iptables -L            # 查看防火墙设置

    route -n               # 查看路由表

    netstat -lntp          # 查看所有监听端口

    netstat -antp          # 查看所有已经建立的连接

    netstat -s             # 查看网络统计信息

### 进程

    ps -ef                 # 查看所有进程
 
    top                    # 实时显示进程状态

### 用户
    
     w                      # 查看活动用户
    
    id &lt;用户名&gt;            # 查看指定用户信息

    last                   # 查看用户登录日志

    cut -d: -f1 /etc/passwd   # 查看系统所有用户

    cut -d: -f1 /etc/group    # 查看系统所有组

    crontab -l             # 查看当前用户的计划任务

### 服务
    
    chkconfig --list       # 列出所有系统服务
    
    chkconfig --list | grep on    # 列出所有启动的系统服务

### 程序
    
    rpm -qa                # 查看所有安装的软件包

### 命令组合
<table width="770" border="0" cellpadding="0" cellspacing="0">
   <colgroup><col>
   <col width="278">
   <col width="456">
   </colgroup><tbody><tr>
    <td height="24">
    </td><td>任务</td>
    <td>命令组合</td>
   </tr>
   <tr height="56" >
    <td height="56">1</td>
    <td >删除0字节文件</td>
    <td >find . -type f -size 0 -exec rm -rf {} \;<br>find . type f -size 0 -delete</td>
   </tr>
   <tr height="56" >
    <td height="56">2</td>
    <td >查看进程，按内存从大到小排列</td>
    <td >ps -e -o "%C : %p : %z : %a"|sort -k5 -nr</td>
   </tr>
   <tr height="56" >
    <td height="56">3</td>
    <td >按cpu利用率从大到小排列</td>
    <td >ps -e -o "%C : %p : %z : %a"|sort -nr</td>
   </tr>
   <tr height="56" >
    <td height="56">4</td>
    <td >打印说cache里的URL</td>
    <td >grep -r -a jpg /data/cache/* | strings | grep "http:" | awk -F'http:' '{print "http:"$2;}'</td>
   </tr>
   <tr height="56" >
    <td height="56">5</td>
    <td >查看http的并发请求数及其TCP连接状态</td>
    <td >netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'</td>
   </tr>
   <tr height="56" >
    <td height="56">6</td>
    <td >sed在这个文里Root的一行，匹配Root一行，将no替换成yes。</td>
    <td >sed -i '/Root/s/no/yes/' /etc/ssh/sshd_config<span>&nbsp;</span></td>
   </tr>
   <tr height="86.67">
    <td height="86.67">7</td>
    <td >如何杀掉mysql进程</td>
    <td >ps aux |grep mysql |grep -v grep<span>&nbsp; </span>|awk '{print $2}' |xargs kill -9 <br>killall -TERM mysqld<br>kill -9 `cat /usr/local/apache2/logs/httpd.pid`<span>&nbsp;</span></td>
   </tr>
   <tr height="56" >
    <td height="56">8</td>
    <td >显示运行3级别开启的服务(从中了解到cut的用途，截取数据)</td>
    <td >ls /etc/rc3.d/S* |cut -c 15-<span>&nbsp;</span></td>
   </tr>
   <tr height="132" style="height:99.00pt;mso-height-source:userset;mso-height-alt:1980;">
    <td height="132" style="height:99.00pt;" x:num="">9</td>
    <td >如何在编写SHELL显示多个信息，用EOF</td>
    <td >cat &lt;&lt; EOF<br>+--------------------------------------------------------------+<br>|<span>&nbsp;&nbsp; </span>=== Welcome to Tunoff services ===<span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span>|<br>+--------------------------------------------------------------+<br>EOF</td>
   </tr>
   <tr height="114.67">
    <td height="114.67">10</td>
    <td >for的用法(如给mysql建软链接)</td>
    <td >cd /usr/local/mysql/bin<br>for i in *<br>do ln /usr/local/mysql/bin/$i /usr/bin/$i<br>done</td>
   </tr>
   <tr height="88">
    <td height="88">11</td>
    <td >取IP地址</td>
    <td >ifconfig eth0 |grep "inet addr:" |awk '{print $2}'|cut -c 6- <br>ifconfig | grep 'inet addr:'| grep -v '127.0.0.1' |cut -d: -f2 | awk '{ print $1}'</td>
   </tr>
   <tr height="56" >
    <td height="56">12</td>
    <td >内存的大小</td>
    <td >free -m |grep "Mem" | awk '{print $2}'</td>
   </tr>
   <tr height="68">
    <td height="68">13</td>
    <td >查看80端口的连接，并排序</td>
    <td >netstat -an -t | grep ":80" | grep ESTABLISHED | awk '{printf "%s %s\n",$5,$6}' | sort</td>
   </tr>
   <tr height="72">
    <td height="72">14</td>
    <td >查看Apache的并发请求数及其TCP连接状态</td>
    <td >netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'</td>
   </tr>
   <tr height="56" >
    <td height="56">15</td>
    <td >统计一下服务器下面所有的jpg的文件的大小</td>
    <td >find / -name *.jpg -exec wc -c {} \;|awk '{print $1}'|awk '{a+=$1}END{print a}'</td>
   </tr>
   <tr height="56" >
    <td height="56">16</td>
    <td >CPU的数量</td>
    <td >cat /proc/cpuinfo |grep -c processor</td>
   </tr>
   <tr height="56" >
    <td height="56">17</td>
    <td >CPU负载<span>&nbsp;</span></td>
    <td >cat /proc/loadavg</td>
   </tr>
   <tr height="56" >
    <td height="56">18</td>
    <td >CPU负载<span>&nbsp;</span></td>
    <td >mpstat 1 1</td>
   </tr>
   <tr height="56" >
    <td height="56">19</td>
    <td >内存空间<span>&nbsp;</span></td>
    <td >free</td>
   </tr>
   <tr height="56" >
    <td height="56">20</td>
    <td >磁盘空间<span>&nbsp;</span></td>
    <td >df -h</td>
   </tr>
   <tr height="56" >
    <td height="56">21</td>
    <td >如发现某个分区空间接近用尽，可以进入该分区的挂载点，用以下命令找出占用空间最多的文件或目录</td>
    <td >du -cks * | sort -rn | head -n 10</td>
   </tr>
   <tr height="56" >
    <td height="56">22</td>
    <td >磁盘I/O负载<span>&nbsp;</span></td>
    <td >iostat -x 1 2</td>
   </tr>
   <tr height="56" >
    <td height="56">23</td>
    <td >网络负载<span>&nbsp;</span></td>
    <td >sar -n DEV</td>
   </tr>
   <tr height="56" >
    <td height="56">24</td>
    <td >网络错误<span>&nbsp;</span></td>
    <td >netstat -i<br>cat /proc/net/dev</td>
   </tr>
   <tr height="56" >
    <td height="56">25</td>
    <td >网络连接数目</td>
    <td >netstat -an | grep -E “^(tcp)” | cut -c 68- | sort | uniq -c | sort -n</td>
   </tr>
   <tr height="56" >
    <td height="56">26</td>
    <td >进程总数</td>
    <td >ps aux | wc -l</td>
   </tr>
   <tr height="56" >
    <td height="56">27</td>
    <td >查看进程树</td>
    <td >ps aufx</td>
   </tr>
   <tr height="56" >
    <td height="56">28</td>
    <td >可运行进程数目</td>
    <td >vmwtat 1 5</td>
   </tr>
   <tr height="56" >
    <td height="56">29</td>
    <td >检查DNS Server工作是否正常，这里以61.139.2.69为例</td>
    <td >dig www.baidu.com @61.139.2.69</td>
   </tr>
   <tr height="56" >
    <td height="56">30</td>
    <td >检查当前登录的用户个数</td>
    <td >who | wc -l</td>
   </tr>
   <tr height="112">
    <td height="112">31</td>
    <td >日志查看、搜索</td>
    <td >cat /var/log/rflogview/*errors<br>grep -i error /var/log/messages<br>grep -i fail /var/log/messages<br>tail -f -n 2000 /var/log/messages</td>
   </tr>
   <tr height="56" >
    <td height="56">32</td>
    <td >内核日志</td>
    <td >dmesg</td>
   </tr>
   <tr height="56" >
    <td height="56">33</td>
    <td >时间</td>
    <td >date</td>
   </tr>
   <tr height="56" >
    <td height="56">34</td>
    <td >已经打开的句柄数</td>
    <td >lsof | wc -l</td>
   </tr>
   <tr height="56" >
    <td height="56">35</td>
    <td >网络抓包，直接输出摘要信息到文件。</td>
    <td >tcpdump -c 10000 -i eth0 -n dst port 80 &gt; /root/pkts</td>
   </tr>
   <tr height="96">
    <td height="96">36</td>
    <td >然后检查IP的重复数 并从小到大排序 注意 "-t\<span>&nbsp; </span>+0" 中间是两个空格，less命令的用法。</td>
    <td >less pkts | awk {'printf $3"\n"'} | cut -d. -f 1-4 | sort | uniq -c | awk {'printf $1" "$2"\n"'} | sort -n -t\<span>&nbsp; </span>+0</td>
   </tr>
   <tr height="56" >
    <td height="56">37</td>
    <td >kudzu查看网卡型号</td>
    <td >kudzu --probe --class=network</td>
   </tr>
  </tbody></table>

