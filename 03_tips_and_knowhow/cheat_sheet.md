# DOP-C02 直前暗記シート

> 試験前にここだけ見る。「なぜ正解か」を思い出すための一枚。

---

## 鉄則3か条

1. **管理アカウントは触らない** → 委任管理者アカウントに任せる
2. **AWSネイティブ機能優先** → EventBridge+Lambda の自前実装より既存機能を使う
3. **予防 vs 事後** → セキュリティサービスの役割を区別する

---

## ドメイン1：SDLC・デプロイ戦略（22%）

### デプロイ戦略の比較

| 戦略 | 特徴 | ロールバック | キーワード |
|---|---|---|---|
| **Blue/Green** | 新環境を丸ごと用意→瞬時切り替え | ALBをBlueに戻すだけ | 「瞬時に切り替え」 |
| **Canary** | 新バージョンに10%→異常なし→全量 | 自動（CloudWatchアラーム） | 「段階的」「10%」 |
| **Rolling** | 少しずつ順番に更新 | 複雑（途中状態が混在） | 「順次更新」 |
| **AllAtOnce** | 全台同時更新 | ダウンタイムあり | 「一括」 |

### Blue/Green の順序（順番を間違えると不正解）
```
❌ 切り替え → デプロイ（未完成環境にトラフィックが流れる）
✅ デプロイ（Green完成）→ 確認 → ALBターゲットグループ切り替え
```

### ALB切り替え vs DNS切り替え
- **ALBターゲットグループ切り替え** = 瞬時・確実（ALBがすでにあるなら必ずこっち）
- **Route 53 DNS変更** = TTL分の遅延・キャッシュ問題あり（ALBがない時だけ）

---

### SAM（Serverless Application Model）

```yaml
# これだけでカナリアデプロイ完成
DeploymentPreference:
  Type: LambdaCanary10Percent10Minutes
  Alarms:
    - !Ref MyCloudWatchAlarm
```

- SAMが内部でCodeDeploy・Lambda Alias・appspec.ymlを**自動管理**
- → CodeDeployを手動で作るのはNG（二重管理になる）
- → appspec.ymlを手動で書くのもNG（SAMが自動生成する）
- CI/CDパイプライン = `CodeCommit → CodePipeline → CodeBuild（sam deploy）`

---

### CodeDeploy ライフサイクルフックの順序

```
BeforeInstall → Install → AfterInstall → ApplicationStart → ValidateService
  ↓（Blue/Green・ELB統合時のみ）
BeforeAllowTraffic → AllowTraffic → AfterAllowTraffic
```

**AllowTrafficで失敗 ＋ ログに何もない** → **ALBヘルスチェック設定ミス**
- ログにエラーあり → appspec.ymlエラー or IAM権限不足
- AllowTraffic前で失敗 → SSM Agent未インストール

---

### buildspec.yml vs appspec.yml

| ファイル | 使うサービス | 役割 |
|---|---|---|
| `buildspec.yml` | CodeBuild | ビルド・テスト・パッケージング手順 |
| `appspec.yml` | CodeDeploy | デプロイ手順・ライフサイクルフック |

---

## ドメイン2：設定管理・IaC（17%）

### CloudFormation スタック状態

| 状態 | 意味 | 対処 |
|---|---|---|
| `UPDATE_ROLLBACK_FAILED` | 更新失敗 ＋ ロールバックも失敗 | `continue-update-rollback` を実行 |
| `ROLLBACK_COMPLETE` | 初回作成失敗後のロールバック完了 | スタックを削除して再作成 |
| `DELETE_FAILED` | 削除失敗 | 依存リソースを手動削除してから再試行 |

**`continue-update-rollback` の前にやること**
1. CloudTrailで拒否されたAPIを確認
2. IAMロールに不足権限を追加
3. コマンド実行（`ResourcesToSkip` で問題リソースをスキップも可）

---

### Systems Manager サービスの使い分け

| 機能 | 用途 | パッチ管理に使えるか |
|---|---|---|
| **Patch Manager** | パッチ管理専用。ベースライン・承認・スケジュール対応 | ✅ 最適 |
| **メンテナンスウィンドウ** | 業務時間外のスケジュール実行 | ✅ Patch Managerと組み合わせる |
| **Run Command** | 個別コマンドの即時実行 | ❌ 単体では不十分（自動化・管理機能なし） |
| **Session Manager** | インタラクティブなシェルアクセス | ❌ パッチ配布機能なし |
| **Parameter Store** | 設定値・シークレットの管理 | - |

### ハイブリッド環境のSSM登録

| リソース種別 | IAM設定 | SSM登録方法 |
|---|---|---|
| EC2インスタンス | インスタンスプロファイル | 起動時に自動（SSM Agent標準搭載） |
| オンプレミスサーバー | サービスロール | **マネージドインスタンスアクティベーション** ＋ SSM Agent手動インストール |
| IoT Greengrassデバイス | トークン交換ロールを更新 | Greengrass経由でSSM Agentをデプロイ |

---

## ドメイン3：耐障害性（15%）

### Route 53 ルーティングポリシーの使い分け

| ポリシー | 用途 | 間違えやすいポイント |
|---|---|---|
| **レイテンシーベース** | 実測遅延が最小のリージョンへ誘導 | 「遅延削減」「地理的分散」はこれ |
| **フェイルオーバー** | 障害時にプライマリ→セカンダリ切替 | 通常時は片方しか使わない。遅延は下がらない |
| **地理的（Geolocation）** | ユーザーの国・地域でルーティング | コンプライアンス・法規制対応 |
| **加重（Weighted）** | 割合でトラフィックを分散 | カナリアデプロイに使える |

### マルチリージョン構成の3点セット
```
新リージョンにALB + Auto Scalingグループ（個別作成）
  ＋ Route 53 レイテンシーベースルーティング
  ＋ DynamoDB グローバルテーブル
```
- **ALB / Auto Scaling はリージョナルサービス**（グローバルALBは存在しない）
- DynamoDB 複数リージョン = グローバルテーブル（手動レプリケーション設定は不要）

### Auto Scaling ウォームプール

| 状態 | 料金 | 起動速度 | 用途 |
|---|---|---|---|
| **Stopped** ✅ | EBSのみ | 速い（初期化済み） | コスト効率重視 |
| Running | フル課金 | 速い | 超厳格なレスポンス要件 |
| Standby | フル課金 | 変わらず | メンテナンス用 |

---

## ドメイン4：モニタリング・ロギング（15%）

### CloudWatch / CloudTrail / Config の役割分担

| サービス | 何を監視するか | ユースケース |
|---|---|---|
| **CloudWatch** | メトリクス・ログ・アラーム | アプリのパフォーマンス、スケーリングトリガー |
| **CloudTrail** | APIコール（誰が何をしたか） | 監査ログ、不正アクセス調査 |
| **AWS Config** | リソース設定の変更・コンプライアンス評価 | 「いつ設定が変わったか」「基準を満たしているか」 |

### CloudWatch アラームの設定パターン

```
個別アラーム（Lambda関数ごと） → カナリアデプロイの自動ロールバックに使う
複合アラーム（複数アラームの AND/OR） → 粒度が粗い。Lambda単体監視には不向き
```

---

## ドメイン5：インシデント・イベント（14%）

### EventBridge vs SNS vs SQS

| サービス | 特徴 | 主な用途 |
|---|---|---|
| **EventBridge** | ルールベースのイベントルーティング | AWSサービスイベントのトリガー |
| **SNS** | Pub/Sub通知（ファンアウト） | メール通知、複数サービスへの同時配信 |
| **SQS** | キュー（順序保証・再処理可能） | 非同期処理、リトライが必要な処理 |

### Lambda エラー処理
- **DLQ（デッドレターキュー）** = 失敗したイベントを別のSQS/SNSに送る
- **同期呼び出し** = エラーは呼び出し元に返る
- **非同期呼び出し** = 2回リトライ後にDLQへ

---

## ドメイン6：セキュリティ・コンプライアンス（17%）

### ガバナンス三種の神器

| サービス | 役割 | タイプ |
|---|---|---|
| **SCP** | IAM権限の上限設定（禁止専門） | 予防（許可はできない） |
| **Firewall Manager** | WAF/Shield/SGを自動適用 | 予防（リソース作成時に自動付与） |
| **AWS Config** | リソース設定のコンプライアンス評価 | 事後検出（＋修復） |
| **GuardDuty** | 脅威検出（異常なAPIコール・通信） | 事後検出（修復機能なし） |

```
新リソース作成の流れ
  SCP：そもそもこの操作を許可するか（権限チェック）
    ↓
  Firewall Manager：WAF/SGを自動でつける（予防的適用）
    ↓
  Config：設定が基準を満たしているか評価（事後チェック）
    ↓
  GuardDuty：怪しい動きを検知（継続的脅威監視）
```

### 「予防」か「事後」かで選ぶ

- 「設定漏れを**防ぐ**」「**自動適用**したい」→ **Firewall Manager**
- 「コンプライアンスを**確認**したい」「**検出して通知**したい」→ **Config**
- 「脅威を**検知**したい」→ **GuardDuty**

---

### Organizations・委任管理者の原則

```
管理アカウント（Organizations管理）
  ├── セキュリティアカウント（Security Hub / Firewall Manager / Config 委任管理者）
  ├── ログアーカイブアカウント（CloudTrail / S3ログ集約）
  └── 各メンバーアカウント
```

- 管理アカウントで**直接セキュリティ設定をするのはNG**
- 各サービスは専用アカウントに**委任管理者**として設定する

### Config の組織管理

| 方法 | 評価 |
|---|---|
| **コンフォーマンスパック** ＋ 委任管理者からConfig直接配布 ✅ | 正解。StackSets不要 |
| StackSets 経由でデプロイ ❌ | Config に配布機能があるので不要・複雑 |
| 管理アカウントから直接 ❌ | 管理アカウントは使わない |

### SCP vs IAM Identity Center

| 目的 | 使うもの |
|---|---|
| 組織レベルで「禁止」する | SCP |
| 人によって見える範囲を変える（許可制御） | **IAM Identity Center の許可セット** |
| SCPは「許可」ができない | → 細かいアクセス制御はIAM Identity Centerが担う |

### KMS キーローテーション監視

- KMS単体に通知機能はない
- Security Hub / Trusted Advisorにも対応機能なし
- → **Config カスタムルール（Lambda）** で実装 ＋ SNS通知

---

## 頻出の「引っかけパターン」

| 引っかけ | 正しい理解 |
|---|---|
| 「EventBridge + Lambda で自動化」 | AWSネイティブ機能があれば不要（例：Security Hub自動登録） |
| 「StackSetsでConfigをデプロイ」 | Configコンフォーマンスパックで直接配布できる |
| 「GuardDutyでWAFを適用」 | GuardDutyはWAF管理機能を持たない |
| 「フェイルオーバールーティングで遅延削減」 | フェイルオーバーは障害対応用。遅延削減はレイテンシーベース |
| 「Run CommandでパッチをDeployする」 | パッチ管理はPatch Manager（Run Commandは単発実行用） |
| 「グローバルALBを設定する」 | ALBはリージョナルサービス。グローバルALBは存在しない |
| 「SCPで特定ユーザーにのみ許可」 | SCPは禁止のみ。許可制御はIAM Identity Center |
| 「appspec.ymlをSAMで手動作成」 | SAMが自動生成する |
| 「管理アカウントで委任管理者と同じ設定」 | 管理アカウントに設定するのはベストプラクティス違反 |
| 「CodeDeployエージェント未インストール → AllowTrafficで失敗」 | エージェント未インストールはBeforeInstallで失敗する |

---

## 最後に確認する思考フレームワーク

**問題を読んだら最初に確認すること**

1. **何が課題か？**（遅延・コスト・セキュリティ・自動化・コンプライアンス）
2. **予防か事後か？**（防ぎたい → Firewall Manager / 検出したい → Config / GuardDuty）
3. **自動化のレベルは？**（ネイティブ機能 > カスタム実装）
4. **誰が管理するか？**（管理アカウントは使わない → 委任管理者）
5. **ログに何か残っているか？**（ない → 外部サービス側が原因）
