## ラボ環境

- ホスト:AMD RYZEN7 / 32GB RAM / VMware
- ゲスト:
 - VM1:Windows Server 2022 ADC01(ドメインコントローラ / Splunk Indexe)
 - VM2:Windows Server 2022 MEMBER01(ファイルサーバー / UF)
 - VM3:Windows11 WORKSTATION01(クライアント / UF)
- ログ収集：
 - 各ホストからADC01上のSplunk EnterpriseへWinEventLog(Security/System/Application)を転送

各VMのネットワークはホストオンリーとする


Splunk:Enterprise Free(ログ解析)
ADC01(192.168.2.130)
    Splunk Enterprise(Indexer+Search Header)
    Universal Forwader → MEMBER01/WORKSTATION01
    Windows Event Log(Security/System)収集
MEMBER01(192.168.229.137)
    Universal Forwader → ADC01送信
    Defenderログ
WORKSTATION01(192.168.2.132)
    Universal Forwader → ADC01送信
    Powershell BlockRule

## Splunkの導入
1. splunk.comからsplunkenterpriseをダウンロードしインストール
2. splunk.comからsplunkforwaderをダウンロードしインストール
3. firewall設定でoutboundとinboundの該当ポートを許可
    outbound:9997を許可
    inbound:8089を許可
    **Indexerの場合はinboundに9997の許可が必要**
4. splunkにログインし「設定」の「転送と受信」から「受信の設定」に入り、9997番ポートを追加する
5. outputs.confで転送先を設定する
今回は以下の設定（C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf）
```
[tcpout]
defaultGroup = splunk_indexer

[tcpout:splunk_indexer]
server = 192.168.2.130:9997

[forwarder]
indexAndForward = false

```
6. inputs.confで取得するデータを設定する
今回は以下の設定（C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf）
```
[WinEventLog://Security]
disabled = 0
start_from = oldest
index = windows
sourcetype = WinEventLog:Security

[WinEventLog://System]
disabled = 0
start_from = oldest
index = windows
sourcetype = WinEventLog:System

[WinEventLog://Application]
disabled = 0
start_from = oldest
index = windows
sourcetype = WinEventLog:Application
```
7. splunkforwaderを再起動する
コマンドプロンプトでprogramfiles\splunkforwader\bin\に移動し、以下コマンドを実行
```
splunk restart
```
8. インデックスの設定
splunkにログインし、「設定」から「インデックス」を選択し、新規インデックス(windows)を作成する。
9. splunkを再起動する
コマンドプロンプトでprogramfiles\splunk\bin\に移動し、以下コマンドを実行
```
splunk restart
```
10. splunkにログインしサーチからイベントログが取れているかを確認する。