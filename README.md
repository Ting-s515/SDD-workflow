# SDD Workflow

`SDD Workflow` 是一套將需求整理、提案、實作、審查與完成狀態同步串接起來的技能流程，目的是讓規格文件可以穩定轉換為可執行任務、實作結果與後續驗證產出。

本文件聚焦於兩個層次：

- 核心流程：需求如何從規格文檔一路走到實作完成
- 延伸流程：在核心流程之外，可選擇接入的 AC-first、Feature File 與 React 審查能力

## 核心技能總覽

整體流程以規格需求文檔為起點，先透過 `propose` 建立提案與任務，再交由 `apply` 實作，最後視需求補上測試並同步完成狀態。

## 核心流程階段對應表

| 階段            | 技能             | 產出 / 任務                                                      | 觸發方式                                                  |
| --------------- | ---------------- | ---------------------------------------------------------------- | --------------------------------------------------------- |
| 1. 規劃提案     | `propose`        | 讀取規格文檔，識別功能清單，並為每個功能建立三份文檔             | `propose`、`幫我規劃這個功能`、提供規格文檔路徑           |
| 1a. 結構化流程  | `clarify-flow`   | `01-flow.md`（由 `propose` 自動呼叫）                            | `propose` 內部呼叫；或 `clarify flow`、`把這段描述結構化` |
| 1b. 驗收條件    | `export-gherkin` | `02-gherkin.md`（由 `propose` 自動呼叫）                         | `propose` 內部呼叫；或 `幫我轉成 Gherkin`                 |
| 1c. 任務清單    | `propose` 本體   | `03-tasks.md`（由 `propose` 自行產出）                           | -                                                         |
| 2. 實作任務     | `apply`          | 逐一實作 `03-tasks.md` 任務，更新 checkbox 狀態                  | `apply`、`開始實作`、指定 `propose` 資料夾路徑            |
| 2a. Code Review | `code-reviewer`  | `apply` 全部完成後由 sub-agent 自動執行，更新 `[x]` -> `[x][cr]` | `apply` 完成後自動觸發                                    |
| 3. 補上單元測試 | `bdd-unit-test`  | 產出對應語言的測試檔案（`[manual]` 任務，需新 session 手動觸發） | `幫這個檔案寫單測`、`write unit test`                     |
| 4. 同步完成狀態 | `propose-sync`   | 掃描所有 `[x][cr]` 完成的功能，同步回規格文檔的 `## 已完成` 區塊 | `propose-sync`、`同步完成狀態`                            |

## 核心技能相依關係

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

## 核心流程補充說明

- `propose` 不只建立三份文檔，也會回寫規格文檔中的 `> propose:` 標記，作為後續 `propose-sync` 判斷掃描根目錄的依據。
- `propose` 可處理一般功能需求，也可處理 `bug fix list` 中標記為 `[propose]` 或需進一步分析的項目。
- `apply` 只會自動執行一般任務；標記為 `[manual]` 的任務會保留給使用者在新 session 手動觸發。
- `code-reviewer` 是核心流程的一部分；若變更包含 React 前端檔案，審查時會額外套用 `react-design` 的原則。

## 延伸技能

以下技能不是核心主線的必經步驟，但可接入 workflow，補強需求驗收、測試先行與 BDD 執行能力。

| 技能                  | 角色                                          | 適用時機                                          |
| --------------------- | --------------------------------------------- | ------------------------------------------------- |
| `export-ac`           | 先從需求整理出 `AC.md` 驗收準則文件           | 需要先定義完成標準，再進入實作                    |
| `ac-to-test`          | 將 `AC.md` 轉成紅燈測試骨架                   | 想採用 AC-first / test-first 流程時               |
| `export-feature-file` | 將規格或 Gherkin 轉成可執行的 `.feature` 檔案 | 需要接入 Reqnroll、Cucumber、Behave 等 BDD 框架時 |
| `react-design`        | 提供 React 架構與最佳實踐檢查原則             | Code review 或前端設計討論包含 React 時           |

## 延伸流程範例

### AC-first 路徑

```text
規格需求文檔
    │
    ▼
export-ac ──► AC.md
    │
    ▼
ac-to-test ──► 測試骨架（紅燈）
    │
    ▼
propose / apply ──► 實作與驗證
```

### BDD 執行檔路徑

```text
01-flow.md / 需求規格
    │
    ▼
export-gherkin ──► 02-gherkin.md
    │
    └──► export-feature-file ──► .feature
```

## 使用方式

1. 準備規格需求文檔。
2. 執行 `propose` 產出 `01-flow.md`、`02-gherkin.md` 與 `03-tasks.md`。
3. 執行 `apply` 逐步實作任務並完成 code review。
4. 視需求使用 `bdd-unit-test` 補上實作後單元測試，或使用 `export-ac` + `ac-to-test` 走測試先行流程。
5. 若需要接入 BDD 測試框架，可使用 `export-feature-file` 產出 `.feature`。
6. 執行 `propose-sync` 將完成狀態同步回原始規格文檔。

。
