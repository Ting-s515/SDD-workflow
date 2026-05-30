<p dir="auto">
  <a href="./README.md"><img src="https://img.shields.io/badge/docs-English-blue" alt="English" /></a>
  <a href="./README.zh-TW.md"><img src="https://img.shields.io/badge/docs-%E7%B9%81%E9%AB%94%E4%B8%AD%E6%96%87-yellow" alt="繁體中文" /></a>
</p>

# SDD Workflow

這是一套以規格驅動開發（Specification-Driven Development, SDD）為核心設計的技能工作流。此 repo 主要說明 `propose`、`apply`、`propose-sync` 之間的依賴流程，以及它們直接呼叫的技能。

它適合這幾種情境：

- 想把自然語言需求整理成可執行的開發任務
- 想用結構化 workflow 約束 AI 實作，讓流程更可控，並持續對齊需求
- 想把規格、實作、測試與完成狀態串成同一條工作流
- document as code 的實踐者，想讓規格文檔不只是說明，而是直接驅動開發

## 這個 Repo 提供什麼

- 一條可重複使用的 `propose -> apply -> propose-sync` 核心流程
- 主流程直接依賴的技能定義
- AC-first、可執行 Feature File、React 設計與 code review 等延伸輔助技能

## Workflow 一覽

```text
需求規格
  ↓
propose
  ├─ clarify-flow -> 01-flow.md
  ├─ export-gherkin -> 02-gherkin.md
  └─ 03-tasks.md
  ↓
apply
  ├─ bdd-unit-test -> [BDD]
  ├─ 實作與測試驗證
  └─ [x][BDD]
  ↓
propose-sync
  ↓
回寫規格文檔的完成狀態
```

## Quick Start

### 1. 準備規格文檔

先準備一份需求文檔，例如：

```md
## 商品折扣功能

使用者可在結帳頁輸入折扣碼，系統需驗證是否有效並更新訂單金額。
未達門檻不可套用，已過期折扣碼需回傳錯誤。
```

### 2. 使用 `propose`

讓 workflow 先把需求拆成可實作的中介文檔。

```text
propose docs/spec.md
```

預期產出：

```text
docs/propose/<feature-name>/
  01-flow.md
  02-gherkin.md
  03-tasks.md
```

### 3. 使用 `apply`

依照任務清單執行 TDD 流程。`apply` 會先替所有一般任務補 BDD 測試並標記 `[BDD]`，再逐項實作、跑測試，通過後更新為 `[x][BDD]`。

```text
apply docs/propose/<feature-name>
```

### 4. 同步完成狀態

使用 `propose-sync` 同步回原始規格文檔。

未被 `propose`、`apply` 或 `propose-sync` 直接呼叫的技能，都屬於延伸輔助技能，不算主流程。

## Repository Structure

```text
skills/
  # 主流程與直接依賴技能
  propose/              核心提案入口
  clarify-flow/         將需求整理成結構化流程
  export-gherkin/       將流程轉成 Gherkin 驗收條件
  apply/                依任務清單執行 TDD 測試與實作
  bdd-unit-test/        產出 BDD 單元測試
  propose-sync/         同步已完成功能回規格文檔

  # 延伸輔助技能
  export-ac/            延伸：先產出 AC 文件
  ac-to-test/           延伸：由 AC 產出紅燈測試骨架
  export-feature-file/  延伸：輸出可執行 .feature
  code-reviewer/        延伸：對照規格與 git diff 執行 code review
  react-design/         延伸：React 設計與 review 原則

docs/
  document.md           技能總覽文件
```

## 核心流程

核心流程以規格文檔為起點，先建立提案與任務，再進入 TDD 實作與驗證，最後同步完成狀態。

只有被 `propose`、`apply` 或 `propose-sync` 直接使用的技能會列入核心流程表。

| 階段            | 技能             | 產出 / 任務                |
| --------------- | ---------------- | -------------------------- |
| 1. 規劃提案     | `propose`        | 識別功能清單並建立三份文檔 |
| 1a. 結構化流程  | `clarify-flow`   | `01-flow.md`               |
| 1b. 驗收條件    | `export-gherkin` | `02-gherkin.md`            |
| 1c. 任務清單    | `propose` 本體   | `03-tasks.md`              |
| 2. TDD 實作     | `apply`          | 先補測試，再依序完成 `03-tasks.md` |
| 2a. 測試生成    | `bdd-unit-test`  | `apply` 內部呼叫，或手動替指定檔案補測試 |
| 3. 同步完成狀態 | `propose-sync`   | 回寫規格文檔的 `## 已完成` |

### 核心流程補充

- `propose` 會回寫規格文檔中的 `> propose:` 標記，供後續 `propose-sync` 掃描使用。
- `propose` 可處理一般功能需求，也可處理 `## bug fix list` 中的 `[quick-fix]`、`[propose]` 與未標記項目。
- `apply` 只會自動執行一般任務；標記為 `[manual]` 的任務會保留給新 session 手動觸發。
- `apply` 使用 `[BDD]` 表示測試已撰寫，使用 `[x][BDD]` 表示測試、實作與驗證完成。
- `propose-sync` 以 `[x][BDD]`、`[x][widget-test]` 或 `[x]` 判斷非 `[manual]` 任務完成，不再依賴 `[x][cr]`。
- 沒有被 `propose`、`apply` 或 `propose-sync` 直接呼叫的技能，一律視為延伸輔助技能。

## 延伸輔助技能

如果你只想理解主流程，可以先跳過這段。以下技能不是 `propose -> apply -> propose-sync` 的依賴，而是圍繞主流程使用的輔助能力。

| 技能                  | 角色                                          | 適用時機                                          |
| --------------------- | --------------------------------------------- | ------------------------------------------------- |
| `export-ac`           | 先從需求整理出 `AC.md` 驗收準則文件           | 需要先定義完成標準，再進入實作                    |
| `ac-to-test`          | 將 `AC.md` 轉成紅燈測試骨架                   | 想採用 AC-first / test-first 流程時               |
| `export-feature-file` | 將規格或 Gherkin 轉成可執行的 `.feature` 檔案 | 需要接入 Reqnroll、Cucumber、Behave 等 BDD 框架時 |
| `code-reviewer`       | 對照規格與 git diff 執行審查並存 report       | 需要獨立 review 階段時                            |
| `react-design`        | 提供 React 架構與最佳實踐檢查原則             | 前端設計討論、React 實作或 React code review 時   |

### 延伸流程範例

#### AC-first 路徑

```text
需求規格
  ↓
export-ac -> AC.md
  ↓
ac-to-test -> 測試骨架
```

#### BDD 執行檔路徑

```text
需求規格 / 01-flow.md
  ↓
export-gherkin -> 02-gherkin.md
  ↓
export-feature-file -> .feature
```

## 最小使用示例

一條典型路徑會像這樣：

1. 你先有一份 `docs/spec.md`
2. 執行 `propose docs/spec.md`
3. 產出 `docs/propose/<feature>/01-flow.md`
4. 產出 `docs/propose/<feature>/02-gherkin.md`
5. 產出 `docs/propose/<feature>/03-tasks.md`
6. 執行 `apply docs/propose/<feature>`
7. `apply` 先補測試標記 `[BDD]`，再實作並更新為 `[x][BDD]`
8. 執行 `propose-sync` 回寫來源規格文檔

若需要 code review、AC-first 測試或可執行 BDD 檔，再獨立使用延伸輔助技能。

## 適用對象

- 想建立 AI 可穩定開發流程的個人開發者
- 想把規格文檔與實作產出綁在一起的團隊
- 想降低「需求理解」與「直接開發」之間落差的開發者/非技術人員

## 相關文件

- 技能總覽文件：[docs/document.md](docs/document.md)
- 技能定義目錄：[skills](skills)
- 授權條款：[LICENSE](LICENSE)
