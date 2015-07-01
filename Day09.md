# Reverse Proxy Server - the Load Balancing Series

## Front と Back に分ける - 負荷分散 - Series01

**今回確認するポイント**

* 過負荷対策に負荷分散を考える
* Reverse Proxy Server を構築する

**環境情報**

- [client] 10.0.2.4
- [Reverse Proxy Server]  10.0.2.7  
  ubuntu14.04 + apache2 (port 80) + mod_proxy
- [application server (main)] 10.0.2.6
- [application server (test)] 10.0.2.8  
  ubuntu14.04 + apache2 (port 80)

## Change a server into the Plural server

(before)
```
                 Front&End
 [client]+-----+[LAMP server]
 [client]+
```

(change into)
```
                 Front                 Back
 [client]+-----+[Reverse Proxy]+-----+[LAMP]
 [client]+                     +-----+[LAMP]
```
```
[point]
・One Server に [httpd][Web App][mysql] を構築している環境を分解していく。
・この Course では、[httpd] と [Web App][mysql] を分ける。
・[Reverse Proxy] とCall しているが、最終的に「Load Balancing」及び「Caching」を行う。

[補足]
・筐体単位に役割を分解することで、パフォーマンス劣化に耐える構成を準備
```
## how to use the proxy module - Apache2

***[add module]***
```
 sudo a2enmod proxy proxy_http proxy_html
```
```
[point]
・/etc/apache2/mods-avalilable/ 以下にある module を
  /etc/apache2/mods-enabled/    以下に link として追加
  module がない場合は apt-get で install から実施する必要あり
```

***[Reverse Proxy Setting]*** (proxy into the test server)

/etc/apache2/mods-enabled/proxy.conf (edit by vim)
```
<IfModule mod_proxy.c>
 # add from here
 ProxyPass /tnitter/ http://10.0.2.8/
 ProxyPassReverse /tnitter/ http://10.0.2.8/
 SetOutputFilter INFLATE;proxy-html;DEFLATE
 ProxyHTMLURLMap http://10.0.2.8 /tnitter
 # add to here
</IfModule>
```

/etc/apache2/mods-enabled/proxy_html.load (edit by vim too)
```
 # add from here
 LoadFile /usr/lib/x86_64-linux-gnu/libxml2.so.2
 # add to here
```
```
sudo service apache2 restart
```
```
[point]
・load する lib module は 環境によりことなるため
  ls -alF /usr/lib/x86_64-linux-gnu|grep libxml
  と言うように、自分で確認すべき
```

```
[補足]
・今回は面倒なので apache を Reverse Proxy Serve としてます。
・パフォーマンスをもっと考慮するなら nginx を front に、apache を App 用 に。
```

## How can get the CSS and the other path

***set each path by the Pattern Match***

/etc/apache2/mods-enabled/proxy.conf (edit by vim again)
```
<IfModule mod_proxy.c>
 ProxyPass /tnitter/ http://10.0.2.8/
 ProxyPassReverse /tnitter/ http://10.0.2.8/
 
 # add from here
 ProxyPassMatch ^/(.*)$ http://10.0.2.8/$1
 # add to here
 
 SetOutputFilter INFLATE;proxy-html;DEFLATE
 ProxyHTMLURLMap http://10.0.2.8 /tnitter
</IfModule>
```

```
[point]
・プログラム上該当する 相対path 分[ProxyPass][ProxyPassReverse]を追記のは現実的でない。
・上記の Pattern Matching の効果で、/ (root) も、正常にredirect されるようになる。
```
