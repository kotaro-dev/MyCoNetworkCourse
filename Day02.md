# Trouble shooting in the apache server

## apache が停止した場合のTrouble調査手順

**今回利用する道具**

* browser
* ping
* tracert
* teraterm (connect into the apache server)

## before trouble occurrence

**正常な稼動状況で動作を確認**
- browser - URL: http://192.168.1.101/
- ping    - ping 192.168.1.101
- tracert - tracert 192.168.1.101

- browser -> you can see your application first-page.
- ping    -> no packet loss
- tracert -> you traced into the server

```
C:\Users\MyCo>ping 192.168.1.101

192.168.1.101 に ping を送信しています 32 バイトのデータ:
192.168.1.101 からの応答: バイト数 =32 時間 =1ms TTL=63
192.168.1.101 からの応答: バイト数 =32 時間 =1ms TTL=63
192.168.1.101 からの応答: バイト数 =32 時間 =1ms TTL=63
192.168.1.101 からの応答: バイト数 =32 時間 =1ms TTL=63

192.168.1.101 の ping 統計:
    パケット数: 送信 = 4、受信 = 4、損失 = 0 (0% の損失)、
ラウンド トリップの概算時間 (ミリ秒):
    最小 = 1ms、最大 = 1ms、平均 = 1ms

C:\Users\MyCo>tracert 192.168.1.101

ZEROEXAM [192.168.1.101] へのルートをトレースしています
経由するホップ数は最大 30 です:

  1    <1 ms    <1 ms    <1 ms  192.168.2.254
  2     1 ms     1 ms     1 ms  ZEROEXAM [192.168.1.101]

トレースを完了しました。
```

## someone stopped apache 

**you didn't know what someone stop one.**
[someone]
- teraterm - connect into 192.168.1.101

```
someone@zeroexam:~$ sudo service apache2 stop
 * Stopping web server apache2                                                  
 *
```

## investigate the point of trouble

**browser で web page が見れなくなったので調査**

- browser -> you can't see your application web page.
- ping    -> no packet loss (same as before)
- tracert -> you traced into the server (same as before)

## recovery apache server

**ping が通っているので、remote接続できるか試す**

-teraterm -> connect into 192.168.1.101 (using ssh)

**server に login 後 sever 状況をチェック**

[investigation]
- check apache process    -> ps -ef|grep apache2
- check memory condition  -> free (or you can use [top] command)
- check hard disk volume  -> df -h
- check apache2 error.log -> sudo vim /var/log/apache2/error.log ([shift]+[g] into bottom)

```
MyCo@zeroexam:~$ ps -ef|grep apache2
MyCo    2786  1802  0 15:43 pts/0    00:00:00 grep --color=auto apache2
```
```
MyCo@zeroexam:~$ free
             total       used       free     shared    buffers     cached
Mem:       1017828     675352     342476       5872      93084     331248
-/+ buffers/cache:     251020     766808
Swap:      1044476          0    1044476
```
```
MyCo@zeroexam:~$ df -h
Filesystem                          Size  Used Avail Use% Mounted on
/dev/mapper/ubuntupreseed--vg-root  7.9G  2.4G  5.2G  32% /
none                                4.0K     0  4.0K   0% /sys/fs/cgroup
udev                                487M  4.0K  486M   1% /dev
tmpfs                               100M  500K   99M   1% /run
none                                5.0M     0  5.0M   0% /run/lock
none                                497M     0  497M   0% /run/shm
none                                100M     0  100M   0% /run/user
/dev/sda1                           236M   97M  127M  44% /boot
```
```
MyCo@zeroexam:~$ sudo vim /var/log/apache2/error.log
...
[Thu Jun 04 12:56:30.343910 2015] [mpm_prefork:notice] [pid 2629] AH00163: Apache/2.4.7 (Ubuntu) 
PHP/5.5.9-1ubuntu4.9 OpenSSL/1.0.1f configured -- resuming normal operations
[Thu Jun 04 12:56:30.343979 2015] [core:notice] [pid 2629] AH00094: Command line: '/usr/sbin/apache2'
[Thu Jun 04 15:43:41.000078 2015] [mpm_prefork:notice] [pid 2629] AH00169: caught SIGTERM, shutting down

([shift]+[q] -> [q!] で vim終了)
```

**apache の停止を確認、memoryやhdd fullではなく停止されただけなので再起動**

```
MyCo@zeroexam:~$ sudo service apache2 start
 * Starting web server apache2                                                  
 *
```
```
MyCo@zeroexam:~$ ps -ef|grep apache2
root      2816     1  0 15:52 ?        00:00:00 /usr/sbin/apache2 -k start
www-data  2820  2816  0 15:52 ?        00:00:00 /usr/sbin/apache2 -k start
www-data  2821  2816  0 15:52 ?        00:00:00 /usr/sbin/apache2 -k start
www-data  2822  2816  0 15:52 ?        00:00:00 /usr/sbin/apache2 -k start
www-data  2823  2816  0 15:52 ?        00:00:00 /usr/sbin/apache2 -k start
www-data  2824  2816  0 15:52 ?        00:00:00 /usr/sbin/apache2 -k start
MyCo    2830  1802  0 15:53 pts/0    00:00:00 grep --color=auto apache2
```

## (Restart apache) See the web page in your client

**Client に 戻って browser で Web Pageを開く**

- browser -> you can see your application web-page. (again)

