# dropfast

一時ファイル共有サービス。アップロードすると期限と回数制限付きの URL が発行され、指定回数のダウンロードが終わると R2 から削除される。暗号化はブラウザで行い、サーバーは平文もファイル名も知らない。

https://dropfast.tommykeyapp.com/ (未デプロイ)

## 予定スタック

| 層 | 採用 |
|---|---|
| 配信 | Cloudflare Workers 1本 (画面は Workers Static Assets、API は Hono) |
| 画面 | Vite + React + TypeScript の単一ページアプリケーション (Single Page Application, SPA)。デジタル庁デザインシステムのトークンと React 版サンプル部品 |
| メタデータと回数制御 | SQLite 版 Durable Objects (ファイル1件に1オブジェクト、アラームで期限切れを削除) |
| ファイル本体 | R2 (署名付き URL でブラウザから直接読み書き、ライフサイクル規則で保険の削除) |
| 暗号化 | ブラウザ側で Advanced Encryption Standard Galois/Counter Mode (AES-GCM) 256bit、64KiB ごとのチャンク暗号化 (WebCrypto) |
| 不正利用対策 | Turnstile、Workers Rate Limiting |
| 設定と Infrastructure as Code (IaC) | `wrangler.jsonc`。R2・オリジン間リソース共有 (Cross-Origin Resource Sharing, CORS)・Turnstile は Terraform (Cloudflare provider v5) |
| CI/CD | GitHub Actions + `cloudflare/wrangler-action` |
| 運用指標 | Workers Logs、Analytics Engine |

## 機能予定

- ファイル本体は Worker を通さない。ブラウザが暗号化し、`POST /api/uploads` で受け取った R2 の署名付き PUT URL へ直接送る
- 発行 URL: `https://dropfast.tommykeyapp.com/d/{id}#{key}`。鍵は fragment にあり、サーバーに送られない
- ダウンロード: `POST /api/downloads/{id}/consume` で Durable Object が残り回数を減らし、署名付き GET URL を返す。ブラウザで復号する
- 回数が0になったら、署名付き URL の期限が切れた後に Durable Object のアラームで R2 から削除する
- パスワード追加オプション (Password-Based Key Derivation Function 2 (PBKDF2) で fragment の鍵を包む)
- 期限切れは Durable Object のアラームで削除し、R2 のライフサイクル規則 (8日) で取りこぼしを消す

## 開発ステータス

[open issues](https://github.com/tommykey-apps/dropfast/issues) を参照。
