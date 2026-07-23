## ADDED Requirements

### Requirement: 無狀態批次執行單元，用完即釋放
Python 評測工具（DeepEval/Promptfoo 雙軌）SHALL 以**無狀態、用完即釋放的批次執行單元**運行，執行完畢即釋放運算資源、不占用常駐運算（部署選型如 K8s CronJob 等為 design.md illustrative，非本 requirement 規範）。

#### Scenario: 回歸批次觸發評測
- **WHEN** 一次回歸批次觸發 DeepEval 評測工作
- **THEN** 該工作於預算窗口內完成並釋放運算資源，完成後不占用常駐運算（如 10 分鐘，illustrative default，G2）

### Requirement: 高風險分類的即時評測例外
高風險分類的評測工作單元 SHALL **例外於批次限制**，以**低延遲、隨時可用的執行模式**（常駐評測單元或同步呼叫）運行，以免延誤高風險偵測時效。此例外**明文打破「用完即釋放、零常駐」成本主軸**，其較高常駐成本 SHALL 計入 `reliability-lifecycle-scorecard` 指標 2/9。

#### Scenario: 高風險任務需即時評測
- **WHEN** 高風險分類的任務需要評測分數以進行即時路由或閘門判定
- **THEN** 以低延遲、隨時可用的執行模式取得分數，不排入批次佇列

#### Scenario: 低風險任務走批次
- **WHEN** 低風險分類的任務需要評測
- **THEN** 排入無狀態批次執行單元，完成後釋放運算資源

### Requirement: 路由新鮮度刷新批次（治理式、成本揭露）
系統 SHALL 提供**治理式、可設定頻率的排程刷新批次**，為穩定生產、近期無回歸事件的 `(model, category)` 組合重新產生品質分數，避免路由分數單調老化為 stale 而退化成準冷啟動；此刷新批次仍為無狀態批次執行單元（非常駐），其固定運算成本 SHALL 誠實揭露並計入 `reliability-lifecycle-scorecard` 指標 1/2。若治理選擇停用刷新，stale 分數 SHALL 於 `enterprise-reporting` 明示為預期行為。

#### Scenario: 穩定分類分數老化觸發刷新
- **WHEN** 某 `(model, category)` 無回歸事件且其品質分數接近新鮮度上限，且刷新批次為啟用狀態
- **THEN** 排程刷新批次重新產生該組合品質分數，成本計入指標 1/2

#### Scenario: 刷新停用時明示 stale
- **WHEN** 治理停用刷新批次且某組合分數已 stale
- **THEN** `enterprise-reporting` 明示該路由以 stale 分數運作為預期行為，非缺陷

### Requirement: 版本化契約與相容性回歸
控制平面（.NET）與資料平面（Python）介面 SHALL 以 HTTP/gRPC + 版本化 schema 固定；Python 側版本升級不得破壞既有契約而不觸發相容性回歸。破壞性變更 SHALL 附遷移路徑（SemVer major，G7）。

#### Scenario: DeepEval 升版破壞欄位
- **WHEN** DeepEval 從 vX 升到 vX+1 且回傳欄位變更
- **THEN** 契約測試套件於合併前失敗，並要求提供遷移路徑
