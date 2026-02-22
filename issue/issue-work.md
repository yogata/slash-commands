---
description: 計画立案からコミットまでを一気通貫で実行する（@plan + /start-work + commit統合）
---

# 実装パイプライン

選択したIssueに対して、計画立案から実装・コミットまでを一気通貫で実行します。
常にgit worktreeを使用して、メインの作業ディレクトリを汚さずに開発を行います。
複数Issueを指定した場合は、並列実行します。

**SDD連携**: パターンBの場合、specifications.mdの更新とADR statusの更新を行います。

## 引数

- `<サブコマンド>` — 実行するステップを指定（省略時は全ステップ実行）
  - `test` — 動作確認・テストのみ実行
  - `commit` — コミットのみ実行
  - `pr` — PR作成のみ実行
- `<Issue番号>` — 対象のIssue番号
  - 単一: `101`
  - 複数: `101,102,103`（カンマ区切りで並列実行）
  - 省略時は直前のissue-createまたはissue-workで操作したIssueを使用

### 使用例

```bash
# 全ステップ実行（計画〜PR作成）- サブコマンド省略
/issue-work 101

# テスト以降を実施
/issue-work test 101
/issue-work test  # 直前のIssueを使用

# コミット以降を実行
/issue-work commit 101

# PR作成以降を実行
/issue-work pr 101

# 複数Issueを並列実行
/issue-work test 101,102,103
```

## パターン判定

Issueのラベルから規模パターンを判定します：

| ラベル                   | パターン | specs/更新 |
| ------------------------ | -------- | ---------- |
| `bug`, `critical`        | A（小）  | なし       |
| `feature`, `enhancement` | B（中）  | あり       |

## 手順

### A. 単一Issueの場合

1. **Issueと文脈の確認**
   - 対象のIssueを確認する:

   ```bash
   gh issue view $ISSUE_NUMBER
   ```

   - ラベルから文脈（機能追加 or バグ修正）と規模（パターンA/B）を判定する:
     - `enhancement`, `feature` → 機能追加（パターンB）
     - `bug`, `critical` → バグ修正（パターンA）

2. **Worktree作成**
   - worktree用ディレクトリを作成:

   ```bash
   mkdir -p .worktrees
   ```

   - 文脈に応じて、以下の命名規則でworktreeを作成する:
     - **機能追加**: `git worktree add .worktrees/$ISSUE_NUMBER-feature -b feature/issue-$ISSUE_NUMBER`
     - **バグ修正**: `git worktree add .worktrees/$ISSUE_NUMBER-fix -b fix/issue-$ISSUE_NUMBER`
   - 作成したworktreeに移動する:

   ```bash
   cd .worktrees/$ISSUE_NUMBER-<type>
   ```

3. **@planによる計画立案**
   - 実装計画は `specs/` とは分離し、 Plan agent の既定場所に保存する
   - Plan agentを使用して、Issueごとに実装計画を立案する:

   ```
   @plan Issue #$ISSUE_NUMBERの実装計画を立ててください。実装計画のファイル名はissue番号と関連付けてください。
   ```

   - 文脈に応じて計画を作成する:
     - **機能追加**: 設計方針、変更対象、実装手順、テスト方針、見積もり
     - **バグ修正**: 修正対象、変更内容、テスト方針、リスク、見積もり

4. **/start-workによる実装**
   - 立案した計画に基づいて実装を開始する:

   ```
   /start-work
   ```

   - 計画に従い、コード作成・修正およびテストを実施する

5. **specifications.md（HLD）更新（パターンBのみ）**
   - **パターンBの場合のみ**、`specs/specifications.md`（HLD）を更新する:
     - 実装で確定した仕様を追加・更新
     - アーキテクチャ、コンポーネント間の関係、データフローを記述

   ```markdown
   ## {機能名}

   ### アーキテクチャ

   - [システム全体の構造]
   - [コンポーネント間の関係]

   ### データフロー

   - [データの流れ]
   ```

6. **implementation-guide.md（LLD）作成（パターンBのみ）**
   - **パターンBの場合のみ**、`specs/implementation-guide.md`（LLD）を作成・更新する:
     - クラス設計、関数シグネチャ、アルゴリズムを記述

   ```markdown
   ## {機能名}

   ### クラス設計

   - [クラス名と責務]
   - [メソッド一覧]

   ### 関数シグネチャ

   - [関数名、引数、戻り値]

   ### アルゴリズム

   - [主要な処理ロジック]
   ```

7. **ADR status更新（パターンBのみ、ADR作成時）**
   - **パターンBの場合のみ**、ADRが作成されている場合にstatusを更新:
     - まず `specs/adr/` ディレクトリ内に該当機能のADR（status: proposed）が存在するか確認
     - ADRが存在する場合: `proposed` → `accepted` に更新
     - ADRが存在しない場合: 更新をスキップ（アーキテクチャ的に重要でない機能のため）

   > ADRなし: アーキテクチャ的に重要でない変更のため、ADR更新をスキップします

   ```markdown
   ## Status

   accepted
   ```

8. **動作確認・テスト**
   - ユニットテストとE2Eテストを行うこと
   - 既存のテストが通ることを確認する
   - 新規機能の場合はテストを追加し、パスすることを確認する
   - バグ修正の場合は再現手順で問題が解消されたことを確認する

9. **コミット**
   - 変更内容をステージングする:

   ```bash
   git add -A
   ```

   - 文脈に応じたコミットメッセージでコミットする:
     - **機能追加**: `git commit -m "feat: <実装内容の要約> (#$ISSUE_NUMBER)"`
     - **バグ修正**: `git commit -m "fix: <修正内容の要約> (#$ISSUE_NUMBER)"`
   - **注意**: specs/のコミットは `/issue-close` で行う（コードと同時）

10. **プッシュ**
    - ブランチをリモートリポジトリにプッシュする:

    ```bash
    git push origin HEAD
    ```

11. **PR作成**
    - 既存のPRを確認する:

    ```bash
    gh pr list --head $(git branch --show-current) --state open
    ```

    - PRが存在する場合は更新する:

    ```bash
    gh pr edit
    ```

    - PRが存在しない場合は作成する:

    ```bash
    gh pr create --base main --title "feat/fix: {要約} (#$ISSUE_NUMBER)" --body "Issueの要約 + 実装内容の概要 + テスト結果"
    ```

12. **レビュー依頼**
    - ユーザーにレビューを促す:
      > PRを作成/更新しました: {PR_URL}
      > ユーザーレビューをお願いします.
      >
      > - OK → `/issue-close`
      > - NG（仕様バグ）→ `/issue-update` → `/issue-work`
      > - NG（実装バグ）→ `/issue-update --comment` → `/issue-work`

13. **Worktreeの状態確認**
    - レビュー待ちの間、worktreeは保持される
    - メインディレクトリに戻る場合:

    ```bash
    cd ../..
    ```

    - 再度worktreeで作業する場合:

    ```bash
    cd .worktrees/$ISSUE_NUMBER-<type>
    ```

### B. 複数Issueの場合

1. **各Issueの確認**
   - 指定されたIssueをすべて確認する:

   ```bash
   gh issue view 101
   gh issue view 102
   gh issue view 103
   ```

   - 各Issueの文脈（機能追加/バグ修正）を判定する

2. **競合検出**
   - 各Issueの影響ファイルを予測する:
     - Issueのラベル・コンポーネント分析
     - 類似Issueの過去実績から影響範囲を推測
   - 同一ファイルを編集するIssueペアを検出
   - 結果に基づいてグループ化:
     - **競合なしグループ**: 全て並列実行可能
     - **競合ありグループ**: 依存順序で直列実行

3. **Worktree作成**
   - worktree用ディレクトリを作成:

   ```bash
   mkdir -p .worktrees
   ```

   - 各Issue用のworktreeを作成:

   ```bash
   git worktree add .worktrees/101-feature-name -b feature/issue-101
   git worktree add .worktrees/102-bugfix-name -b fix/issue-102
   ```

4. **並列実行**
   - **競合なしグループ**: `run_in_background=true` で同時実行
   - **競合ありグループ**: 依存順序で順次実行
   - 各worktreeで以下を実行:
     - @plan による計画立案
     - /start-work による実装
     - テスト・コミット

5. **完了待ち合わせ**
   - 全タスクの完了を確認
   - 失敗したIssueがあれば報告

6. **各PR作成**
   - 各worktreeでPRを作成:

   ```bash
   cd .worktrees/101-feature-name
   gh pr create --base main --title "feat: ... (#101)" --body "..."
   ```

7. **レビュー依頼**
   - 全PRのURLを報告
   - ユーザーにレビューを促す（NG分類の説明を含む）

## Worktree 構造

```
/project (main worktree)
  /.worktrees/
    /101-feature-name (feature/issue-101)
    /102-bugfix-name (fix/issue-102)
```

## 競合検出ロジック

1. **影響ファイル予測**
   - Issueのラベル・コンポーネント分析
   - 過去の類似PRから変更ファイルを推測
   - 手動指定も可能（ユーザーが `--files` オプションで指定）

2. **競合判定**

   ```
   同一ファイルが複数Issueで編集される → 競合あり
   ```

3. **グループ化**
   - 競合なし → 並列グループ
   - 競合あり → 依存グラフ構築 → 直列グループ

## specs/競合対応

複数Issueが `specs/` 以下のファイルを編集する場合、通常のGit競合と同様に対応します：

### 競合発生時の対応

1. **競合の検知**
   - 他のPRがマージされた後、自分のPRで競合が発生

2. **競合解決**

   ```bash
   # 最新のmainを取り込む
   git fetch origin
   git rebase origin/main
   # または
   git merge origin/main

   # 競合箇所を解決
   # ... エディタで編集 ...

   # 解決完了
   git add specs/requirements.md
   git rebase --continue  # または git commit

   # 再プッシュ
   git push --force-with-lease
   ```

3. **PR再レビュー**
   - PRは自動的に更新される
   - 再レビュー後にマージ可能になる

### 競合回避のコツ

- 大きな変更は早めにマージする
- specs/の編集は必要最小限にする

## エラーハンドリング

| エラー               | 対処                             |
| -------------------- | -------------------------------- |
| worktree作成失敗     | クリーンアップして終了           |
| 並列タスクの一部失敗 | 成功分は維持、失敗分を報告       |
| 同一ブランチ名の競合 | 既存ブランチを削除またはスキップ |
| PR作成失敗           | コミットは保持、手動PR作成を案内 |

## 中断時のクリーンアップ

処理が途中で中断された場合、またはユーザーが明示的にキャンセルした場合は、以下の手順でクリーンアップする：

### 単一Issueの場合

```bash
# 1. メインディレクトリに戻る
cd <project_root>

# 2. worktreeを強制削除
git worktree remove .worktrees/$ISSUE_NUMBER-feature --force
# または
git worktree remove .worktrees/$ISSUE_NUMBER-fix --force

# 3. ローカルブランチを削除
git branch -D feature/issue-$ISSUE_NUMBER
# または
git branch -D fix/issue-$ISSUE_NUMBER

# 4. リモートブランチが存在する場合は削除
git push origin --delete feature/issue-$ISSUE_NUMBER 2>/dev/null || true
# または
git push origin --delete fix/issue-$ISSUE_NUMBER 2>/dev/null || true
```

### 複数Issueの場合

```bash
# 1. メインディレクトリに戻る
cd <project_root>

# 2. 各worktreeとブランチを削除
for issue in 101 102 103; do
  git worktree remove .worktrees/${issue}-feature --force 2>/dev/null || true
  git worktree remove .worktrees/${issue}-fix --force 2>/dev/null || true
  git branch -D feature/issue-${issue} 2>/dev/null || true
  git branch -D fix/issue-${issue} 2>/dev/null || true
  git push origin --delete feature/issue-${issue} 2>/dev/null || true
  git push origin --delete fix/issue-${issue} 2>/dev/null || true
done

# 3. リモート追跡情報を更新
git fetch --prune
```

## レビューNG時の対応

| タイプ       | 原因                        | 対処                                                            |
| ------------ | --------------------------- | --------------------------------------------------------------- |
| **仕様バグ** | Issueの要件が不完全・間違い | `/issue-update`（本文更新）→ `/issue-work` 再実行               |
| **実装バグ** | AIの実装が要件通りでない    | `/issue-update --comment`（コメント追加）→ `/issue-work` 再実行 |

**重要**: どちらの場合も、既存PRは同じブランチへの追加コミットで自動更新される。

## 次のステップ

処理が正常に完了したら、最後に以下を出力してください:

> ✅ 完了しました。次のステップ: `/issue-close` を使用して記録追記とクローズを行いますか？

### 注意事項

- PR作成後は、コミットメッセージとPRタイトルにIssue番号を必ず含めてください
- 作成したPRのURLを実行結果に含めて報告してください
- 複数Issueの場合は、全PRのURLをリスト形式で報告してください
