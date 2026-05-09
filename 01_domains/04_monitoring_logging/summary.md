# ドメイン4：モニタリングとロギング

---

## 問題47：CloudWatch ログ監視 → メトリクスフィルター ＋ アラーム ＋ SNS ⭐ 難易度：低
**結果：正解**

### 一言でいうと
「ログから特定文字列を検知して通知 → CloudWatch メトリクスフィルター → アラーム → SNS」

### カテゴリ別ポイント

#### ログから通知までの正しいフロー
```
ファイアウォールログ → CloudWatch Logs
  → メトリクスフィルター（"CRITICAL"パターンを検知）
  → カスタムメトリクス（発生回数をカウント）
  → CloudWatch アラーム（閾値超えで発火）
  → SNS → セキュリティチームへ通知
```

#### サービスの役割
| サービス | できること | 今回使えるか |
|---|---|---|
| **CloudWatch メトリクスフィルター** ✅ | ログ内の文字列パターンを検出 → メトリクス化 | 使える |
| CloudWatch Synthetics | WebサイトやAPIの外形監視 | ❌ ログ解析機能なし |
| Firewall Manager | WAF/SGポリシーのガバナンス | ❌ CloudWatch Logsのログ監視機能なし |
| GuardDuty | VPCフローログ・CloudTrail等から脅威検出 | ❌ カスタムログフォーマットは対象外 |

### 罠
| 選択肢 | なぜ間違い |
|---|---|
| CloudWatch Synthetics | 外形監視専用。ログ内の文字列は検出できない |
| Firewall Manager + EventBridge | Firewall Managerはガバナンス用。ログ監視機能なし |
| GuardDuty | カスタムログフォーマットのファイアウォールログは解析対象外 |

### 覚え方
> **「ログの特定パターンを検知 → メトリクスフィルター」**  
> GuardDutyは自分が知っているデータソース（VPCフローログ等）しか見ない

---

## 問題4（P2）：ログのリアルタイム加工（Kinesis Data Firehose ＋ Lambda） ⭐⭐ 難易度：高
**結果：不正解**

### 一言でいうと
「ログをリアルタイムで中身ごと加工 → Network Firewall 送信先を Firehose に変更 → Lambda で変換 → S3」

### カテゴリ別ポイント

#### EventBridge 入力変換の限界（ここで選択ミス）
```
EventBridge 入力変換でできること：
  → イベントのメタデータ変換（S3パス・サイズ等）
EventBridge 入力変換でできないこと：
  → S3オブジェクトの中身（ログの内容）を読んで変換
```

#### 正しいアーキテクチャ
```
Network Firewall → Kinesis Data Firehose
                       ↓ Lambda でリアルタイム変換
                       ↓
                      S3（加工済みログ）
```

#### S3 トリガー + Lambda が不適切な理由
- 同じバケットに保存 → 再帰呼び出しループのリスク
- S3書き込み後のトリガー → 真のリアルタイム処理ではない

### 覚え方
> **「ログの中身を加工したい → Firehose ＋ Lambda」**  
> EventBridge 入力変換はメタデータのみ。ファイルの中身は読めない

---

## 問題6（P2）：EKS メモリメトリクス収集（Container Insights） ⭐⭐ 難易度：高
**結果：不正解**

### 一言でいうと
「EKSメモリ = CloudWatchエージェント（AMI込み）＋ インスタンスプロファイルに権限 ＋ pod_memory_utilization で分析」

### カテゴリ別ポイント

#### メトリクスの粒度
| メトリクス | 粒度 | 今回の要件 |
|---|---|---|
| **pod_memory_utilization** ✅ | ポッド（アプリ）単位 | アプリのメモリ問題特定に最適 |
| node_memory_utilization ❌ | EC2インスタンス全体 | 粒度が粗くアプリ特定不可 |

#### ディメンション（絞り込みの軸）
| ディメンション | 範囲 | 今回の要件 |
|---|---|---|
| **Service** ✅ | 特定サービスの全ポッド集計 | アプリ単位の分析に最適 |
| ClusterName ❌ | クラスタ全体 | 広すぎる |

#### 権限をどこに付けるか
```
✅ インスタンスプロファイルの IAM ロール ＋ CloudWatchAgentServerPolicy
   → EC2インスタンス自体が CloudWatch に送信する権限

❌ サービスアカウントロール
   → ポッド内アプリが AWS API を叩く権限（エージェントには無関係）
```

### 覚え方
> **「CloudWatchエージェントの権限 = インスタンスプロファイル」（サービスアカウントではない）**  
> **「EKSのアプリメモリ = pod_memory_utilization ＋ Service ディメンション」**

---

## 問題29（P2）：Kinesis ＋ Lambda のログ処理遅延改善 ⭐ 難易度：中
**結果：不正解**

### 一言でいうと
「Kinesis 遅延改善 = シャード数増加 ＋ ParallelizationFactor ＋ Enhanced Fan-out」

### カテゴリ別ポイント

#### 3つの改善策
| 方法 | 効果 |
|---|---|
| シャード数を増加 | 並列処理容量を増加（1シャード = 1MB/秒） |
| **ParallelizationFactor を増加（1-10）** | 1シャードを複数 Lambda が並列処理 |
| **Enhanced Fan-out** | 各コンシューマに専用 2MB/秒。プッシュ型で平均200ms以下 |

#### ReportBatchItemFailures がダメな理由（選んだ答え）
```
この機能 = 「どのレコードが失敗したか」を報告する信頼性機能
→ 遅延とは無関係
→ 無効化するとデータがサイレントにドロップされる危険
```

### 覚え方
> **「Kinesis 遅延 → シャード数 ＋ ParallelizationFactor ＋ Enhanced Fan-out」**  
> バッチサイズ増加は遅延悪化（コスト効率向上目的）
