## 网络测量工具
#### fping测lossrate和rta
- 下载解压fping-3.10.tar.gz
- 下载解压check_fping：https://exchange.nagios.org/directory/Plugins/Network-Protocols/ICMP/check_fping/details
- 用法：./check_fping -4 -H #desip# -b 64 -n 100 -i 20

```
./check_fping -h
check_fping v2.2.1 (nagios-plugins 2.2.1)
Copyright (c) 1999 Didi Rieder <adrieder@sbox.tu-graz.ac.at>
Copyright (c) 2000-2014 Nagios Plugin Development Team
	<devel@nagios-plugins.org>

This plugin will use the fping command to ping the specified host for a fast check
Note that it is necessary to set the suid flag on fping.


Usage:
 check_fping <host_address> -w limit -c limit [-b size] [-n number] [-T number] [-i number]

Options:
 -h, --help
    Print detailed help screen
 -V, --version
    Print version information
 --extra-opts=[section][@file]
    Read options from an ini file. See
    https://www.nagios-plugins.org/doc/extra-opts.html
    for usage and examples.
 -4, --use-ipv4
    Use IPv4 connection
 -6, --use-ipv6
    Use IPv6 connection
 -H, --hostname=HOST
    name or IP Address of host to ping (IP Address bypasses name lookup, reducing system load)
 -w, --warning=THRESHOLD
    warning threshold pair
 -c, --critical=THRESHOLD
    critical threshold pair
 -b, --bytes=INTEGER
    size of ICMP packet (default: 56)
 -n, --number=INTEGER
    number of ICMP packets to send (default: 1)
 -T, --target-timeout=INTEGER
    Target timeout (ms) (default: fping's default for -t)
 -i, --interval=INTEGER
    Interval (ms) between sending packets (default: fping's default for -p)
 -S, --sourceip=HOST
    name or IP Address of sourceip
 -I, --sourceif=IF
    source interface name
 -v, --verbose
    Show details for command-line debugging (Nagios may truncate output)

 THRESHOLD is <rta>,<pl>%% where <rta> is the round trip average travel time (ms)
 which triggers a WARNING or CRITICAL state, and <pl> is the percentage of
 packet loss to trigger an alarm state.

 IPv4 is used by default. Specify -6 to use IPv6.

Send email to help@nagios-plugins.org if you have questions regarding use
of this software. To submit patches or suggest improvements, send email to
devel@nagios-plugins.org

```


#### iperf测瓶颈带宽
- 被测两端都下载解压iperf-2.0.5-source.tar.gz
- 编译：

```
./configure 
make 
make install
```
- 下载解压check_iperf.pl
- 用法：

```
被测试端：
关掉防火墙
iperf -s

测试端：
./check_iperf.pl 173.199.122.225 1 4

[root@ovs ~]# ./check_iperf.pl -h
 ./check_iperf.pl <target host> "Critical speed" "Warning speed": 
 	this plugin returns iperf usage

 examples:
	./check_iperf.pl <host> 10 15: critical if speed under 10 units
	./check_iperf.pl <host> 10 15:50: warning if speed out of 15:50 units
	./check_iperf.pl <host> 10:100 15:90
	./check_iperf.pl <host> 10:40 15:90

```

## 测量结果
#### VULTR
###### New York

```
Location: New Jersey
CPU: 1 vCore
RAM: 1024 MB
Storage: 25 GB SSD
Bandwidth: 0 GB of 1000 GB
OS: CentOS 6 x64

[root@ovs ~]# ./check_fping -4 -H dstip -b 64 -n 100 -i 20
FPING OK - dstip (loss=0%, rta=312.000000 ms)|loss=0%;;;0;100 rta=0.312000s;;;0.000000

[root@ovs ~]# ./check_iperf.pl dstip 1 4
DEBUG: /usr/local/bin/iperf -c dstip -m
DEBUG: '[  3]  0.0-10.0 sec  4.75 MBytes  3.98 Mbits/sec' ; '3.98', 'Mbits/sec' 
Warning: iperf speed of 'dstip': 3.98 (Mbits/sec) < 4.

```

###### Chicago

```
Location: Chicago
CPU: 1 vCore
RAM: 1024 MB
Storage: 25 GB SSD
Bandwidth: 0 GB of 1000 GB
OS: CentOS 7 x64

[root@ovs ~]# ./check_fping -4 -H dstip -b 64 -n 100 -i 20
FPING OK - dstip (loss=1%, rta=322.000000 ms)|loss=1%;;;0;100 rta=0.322000s;;;0.000000

[root@ovs ~]# ./check_iperf.pl dstip 1 4
DEBUG: /usr/local/bin/iperf -c dstip -m
DEBUG: '[  3]  0.0-10.1 sec  3.75 MBytes  3.12 Mbits/sec' ; '3.12', 'Mbits/sec' 
Warning: iperf speed of 'dstip': 3.12 (Mbits/sec) < 4.

```

###### Miami

```
Location: Miami
CPU: 1 vCore
RAM: 1024 MB
Storage: 25 GB SSD
Bandwidth: 0 GB of 1000 GB
OS: CentOS 7 x64

[root@ovs ~]# ./check_fping -4 -H dstip -b 64 -n 100 -i 20
FPING OK - dstip (loss=0%, rta=321.000000 ms)|loss=0%;;;0;100 rta=0.321000s;;;0.000000

[root@ovs ~]# ./check_iperf.pl dstip 1 4
DEBUG: /usr/local/bin/iperf -c dstip -m
DEBUG: '[  3]  0.0-10.0 sec  3.12 MBytes  2.61 Mbits/sec' ; '2.61', 'Mbits/sec' 
Warning: iperf speed of 'dstip': 2.61 (Mbits/sec) < 4.

```

###### Atlanta

```
Location: Atlanta
CPU: 1 vCore
RAM: 1024 MB
Storage: 25 GB SSD
Bandwidth: 0 GB of 1000 GB
OS: CentOS 7 x64

[root@ovs ~]# ./check_fping -4 -H dstip -b 64 -n 100 -i 20
FPING OK - dstip (loss=0%, rta=261.000000 ms)|loss=0%;;;0;100 rta=0.261000s;;;0.000000

[root@ovs ~]# ./check_iperf.pl dstip 1 4
DEBUG: /usr/local/bin/iperf -c dstip -m
DEBUG: '[  3]  0.0-10.2 sec  6.88 MBytes  5.67 Mbits/sec' ; '5.67', 'Mbits/sec' 
OK: iperf returns 5.67 Mbits/sec (wsize 18.9 K).

```

###### Tokyo

```
Location: Tokyo
CPU: 1 vCore
RAM: 512 MB
Storage: 20 GB SSD
Bandwidth: 0 GB of 1000 GB
OS: CentOS 7 x64

[root@ovs ~]# ./check_fping -4 -H dstip -b 64 -n 100 -i 20
FPING OK - 45.63.122.110 (loss=0%, rta=155.000000 ms)|loss=0%;;;0;100 rta=0.155000s;;;0.000000

[root@ovs ~]# ./check_iperf.pl dstip 1 4
DEBUG: /usr/local/bin/iperf -c dstip -m
DEBUG: '[  3]  0.0-10.4 sec  26.5 MBytes  21.4 Mbits/sec' ; '21.4', 'Mbits/sec' 
OK: iperf returns 21.4 Mbits/sec (wsize 18.9 K).

```