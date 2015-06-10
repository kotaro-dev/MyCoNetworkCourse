# Send a http protocol with the telnet

## 最近の Windows では Telnet Client は OFF です。(default install)

**今回確認するポイント**

* Telnet Client の有効化 (windows)
* Telnet Client を使う   (http接続)
* Telnet Client の無効化 (後始末)
* 注意事項：Telnet Server は起動しない事

## Telnet Client を 有効化 する手順

**to enable the Telnet Client(GUI)**

```
[起動経路]
(左下)[スタート]->[コントロールパネル]->[プログラム]->[プログラムと機能]
->「windowsの機能の有効化または無効化」
[有効化]
「Telnet クライアント」を check する
(click [OK])

[注意]
「Telnet サーバー」は 有効化しない事
[理由]
外部から telnet 接続されてしまいます。
(firewallの設定等で、saftyが入っていますが、必要のないserverは起動させないこと！)
```

**to enable the Telnet Client(CLI)**

```
[CLI]
(コマンドプロンプト)
pkgmgr /iu:"TelnetClient" [Enter]
(load icon 状態になり、有効化処理を行います)

[ポイント]
"TelnetClient" は、大文字・小文字の区別をしっかり意識して入力。
```

## Telnet Client を 使ってみる

**Connect to the Apache2 Server with the Telnet Client**
- 接続先IP - 10.0.2.6
- 接続Port - 80 (default)
- User/Passwd - none

```
[Command Prompt]
telnet 10.0.2.6 80

(画面が telnet に切り替わる)
(入力できるが、入力内容が全く見えない)
GET /hello.html と入力したいが、入力ミス！
```

**Ctrl+] and set localecho**

```
[Command Prompt]
telnet 10.0.2.6 80

(画面が telnet に切り替わる)
[Ctrl]+[]] key で escape する

Microsoft Telnet> set localecho [enter]
ローカル エコー： オン
Microsoft Telnet> [enter] (戻る)

GET /hello.html[enter]
(今後は、local にも echo されるので見える！)

GET Request の結果が表示される
(hello.html の中身が表示される)
```

**telnet 10.0.2.6 http**

```
telnet 10.0.2.6 http
telnet 10.0.2.6 smtp
telnet 10.0.2.6 ssh

telnet は port を指定して server に接続できる 古くからある 道具です。

web sites へ接続できない状態が発生した際に
・http protocol の network 通信側に問題があるのか？
・browser のupdate 等による tool側に問題があるのか？
を、切り分ける際に 利用することが可能です。

最近は、Security の観点からも 利用を勧めるケースが少なくなりましたが
覚えておくと どうしようも無いときに、問題の切り分けに役立つかも知れません。
```

## Telnet Client を 無効化する

**to diable the Telnet Client(CLI)**

```
[CLI]
(コマンドプロンプト)
pkgmgr /uu:"TelnetClient" [Enter]
(load icon 状態になり、無効化処理を行います)

[ポイント]
再起動を促される場合があります。
可能なら再起動して Telnet Client を完全に無効化して下さい。
```

## Telnet Client の 注意点

```
Telnet 通信は 全て 平文です。
wireshark 等の packet caputure tool を利用すれば
通信内容は全て見ることができます。
password のような 秘匿性の高い 情報でチェックするのはしない様に。
```
