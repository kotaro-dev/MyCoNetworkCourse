# Cache Server - the Load Balancing Series

## Cache Serverを用意する - 負荷分散 - Series02

**今回確認するポイント**

* 過負荷対策に負荷分散を考える
* Back End へのRequest 軽減用に Cache Server を構築する

**環境情報**

- [client] 10.0.2.4
- [Reverse Proxy Server]  10.0.2.7  
  ubuntu14.04 + apache2 (port 80) + mod_proxy
- [Cache Server]  10.0.2.7  
  ubuntu14.04 + apache2 (port 80) + mod_cache
- [application server (main)] 10.0.2.6
- [application server (test)] 10.0.2.8  
  ubuntu14.04 + apache2 (port 80)

## Change a server into the Plural server

(before)
```
                 Front                 Back
 [client]+-----+[Reverse Proxy]+-----+[LAMP]
 [client]+                     +-----+[LAMP]
 ```

(change into)
```
                 Front                 Back
 [client]+-----+[Reverse Proxy]+-----+[LAMP]
 [client]+     +[Cache]        +-----+[LAMP]
```

```
[point]
・Reverse Proxy Server を Cache Server としても機能させる
```

## how to use the cache module - Apache2

***[add module]***
```
 sudo a2enmod cache cache_disk
```

***[Cache Setting]*** (caching test with the test server)

/etc/apache2/mods-enabled/cache_disk.conf (edit by vim)
```
<IfModule mod_cache_disk.c>
       CacheRoot /var/cache/apache2/mod_cache_disk

       #add from here
       CacheEnable disk /
       CacheIgnoreCacheControl On
       CacheIgnoreHeaders Set-Cookie
       CacheDefaultExpire 86400
       CacheMaxExpire 172800
       #add to here

   CacheDirLevels 2
   CacheDirLength 1

</IfModule>
```
```
sudo service apache2 restart
```
```
[point]
・CacheRoot ： default のまま

・CacheEnable ：disk cache - [/] 以下の全てをcache
・CacheIgnoreCacheControl on ： client から no-cache が送られてもcacheする
・CacheIgnoreHeaders Set-Cookie：cache に cookie set を含めない
・CacheDefaultExpire 86400 ：3600 × 24 = 1 day 
・CacheMaxExpire 172800：3600 × 24 × 2 = 2 days 

・CacheDirLevels 2 ：Cache data の階層と構造を[CacheDirLength]との組合わせで決める
・CacheDirLength 1 ：
```

## Cache Test

(set a test php in the test server)

test.php
```
<?php
sleep(10);

header('Content-Type: text/plain; charset=UTF-8');
header('Cache-Control: max-age=60');

echo "Hello Test World!\n";
echo date("r");
```

```
[補足]：
・10s - sleep させて遅延を演出
・以下の動作を比較する
 (cache あり) http://10.0.2.7/test.php
 (cache なし) http://10.0.2.8/test.php
 
・cache あり・なし 両方とも １回目は10s かかる
  cache あり の２回目は、即 response が戻ってくる
  (ただし、date の結果は、１回目と同じ。時間が進んでいない)
  cache なし の２回目は、10s かかる。
  (date の結果は、正しく更新されている)
```
