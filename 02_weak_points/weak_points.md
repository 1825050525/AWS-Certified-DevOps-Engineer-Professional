# 弱点問題まとめ - DOP-C02

> 追記例：
> 「弱点を追加して。ドメイン：1（SDLC）、サービス：CodeDeploy、理由：Blue/GreenとCanaryの違いが曖昧、ポイント：Blue/Greenは完全な新環境に切替・Canaryは段階的にトラフィックを移行」

---

<!-- 弱点問題をここに蓄積する -->

## 問題33（P2）｜ドメイン2（IaC・設定管理）｜難易度：中
**サービス：SSM State Manager / Run Command / Patch Manager / Maintenance Window**
**理由：Patch Manager を選んだ。Patch Manager はパッチ適用専用で OS 設定管理ではない**
**ポイント：**
- 「起動後に即座に設定適用」→ State Manager の関連付け（Association）
- Patch Manager = パッチ・更新プログラムの適用のみ。OS 設定管理には使えない
- Maintenance Window = スケジュール実行。「即座に」には対応できない

---

## 問題29（P2）｜ドメイン4（モニタリング）｜難易度：中
**サービス：Kinesis / Lambda ParallelizationFactor / Enhanced Fan-out**
**理由：ReportBatchItemFailures を無効化を選んだ。これは信頼性機能で遅延と無関係**
**ポイント：**
- Kinesis 遅延改善 = シャード数増加 ＋ ParallelizationFactor ＋ Enhanced Fan-out
- ParallelizationFactor = 1シャードを複数 Lambda が並列処理（1〜10）
- ReportBatchItemFailures = 失敗レコードの報告機能。遅延改善とは無関係

---

## 問題17（P2）｜ドメイン1（SDLC）｜難易度：中
**サービス：CodeArtifact / アップストリームリポジトリ / ドメイン**
**理由：「チーム間リソースポリシー」を選んだ。N×(N-1)個の設定が必要でスケールしない**
**ポイント：**
- 共有ライブラリ = アップストリームリポジトリ（パッケージがない時に自動取得）
- 中央管理 = 共有サービスアカウントにドメイン ＋ CreateRepository権限で各チームが自分でリポジトリ作成
- チーム間の直接リソースポリシーは管理が煩雑でスケールしない

---

## 問題12（P2）｜ドメイン3（耐障害性）｜難易度：中
**サービス：S3 Batch Operations / S3 レプリケーション / Lambda**
**理由：SQS + Lambda で PutObject する構成を選んだ。大規模データには Lambda 単体は不向き**
**ポイント：**
- 大規模S3オブジェクトの再処理 → S3 Batch Operations（Lambda の時間・サイズ制限を回避）
- S3ReplicateObject = S3 ネイティブのレプリケーション再実行（ダウンロード→PutObject は非効率）
- Lambda はジョブ作成の「トリガー役」として使う

---

## 問題11（P2）｜ドメイン6（セキュリティ）｜難易度：中
**サービス：Control Tower コントロール / CloudFormation フック**
**理由：Config ルールで「未然に防ぐ」要件を満たせると思った。Config は事後検出のみ**
**ポイント：**
- 「未然に防ぐ」→ プロアクティブコントロール（CloudFormation フック）
- Config ルール = 事後検出。バケットは既に作られる → 阻止できない
- プロアクティブ vs ディテクティブ：「防ぐ」か「検出する」かで判断

---

## 問題8（P2）｜ドメイン6（セキュリティ）｜難易度：高
**サービス：Organizations / IAM クロスアカウント / Lambda**
**理由：Organizations 委任管理で CreateAccount できると思った。委任管理の対象外**
**ポイント：**
- organizations:CreateAccount は委任管理対象外。管理アカウントからのみ実行可能
- 解決策 = 管理アカウントにIAMロール → 専用アカウントのLambda実行ロールARNを信頼
- 信頼ポリシーのPrincipal に lambda.amazonaws.com は NG（任意のアカウントのLambdaが引き受け可能になる）

---

## 問題7（P2）｜ドメイン6（セキュリティ）｜難易度：高
**サービス：IAM ロール / 信頼ポリシー / PassRole / CloudFormation サービスロール**
**理由：CFNDeploymentロールの信頼ポリシーで開発者IAMロールを許可してしまった。cloudformation.amazonaws.com にすべきだった**
**ポイント：**
- CFNDeploymentロールの信頼ポリシーのPrincipal = cloudformation.amazonaws.com（開発者ではない）
- 開発者は PassRole で CFN にロールを渡すだけ（自分で引き受けない）
- iam:PassedToService 条件でロールを渡せるサービスを制限できる

---

## 問題6（P2）｜ドメイン4（モニタリング）｜難易度：高
**サービス：Container Insights / CloudWatch Agent / EKS**
**理由：ADOT と node_memory_utilization を選んだ。pod レベルのメトリクスが必要なことを見落とした**
**ポイント：**
- EKS アプリメモリ → pod_memory_utilization ＋ Service ディメンション（node 単位は粒度が粗い）
- CloudWatchAgentServerPolicy → インスタンスプロファイルの IAM ロールに付与（サービスアカウントではない）
- 新規 EC2 に自動でエージェントを入れたい → AMI に含める

---

## 問題4（P2）｜ドメイン4（モニタリング）｜難易度：高
**サービス：Kinesis Data Firehose / EventBridge / Network Firewall**
**理由：EventBridge の入力変換でログの中身を変換できると思った**
**ポイント：**
- EventBridge 入力変換はメタデータのみ。S3オブジェクトの中身は読めない
- ログをリアルタイムで加工 → Firehose ＋ Lambda（送信前に変換）
- Network Firewall のログ送信先 → S3 / CloudWatch Logs / Firehose の3択

---

## 問題76｜ドメイン1（SDLC）｜難易度：高
**サービス：EKS / IAM 信頼関係 / aws-auth ConfigMap / CodeBuild**
**理由：Control Tower 管理アカウントを経由する信頼関係を選んだ。EKS の aws-auth ConfigMap の存在を見落とした**
**ポイント：**
- EKS クロスアカウント = IAM信頼関係（アプリ側がDevOpsを信頼）＋ aws-auth ConfigMap 更新の2段階
- Control Tower は組織管理用。通常のクロスアカウントアクセスには不要
- IAM権限だけでは EKS に入れない。Kubernetes RBAC への aws-auth マッピングも必要

---

## 問題75｜ドメイン1（SDLC）｜難易度：中
**サービス：CodeBuild / ECR / Docker 認証**
**理由：Secrets Manager で ECR 認証できると思った。get-login-password による動的トークン取得が必要なことを知らなかった**
**ポイント：**
- ECR へのアクセス = IAMロール権限 ＋ `aws ecr get-login-password` → `docker login` が必須
- ECRトークンは12時間有効の一時トークン。Secrets Managerの静的認証情報とは別物
- S3は IAM権限だけで OK だが ECR は Docker 認証が追加で必要

---

## 問題61｜ドメイン1（SDLC）｜難易度：高
**サービス：CodePipeline / KMS / S3バケットポリシー / IAMロール**
**理由：IAMロール作成は正解だったが、S3バケットポリシーの更新が必要なことを見落とした**
**ポイント：**
- クロスアカウント = IAMロール（アイデンティティ側）＋ S3バケットポリシー（リソース側）の両方が必要
- KMS暗号化 ＋ クロスアカウント → カスタマーマネージドキー必須（AWSマネージドはキーポリシー変更不可）
- 「IAMロールに権限があるのにアクセス拒否」→ リソースポリシーが欠けているサイン

---

## 問題60｜ドメイン3（耐障害性）｜難易度：高
**サービス：AWS PrivateLink / VPCピアリング / NLB**
**理由：VPCピアリングを選んだ。同一CIDRでは使えないことと、190本という接続数の問題に気付けなかった**
**ポイント：**
- 同一CIDRのVPC間通信 → VPCピアリングは不可（CIDR重複）→ PrivateLink
- PrivateLink = 提供側にNLB ＋ 利用側にVPCエンドポイント（既存環境の変更最小）
- 20 VPC のピアリングは 190 本必要 → スケールしない

---

## 問題51｜ドメイン6（セキュリティ）｜難易度：中
**サービス：EventBridge / AWS Config / SNS / 入力変換**
**理由：「restricted-ssh 専用でフィルタ」ではなく「全NON_COMPLIANTを拾う」を選んでしまった**
**ポイント：**
- EventBridge は特定の Config ルール名で絞ることができる（全部拾うのは NG）
- 通知に詳細情報を含める = EventBridge の**入力変換（Input Transformation）**
- Config → SNS 直接通知は不可。必ず EventBridge 経由

---

## 問題44｜ドメイン1（SDLC）｜難易度：中
**サービス：CodeDeploy / ALB ヘルスチェック**
**理由：AllowTraffic ステージではエージェント未インストールが問題と考えたが、実際は ALB ヘルスチェック設定ミス**
**ポイント：**
- ログに理由がない = CodeDeploy の外（ALB 側）で失敗している
- AllowTraffic = ALB がインスタンスを Healthy と認めるまで待つステージ
- SSM Agent 未インストールなら BeforeInstall で失敗する（AllowTraffic まで到達しない）

---

## 問題43｜ドメイン2（設定管理・IaC）｜難易度：高
**サービス：Systems Manager Patch Manager / Run Command / SSM Agent / マネージドインスタンスアクティベーション**
**理由：Run Command でもパッチ管理ができると思った。IAM インスタンスプロファイルの設定が必須なことを見落とした**
**ポイント：**
- パッチ管理 = Patch Manager ＋ メンテナンスウィンドウ（Run Command は単体では不十分）
- オンプレミス登録 = マネージドインスタンスアクティベーション ＋ SSM Agent インストール
- EC2 には IAM インスタンスプロファイル、IoT/オンプレには IAM サービスロールが必要

---

## 問題32｜ドメイン6（セキュリティ）｜難易度：高
**サービス：Config コンフォーマンスパック / StackSets / 委任管理者**
**理由：コンフォーマンスパックの選択は正しかったが、StackSets 経由でデプロイすると選んでしまった**
**ポイント：**
- コンフォーマンスパックは Config ネイティブで組織全体に配布できる（StackSets 不要）
- デプロイは「委任管理者アカウントから Config で直接」が正解
- 管理アカウントは使わない（組織管理に専念させる）

---

## 問題29｜ドメイン6（セキュリティ）｜難易度：中
**サービス：Firewall Manager / AWS Config / GuardDuty / WAF**
**理由：GuardDuty で委任 + Config で自動適用できると思った。サービスの役割の違いを混同**
**ポイント：**
- 予防的WAF強制適用 → Firewall Manager（委任 ＋ ポリシー作成の2セット）
- Config は事後検出のみ。新規リソースへの即時適用はできない
- GuardDuty は脅威検出専用。WAF 管理機能はない

---

## 問題28｜ドメイン3（耐障害性）｜難易度：中
**サービス：Route 53 / ALB / DynamoDB グローバルテーブル**
**理由：フェイルオーバールーティングを遅延削減に使えると思った。「グローバルALB」という存在しないサービスを選んだ**
**ポイント：**
- 遅延削減 → レイテンシーベースルーティング（フェイルオーバーは障害対応用）
- ALB / Auto Scaling はリージョナルサービス。新リージョンに個別作成が必要
- DynamoDB の複数リージョン対応 → グローバルテーブル（手動レプリケーションは不要）

---

## 問題25｜ドメイン1（SDLC）｜難易度：高
**サービス：SAM / CodeDeploy / CloudWatch**
**理由：SAMがCodeDeployとappspec.ymlを自動管理することを知らなかった**
**ポイント：**
- SAM の DeploymentPreference 1行でカナリアデプロイ完結
- CodeDeploy の手動設定・appspec.yml の手動作成は SAM 利用時は不要
- 監視は CloudWatch 個別アラーム（複合アラームは粒度が粗い）
