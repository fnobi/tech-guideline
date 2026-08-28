# CLAUDE.md

このリポジトリは技術ガイドラインと、新規プロジェクトのセットアップ作業場を兼ねたリポジトリ。

---

## リポジトリ構成

```
/
├── docs/
│   └── GUIDELINES.md       # 技術選定・設計方針（必読）
├── skills/
│   └── setup-project.md    # 新規プロジェクトセットアップ手順
├── templates/
│   └── hinagata-next/      # Next.js 雛形（Pages Router、submodule）
└── projects/               # .gitignore済み・作業ディレクトリ
    └── {プロジェクト名}/   # セットアップ後、別リポジトリに切り出す
```

---

## 作業の種類と参照先

### 新規プロジェクトをセットアップしたい

`skills/setup-project.md` を読んで、手順に従って進める。

### 技術選定・設計方針を確認したい

`docs/GUIDELINES.md` を読む。

### ガイドラインやスキル自体を更新したい

ユーザーの指示に従って `docs/` または `skills/` 以下のファイルを編集する。
`projects/` 以下は触らない。

---

## 注意事項

- `projects/` 以下は `.gitignore` 対象。このリポジトリにはコミットされない
- 技術的な判断に迷ったときは必ず `docs/GUIDELINES.md` を参照する
- ガイドラインから逸脱する場合は、該当プロジェクトの `docs/ADR.md` に理由を記録する
