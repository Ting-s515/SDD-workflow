<p dir="auto">
  <a href="./README.md"><img src="https://img.shields.io/badge/docs-English-blue" alt="English" /></a>
  <a href="./README.zh-TW.md"><img src="https://img.shields.io/badge/docs-%E7%B9%81%E9%AB%94%E4%B8%AD%E6%96%87-yellow" alt="繁體中文" /></a>
</p>

# SDD Workflow

這是一套以規格驅動開發（Specification-Driven Development, SDD）為核心設計的技能工作流。此 repo 用來說明如何將需求文檔逐步轉成結構化流程、驗收條件、任務清單、實作、審查與完成狀態同步。

它適合這幾種情境：

- 想把自然語言需求整理成可執行的開發任務
- 想讓 AI 實作流程更可追蹤，而不是直接跳進寫碼
- 想把規格、實作、review、測試與完成狀態串成同一條工作流

## 這個 Repo 提供什麼

- 一條可重複使用的 SDD 核心流程
- 一組對應流程節點的技能定義
- 可選接入的 AC-first、Feature File 與 React 審查延伸能力

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
  ├─ 實作任務
  └─ code-reviewer
  ↓
bdd-unit-test (optional, manual)
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

依照任務清單逐步實作。

```text
apply docs/propose/<feature-name>
```

### 4. 補測試或同步完成狀態

- 實作後補單元測試：`bdd-unit-test`
- 同步回原始規格文檔：`propose-sync`

## Repository Structure

```text
skills/
  propose/              核心提案入口
  clarify-flow/         將需求整理成結構化流程
  export-gherkin/       將流程轉成 Gherkin 驗收條件
  apply/                依任務清單逐步實作
  code-reviewer/        對照規格執行 code review
  bdd-unit-test/        實作後補單元測試
  propose-sync/         同步已完成功能回規格文檔

  export-ac/            延伸：先產出 AC 文件
  ac-to-test/           延伸：由 AC 產出紅燈測試骨架
  export-feature-file/  延伸：輸出可執行 .feature
  react-design/         延伸：React 設計與 review 原則

docs/
  document.md           技能總覽文件
```

## 核心流程

核心流程以規格文檔為起點，先建立提案與任務，再進入實作與審查，最後同步完成狀態。

| 階段            | 技能             | 產出 / 任務                |
| --------------- | ---------------- | -------------------------- |
| 1. 規劃提案     | `propose`        | 識別功能清單並建立三份文檔 |
| 1a. 結構化流程  | `clarify-flow`   | `01-flow.md`               |
| 1b. 驗收條件    | `export-gherkin` | `02-gherkin.md`            |
| 1c. 任務清單    | `propose` 本體   | `03-tasks.md`              |
| 2. 實作任務     | `apply`          | 依序完成 `03-tasks.md`     |
| 2a. Code Review | `code-reviewer`  | 統一審查並更新 `[x][cr]`   |
| 3. 補上單元測試 | `bdd-unit-test`  | 產出實作後測試             |
| 4. 同步完成狀態 | `propose-sync`   | 回寫規格文檔的 `## 已完成` |

### 核心流程補充

- `propose` 會回寫規格文檔中的 `> propose:` 標記，供後續 `propose-sync` 掃描使用。
- `propose` 可處理一般功能需求，也可處理 `bug fix list` 中需進一步提案的項目。
- `apply` 只會自動執行一般任務；標記為 `[manual]` 的任務會保留給新 session 手動觸發。
- `code-reviewer` 屬於核心流程的一部分，不是額外附加步驟。

## 延伸技能

如果你只想理解主流程，可以先跳過這段。以下技能不是必經步驟，但能補強驗收與測試策略。

| 技能                  | 角色                                          | 適用時機                                          |
| --------------------- | --------------------------------------------- | ------------------------------------------------- |
| `export-ac`           | 先從需求整理出 `AC.md` 驗收準則文件           | 需要先定義完成標準，再進入實作                    |
| `ac-to-test`          | 將 `AC.md` 轉成紅燈測試骨架                   | 想採用 AC-first / test-first 流程時               |
| `export-feature-file` | 將規格或 Gherkin 轉成可執行的 `.feature` 檔案 | 需要接入 Reqnroll、Cucumber、Behave 等 BDD 框架時 |
| `react-design`        | 提供 React 架構與最佳實踐檢查原則             | 前端設計討論或 React code review 時               |

### 延伸流程範例

#### AC-first 路徑

```text
需求規格
  ↓
export-ac -> AC.md
  ↓
ac-to-test -> 測試骨架
  ↓
propose / apply -> 實作與驗證
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
7. 實作完成後由 `code-reviewer` 統一審查
8. 視需要執行 `bdd-unit-test` 或 `propose-sync`

## 適用對象

- 想建立 AI 可重複執行開發流程的個人開發者
- 想把規格文檔與實作產出綁在一起的團隊
- 想降低「需求理解」與「直接開發」之間落差的人

## 文件範圍

本 repo 主要說明 `SDD Workflow` 的核心流程與直接相關的延伸技能，不包含通用輔助型技能，例如搜尋 skill、回退變更、刪除特殊檔案或匯出對話等工具型能力。

## 相關文件

- 技能總覽文件：[docs/document.md](docs/document.md)
- 技能定義目錄：[skills](skills)
- 授權條款：[LICENSE](LICENSE)
