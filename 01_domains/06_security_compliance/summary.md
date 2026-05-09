# ドメイン6：セキュリティとコンプライアンス

---

## 問題24：Control Tower + Security Hub の組織管理 ⭐難易度：高
**結果：正解**

### 概要
AWS Organizations + Control Tower 環境で Security Hub による CIS ベンチマーク準拠の監視体制を構築する。

### カテゴリ別ポイント

#### 組織管理・委任管理者
- 管理アカウントは組織管理に専念させ、Security Hub は**専用セキュリティアカウントに委任**する
- 管理アカウントで直接 CIS ベンチマークを設定するのはベストプラクティス違反

#### アクセス制御
- 「人によって見える範囲を変える」細かい許可制御 → **IAM Identity Center のアクセス許可セット**
- SCP は「禁止」しかできないため、個別の許可制御には使えない
```
セキュリティ部門 → 許可セットA（全アカウントの統合結果）
一般ユーザー    → 許可セットB（自分のアカウント分のみ）
```

#### 新規アカウントの自動登録
- Security Hub の**自動登録機能**を有効化するだけでよい
- EventBridge + Lambda の自前実装は不要（AWSネイティブ機能を優先）

### 正解の組み合わせ
1. IAM Identity Center でアクセス許可セットを作成・割り当て
2. 委任管理者を専用セキュリティアカウントに設定 ＋ CISベンチマーク実施
3. Security Hub 自動登録を有効化

### 覚え方
> 「管理アカウントは触らない」「許可制御は IAM Identity Center」「自動化はネイティブ機能優先」

---

## 問題29：WAF Web ACL の組織全体への強制適用（Firewall Manager） ⭐ 難易度：中
**結果：不正解**

### 一言でいうと
「予防的にWAFを全リソースに強制適用 → Firewall Manager 一択」

### カテゴリ別ポイント

#### セキュリティサービスの役割分類（ここが核心）
| サービス | 種別 | WAF適用 | 今回の用途 |
|---|---|---|---|
| **Firewall Manager** ✅ | 予防・強制 | できる（自動適用） | 正解 |
| AWS Config | 事後監視 | できない（検出のみ） | 不正解 |
| GuardDuty | 事後検出 | できない（脅威検出のみ） | 不正解 |

#### Firewall Manager の2つの役割
```
① 委任管理者に設定
   管理アカウント → セキュリティアカウントに Firewall Manager を委任
   → 責任分離 + セキュリティ専門チームが一元管理

② Firewall Manager ポリシーを作成
   → 新規作成の ALB / API Gateway に WAF Web ACL を自動適用
   → 保護されない期間ゼロ・設定漏れゼロ
```

#### なぜ Config では不十分か
```
Config：リソース作成 → 検出（タイムラグ） → アラート → 修復
                              ↑ 一時的に保護なし期間が発生

Firewall Manager：リソース作成 → 即座に WAF 自動適用
```

### 罠
| 選択肢 | なぜ間違い |
|---|---|
| GuardDuty を委任 | WAF 管理機能なし。脅威検出専用サービス |
| Config 管理ルール | 事後検出のみ。予防的自動適用はできない |

### 覚え方
> **予防（防ぐ）→ Firewall Manager**  
> **事後監視（見つける）→ Config**  
> **脅威検出（怪しい動きを知る）→ GuardDuty**

---

## 問題30：KMS キーローテーション監視（Config カスタムルール） ⭐ 難易度：中
**結果：正解**

### 一言でいうと
「KMSの手動ローテーション監視 = AWS Config カスタムルール ＋ SNS通知」

### カテゴリ別ポイント

#### 各サービスの機能範囲（ここが勝負）
| サービス | KMSローテーション監視 | 理由 |
|---|---|---|
| **Config カスタムルール** ✅ | できる | Lambda で KMS API を叩いて独自評価ロジックを実装可能 |
| KMS 単体 | できない | ローテーション間隔監視・通知機能は持たない |
| Security Hub | できない | KMS 固有の詳細監視は対象外（他サービスの集約が主役割） |
| Trusted Advisor | できない | KMS 固有チェックは含まれていない |

#### Config カスタムルールの仕組み
```
Config カスタムルール（定期実行）
  → Lambda（KMS ListKeys / DescribeKey で日付取得）
  → 90日超 → NON_COMPLIANT
  → SNS → セキュリティチームへ通知
```

#### なぜカスタムルールか
- 手動ローテーション監視用のマネージドルールは AWS に存在しない
- → Lambda で独自ロジックを実装するカスタムルールが必要

### 罠
| 選択肢 | なぜ間違い |
|---|---|
| KMS → SNS 直接通知 | KMS にその機能はない |
| Trusted Advisor → SNS | KMS ローテーション監視機能なし |
| Security Hub → SNS | KMS 固有の詳細チェックは対象外 |

### 覚え方
> 「AWSネイティブに存在しない監視要件 → Config **カスタム**ルール（Lambda で実装）」  
> マネージドルールで足りない時はカスタムルールが答え

---

## 問題32：Config コンフォーマンスパック ＋ 委任管理者 ⭐⭐ 難易度：高
**結果：不正解**

### 一言でいうと
「組織全体への Config ルール統一管理 → コンフォーマンスパック ＋ 委任管理者アカウントから直接デプロイ」

### カテゴリ別ポイント

#### コンフォーマンスパックとは
- Config ルール ＋ 修復アクションを 1 つのパッケージにまとめたもの
- 組織全体への一括配布・新規アカウントへの自動適用・非管理者の変更防止が標準機能

#### デプロイ方法の比較（ここで間違えた）
| 方法 | 評価 | 理由 |
|---|---|---|
| 委任管理者アカウントから **Config ネイティブ**でデプロイ ✅ | 正解 | Config 自体が組織全体配布機能を持つ。StackSets 不要 |
| 委任管理者から **StackSets** でデプロイ ❌ | 不正解 | 不必要な複雑性。Config ネイティブ機能で足りる |
| 管理アカウントからデプロイ ❌ | 不正解 | 管理アカウントは組織管理に専念させる |

#### 委任管理者アカウントを使う理由
```
管理アカウント    → 組織構造の管理に専念
委任管理者アカウント → Config の運用・配布を担当
→ 責任分離 ＋ 管理アカウントへのアクセスを最小化
```

### 罠
| 誤解 | 正しい理解 |
|---|---|
| StackSets が必要 | Config がネイティブで組織全体配布機能を持つので不要 |
| 管理アカウントからデプロイ | 委任管理者アカウントからがベストプラクティス |

### 覚え方
> **Config ルールの組織管理 = コンフォーマンスパック ＋ 委任管理者の Config ネイティブ配布**  
> StackSets は「Config に配布機能がない時の代替手段」。Config には既にある

---

## 問題49：セキュリティグループ SSH 無制限許可の検知（Config restricted-ssh） ⭐ 難易度：中
**結果：正解**

### 一言でいうと
「SSH 0.0.0.0/0 を検知 → Config の `restricted-ssh` マネージドルール一択」

### カテゴリ別ポイント

#### なぜ EventBridge + CloudTrail だけでは不十分か
```
EventBridge（AuthorizeSecurityGroupIngress）
  → 「誰かがインバウンドルールを追加した」は検知できる
  → でも「ポート22 ＋ 0.0.0.0/0 かどうか」は判定できない
  → 偽陽性が大量発生

Config restricted-ssh ルール
  → セキュリティグループの設定内容を直接評価
  → ポート22が0.0.0.0/0から許可されているか自動判定
  → 違反時 → NON_COMPLIANT → SNS通知
```

#### 他サービスが不適切な理由
| サービス | なぜダメ |
|---|---|
| GuardDuty + Security Hub | 動的な脅威検出専用。設定の静的監視は対象外 |
| Amazon Inspector | EC2インスタンス内の脆弱性スキャン。セキュリティグループは対象外 |

### 覚え方
> 「SSH無制限許可を検知」→ Config `restricted-ssh`（マネージドルールで一発）  
> EventBridgeは「変更があった」を検知するだけ。「何の変更か」は Config が評価する

---

## 問題51：Config 違反通知（EventBridge 入力変換 ＋ SNS） ⭐⭐ 難易度：中
**結果：不正解**

### 一言でいうと
「Config 違反通知 = EventBridge で特定ルールに絞り ＋ 入力変換でメッセージ整形 → SNS」

### カテゴリ別ポイント

#### 正しいフロー
```
Config（restricted-ssh 違反）
  → EventBridge（restricted-ssh の NON_COMPLIANT のみに絞る）
  → 入力変換（SG名・IDを含むメッセージに整形）
  → SNS → 受信者へ通知
```

#### 間違えたポイント
```
❌ 全 NON_COMPLIANT を拾う
   → restricted-ssh 以外の無関係な違反も通知が飛ぶ

✅ restricted-ssh ルール専用でフィルタ
   → 必要な違反だけを確実にキャプチャ
```

#### 入力変換（Input Transformation）
- EventBridge の標準機能でイベントの JSON を整形できる
- SGの名前・IDなど必要な情報だけを抽出してメッセージに含める
- SNS フィルターポリシーは**選別はできるが整形はできない** → 要件を満たせない

### 罠
| 選択肢 | なぜ間違い |
|---|---|
| Config → SNS 直接通知 | Config はSNSへ直接通知する機能を持たない |
| 全 NON_COMPLIANT をキャプチャ | 無関係なルール違反も拾ってしまう |
| Run Command 経由で SNS 転送 | Run Command は EC2 コマンド実行用。通知パイプラインには不適切 |

### 覚え方
> **「Config 違反通知 = EventBridge（特定ルールに絞る）＋ 入力変換（整形）＋ SNS」**  
> 「全部拾う」より「必要なものだけ絞る」のが正解

---

## 問題52：ハードコーディングされた認証情報の検出と置き換え ⭐ 難易度：中
**結果：正解**

### 一言でいうと
「コードの秘密情報を自動検出 = CodeGuru Reviewer、保管 = Secrets Manager」

### カテゴリ別ポイント

#### CodeGuru の2種類（ここが罠）
| サービス | 役割 |
|---|---|
| **CodeGuru Reviewer** ✅ | 静的コード解析。ハードコーディングされた認証情報を検出 |
| CodeGuru Profiler ❌ | 実行時パフォーマンス分析（CPU・メモリ）。静的問題は検出不可 |

#### Secrets Manager vs Parameter Store
| | Secrets Manager ✅ | Parameter Store |
|---|---|---|
| 用途 | DB認証情報・APIキー専用 | 設定値・フラグ管理 |
| 自動ローテーション | あり（RDS統合） | なし |

### 覚え方
> **「Reviewer = コードを読む、Profiler = 動きを見る」**  
> **「DB認証情報 → Secrets Manager」「設定値 → Parameter Store」**

---

## 問題54：クロスアカウント IAM アクセス（STS AssumeRole） ⭐ 難易度：中
**結果：正解**

### 一言でいうと
「管理アカウントから呼ぶ → 管理側に AssumeRole 許可 ＋ メンバー側に信頼ポリシー ＋ EC2読み取り権限」

### カテゴリ別ポイント

#### 必須の3点セット
```
✅ 管理アカウント：IAMロールに sts:AssumeRole → メンバーのロールARN を許可
✅ メンバーアカウント：信頼ポリシーで管理アカウントがロールを引き受けてよいと設定
✅ メンバーアカウント：AmazonEC2ReadOnlyAccess を付与
```

#### よくある方向性ミス
```
❌ 「メンバーが管理アカウントのロールを引き受ける」→ 向きが逆
❌ 「管理アカウントに EC2ReadOnly だけ付与」→ アカウント境界を越えられない
```

### 覚え方
> **「行く側（管理）に AssumeRole 許可、来られる側（メンバー）に信頼ポリシー ＋ 権限」**  
> クロスアカウントは3点セットで設定する

---

## 問題7（P2）：CFN専用ロールの設定（PassRole vs AssumeRole） ⭐⭐ 難易度：高
**結果：不正解**

### 一言でいうと
「開発者はCFNにロールを"渡す"だけ（PassRole）。引き受けるのはCloudFormationサービス」

### カテゴリ別ポイント

#### 信頼ポリシーのPrincipalを誰にするか（ここで選択ミス）
```
❌ 開発者IAMロールが引き受けられる設定
  → 開発者が直接ロールを引き受け → CFN以外にも使える → 要件違反

✅ cloudformation.amazonaws.com が引き受けられる設定
  → CloudFormationサービスだけが引き受け可能
  → 開発者はCFN経由でのみ利用できる
```

#### 開発者に必要な権限
```
✅ ReadOnlyAccess（直接リソース変更を防止）
✅ cloudformation:CreateStack / UpdateStack 等
✅ iam:PassRole（passedToService = cloudformation.amazonaws.com のみ）
```

### 覚え方
> **「CFNDeploymentロールのPrincipal = cloudformation.amazonaws.com（開発者ではない）」**  
> 開発者は PassRole で CFN にロールを渡すだけ。自分で引き受けない

---

## 問題8（P2）：Organizations CreateAccount のクロスアカウントアクセス ⭐⭐ 難易度：高
**結果：不正解**

### 一言でいうと
「organizations:CreateAccount は委任管理不可 → 管理アカウントのIAMロールをクロスアカウントで引き受ける」

### カテゴリ別ポイント

#### Organizations 委任管理の制限（ここが落とし穴）
```
委任管理できる：Config / Security Hub / GuardDuty などの管理
委任管理できない：organizations:CreateAccount（アカウント作成）
→ CreateAccount は管理アカウントからしか実行できない
→ クロスアカウントIAMロールで解決
```

#### 信頼ポリシーのPrincipal
```
❌ lambda.amazonaws.com → 任意のアカウントのLambdaが引き受け可能（リスク）
✅ 専用アカウントのLambda実行ロールARN → 特定ロールのみに制限（安全）
```

### 覚え方
> **「Organizations委任管理 = CreateAccount は対象外」**  
> 信頼ポリシーのPrincipalにはサービスプリンシパルではなく特定のロールARNを指定する

---

## 問題11（P2）：Control Tower プロアクティブ vs ディテクティブコントロール ⭐ 難易度：中
**結果：不正解**

### 一言でいうと
「未然に防ぐ = プロアクティブコントロール（CloudFormation フックで作成前に検証）」

### カテゴリ別ポイント

#### Control Tower コントロールの2種類
| コントロール | タイミング | 仕組み | 阻止できるか |
|---|---|---|---|
| **プロアクティブ** ✅ | 作成"前" | CloudFormation フック | できる |
| ディテクティブ ❌ | 作成"後" | Config / CloudTrail | できない（検出のみ） |

### 罠
| 選択肢 | なぜダメ |
|---|---|
| Config ルール | 事後検出のみ。バケットは既に作られる |
| SCP | APIアクション制御はできるがバケット設定内容の詳細検証は困難 |
| ディテクティブコントロール | 検出はできるが阻止できない |

### 覚え方
> **「未然に防ぐ」→ プロアクティブ、「検出して通知」→ ディテクティブ（Config）**  
> CloudFormation で作るリソースを作成前に検証 → CloudFormation フック

---

## 問題37（P2）：SCP による IP アドレスベースの AWS API 制御 ⭐ 難易度：中
**結果：不正解**

### 一言でいうと
「指定範囲外のIPからのAWS APIアクションをブロック = SCPでDeny ＋ 組織ルートに適用」

### カテゴリ別ポイント

#### SCP vs Firewall Manager の役割（ここで選択ミス）
```
SCP（Service Control Policy）✅
  → AWS APIアクション（コンソール・CLI・SDK）を制御
  → aws:SourceIp 条件でIPアドレスベースの制御が可能

Firewall Manager / Network Firewall ❌
  → VPC内外のネットワークトラフィック（パケットレベル）を制御
  → AWS APIの呼び出し自体は制御できない
```

#### Deny vs Allow どちらを使うか
```
✅ 範囲外を Deny → 他のポリシーで上書き不可。確実にブロック
❌ 範囲内のみ Allow → 他のIAMポリシーで上書きされる可能性あり
```

### 覚え方
> **「AWSアカウントでのアクションをブロック → SCP」「VPCトラフィックをブロック → Network Firewall」**  
> SCP の明示的 Deny は最強（IAMポリシーでも上書き不可）
