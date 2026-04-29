# AWS DOP-C02 学習リポジトリ - Claude Code 設定

## プロジェクト概要
AWS Certified DevOps Engineer - Professional（DOP-C02）の合格を目指す学習管理リポジトリ。

## 受験者プロフィール（個人情報は記載しない）
- 開発エンジニア 2年
- クラウドエンジニア 1年
- 保有資格：AWS CLF / AIF / SAA / SOA / DVA / 基本情報技術者
- 教材：Udemy「AWS DOP-C02実践問題集」（問題集3つ × 3周 完了済み）
- 学習済み時間：約30時間
- 受験日：2025年5月10日
- 週学習時間：平日1h × 5日 + 土日5h = 週10時間

## ⚠️ 個人情報ポリシー（必ず遵守）
- 氏名・企業名・所属・連絡先などの個人を特定できる情報は一切記載しない
- パスワード・APIキー・トークン等の認証情報は絶対に記載しない
- 固有名詞（職場名・学校名・プロジェクト名等）は記載しない
- スコア・日付・学習内容のみ記録する

## サブエージェント一覧
| エージェント | ファイル | 役割 |
|------------|---------|------|
| PM（進捗管理） | `.claude/agents/pm.md` | 進捗記録・カウントダウン管理・残日数での優先度調整 |
| チューター | `.claude/agents/tutor.md` | サービス解説・問題解説・混同しやすい選択肢の整理 |
| 試験戦略アドバイザー | `.claude/agents/strategist.md` | ドメイン別得点戦略・時間配分・残り期間の最適化 |
| 弱点分析 | `.claude/agents/weak_analyst.md` | 間違いパターン分析・根本原因特定・優先復習リスト生成 |

## スキル一覧
| スキル | ファイル | 用途 |
|--------|---------|------|
| 進捗記録 | `.claude/skills/log_progress.md` | 日次学習ログの記録フォーマット |
| 弱点登録 | `.claude/skills/add_weak_point.md` | 間違えた問題の記録フォーマット |
| サービスまとめ | `.claude/skills/generate_service_summary.md` | AWSサービス別まとめの生成ルール |

## フォルダ構成
```
dop-study/
├── CLAUDE.md                        ← このファイル
├── README.md                        ← リポジトリ概要
├── .claude/
│   ├── agents/
│   │   ├── pm.md
│   │   ├── tutor.md
│   │   ├── strategist.md
│   │   └── weak_analyst.md
│   └── skills/
│       ├── log_progress.md
│       ├── add_weak_point.md
│       └── generate_service_summary.md
├── 00_overview/
│   ├── exam_overview.md             ← 試験概要・スペック
│   ├── study_plan.md                ← 残り期間の学習計画
│   └── strategy.md                  ← 合格戦略
├── 01_domains/                      ← ドメイン別まとめ
│   ├── 01_sdlc_automation/          ← ★最重点（22%）CI/CD
│   ├── 02_config_management_iac/    ← ★重点（17%）CloudFormation/CDK
│   ├── 03_resilient_cloud/          ← ★重点（15%）HA/DR
│   ├── 04_monitoring_logging/       ← 重点（15%）CloudWatch/X-Ray
│   ├── 05_incident_event/           ← 重点（14%）EventBridge/Lambda
│   └── 06_security_compliance/      ← 重点（17%）IAM/SCP
├── 02_weak_points/
│   └── weak_points.md               ← 間違えた問題まとめ
├── 03_tips_and_knowhow/
│   └── exam_tips.md                 ← 試験テクニック・ノウハウ
├── 04_progress/
│   ├── daily_log.md                 ← 日次ログ
│   └── udemy_scores.md              ← Udemyスコア記録
└── 05_service_reference/            ← サービス別クイックリファレンス
```

## 問題解説ワークフロー（メインの使い方）

問題文・選択肢・解説をそのまま貼り付けて「解説して」と言うだけでOK。

Claudeが自動で：
1. 指定スタイルで解説（全体像 → 正解分解 → 罠 → 覚え方）
2. 「もっと教えて」「[サービス名]を深掘りして」で深掘りモードへ
3. 解説後に `01_domains/[ドメイン]/summary.md` を自動更新
4. 不正解の場合は `02_weak_points/weak_points.md` にも追記

## よく使うコマンド
```
# 問題解説（メイン）
問題文・解説をそのまま貼り付けて「解説して」

# 深掘り
「IAM Identity Centerをもっと詳しく教えて」
「SCPとIAMポリシーの違いを図で説明して」

# 学習記録
「PMエージェントとして今日の学習を記録して。問題集1を75問中58問正解」

# 弱点登録
「弱点を追加して。ドメイン：SDLC、サービス：CodeDeploy、内容：デプロイ戦略の使い分けが曖昧」

# 戦略確認
「戦略アドバイザーとして、試験まで残りX日の時点での優先学習内容を教えて」

# 弱点分析
「弱点分析エージェントとして、02_weak_points/weak_points.md を分析して今週の優先復習リストを作って」
```
