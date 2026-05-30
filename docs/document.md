# SDD Workflow 技能總覽

本文件對齊 `skills/` 目錄中目前的技能定義，說明各技能的觸發時機、輸入輸出、相依關係與任務狀態標記。

## 核心流程

```text
規格需求文檔
    │
    ▼
propose
    ├─ clarify-flow -> 01-flow.md
    ├─ export-gherkin -> 02-gherkin.md
    └─ 03-tasks.md
            │
            ▼
apply
    ├─ 讀取 01-flow.md / 02-gherkin.md / 03-tasks.md
    ├─ 先用 bdd-unit-test 為一般任務補測試
    ├─ 將任務標記為 [BDD]
    ├─ 逐項實作並跑測試
    └─ 測試通過後標記為 [x][BDD]
            │
            ▼
propose-sync
    └─ 將完成項目回寫到規格文檔的 ## 已完成 區塊
```

## 流程階段對應表

| 階段 | 技能 | 主要產出 / 任務 | 觸發方式 |
| --- | --- | --- | --- |
| 1. 規劃提案 | `propose` | 掃描規格文檔、識別功能與 bug fix，建立 feature folder 與三份文檔 | `propose`、`幫我規劃這個功能`、提供規格文檔路徑 |
| 1a. 結構化流程 | `clarify-flow` | `01-flow.md`，將流程、if/else、遍歷與邊界條件結構化 | `propose` 內部呼叫；或 `clarify flow`、`把這段描述結構化` |
| 1b. 驗收條件 | `export-gherkin` | `02-gherkin.md`，產出人類可讀的 Gherkin 行為規格 | `propose` 內部呼叫；或 `幫我轉成 Gherkin` |
| 1c. 任務清單 | `propose` 本體 | `03-tasks.md`，拆分可實作任務並加入 `[manual]` 測試任務 | `propose` 流程內產出 |
| 2. TDD 實作 | `apply` | 先補 BDD 測試，再逐項實作與驗證，更新任務 checkbox | `apply`、`開始實作`、指定 `docs/propose/<feature>/` |
| 2a. 測試生成 | `bdd-unit-test` | 依檔案或任務產出 BDD 單元測試，涵蓋 Happy / Edge / Error / State Changes | `apply` 內部呼叫；或 `幫這個檔案寫單測` |
| 3. 同步完成狀態 | `propose-sync` | 掃描 `03-tasks.md` 完成狀態，更新來源規格的 `## 已完成` 區塊 | `propose-sync`、`同步完成狀態` |

## 任務狀態標記

`apply` 與 `propose-sync` 對方括號標記皆採大小寫不敏感比對。

| 標記 | 使用位置 | 意義 |
| --- | --- | --- |
| `- [ ] Tx:` | `03-tasks.md` | 一般任務尚未撰寫測試，也尚未實作 |
| `- [BDD] Tx:` | `03-tasks.md` | 測試已撰寫，等待實作 |
| `- [x][BDD] Tx:` | `03-tasks.md` | 非 UI 功能已完成 BDD 測試、實作與驗證 |
| `- [x][widget-test] Tx:` | `03-tasks.md` | UI 功能已完成 Widget Test 驗證 |
| `- [x] Tx:` | `03-tasks.md` | 已完成的實作、文件更新或驗證任務；在 `apply` 中也可視為舊版完成標記 |
| `[manual]` | `03-tasks.md` | 手動任務，不由 `apply` 自動執行，也不影響 `propose-sync` 完成判斷 |

## 核心技能細節

### `propose`

- 讀取規格需求文檔，識別一般功能、`## bug fix list` 與已完成項目。
- 未指定 `frontend/` 或 `backend/` 時，先詢問根路徑；後續所有 feature folder 沿用同一根路徑。
- 已存在於 `## 已完成` 或已有 `> propose:` 標記的功能會跳過，避免重複提案。
- `bug fix list` 依 `[quick-fix]`、`[propose]` 或自動判斷分流；需要提案的 bug 建立 `fix-<slug>/`。
- 確認功能清單後，先回寫 `> propose: ...` 標記，再依序產出 `01-flow.md`、`02-gherkin.md`、`03-tasks.md`。

### `clarify-flow`

- 將鬆散需求改寫為可追蹤的結構化流程。
- 輸出前需檢查空值、邊界數值、資源不存在、狀態衝突、權限、重複操作與外部依賴失敗等缺漏。
- 若需求未涵蓋重要邊界，先詢問使用者；若使用者排除特定情境，需標示為 Out of Scope。

### `export-gherkin`

- 將需求或 `01-flow.md` 轉成 Gherkin 規格，用於人類對齊需求。
- 使用業務語言、聚焦單一行為，避免畫面互動細節。
- 使用變數代表數值，並保持 When 最小化。

### `apply`

- 從使用者提供的 propose folder 自動推斷根路徑，尋找 `01-flow.md`、`02-gherkin.md`、`03-tasks.md`。
- 先進入測試先行階段：所有一般任務都補上 BDD 測試並標記 `[BDD]` 後，才開始實作。
- `02-gherkin.md` 在 apply 過程中為唯讀；若驗收條件矛盾，停止並回報使用者。
- 逐項實作時只修正實作程式碼或與規格不一致的測試，不修改 `02-gherkin.md`。
- 所有一般任務完成後，執行本次功能相關完整測試並確認通過。

### `bdd-unit-test`

- 可被 `apply` 用於任務級 TDD 測試，也可由使用者指定一或多個檔案手動補單元測試。
- 依副檔名載入對應 reference；支援 TypeScript / JavaScript、C#、Java、Python、Flutter/Dart，未知語言則用通用測試慣例推斷。
- 測試範圍聚焦純邏輯、服務層、工具函式、資料處理與狀態管理邏輯，不處理 UI 渲染、視覺回歸、E2E 或整合測試。
- 場景需涵蓋 Happy Path、Edge Cases、Error Cases 與 State Changes。

### `propose-sync`

- 依來源規格中的 `> propose:` 標記推斷掃描根目錄。
- 掃描各 feature folder 的 `03-tasks.md`，只判斷任務行，不判斷一般段落或標題。
- `[manual]` 任務不納入完成判斷。
- 非 `[manual]` 任務全部完成後，將功能寫入規格文檔最上方 `## 已完成` 區塊；既有區塊會完整替換，避免重複累積。

## 延伸技能

| 技能 | 目的 | 主要輸出 |
| --- | --- | --- |
| `export-ac` | 實作前先整理 Acceptance Criteria，明確 In / Out Scope 與測試策略 | `AC-<input-name>.md` |
| `ac-to-test` | 將 `AC.md` 每條 Given / When / Then 轉成紅燈測試骨架 | 對應語言的測試檔 |
| `export-feature-file` | 將規格或 Gherkin 轉成可被 BDD 框架執行的 `.feature` | `.feature` 與 Step Definitions 提示 |
| `code-reviewer` | 對照規格文檔與 git diff 審查程式碼，固定輸出 review report | 獨立 review 階段，不是 `propose -> apply -> propose-sync` 的依賴 |
| `react-design` | 提供 React 組件、hooks、services、context 與註解規範 | React 設計與 review 判斷基準 |

### `code-reviewer`

- 獨立於新版核心流程；`propose`、`apply`、`propose-sync` 不依賴它。
- 用於對照規格文檔與 git diff 審查程式碼，不是 `apply` 技能內建的自動任務標記流程。
- 若 diff 包含 React 前端檔案，先載入 `react-design` 原則審查，再做規格符合度與通用程式碼審查。
- 完成 review 後，需將與對話輸出一致的摘要存到 `docs/code-review-report/code-review-YYYY-MM-DD-<feature-slug>.md`。
- 同一天同一規格重複審查時覆蓋舊檔，但輪數依既有檔案標題遞增。

## 延伸流程範例

### AC-first 路徑

```text
需求規格
  ↓
export-ac -> AC.md
  ↓
ac-to-test -> 紅燈測試骨架
  ↓
propose / apply -> TDD 實作與驗證
```

### 可執行 BDD 路徑

```text
需求規格 / 02-gherkin.md
  ↓
export-feature-file -> .feature
  ↓
依框架撰寫或補齊 Step Definitions
```

### React 審查路徑

```text
React diff
  ↓
code-reviewer
  ├─ react-design 原則
  ├─ 規格符合度
  └─ 通用程式碼審查
```
