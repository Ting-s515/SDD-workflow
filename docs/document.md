# Propose Workflow 技能總覽

## 流程階段對應表

| 階段 | 技能 | 產出 / 任務 | 觸發方式 |
| --- | --- | --- | --- |
| 1. 規劃提案 | `propose` | 讀取規格文檔，識別功能清單，並為每個功能建立三份文檔 | `propose`、`幫我規劃這個功能`、提供規格文檔路徑 |
| 1a. 結構化流程 | `clarify-flow` | `01-flow.md`（由 `propose` 自動呼叫） | `propose` 內部呼叫；或 `clarify flow`、`把這段描述結構化` |
| 1b. 驗收條件 | `export-gherkin` | `02-gherkin.md`（由 `propose` 自動呼叫） | `propose` 內部呼叫；或 `幫我轉成 Gherkin` |
| 1c. 任務清單 | `propose` 本體 | `03-tasks.md`（由 `propose` 自行產出） | - |
| 2. 實作任務 | `apply` | 逐一實作 `03-tasks.md` 任務，更新 checkbox 狀態 | `apply`、`開始實作`、指定 `propose` 資料夾路徑 |
| 2a. Code Review | `code-reviewer` | `apply` 全部完成後由 sub-agent 自動執行，更新 `[x]` -> `[x][cr]` | `apply` 完成後自動觸發 |
| 3. 補上單元測試 | `bdd-unit-test` | 產出對應語言的測試檔案（`[manual]` 任務，需新 session 手動觸發） | `幫這個檔案寫單測`、`write unit test` |
| 4. 同步完成狀態 | `propose-sync` | 掃描所有 `[x][cr]` 完成的功能，同步回規格文檔的 `## 已完成` 區塊 | `propose-sync`、`同步完成狀態` |

## 技能相依關係

```text
規格需求文檔
    │
    ▼
propose ──► clarify-flow ──► 01-flow.md
    │                            │
    │      export-gherkin ◄──────┘
    │           │
    │           ▼
    │       02-gherkin.md
    │
    └──► 03-tasks.md
              │
              ▼
            apply
              ├──► 逐任務實作
              └──► code-reviewer (sub-agent, 自動)
                        │
                        ▼
                   [x][cr] 完成
                        │
                 bdd-unit-test (manual)
                        │
                        ▼
            propose-sync ──► 規格文檔 `## 已完成`
```
