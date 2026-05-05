# 弱点問題まとめ - DOP-C02

> 追記例：
> 「弱点を追加して。ドメイン：1（SDLC）、サービス：CodeDeploy、理由：Blue/GreenとCanaryの違いが曖昧、ポイント：Blue/Greenは完全な新環境に切替・Canaryは段階的にトラフィックを移行」

---

<!-- 弱点問題をここに蓄積する -->

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
