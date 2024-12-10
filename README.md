# 「EC2へのLaravelデプロイ」マニュアル（初版）

!> 12月10日に開始したばかりです

?> 　生成AIも活用して記載しています

!> **Important** notice with `inline code` and additional placeholder text used
to force the content to wrap and span multiple lines.

?> **Tip** notice with `inline code` and additional placeholder text used to
force the content to wrap and span multiple lines.

!> 簡単にLaravelでプロジェクトを作成し試すことをお勧めします


## 1. マニュアル概要
- マニュアルの目的
  - AWS環境を活用し、LaravelアプリケーションをEC2にデプロイする基本的な手順を習得する。
  - インフラ構成要素（ALB, EC2, S3, RDS, SES, CloudWatch）の連携を理解する。

---

## 2. AWSのインフラ構成概要
### 2.1 ALB（ロードバランサー）
- ALBの役割
## 2.1.1 トラフィックの分散
ALBは、ユーザーから送られてくるリクエスト（HTTP/HTTPS）を複数のEC2インスタンスやコンテナサービス（ECSやEKS）に均等に分散します。これにより、次のようなメリットがあります：

- サーバー負荷のバランスを保つ
- サーバーダウンによるサービス停止のリスクを軽減
- リソースの効率的な利用
## 2.1.2 アプリケーションレベルでのルーティング
ALBはリクエスト内容に基づいて柔軟なルーティングを行えます。

- パスベースルーティング
?> 例：/api/*のリクエストはバックエンドのAPIサーバーへ、/static/*はS3や別のサーバーへ送る。
- ホストベースルーティング
?>例：api.example.comはAPIサーバーに、www.example.comはフロントエンドサーバーにリクエストを振り分ける。
- HTTPヘッダーやメソッドに基づくルーティング
- ヘッダーやクエリパラメータに基づいてターゲットを切り替える。
## 2.1.3 スケーラビリティ
ALBはAWS Auto Scalingと連携可能です。トラフィックが増加すると自動的に新しいインスタンスを追加し、負荷に応じてサーバーの数を調整します。

## 2.1.4 セキュリティ機能
ALBには以下のセキュリティ関連の機能があります：

- HTTPSによる通信の暗号化
- SSL証明書を利用して通信を暗号化する。
- AWS WAF（Web Application Firewall）との統合
- 不正なリクエストをブロックする。
- セキュリティグループとの連携
- 特定のIPやポートからの接続を制限する。
## 2.1.5 ヘルスチェック
ALBはバックエンドのターゲット（EC2インスタンスやコンテナ）が正常に動作しているかを定期的に確認します。

異常なターゲットへのリクエスト送信を停止。
ターゲットが復旧すると、自動的にトラフィックを再開。
6. ログとモニタリング
ALBは以下の監視機能を提供し、運用管理を支援します：

アクセスログ
リクエストの詳細情報（元のIP、リクエストURI、レスポンスコードなど）を記録。
CloudWatchとの統合
メトリクス（リクエスト数、エラー率、ターゲットの応答時間など）を可視化。
ALBを利用するメリット
高可用性
負荷分散により、サーバー障害が発生してもサービスを継続できる。
スケーラビリティ
トラフィック量に応じた動的なスケーリングが可能。
運用効率
自動化されたヘルスチェックやルーティングによる管理の簡略化。
柔軟性
アプリケーションの構造やリクエスト内容に応じたきめ細かいルーティング設定。
実際の利用例
ECサイト
高トラフィック時に複数のEC2インスタンスで負荷を分散。
特定のサブドメインで異なるサービスを提供。
マイクロサービス
APIゲートウェイとして機能し、各サービスにリクエストを振り分ける。
SaaSアプリケーション
エンドユーザーに対する高可用性の提供。

- ALBの設定とLaravelアプリとの連携方法

### 2.2 EC2（サーバー）
- EC2の役割
- EC2インスタンスのセットアップ手順
  - Amazon Linux 2またはUbuntuの選定
  - 必要なパッケージ（PHP, Composer, Gitなど）のインストール

### 2.3 S3（ストレージ）
- S3の利用目的
  - 静的ファイル（画像やCSS）の管理
- S3バケットの作成と設定方法

### 2.4 RDS（データベース）
- RDSのセットアップ
  - MySQLまたはPostgreSQLの設定
- Laravelの環境ファイル（`.env`）との接続設定

### 2.5 SES（メール送信）
- SESのセットアップ
  - メール送信ドメインの認証
  - Laravelアプリケーションとの統合

### 2.6 CloudWatch（ログ監視）
- CloudWatchの役割
  - ログの統合とSlack通知設定
- ALBログの収集と分析

---

## 3. 必要な基礎知識
### 3.1 SSH接続
- SSHの基本操作
  - 秘密鍵・公開鍵の生成
  - EC2への接続方法

### 3.2 Deployerの基本操作
- Deployerのインストール
- Deployerの設定ファイル（`deploy.php`）の作成方法

### 3.3 GitHub Actionsの概要
- GitHub Actionsの基本構造
  - ワークフロー（`.github/workflows`）の作成
- サーバーへの接続設定
  - シークレット（Secrets）の利用

---

## 4. ローカル環境での準備
### 4.1 Laravelプロジェクトの準備
- Laravelアプリケーションのローカル開発環境構築
- `.env`ファイルの設定

### 4.2 Deployerの動作確認
- ローカル環境でのデプロイ実行
  - SSH接続を利用したサーバーへの簡易デプロイテスト

---

## 5. GitHub Actionsの設定
### 5.1 Secretsの設定
- GitHubリポジトリで必要な環境変数の登録
  - SSH鍵
  - AWSの認証情報（Access Key, Secret Key）

### 5.2 ワークフローの作成
- LaravelプロジェクトをデプロイするGitHub Actionsのワークフロー作成
  - Deployerコマンドの自動実行設定

---

## 6. デプロイ手順
### 6.1 ローカル環境でのテストデプロイ
- Deployerを使用したテストデプロイ手順
- エラー対応例

### 6.2 GitHub Actionsでのデプロイ
- ワークフロー実行方法
- 成功時・失敗時の対応方法

---

## 7. 運用・監視
### 7.1 CloudWatchでのログ監視
- ログ収集とアラート設定
- Slack通知の実装方法

### 7.2 継続的改善
- 新たなエラー発生時のトラブルシューティング方法
- 新機能追加時のデプロイ手順更新

---
