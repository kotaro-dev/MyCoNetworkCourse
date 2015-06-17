# isolate the problem

## 問題の切り分け手順と判断理由

**今回のポイント**

* OSI Reference Model
* TCP/IP Model
* port
* check syslog and error log

## 基本的な調査手順  

*利用しているweb service が急に表示されなくなった場合を想定*

#### [1] ping - start position

- reason for ping -> ([3] ping divide into two groups - backyard)

*backgroundを学ぶ理由*

```
「何故最初に[ping]を実行すべきか？」の説明を
Network Model を背景に記載しています。
Trouble Shooting は、問題箇所 及び 原因を切り分けていく作業です。
Network構成図を用いて、平面的に問題箇所を切り分け、
Network Modelを理解して、階層化されたデータの流れを意識して下さい。
```

- ping success -> ([1-1] connect into the server)

*ping success sample*
```
C:\Users\IEUser>ping 10.0.2.6

10.0.2.6 に ping を送信しています 32 バイトのデータ:
10.0.2.6 からの応答: バイト数 =32 時間 =1ms TTL=64
10.0.2.6 からの応答: バイト数 =32 時間 <1ms TTL=64
10.0.2.6 からの応答: バイト数 =32 時間 <1ms TTL=64
10.0.2.6 からの応答: バイト数 =32 時間 <1ms TTL=64

10.0.2.6 の ping 統計:
    パケット数: 送信 = 4、受信 = 4、損失 = 0 (0% の損失)、
ラウンド トリップの概算時間 (ミリ秒):
    最小 = 0ms、最大 = 1ms、平均 = 0ms
```

- ping error   -> ([2] tracert)

*ping error sample*
```
C:\Users\IEUser>ping 10.0.2.5

10.0.2.5 に ping を送信しています 32 バイトのデータ:
要求がタイムアウトしました。
要求がタイムアウトしました。
要求がタイムアウトしました。
要求がタイムアウトしました。

10.0.2.5 の ping 統計:
    パケット数: 送信 = 4、受信 = 0、損失 = 4 (100% の損失)、
```

#### [1-1] connect into the server with RDP or ssh (ping success)

- connet success -> ([1-1-1] check the service and show the log)

- connet error   -> ([1-1-2] check the used port and stand up to go the server room)

###### [1-1-1] check the service and show the log (connect success)

```
tasklist
service list (net start) (sc query type= service|findstr DISPLAY_NAME)
```

```
[説明]
まず、「該当の service が停止していないか？」を確認します
次に、「その service のlog を checkします」(特にerror.log)
さらに「OS の system log も checkします」

併せて「OS と service のperformance もcheckです」

時間的余裕があれば、connection 数 と session 数 memory使用量、CPU負荷等
気になる要素を、時間軸でチェックできれば原因解明に役立つと思います。

とは言え、緊急に対応する必要がある場合、
せめて 上記のlog のbackup と tasklist 等でperformance の状態の記録を取得した上で
再起動(最終手段) を行ってください。

再起動後に、正常に接続できるか？
serviceのlog がerrorを出力していないか？
など、起動後のチェックも忘れないようにしましょう。
```

###### [1-1-2] check the used port and stand up to go the server room (connect error)

port check

```
[説明]
Security 目的で RDP や ssh の接続port が変更されている可能性があります。
社内的に 許可されているなら port scan で open な port をcheckする手があります。
```

Walk into the server room!

```
[説明]
ServerにRemoteで接続できないのであれば
もはや、Server Roomまで行くしかありません！
Server Room で Monitor を使い問題解消して下さい。
```

#### [2] tracert - (ping error)

**tracert**
**ipconfig /all |ifconfig (ip a)**

- inside the gateway      -> ([2-1] client area)
- loss between the router -> ([2-2] middle area)
- get the server gateway  -> ([2-3] server area)

*tracert sample*
```
C:\Users\IEUser>tracert 10.0.3.6

apache-svr [10.0.3.6] へのルートをトレースしています
経由するホップ数は最大 30 です:

  1    <1 ms     1 ms     1 ms  10.0.2.1
  2    <1 ms    <1 ms    <1 ms  10.0.3.1
  3     1 ms    <1 ms    <1 ms  apache-svr [10.0.3.6]

トレースを完了しました。
```
```
[説明]
上記のsampleの場合
[Client Area] - 10.0.2.1 まで到達しない場合
[Middle Area] - 10.0.3.1 まで到達しない場合
[Server Area] - 10.0.3.6   に到達しない場合
```

###### [2-1] client area

Client area - 自端末のGatewayまでのArea
```
解決すべき人：自己解決すべき！
要因：自端末 もしくは 周辺の機器の影響。

参考事例：
・note pc - 他のnetworkで作業。MTGのため移動。繋がらない！(と騒ぐ) 
  [対応]：ipocnfig /release -> ipconfig /renew
・自分だけ Internetに接続できない。(と騒ぐ)
  [対応]：ipconfig /all で情報が取得できるか確認。
  取得できない場合：LAN線等 物理的な要因をCheck
  取得できる場合：ipconfig の結果を丁寧にみる。(IP address が周りと違わないか？)
   ->固定IP。無許可DHCP。など
```

###### [2-2] middle area

Middle Area - 自端末のGatewayからServer AreaのGatewayの間
```
相談相手：Network管理者に相談するArea
要因：Router 等の設定の可能性が高い。

参考事例：
・iptable の設定(変更)ミス - 入力ミス。意図しないloop設定
  [対応]：再設定したら、きちんと疎通確認して下さい。
・pingは通るが、http/https で通信できない。
  [対応]：通信の目的もきちんと把握して設定お願いします。
```

###### [2-3] server area

Server Area - Server AreaのGatewayからServer まで
```
解決すべき人：Server管理者が解決して下さい。
要因：Server PC もしくは その周辺機器の影響。

参考事例：
・IPの変更 - DHCP 設定で、IP が変更されてしまった。
  [対応]：固定IP + DNS。
・PC server 移設 - これまで同一Networkで作業。設定後移設。繋がらない？
  [POINT]：windows の場合：firewall の scope 設定の影響を確認

```

#### [3] ping divide into two groups - backyard

**OSI Reference Model & TCP/IP Model**

```
   [OSI Reference Model]    [TCP/IP Model]

       [7]|Application |    |Application      |  APDU
       [6]|Presentation|    |                 |  PPDU
       [5]|Session     |    |                 |  SPDU
 
       [4]|Transport   |    |Transport        |  TPDU
    -----------------------------------------------
[ping] [3]|Network     |    |Internet         |  packet
 
       [2]|Data link   |    |Network Interface|  Frame
       [1]|Physical    |    |                 |  Bit
```

*説明１:ping work on the layer 3*
```
ping は layer3 で packet をやり取りします。

ping が 通る(疎通確認OK) ＝ layer3 でnetworkが繋がっている事を証明しています。
この場合、web page が参照できない問題の原因は、上位のlayer にあることになります。

疎通Errorの場合、原因は、networkを構成している機器や設定の問題です。
経路のどこでpacket loss しているのかを tracert で check すべきです。

このように、ping だけで、
「Web Service提供側にあるか？」
「Infra-Network設置側にあるか？」
大まかに切り分けることができます。
これは、原因調査の相談先(依頼先)の切り分けでもあります。
```

**Data Flow & OSI Reference Model**

```
       [7] |browser|                        |Apache2|+---+|php|+---+|mysql|
       [6]     │                               ↑
       [5]     │                               │
               │                               │
       [4]     │                               │
               │                               │
    -----------------------------------------------
[ping] [3]     │   [router]         [router]   │
               │    ↑   │           ↑     │    │
       [2]     │    │   │           │     │    │
       [1]     │    │   │           │     │    │
               ↓    │   ↓           │     ↓    │
          [ethernet]┘   └[fiber    ]┘     └[ethernet]
                         [satellite]

        [client Area]   [middle Area]     [server Area]
```

*説明２:Data Flow and Responsible Area*
```
ping error ≠ 「Infra担当者へ丸投げ」
あなたが、プロ(少なくともIT関連の社員)なら、まだ調べることがあります。
丸投げしてはいけません！

tracert の結果で、大きく下記３つのAreaに分けます。

Area分割：
[Client Area] - 自端末のgatewayまで到達してるか？
[Middle Area] - 自端末のgatewayからserverのgatewayの間(上記２つのAreaの間)
[Server Area] - serverの属するnetworkのgatewayまで到達しているか？

責任範囲：
[Client Area] - 自端末 もしくは 周辺の機器の影響。自己解決して下さい。
[Middle Area] - Router 等の設定の可能性が高い。network管理者に相談です。
[Server Area] - server PC もしくは その周辺機器の影響。Server管理者が解決して下さい。
```
