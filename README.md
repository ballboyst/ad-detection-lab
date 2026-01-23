# 公務員CSIRTのAD攻撃検知ラボ

## 目的
公務員CSIRT経験者として、民間企業CSIRTリーダー転職を見据えた**AD攻撃→Splunk検知の実務代替ラボ**を構築

## 環境構成
ホスト:AMD RYZEN7 + RAM 32GB + VMware
VM1:Windows Server 2022 DC(adc01.local)
VM2:Windows Server 2022 Member(ファイルサーバー)
VM3:Windows11(社員ＰＣ)

Splunk:Enterprise Free(ログ解析)
ADC01(192.168.229.136)
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
