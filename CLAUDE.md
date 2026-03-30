# claude-plugins

gotchane の Claude Code Marketplace 用プラグイン（スキル）リポジトリ。

## リポジトリ構造

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace 定義（オーナー情報・プラグイン一覧）
└── plugins/
    └── <plugin-name>/
        ├── .claude-plugin/
        │   └── plugin.json     # プラグインメタデータ
        └── skills/
            └── <skill-name>/
                └── SKILL.md    # スキル本体（frontmatter + 実行指示）
```

## 現在のスキル一覧

| スキル名 | カテゴリ | 説明 |
|---------|---------|------|
| `pr-review-assist` | code-quality | PRの差分から目的・前提知識・シーケンス図・レビュー指摘を一括生成 |

## 新しいスキルを追加するときの手順

### 1. プラグインディレクトリを作成

```
plugins/<plugin-name>/
├── .claude-plugin/plugin.json
└── skills/<skill-name>/SKILL.md
```

### 2. plugin.json を記述

```json
{
  "name": "<plugin-name>",
  "description": "<スキルの一言説明>",
  "version": "1.0.0"
}
```

### 3. SKILL.md を記述

```markdown
---
name: <skill-name>
description: >
  （トリガー条件を含む詳細な説明。
  「〇〇して」「〇〇を見て」など、ユーザーが発しそうな言葉を列挙する。
  Claude がトリガーを判断する材料になるため、具体的に書く。）
---

# スキルタイトル

（スキルの目的・ゴールを1〜2段落で）

## 実行フロー

### Step 1: ...
```

### 4. marketplace.json にエントリを追加

`.claude-plugin/marketplace.json` の `plugins` 配列に追記する。

```json
{
  "name": "<plugin-name>",
  "source": "./plugins/<plugin-name>",
  "description": "<スキルの一言説明>",
  "version": "1.0.0",
  "category": "<category>"
}
```

利用可能なカテゴリ例: `code-quality`, `productivity`, `documentation`, `testing`, `devops`

## スキル設計のベストプラクティス

### SKILL.md の description（frontmatter）

- **トリガーフレーズを網羅する**: ユーザーが使いそうな日本語表現を複数列挙する。Claude はここを読んでスキルを呼ぶかどうか判断する。
- **目的を一文で言える**: 「〇〇するスキル」という形で何をもたらすかを明示する。
- **前提条件を書く**: `gh` CLI が必要、特定の言語スタックが対象、など。

### 実行フローの設計

- **ステップは 3〜6 個**: 多すぎると Claude が迷い、少なすぎると粗い出力になる。
- **出力フォーマットをコードブロックで示す**: Claude への指示ではなく、実際の出力例を貼る。
- **追加コマンドを用意する**: ユーザーが「もっと詳しく」と言ったときの対応を書いておく。
- **注意事項で境界を定める**: スキルが対応しないケースや限界を明示し、無用な幻覚を防ぐ。

### バージョン管理

- 破壊的変更（出力フォーマット変更・ステップ削除）は `major` バージョンを上げる。
- 機能追加・説明の改善は `minor` を上げる。
- 誤字・表現の微修正は `patch` を上げる。
- `plugin.json` と `marketplace.json` の両方のバージョンを揃えて更新する。
