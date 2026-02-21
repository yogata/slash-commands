# HLD/LLD概念のSDDワークフローへの導入

## TL;DR

> **Quick Summary**: 仕様駆動開発（SDD）のSpecifications領域をHLD（High-Level Design）とLLD（Low-Level Design）に分割し、`implementation-guide.md`を新規追加する。
> 
> **Deliverables**:
> - 更新: `issue/README.md` - SDD連携セクション、ディレクトリ構造、境界線判定フロー
> - 更新: `issue/issue-req.md` - specifications.md（HLD）の更新を明確化
> - 更新: `issue/issue-work.md` - implementation-guide.md（LLD）の作成を追加
> - 更新: `issue/issue-close.md` - implementation-guide.mdのコミット処理を追加
> 
> **Estimated Effort**: Short
> **Parallel Execution**: NO - 順次更新
> **Critical Path**: README → issue-req → issue-work → issue-close

---

## Context

### Original Request
HLD/LLDの概念をスラッシュコマンドおよびREADMEに導入し、ワークフローを修正する。

### Interview Summary
**Key Discussions**:
- specifications.mdはHLDとする
- implementation-guide.mdを新規に作成してLLDとする
- ドキュメント類はspecs/に配置する

**Research Findings**:
- 現在の3領域（Requirements/Specifications/ADR）を4領域に拡張
- HLD: システム全体の構造、アーキテクチャ、コンポーネント間の関係
- LLD: クラス設計、関数シグネチャ、アルゴリズム

---

## Work Objectives

### Core Objective
SDDワークフローにHLD/LLD概念を導入し、設計の粒度を明確に分離する。

### Concrete Deliverables
- `issue/README.md` - 4領域への拡張、HLD/LLD概念定義の追加
- `issue/issue-req.md` - HLD（specifications.md）作成の明確化
- `issue/issue-work.md` - LLD（implementation-guide.md）作成の追加
- `issue/issue-close.md` - implementation-guide.mdのコミット処理追加

### Definition of Done
- [ ] 全ファイルが更新され、HLD/LLDの概念が一貫して反映されている
- [ ] 境界線の判定フローが更新されている
- [ ] specs/更新タイミング表が更新されている

### Must Have
- 4領域の責任領域表
- HLD vs LLDの概念定義表
- 更新されたディレクトリ構造

### Must NOT Have (Guardrails)
- 既存のワークフロー（req → create → work → close）を壊さない
- 既存のパターンA/Bの判定ロジックを変更しない

---

## Verification Strategy

### Test Decision
- **Infrastructure exists**: NO
- **Automated tests**: None
- **Framework**: none

### QA Policy
ドキュメント整合性の確認：
- 全ファイルで用語が一貫しているか
- フローが論理的に正しいか

---

## Execution Strategy

### Sequential Execution

```
Step 1: README.md更新（基盤）
├── 3つの責任領域 → 4つの責任領域へ拡張
├── HLD vs LLDの概念定義表を追加
├── specs/ディレクトリ構造を更新
└── 境界線の判定フローを更新

Step 2: issue-req.md更新
├── パターンB: specifications.md（HLD）の更新を明確化
└── HLDの記述内容例を追加

Step 3: issue-work.md更新
├── specifications.md更新 → HLD更新に変更
├── implementation-guide.md（LLD）作成ステップを追加
└── LLDの記述内容例を追加

Step 4: issue-close.md更新
├── specs/コミット対象にimplementation-guide.mdを追加
└── 実装記録テンプレートに関連SpecsにLLDを追加
```

### Dependency Matrix

- **Step 1**: — — 2, 3, 4
- **Step 2**: 1 — 3, 4
- **Step 3**: 1, 2 — 4
- **Step 4**: 1, 2, 3 —

---

## TODOs

- [ ] 1. README.md: 4つの責任領域への拡張

  **What to do**:
  - 「3つの責任領域」セクションを「4つの責任領域」に変更
  - Implementation Guide (LLD)の行を追加
  - 作成タイミング列を追加

  **Must NOT do**:
  - 既存のRequirements、ADRの定義を変更しない

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: テーブルの行追加のみの軽微な変更
  - **Skills**: []
    - なし（単純な編集）

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Sequential
  - **Blocks**: 2, 3, 4
  - **Blocked By**: None

  **References**:
  - `issue/README.md:27-35` - 現在の3つの責任領域テーブル

  **Acceptance Criteria**:
  - [ ] テーブルにImplementation Guide (LLD)の行が追加されている
  - [ ] 作成タイミング列が追加されている
  - [ ] 全4行が正しく表示されている

  **QA Scenarios**:
  ```
  Scenario: 4つの責任領域テーブルの確認
    Tool: Read
    Preconditions: README.mdが更新済み
    Steps:
      1. README.mdの「4つの責任領域」セクションを読み込む
      2. テーブルの行数が4であることを確認
      3. Implementation Guide (LLD)の行が存在することを確認
    Expected Result: 4行のテーブルが表示される
    Evidence: .sisyphus/evidence/task-1-table-check.md
  ```

  **Commit**: NO
  - Files: issue/README.md

---

- [ ] 2. README.md: HLD vs LLDの概念定義表を追加

  **What to do**:
  - 4つの責任領域セクションの直後に「HLD vs LLD の概念定義」セクションを追加
  - 観点、焦点、記述内容、質問に答える、読者、粒度の表を作成

  **Must NOT do**:
  - 既存のセクションと重複しない

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 新規セクションの追加
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Sequential
  - **Blocks**: 3, 4
  - **Blocked By**: 1

  **References**:
  - `specifications.md` - HLD vs LLDの概念定義表（ユーザーが提供）

  **Acceptance Criteria**:
  - [ ] 「HLD vs LLD の概念定義」セクションが追加されている
  - [ ] 6行×3列の表が存在する
  - [ ] 各列が正しい内容を持つ

  **QA Scenarios**:
  ```
  Scenario: HLD vs LLD概念定義表の確認
    Tool: Read
    Preconditions: README.mdが更新済み
    Steps:
      1. README.mdから「HLD vs LLD」セクションを読み込む
      2. 表が存在することを確認
      3. 「焦点」「記述内容」「質問に答える」「読者」「粒度」の行が存在することを確認
    Expected Result: 完全な概念定義表が表示される
    Evidence: .sisyphus/evidence/task-2-hld-lld-table.md
  ```

  **Commit**: NO

---

- [ ] 3. README.md: specs/ディレクトリ構造を更新

  **What to do**:
  - ディレクトリ構造に`implementation-guide.md`を追加
  - ドキュメント構成表にImplementation Guideを追加

  **Must NOT do**:
  - 既存のディレクトリ構造を削除しない

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Sequential
  - **Blocks**: 4
  - **Blocked By**: 1, 2

  **References**:
  - `issue/README.md:37-55` - 現在のspecs/ディレクトリ構造

  **Acceptance Criteria**:
  - [ ] `implementation-guide.md`がディレクトリ構造に追加されている
  - [ ] ドキュメント構成表が更新されている

  **QA Scenarios**:
  ```
  Scenario: ディレクトリ構造の確認
    Tool: Read
    Preconditions: README.mdが更新済み
    Steps:
      1. specs/ディレクトリ構造セクションを読み込む
      2. implementation-guide.mdが含まれていることを確認
    Expected Result: implementation-guide.mdが構造に含まれる
    Evidence: .sisyphus/evidence/task-3-dir-structure.md
  ```

  **Commit**: NO

---

- [ ] 4. README.md: 境界線の判定フローを更新

  **What to do**:
  - 境界線の判定フロー図にLLDの分岐を追加
  - 「ユーザー視点？→ No → 実装詳細？→ Yes → Implementation Guide (LLD)」の分岐を追加

  **Must NOT do**:
  - 既存のフローを壊さない

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Sequential
  - **Blocks**: 5
  - **Blocked By**: 1, 2, 3

  **References**:
  - `issue/README.md:56-75` - 現在の境界線判定フロー

  **Acceptance Criteria**:
  - [ ] 境界線判定フローにLLDの分岐が追加されている
  - [ ] フローが論理的に正しい

  **QA Scenarios**:
  ```
  Scenario: 境界線判定フローの確認
    Tool: Read
    Preconditions: README.mdが更新済み
    Steps:
      1. 境界線の判定セクションを読み込む
      2. LLD（Implementation Guide）への分岐が存在することを確認
    Expected Result: LLDへの分岐が存在する
    Evidence: .sisyphus/evidence/task-4-flow.md
  ```

  **Commit**: NO

---

- [ ] 5. README.md: specs/更新タイミング表を更新

  **What to do**:
  - specs/更新タイミング表にImplementation Guide列を追加
  - issue-workでのLLD作成を反映

  **Must NOT do**:
  - 既存のタイミングを変更しない

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Sequential
  - **Blocks**: 6
  - **Blocked By**: 1, 2, 3, 4

  **References**:
  - `issue/README.md:166-172` - 現在のspecs/更新タイミング表

  **Acceptance Criteria**:
  - [ ] Implementation Guide列が追加されている
  - [ ] issue-work行にLLD作成が反映されている
  - [ ] issue-close行にLLDコミットが反映されている

  **QA Scenarios**:
  ```
  Scenario: 更新タイミング表の確認
    Tool: Read
    Preconditions: README.mdが更新済み
    Steps:
      1. specs/更新タイミング表を読み込む
      2. Implementation Guide列が存在することを確認
    Expected Result: Implementation Guide列が存在する
    Evidence: .sisyphus/evidence/task-5-timing-table.md
  ```

  **Commit**: YES
  - Message: `docs(issue): add HLD/LLD concept to README`
  - Files: issue/README.md

---

- [ ] 6. issue-req.md: specifications.md（HLD）の更新を明確化

  **What to do**:
  - パターンBの手順4「Draft更新」を更新
  - `specs/specifications.md`の更新内容にHLDとしての記述例を追加
  - 「アーキテクチャ、コンポーネント間の関係、データフロー」を記述例に含める

  **Must NOT do**:
  - パターンAの手順を変更しない

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Sequential
  - **Blocks**: 7
  - **Blocked By**: 5

  **References**:
  - `issue/issue-req.md:79-127` - パターンBの手順

  **Acceptance Criteria**:
  - [ ] Draft更新の説明にHLDの記述内容例が追加されている
  - [ ] specifications.mdの位置づけがHLDであることが明記されている

  **QA Scenarios**:
  ```
  Scenario: issue-reqのHLD記述確認
    Tool: Read
    Preconditions: issue-req.mdが更新済み
    Steps:
      1. issue-req.mdのパターンBセクションを読み込む
      2. HLDの記述内容例が存在することを確認
    Expected Result: HLD記述例が存在する
    Evidence: .sisyphus/evidence/task-6-req-hld.md
  ```

  **Commit**: NO

---

- [ ] 7. issue-work.md: implementation-guide.md（LLD）の作成を追加

  **What to do**:
  - 手順5「specifications.md更新」を「HLD更新」に名称変更
  - 手順6として「implementation-guide.md（LLD）作成」を新規追加
  - LLDの記述内容例を追加（クラス設計、関数シグネチャ、アルゴリズム）

  **Must NOT do**:
  - 既存の手順番号を壊さない（追加により番号を繰り下げる）

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Sequential
  - **Blocks**: 8
  - **Blocked By**: 6

  **References**:
  - `issue/issue-work.md:104-126` - specifications.md更新セクション

  **Acceptance Criteria**:
  - [ ] LLD作成の手順が追加されている
  - [ ] 記述内容例が追加されている
  - [ ] 後続の手順番号が正しく繰り下がっている

  **QA Scenarios**:
  ```
  Scenario: issue-workのLLD手順確認
    Tool: Read
    Preconditions: issue-work.mdが更新済み
    Steps:
      1. issue-work.mdの手順セクションを読み込む
      2. implementation-guide.md（LLD）作成の手順が存在することを確認
    Expected Result: LLD作成手順が存在する
    Evidence: .sisyphus/evidence/task-7-work-lld.md
  ```

  **Commit**: NO

---

- [ ] 8. issue-close.md: implementation-guide.mdのコミット処理を追加

  **What to do**:
  - 手順3「specs/コミット」に`specs/implementation-guide.md`を追加
  - 実装記録テンプレートの「関連Specs」にLLDを追加

  **Must NOT do**:
  - パターンAの手順を変更しない

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Sequential
  - **Blocks**: None
  - **Blocked By**: 7

  **References**:
  - `issue/issue-close.md:46-54` - specs/コミットセクション
  - `issue/issue-close.md:79-102` - 実装記録テンプレート

  **Acceptance Criteria**:
  - [ ] specs/コミットにimplementation-guide.mdが追加されている
  - [ ] 実装記録テンプレートにLLDが追加されている

  **QA Scenarios**:
  ```
  Scenario: issue-closeのLLD対応確認
    Tool: Read
    Preconditions: issue-close.mdが更新済み
    Steps:
      1. issue-close.mdのspecs/コミットセクションを読み込む
      2. implementation-guide.mdが含まれていることを確認
    Expected Result: implementation-guide.mdがコミット対象に含まれる
    Evidence: .sisyphus/evidence/task-8-close-lld.md
  ```

  **Commit**: YES
  - Message: `docs(issue): add LLD to issue-work and issue-close`
  - Files: issue/issue-req.md, issue/issue-work.md, issue/issue-close.md

---

## Final Verification Wave

- [ ] F1. **Plan Compliance Audit** — `oracle`
  全TODOの完了確認、HLD/LLD概念が一貫して反映されているか検証。

- [ ] F2. **Document Consistency Review** — `unspecified-high`
  全ファイルで用語が一貫しているか確認。

---

## Commit Strategy

- **Commit 1**: `docs(issue): add HLD/LLD concept to README` — issue/README.md
- **Commit 2**: `docs(issue): add LLD to issue-work and issue-close` — issue/issue-req.md, issue/issue-work.md, issue/issue-close.md

---

## Success Criteria

### Verification Commands
```bash
# 全ファイルの更新確認
git diff --stat
```

### Final Checklist
- [ ] 全4ファイルが更新されている
- [ ] HLD/LLDの概念が一貫して反映されている
- [ ] 境界線判定フローが更新されている
- [ ] specs/更新タイミング表が更新されている
