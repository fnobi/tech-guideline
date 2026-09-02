# skill: 新規プロジェクトセットアップ

このリポジトリの `projects/` 以下に新規プロジェクトをセットアップするためのスキル。
技術選定・設計方針は必ず `docs/GUIDELINES.md` を参照すること。

---

## 手順

### 1. GUIDELINES.md を読む

作業を始める前に `docs/GUIDELINES.md` を読み込み、技術選定の方針を把握する。

### 2. プロジェクト情報をヒアリングする

以下の項目を順番にユーザーに確認する。
**すべて確認してから作業に入ること。途中で追加質問しない。**

---

#### 確認項目

**① プロジェクト名**

`projects/` 以下のディレクトリ名と、package name のスコープ（`@{プロジェクト名}/...`）に使う。

```
例: myapp → projects/myapp/  /  @myapp/core
```

---

**② 必要なプラットフォーム**（複数選択可）

- Web フロント（Next.js）
- サーバー処理（Firebase Functions）
- 認証（Firebase Authentication）
- データベース（Firestore）
- その他（具体的に）

→ 複数プラットフォームが混在する場合は **モノレポ構成**になる（GUIDELINES.md 参照）。

---

**③ Web フロントが必要な場合：ページ構成のざっくりした内容**

静的生成で問題ないか確認するための情報として聞く。
原則として Pages Router + 静的生成で進めるが、以下に該当する場合は App Router を検討する旨を伝える：

- ページごとに異なるデータを毎回サーバーから取得したい
- ストリーミングや細粒度のローディングUIが必要

→ 該当する場合はユーザーに確認の上、ADR に記録する。

---

**④ CI/CD・インフラの要件**

- 自動デプロイは必要か（GitHub Actions 等）
- デプロイ先（Firebase Hosting / その他）
- GCP リソースを Terraform で管理する必要があるか（GUIDELINES.md 参照）

---

**⑤ その他の特記事項**

GUIDELINES.md の標準構成から外れる可能性がある要件があれば確認する。
（例：別言語の利用、外部サービス連携、特殊なインフラ要件など）

---

### 3. 構成を提示してユーザーに確認する

ヒアリング内容をもとに、以下を提示する。

- 作成するディレクトリ構成
- 採用する技術スタック
- GUIDELINES.md から逸脱する点がある場合はその理由と ADR への記録方針

ユーザーの承認を得てから実装に進む。

---

### 4. セットアップを実行する

`projects/{プロジェクト名}/` 以下にプロジェクトを構築する。

実装の詳細は `docs/GUIDELINES.md` に従う。主なチェックポイント：

- [ ] `pnpm-workspace.yaml` の配置（モノレポの場合）
- [ ] `packages/core/` の作成（モノレポの場合）
- [ ] package name が `@{プロジェクト名}/...` 形式になっている
- [ ] ルートに `eslint.shared.mjs` を配置し、各 package で継承している
- [ ] 各 package の `tsconfig.json` にエイリアス設定がある
- [ ] 相対パス import が存在しない（ESLint で担保されている）
- [ ] `templates/hinagata-next/` を参考にリポジトリ全体（root構成・`packages/core`・`packages/functions`・Firebase 設定・Terraform・CI 等）を構成している
- [ ] Web フロントが必要な場合は `templates/hinagata-next/packages/web` を参考に Next.js を構成している（不要な場合は `packages/web` は作成しない）
- [ ] `templates/hinagata-next` に含まれるファイルは安易に削除していない（`mock` と名のつくファイル群も含め、初期構築時の利便性のために用意されているものが多いため、開発序盤はそのまま残す）
- [ ] `.prettierrc` がルートに存在する
- [ ] GUIDELINES.md から逸脱した点は `docs/ADR.md` に記録している

---

### 5. 完了報告

セットアップ完了後、以下を報告する。

- 作成したディレクトリ構成
- 各 package の起動・ビルドコマンド
- 別リポジトリへの切り出し手順

```bash
# 切り出し手順（参考）
cd projects/{プロジェクト名}
git init
git add .
git commit -m "initial commit"
git remote add origin {新しいリポジトリのURL}
git push -u origin main
```
