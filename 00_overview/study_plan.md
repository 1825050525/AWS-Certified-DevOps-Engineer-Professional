# 学習計画（2026年5月〜8月）

## 全体スケジュール
- 学習期間：約4ヶ月（17週）
- 週学習時間：9時間（平日1h×5 + 休日4h）
- 総学習時間見込み：約153時間

## ドメイン別出題比率と優先度
| ドメイン | 出題比率 | 優先度 | 備考 |
|---------|---------|--------|------|
| Domain 1: SDLC Automation | 22% | 最高 | CodePipeline/CodeDeploy中心 |
| Domain 6: Security and Compliance | 17% | 高 | IAM/SSM/Secrets Manager |
| Domain 2: Configuration Management and IaC | 17% | 高 | CloudFormation/CDK |
| Domain 3: Resilient Cloud Solutions | 15% | 中 | ECS/EKS/Auto Scaling |
| Domain 4: Monitoring and Logging | 15% | 中 | CloudWatch/Config/X-Ray |
| Domain 5: Incident and Event Response | 14% | 中 | EventBridge/OpsWorks |

## 月別ロードマップ

### 5月：インプット（基盤固め）
- [ ] AWS公式試験ガイドを精読してドメイン全体像を把握
- [ ] Udemy DOP講座（Stephane Maarek）第1〜2週分を視聴
- [ ] Tutorial Dojo 初回模試（実力チェック・弱点ドメイン特定）
- [ ] Domain 1（SDLC Automation）の集中インプット
  - CodePipeline, CodeBuild, CodeDeploy, CodeCommit
  - Blue/Green・Canary・Rolling デプロイ戦略
- [ ] Domain 6（Security and Compliance）の集中インプット
  - IAM Policies, SSM Parameter Store, Secrets Manager
  - Config Rules, Security Hub, GuardDuty
- **目標スコア**：Tutorial Dojo 模試 60%

### 6月：インプット完了＋アウトプット開始
- [ ] Udemy DOP講座 残り部分を視聴完了
- [ ] Domain 2（CloudFormation/CDK/SAM）の集中インプット
  - StackSets, Nested Stacks, Custom Resources
  - CDK Constructs, Elastic Beanstalk
- [ ] Domain 3（Resilient Cloud Solutions）の集中インプット
  - ECS/EKS デプロイ戦略
  - Auto Scaling Policy・ALB/NLB
  - Multi-Region / Disaster Recovery パターン
- [ ] Tutorial Dojo 模試2回実施・ドメイン別分析
- **目標スコア**：Tutorial Dojo 模試 70%

### 7月：アウトプット強化＋弱点克服
- [ ] Domain 4（CloudWatch/Config/X-Ray）の集中インプット
  - CloudWatch Metrics・Alarms・Logs Insights
  - AWS Config Rules・Conformance Packs
- [ ] Domain 5（Incident and Event Response）の集中インプット
  - EventBridge Rules, SNS, SQS
  - OpsWorks, Systems Manager OpsCenter
- [ ] Tutorial Dojo 模試を週2回ペース
- [ ] 03_weak_points を集中復習
- **目標スコア**：Tutorial Dojo 模試 78%

### 8月：最終仕上げ・本番
- [ ] 全ドメインの苦手問題総復習
- [ ] Tutorial Dojo・Whizlabs で毎日模試1セット
- [ ] AWS公式サンプル問題を繰り返し
- [ ] 試験申込・日程確定（月初）
- [ ] 試験当日のシミュレーション（180分・時間配分）
- **目標スコア**：Tutorial Dojo 模試 85% → 本番 800+

## 週次ルーティン
```
月：Udemy視聴 or ドメインまとめ読み（1h）
火：Tutorial Dojo 模試 30問 + 解説復習（1h）
水：苦手問題・weak_points 見直し（1h）
木：Tutorial Dojo 模試 30問 + 解説復習（1h）
金：サービス別まとめをClaudeCodeで生成・整理（1h）
土：Tutorial Dojo フル模試（65問・180分）（2h）
日：週次進捗まとめ → PMエージェントで記録 + 戦略調整（2h）
```

## 重点学習サービス（DOP頻出）
### Domain 1: SDLC Automation（22%）
- AWS CodePipeline（パイプライン設計・承認アクション）
- AWS CodeBuild（buildspec.yml・環境変数・アーティファクト）
- AWS CodeDeploy（AppSpec・デプロイ設定・ロールバック）
- AWS CodeCommit（ブランチ戦略・プルリクエスト）
- Amazon ECR（イメージスキャン・ライフサイクルポリシー）

### Domain 2: Configuration Management and IaC（17%）
- AWS CloudFormation（ドリフト検出・StackSets・カスタムリソース）
- AWS CDK（Constructs・cdk synth/deploy）
- AWS Elastic Beanstalk（デプロイポリシー・.ebextensions）
- AWS OpsWorks（Chef/Puppet・レイヤー）

### Domain 3: Resilient Cloud Solutions（15%）
- Amazon ECS / EKS（Fargate・タスク定義・サービス更新）
- AWS Auto Scaling（スケーリングポリシー・ウォームアップ）
- Elastic Load Balancing（ALB/NLB・ターゲットグループ）
- Amazon Route 53（フェイルオーバー・ヘルスチェック）

### Domain 4: Monitoring and Logging（15%）
- Amazon CloudWatch（カスタムメトリクス・ログ・ダッシュボード）
- AWS X-Ray（トレース・サンプリングルール）
- AWS Config（Rules・Remediation・Conformance Packs）
- AWS CloudTrail（マルチリージョン・ログファイル検証）

### Domain 5: Incident and Event Response（14%）
- Amazon EventBridge（Rules・Patterns・Targets）
- AWS Systems Manager（OpsCenter・Automation・Run Command）
- Amazon SNS / SQS（通知・キュー・DLQ）
- AWS Lambda（イベントドリブン・エラーハンドリング）

### Domain 6: Security and Compliance（17%）
- AWS IAM（Policy評価・SCPs・Permission Boundaries）
- AWS Systems Manager Parameter Store（暗号化・バージョニング）
- AWS Secrets Manager（自動ローテーション）
- Amazon Inspector / Security Hub / GuardDuty（脅威検知）
- AWS KMS（CMK・キーポリシー）
