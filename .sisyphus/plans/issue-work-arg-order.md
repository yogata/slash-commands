# issue-work 引数順番変更 + issue-pr 廃止

## TL;DR

> **Quick Summary**: `/issue-work` コマンドの引数順番を `<サブコマンド> <Issue番号>` に変更し、`/issue-pr` コマンドを廃止する
> 
> **Deliverables**:
> - issue/issue-work.md の引数説明・使用例変更
> - issue/issue-pr.md の削除
> - issue/README.md の issue-pr 参照削除
> 
> **Estimated Effort**: Quick
> **Parallel Execution**: YES - 3 tasks (Wave 1)
> **Critical Path**: なし（全タスク独立）

---

## Context

### Original Request
- `/issue-work` の引数順番を `cmd issue` に変更したい（cmd は省略可能は現状維持）
- `issue-pr` はもはや不要にできるのでは？

### Interview Summary
**Key Discussions**:
- 引数順番: `<Issue番号> <サブコマンド>` → `<サブコマンド> <Issue番号>`
- Issue番号は省略可能（現状維持）: `/issue-work test`
- サブコマンド省略時の動作: `/issue-work 101` → 全ステップ実行（現状維持）
- 複数Issue指定: `/issue-work test 101,102,103`
- issue-v1/ ディレクトリは対象外

### Metis Review
**Identified Gaps** (addressed):
- issue-v1/ ディレクトリの扱い → 対象外とする
- 複数Issue指定時の構文 → `test 101,102,103` に統一
- Issue番号のみ指定時の動作 → 全ステップ実行（現状維持）

---

## Work Objectives

### Core Objective
`/issue-work` コマンドの引数順番を直感的な順序に変更し、重複する `/issue-pr` コマンドを廃止する。

### Concrete Deliverables
- `issue/issue-work.md`: 引数説明と使用例を新しい順序に更新
- `issue/issue-pr.md`: ファイル削除
- `issue/README.md`: issue-pr への参照を削除

### Definition of Done
- [x] `/issue-work test 101` 形式の使用例が issue-work.md にある
- [x] `issue/issue-pr.md` が存在しない
- [x] `grep -r "issue-pr" issue/` でヒットしない

### Must Have
- 引数順番の変更（`<サブコマンド> <Issue番号>`）
- Issue番号省略時の動作維持
- サブコマンド省略時の動作維持（全ステップ実行）
- 複数Issue指定対応（`test 101,102,103`）
- issue-pr.md の削除

### Must NOT Have (Guardrails)
- `issue-v1/` ディレクトリへの変更
- 機能の変更（動作は現状維持）
- 後方互換性のサポート（旧引数順番は非サポート）

---

## Verification Strategy (MANDATORY)

### Test Decision
- **Infrastructure exists**: N/A
- **Automated tests**: None
- **Framework**: N/A

### QA Policy
ドキュメント変更のみ。以下の検証を実施：
- grep で issue-pr 参照が残っていないことを確認
- issue-work.md の使用例が正しい形式になっていることを確認

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (3 tasks in parallel):
├── Task 1: issue-work.md 引数順番変更 [quick]
├── Task 2: issue-pr.md 削除 [quick]
└── Task 3: issue/README.md issue-pr参照削除 [quick]
```

### Agent Dispatch Summary
- **1**: **3** — T1-T3 すべて `quick`

---

## TODOs

- [x] 1. issue-work.md 引数順番変更

  **What to do**:
  - 引数セクションの順序を `<サブコマンド>` → `<Issue番号>` に変更
  - 使用例を新しい順序に更新:
    - `/issue-work test 101` （Issue番号あり）
    - `/issue-work test` （Issue番号省略）
    - `/issue-work 101` （サブコマンド省略 = 全ステップ）
    - `/issue-work test 101,102,103` （複数Issue）
  - 複数Issue指定時の構文説明を追加

  **Must NOT do**:
  - 機能の変更
  - issue-v1/ への波及

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: テキスト編集のみの軽微な変更
  - **Skills**: []（不要）

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 2, 3)
  - **Blocks**: なし
  - **Blocked By**: なし

  **References**:
  - `issue/issue-work.md:13-39` - 引数セクションと使用例

  **Acceptance Criteria**:
  - [ ] 引数セクションの順序が `<サブコマンド>` → `<Issue番号>` になっている
  - [ ] 使用例が新しい順序に更新されている
  - [ ] サブコマンド省略時の動作説明が維持されている
  - [ ] 複数Issue指定の構文説明が追加されている

  **QA Scenarios**:

  ```
  Scenario: 引数セクションの順序確認
    Tool: Bash (grep)
    Preconditions: issue/issue-work.md が存在
    Steps:
      1. grep -n "サブコマンド" issue/issue-work.md | head -5
      2. grep -n "Issue番号" issue/issue-work.md | head -5
    Expected Result: サブコマンドの説明がIssue番号より前にある
    Evidence: .sisyphus/evidence/task-1-arg-order.txt
  ```

  **Commit**: YES
  - Message: `docs(issue-work): 引数順番を <サブコマンド> <Issue番号> に変更`
  - Files: `issue/issue-work.md`

---

- [x] 2. issue-pr.md 削除

  **What to do**:
  - `issue/issue-pr.md` を削除する
  - 削除確認メッセージを出力する

  **Must NOT do**:
  - `issue-v1/issue-pr.md` を削除する

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: ファイル削除のみの軽微な変更
  - **Skills**: []（不要）

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 3)
  - **Blocks**: なし
  - **Blocked By**: なし

  **References**:
  - `issue/issue-pr.md` - 削除対象ファイル

  **Acceptance Criteria**:
  - [ ] `issue/issue-pr.md` が存在しない
  - [ ] `issue-v1/issue-pr.md` は存在する（削除されていない）

  **QA Scenarios**:

  ```
  Scenario: ファイル削除確認
    Tool: Bash (ls)
    Preconditions: issue/ ディレクトリが存在
    Steps:
      1. ls issue/issue-pr.md 2>&1 || echo "File not found"
    Expected Result: "File not found" または類似のエラーメッセージ
    Evidence: .sisyphus/evidence/task-2-delete.txt
  ```

  **Commit**: YES
  - Message: `docs(issue): issue-pr.md を削除（issue-work pr で代替）`
  - Files: `issue/issue-pr.md` (deleted)

---

- [x] 3. issue/README.md issue-pr参照削除

  **What to do**:
  - 行19: コマンド一覧表から `/issue-pr` 行を削除
  - 行190: 「※ ワークフロー分断時: /issue-pr で...」を削除
  - 行330-350: `/issue-pr` セクション全体を削除
  - 行497: 詳細リンクから `/issue-pr` 行を削除

  **Must NOT do**:
  - `issue-v1/README.md` を変更する
  - 他のセクションへの影響

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: テキスト編集のみの軽微な変更
  - **Skills**: []（不要）

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 2)
  - **Blocks**: なし
  - **Blocked By**: なし

  **References**:
  - `issue/README.md:19` - コマンド一覧表
  - `issue/README.md:190` - レビューサイクル注記
  - `issue/README.md:330-350` - `/issue-pr` セクション
  - `issue/README.md:497` - 詳細リンク

  **Acceptance Criteria**:
  - [ ] `grep -n "issue-pr" issue/README.md` でヒットしない
  - [ ] コマンド一覧表が6コマンド→5コマンドになっている

  **QA Scenarios**:

  ```
  Scenario: issue-pr参照削除確認
    Tool: Bash (grep)
    Preconditions: issue/README.md が存在
    Steps:
      1. grep -n "issue-pr" issue/README.md
    Expected Result: ヒットなし（終了コード1）
    Evidence: .sisyphus/evidence/task-3-grep.txt
  ```

  **Commit**: YES
  - Message: `docs(issue): README.md から issue-pr への参照を削除`
  - Files: `issue/README.md`

---

## Final Verification Wave (MANDATORY — after ALL implementation tasks)

- [x] F1. **Plan Compliance Audit** — `quick`
  - grep で issue-pr 参照が `issue/` ディレクトリに残っていないことを確認
  - issue-work.md の使用例が正しい形式になっていることを確認
  Output: `issue-pr refs [0] | issue-work usage [correct] | VERDICT: APPROVE`

---

## Commit Strategy

3つのタスクは独立しているため、個別にコミット:
- Task 1: `docs(issue-work): 引数順番を <サブコマンド> <Issue番号> に変更`
- Task 2: `docs(issue): issue-pr.md を削除（issue-work pr で代替）`
- Task 3: `docs(issue): README.md から issue-pr への参照を削除`

---

## Success Criteria

### Verification Commands
```bash
# issue-pr 参照が残っていないことを確認
grep -r "issue-pr" issue/ | wc -l
# Expected: 0

# issue-work.md の使用例確認
grep -n "issue-work" issue/issue-work.md | grep -E "test [0-9]"
# Expected: /issue-work test 101 形式の使用例が表示される
```

### Final Checklist
- [x] All "Must Have" present
- [x] All "Must NOT Have" absent
- [x] No issue-pr references in issue/ directory
- [x] Usage examples in correct order
