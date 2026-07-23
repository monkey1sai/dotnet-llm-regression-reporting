## ADDED Requirements

### Requirement: 效用函式路由與品質硬約束
路由決策 SHALL 以效用函式（品質分數 − 成本/延遲懲罰）選擇模型/管線，品質下限為不可協商硬約束。
路由目標為「品質 SLA 硬約束下的成本受限最佳化」，非成本極小化。

#### Scenario: 候選品質低於門檻
- **WHEN** 某任務分類的候選模型 A 成本最低但品質分數低於門檻
- **THEN** 路由器排除 A，即使其成本更低

#### Scenario: 品質達標時取較低成本
- **WHEN** 候選 A、B 品質皆達門檻且 A 成本較低
- **THEN** 路由器選 A 並記錄效用分數與選中理由

### Requirement: 品質訊號來源、新鮮度與冷啟動預設
即時路由使用的品質分數 SHALL 取自該 `(model, category)` 組合**最近一次 `eval-backend-bridge` 批次 Job 產出**的品質分數；SHALL 定義新鮮度上限（illustrative default，待校準，G2/G3），超過上限 SHALL 標記 `stale` 並寫入 `enterprise-reporting` 路由追蹤欄位；**冷啟動**（該組合無任何歷史批次分數）SHALL 預設路由至**治理指定的保守預設模型（非最低成本）**，並標記 `quality-score-provenance=cold-start`。路由**不得**阻塞等待批次完成。

#### Scenario: 品質分數過期
- **WHEN** 某 `(model, category)` 的最近批次品質分數已超過新鮮度上限
- **THEN** 路由仍可進行但該分數標記 `stale`，並寫入路由追蹤欄位供稽核

#### Scenario: 冷啟動無歷史分數
- **WHEN** 某模型/分類尚無任何批次評測分數
- **THEN** 路由至治理指定保守預設模型，標記 `quality-score-provenance=cold-start`，不路由至最低成本候選

### Requirement: 路由決策可獨立覆核
每個路由決策紀錄 SHALL 使稽核者**獨立於路由演算法本身，僅憑決策紀錄之候選集合與效用分數即可重算選中結果**是否正確且未違反品質下限。

#### Scenario: 稽核者重算選型
- **WHEN** 稽核者僅取用某次路由的候選集合與各候選效用分數
- **THEN** 可重現「選中項為最高效用且未違反品質下限」的結論，無需存取路由演算法內部狀態

### Requirement: 可觀測失敗訊號（不可依賴 HTTP 200）
每個路由決策點 SHALL 產出可觀測失敗訊號，並記錄於 `enterprise-reporting` 路由追蹤欄位；MUST NOT 以 HTTP 200 作為成功的唯一判準。

#### Scenario: 200 但內容空或截斷
- **WHEN** 下游模型回傳 200 但內容為空或截斷
- **THEN** 路由器判定為失敗、觸發 failover，並寫入失敗事件

### Requirement: failover 與每日批次誤路由對帳
路由層 SHALL 具備 failover 與**每日一次**（非即時）誤路由對帳批次工作；對帳結果超過門檻（illustrative default，G2）SHALL 觸發告警工單而非自動重跑生產流量。

#### Scenario: routing-collapse 偵測
- **WHEN** 每日對帳發現同分類路由目的地分布劇烈偏移
- **THEN** 系統產生告警工單但不自動封鎖生產流量

#### Scenario: 下游失敗觸發 failover
- **WHEN** 選中的路由目的地回傳可觀測失敗訊號
- **THEN** 路由層依序試下一候選並記錄 failover 事件

### Requirement: 批次/快取成本優化屬性
路由器 SHALL 將「是否可批次/可快取」列為分類屬性，優先對低風險分類使用批次 API 與 prompt caching。

#### Scenario: 低風險任務累積批次視窗
- **WHEN** 低風險分類任務累積滿批次視窗（如 15 分鐘或 100 筆，illustrative default，G2）
- **THEN** 以批次 API 送出，成本報表顯示相對即時呼叫的節省百分比

### Requirement: 候選前置安全篩查
路由候選（Router-Strategy）SHALL 於候選產生階段通過供應鏈/LLM 應用安全最低篩查（含 OWASP LLM Top10 對映之最低控制點，G10 工程參考）；未通過者不得進入效用函式評比。

#### Scenario: 候選未過安全篩查
- **WHEN** 某候選模型/供應商未通過依賴授權（GPL）或 prompt-injection 最低篩查
- **THEN** 該候選於產生階段即被排除，不進入路由評比

### Requirement: 設定式 circuit breaker 與人工 kill-switch
路由層 SHALL 提供**設定式** circuit breaker 與全域人工 kill-switch（治理參數，非常駐監控服務）；觸發條件為顯性可設定閾值（G3）。

#### Scenario: 下游連續失敗觸發斷路
- **WHEN** 某路由目的地連續失敗數超過設定閾值（illustrative default，G2）
- **THEN** circuit breaker 斷開該目的地並 failover，事件回寫 ontology-ledger

#### Scenario: 人工觸發全域 kill-switch
- **WHEN** 治理人員啟動全域 kill-switch
- **THEN** 路由層停止對受影響分類發送新請求並記錄啟動者與時間
