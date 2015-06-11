# Update the Apache Port and how to check it
  Change a Port? Check the Port!

## Apache の Port 設定変更手順 と 反映確認方法

**今回確認するポイント**

* Apache port conf 変更   (ubuntu14.04)
* Apache 利用Port確認     (netstat -an)
* local & foreign address (in to out)

**環境情報**

- [client] 10.0.2.4
- [server] 10.0.2.6
  ubuntu14.04 + apache2 (default setting)

## Apache の Port を変更 (80 -> 81)

**How to update the apache2 conf files**

(login apache server with teraterm)
```
(alias ll='ls -alF')
@tnitter:~$ ll /etc/apache2
total 88
drwxr-xr-x  8 root root  4096 Jun 11 16:29 ./
drwxr-xr-x 86 root root  4096 Jun 11 07:11 ../
-rw-r--r--  1 root root  7239 May 21 15:45 apache2.conf
drwxr-xr-x  2 root root  4096 May 12 16:20 conf-available/
drwxr-xr-x  2 root root  4096 May 12 16:20 conf-enabled/
-rw-r--r--  1 root root  1782 May 13 16:01 envvars
-rw-r--r--  1 root root 31063 Jan  3  2014 magic
drwxr-xr-x  2 root root 12288 Jun  4 12:03 mods-available/
drwxr-xr-x  2 root root  4096 May 21 15:07 mods-enabled/
-rw-r--r--  1 root root   333 Jun 11 13:32 ports.conf
drwxr-xr-x  2 root root  4096 Jun 11 13:35 sites-available/
drwxr-xr-x  2 root root  4096 May 13 16:29 sites-enabled/
@tnitter:~$ cd /etc/apache2
@tnitter:/etc/apache2$ sudo vim ports.conf
```
ports.conf (edit by vim)
```
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

#Listen 80
Listen 81  <- {change port from 80}

<IfModule ssl_module>
        Listen 443
</IfModule>
```
```
@tnitter:/etc/apache2$ ll sites-available
total 20
drwxr-xr-x 2 root root 4096 Jun 11 13:35 ./
drwxr-xr-x 8 root root 4096 Jun 11 18:00 ../
-rw-r--r-- 1 root root 1375 Jun 11 13:32 000-default.conf
-rw-r--r-- 1 root root 6538 May 13 16:38 default-ssl.conf
@tnitter:/etc/apache2$ sudo vim sites-available/000-default.conf
```
000-default.conf (edit by vim too)
```
<VirtualHost *:81>  <- {change here into 81}
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        ・・・・
</VirtualHost>
```
restart (for reflect)
```
@tnitter:/etc/apache2$ sudo service apache2 restart
 * Restarting web server apache2                                         [ OK ]
```

**Chack using port number by the apache2**

```
@tnitter:/etc/apache2$ netstat -an -46|grep 81
tcp6       0      0 :::81                   :::*                    LISTEN
```
```
[解説]
Apache の設定file を更新しただけでは変更が反映されない。
再起動を行い、Port の変更を反映させること。

Port変更が正しく反映されたかを確認するために
netstat tool を用いて確認する。
81 Port が Listen 状態で 待ち受けていれば
Apache が 81 番 Port で正常に稼動していることが示せる。

[補足]
Linux と Windows では netstat の引数が異なる。
linux の場合 -an のみでは unix domain socket の情報も表示されてしまうので
netstat -an -46 と IPv4 と IPv6 の情報に絞り込む。
```

## netstat の local address & foreign address

**Watch port routing with ssh protocol**

```
[network layout]

            -> port 22
[10.0.2.4]+-----------------+[10.0.2.6]
            <- port 56705

Teraterm:ssh                ssh server:sshd

connect port 22 (request)-> (accept) port 22

catch return port 56705  <- (return) port 56705
                            (temporary)
```
[ssh server]
```
sasaki@tnitter:/etc/apache2$ netstat -an -46|grep 22
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 10.0.2.6:22             10.0.2.4:56705          ESTABLISHED
tcp        0      0 10.0.2.6:22             10.0.2.4:56707          ESTABLISHED
tcp6       0      0 :::22                   :::*                    LISTEN
```
[ssh client]
```
C:\Users\IEUser>netstat -ano|findstr 22
  TCP    10.0.2.4:56705         10.0.2.6:22            ESTABLISHED     1804
  TCP    10.0.2.4:56707         10.0.2.6:22            ESTABLISHED     1720
```
```
[解説]
Server 側で [ESTABLISHED]状態 ２つのport が
Client 側の ２つの teraterm と接続して通信を確立している。

Server 側の [LISTEN]状態 の process(IPv4/IPv6) は
Passive に 次の ssh 接続要求を待機している状態

[補足]
ssh  で port の状態を説明したのは、接続が維持されている状況が見やすいため。
http でも同様の検証はできるが、Client側は [TIME_WAIT] のはず。
```
