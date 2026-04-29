# AWS DOP-C02 学習リポジトリ

## 🎯 目標
**2025年5月10日 AWS Certified DevOps Engineer - Professional（DOP-C02）合格**

---

## 📋 試験スペック

| 項目 | 内容 |
|------|------|
| 正式名称 | AWS Certified DevOps Engineer - Professional |
| バージョン | DOP-C02 |
| 問題数 | 75問（うち採点対象外10問） |
| 試験時間 | 180分（+入退場等10分 = 計190分） |
| 形式 | 単一選択・複数選択（択一・複択混在） |
| 合格スコア | 750点 / 1000点（スケールスコア） |
| 受験料 | $300 USD（他AWS資格保有で50%割引クーポンあり） |
| 言語 | 日本語・英語（試験中に切替可能） |
| 配信 | Pearson VUE（テストセンター or オンライン） |

---

## 📊 ドメイン別出題割合（DOP-C02）

| # | ドメイン | 割合 | 優先度 |
|---|---------|------|--------|
| 1 | SDLC のオートメーション | **22%** | ★★★★★ |
| 2 | 構成管理と IaC | **17%** | ★★★★☆ |
| 3 | 耐障害性のあるクラウドソリューション | **15%** | ★★★★☆ |
| 4 | モニタリングとロギング | **15%** | ★★★☆☆ |
| 5 | インシデントとイベントへの対応 | **14%** | ★★★☆☆ |
| 6 | セキュリティとコンプライアンス | **17%** | ★★★★☆ |

---

## 🔑 頻出サービス一覧

### ドメイン1：SDLC オートメーション（22%）
- CodePipeline / CodeBuild / CodeDeploy / CodeCommit
- デプロイ戦略：Blue/Green・Canary・Rolling・In-Place
- Elastic Beanstalk / ECS / EKS のデプロイ
- Artifact管理：ECR・S3・CodeArtifact

### ドメイン2：構成管理・IaC（17%）
- CloudFormation（スタック・ネストスタック・ドリフト検出・変更セット）
- AWS CDK / AWS SAM
- Systems Manager（SSM Documents・Parameter Store・Patch Manager）
- AWS Config / Service Catalog

### ドメイン3：耐障害性（15%）
- Auto Scaling / ALB / Route 53 フェイルオーバー
- DRパターン：Backup/Restore・Pilot Light・Warm Standby・Multi-Site
- RDS Multi-AZ / Aurora Global / DynamoDB Global Tables
- マルチリージョン構成

### ドメイン4：モニタリング・ロギング（15%）
- CloudWatch（メトリクス・ログ・アラーム・ダッシュボード・Logs Insights）
- X-Ray / CloudTrail
- AWS Config Rules / Security Hub
- 集中ログ管理（Kinesis Firehose → S3）

### ドメイン5：インシデント・イベント対応（14%）
- EventBridge（ルール・パターン・スケジュール）
- Lambda（自動修復トリガー）
- Systems Manager OpsCenter / Incident Manager
- SNS / SQS による通知・キューイング

### ドメイン6：セキュリティ・コンプライアンス（17%）
- IAM（ポリシー・ロール・Permission Boundary）
- Organizations / SCP（サービスコントロールポリシー）
- Secrets Manager / KMS
- GuardDuty / Security Hub / Macie

---

## 📅 残り学習スケジュール

```
受験日：2025年5月10日
現在：2025年4月29日
残り：約11日

週10時間（平日1h × 5 + 土日5h）
残り学習時間見込み：約15〜17時間
```

> ⚡ **すでに30時間・問題集3周完了 → 仕上げフェーズ**
> 新しいことを学ぶより「弱点の徹底補強」に集中する

---

## ⚠️ 個人情報ポリシー
- 氏名・企業名・所属・連絡先等は**一切記載しない**
- パスワード・APIキー・トークン等は**絶対に記載しない**
- スコア・日付・学習内容のみ記録する

---

## 🔗 公式リソース
- [試験ガイド（DOP-C02）](https://d1.awsstatic.com/ja_JP/training-and-certification/docs-devops-pro/AWS-Certified-DevOps-Engineer-Professional_Exam-Guide.pdf)
- [AWS公式サンプル問題](https://d1.awsstatic.com/ja_JP/training-and-certification/docs-devops-pro/AWS-Certified-DevOps-Engineer-Professional_Sample-Questions.pdf)
- [AWS Skill Builder（公式模擬試験）](https://skillbuilder.aws/)
- [AWS Black Belt Online Seminar](https://aws.amazon.com/jp/events/aws-event-resource/archive/)
