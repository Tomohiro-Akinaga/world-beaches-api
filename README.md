<!-- ロゴ・バナー画像があればここに配置（任意） -->
<!-- 例: <img src="docs/images/logo.png" width="120" /> -->

<div align="center">

# 世界の名ビーチ API

**世界の名ビーチを、統一された「読み物」として届ける読み取り専用 REST API**

<!-- バッジ（不要なものは削除）。技術スタックに合わせて調整してください -->
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![NestJS](https://img.shields.io/badge/NestJS-E0234E?logo=nestjs&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?logo=prisma&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![License](https://img.shields.io/badge/license-All%20Rights%20Reserved-red)

[概要](#概要) ・ [主な機能](#主な機能) ・ [技術スタック](#技術スタック) ・ [セットアップ](#セットアップ)

</div>

---

## 概要

世界の著名なビーチの情報は各メディア（CNN・Tripadvisor・Lonely Planet など）に散在し、選定基準もばらばらで、機械可読な統一データソースが存在しません。主要なリストの一部は更新も止まっています。

このアプリは、著名なビーチを独自にキュレーションし、日本語の紹介文を主コンテンツとした統一 REST API として提供します。一覧・個別取得・国フィルタに対応した**読み取り専用 API** です（NestJS / PostgreSQL / Prisma の学習・ポートフォリオを主目的としています）。

## 主な機能

<!-- 「ユーザーが何をできるか」を機能単位で。実装済みと開発中を分けると親切。 -->

- 🚧 ビーチ一覧取得：`page` / `limit` によるページネーション付きで一覧を取得できる（`id` 昇順）
- 🚧 ビーチ個別取得：`slug` を指定して 1 件を取得できる（存在しなければ `404`）
- 🚧 国によるフィルタ：`country` で絞り込める（英語値への完全一致・大文字小文字無視）
- 🚧 API 仕様書：`@nestjs/swagger` による OpenAPI 自動生成

> 現在は要件定義・基本設計が完了し、実装に着手する段階です。

## 技術スタック

<!-- 自分が触れた範囲を正直に。バージョンを書くと再現性・信頼性が上がる。 -->

| 領域 | 使用技術 |
| --- | --- |
| 言語 | TypeScript 5 |
| フレームワーク | NestJS 11 |
| データベース | PostgreSQL 16 |
| ORM | Prisma（予定） |
| バリデーション | class-validator / class-transformer（予定） |
| API ドキュメント | @nestjs/swagger（OpenAPI 自動生成・予定） |
| テスト | Jest / Supertest |
| 画像ホスティング | Cloudflare R2 |
| インフラ・CI/CD | GitHub Actions（Lint / Build / Test） |

## アーキテクチャ

<!-- 任意だが、技術的な判断を示せる強力なセクション。 -->

```
Client ──HTTP/JSON──▶ NestJS REST API ──Prisma──▶ PostgreSQL
   │
   └───画像取得──────▶ Cloudflare R2

Seed スクリプト ─初期投入/順次追加─▶ PostgreSQL（画像は R2 にホスト）
```

- ランタイムでの外部 API 呼び出しはなし。画像はクライアントが Cloudflare R2 から直接取得する。
- データは seed スクリプトで DB へ投入する（`slug` をキーに upsert、admin 画面なし）。
- 詳細は [要件定義書](docs/requirements.md) / [基本設計書](docs/basic-design.md) を参照。

採用理由：小規模な単一リソース API のため、NestJS の標準構成（module / controller / service / DTO）で 1 モジュールに収め、型安全な Prisma で DB アクセスを統一した。

## セットアップ

### 前提環境

- Node.js >= 20
- Docker（PostgreSQL をコンテナで起動）

### インストールと起動

```bash
# リポジトリのクローン
git clone https://github.com/Tomohiro-Akinaga/world-beaches-api.git
cd world-beaches-api/backend

# 依存関係のインストール
npm install

# 環境変数の設定（.env.example をコピーして編集）
cp .env.example .env

# PostgreSQL コンテナの起動
docker compose up -d

# スキーマ適用とデータ投入
npx prisma migrate dev
npx prisma db seed

# 開発サーバーの起動（デフォルト http://localhost:3000）
npm run start:dev
```

> 開発コマンドは `backend/` ディレクトリで実行します。

### 環境変数

| 変数名 | 説明 | 必須 |
| --- | --- | :---: |
| `DATABASE_URL` | PostgreSQL 接続文字列 | ✅ |
| `CORS_ORIGIN` | 許可する CORS オリジン | |
| `R2_PUBLIC_DOMAIN` | 画像配信用の Cloudflare R2 ドメイン | |

## 使い方

<!-- 主要なユースケースを手順で。 -->

```bash
# 一覧取得（ページネーション）
curl "http://localhost:3000/api/v1/beaches?page=1&limit=20"

# 国で絞り込み（完全一致・大文字小文字無視）
curl "http://localhost:3000/api/v1/beaches?country=Seychelles"

# slug で個別取得
curl "http://localhost:3000/api/v1/beaches/grande-anse-la-digue"
```

一覧系は `data`（配列）+ `meta`（`total` / `page` / `limit` / `totalPages`）、個別系は単一オブジェクトを返します。

## ディレクトリ構成

<!-- 任意。設計意図を伝えたい場合に。 -->

```
.
├── backend/            # NestJS アプリ（開発コマンドはここで実行）
│   ├── src/            # module / controller / service / DTO
│   ├── prisma/         # schema.prisma / seed.ts / beaches.ts（データ正本）
│   └── test/           # E2E テスト
├── docs/               # 要件定義・基本設計
└── .github/workflows/  # CI（Lint / Build / Test）
```

## 工夫した点・技術的なチャレンジ

<!-- ポートフォリオとして最も差がつくセクション。 -->

- **データ正本を Git 管理下に置く設計**：DB へ直接書き込まず、リポジトリ内の TS ファイルを正本とし、`slug` をキーに upsert する seed 設計により、再実行しても重複・全削除が起きず、順次追加に耐える再現可能なデータ運用を実現した。
- **設計を先に固める**：要件定義書・基本設計書を先行して作成し、API 仕様・エラー形式・ページネーションの契約を確定させてから実装に入る進め方を採用した。

## 今後の展望

- [ ] Prisma スキーマ・マイグレーション整備
- [ ] Beach モジュール（一覧・個別・国フィルタ・ページネーション）の実装
- [ ] `@nestjs/swagger` による API 仕様書の公開
- [ ] 紹介文・画像データの拡充とデプロイ

## ライセンス

**All Rights Reserved.**

本リポジトリ内のソースコードおよび関連ファイルの著作権は作者に帰属します。
ソースコードの閲覧は自由ですが、作者の許可なく以下の行為を行うことを禁止します。

- 複製・改変・再配布
- 商用・非商用を問わない再利用
- 本コードを利用した派生物の公開

利用をご希望の場合は、[作者](#作者)までお問い合わせください。

## 作者

- Name: Tomohiro Akinaga
- Portfolio: <!-- 公開URLがあれば記載 -->
- X: <!-- @username -->
- GitHub: [@Tomohiro-Akinaga](https://github.com/Tomohiro-Akinaga)

---
