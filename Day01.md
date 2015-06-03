# Get Your PC Information

## PC情報漏洩対策として利用端末の基礎情報を取得する

**今回利用するコマンド**

* ifconfig /all
* systeminfo
* hostname|nslookup

## ifconfig /all

**利用端末の下記の情報を取得します**
- PC name
- IP Address
- MAC address

```
C:\Users\MyCo>ipconfig /all

Windows IP 構成

   ホスト名 . . . . . . . . . . . . : MyCowin7
   プライマリ DNS サフィックス . . . . . . . :
   ノード タイプ . . . . . . . . . . . . : ハイブリッド
   IP ルーティング有効 . . . . . . . . : いいえ
   WINS プロキシ有効 . . . . . . . . : いいえ
   DNS サフィックス検索一覧 . . . . . . : workgroup

イーサネット アダプター ローカル エリア接続:

   接続固有の DNS サフィックス . . . : workgroup
   説明. . . . . . . . . . . . . . . : Intel(R) 82579LM Gigabit Network Connecti
on
   物理アドレス. . . . . . . . . . . : 12-FE-B4-AD-7E-5D
   DHCP 有効 . . . . . . . . . . . . : はい
   自動構成有効. . . . . . . . . . . : はい
   リンクローカル IPv6 アドレス. . . . : fe60::80ec:a40c:c4aa:24d4%08(優先)
   IPv4 アドレス . . . . . . . . . . : 192.168.1.101(優先)
   サブネット マスク . . . . . . . . : 255.255.255.0
   リース取得. . . . . . . . . . . . : 2015年6月1日 12:37:49
   リースの有効期限. . . . . . . . . : 2015年6月4日 12:37:50
   デフォルト ゲートウェイ . . . . . : 192.168.1.254
   DHCP サーバー . . . . . . . . . . : 192.168.1.254
   DHCPv6 IAID . . . . . . . . . . . : 236256949
   DHCPv6 クライアント DUID. . . . . . . . : 00-01-00-01-1B-06-0D-FB-14-FE-B5-FD-6E-5C
   DNS サーバー. . . . . . . . . . . : 192.168.1.1
                                       192.168.1.2
   NetBIOS over TCP/IP . . . . . . . : 有効

Tunnel adapter isatap.workgroup:

   メディアの状態. . . . . . . . . . : メディアは接続されていません
   接続固有の DNS サフィックス . . . : workgroup
   説明. . . . . . . . . . . . . . . : Microsoft ISATAP Adapter
   物理アドレス. . . . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP 有効 . . . . . . . . . . . . : いいえ
   自動構成有効. . . . . . . . . . . : はい

Tunnel adapter Teredo Tunneling Pseudo-Interface:

   メディアの状態. . . . . . . . . . : メディアは接続されていません
   接続固有の DNS サフィックス . . . :
   説明. . . . . . . . . . . . . . . : Teredo Tunneling Pseudo-Interface
   物理アドレス. . . . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP 有効 . . . . . . . . . . . . : いいえ
   自動構成有効. . . . . . . . . . . : はい

```

## systeminfo

**利用端末の下記の情報を取得します**
- OS name
- OS version
- Maker name
- model number

```
C:\Users\MyCo>systeminfo

ホスト名:               MyCowin7
OS 名:                  Microsoft Windows 7 Professional 
OS バージョン:          6.1.7601 Service Pack 1 ビルド 7601
OS 製造元:              Microsoft Corporation
OS 構成:                スタンドアロン ワークステーション
OS ビルドの種類:        Multiprocessor Free
登録されている所有者:   mycom
登録されている組織:     
プロダクト ID:          00821-OEM-8993981-00496
最初のインストール日付: 2014/05/15, 14:44:59
システム起動時間:       2015/06/01, 12:37:13
システム製造元:         Dell Inc.
システム モデル:        OptiPlex 990
システムの種類:         x64-based PC
プロセッサ:             1 プロセッサインストール済みです。
                        [01]: Intel64 Family 6 Model 42 Stepping 7 GenuineIntel ~2574 Mhz
BIOS バージョン:        Dell Inc. A10, 2011/11/24
Windows ディレクトリ:   C:\Windows
システム ディレクトリ:  C:\Windows\system32
起動デバイス:           \Device\HarddiskVolume1
システム ロケール:      ja;日本語
入力ロケール:           ja;日本語
タイム ゾーン:          (UTC+09:00) 大阪、札幌、東京
物理メモリの合計:       16,265 MB
利用できる物理メモリ:   10,833 MB
仮想メモリ: 最大サイズ: 32,528 MB
仮想メモリ: 利用可能:   26,395 MB
仮想メモリ: 使用中:     6,133 MB
ページ ファイルの場所:  C:\pagefile.sys
ドメイン:               WORKGROUP
ログオン サーバー:      \\MyCowin7
ホットフィックス:       294 ホットフィックスがインストールされています。
                        [01]: KB2849697
                        .....
                        [245]: KB3020369
                        [24
ネットワーク カード:    4 NIC(s) インストール済みです。
                        [01]: Intel(R) 82579LM Gigabit Network Connection
                              接続名:		ローカル エリア接続
                              DHCP が有効:	はい
                              DHCP サーバー:	192.168.1.254
                              IP アドレス
                              [01]: 192.168.1.101
                              [02]: fe60::80ec:a40c:c4aa:24d4
```

## hostname|nslookup

**DNSに登録されているか、いないか を確認します**

```
C:\Users\MyCo>hostname|nslookup

既定のサーバー:  dns.workgroup
Address:  192.168.1.1

> サーバー:  dns.workslan
Address:  192.168.1.1

*** dns.workgroup が MyCowin7 を見つけられません: Non-existent domain

```
DNS Serverに登録されている場合は、PC name + IP Address が表示されます。
