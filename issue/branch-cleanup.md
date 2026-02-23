---
description: mainにマージ済みのリモート・ローカルブランチ・worktreeを一括削除
---

# ブランチクリーンアップ

mainブランチにマージ済みのリモートブランチ、ローカルブランチ、worktreeを一括削除します。

## 手順

### 1. 現在の状態確認

現在のブランチとworktreeの状態を確認する:

```bash
# 現在のブランチ
git branch --show-current

# worktree一覧
git worktree list

# マージ済みローカルブランチ
git branch --merged main

# マージ済みリモートブランチ
git branch -r --merged main
```

### 2. 削除対象の特定

mainにマージ済みで、かつ以下のパターンに一致するブランチを特定:

- `feature/*`
- `fix/*`
- `bugfix/*`
- `hotfix/*`
- `chore/*`
- `refactor/*`

**除外条件:**
- `main`, `master`, `develop`, `staging` は削除しない
- 未マージのブランチは削除しない
- ユーザーが明示的に残したいと指定したブランチは削除しない

### 3. 削除対象の提示と確認

削除対象ブランチを一覧表示し、ユーザーに確認を求める:

```
削除対象:
=== ローカルブランチ ===
- feature/issue-123
- fix/issue-456

=== リモートブランチ ===
- origin/feature/issue-123
- origin/fix/issue-456

=== Worktree ===
- .worktrees/123-feature

続行しますか？ (y/n)
```

### 4. Worktree削除

確認後、worktreeから削除:

```bash
# 削除対象worktreeの削除
git worktree remove <worktree-path>

# 強制削除が必要な場合
git worktree remove --force <worktree-path>
```

### 5. mainブランチに切り替え

```bash
git checkout main
git pull origin main
```

### 6. ローカルブランチ削除

```bash
git branch -d <branch-name>

# 削除に失敗する場合（未マージ警告）
git branch -D <branch-name>  # 注意: 強制削除
```

### 7. リモートブランチ削除

```bash
git push origin --delete <branch-name>
```

### 8. Prune実行

リモート追跡ブランチを整理:

```bash
git fetch --prune
```

### 9. 結果報告

削除完了後、以下を報告:

- 削除したローカルブランチ数とブランチ名
- 削除したリモートブランチ数とブランチ名
- 削除したworktree数とパス
- 削除できなかったブランチ（あれば理由付き）

## エラーハンドリング

| エラー | 対処 |
|--------|------|
| worktree内に未コミット変更 | 警告表示、スキップするか確認 |
| ローカルブランチ削除失敗（未マージ） | スキップして次へ |
| リモートブランチ削除失敗 | エラー表示、ローカルのみ削除継続 |
| mainにいない場合 | 自動的にmainに切り替え |

## オプション

引数で以下を指定可能:

- `--dry-run` — 削除対象を表示のみ（実際には削除しない）
- `--local-only` — ローカルブランチのみ削除
- `--remote-only` — リモートブランチのみ削除
- `--include-pattern <pattern>` — 特定パターンのみ対象（例: `--include-pattern "feature/*"`）

## フロー完了

処理が正常に完了したら、最後に以下を出力してください:

> ✅ ブランチクリーンアップが完了しました。
