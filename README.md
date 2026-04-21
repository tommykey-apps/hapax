# dropfast

一時ファイル共有サービス — アップロードすると期限・回数制限付きの URL が発行され、ダウンロードが 1 回 (または指定回数) 終わると S3 から削除される。クライアントサイド暗号化でゼロ知識。

🌐 https://dropfast.tommykeyapp.com/ (未デプロイ)

## 予定スタック

| | |
|---|---|
| バックエンド | **Rust + axum** (Lambda arm64, `cargo-lambda` でビルド) |
| フロント | SvelteKit 2 (burnnote から流用) |
| DB | DynamoDB (メタデータ、TTL) |
| ストレージ | **S3 + presigned URL** (アップロード / ダウンロード両方、Lifecycle で物理削除) |
| 暗号化 | クライアントサイド AES-256-GCM (WebCrypto API、burnnote と同方式) |
| IaC | Terraform |
| CI/CD | GitHub Actions |
| 配信 | CloudFront (S3 SPA + API Gateway デュアルオリジン) |

## 機能予定

- **直接アップロード不可**: クライアントが暗号化 → `POST /api/uploads/init` でメタ登録 → presigned PUT URL を受け取り → S3 に直接アップロード
- 発行 URL: `https://dropfast.tommykeyapp.com/d/{id}#{key}` (鍵は fragment、burnnote と同じゼロ知識)
- ダウンロード: presigned GET URL を 1 回限り発行 → クライアントで復号 → 完了後にメタ削除
- パスワード追加オプション (PBKDF2 で URL fragment 鍵を強化)
- S3 Lifecycle で期限切れオブジェクトを強制削除 (DynamoDB TTL と二重の安全策)

## 開発ステータス

[open issues](https://github.com/tommykey-apps/dropfast/issues) を参照。
