# 公務員CSIRTのAD攻撃検知ラボ

## 目的
公務員CSIRT経験者として、民間企業CSIRTリーダー転職を見据えた**AD攻撃→Splunk検知の実務代替ラボ**を構築

## 環境構成
ホスト:AMD RYZEN7 + RAM 32GB + VMware
VM1:Windows Server 2022 ADC01(adc01.local)
VM2:Windows Server 2022 MEMBER01(ファイルサーバー)
VM3:Windows11 WORKSTATION01(社員ＰＣ)
    各VMのネットワークはホストオンリーとする


Splunk:Enterprise Free(ログ解析)
ADC01(192.168.2.139)
    Splunk Enterprise(Indexer+Search Header)
    Universal Forwader → MEMBER01/WORKSTATION01
    Windows Event Log(Security/System)収集
MEMBER01(192.168.229.137)
    Universal Forwader → ADC01送信
    Sysmonログ
    Defenderログ
WORKSTATION01(192.168.229.138)
    Universal Forwader → ADC01送信
    Sysmon ProcessCreate
    Powershell BlockRule

## Splunkの導入
1. splunk.comからsplunkenterpriseをダウンロードしインストール
2. splunk.comからsplunkforwaderをダウンロードしインストール
3. firewall設定でoutboundとinboundの該当ポートを許可
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
[WindEventLog://Security]
disabled = 0
start_from = oldest
index = windows
sourcetype = WinEventLog:Security

[WindEventLog://System]
disabled = 0
start_from = oldest
index = windows
sourcetype = WinEventLog:System

[WindEventLog://Application]
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