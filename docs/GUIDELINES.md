# 技術選定・設計ガイドライン

プロジェクト横断で適用する技術選定の方針と設計判断の基準をまとめたドキュメント。
新規プロジェクト立ち上げ時や、技術的な意思決定の際にここを参照する。

---

## 言語

### TypeScript を原則使用

プラットフォームによらず、コードは極力 TypeScript で記述する。
よほどの理由（プラットフォーム制約・既存資産の引き継ぎ等）がない限り、別言語を選択しない。

---

## パッケージ管理

### pnpm を使用

TypeScript / JavaScript を使うすべてのプロジェクトで pnpm を採用する。
npm・yarn は使用しない。

---

## リポジトリ構成

### 単一プラットフォームの場合

シンプルな単一 package 構成でよい。

### 複数プラットフォームが混在する場合（例：Webフロント + サーバー処理）

**pnpm workspace を使ったモノレポ構成**にする。

```
/
├── packages/
│   ├── core/          # 全 package 共通の型・スキーマ定義
│   ├── web/           # Next.js フロント
│   ├── functions/     # Firebase Functions 等のサーバー処理
│   └── ...
├── pnpm-workspace.yaml
└── package.json
```

#### `core` package の役割

- すべての package から参照可能な共通リソースを置く
- Zod スキーマ・共通型定義・定数などが対象
- ビジネスロジックは含めない（あくまで型・スキーマの置き場）

---

## Lint

### ESLint を必ず導入

すべてのプロジェクトで ESLint を導入する。
モノレポ構成の場合はすべての package に ESLint を適用する。
設定フォーマットは **flat config（`eslint.config.mjs`）** を使用する。

#### ルール管理の方針

共通ルールをリポジトリルートの `eslint.shared.mjs` に export し、各 package の `eslint.config.mjs` で spread して継承する。

```js
// eslint.shared.mjs（ルート）
export const sharedRules = {
  // TypeScript
  "no-unused-vars": 0,
  "@typescript-eslint/no-unused-vars": 2,
  "@typescript-eslint/no-explicit-any": 2,
  "@typescript-eslint/explicit-member-accessibility": 2,
  "@typescript-eslint/no-unnecessary-type-assertion": 2,
  "@typescript-eslint/consistent-type-imports": ["error", {
    prefer: "type-imports",
    fixStyle: "inline-type-imports",
    disallowTypeAnnotations: true
  }],
  "@typescript-eslint/consistent-type-exports": "error",
  // import
  "import/consistent-type-specifier-style": ["error", "prefer-inline"],
  "import/no-duplicates": 2,
  // 相対パス import 禁止（後述）
  "no-restricted-imports": ["error", { patterns: ["./", "../"] }]
};
```

```js
// packages/web/eslint.config.mjs（各 package）
import { sharedRules } from "../../eslint.shared.mjs";

export default [
  {
    rules: {
      ...sharedRules,
      // package 固有のルールをここに追記
    }
  }
];
```

#### Prettier との併用

フォーマットは Prettier に委ねる。`.prettierrc` をリポジトリルートに置く。
ESLint のフォーマット系ルールは設定しない（競合を避けるため）。

---

## Import

### エイリアスを使用し、相対パス import を禁止

あらゆるプラットフォーム・package において、相対パスによる import（`../` `./` 等）は禁止する。
必ずエイリアスを設定して使用すること。
禁止は ESLint の `no-restricted-imports` ルール（`eslint.shared.mjs` 内で設定）によって機械的に担保する。

```ts
// NG
import { something } from '../../utils/something'
import { foo } from './foo'

// OK
import { something } from '@/utils/something'   // package 内
import { SomeType } from '@hinagata-next/core'   // core package 参照
```

#### tsconfig の設定（`packages/web` の場合）

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

#### core package の参照

`packages/core` は pnpm workspace の package として定義し、package name でそのまま import する。
tsconfig の `paths` に追記する必要はなく、pnpm workspace のシンボリックリンクで解決される。

```ts
// packages/web/src/somewhere.ts
import type { SomeSchema } from '@myproject/core'
```

#### package name の命名規則

`@{プロジェクト名}/{package名}` の形式とする。
リポジトリ名には会社ルールのプレフィックス等が含まれる場合があるため、リポジトリ名ではなくプロジェクト名をベースにする。

```
@myproject/core
@myproject/web     （必要に応じて）
@myproject/functions
```

#### Next.js の場合の注意

Next.js は `tsconfig.json` の `paths` を自動的に読み取るため、`next.config` 側での alias 設定は原則不要。

---

## Web フロント

### Next.js を使用（Pages Router / 静的生成ベース）

Web フロントが必要な場合は **Next.js** を採用する。
雛形リポジトリ: [fnobi/hinagata-next](https://github.com/fnobi/hinagata-next)

#### 構成方針

- **Pages Router を原則使用**（App Router は例外的な採用に留める）
- **SSR は使用しない**。`output: 'export'` による静的ファイル書き出しを基本とする
  - `pnpm build` で `dist/` 以下に静的ファイルが生成される
  - `NEXT_PUBLIC_BASE_PATH` 指定でサブディレクトリ配置に対応
- サーバー処理はフロントに混在させず、**別 package（Firebase Functions 等）として明確に分離する**

#### デフォルトで採用するライブラリ

| 用途 | ライブラリ |
|------|-----------|
| 状態管理 | Zustand |
| CSS-in-JS | emotion |
| テスト | Jest + Testing Library |

#### App Router を検討してよいケース

以下のような明確な要件がある場合に限り、App Router の採用を検討する。
ただし ADR に理由を記録すること。

- Server Components による細粒度なデータ取得が有効な場面
- Streaming / Suspense を活用したい場面

---

## サーバー処理・認証・データベース

### Firebase をデフォルト採用

サーバー処理・認証・データベースのいずれかの要件が発生した場合、**Firebase を第一候補**とする。

| 要件 | 採用技術 |
|------|----------|
| サーバー処理 / API | Firebase Functions |
| 認証 | Firebase Authentication |
| データベース | Firestore |

#### 採用の考え方

- これら3つは同一プラットフォームで完結するため、個別に別インフラを採用するメリットは少ない
- 要件が発生した都度、Firebase で倒せないか検討する
- **要件がないものは最初から入れない**（例：DB 要件がないのに Firestore を入れない）
- Firebase で明らかに解決できない要件（大規模バッチ・別言語ランタイムが必要 等）が生じた場合に、他の GCP サービスを検討する

---

## インフラ定義

### Firebase 設定で表現できることは Firebase 管理

`firebase.json` や Firebase CLI の設定・デプロイ機能で完結することは、そちらで管理する。

### Terraform は Firebase では表現できない GCP リソースに限定

以下のようなケースで Terraform を使用する：

- 自動デプロイ（CI/CD）の設定に伴うサービスアカウントの権限設定
- Firebase コンソール・CLI では操作できない GCP レベルのリソース管理
- 複数プロジェクト横断のリソース定義

Terraform を導入する場合は `terraform/` ディレクトリをリポジトリルートに置く。

---

## 判断フロー早見表

```
新規プロジェクト開始
│
├─ JS/TS 以外の言語が必要か？
│   ├─ Yes → 理由を明記した上で例外として採用
│   └─ No  → TypeScript で進める
│
├─ 複数プラットフォームが混在するか？
│   ├─ Yes → pnpm workspace モノレポ構成 + core package
│   └─ No  → 単一 package 構成
│
├─ Web フロントが必要か？
│   ├─ Yes → Next.js (Pages Router / 静的生成ベース)
│   └─ No  → スキップ
│
├─ サーバー処理 / 認証 / DB が必要か？
│   ├─ Yes → Firebase (Functions / Auth / Firestore) を第一候補
│   └─ No  → スキップ
│
└─ Firebase では表現できない GCP リソースが必要か？
    ├─ Yes → Terraform で定義
    └─ No  → スキップ
```

---

## 例外・逸脱の記録

このガイドラインから逸脱する場合は、プロジェクトの `docs/ADR.md`（Architecture Decision Record）に理由を記録する。

```markdown
## [日付] [決定内容の概要]

### 背景
（なぜ標準構成では対応できなかったか）

### 決定
（何をどう変えたか）

### トレードオフ
（この選択のデメリット・将来のリスク）
```
