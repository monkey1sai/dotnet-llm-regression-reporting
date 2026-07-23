## ADDED Requirements

> 對既有 `performance-measurement` 的 additive 擴充（以 ADDED 新增可選欄位需求，非重寫既有 requirement，故不用 MODIFIED，避免憑空 Modified）。相依 Phase 0 後與 base 並存。

### Requirement: 成本可觀測欄位
系統 SHALL 新增可選欄位 cost-per-eval 與 cache-hit-rate，供 `category-router` 效用函式使用；cost-per-eval 的計算公式與資料來源型態 SHALL 與 `reliability-lifecycle-scorecard` 指標 1 一致（雲端帳單 API 匯出 + OTel span cost 屬性，G4）。可選欄位缺漏時 SHALL 標記為 unavailable 而非以零值誤導。

#### Scenario: 記錄每次評測成本
- **WHEN** 一次評測完成且帳單/OTel cost 屬性可得
- **THEN** 報表記錄 cost-per-eval 與 cache-hit-rate

#### Scenario: 成本來源不可得
- **WHEN** 帳單或 OTel cost 屬性在該次評測不可得
- **THEN** cost-per-eval 標記為 unavailable，不以零值計入路由效用函式
