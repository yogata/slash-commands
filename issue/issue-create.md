---
description: /issue-req の結果をもとにGitHub Issueを作成する
---

# Issue登録

/issue-req で定義した要件または分析結果をもとに、GitHub Issueを作成してください。
規模パターン（A/B）に応じて処理が異なります。

## パターン判定

/issue-req で判定されたパターンに従って処理します：

| パターン    | 処理                                | ラベル                   |
| ----------- | ----------------------------------- | ------------------------ |
| **A（小）** | Issue本文 + コメント（分析結果）    | `bug` 等                 |
| **B（中）** | Issue本文 + コメント + docs/紐付け  | `feature`, `enhancement` |

## テンプレート

Issue本文は `.github/ISSUE_TEMPLATE/` のテンプレートを使用します。
GitHub公式パターンに準拠し、Issue本文は「事実/要望のみ」とし、分析結果はコメントとして追加します。

| パターン    | テンプレートファイル          | CLIコマンド                                      |
| ----------- | ----------------------------- | ------------------------------------------------ |
| **A（小）** | `.github/ISSUE_TEMPLATE/bug_report.md` | `gh issue create --template "Bug Report"` |
| **B（中）** | `.github/ISSUE_TEMPLATE/feature_request.md` | `gh issue create --template "Feature Request"` |

---

## 手順

### パターンA（小規模・バグ修正）

1. **Issue本文の作成** — 一時ファイル経由で複数行本文を渡す（Windows対応）

   ```powershell
   # temp/ディレクトリの確認・作成
   if (-not (Test-Path "temp")) { New-Item -ItemType Directory -Path "temp" -Force }

   # Issue本文を一時ファイルに書き出し
   @"
   ## Description

   [バグの説明]

   ## Steps to Reproduce

   1. [手順1]
   2. [手順2]

   ## Expected Behavior

   [期待動作]
   "@ | Out-File -FilePath "temp/issue-body.md" -Encoding utf8

   # Issue作成
   gh issue create --title "<タイトル>" --body-file "temp/issue-body.md" --label "bug"
   ```

   テンプレート項目（`.github/ISSUE_TEMPLATE/bug_report.md` 参照）:
   - Description（バグの説明）— 必須
   - Steps to Reproduce（再現手順）— 必須
   - Expected Behavior（期待動作）— 必須
   - Actual Behavior（実際の動作）— オプション
   - Screenshots/Videos — オプション
   - Additional Context — オプション

2. **Issueコメント追加** — 分析結果

   ```powershell
   # コメント本文を一時ファイルに書き出し
   @"
   ## 根本原因

   [なぜ起きているか]

   ## 影響範囲

   [どこに影響するか]

   ## 対応方針

   [どう対処すべきか]
   "@ | Out-File -FilePath "temp/comment-body.md" -Encoding utf8

   # コメント追加
   gh issue comment {N} --body-file "temp/comment-body.md"
   ```

   コメント内容:
   - 根本原因（なぜ起きているか）
   - 影響範囲（どこに影響するか）
   - 対応方針（どう対処すべきか）

3. **完了報告**

   > ✅ Issue #{N} を作成しました（パターンA）。
   > Issue本文 + コメント（分析結果）を追加しました。
   > 次のステップ: `/issue-work {N}` を使用して実装を開始しますか？

---

### パターンB（中規模・新機能追加）

1. **Issue本文の作成** — 一時ファイル経由で複数行本文を渡す（Windows対応）

   ```powershell
   # temp/ディレクトリの確認・作成
   if (-not (Test-Path "temp")) { New-Item -ItemType Directory -Path "temp" -Force }

   # Issue本文を一時ファイルに書き出し
   @"
   ## Proposal Summary

   [機能の概要（一言で説明）]

   ## Problem Statement

   [解決したい課題]

   ## Proposed Solution

   [提案する解決策]
   "@ | Out-File -FilePath "temp/issue-body.md" -Encoding utf8

   # Issue作成
   gh issue create --title "<タイトル>" --body-file "temp/issue-body.md" --label "enhancement"
   ```

   テンプレート項目（`.github/ISSUE_TEMPLATE/feature_request.md` 参照）:
   - Proposal Summary（機能の概要）— 必須
   - Problem Statement（解決する課題）— 必須
   - Proposed Solution（提案する解決策）— 必須
   - Use Cases（使用例）— オプション
   - Alternatives（代替案）— オプション
   - Additional Context — オプション

2. **Issueコメント追加** — 技術検討

   ```powershell
   # コメント本文を一時ファイルに書き出し
   @"
   ## 技術的なアプローチ

   [アプローチの説明]

   ## アーキテクチャ決定

   [決定事項]

   ## 実装のスコープ

   [スコープの境界]
   "@ | Out-File -FilePath "temp/comment-body.md" -Encoding utf8

   # コメント追加
   gh issue comment {N} --body-file "temp/comment-body.md"
   ```

   コメント内容:
   - 技術的なアプローチ
   - アーキテクチャ決定
   - 実装のスコープ

3. **docs/へのIssue番号紐付け**: Draftで作成した `docs/requirements.md` と `docs/adr/NNN-xxx.md` にIssue番号を追記

4. **完了報告**

   > ✅ Issue #{N} を作成しました（パターンB）。
   > Issue本文 + コメント（技術検討）を追加しました。
   > Draftの docs/requirements.md、docs/adr/NNN-xxx.md にIssue番号を紐付けました。
   > 次のステップ: `/issue-work {N}` を使用して実装を開始しますか？

---

## ラベル選定ルール

| パターン              | ラベル                   |
| --------------------- | ------------------------ |
| パターンA（バグ修正） | `bug`, `critical`        |
| パターンB（機能追加） | `enhancement`, `feature` |

既存のラベル一覧は `gh label list` で確認してください。

---

## 使用例

### パターンA

```
/issue-req
> ログイン時に500エラーが出る
# → パターンAと判定

/issue-create
# → Issue #123 作成（パターンA）
# → Issue本文 + コメント（分析結果）
```

### パターンB

```
/issue-req
> ユーザー管理機能を作りたい
# → パターンBと判定
# → Draft更新

/issue-create
# → Issue #124 作成（パターンB）
# → Issue本文 + コメント（技術検討）
# → docs/にIssue番号紐付け
```
