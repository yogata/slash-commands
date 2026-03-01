---
description: /issue-req の結果をもとにGitHub Issueを作成する
---

# Issue登録

/issue-req で定義した要件または分析結果をもとに、GitHub Issueを作成してください。
規模パターン（A/B）に応じて処理が異なります。

## パターン判定

/issue-req で判定されたパターンに従って処理します：

| パターン    | 処理                    | ラベル                   |
| ----------- | ----------------------- | ------------------------ |
| **A（小）** | Issue本文のみ作成       | `bug` 等                 |
| **B（中）** | Issue作成 + docs/紐付け | `feature`, `enhancement` |

## 手順

### パターンA（小規模・バグ修正）

1. **Issue本文の作成**

   ```markdown
   ## 事象

   [何が起きているかの説明]

   ## 根本原因

   [原因の分析結果]

   ## 影響範囲

   [影響を受けるコンポーネント・機能]

   ## 再現手順

   1. [手順1]
   2. [手順2]

   ## 期待される動作

   [本来どう動くべきか]

   ## 対応方針

   [推奨される修正アプローチ]
   ```

2. **ラベルの選定**
   - `bug`, `critical` など
   - 既存のラベル一覧は `gh label list` で確認する

3. **Issueの作成**

   ```bash
   gh issue create --title "<タイトル>" --body "<本文>" --label "<ラベル>"
   ```

4. **完了報告**

   > ✅ Issue #{N} を作成しました（パターンA）。
   > 次のステップ: `/issue-work {N}` を使用して実装を開始しますか？

---

### パターンB（中規模・新機能追加）

1. **Issue本文の作成**

   ```markdown
   ## 概要

   [機能の概要と目的]

   ## 要件

   - [要件1]
   - [要件2]

   ## スコープ

   - 対象: [実装範囲]
   - 対象外: [今回の実装対象外]

   ## 技術的なメモ

   [想定される技術的アプローチや制約などがあれば記載]

   ## 関連docs

   - Requirements: `docs/requirements.md#xxx`
   - ADR: `docs/adr/NNN-xxx.md`
   ```

2. **ラベルの選定**
   - `enhancement`, `feature` など
   - 既存のラベル一覧は `gh label list` で確認する

3. **Issueの作成**

   ```bash
   gh issue create --title "<タイトル>" --body "<本文>" --label "<ラベル>"
   ```

4. **docs/へのIssue番号紐付け**
   - Draftで作成した `docs/requirements.md` と `docs/adr/NNN-xxx.md` にIssue番号を追記

5. **完了報告**

   > ✅ Issue #{N} を作成しました（パターンB）。
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
```

### パターンB

```
/issue-req
> ユーザー管理機能を作りたい
# → パターンBと判定
# → Draft更新

/issue-create
# → Issue #124 作成（パターンB）
# → docs/にIssue番号紐付け
```
