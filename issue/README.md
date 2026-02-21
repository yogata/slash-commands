---
description: issue-* コマンドセットの使用ガイド
---

# issue-\* コマンド使用ガイド

機能追加とバグ修正を統一された `issue-*` コマンドセットでサポートします。
**仕様駆動開発（SDD）**と連携し、要件から実装までのトレーサビリティを確保します。

## 概要

issue-\* コマンドはGitHub Issue管理とGit操作を効率化するワークフローです。5コマンドで以下をカバーします：

| コマンド        | 役割             | フェーズ    |
| --------------- | ---------------- | ----------- |
| `/issue-req`    | 要件定義         | 思考・分析  |
| `/issue-create` | Issue登録        | GitHub作成  |
| `/issue-work`   | 実装パイプライン | 実行        |
| `/issue-update` | Issue更新        | GitHub更新  |
| `/issue-close`  | 完了処理         | GitHub・Git |

---

## SDD（仕様駆動開発）との連携

### 4つの責任領域

issue-\*コマンドは、以下の4つの責任領域を明確に分離して管理します：

| 領域                        | 責任                       | 視点                       | ファイル                         | 作成タイミング  |
| --------------------------- | -------------------------- | -------------------------- | -------------------------------- | --------------- |
| **Requirements**            | 「何が必要か」             | ユーザー・ステークホルダー | specs/requirements.md            | `/issue-req`    |
| **Specifications (HLD)**    | 「何を作るか」「全体はどうあるか」 | システム・プロダクト | specs/specifications.md          | `/issue-req`    |
| **Implementation Guide (LLD)** | 「どう実装するか」      | コード                     | specs/implementation-guide.md    | `/issue-work`   |
| **ADR**                     | 「なぜその選択か」         | アーキテクト               | specs/adr/001-xxx.md             | `/issue-req`    |

### HLD vs LLD の概念定義

| 観点             | High-Level Design (HLD)                      | Low-Level Design (LLD)                   |
| ---------------- | -------------------------------------------- | ---------------------------------------- |
| 焦点             | システム全体の構造                           | 個々のコンポーネントの内部               |
| 記述内容         | アーキテクチャ、コンポーネント間の関係、データフロー | クラス設計、関数シグネチャ、アルゴリズム |
| 質問に答える     | 「何を作るか」「全体はどうあるか」           | 「どう実装するか」                       |
| 読者             | ステークホルダー、アーキテクト               | 開発者                                   |
| 粒度             | システム〜モジュール                         | クラス〜関数                             |

### specs/ディレクトリ構造

```
specs/
├── requirements.md          # プロジェクト全体の要求
├── specifications.md        # プロジェクト全体の仕様（HLD）
├── implementation-guide.md  # 実装ガイド（LLD）
└── adr/
    ├── 001-xxx.md           # 意思決定記録（番号付き）
    ├── 002-xxx.md
    └── ...
```

| ドキュメント              | 構成         | 理由                         | 種類   |
| ------------------------- | ------------ | ---------------------------- | ------ |
| **Requirements**          | 1ファイル    | 「現在の状態」を一箇所に集約 | -      |
| **Specifications**        | 1ファイル    | 「現在の状態」を一箇所に集約 | HLD    |
| **Implementation Guide**  | 1ファイル    | 「現在の状態」を一箇所に集約 | LLD    |
| **ADR**                   | 複数ファイル | 「意思決定の歴史」を蓄積     | -      |

### 境界線の判定

```
記述しようとしている内容
        │
        ▼
┌───────────────────┐
│ ユーザー視点？      │──Yes──→ Requirements
│ （何が欲しいか）    │
└─────────┬─────────┘
          │ No
          ▼
┌───────────────────┐
│ なぜその選択か？    │──Yes──→ ADR
│ （判断根拠）       │
└─────────┬─────────┘
          │ No
          ▼
┌───────────────────┐
│ 実装詳細か？        │──Yes──→ Implementation Guide (LLD)
│ （関数・クラス等）  │        （どう実装するか）
└─────────┬─────────┘
          │ No
          ▼
    Specifications (HLD)
    （何を作るか・全体はどうあるか）
```

---

## 規模別運用パターン

変更の規模に応じて、2つのパターンを自動判定します：

### パターン一覧

| パターン    | 規模               | specs/更新 | ADR  | 判定方法 |
| ----------- | ------------------ | ---------- | ---- | -------- |
| **A（小）** | バグ修正・軽微変更 | なし       | なし | 自動判定 |
| **B（中）** | 新機能追加         | あり       | 1個  | 自動判定 |

### 自動判定ロジック

```
/issue-req 実行時
    │
    ▼
┌─────────────────────────────────────────┐
│ キーワード判定                            │
├─────────────────────────────────────────┤
│ 「修正」「変更」「バグ」「エラー」等       │
│          │                               │
│          └─→ パターンA（小規模）          │
│                                          │
│ 「追加」「新規」「作成」「実装」「機能」等  │
│          │                               │
│          └─→ パターンB（中規模）          │
└─────────────────────────────────────────┘
```

---

## ワークフロー

### 基本フロー

```
/issue-req → /issue-create → /issue-work → /issue-close
```

**文脈による自動切り替え**:

- 機能追加: `/issue-req` で要件定義
- バグ修正: `/issue-req` で問題分析

### SDD連携フロー

```
/issue-req
    │
    ├─ パターンA: Issue本文のみ → /issue-create
    │
    └─ パターンB: Draft更新
         - requirements.md 追加
         - ADR追加（status: proposed）
         │
         ▼
    /issue-create → Issue作成
         │
         ▼
    /issue-work
         - specifications.md 更新
         - ADR status更新（proposed → accepted）
         - 実装
         │
         ▼
    /issue-close
         - specs/コミット（コードと同時）
         - PRマージ → Issueクローズ
```

### レビューサイクル

PR作成後、ユーザーによるレビューが行われます：

```
/issue-work（PR作成）→ ユーザーレビュー（Web）
  ├─ OK: /issue-close（PRマージ）
  │
  ├─ NG（仕様バグ）: /issue-update（本文更新）→ /issue-work（再実装）
  │   └─ 要件の不足・誤りがある場合
  │
  └─ NG（実装バグ）: /issue-update --comment（コメント追加）→ /issue-work（再実装）
      └─ 実装が要件通りでない場合
```

### specs/更新タイミング

| フェーズ         | Requirements | Specifications (HLD) | Implementation Guide (LLD) | ADR          | コミット                 |
| ---------------- | ------------ | -------------------- | -------------------------- | ------------ | ------------------------ |
| **issue-req**    | 追加・更新   | （空または概要）     | -                          | proposed追加 | なし                     |
| **issue-create** | Issue紐付け  | -                    | -                          | -            | なし                     |
| **issue-work**   | -            | 実装に合わせて更新   | 実装詳細を記述             | accepted更新 | なし                     |
| **issue-close**  | 最終確認     | 最終確認             | 最終確認                   | 最終確認     | **あり**（コードと一緒） |

### 競合対応

specs/の競合は通常のGit競合解決と同じです：

```bash
# 競合発生時の対応
git fetch origin
git rebase origin/main  # または git merge origin/main

# 競合解決
# ... エディタで競合箇所を修正 ...

git add specs/requirements.md
git rebase --continue  # または git commit

git push --force-with-lease
```

---

## 各コマンドの詳細

### `/issue-req` — 要件定義

機能追加またはバグ修正の要件を整理・定義します。

**用途:**

- 機能追加: 要件の概要・目的・スコープを整理
- バグ修正: エラーログ・再現手順を分析し原因を特定
- @planへの入力を整える
- **規模判定: パターンA/Bを自動判定**

**オプション:**

- `--feature` — 機能追加モード
- `--bug` — バグ修正モード
- （省略時は自動判定）

**出力:**

- 機能追加: 構造化された要件定義（概要・目的・要件リスト・スコープ）
- バグ修正: 事象分析（概要・根本原因・影響範囲・対応方針）
- **パターンB: Draft更新（requirements.md、ADR）**

**次のステップ:**

```
/issue-create
```

---

### `/issue-create` — Issue登録

定義した要件または分析結果をもとに、GitHub Issueを作成します。

**用途:**

- GitHub Issue作成
- ラベル選定（enhancement, feature / bug, critical）
- Issue番号の取得

**引数:**

- （なし）— 直前のreqの結果を使用

**出力:**

- 作成されたGitHub Issue
- Issue番号（後続のステップで必要）

**ラベル選定ルール:**

- 機能追加: `enhancement`, `feature`
- バグ修正: `bug`, `critical`

**次のステップ:**

```
/issue-work <Issue番号>
```

---

### `/issue-work` — 実装パイプライン

選択したIssueに対して、計画立案から実装・コミットまでを一気通貫で実行します。

**用途:**

- ブランチ作成（feature/fix命名規則）
- @planによる計画立案
- /start-workによる実装
- コミット・プッシュ・PR作成（feat/fixメッセージ形式）
- **パターンB: specifications.md更新、ADR status更新**

**引数:**

- `<Issue番号>` — 対象のIssue番号
  - 単一: `101`
  - 複数: `101,102,103`（カンマ区切りで並列実行）
  - 省略時は直前のissue-createで作成したIssueを使用

**内部フロー:**

1. Issue確認 + 文脈判定（feature/bug）
2. Worktree作成: `git worktree add .worktrees/{N}-feature -b feature/issue-{N}`
3. `@plan` による計画立案
4. `/start-work` による実装
5. **specifications.md更新（パターンB）**
6. **ADR status更新: proposed → accepted（パターンB）**
7. テスト実施
8. コミット: `git commit -m "feat/fix: ... (#{N})"`
9. プッシュ
10. PR作成

**Git Worktree構造:**

```
/project (main worktree)
  /.worktrees/
    /101-feature (feature/issue-101)
    /102-fix (fix/issue-102)
```

**出力:**

- 変更したファイルと変更内容
- テスト結果
- コミットハッシュ

**次のステップ:**

```
/issue-close <Issue番号>
```

---

### `/issue-close` — 完了処理

実施した内容をGitHub Issueに記録として追記し、クローズ後に作業ブランチを削除します。

**用途:**

- 対応記録の追記
- PRマージ
- Issueクローズ
- ブランチクリーンアップ
- 規約チェック（記録なしクローズ防止）
- **specs/コミット（コードと同時）**

**引数:**

- `<Issue番号>` — 対象のIssue番号（省略時は直前のissue-workで使用したIssueを使用）

**内部フロー:**

1. 記録が未追記のことを確認（規約チェック）
2. 記録作成（実装記録/対応記録）
3. Issueへのコメント追記
4. **specs/コミット（requirements.md、specifications.md、ADR）**
5. PRマージ: `gh pr merge <N> --merge --delete-branch` (Merge Commit + リモートブランチ削除)
6. Issueクローズ: `gh issue close <N> --reason completed`
7. Worktree削除: `git worktree remove .worktrees/{N}-feature`
8. ローカルブランチ削除: `git branch -d ...` + `git fetch --prune`

**出力:**

- クローズしたIssue番号
- 削除したブランチ名

---

## Git 規約

### ブランチ命名

- 機能追加: `feature/issue-{NUMBER}`
- バグ修正: `fix/issue-{NUMBER}`

### コミットメッセージ

- 機能: `feat: {要約} (#{ISSUE_NUMBER})`
- バグ: `fix: {要約} (#{ISSUE_NUMBER})`

### GitHub ラベル

- 機能追加: `enhancement`, `feature`
- バグ修正: `bug`, `critical`

---

## アンチパターン（回避すべき操作）

- フローのステップをスキップしない（req → create → work → close は必須）
- 作業ブランチを削除せずに次のIssueに進まない
- Issueへの記録追記なしでクローズしない
- コミットメッセージにIssue番号を含めない
- PRなしで `/issue-close` を実行しない
- squash/rebase マージを使用しない
- specs/の競合を無視してマージしない

---

## 使用例

### パターンA（小規模・バグ修正）

```
# 1. 要件定義（自動的にパターンAと判定）
/issue-req
> ログイン時にエラーが出る

# 2. Issue作成
/issue-create
# → Issue #123 作成

# 3. 実装
/issue-work 123
# → Worktree作成 → 計画 → 実装 → PR作成

# 4. 完了
/issue-close 123
# → 記録追記 → クローズ → Worktree/ブランチ削除
```

### パターンB（中規模・新機能）

```
# 1. 要件定義（自動的にパターンBと判定）
/issue-req
> ユーザー管理機能を作りたい
# → Draft更新（requirements.md、ADR）

# 2. Issue作成
/issue-create
# → Issue #124 作成

# 3. 実装
/issue-work 124
# → specifications.md更新 → ADR accepted更新 → 実装 → PR作成

# 4. 完了
/issue-close 124
# → specs/コミット → 記録追記 → クローズ
```

---

## 親子Issueの紐付け

複数のIssueをまとめて管理したい場合は、GitHubの**Sub-issues機能**を使用してください：

- 子Issueを親Issueに紐付けることで、進捗を一元管理
- 親Issue画面で子Issueのステータスを確認可能

---

## 詳細

各コマンドの詳細な手順は以下を参照してください：

- `/issue-req` — [issue-req.md](./issue-req.md)
- `/issue-create` — [issue-create.md](./issue-create.md)
- `/issue-work` — [issue-work.md](./issue-work.md)
- `/issue-update` — [issue-update.md](./issue-update.md)
- `/issue-close` — [issue-close.md)
