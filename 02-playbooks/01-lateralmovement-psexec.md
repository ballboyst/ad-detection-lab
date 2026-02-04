# 01-AD横移動検知・封じ込めプレイブック
- MITRE ATT&CK: TA0008 Lateral Movement（PsExec / RDP）
- 対象イベント: 5145, 7045
- 対象ホスト: MEMBER01 → DC1 間の横移動


## 検知シナリオ

DOMAIN\Administrator が WORKSTATION01 から MEMBER01 へ PsExec を用いて横移動し、  
MEMBER01 上の ADMIN$ 共有へのアクセス（Event ID 5145）から PSEXESVC サービス作成（Event ID 7045）に至るチェーンを検知する。  
検知から 5 分以内に、PsExec を実行した攻撃元ホスト（WORKSTATION01）をネットワーク隔離し、  
MEMBER01 上に作成された PSEXESVC サービスおよび `C:\Windows\PSEXESVC.exe` を削除する。



## Splunk検知クエリ

### 1. 5145 ADMIN$ => 7045 PsExec 攻撃チェーン検知
```spl
index=windows (EventCode=5145 OR EventCode=7045)
  ("ADMIN$" OR "管理共有" OR "PSEXESVC" OR "PsExec")
| eval ShareName   = coalesce('共有名',"Share Name",'ShareName')
| eval IpAddress   = coalesce('IP アドレス','IpAddress','dest_ip','送信元アドレス')
| eval AccountName = coalesce('アカウント名',AccountName,user)
| eval ServiceRaw  = coalesce('サービス名',ServiceName,param1,Name)
| eval ImageRaw    = coalesce('サービス ファイル名',ImagePath,param2,ImagePathActual)
| eval ServiceName = case(
    EventCode=7045 AND isnotnull(ServiceRaw), ServiceRaw,
    EventCode=5145, "N/A (share access)",
    1=1, "N/A"
  )
| eval ImagePath = case(
    EventCode=7045 AND isnotnull(ImageRaw), ImageRaw,
    EventCode=5145, "N/A (share access)",
    1=1, "N/A"
  )
| eval alert = case(
    EventCode=5145 AND like(ShareName,"%ADMIN$%"),          "INITIAL ACCESS",
    EventCode=7045 AND like(ServiceName,"%PSEXESVC%"),      "LATERAL MOVEMENT",
    1=1,                                                    "INFO"
  )
| where alert!="INFO"
| table _time, AccountName, IpAddress, host, alert, EventCode, ShareName, ServiceName, ImagePath
| sort - _time
```


## 対応手順（目標: 検知から5分以内）

- T+0〜T+1分: ダッシュボードでアラート確認
  - PsExec LATERAL MOVEMENT DETECTION ダッシュボードで検知件数「1以上」を確認
  - テーブルから AccountName / IpAddress / host（攻撃元）を特定

- T+1〜T+3分: 攻撃元ホストのネットワーク隔離
  - 管理端末から以下を実行（例: DC1 から）
  - `netsh advfirewall firewall add rule name="Isolate_PsExec" dir=in action=block remoteip=<攻撃元IP>`

- T+3〜T+5分: 悪性サービス削除と痕跡確認
  - 対象ホストで PSEXESVC サービス削除: `sc delete PSEXESVC`
  - `C:\Windows\` 直下の `PSEXESVC*.exe` を削除
  - Splunk で 7045 / 4688 が収束していることを確認


## 検証結果（ラボ）

- テスト回数: 3回
- 平均検知時間（MTTD）: XX 秒（5145 → 7045 がダッシュボードに反映されるまで）
- 平均封じ込め時間（MTTC）: X 分 X 秒（検知〜netsh隔離ルール適用完了まで）
- 想定誤検知:
  - 管理用途での正規 PsExec 利用時にも検知されるため、運用時は管理用アカウント / 管理端末をホワイトリスト化予定
