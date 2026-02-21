---
description: PRをマージし、対応記録を追記し、Issueをクローズしてブランチを削除する
---

# 完了処理

PRをマージし、GitHub Issueに記録として追記し、クローズ後にworktreeとブランチを削除します。
複数Issueを並列実行した場合は、各Issueごとに処理します。

**SDD連携**: パターンBの場合、specs/のコミット（コードと同時）を行います。

## 引数

- `<Issue番号>` — 対象のIssue番号
  - 単一: `101`
  - 複数: `101,102,103`（カンマ区切り）
  - 省略時は直前のissue-workで使用したIssueを使用

## パターン判定

Issueのラベルから規模パターンを判定します：

| ラベル | パターン | specs/コミット |
|-------|---------|--------------|
| `bug`, `critical` | A（小） | なし |
| `feature`, `enhancement` | B（中） | あり |

## 手順

### A. 単一Issueの場合

1. **PRの存在確認**
    - 現在のブランチに対応するPRを確認する:
    ```bash
    gh pr list --head $(git branch --show-current) --state open
    ```
    - PRが存在しない場合: エラー終了。「先に /issue-work を実行してください」
    - PRが存在することを確認してから次のステップへ進む

2. **クローズ前の確認**
    - 記録がまだ追記されていないことを確認する:
    ```bash
    gh issue view $ISSUE_NUMBER --comments
    ```

3. **specs/コミット（パターンBのみ）**
    - **パターンBの場合のみ**、specs/の変更をコミットする:
    ```bash
    git add specs/requirements.md specs/specifications.md specs/implementation-guide.md specs/adr/
    git commit -m "docs(specs): update specs for #$ISSUE_NUMBER"
    git push origin HEAD
    ```
    - PRは自動的に更新される

4. **PRマージ**
    - PR番号を特定する:
    ```bash
    PR_NUMBER=$(gh pr list --head $(git branch --show-current) --json number --jq '.[0].number')
    ```
    - PRの状態を確認する:
    ```bash
    gh pr view $PR_NUMBER --json state
    ```
    - **エラーハンドリング**:
      - PRがDRAFTの場合: `gh pr ready $PR_NUMBER` でreadyに変換
      - PRがCLOSEDの場合: エラー終了。「PRは既にクローズされています」
      - PRがMERGEDの場合: 「PRは既にマージ済み」としてブランチ削除のみ実行
    - マージを実行する（リモートブランチも削除）:
    ```bash
    gh pr merge $PR_NUMBER --merge --delete-branch
    ```
    - **コンフリクト時**: エラー終了。「手動でコンフリクト解決が必要です」

5. **記録の作成**
    - コミットメッセージは `Conventional Commits` に従うこと
    - 文脈（機能追加 or バグ修正）に応じて、以下のいずれかの構造で記録を作成する:

    **A. 機能追加の場合 (Feature / パターンB)**
    ```markdown
    ## 実装記録

    ### 実装日時
    YYYY-MM-DD

    ### 実装概要
    - [実装した機能の概要]
    - [主要な変更点]

    ### コミット
    - <コミットハッシュ>: <コミットメッセージ>

    ### テスト結果
    - [実施したテストとその結果]

    ### 関連Specs
    - Requirements: `specs/requirements.md#xxx`
    - Specifications (HLD): `specs/specifications.md#xxx`
    - Implementation Guide (LLD): `specs/implementation-guide.md#xxx`
    - ADR: `specs/adr/NNN-xxx.md`

    ### 補足事項
    - [使用上の注意や設定変更などがあれば記載]
    ```

    **B. バグ修正の場合 (Bug / パターンA)**
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

6. **Issueへのコメント追記**
    - コミットメッセージは `Conventional Commits` に従うこと
    - 以下のコマンドで追記する:
    ```bash
    gh issue comment $ISSUE_NUMBER --body "<記録>"
    ```

7. **追記結果の確認**
    - 記録が追記されたことを確認する:
    ```bash
    gh issue view $ISSUE_NUMBER --comments
    ```

8. **Issueのクローズ**
    - 記録が追記されていることを確認した上で、以下のコマンドでクローズする:
    ```bash
    gh issue close $ISSUE_NUMBER --reason completed
    ```

9. **クローズ結果の確認**
    - ステータスが `CLOSED` であることを確認する:
    ```bash
    gh issue view $ISSUE_NUMBER
    ```

10. **Devサーバーの停止**
    - worktreeディレクトリでdevサーバーが起動している場合は停止する
    - 停止が必要な例:
      - `Ctrl+C` でサーバーを停止
      - または `pkill -f "npm run dev"` / `pkill -f "yarn dev"` など
    - サーバーが起動していない場合はスキップ

11. **Worktreeとブランチのクリーンアップ**
    - メインディレクトリに戻る:
    ```bash
    cd <project_root>
    ```
    - main ブランチを最新にする:
    ```bash
    git checkout main
    git pull
    ```
    - worktreeを削除する:
      - 機能追加: `git worktree remove .worktrees/$ISSUE_NUMBER-feature`
      - バグ修正: `git worktree remove .worktrees/$ISSUE_NUMBER-fix`
    - 作業ブランチを削除する:
      - 機能追加: `git branch -d feature/issue-$ISSUE_NUMBER`
      - バグ修正: `git branch -d fix/issue-$ISSUE_NUMBER`
    - リモート追跡ブランチを削除する:
    ```bash
    git fetch --prune
    ```

12. **Planファイルのアーカイブ**
    - 本タスクに関連する plan、notepadsを含むファイル一式をアーカイブする

13. **完了報告**
    - 完了した内容をユーザーに報告する:
      - マージしたPR番号
      - クローズしたIssue番号
      - 削除したworktree名・ブランチ名
      - specs/コミット内容（パターンBの場合）
      - 残課題があれば報告

---

### C. 複数Issueの場合

1. **各Issueの処理**
    - 各Issueに対して順次（または並列）で以下を実行:
      - PRの存在確認
      - specs/コミット（パターンB）
      - PRマージ
      - 記録作成・追記
      - Issueクローズ

2. **Worktree クリーンアップ**
    - 全Issueの処理完了後、worktreeを削除:
    ```bash
    git worktree remove .worktrees/101-feature-name
    git worktree remove .worktrees/102-bugfix-name
    ```

3. **完了報告**
    - 全PR番号（マージ済み）
    - 全Issue番号（クローズ済み）
    - 削除したブランチ名・worktree名
    - specs/コミット内容（パターンBの場合）

## エラーハンドリング

| エラー | 対処 |
|--------|------|
| PRが存在しない | エラー終了。「先に /issue-work を実行してください」 |
| PRがCLOSED | エラー終了。「PRは既にクローズされています」 |
| PRがDRAFT | `gh pr ready` でreadyに変換してからマージ |
| PRがMERGED済み | ブランチ削除のみ実行 |
| コンフリクト | エラー終了。「手動でコンフリクト解決が必要です」 |
| worktree削除失敗 | 強制削除: `git worktree remove --force` |

## フロー完了

処理が正常に完了したら、最後に以下を出力してください:

> 🎉 フローが完了しました。
