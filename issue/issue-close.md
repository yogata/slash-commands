---
description: PRをマージし、対応記録を追記し、Issueをクローズしてブランチを削除する
---

# 完了処理

PRをマージし、GitHub Issueに記録として追記し、クローズ後にworktreeとブランチを削除します。
複数Issueを並列実行した場合は、各Issueごとに処理します。

**SDD連携**: パターンBの場合、docs/のコミット（コードと同時）を行います。

## 引数

- `<Issue番号>` — 対象のIssue番号
  - 単一: `101`
  - 複数: `101,102,103`（カンマ区切り）
  - 省略時は直前のissue-workで使用したIssueを使用

## パターン判定

Issueのラベルから規模パターンを判定します：

| ラベル                   | パターン | docs/コミット |
| ------------------------ | -------- | ------------- |
| `bug`, `critical`        | A（小）  | なし          |
| `feature`, `enhancement` | B（中）  | あり          |

---

## 手順

### 1. 前提確認

1. **PRの存在確認**

   ```bash
   gh pr list --head $(git branch --show-current) --state open
   ```

   - PRなし → エラー終了。「先に /issue-work を実行してください」

2. **PRの状態確認**

   ```bash
   PR_NUMBER=$(gh pr list --head $(git branch --show-current) --json number --jq '.[0].number')
   gh pr view $PR_NUMBER --json state
   ```

   - DRAFT → `gh pr ready $PR_NUMBER` でreadyに変換
   - CLOSED → エラー終了。「PRは既にクローズされています」
   - MERGED → 「PRは既にマージ済み」としてブランチ削除のみ実行

### 2. docs/コミット（パターンBのみ）

```bash
git add docs/requirements.md docs/specifications.md docs/implementation-guide.md docs/adr/
git commit -m "docs(docs): update docs for #$ISSUE_NUMBER"
git push origin HEAD
```

### 3. PRマージ

```bash
gh pr merge $PR_NUMBER --merge --delete-branch
```

- **コンフリクト時**: エラー終了。「手動でコンフリクト解決が必要です」

### 4. 記録の作成・追記

**記録テンプレート（機能追加 / パターンB）**:

```markdown
## 実装記録

### 実装日時

YYYY-MM-DD

### 実装概要

- [実装した機能の概要]

### コミット

- <コミットハッシュ>: <コミットメッセージ>

### テスト結果

- [実施したテストとその結果]

### 関連docs

- Requirements: `docs/requirements.md#xxx`
- Specifications: `docs/specifications.md#xxx`
- Implementation Guide: `docs/implementation-guide.md#xxx`
- ADR: `docs/adr/NNN-xxx.md`
```

**記録テンプレート（バグ修正 / パターンA）**:

```markdown
## 対応記録

### 対応日時

YYYY-MM-DD

### 対応内容

- [変更したファイルと変更内容の概要]

### コミット

- <コミットハッシュ>: <コミットメッセージ>

### テスト結果

- [実施したテストとその結果]

### 残課題

- [残っている課題があれば記載。なければ「なし」]
```

**Issueへ追記**:

```bash
gh issue comment $ISSUE_NUMBER --body "<記録>"
```

### 5. Issueクローズ

```bash
gh issue close $ISSUE_NUMBER --reason completed
```

### 6. クリーンアップ

1. **Devサーバー停止**（起動している場合）
   - `Ctrl+C` または `pkill -f "npm run dev"`

2. **メインディレクトリに戻る**

   ```bash
   cd <project_root>
   ```

3. **mainブランチを最新化**

   ```bash
   git checkout main
   git pull
   ```

4. **worktree削除**

   ```bash
   # 機能追加
   git worktree remove .worktrees/$ISSUE_NUMBER-feature
   # バグ修正
   git worktree remove .worktrees/$ISSUE_NUMBER-fix
   ```

5. **作業ブランチ削除**

   ```bash
   # 機能追加
   git branch -d feature/issue-$ISSUE_NUMBER
   # バグ修正
   git branch -d fix/issue-$ISSUE_NUMBER
   ```

6. **リモート追跡ブランチ削除**
   ```bash
   git fetch --prune
   ```

### 7. Planファイルのアーカイブ

本タスクに関連する plan、notepadsを含むファイル一式を `.sisyphus/archives` にアーカイブする

### 8. 完了報告

- マージしたPR番号
- クローズしたIssue番号
- 削除したworktree名・ブランチ名
- docs/コミット内容（パターンBの場合）
- 残課題があれば報告

---

## 複数Issueの場合

各Issueに対して手順1〜5を順次（または並列）実行後、まとめてクリーンアップを行います。

### クリーンアップ（一括）

```bash
# メインディレクトリに戻る
cd <project_root>

# mainブランチを最新化
git checkout main
git pull

# 各worktree削除
git worktree remove .worktrees/101-feature-name
git worktree remove .worktrees/102-bugfix-name

# 各作業ブランチ削除
git branch -d feature/issue-101
git branch -d fix/issue-102

# リモート追跡ブランチ削除
git fetch --prune
```

### 完了報告

- 全PR番号（マージ済み）
- 全Issue番号（クローズ済み）
- 削除したブランチ名・worktree名（一覧）
- docs/コミット内容（パターンBの場合）

---

## エラーハンドリング

| エラー           | 対処                                                |
| ---------------- | --------------------------------------------------- |
| PRが存在しない   | エラー終了。「先に /issue-work を実行してください」 |
| PRがCLOSED       | エラー終了。「PRは既にクローズされています」        |
| PRがDRAFT        | `gh pr ready` でreadyに変換してからマージ           |
| PRがMERGED済み   | ブランチ削除のみ実行                                |
| コンフリクト     | エラー終了。「手動でコンフリクト解決が必要です」    |
| worktree削除失敗 | 強制削除: `git worktree remove --force`             |

## フロー完了

処理が正常に完了したら、最後に以下を出力してください:

> 🎉 フローが完了しました。
