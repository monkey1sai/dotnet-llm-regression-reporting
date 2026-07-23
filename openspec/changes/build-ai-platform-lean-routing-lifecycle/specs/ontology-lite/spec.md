## ADDED Requirements

### Requirement: 八個一等實體（關聯式 + JSON Schema registry）
系統 SHALL 以既有關聯式 DB（沿用 `enterprise-reporting` 儲存）+ JSON Schema registry 定義八個一等實體：Model、Provider、Route、EvalSuite、RegressionEvent、LifecycleStage/GovernanceGate、Lineage、**Decision/Action**（對映 Palantir「Ontology 代表決策而非資料」）；並提供 JSON/CSV 匯出，MUST NOT 引入圖資料庫。schema、路由規則、權限策略 SHALL 可匯出為廠商中立格式並明訂遷移與退場路徑（反 lock-in）。依 G7 只做單一關聯式具體實作，不建 Ontology-Adapter 抽象層。

#### Scenario: 三實體一次 join 查出
- **WHEN** 查詢 Model、Route、RegressionEvent 三實體關聯
- **THEN** 可用一次 SQL join 查出並匯出為 JSON 供外部稽核工具讀取

#### Scenario: 決策可追蹤匯出
- **WHEN** 稽核者查詢某高風險路由的決策來源
- **THEN** 可自 Decision/Action 實體取得決策紀錄、對抗複核收斂/升級結論與 Lineage

### Requirement: schema 版本治理與遷移
本體 schema 變更 SHALL 有向後相容遷移路徑與明確退場流程；**禁止破壞性變更未經遷移腳本直接上線**。

#### Scenario: 破壞性 schema 變更
- **WHEN** 一個實體欄位以破壞性方式變更且無遷移腳本
- **THEN** 變更被拒絕，要求提供遷移腳本與退場流程

### Requirement: 圖資料庫升級的顯性觸發條件（待校準）
系統 SHALL 明文記錄「暫緩投資圖資料庫」的觸發重評估條件，並標註為「上線初期預設門檻，需於正式量測後校準」而非既定工程常數（G2）；超過條件 SHALL 提案評估遷移。此為 future work 觸發條件，非現在就建 adapter 抽象層（G7）。

#### Scenario: 查詢複雜度超標觸發評估
- **WHEN** 關聯查詢深度超過 3 層 join 且 P95 查詢延遲超過 500ms 連續兩週（illustrative default，首次季度覆核須依實測重新確認）
- **THEN** 觸發「評估圖資料庫」的提案階段任務
