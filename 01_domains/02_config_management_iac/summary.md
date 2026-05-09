# ドメイン2：設定管理とIaC

---

## 問題35：CloudFormation UPDATE_ROLLBACK_FAILED からの復旧 ⭐ 難易度：中
**結果：正解**

### 一言でいうと
「UPDATE_ROLLBACK_FAILED → IAM 権限を修正して `continue-update-rollback` を実行」

### カテゴリ別ポイント

#### CloudFormation スタックの状態遷移
```
UPDATE_IN_PROGRESS
  → 更新失敗
UPDATE_ROLLBACK_IN_PROGRESS
  → ロールバックも失敗
UPDATE_ROLLBACK_FAILED  ← 今回の状態
  → 通常の操作は全て不可
  → できるのは continue-update-rollback のみ
```

#### 回復手順（2ステップ）
```
① 失敗原因を特定・修正
   例）IAM ロールに不足権限を付与
      （autoscaling:* / ec2:* / iam:PassRole 等）

② aws cloudformation continue-update-rollback を実行
   → ロールバック処理を再開
   → スタックが UPDATE_ROLLBACK_COMPLETE に戻る
```

#### よくある失敗原因
| 原因 | 対処 |
|---|---|
| IAM 権限不足 | 必要な権限を IAM ロールに付与 |
| リソース依存関係の競合 | ResourcesToSkip でリソースをスキップ |
| サービス制限超過 | クォータ増加後に再実行 |

### 罠
| 選択肢 | なぜ間違い |
|---|---|
| `cancel-update-stack` | 更新中のキャンセル用。ロールバック失敗状態には効かない |
| `rollback-stack` | 存在しないコマンド |
| `update-stack-set` | StackSets 用。単一スタックには使わない |
| リソースを手動削除 | スタック外の変更は禁止 ＋ 整合性が壊れる |

### 覚え方
> **UPDATE_ROLLBACK_FAILED = `continue-update-rollback` 一択**  
> 実行前に CloudTrail で拒否 API を確認して権限を修正してから実行

---

## 問題59：UPDATE_ROLLBACK_FAILED からの2ステップ回復 ⭐ 難易度：低
**結果：正解**（Q35 の関連問題）

### 一言でいうと
「手動でリソースを修正 → ContinueUpdateRollback で再開」の2ステップ

### カテゴリ別ポイント

#### 正しい回復手順
```
① 手動修正：CloudFormation が期待する状態にリソースを合わせる
   （依存関係の削除・外部変更の元戻し等）
② ContinueUpdateRollback：ロールバックを中断地点から再開
```

#### ドリフト検出はなぜダメか
- ドリフト検出 = 差分を**確認するだけ**。自動回復機能はない

### 覚え方
> **「まず手動で原因を取り除いてから、ContinueUpdateRollback で再開」**  
> ドリフト検出は「診断」ツール。「回復」ツールではない

---

## 問題43：Systems Manager ハイブリッド環境パッチ管理 ⭐⭐ 難易度：高
**結果：不正解**

### 一言でいうと
「ハイブリッド環境の統合パッチ管理 = IAM設定 ＋ SSMエージェント登録 ＋ Patch Manager ＋ メンテナンスウィンドウ」

### カテゴリ別ポイント

#### リソース種別ごとの登録方法（ここが核心）
| リソース | IAM 設定 | SSM 登録方法 |
|---|---|---|
| EC2 インスタンス | IAM **インスタンスプロファイル** | 起動時に自動（SSM Agent 標準搭載） |
| IoT Greengrass デバイス | IAM **トークン交換ロール**を更新 | Greengrass 経由で SSM Agent をデプロイ |
| オンプレミスサーバー | IAM **サービスロール** | マネージドインスタンスアクティベーション ＋ 手動 SSM Agent インストール |

#### パッチ管理の正しい構成
```
Patch Manager（ベースライン定義 + 自動スキャン）
    ↓
メンテナンスウィンドウ（業務時間外スケジュール）
    ↓
EC2 / IoT Greengrass / オンプレミス を統合管理
```

#### Run Command vs Patch Manager（ここで選択ミス）
| 機能 | 用途 |
|---|---|
| **Patch Manager** ✅ | パッチ管理専用。ベースライン・承認・スケジュール対応 |
| Run Command ❌ | 個別コマンドの即時実行。単体ではパッチ管理に不十分 |
| Session Manager ❌ | インタラクティブなシェルアクセス。パッチ配布機能なし |

### 罠
| 選択肢 | なぜ間違い |
|---|---|
| Run Command でパッチ配布 | ベースライン管理・承認・スケジュール機能がない |
| EventBridge + Run Command | 動くが過度に複雑。Patch Manager の標準機能で足りる |
| Session Manager でパッチ配布 | パッチ配布機能は持っていない |

### 覚え方
> オンプレミス/IoT を SSM に登録するには「アクティベーション ＋ SSM Agent」が必須  
> パッチ管理は **Patch Manager**。Run Command は「その場で1回実行」用

---

## 問題50：AMI ID の自動配布（EC2 Image Builder ＋ Parameter Store ＋ CloudFormation） ⭐ 難易度：低
**結果：正解**

### 一言でいうと
「AMI自動作成 = Image Builder、配布 = Parameter Store、参照 = CloudFormation動的参照」

### カテゴリ別ポイント

#### 全体の流れ
```
EC2 Image Builder → AMI自動作成 → Parameter Store に ARN を保存
                                          ↓
CloudFormation {{resolve:ssm:パラメータ名}} で自動参照
→ デプロイ時に常に最新AMIを取得（テンプレート変更不要）
```

#### CloudFormation 動的参照
```yaml
ImageId: "{{resolve:ssm:/company/approved-ami/latest}}"
# Parameter Store の値を更新するだけで全チームに反映
```

#### 他の選択肢が不適切な理由
| 手法 | 問題点 |
|---|---|
| SNS + Lambda | 各チームが Lambda を実装・保守する必要がある |
| S3 オブジェクト経由 | CloudFormation からの参照が複雑（カスタムリソース必要） |
| CodePipeline 経由 | AMI配布だけに対してアーキテクチャが過剰 |

### 覚え方
> **「AMI配布はParameter Store一択」「CloudFormationは`{{resolve:ssm}}`で動的参照」**  
> テンプレートを変更せず、パラメータ更新だけで全チームに反映されるのが強み

---

## 問題55：CloudFormation cfn-init ＋ cfn-hup による設定管理 ⭐ 難易度：中
**結果：正解**

### 一言でいうと
「起動時の初期設定 = cfn-init、テンプレート変更後の自動更新 = cfn-hup」

### カテゴリ別ポイント

#### cfn-init と cfn-hup の役割分担
| ツール | タイミング | 役割 |
|---|---|---|
| **cfn-init** | 起動時に1回だけ | Metadata を読んで初期設定を適用 |
| **cfn-hup** | 起動後もずっと動き続ける | Metadata 変更を定期監視 → 差分を自動適用 |

```
cfn-init だけでは → 起動後のテンプレート変更に追従できない
cfn-hup がセットで → 既存インスタンスにも変更が自動反映される
```

### 覚え方
> **「cfn-init = 生まれた時に設定する、cfn-hup = 変化に追従し続ける」**  
> 2つセットで初めて「迅速な更新反映」要件を満たす

---

## 問題33（P2）：EC2 起動後に即座に設定適用（SSM State Manager） ⭐ 難易度：中
**結果：不正解**

### 一言でいうと
「起動後に即座に設定を適用 → State Manager の関連付け（Association）」

### カテゴリ別ポイント

#### SSM の4機能の使い分け
| 機能 | 役割 | 「即座に」対応 |
|---|---|---|
| **State Manager** ✅ | 設定状態の定義・維持・即時適用 | ✅（SSM Agent 起動時に即時） |
| Run Command | オンデマンドのコマンド実行 | ❌（手動トリガーが必要） |
| Maintenance Window | 定期スケジュール実行 | ❌（最大24時間待つ可能性） |
| **Patch Manager** ❌（選んだ答え） | パッチ・更新プログラムの適用 | ❌（OS設定管理ではない） |

### 覚え方
> **「起動後に即座に設定 → State Manager」「パッチを当てる → Patch Manager」**  
> State Manager = 「この状態であるべき」を定義し続けるサービス（ドリフト防止も兼ねる）

---

## 問題56：CloudFormation ネストスタック ＋ StackSets ＋ SNS通知 ⭐ 難易度：低
**結果：正解**

### 一言でいうと
「順序制御 = ネストスタック、マルチリージョン展開 = StackSets、通知 = SNS」

### カテゴリ別ポイント

#### ネストスタック vs StackSets の役割分担
| 機能 | 用途 |
|---|---|
| **ネストスタック** | 複数スタック間の依存関係・実行順序を制御 |
| **StackSets** | 複数リージョン・アカウントへの一括展開 |

#### 通知の流れ
```
CloudFormation イベント → EventBridge → SNS → データエンジニアリング部門
```

#### 罠
- **ドリフト検出** → 差異検出機能。リアルタイム変更通知ではない
- **S3イベントトリガー** → 非同期のため順序保証が困難

### 覚え方
> **「ネストスタック = 順序制御、StackSets = マルチリージョン」**  
> ドリフト検出を「変更通知」と混同しない

---

## 問題73：ネストスタック ＋ Fn::ImportValue ＋ ChangeSet ⭐ 難易度：中
**結果：正解**

### 一言でいうと
「再利用可能な開発環境 = ネストスタック ＋ Fn::ImportValue（Resources内）＋ ChangeSet で安全更新」

### カテゴリ別ポイント

#### Fn::ImportValue を書ける場所（よく間違える）
```
✅ Resources / Outputs / Conditions → 使える
❌ Parameters → 使えない（ユーザー入力値の定義場所）
```

#### ネストスタック vs StackSets
| | ネストスタック ✅ | StackSets ❌ |
|---|---|---|
| 用途 | 単一アカウント内での環境モジュール化・再利用 | 複数アカウント・複数リージョンへ一括展開 |
| 今回の要件 | ✅ 適切 | ❌ 過剰 |

### 覚え方
> **「Fn::ImportValue は Parameters に書けない。Resources/Outputs/Conditions のみ」**  
> StackSets は複数アカウント用。単一アカウント環境管理はネストスタック
