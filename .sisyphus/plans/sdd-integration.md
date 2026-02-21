# SDD統合：issue-*コマンドへの仕様駆動開発導入

## TL;DR

> **Quick Summary**: issue-*コマンドセットに仕様駆動開発（SDD）概念を統合し、Requirements/Specifications/ADRを管理する仕組みを追加する
>
> **Deliverables**:
> - 更新された issue/README.md（SDD概念説明）
> - 更新された各コマンド（issue-req, issue-create, issue-work, issue-close）
> - specs/ディレクトリ構造定義
>
> **Estimated Effort**: Medium
> **Parallel Execution**: NO - ドキュメント間の整合性が必要
> **Critical Path**: README.md → issue-req.md → issue-create.md → issue-work.md → issue-close.md

---

## Context

### 議論の背景

1. **SDD概念の改良**：従来の「自然言語廃止」という過激なアプローチから、「3つの責任領域への分離」へ転換
2. **issue-reqの拡張**：要件深堀りの結果をSpecsとして保存するワークフローを追加
3. **規模別運用**：パターンA/B/C（小/中/大）で自動判定＋明示指定

### 合意事項

#### 3つの責任領域

| 領域 | 責任 | 視点 | ファイル |
|-----|------|------|---------|
| **Requirements** | 「何が必要か」 | ユーザー | requirements.md |
| **Specifications** | 「どう動くか」 | システム | specifications.md |
| **ADR** | 「なぜその選択か」 | アーキテクト | adr/001-xxx.md |

#### 規模別運用パターン

| パターン | 規模 | specs/更新 | ADR | 判定方法 |
|---------|------|----------|-----|---------|
| **A（小）** | バグ修正・軽微変更 | なし | なし | 自動判定 |
| **B（中）** | 新機能追加 | あり | 1個 | 自動判定 |
| **C（大）** | Epic・大規模変更 | あり | 複数 | ユーザー明示（--epic） |

#### specs/更新タイミング

- issue-req完了時：Draft更新（コミットなし）
- issue-work中：specifications.md更新
- issue-close時：Final確定・コミット（コードと同時）

#### 競合対応

specs/の競合は通常のGit競合解決と同じ：
1. `git rebase origin/main` または `git merge origin/main`
2. 競合解決
3. force push
4. PR再レビュー

---

## Work Objectives

### Core Objective

issue-*コマンドセットにSDD概念を統合し、要件から実装までのトレーサビリティを確保する

### Concrete Deliverables

1. `issue/README.md` - SDD概念、規模別パターン、更新後のコマンド説明
2. `issue/issue-req.md` - 規模判定、Draft更新、--epicオプション
3. `issue/issue-create.md` - パターン別処理、親子Issue紐付け
4. `issue/issue-work.md` - specifications.md更新、ADR status更新
5. `issue/issue-close.md` - specs/コミット処理
6. specs/ディレクトリ構造定義

### Definition of Done

- [x] 各コマンドのドキュメントが更新されている
- [x] README.mdにSDD概念が説明されている
- [x] 規模判定ロジックが明記されている
- [x] specs/ディレクトリ構造が定義されている
- [x] パターンCが削除され、A/Bの2パターンになっている

---

## TODOs

- [x] 1. issue/README.mdを更新

  **What to do**:
  - SDD概念セクションを追加（Requirements/Specifications/ADRの3領域）
  - 規模別運用パターン（A/B/C）の説明を追加
  - specs/ディレクトリ構造の説明を追加
  - 各コマンドの拡張内容を反映
  - ワークフロー図を更新

  **Must NOT do**:
  - 既存のワークフロー説明を削除しない（拡張する）
  - コマンド単体の説明をREADMEに重複させない

  **References**:
  - `.sisyphus/drafts/sdd-concept-v2.md` - 合意したSDD概念
  - `issue/README.md` - 現在の内容

  **Acceptance Criteria**:
  - [x] SDD概念が追加されている
  - [x] パターンA/B/Cが説明されている
  - [x] --epicオプションが説明されている

---

- [x] 2. issue-req.mdを更新

  **What to do**:
  - 規模判定ロジックを追加（パターンA/Bの自動判定）
  - `--epic` / `--parent` オプションを追加（パターンC）
  - Draft更新フローを追加
  - 出力形式を更新（パターン別）

  **Must NOT do**:
  - 既存の機能追加/バグ修正フローを削除しない

  **References**:
  - `issue/issue-req.md` - 現在の内容
  - `.sisyphus/drafts/sdd-concept-v2.md` - 改良5の詳細

  **Acceptance Criteria**:
  - [x] 規模判定ロジックが追加されている
  - [x] --epicオプションが追加されている
  - [x] Draft更新フローが追加されている

---

- [x] 3. issue-create.mdを更新

  **What to do**:
  - パターン別の処理を追加
  - パターンC（親Issue）の作成フローを追加
  - 子Issue作成の指示を追加
  - ラベル選定ルールに `epic` を追加

  **Must NOT do**:
  - 既存のIssue作成フローを削除しない

  **References**:
  - `issue/issue-create.md` - 現在の内容

  **Acceptance Criteria**:
  - [x] パターン別処理が追加されている
  - [x] 親子Issue紐付けが説明されている

---

- [x] 4. issue-work.mdを更新

  **What to do**:
  - specifications.md更新フローを追加（パターンB/C）
  - ADR status更新（proposed → accepted）を追加
  - specs/競合対応の説明を追加

  **Must NOT do**:
  - 既存の実装フローを削除しない

  **References**:
  - `issue/issue-work.md` - 現在の内容

  **Acceptance Criteria**:
  - [x] specifications.md更新フローが追加されている
  - [x] 競合対応が説明されている

---

- [x] 5. issue-close.mdを更新

  **What to do**:
  - specs/コミット処理を追加
  - パターンC（親Issue）の完了条件を追加
  - 記録作成テンプレートにSpecs参照を追加

  **Must NOT do**:
  - 既存のクローズフローを削除しない

  **References**:
  - `issue/issue-close.md` - 現在の内容

  **Acceptance Criteria**:
  - [x] specs/コミット処理が追加されている
  - [x] 親Issue完了条件が説明されている

---

- [x] 6. specs/ディレクトリ構造を定義

  **What to do**:
  - specs/ディレクトリの構造をドキュメント化
  - 初期ファイルのテンプレートを作成
  - ADRテンプレートを作成

  **Must NOT do**:
  - 実際にファイルを作成しない（定義のみ）

  **References**:
  - `.sisyphus/drafts/sdd-concept-v2.md` - 改良1のファイル構造

  **Acceptance Criteria**:
  - [x] ディレクトリ構造が定義されている
  - [x] 各ファイルのテンプレートが記載されている

  **定義内容**:

  ### ディレクトリ構造

  ```
  specs/
  ├── requirements.md      # プロジェクト全体の要求（人間が書く）
  ├── specifications.md    # プロジェクト全体の仕様（人間が書く）
  └── adr/
      ├── 001-xxx.md       # 意思決定記録（番号付き）
      ├── 002-xxx.md
      └── ...
  ```

  ### requirements.md テンプレート

  ```markdown
  # Requirements

  このドキュメントは、プロジェクト全体の要件を管理します。

  ---

  ## {機能名}

  ### 概要
  [機能の概要]

  ### 要件
  - [要件1]
  - [要件2]

  ### 関連Issue
  - #xxx

  ---
  ```

  ### specifications.md テンプレート

  ```markdown
  # Specifications

  このドキュメントは、プロジェクト全体の仕様を管理します。

  ---

  ## {機能名}

  ### 仕様
  - [具体的な仕様]
  - [APIエンドポイント]
  - [データ構造]

  ### 技術制約
  - [制約事項]

  ### 関連Issue
  - #xxx

  ---
  ```

  ### ADR テンプレート

  ```markdown
  # ADR-{番号}: {タイトル}

  ## Status
  proposed | accepted | deprecated | superseded

  ## Context
  [背景・制約・問題提起]

  ## Decision
  [決定内容]

  ## Alternatives
  [検討した代替案と却下理由]

  ## Consequences
  [結果・影響・トレードオフ]

  ## References
  - Issue #xxx
  ```

---

- [x] 7. パターンCを削除してA/Bの2パターンに簡素化

  **背景**:
  - 当初の想定（複数スプリントにまたぐEpic）と実際の用途が異なった
  - 実際は「数日〜1週間、3〜5個の子Issue」規模
  - この規模ならGitHub Sub-issues機能で紐付けるだけで十分

  **What to do**:
  - `issue/README.md`: パターンCを削除、親子紐付けの説明を追加
  - `issue/issue-req.md`: `--epic` / `--parent` オプションを削除
  - `issue/issue-create.md`: パターンCの処理を削除
  - `issue/issue-work.md`: パターンCの記述をパターンBに統合
  - `issue/issue-close.md`: パターンCの完了条件を削除

  **Must NOT do**:
  - パターンA/Bの機能を削除しない
  - specs/更新フローを削除しない

  **Acceptance Criteria**:
  - [x] パターンがA/Bの2つになっている
  - [x] --epic/--parentオプションが削除されている
  - [x] 親子紐付けはGitHub Sub-issuesで対応することが記載されている
  - [x] 各ファイルからパターンCの記述が削除されている

---

- [x] 2. issue-req.mdを更新

  **What to do**:
  - 規模判定ロジックを追加（パターンA/Bの自動判定）
  - `--epic` / `--parent` オプションを追加（パターンC）
  - Draft更新フローを追加
  - 出力形式を更新（パターン別）

  **Must NOT do**:
  - 既存の機能追加/バグ修正フローを削除しない

  **References**:
  - `issue/issue-req.md` - 現在の内容
  - `.sisyphus/drafts/sdd-concept-v2.md` - 改良5の詳細

  **Acceptance Criteria**:
  - [ ] 規模判定ロジックが追加されている
  - [ ] --epicオプションが追加されている
  - [ ] Draft更新フローが追加されている

---

- [x] 3. issue-create.mdを更新

  **What to do**:
  - パターン別の処理を追加
  - パターンC（親Issue）の作成フローを追加
  - 子Issue作成の指示を追加
  - ラベル選定ルールに `epic` を追加

  **Must NOT do**:
  - 既存のIssue作成フローを削除しない

  **References**:
  - `issue/issue-create.md` - 現在の内容

  **Acceptance Criteria**:
  - [ ] パターン別処理が追加されている
  - [ ] 親子Issue紐付けが説明されている

---

- [x] 4. issue-work.mdを更新

  **What to do**:
  - specifications.md更新フローを追加（パターンB/C）
  - ADR status更新（proposed → accepted）を追加
  - specs/競合対応の説明を追加

  **Must NOT do**:
  - 既存の実装フローを削除しない

  **References**:
  - `issue/issue-work.md` - 現在の内容

  **Acceptance Criteria**:
  - [ ] specifications.md更新フローが追加されている
  - [ ] 競合対応が説明されている

---

- [x] 5. issue-close.mdを更新

  **What to do**:
  - specs/コミット処理を追加
  - パターンC（親Issue）の完了条件を追加
  - 記録作成テンプレートにSpecs参照を追加

  **Must NOT do**:
  - 既存のクローズフローを削除しない

  **References**:
  - `issue/issue-close.md` - 現在の内容

  **Acceptance Criteria**:
  - [ ] specs/コミット処理が追加されている
  - [ ] 親Issue完了条件が説明されている

---

- [x] 6. specs/ディレクトリ構造を定義

  **What to do**:
  - specs/ディレクトリの構造をドキュメント化
  - 初期ファイルのテンプレートを作成
  - ADRテンプレートを作成

  **Must NOT do**:
  - 実際にファイルを作成しない（定義のみ）

  **References**:
  - `.sisyphus/drafts/sdd-concept-v2.md` - 改良1のファイル構造

  **Acceptance Criteria**:
  - [x] ディレクトリ構造が定義されている
  - [x] 各ファイルのテンプレートが記載されている

  **定義内容**:

  ### ディレクトリ構造

  ```
  specs/
  ├── requirements.md      # プロジェクト全体の要求（人間が書く）
  ├── specifications.md    # プロジェクト全体の仕様（人間が書く）
  └── adr/
      ├── 001-xxx.md       # 意思決定記録（番号付き）
      ├── 002-xxx.md
      └── ...
  ```

  ### requirements.md テンプレート

  ```markdown
  # Requirements

  このドキュメントは、プロジェクト全体の要件を管理します。

  ---

  ## {機能名}

  ### 概要
  [機能の概要]

  ### 要件
  - [要件1]
  - [要件2]

  ### 関連Issue
  - #xxx

  ---
  ```

  ### specifications.md テンプレート

  ```markdown
  # Specifications

  このドキュメントは、プロジェクト全体の仕様を管理します。

  ---

  ## {機能名}

  ### 仕様
  - [具体的な仕様]
  - [APIエンドポイント]
  - [データ構造]

  ### 技術制約
  - [制約事項]

  ### 関連Issue
  - #xxx

  ---
  ```

  ### ADR テンプレート

  ```markdown
  # ADR-{番号}: {タイトル}

  ## Status
  proposed | accepted | deprecated | superseded

  ## Context
  [背景・制約・問題提起]

  ## Decision
  [決定内容]

  ## Alternatives
  [検討した代替案と却下理由]

  ## Consequences
  [結果・影響・トレードオフ]

  ## References
  - Issue #xxx
  ```

---

## Final Verification Wave

- [x] F1. **整合性確認** - 各コマンド間のフローが矛盾なく記述されている
- [x] F2. **完全性確認** - 合意事項がすべてドキュメントに反映されている
- [x] F3. **パターンC削除確認** - パターンCが削除され、A/Bの2パターンになっている

---

## Commit Strategy

各ファイル更新後にコミット：
1. `docs(issue): add SDD concept to README`
2. `docs(issue-req): add scale detection and draft update`
3. `docs(issue-create): add pattern-based processing`
4. `docs(issue-work): add specs update flow`
5. `docs(issue-close): add specs commit flow`

---

## Success Criteria

### Verification Commands

```bash
# 各ファイルが存在し、更新されている
cat issue/README.md | grep "SDD"
cat issue/issue-req.md | grep "パターン"
cat issue/issue-create.md | grep "親子"
```

### Final Checklist

- [x] README.mdにSDD概念が追加されている
- [x] 規模判定（A/B）が説明されている
- [x] 各コマンドの変更点が反映されている
- [x] --epic/--parentオプションが削除されている
- [x] 親子紐付けはGitHub Sub-issuesで対応することが記載されている
