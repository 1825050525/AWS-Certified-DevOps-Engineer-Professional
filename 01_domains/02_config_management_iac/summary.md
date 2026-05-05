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
