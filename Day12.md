# Isolate the DB Server - the Load Balancing Series

## DB Serverを分離する - 負荷分散 - Series04

**今回確認するポイント**

* DB Server を LAMP 環境から切り離し独立させる。

**環境情報**

- [client] 10.0.2.4
- [Load Balancer]  10.0.2.7  
  ubuntu14.04 + apache2 (port 80) + mod_proxy_balancer  
- [Cache Server]  10.0.2.7  
  ubuntu14.04 + apache2 (port 80) + mod_cache  
- [application server (main)] 10.0.2.6
- [application server (test)] 10.0.2.8  
  ubuntu14.04 + apache2 (port 80)  
- [DB server] 10.0.2.10  

## Change a server structure with the Load Balancer

(before)
```
                 Front                 Back
 [client]+-----+[Load Balancer]+-----+[LAMP]
 [client]+     +[Cache]        +-----+[LAMP]
```

(change into)
```
                 Front                 Back
 [client]+-----+                |+-----+[AP]+-----+|
 [client]+-----+[Load Balancer]+|                  |+-----+ [DB]
               +[Cache]         |+-----+[AP]+-----+|
```

```
[point]
・DB Server を LAMP 構成から 物理的に分離。
・mysql の場合、local 外から接続するには、別途権限を付与する必要がある。
```

## how to connect into the divide DB server - mysql (10.0.2.10)

***[add privileges]***

(connect - in the mysql server)
```
mysql -u root -pr00t
```
(check user conditions)
```
use mysql;
SELECT Host,User,Password,Select_priv,Insert_priv,Update_priv, Delete_priv FROM user;
```
(add all privileges to the tinitt user, it can access from every ip address. and flush DB.)
```
grant all privileges on *.* to tinitt@'%' identified by 'tinitt' with grant option;
flush privileges;
```

***[change the mysql.conf]***

sudo vim /etc/mysql/mysql.cnf  (edit by vim)
```
(L47) : comment out
#bind-address           = 127.0.0.1
```
```
Line 47 周辺の上記設定をコメントアウト。
```

## How to Check the change - mysql (10.0.2.10)
(use netstat command - show result)

***(before)***
```
@tnitterdb:~$ netstat -tlpn
 
 Active Internet connections (only servers)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
 tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
 tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN      -
 tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
                   ~~~~~~~~~~~~~~
```

***(after)***
```
@tnitterdb:~$ sudo netstat -tlpn
 
 Active Internet connections (only servers)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
 tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      845/sshd
 tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN      575/smbd
 tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      2744/mysqld
                   ~~~~~~~~~~~~~~
```

```
[解説]：
・netstat で mysql の local address の変化を確認。
->localhost 固定 だったのが、0.0.0.0 で 外部公開状態になっていることが分かる。
```

## Change AP Server setting
(change the db connect settings. main & test ap server)

sudo vim /var/www/tinitter/config.php (edit by vim)
```
$db_settings = [
    'driver'   => 'mysql',
#    'host'     => 'localhost',
    'host'     => '10.0.2.10',
    'database' => 'tinitt',
    'username'  => 'tinitt',
    'password'  => 'tinitt',
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
];
```

```
[説明]：
・'host' の部分を、 'localhost' から DB Server の '10.0.2.10' に変更するだけ。
・main ap も test ap も変更すること。
```

```
[解説]：
・これで、back side の AP server が交互に利用され、DB は共通の server を利用。
->Front の Load Balancer を、 DNS で domain 名登録する事で、更に冗長構成を構築できる。
```
