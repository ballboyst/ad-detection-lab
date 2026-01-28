# 01-AD横移動検知・封じ込めプレイブック
**MITRE ATT&CK: TA0008 Lateral Movement** | **EventCode=5145/4776/4624**

## 検知シナリオ
MEMBER01 → DC1への**PsExec/RDP**横移動攻撃検知→**5分封じ込め**


## Splunk検知クエリ

### 1. PsExecサービス作成検知（最速）
```spl
index=wineventlog EventCode=7045 ServiceName!=*rpc* ServiceFileName!=*svchost*
| eval detection=if(match(ServiceFileName,"(?i)(psex|wrmy|random_svc)"),1,0)
| where detection=1
| table _time, host, ServiceName, ServiceFileName, ImagePath

index=wineventlog EventCode=5145 ShareName="\\*\ADMIN$" 
| stats count by host, src_ip, dest_ip values(ShareName) as shares 
| where count>5
| table _time, host, src_ip, dest_ip, shares

index=wineventlog EventCode=4624 LogonType=10
| stats count by src_ip, dest_ip values(host) as targets 
| where count>3
| table _time, src_ip, dest_ip, targets, count
```

## 2. 対応手順（5分）
T+0:攻撃元IP特定
T+2:netsh advfirewallで隔離
T+4:サービス停止(sc delete)

## 3. 検証結果
ラボMTTD：X秒、封じ込め:X分X秒