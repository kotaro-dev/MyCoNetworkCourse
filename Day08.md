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
