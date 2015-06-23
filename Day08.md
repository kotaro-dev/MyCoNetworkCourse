# Stress test and Active connection check

## JMeterを使った負荷テストの手順と Application serverでのConnection確認方法

**今回確認するポイント**

* JMeter Clustering Setting (windows)
* Active connection 確認コマンド (netstat base)

**環境情報**

- [jmeter master] 10.0.2.4
- [jmeter slave]  10.0.2.5
- [jmeter slave]  10.0.2.4 (use for slave too)
- [application server] 10.0.2.8
  ubuntu14.04 + apache2 (port 8000)
  
## JMeter Clustering Setting

**Remote_hostsの編集**

([jmeter.properties]の編集 [L189])
```
(修正前)
# Remote Hosts - comma delimited
remote_hosts=127.0.0.1
#remote_hosts=localhost:1099,localhost:2010

# RMI port to be used by the server (must start rmiregistry with same port)
#server_port=1099
```
```
(修正後)
# Remote Hosts - comma delimited
remote_hosts=10.0.2.5,10.0.2.4
#remote_hosts=localhost:1099,localhost:2010

# RMI port to be used by the server (must start rmiregistry with same port)
server_port=1099
```

## Start Slave Service

**JMeter Slave start**

(in the jmeter slave pc.[10.0.2.5][10.0.2.4])

- slave 側 jmeter.properties 変更なし
- [jmeter-server.bat] を double click
- JMeter Master(Controller)からの指示待ち

## JMeter Master(Controller) Start Up

(in the jmeter master pc.[10.0.2.4])

- [jmeter.bat] を double click
- start up the GUI menu
- Run the all remote test

## Watch the activce connection

(in the Application Server.[10.0.2.8])

**one liner**

```
watch -n 1 "sudo netstat -ant | grep ':8000' | awk {'print $6 ,$5'}|sed -e 's/:[^.]*$//' |sort|uniq -c"
```

**make a shell and run it**

```
[conn_chk.sh]
#!/bin/sh
sudo netstat -ant | grep ':8000' | awk {'print $6 ,$5'}|sed -e 's/:[^.]*$//' |sort|uniq -c
```
run conn_chk.sh
```
watch -n 1 ./conn_chk.sh
```

result sample:
```
Every 1.0s: ./conn_chk.sh                               Mon Jun 22 16:42:49 2015

      8 CLOSE_WAIT 10.0.2.5
     35 ESTABLISHED 10.0.2.4
    231 ESTABLISHED 10.0.2.5
      1 LISTEN
    202 SYN_RECV 10.0.2.4
     23 SYN_RECV 10.0.2.5
      6 TIME_WAIT 10.0.2.5
```
explanation:
```
[count]   [state]     [foreign address]
      8   CLOSE_WAIT  10.0.2.5
     35   ESTABLISHED 10.0.2.4
```

## Watch the memory and each process

(in the Application Server.[10.0.2.8])

**[free] or [free -m]**

before the stress test:
```
@tnitter:~$ free
             total       used       free     shared    buffers     cached
Mem:       1017828     335792     682036       7152       1692      52636
-/+ buffers/cache:     281464     736364
Swap:      1044476      78000     966476
```

started and testing the http request:
```
@tnitter:~$ free
             total       used       free     shared    buffers     cached
Mem:       1017828     937544      80284       7156       1632      58568
-/+ buffers/cache:     877344     140484
Swap:      1044476      77996     966480
```

```
[point]：
[total]=[used]+[free]
・stress test を開始して負荷をかけると、[free] memory が減少して[used] が増加していく。
・free memoryを buffer/cached して高速化を図っているので実際の利用可能memory等の情報は２行目

・[-/+] となっているのは、[buffer/cached]の値を、1行目の[used]からは[-], [free]には[+] している意味

[Swap]：HDD上の仮想memory領域。main memoryが不足した場合に退避させるのに利用。
・もし、Swapが頻繁に利用されているなら、main memory の増強を検討すべき。

[参考]：
・machine 起動後に、free(２行目) が減少し続けている場合は、memory leak を疑うこと。
  その場合、下記のTOP コマンド等 でleakしている process を調査すべき。
```
**[top] -> [ctrl]+[m]**

before the stress test:
```
top - 14:32:39 up 5 days,  1:25,  3 users,  load average: 0.00, 2.23, 9.49
Tasks: 104 total,   1 running, 103 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   1017828 total,   335920 used,   681908 free,     1684 buffers
KiB Swap:  1044476 total,    78000 used,   966476 free.    52636 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 1758 ntp       20   0   31448   1256   1084 S  0.3  0.1   0:46.74 ntpd
    1 root      20   0   33696   2368   1252 S  0.0  0.2   0:16.53 init
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd
    3 root      20   0       0      0      0 S  0.0  0.0   0:07.51 ksoftirqd/0
    4 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0
    5 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H
    7 root      20   0       0      0      0 S  0.0  0.0   0:17.74 rcu_sched
    8 root      20   0       0      0      0 S  0.0  0.0   0:53.35 rcuos/0
    9 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh
   10 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcuob/0
```

started and testing the http request:
```
top - 14:35:50 up 5 days,  1:28,  3 users,  load average: 5.79, 2.40, 8.13
Tasks: 195 total, 103 running,  92 sleeping,   0 stopped,   0 zombie
%Cpu(s): 85.3 us, 12.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  2.7 si,  0.0 st
KiB Mem:   1017828 total,   719076 used,   298752 free,     6324 buffers
KiB Swap:  1044476 total,    77996 used,   966480 free.   108044 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 1592 root      20   0  318208   7496   6128 R  2.3  0.7   0:36.43 apache2
 4789 www-data  20   0  320948  13124   8076 R  1.7  1.3   0:00.10 apache2
 4795 www-data  20   0  320948  13124   8076 R  1.7  1.3   0:00.10 apache2
 4810 www-data  20   0  320948  13108   8076 R  1.7  1.3   0:00.06 apache2
 4317 www-data  20   0  320948  13144   8084 R  1.3  1.3   0:00.59 apache2
 4319 www-data  20   0  320952  13148   8088 R  1.3  1.3   0:00.56 apache2
 4342 www-data  20   0  320948  13140   8084 R  1.3  1.3   0:00.54 apache2
 4434 www-data  20   0  320948  13140   8084 R  1.3  1.3   0:00.48 apache2
 ```

```
[point]:
・TOP は実行中の process の一覧を表示させるコマンド
  上部にmemory関連の情報を表示。
  [ctrl]+[m](大文字の[M]) で memory利用量の大きい順に並び替えている。
・apache へ stress をかけた瞬間に、一気にapache2 process で埋め尽くされた感じ。  
```
