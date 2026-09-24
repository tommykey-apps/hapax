# dropfast

一時ファイル共有サービス。アップロードすると期限と回数制限付きの URL が発行され、指定回数のダウンロードが終わると削除される。暗号化はブラウザで行い、サーバーは平文もファイル名も知らない。

https://dropfast.tommykeyapp.com/ (未デプロイ)

## 予定スタック

| 層 | 採用 |
|---|---|
| 配信 | CloudFront。画面は S3、`/api/*` は API Gateway (HTTP API) |
| API | TypeScript + Hono on Lambda (Node.js 22、arm64) |
| メタデータと回数制御 | DynamoDB (条件付き更新で回数を減らす) |
| ファイル本体 | S3 (署名付き URL でブラウザから直接読み書き) |
| 削除 | EventBridge Scheduler の1回きりの予約で削除用 Lambda を呼ぶ。S3 のライフサイクル規則と DynamoDB の有効期限 (Time To Live, TTL) は保険 |
| 画面 | Vite + React の単一ページアプリケーション (Single Page Application, SPA)。デジタル庁デザインシステムのトークンと React 版サンプル部品 |
| 暗号化 | ブラウザ側で Advanced Encryption Standard Galois/Counter Mode (AES-GCM) 256bit、64KiB ごとのチャンク暗号化 (WebCrypto) |
| 不正利用対策 | Turnstile、API Gateway のスロットリング |
| Infrastructure as Code (IaC) | Terraform |
| ローカル開発 | DynamoDB Local と S3Mock (docker compose) |
| CI/CD | GitHub Actions |
| 運用指標 | CloudWatch (埋め込みメトリクス形式とダッシュボード) |

## 機能予定

- ファイル本体は API を通さない。ブラウザが暗号化し、`POST /api/uploads` で受け取った S3 の署名付き PUT URL へ直接送る
- 発行 URL: `https://dropfast.tommykeyapp.com/d/{id}#{key}`。鍵は fragment にあり、サーバーに送られない
- ダウンロード: `POST /api/downloads/{id}/consume` で DynamoDB の残り回数を減らし、署名付き GET URL を返す。ブラウザで復号する
- 回数が0になったら、署名付き URL の期限が切れた後に EventBridge Scheduler の予約で S3 から削除する
- パスワード追加オプション (Password-Based Key Derivation Function 2 (PBKDF2) で fragment の鍵を包む)
- 期限切れも同じ予約で削除し、S3 のライフサイクル規則 (8日) で取りこぼしを消す

## 開発ステータス

[open issues](https://github.com/tommykey-apps/dropfast/issues) を参照。作業順は #1 の冒頭に書いてある。
