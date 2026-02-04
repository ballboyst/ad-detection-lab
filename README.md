# 公務員CSIRTのAD攻撃検知ラボ

## 目的
CSIRT で求められる AD 攻撃検知とインシデント対応スキルを身につけるための **AD攻撃→Splunk検知の実践ラボ**です。

このリポジトリは、3台構成の AD ラボ上で PsExec を用いた横移動（TA0008）を検知し、
5分以内に封じ込める一連の流れを再現することを目的としています。


## このリポジトリの読み方

時間がない方向けのおすすめ閲覧順は、以下の通りです。

1. 本 README（目的とラボ構成の概要）
2. `03-dashboards/` のスクリーンショット
3. `02-playbooks/01-AD-lateral-movement.md` の検知シナリオと対応手順
4. 必要に応じて `01-lab/` の構築メモ（再現したい場合）


## このラボで検証している検知

- 対応タクティクス：TA0008 Lateral Movement
- 対応テクニック例：T1570,T1021.002
- PsExec による ADMIN$ 共有アクセス（Event ID 5145）
- PSEXESVC サービス作成による横移動実行（Event ID 7045）
- 日本語 / 英語 Windows イベントログ両対応のフィールド正規化（ShareName, ServiceName など）
- Splunk ダッシュボードによる 5145 → 7045 攻撃チェーンの可視化


## 構成

```mermaid
graph TD;
    DC1[ADC01<br/>AD+DNS<br/>Splunk Free]
    C1[MEMBER01<br/>Splunk UF]
    C2[WORKSTATION01<br/>Splunk UF]
    
    DC1 --- |WinEventLog|C1
    DC1 --- |WinEventLog|C2
    
    style DC1 fill:#e1f5fe
    style C1 fill:#f3e5f5
    style C2 fill:#f3e5f5

```
- ADC1: AD DS / DNS / Splunk (Windowsログ集約・検知)

- MEMBER01: ドメイン参加サーバ（横移動の標的）

- WORKSTATION01: ドメイン参加クライアント（攻撃元想定）


## ダッシュボード

- `PsExec LATERAL MOVEMENT DETECTION`（03-dashboards/）
  - 過去24時間の PsExec 横移動検知数を Single Value で表示（1件以上で赤表示）
  - 下部テーブルで 5145（ADMIN$ アクセス）→7045（PSEXESVC サービス作成）の攻撃チェーンを色分け表示
  - 日本語 / 英語 Windows イベントログ両対応のフィールド正規化を実装

## プレイブック

- [01-AD横移動検知・封じ込めプレイブック](./02-playbooks/01-AD-lateral-movement.md)
  - 対象: WORKSTATION01 → MEMBER01 間の PsExec 横移動
  - 使用クエリ: `PsExec LATERAL MOVEMENT DETECTION` ダッシュボード内の検索と同一
  - 成功条件: PsExec 実行から 5分以内に、ダッシュボードで検知 → 攻撃元ホスト（WORKSTATION01）隔離 → MEMBER01 上の PSEXESVC サービス削除まで完了


## ステータス

- ✅ PsExec 横移動（5145/7045）の検知とダッシュボード化
- ✅ 5分以内封じ込めプレイブック（netsh隔離 + sc delete）
- ⏳ 4624/4776 を使ったログオン経路トレース（別プレイブックとして追加予定）

## ラボ環境（概要）

- ホスト: AMD Ryzen 7 / 32GB RAM / VMware
- ゲスト:
  - VM1: Windows Server 2022 ADC01（ドメインコントローラ / Splunk Indexer）
  - VM2: Windows Server 2022 MEMBER01（ファイルサーバ / UF）
  - VM3: Windows 11 WORKSTATION01（クライアント / UF）
- ログ収集:
  - 各ホストから ADC01 上の Splunk Enterprise へ WinEventLog（Security / System / Application）を転送

詳細なセットアップ手順は [01-lab/SPLUNK_SETUP.md](./01-lab/SPLUNK_SETUP.md) を参照してください。
