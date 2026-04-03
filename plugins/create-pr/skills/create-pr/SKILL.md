---
name: create-pr
description: "git diffからPR descriptionを自動生成してgh pr create --webでブラウザを開くスキル。PR作成、プルリクエスト作成、PR出す、プルリク作る、gh pr createなど、PRを作成したいときに使う。"
allowed-tools: Bash(git *), Bash(gh *)
disable-model-invocation: true
argument-hint: "[base-branch]"
---

# PR Description 自動生成 & 作成

ベースブランチとの差分を分析し、日本語のPR descriptionを生成して `gh pr create --web` でブラウザを開く。

## Step 1: ベースブランチの決定

`$ARGUMENTS` が指定されていればそれを使用。未指定の場合は `release-candidate` をデフォルトとする。

```bash
BASE_BRANCH="${ARGUMENTS:-release-candidate}"
```

## Step 2: 差分情報の収集

以下を順番に実行して変更内容を把握する。

```bash
# 変更量の概要
git diff --stat origin/$BASE_BRANCH...HEAD

# コミット履歴
git log --oneline origin/$BASE_BRANCH...HEAD

# 差分の詳細（大きすぎる場合は --name-only で一覧のみ）
git diff origin/$BASE_BRANCH...HEAD
```

差分が大きい（300行超）場合は `git diff --name-only origin/$BASE_BRANCH...HEAD` でファイル一覧のみ取得し、重要ファイルを選択的に `Read` で読み込む。

## Step 3: PR description の生成

収集した差分・コミット履歴・変更ファイルの内容を分析し、以下のフォーマットで日本語のdescriptionを作成する。

```markdown
## 背景

（なぜこの変更が必要か。解決する課題・要件・issueの背景を記述。）

## 概要

- （主な変更点を箇条書きで3〜5点）

## 変更内容

| ファイル | 変更内容 |
|---------|---------|
| `path/to/file` | 概要 |

## 確認方法

- （動作確認・テスト手順を箇条書きで）
```

PRタイトルも差分から推測して生成する（例: `feat: 買取問題一覧にフィルター機能を追加`）。

## Step 4: PR作成

生成したタイトルとdescriptionを使って `gh pr create --web` を実行する。

```bash
gh pr create \
  --base "$BASE_BRANCH" \
  --title "<生成したタイトル>" \
  --body "<生成したdescription>" \
  --web
```

`--web` によりブラウザが開き、descriptionが自動入力された状態でPR作成画面が表示される。内容を確認・編集してから送信できる。

## 注意事項

- `git push` でリモートにブランチが存在しない場合は先にpushが必要。その旨を伝えてpushを促す。
- 既にPRが存在する場合は `gh pr create` がエラーになるため、その旨を伝える。
- `origin/$BASE_BRANCH` が存在しない場合はベースブランチ名を確認する。
