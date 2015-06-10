# Remote Desktop

## 初めてRemote Desktop を利用する際の確認ポイント

**今回利用 及び 確認するポイント**

* Remote Desktop connection
* Remote Desktop 設定
* firewall 設定 (プログラムの許可)
* firewall 設定 (詳細設定)
* firewall 設定 (scope)

## 一般的なRemote接続手順

**Remote Desktop connection を起動**
- 接続先IP - 10.0.2.5
- 接続Port - 3389 (default)
- User/Passwd - IEUser/Passw0ed! (modern.IE default)

```
[起動経路]
(左下)[スタート]->[アクセサリ]->[Remote Desktop Connection]
[接続]
Computer - 10.0.2.5:3389
(click [Connect])
User     - IEUser
Password - Passw0rd!
(click [OK])
```
## Remote Desktop connection が失敗した場合

*ここからは Remote先のWindowsPC の設定を確認します*

**Remote desktop の利用可否**

```
[起動経路]
[スタート]->[コンピューター](右click:property)->[リモートの設定]
[設定]
×「このコンピューターへの接続を許可しない」：Remote接続できません。
○「リモートデスクトップを実行しているコンピューターからの接続を許可する」に変更します。
```

**Firewall settings(プログラムの許可)**

```
[起動経路]
[スタート]->[コントロールパネル]->[システムとセキュリティ]->[Windowsファイアウォール]
->「windows ファイアウォールを介したプログラムまたは機能を許可する」
[設定]
「リモートディスクトップ」をチェックします。
(「設定の変更」をclickして変更します。)
```

**Firewall settings(詳細設定)**

```
[起動経路]
[スタート]->[コントロールパネル]->[システムとセキュリティ]->[Windowsファイアウォール]
->「詳細設定」->「受信の規制」
[設定]
「リモートディスクトップ(TCP受信)」
Grayed out の場合 「有効」に設定して Green mode に変更します。
「プロファイル」の違いで「ドメイン」「プライベート」「パブリック」と
３箇所設定が必要に見えますが、「プライベート」か「パブリック」で十分です。

(補足)
ここで RDP の port もチェックできます。
もし、RDP protocol の port を変更している場合は、
ここで firewall 規制で許可されている port も変更する必要があります。
```

## それでも接続できない場合

**Firewall settings(scope)**

```
[起動経路]
[スタート]->[コントロールパネル]->[システムとセキュリティ]->[Windowsファイアウォール]
->「詳細設定」->「受信の規制」
[設定]
「リモートディスクトップ(TCP受信)」(dubble click)
->[scope]タブ->[リモートアドレス]

[scope] の設定で、RDP protocol の許可範囲が狭められている場合があります。
security上許可されているならば、
「任意のIPアドレス」に設定を変更する事で、Remote接続が成功する場合があります。
```

## どうやっても接続できない場合

```
そもそも Remote PC が起動しているかどうか？
ping で確認してみてください。
また、tracert で network の確認も試して問題を切り分けて下さい。
```
