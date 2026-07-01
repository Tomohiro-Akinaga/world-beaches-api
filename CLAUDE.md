# CLAUDE.md

このファイルは、このリポジトリで作業する際の Claude Code（claude.ai/code）向けガイダンスです。

## プロジェクト概要

「世界の名ビーチ API」— 世界の有名なビーチの情報（日本語紹介文・画像URLなど）を返す**読み取り専用の REST API**。個人開発。

- 参照系（`GET`）のみ。認証・書き込み系APIはスコープ外。
- ランタイムでの外部API呼び出しはなし。画像はクライアントが Cloudflare R2 から直接取得する。
- データは seed スクリプトでDBへ投入する（admin画面なし）。

詳細は以下を参照:
- 要件定義: [docs/requirements.md](docs/requirements.md)
- 基本設計（外部設計・データ設計）: [docs/basic-design.md](docs/basic-design.md)

## 構成

- リポジトリルート直下に `backend/`（NestJS アプリ）がある。**開発コマンドは `backend/` で実行する。**
- CI は [.github/workflows/ci.yml](.github/workflows/ci.yml)（`backend/` を作業ディレクトリに Lint / Build / Test を実行）。

## 開発コマンド

すべて `backend/` ディレクトリで実行する。

```bash
npm install          # 依存インストール
npm run start:dev    # 開発サーバ（watch）。デフォルト http://localhost:3000
npm run build        # ビルド（nest build）
npm run lint         # ESLint（--fix）
npm run format       # Prettier
npm test             # Jest（*.spec.ts）
npm run test:e2e     # E2E テスト
npm run test:cov     # カバレッジ
```

CI と同じチェックをローカルで再現する場合:

```bash
npx eslint "{src,test}/**/*.ts" --max-warnings=0
npm run build
npm test
```

## 技術スタック

| 区分 | 採用 |
| --- | --- |
| 言語 | TypeScript |
| フレームワーク | NestJS 11 |
| DB / ORM | PostgreSQL / Prisma（予定） |
| バリデーション | class-validator / class-transformer（予定） |
| APIドキュメント | @nestjs/swagger（予定） |
| テスト | Jest / Supertest |
| 画像ホスティング | Cloudflare R2 |

## 設計上の重要な決まりごと

- **API ベースパス**: `/api/v1`。破壊的変更時のみ `/api/v2` を切る。
- **レスポンス形式**: 一覧系は `data`（配列）+ `meta`（`total` / `page` / `limit` / `totalPages`）。個別系は単一オブジェクト（`data` でラップしない）。
- **エラー形式**: NestJS 標準の例外フィルタ形式（`statusCode` / `message` / `error`）に統一。
- **ソート**: `id` 昇順固定。
- **国フィルタ**: 英語 `country` 値への完全一致・大文字小文字無視。
- **seed**: `slug` をキーに **upsert**（再実行しても重複・全削除が起きない）。データ正本はリポジトリ内のTSファイル、DBはその再現コピー。DBへ直接書き込まない。
- **設定値**: DB接続情報・許可オリジン・R2 ドメインは環境変数（`.env`、Git管理外）。

## コーディング規約

- Prettier（[backend/.prettierrc](backend/.prettierrc)）と ESLint（[backend/eslint.config.mjs](backend/eslint.config.mjs)）に従う。
- CI は `--max-warnings=0` なので、警告も残さないこと。
- NestJS の標準構成（module / controller / service / DTO）に従う。

## コミット規約

- コミットメッセージは **Conventional Commits** 形式に従う。
- 形式: `<type>: <subject>`（必要に応じてスコープ `<type>(<scope>): <subject>`）。
- **`<subject>`（説明部分）は日本語で書く。** `<type>` とスコープは英語のまま。
- 主な type: `feat` / `fix` / `docs` / `chore` / `refactor` / `test` / `ci`。
- 例: `docs: CLAUDE.md を追加`, `feat(beach): 一覧エンドポイントを追加`

## プルリクエスト規約

- **PR のタイトルはコミットメッセージと同じ形式**（`<type>: <subject>`、Conventional Commits）で書く。
- `<type>` とスコープは英語、`<subject>`（説明部分）は日本語で書く。
- 内容が一目で分かる簡潔なタイトルにする。
- 本文は [.github/pull_request_template.md](.github/pull_request_template.md) のテンプレートに従う。
- 例: `feat(beach): ビーチ一覧APIを追加`, `fix: 国フィルタの大文字小文字判定を修正`
