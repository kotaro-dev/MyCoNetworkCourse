# Apache2 Load Balancing Server - the Load Balancing Series

## Load Balancer を構築する - 負荷分散 - Series03

**今回確認するポイント**

* Reverse Proxy Serverを Load Balancer に変更する

**環境情報**

- [client] 10.0.2.4
- [Load Balancer]  10.0.2.7  <- change from [Reverse Proxy Server]  
  ubuntu14.04 + apache2 (port 80) + mod_proxy_balancer
- [Cache Server]  10.0.2.7  
  ubuntu14.04 + apache2 (port 80) + mod_cache
- [application server (main)] 10.0.2.6
- [application server (test)] 10.0.2.8  
  ubuntu14.04 + apache2 (port 80)

## Change a server structure with the Load Balancer

(before)
```
                 Front                 Back
 [client]+-----+[Reverse Proxy]+-----+[LAMP]
 [client]+     +[Cache]        +-----+[LAMP]
```

(change into)
```
                 Front                 Back
 [client]+-----+[Load Balancer]+-----+[LAMP]
 [client]+     +[Cache]        +-----+[LAMP]
```

```
[point]
・Reverse Proxy Server を Load Balancer として設定変更。
```

## how to use the proxy_balancer module - Apache2

***[add module]***

```
 sudo a2enmod proxy_balancer
```

***[add method module for load]***

sudo vim mods-enabled/proxy_balancer.load (edit by vim)
```
LoadModule lbmethod_byrequests_module /usr/lib/apache2/modules/mod_lbmethod_byrequests.so
```

***[proxy_balancer Setting]***
```
sudo vim mods-enabled/proxy.conf (edit by vim)
```

***(after  - setting for Load Balancing)***
```
ProxyPass /tnitter balancer://tnitter
ProxyPassMatch ^/(.*)$ balancer://tnitter/$1
ProxyPassReverse /tnitter balancer://tnitter
 
<Proxy balancer://tnitter>
 BalancerMember  http://10.0.2.6
 BalancerMember  http://10.0.2.8
 
 ProxySet lbmethod=byrequests
</Proxy>
 
SetOutputFilter INFLATE;proxy-html;DEFLATE
ProxyHTMLURLMap balancer://tnitter /tnitter
```

***(before - setting for Reverse Proxy)***
```
ProxyPass /tnitter http://10.0.2.8
ProxyPassMatch ^/(.*)$ http://10.0.2.8/$1
ProxyPassReverse /tnitter http://10.0.2.8
 
SetOutputFilter INFLATE;proxy-html;DEFLATE
ProxyHTMLURLMap http://10.0.2.8 /tnitter
```
```
sudo service apache2 restart
```

## Load Balancing Test
(change a HTML Title on the [application server (test)]10.0.2.8)

sudo vim /var/www/tinitter/templates/frame.twig

(main - original)
```
<header class="navbar navbar-inverse bs-docs-nav" role="banner">
    <div class="container">
        <div class="navbar-header">
            <a href="/" class="navbar-brand">Tinitter</a>
        </div>
    </div>
</header>
```

(test - change Line 15)
```
<header class="navbar navbar-inverse bs-docs-nav" role="banner">
    <div class="container">
        <div class="navbar-header">
            <a href="/" class="navbar-brand">Tinitter Test</a>
        </div>
    </div>
</header>
```

```
[補足]：
・[test]側は simpleに 【Tiniter】を【Tiniter Test】に変更しました。
・chrome 等の browser で http://10.0.2.7/tnitter/ にrequest。
->Reload を繰り返すと、【Tiniter】と【Tiniter Test】が交互に表示されます。
```

```
[注意]：
・この時点では、DB も 各LAMP 環境毎に持っているので、かなり変な状態。
  次のCourse で DB server を切り分けます。
```
