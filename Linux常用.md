# LINUX

### 终端常见快捷键
```
Ctrl+c	中断程序（如find /）
Ctrl+d	键盘输入结束或退出终端
Ctrl+s	暂停当前程序，暂停后按下任意键恢复运行（方便看动态log用）
Ctrl+z	将当前程序放到后台运行，恢复到前台为命令fg

Ctrl+a	将光标移至输入行头，相当于Home键
Ctrl+e	将光标移至输入行末，相当于End键
Ctrl+k	删除从光标所在位置到行末

Alt+Backspace	向前删除一个单词
Shift+PgUp	将终端显示向上滚动
Shift+PgDn	将终端显示向下滚动

```

### Shell常用通配符
```
*	匹配 0 或多个字符
?	匹配任意一个字符
[list]	匹配 list 中的任意单一字符
[^list]	匹配 除list 中的任意单一字符以外的字符
[c1-c2]	匹配 c1-c2 中的任意单一字符 如：[0-9] [a-z]
{string1,string2,...}	匹配 string1 或 string2 (或更多)其一字符串
{c1..c2}	匹配 c1-c2 中全部字符 如{1..10}
```

### 搜索文件
- whereis 简单快速
	- 并不从硬盘中依次查找而是直接从数据库中查询
	- 只能搜索二进制文件、man帮助文件、源码文件
	- 如果想获取全面搜索结果可以用locate
	```
	whereis man
	```
- locate 快而全
	- 通过“/var/lib/mlocate/mlocate.db”数据库查找
	- 此数据库非实时更新，系统定时任务每天执行updatedb命令更新一次
	- 所以有时刚添加的文件可能会找不到，需要手动执行一次updatedb命令
	```
	locate /etc/sh	#查找/etc下所有sh开头的文件
	locate /usr/share/*.jpg	#查找/usr/share/下所有jpg文件
	```
- which 小而精
	- shell内建命令
	- 常用来确定是否安装了某个指定软件
	- 只从PATH环境变量路径中搜索命令
	```
	which man
	```
- find 精而细
	- 这几个命令中最强大的
	- 可以通过文件类型、文件名、文件属性进行搜索
	```
	find /etc/ -name interfaces #在/etc/目录下搜索名为interfaces的文件或者目录
	#find [path] [-option] [action]
	
	find ~ -mtime 0 #列出home目录中，当天（24小时之内）有改动的文件
	find ~ -newer /home/shiyanlou/code #列出用户家目录下比code文件新的文件
	```


### 切分文件
```
head -n K file_name > top_file
tail -n L file_name > bottom_file
wc -l file_name
```

### grep
```
#在当前路径下找sth
grep sth . -r
```

### find
```
#查找某文件中的sth
find . -name "*.pom" -exec grep -Hin "scala-2.10" {} \; 找某个文件中的某行，在maven build失败时非常好用
#查找/etc下某个文件夹dir
find /etc -name dir
```
### centos(linux)为网卡设备分配静态IP
```
cd /etc/sysconfig/network-scripts/
拷贝ifcfg-eth0，重命名成对应网卡的名称，修改文件里对应内容
service network restart
```
### 开启linux的路由转发功能
```
vim /etc/sysctl.conf 中的第一项设为1
sysctl -p 生效
```
### 添加了网卡后ifconfig中不显示添加的网卡信息
```
vim /etc/udev/rules.d/70-persistent-net.rules 将网卡地址和名称写好
/etc/sysconfig/network-scripts 中添加ifcfg-ethx文件
ifup ethx
```
### iperf
```
服务器：iperf -s -D -p 2333
客户端：iperf -c [server ip] -p 2333 -f m -t 5
```
### linux新建指定大小文件
```
dd if=/dev/zero of=50M.file bs=1M count=50
```

### 修改swap文件
```
cat /proc/swaps 或 swapon -s  #列出所有的swap files
swapoff -a  #disable掉所有的swap file
vim /etc/fstab  #将相应的行从fstab里面注释掉
#rm掉原来的swapfile
sudo dd if=/dev/zero of=/swapfile bs=1024 count=4194304  #造一个4GB（内存一半）的大文件
chmod 0600 /swapfile  #修改权限
mkswap /swapfile  #mark其为swap file
swapon /swapfile  #激活swap file
vim /etc/fstab  #写入
free -m  #查看虚拟内存
```

### 压缩打包
- zip
    ```
    #压缩  -r递归打包子目录、-q安静模式、-o输出文件
    zip -r -q -o shiyanlou.zip /home/shiyanlou/Desktop  
    du -h shiyanlou.zip
    
    #设置压缩等级（9最大、1最小）
    zip -r -9 -q -o shiyanlou_9.zip /home/shiyanlou/Desktop -x ~/*.zip  #-x排除上次创建的zip文件
    
    #加密压缩包
    zip -r -e -o shiyanlou_encryption.zip /home/shiyanlou/Desktop #-e
    
    #LF转CR+LF (win换行是CR+LF，linux换行是LF)
    zip -r -l -o shiyanlou.zip /home/shiyanlou/Desktop  #-l将LF转CR+LF
    
    
    #解压 到当前目录
    unzip shiyanlou.zip 
    
    #解压 到指定目录
    unzip -q shiyanlou.zip -d ziptest   #如果ziptest不存在则创建
    
    #查看压缩包内容
    upzip -l shiyanlou.zip
    
    #解压编码问题   win是GBK linux是UTF-8
    unzip -O GBK 中文压缩文件.zip
    
    ```

- tar
    ```
    # 创建一个tar包
    tar -cf test.tar /home/test/Desktop #-c创建一个tar包，-f指定创建的文件名
    
    # 解包
    tar -xf test.tar -C tardir  #-x解包，-C到已存在的目录
    
    # 查看
    tar -tf test.tar #只查看不解包
    
    # 压缩
    tar -czf test.tar.gz /home/test/Desttop
    tar -xzf test.tar.gz
    # -z: *.tar.gz; 
    # -J: *.tar.xz;
    # -j: *.tar.bz2;
    ```

### 磁盘相关

- dd简介
    - 参数是“选项=值”的形式，而非标准的“--选项 值”或“-选项=值”
    - 默认从stdin输入，输出到stdout，但可以用if和of改变
    ```
    # 输出到文件
    dd of=test bs=10 count=1 
    或者 dd if=/dev/stdin of=test bs=10 count=1
    
    # 输出到标准输出
    dd if=/dev/stdin of=/dev/stdout bs=10 count=1
    
    # bs(block size)指定块大小，缺省单位Byte
    # count指定块数
    # 如果输入大于指定数量，多余输入将被截取并保留在标准输入
    ```
    - dd在拷贝同时还可以实现数据转换
    ```
    # 将输出的英文字符转换为大写再写入文件，可以在man文档中查看其他所有转换参数
    dd if=/dev/stdin of=test bs=10 count=1 conv=ucase
    ```

- 用dd创建虚拟镜像文件
    ```
    # 从/dev/zero设备创建一个容量为 256M 的空文件
    dd if=/dev/zero of=virtual.img bs=1M count=256
    du -h virtual.img
    ```
    
- mount命令挂载磁盘到目录树
    ```
    # 主机已经挂载的文件系统
    mount
    # 设备名 on 挂载点 type 文件系统类型 (挂在选项)
    
    mqueue on /dev/mqueue type mqueue (rw,relatime)
    tmpfs on /run/user/0 type tmpfs (rw,nosuid,nodev,relatime,size=800944k,mode=700)
    systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=31,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=331752)
    /dev/vda1 on /var/lib/docker/containers type ext4 (rw,noatime,data=ordered)
    /dev/vda1 on /var/lib/docker/overlay2 type ext4 (rw,noatime,data=ordered)

    # 挂在磁盘到目录树
    mount [options] [source] [directory]
    mount [-o [操作选项]] [-t 文件系统类型] [-w|--rw|--ro] [文件系统源] [挂载点]
    eg：mount -o loop -t ext4 virtual.img /mnt
    ```

### Linux下的帮助命令

- type区分内建命令与外部命令
    ```
    # 内建命令
    xxx is a shell builtin
    # 外部命令，外部命令在/usr/bin or /usr/sbin等等中
    xxx is /usr/bin/xxx
    # 该指令为命令别名所设定的名称
    xxx is an alias for xx --xxx
    ```
- help命令
    ```
    # help 命令是用于显示 shell 内建命令的简要帮助信息
    help cd
    help vim #报错
    
    # 外部命令用--help
    ls --help
    ```
- man命令
    ```
    # 比help更详细，不区分内建命令与外部命令
    
    # 章节数	说明
        1	Standard commands （标准命令）
        2	System calls （系统调用）
        3	Library functions （库函数）
        4	Special devices （设备说明）
        5	File formats （文件格式）
        6	Games and toys （游戏和娱乐）
        7	Miscellaneous （杂项）
        8	Administrative Commands （管理员命令）
        9	其他（Linux特定的）， 用来存放内核例行程序的文档。
    ```
- info命令
    ```
    # 完整的显示GNU信息，内容比man更丰富
    ```
    
### crontab简介
- crontab命令存放在crontab文件中以供之后读取和执行
- crontab存储的指令会被其守护进程crond激活
- crond守护进程驻留在后台，每分钟检查一次是否有预定作业要执行
- 在固定的间隔时间执行（分、时、日、月、周）
    ```
    # Example of job definition:
    # .---------------- minute (0 - 59)
    # |  .------------- hour (0 - 23)
    # |  |  .---------- day of month (1 - 31)
    # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
    # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
    # |  |  |  |  |
    # *  *  *  *  * user-name command to be executed
    
    # 编辑
    crontab -e
    # 每个用户使用的crontab -e添加的计划任务都会在/var/spool/cron/crontabs中添加一个该用户自己的任务文档
    
    # 每分钟创建一个以日期为名字的空白文件
    */1 * * * * touch /home/shiyanlou/$(date +\%Y\%m\%d\%H\%M\%S)
    
    # 查看
    crontab -l
    
    # 查看到执行任务命令之后在日志中的信息反馈
    tail -f /var/log/syslog
    
    # 清空crontab
    crontab -r
    ```
- 系统级别的定时任务
    ```
    # 编辑/etc/crontab文件
    
    # cron服务检测时间最小单位是分钟，cron每分钟读取一次/etc/crontab与/var/spool/cron/crontabs中的内容
    
    # /etc目录下cron相关目录有
    ll /etc/ |grep cron
    -rw-------   1 root root      541 Nov 20  2018 anacrontab
    drwxr-xr-x.  2 root root     4096 Mar 11  2019 cron.d
    drwxr-xr-x.  2 root root     4096 Mar 11  2019 cron.daily
    -rw-------   1 root root        0 Nov 20  2018 cron.deny
    drwxr-xr-x.  2 root root     4096 Mar 11  2019 cron.hourly
    drwxr-xr-x.  2 root root     4096 Jun 10  2014 cron.monthly
    -rw-r--r--.  1 root root      451 Jun 10  2014 crontab
    drwxr-xr-x.  2 root root     4096 Jun 10  2014 cron.weekly

    /etc/cron.daily，目录下的脚本会每天执行一次，在每天的6点25分时运行；
    /etc/cron.hourly，目录下的脚本会每个小时执行一次，在每小时的17分钟时运行；
    /etc/cron.monthly，目录下的脚本会每月执行一次，在每月1号的6点52分时运行；
    /etc/cron.weekly，目录下的脚本会每周执行一次，在每周第七天的6点47分时运行；
    ```
    
### 常用
- 命令的顺序执行与选择执行
    ```
    # ; 表示命令顺序执行
    command1;command2
    
    # &&与|| 表示命令选择执行
    command1 && command2 # command1执行成功（返回0）后执行command2
    command1 || command2 # command1执行失败（返回非0）后执行command2
    ```
- 管道命令，是通过socket进行进程间通信
- |命令，将前一个命令的输出作为下一个命令的输入
- cut命令
    ```
    # cut命令，打印每一行的某一字段
    cut /etc/passwd -d ':' -f 1,6 # 打印/etc/passwd文件中以:为分隔符的第1个字段和第6个字段
    # 前五个（包含第五个）
    cut /etc/passwd -c -5
    # 前五个之后的（包含第五个）
    cut /etc/passwd -c 5-
    # 第五个
    cut /etc/passwd -c 5
    # 2到5之间的（包含第五个）
    cut /etc/passwd -c 2-5
    ```
 
- grep命令
    ``` 
    # grep [命令选项]... 用于匹配的表达式 [文件]...
    grep -rnI "aaa" ~ #-r递归，-n打印行号，-I忽略二进制文件
    
    # wc命令，简单小巧的计数工具
    wc /etc/passwd #输出一个文件的行数、单词数、字节数
    # 行数
    wc -l /etc/passwd
    # 单词数
    wc -w /etc/passwd
    # 字节数
    wc -c /etc/passwd
    # 字符数
    wc -m /etc/passwd
    # 最长行字节数
    wc -L /etc/passwd
   ```
    
- sort排序命令
    ```
    # 支持字典排序（默认）、数字排序、按月份排序、随机排序、反转排序、指定特定字段排序
    # 字典排序
    cat /etc/passwd | sort
    # 反转排序
    cat /etc/passwd | sort -r
    # 按特定字段排序
    cat /etc/passwd | sort -t':' -k 3 #-t分隔符，-k字段号，默认按字典序
    cat /etc/passwd | sort -t':' -k 3 -n #-n按数字排序
    ```
    
- uniq去重命令
    ```
    # 只看history中的去重命令
    history | cut -c 8- | cut -d ' ' -f 1 | uniq
    # uniq命令只能去连续重复的行，不是全文去重，所以要全文去重需先排好序
    history | cut -c 8- | cut -d ' ' -f 1 | sort | uniq
    # 或者
    history | cut -c 8- | cut -d ' ' -f 1 | sort -u
    # 输出重复过的行（重复的只输出一个）及重复次数
    history | cut -c 8- | cut -d ' ' -f 1 | sort | uniq -dc
    ```

### 常见的文本处理
- tr命令
    ```
    # 删除一段文本信息中的某些文字，或将其转换
    tr [option]...SET1 [SET2]   #-d删除set1中的字符，-s去除文本中set1指定的重复字符
    
    # 删除 "hello shiyanlou" 中所有的'o','l','h'
    echo 'hello shiyanlou' | tr -d 'olh'
    # 将"hello" 中的ll,去重为一个l
    echo 'hello' | tr -s 'l'
    # 将输入文本，全部转换为大写或小写输出
    echo 'input some text here' | tr '[:lower:]' '[:upper:]'
    ```
- col命令
    ```
    # col 命令可以将Tab换成对等数量的空格键，或反转这个操作
    col [option] #-x将tab转空格 -h将空格转tab
    
    # 查看 /etc/protocols 中的不可见字符，可以看到很多 ^I ，这其实就是 Tab 转义成可见字符的符号
    cat -A /etc/protocols
    # 使用 col -x 将 /etc/protocols 中的 Tab 转换为空格,然后再使用 cat 查看，你发现 ^I 不见了
    cat /etc/protocols | col -x | cat -A
    ```
- join命令
    ```
    # 将两个文件中包含相同内容的那一行合并在一起
    join [option]... file1 file2
        -t  指定分隔符，默认为空格
        -i	忽略大小写的差异
        -1	指明第一个文件要用哪个字段来对比，默认对比第一个字段
        -2	指明第二个文件要用哪个字段来对比，默认对比第一个字
        
    # eg
    echo '1 hello' > file1
    echo '1 shiyanlou' > file2
    join file1 file2
    # 将/etc/passwd与/etc/shadow两个文件合并，指定以':'作为分隔符
    sudo join -t':' /etc/passwd /etc/shadow
    # 将/etc/passwd与/etc/group两个文件合并，指定以':'作为分隔符, 分别比对第4和第3个字段
    sudo join -t':' -1 4 /etc/passwd -2 3 /etc/group
    ```
- paste命令
    ```
    # 简单地将多个文件合并一起，以Tab隔开
    paste [option] file...
        -d	指定合并的分隔符，默认为Tab
        -s	不合并到一行，每个文件为一行
    
    # eg
    echo hello > file1
    echo shiyanlou > file2
    echo www.shiyanlou.com > file3
    paste -d ':' file1 file2 file3
    paste -s file1 file2 file3
    ```
    
### 数据流重定向
- 常用的两个重定向>与>>
    ```
    echo 'aaa' > redirect #rewrite
    echo 'bbb' >> redirect #append
    ```
- 三个特殊设备
    ```
    0	/dev/stdin	标准输入
    1	/dev/stdout	标准输出
    2	/dev/stderr	标准错误
    ```
- 标准错误重定向
    ```
    # 一个命令的输出通常是同时包含了标准输出和标准错误的结果
    
    # 假设file1.c存在，file2.c不存在
    cat file1.c file2.c # 会先将file1.c打印到stdout，file2.c不存在打印到stderr，都是屏幕上
    cat file1.c file2.c > stdoutfile # 会先将file1.c重定向到stdoutfile，file2.c不存在打印到stderr屏幕上
    
    # 将stderr重定向到stdout，再将stdout重定向到文件，注意要将重定向到文件写到前面
    # 注意你应该在输出重定向文件描述符前加上&,否则shell会当做重定向到一个文件名为1的文件中
    cat file1.c file2.c >somefile  2>&1
    # 或者只用bash提供的特殊的重定向符号"&"将标准错误和标准输出同时重定向到文件
    cat file1.c file2.c &>somefilehell
    
    # tee命令同时重定向到多个文件
    echo 'liy' | tee liy1 liy2 #将信息既输出到stdout又输出到liy1、liy2文中件
    ```
- 永久重定向用exec命令
- 创建输出文件描述符
    ```
    # shell中有9个fd
    # 0，1，2是默认fd
    # 我们可以使用3-8的fd
    cd /dev/fd/;ls -Al
    
    # 使用exec穿件新的fd
    zsh
    exec 3>somefile
    # 先进入目录，再查看，否则你可能不能得到正确的结果，然后再回到上一次的目录
    cd /dev/fd/;ls -Al;cd -
    # 注意下面的命令>与&之间不应该有空格，如果有空格则会出错
    echo "this is test" >&3
    cat somefile
    exit
    
    #关闭fd
    exec 3>&-
    ```
    
### 正则
- 基础语法
    - PCRE
        ```
        # PCRE（Perl Compatible Regular Expressions中文含义：perl语言兼容正则表达式）
      # 是一个用 C 语言编写的正则表达式函数库，由菲利普.海泽(Philip Hazel)编写。
        # PCRE是一个轻量级的函数库，比Boost 之类的正则表达式库小得多。
        # PCRE 十分易用，同时功能也很强大，性能超过了 POSIX 正则表达式库和一些经典的正则表达式库。
        ```
    - \ : 将下一个字符标记为一个特殊字符、或一个原义字符。eg\n匹配换行，\\\(匹配\(
    - ^ : 匹配输入字符串的开始位置
    - $ : 匹配输入字符串的结束位置
    - {n} :	n是一个非负整数。匹配确定的n次。例如，“o{2}”不能匹配“Bob”中的“o”，但是能匹配“food”中的两个o。
    - {n,} : n是一个非负整数。至少匹配n次。例如，“o{2,}”不能匹配“Bob”中的“o”，但能匹配“foooood”中的所有o。“o{1,}”等价于“o+”。“o{0,}”则等价于“o*”。
    - {n,m} : m和n均为非负整数，其中n<=m。最少匹配n次且最多匹配m次。例如，“o{1,3}”将匹配“fooooood”中的前三个o。“o{0,1}”等价于“o?”。请注意在逗号和两个数之间不能有空格。
    - \* : 匹配前面的子表达式零次或多次。例如，zo*能匹配“z”、“zo”以及“zoo”。*等价于{0,}。
    - \+ : 匹配前面的子表达式一次或多次。例如，“zo+”能匹配“zo”以及“zoo”，但不能匹配“z”。+等价于{1,}。
    - ? : 匹配前面的子表达式零次或一次。例如，“do(es)?”可以匹配“do”或“does”中的“do”。?等价于{0,1}。
    - . : 匹配除“\n”之外的任何单个字符。要匹配包括“\n”在内的任何字符，请使用像“(.｜\n)”的模式。
    - (pattern) : 匹配pattern并获取这一匹配的子字符串。该子字符串用于向后引用。要匹配圆括号字符，请使用“\(”或“\)”。
    - x｜y : 匹配x或y。例如，“z｜food”能匹配“z”或“food”。“(z｜f)ood”则匹配“zood”或“food”。
    - [xyz] : 字符集合（character class）。匹配所包含的任意一个字符。例如，“[abc]”可以匹配“plain”中的“a”。其中特殊字符仅有反斜线\保持特殊含义，用于转义字符。其它特殊字符如星号、加号、各种括号等均作为普通字符。脱字符^如果出现在首位则表示负值字符集合；如果出现在字符串中间就仅作为普通字符。连字符 - 如果出现在字符串中间表示字符范围描述；如果出现在首位则仅作为普通字符。
    - [^xyz] : 排除型（negate）字符集合。匹配未列出的任意字符。例如，“[^abc]”可以匹配“plain”中的“plin”。
    - [a-z] : 字符范围。匹配指定范围内的任意字符。例如，“[a-z]”可以匹配“a”到“z”范围内的任意小写字母字符。
    [^a-z] : 排除型的字符范围。匹配任何不在指定范围内的任意字符。例如，“[^a-z]”可以匹配任何不在“a”到“z”范围内的任意字符。
    
    - 优先级
        ```
        优先级为从上到下从左到右，依次降低：
        运算符	说明
        \	转义符
        (), (?:), (?=), []	括号和中括号
        *、+、?、{n}、{n,}、{n,m}	限定符
        ^、$、\任何元字符	定位点和序列
        ｜	选择
        ```
- grep命令
    - 打印文本中匹配的模式串
        ```
        用法：
        grep [OPTION]... PATTERN [FILE]...
        ```
    - 三个参数指定三种正则表达式引擎
        ```
        参数	说明
        -E	POSIX扩展正则表达式，ERE
        -G	POSIX基本正则表达式，BRE
        -P	Perl正则表达式，PCRE
        ```
    - BRE eg
        ```
        grep -i -n aaa file.c
        grep -in aaa file.c
        
        grep 'shiyanlou' /etc/group
        grep '^shiyanlou' /etc/group
        
        # 将匹配以'z'开头以'o'结尾的所有字符串
        echo 'zero\nzo\nzoo' | grep 'z.*o'
        # 将匹配以'z'开头以'o'结尾，中间包含一个任意字符的字符串
        echo 'zero\nzo\nzoo' | grep 'z.o'
        # 将匹配以'z'开头,以任意多个'o'结尾的字符串
        echo 'zero\nzo\nzoo' | grep 'zo*'
        
        # grep默认是区分大小写的，这里将匹配所有的小写字母
        echo '1234\nabcd' | grep '[a-z]'
        # 将匹配所有的数字
        echo '1234\nabcd' | grep '[0-9]'
        # 将匹配所有的数字
        echo '1234\nabcd' | grep '[[:digit:]]'
        # 将匹配所有的小写字母
        echo '1234\nabcd' | grep '[[:lower:]]'
        # 将匹配所有的大写字母
        echo '1234\nabcd' | grep '[[:upper:]]'
        # 将匹配所有的字母和数字，包括0-9,a-z,A-Z
        echo '1234\nabcd' | grep '[[:alnum:]]'
        # 将匹配所有的字母
        echo '1234\nabcd' | grep '[[:alpha:]]'
        
        # 排除字符
        echo 'geek\ngood' | grep '[^o]'
        ```
    - ERE eg
        ```
        # 只匹配"zo"
        echo 'zero\nzo\nzoo' | grep -E 'zo{1}'
        # 匹配以"zo"开头的所有单词
        echo 'zero\nzo\nzoo' | grep -E 'zo{1,}'
        
        # 匹配"www.shiyanlou.com"和"www.google.com"
        echo 'www.shiyanlou.com\nwww.baidu.com\nwww.google.com' | grep -E 'www\.(shiyanlou|google)\.com'
        # 或者匹配不包含"baidu"的内容
        echo 'www.shiyanlou.com\nwww.baidu.com\nwww.google.com' | grep -Ev 'www\.baidu\.com'
        ```
- sed命令
    - 用法
        ```
        sed [参数]... [执行命令] [输入文件]...
        
        eg:
        sed -i 's/sad/happy/' test # 表示将test文件中的"sad"替换为"happy"
        ```
    - [执行命令]格式
        ```
        [n1][,n2]command    # 从n1到n2行
        [n1][~step]command  # 从n1开始以step为步进的所有行
        
        # 常用的command
        命令	说明
        s	行内替换
        c	整行替换
        a	插入到指定行的后面
        i	插入到指定行的前面
        p	打印指定行，通常与-n参数配合使用
        d	删除指定行
        
        # 其中一些命令可以在后面加上作用范围，形如：
        sed -i 's/sad/happy/g' test # g表示全局范围
        sed -i 's/sad/happy/4' test # 4表示指定行中的第四个匹配字符串
        ```
    - eg
        ```
        # 打印2-5行
        nl passwd | sed -n '2,5p'
        # 打印奇数行
        nl passwd | sed -n '1~2p'
        
        # 将输入文本中"shiyanlou" 全局替换为"hehe",并只打印替换的那一行，注意这里不能省略最后的"p"命令
        sed -n 's/shiyanlou/hehe/gp' passwd
        
        # 删除第30行
        sed -i '30d' passwd
        ```
- awk命令
    - awk是优良的文本处理工具
    - linux环境中功能最强大的数据处理引擎一只
    - 处理文本的编程语言工具
    - 又叫样式扫描和处理语言
    - awk操作都是pattern {action}模式
        - pattern: 匹配输入文本的“关系式或正则式”
        - action: 匹配后将执行的动作
        - 二者可只有其中一个
        - 没有pattern则默认匹配输入的全部文本
        - 没有action则默认打印匹配内容到屏
    - awk处理文本的方式是将文本分割成一些“字段”，然后再对这些字段处理，可指定分隔符默认是空格分隔
    - 基本格式
        ```
        awk [-F fs] [-v var=value] [-f prog-file | 'program text'] [file...]
        -F: 指定字段分隔符
        -v: 预先为awk程序指定变量
        -f: 指定要执行的程序文件或不加-f直接写上处理程序
        ```
    - awk操作体验
        - awk将文本内容打印到终端
            ```
            echo “I like linux\nwww.baidu.com”
            awk '{print}' test  # 省略了pattern，默认匹配全部内容
            ```
        - 将test的第一行的每个字段单独显示为一行
            ```
            awk '{
            > if(NR==1){
            > print $1 "\n" $2 "\n" $3
            > } else {
            > print}
            > }' test
            
            # 或者
            awk '{
            > if(NR==1){
            > OFS="\n"
            > print $1, $2, $3
            > } else {
            > print}
            > }' test
            
            # NR: awk内建变量，表示当前读入的记录数，简单理解为行数
            # OFS: awk内建变量，表示输出时的字段分隔符
            # $N: awk内建变量，表示引用相应的字段
            # $0: awk内建变量，表示引用当前行的全部内容
            ```
        - 将test的第二行的以点为分段的字段换成以空格为分隔
            ```
            awk -F'.' '{
            > if(NR==2){
            > print $1 "\t" $2 "\t" $3
            > }}' test
            
            # 或者
            awk '
            > BEGIN{
            > FS="."
            > OFS="\t"  # 如果写为一行，两个动作语句之间应该以";"号分开  
            > }{
            > if(NR==2){
            > print $1, $2, $3
            > }}' test
            
            # -F: 预先指定待处理记录的字段分隔符
            # print打印的非变量内容都需要用""一对引号包围起来
            # BEGIN: 其中的动作将在所有动作前最先执行
            ```
    - awk常用的内置变量
        ```
        变量名	说明
        FILENAME	当前输入文件名，若有多个文件，则只表示第一个。如果输入是来自标准输入，则为空字符串
        $0	当前记录的内容
        $N	N表示字段号，最大值为NF变量的值
        FS	字段分隔符，由正则表达式表示，默认为" "空格
        RS	输入记录分隔符，默认为"\n"，即一行为一个记录
        NF	当前记录字段数
        NR	已经读入的记录数
        FNR	当前输入文件的记录数，请注意它与NR的区别
        OFS	输出字段分隔符，默认为" "空格
        ORS	输出记录分隔符，默认为"\n"
        ```

### apt-get
    ```
    # 更新软件源
    sudo apt-get update
    # 升级没有依赖问题的软件包，不解决依赖
    sudo apt-get upgrade
    # 升级并解决依赖关系
    sudo apt-get dist-upgrade
    
    # 卸载软件，但保留配置
    sudo apt-get remove w3m 
    # 不保留配置文件的移除
    sudo apt-get purge w3m
    # 或者 sudo apt-get --purge remove
    # 移除不再需要的被依赖的软件包
    sudo apt-get autoremove
    
    # apt-cache 命令则是针对本地数据进行相关操作的工具，search 在本地的数据库
    sudo apt-cache search softname1 softname2 softname3……
    ```

### Linux进程管理

- top命令
    - top的第一行
        ```
        内容	            解释
        top             表示当前程序的名称
        11:05:18	    表示当前的系统的时间
        up 8 days,17:12	表示该机器已经启动了多长时间
        1 user	        表示当前系统中只有一个用户
        load average: 0.29,0.20,0.25	   分别对应1、5、15分钟内cpu的平均负载
        
        # 假设是2核的机器，load average=2表示cpu占满，1表示占一半，3表示超负荷。一般都是将这个值除以核数来看
        
        # 查看物理CPU的个数
        cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l
        # 每个cpu的核心数
        cat /proc/cpuinfo |grep "physical id"|grep "0"|wc -l
        
        # 若是 load < 0.7 并不会去关注他；
        # 若是 0.7< load < 1 的时候我们就需要稍微关注一下了，虽然还可以应付但是这个值已经离临界不远了；
        # 若是 load = 1 的时候我们就需要警惕了，因为这个时候已经没有更多的资源的了，已经在全力以赴了；
        # 若是 load > 5 的时候系统已经快不行了，这个时候你需要加班解决问题了
        
        # 通常我们都会先看 15 分钟的值来看这个大体的趋势，然后再看 5 分钟的值对比来看是否有下降的趋势
        ```
    - top的第二行
        ```
        # 进程状态统计
        内容	                解释
        Tasks: 26 total	    进程总数
        1 running	        1个正在运行的进程数
        25 sleeping	        25个睡眠的进程数
        0 stopped	        没有停止的进程数
        0 zombie	        没有僵尸进程数
        ```
    - top的第三行
        ```
        # CPU使用情况统计
        内容	            解释
        Cpu(s): 1.0%us  用户空间进程占用CPU百分比
        1.0% sy	        内核空间运行占用CPU百分比
        0.0%ni	        用户进程空间内改变过优先级的进程占用CPU百分比
        97.9%id     	空闲CPU百分比
        0.0%wa	        等待输入输出的CPU时间百分比
        0.1%hi	        硬中断(Hardware IRQ)占用CPU的百分比
        0.0%si	        软中断(Software IRQ)占用CPU的百分比
        0.0%st	        (Steal time) 是 hypervisor 等虚拟服务中，虚拟 CPU 等待实际 CPU 的时间的百分比
        ```
    - top的第四行
        ```
        # 内存使用情况统计
        内容	            解释
        8176740 total	物理内存总量
        8032104 used	使用的物理内存总量
        144636 free	    空闲内存总量
        313088 buffers	用作内核缓存的内存量
        
        # 系统中可用的物理内存最大值并不是 free 这个单一的值，而是 free + buffers + swap 中的 cached 的和
        ```
    - top的第五行
        ```
        # 交换区Swap的使用情况
        内容	    解释
        total	交换区总量
        used	使用的交换区总量
        free	空闲交换区总量
        cached	缓冲的交换区总量,内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖
        ```
    - top的第五行以下
        ```
        # 进程状况
        列名	    解释
        PID	    进程id
        USER    该进程的所属用户
        PR	    该进程执行的优先级 priority 值
        NI	    该进程的 nice 值
        VIRT    该进程任务所使用的虚拟内存的总数
        RES	    该进程所使用的物理内存数，也称之为驻留内存数
        SHR	    该进程共享内存的大小
        S	    该进程进程的状态: S=sleep R=running Z=zombie
        %CPU    该进程CPU的利用率
        %MEM    该进程内存的利用率
        TIME+   该进程活跃的总时间
        COMMAND 该进程运行的名字
        ```
    - 与top交互
        ```
        交互命令	    解释
        q	        退出程序
        I	        切换显示平均负载和启动时间的信息
        P       	根据CPU使用百分比大小进行排序
        M	        根据驻留内存大小进行排序
        i       	忽略闲置和僵死的进程，这是一个开关式命令
        k	        终止一个进程，系统提示输入 PID 及发送的信号值。一般终止进程用 15 信号，不能正常结束则使用 9 信号。安全模式下该命令被屏蔽。
        ```

# Linux网络管理

### Linux配置IP地址的方法
```
1. ifconfig命令 临时配置（少用）
2. setup工具 永久配置（redhat系列特有的）
3. 修改网络配置文件
4. 图形界面配置（少用）
```
- ifconfig命令
	- lo本地回环网卡
	- 临时设置eth0网卡的IP与MASK
	```
	ifconfig eth0 192.168.0.200 netmask 255.255.255.0
	```
- setup工具设置完后service network restart
- 修改配置文件
	- 网卡信息文件
	```
	vim /etc/sysconfig/network-scripts/ifcfg-eth0
	
	DEVICE=eth0 #物理设备名称（要与文件名对应)
	BOOTPROTO=static #是否自动获取IP none|static|dhcp（如果设置dhcp则只需要配置DEVICE、BOOTPROTO、HWADDR、ONBOOT、TPYE、USERCTL其他内容不用配）
	HWADDR=00:0c:29:17:c4:09 #MAC地址
	NM_CONTROLLED=no #是否由Network Manager图形管理工具管理该网络接口。修改保存后立即生效，无需重启。但有坑，建议设置为no，只有安装了图形界面才有用
	ONBOOT=yes #是否随网络服务启动 yes|no
	TYPE=Ethernet #网卡协议类型
	UUID="ac9b66bf-74fb-4bda-b89f-c66ff84c9571" # 唯一识别码，如果镜像是克隆的则需要手动解决冲突
	
	IPADDR=192.168.1.245 # ip地址
	NETMASK=255.255.255.0 # ip对应的子网掩码
	GATEWAY=192.168.1.1 #ip对应的网关地址
	DNS1=192.168.1.1 #指定DNS1地址
	DNS2=192.168.1.2 #指定DNS2地址（有的话配，没有就不配置）
	
	USERCTL= no #非ROOT用户是否允许控制整个设备 yes|no，建议是no
	IPV6INIT=no #是否执行IPv6 yes:支持IPv6 no:不支持IPv6
	```
	- 主机名文件
	```
	vim /etc/sysconfig/network
	
	# 永久修改，需要重启计算机才生效
	# 如果想当时立即生效可以使用临时修改命令 hostname liyangpc
	NETWORKING=yes
	HOSTNAME=liyangpc
	```
	- DNS配置文件
	
	HOSTNAME=liyangpc
	```
	vim /etc/resolv.conf
	
	nameserver 202.106.0.20
	nameserver 202.106.0.21
	```

### 虚拟机网络配置
	- 配置linuxIP地址（如上）
	- 启动网卡
	```
	vim /etc/sysconfig/network-scripts/ifcfg-eth0
	改ONBOOT=yes
	
	service network restart
	```
	- 修改UUD（clone或拷贝的虚拟机需要做这步，如果自行安装的虚拟机不需要这步）
	```
	vim /etc/sysconfig/network-scripts/ifcfg-eth0
	删除MAC地址行
	
	rm -rf /etc/udev/rules.d/70-persistent-net.rules
	删除网卡和MAC地址绑定文件
	
	重启系统
	shutdown -r
	
	```
	- 设置虚拟机网络连接方式
		- 桥接：虚拟机作为一台主机直接用主机的物理网卡VMnet0通信，可以与局域网、公网都能通信
		- NAT：虚拟机用主机的虚拟网卡VMnet8进行通信，可与局域网通信，可以通过主机做的NAT访问公网
		- Host Only：虚拟机用主机的虚拟网卡VMnet1进行通信，只可与局域网通信，不能访问公网
	- 选桥接够修改虚拟机桥接到的具体网卡
		- 编辑-虚拟网络编辑器-桥接-选是无线网卡还是有线网卡


### 网络管理命令

- ifconfig命令
- ifdown、ifup命令
	```
	ifdown eth1
	ifup eth1
	```
- netstat命令
	```
	netstat 选项
	-t: 列出TCP协议端口
	-u: 列出UDP协议端口
	-n: 不使用域名和服务器名，而使用IP地址和端口号
	-l: 仅列出在监听状态的网络服务
	-a: 列出所有网络连接
	
	netstat -tuln
	netstat -an

	netstat -rn # 查看路由表和网关
	```
- route命令
	```
	route -n # 查看路由表可以看到网关
	```
- nslookup命令
	```
	nslookup www.baidu.com # 进行域名与ip的银蛇
	
	nslookup 再输入server #能看到dns服务器
	
	```
- tcpdump命令


