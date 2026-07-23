## ADDED Requirements

> 本 capability **不含 `## MODIFIED Requirements`**：`lifecycle-gates` 為新 capability，不得跨資料夾 `MODIFY` 屬於 `baseline-policy-gates` 的 requirement。四態判定改以下列 `ADDED` requirement **重新陳述**，並**明文承認與 `baseline-policy-gates` 原文重複**；`baseline-policy-gates` 原檔維持原樣。

### Requirement: 四態判定重述為 validate 階段規則（明文承認與 baseline-policy-gates 重複）
`lifecycle-gates` 的「validate 階段」SHALL **重述**既有 `baseline-policy-gates` 的 pass/warn/fail/inconclusive 四態分類邏輯（本需求明文承認此為與 `baseline-policy-gates/spec.md` 重複之陳述，非跨資料夾 MODIFIED）；並新增「風險分層開關」欄位決定是否疊加重閘門。四態須與既有 `regression-execution` 的 `Passed`/`Warning`/`Failed`/`Inconclusive` 狀態語意對齊，且區分 regression、infrastructure failure、skipped、inconclusive。

#### Scenario: 低風險僅跑四態判定
- **WHEN** 低風險分類的 regression 事件
- **THEN** 僅跑四態判定，不疊加重閘門

#### Scenario: 高風險疊加血緣要求
- **WHEN** 高風險分類的 regression 事件
- **THEN** 額外要求血緣附件存在才可判定 pass

### Requirement: 風險分類治理（產出主體、fail-closed 預設、變更受治理）
高/低風險分類 SHALL 為受治理、可稽核之決策：分類由治理指派或由 `ontology-lite` 的 Route/EvalSuite 屬性依明訂 rubric 推導，MUST NOT 為純自我宣告且不留痕；**未分類或分類缺失時 SHALL fail-closed 預設為高風險**（走全套重閘門），MUST NOT fail-open 靜默走輕量閘門；風險分類的變更本身 SHALL 為一次寫入 `ontology-lite` Decision 實體、可被多 agent 對抗複核的受治理決策。

#### Scenario: 任務未分類
- **WHEN** 一個進入 validate 的任務尚未被指派風險分類
- **THEN** 系統 fail-closed 視其為高風險，掛載重閘門，並要求補齊分類與依據

#### Scenario: 風險分類被下調
- **WHEN** 治理人員將某分類自高風險下調為低風險
- **THEN** 該變更寫入 Decision 實體、觸發對抗複核，未經複核不生效

### Requirement: 五階段生命週期與輕量預設
五階段（提案/實驗/驗證/穩定化/治理-汰換）SHALL 各定義進場/退場條件與必要 artifact 清單，且輕量閘門（<5 分鐘、三指標）為預設值（數值為 illustrative default，G2）。五階段對映 NIST AI RMF（Govern/Map/Measure/Manage）與 EU AI Act「重大變更觸發風險等級重評估」（G10 工程參考）。

#### Scenario: 實驗進驗證只需輕量
- **WHEN** 一個新 prompt 版本從「實驗」進入「驗證」
- **THEN** 僅需三指標輕量評測與變更說明，不需血緣文件

#### Scenario: 穩定化進治理需重閘門
- **WHEN** 同一版本從「穩定化」進「治理」
- **THEN** 需 AI-BOM 與血緣附件

### Requirement: 多 agent 對抗式決策複核（含仲裁者與死結預設）
於「驗證→穩定化→治理」階段與高風險分類，系統 SHALL 指派 ≥2 個 agent 以對抗式交叉問答複核決策。收斂與否 SHALL 由一個**指定的、不參與對抗的第三方仲裁角色**依**結構化 checklist**（本輪是否新增反駁證據/新假設/新方法，三項皆布林值）判定，判定依據 SHALL 寫入 `ontology-lite` Decision 實體，供稽核者以同一 checklist 重現。同一問題最多兩輪；**兩輪仍未收斂 SHALL fail-closed**（block 並自動升級人工治理委員會，不得靜默放行）；人工事後覆寫 SHALL 於 Decision 留審批紀錄。

#### Scenario: 高風險決策觸發對抗複核並收斂
- **WHEN** 高風險分類的能力進入「驗證」階段且仲裁者 checklist 判定無新反駁/假設/方法
- **THEN** 判定收斂，定案與三布林值判據寫入 Decision/Lineage

#### Scenario: 兩輪未收斂 fail-closed
- **WHEN** 同一決策點經兩輪對抗仲裁者仍判定有新反駁或僵持
- **THEN** 閘門 block 並自動升級人工治理委員會，不得放行；升級紀錄寫入 Decision

#### Scenario: 低風險不觸發
- **WHEN** 低風險分類的變更進入「驗證」
- **THEN** 不啟動對抗複核，僅跑四態判定（成本裁決）

### Requirement: 供應商靜默改版偵測（統計檢定，非精確雜湊）
外部供應商模型/API 的版本變更 SHALL 被偵測並自動觸發該分類回歸重跑（視為生命週期「重大變更」）。偵測 SHALL 分層：有穩定 model-version 標頭者以標頭指紋比對；**無標頭者以錨定 prompt 集之輸出「語意嵌入餘弦相似度分布」或「輸出長度分布」之統計檢定（管制圖/CUSUM）計算日對日變化量，須超過統計顯著門檻（illustrative default，待校準，G2/G3）且連續 N 天皆超過才判定改版**（取代「雜湊精確比對連兩次不同即觸發」，因 LLM 輸出即使 temperature=0 亦非決定性，精確比對會保證每日誤報）。代理法之**假陽性率與假陰性率**皆 SHALL 納入 `reliability-lifecycle-scorecard` 監控並定期覆核。

#### Scenario: 有版本標頭
- **WHEN** 供應商回傳的 model-version header 與上次記錄不同
- **THEN** 系統自動建立回歸任務並將該路由標記為「待驗證」

#### Scenario: 無版本標頭（統計檢定）
- **WHEN** 供應商 API 不回傳版本資訊，且錨定 prompt 集的日對日語意嵌入/長度分布變化量連續 N 天超過統計顯著門檻
- **THEN** 系統建立回歸任務並標記「待驗證」；單日抖動未達門檻或未連續 N 天則不觸發（避免非決定性誤報）

### Requirement: 高風險閘門的可回滾單元與 RTO
每個高風險閘門判定 SHALL 定義「可回滾單元」與「最大回滾時限（RTO）」，且回滾流程 SHALL 可獨立演練驗證（RTO 數值為 illustrative default，G2/G3）。

#### Scenario: 高風險判定缺回滾單元
- **WHEN** 高風險閘門判定未附可回滾單元定義
- **THEN** 判定不得為 pass，並要求補齊回滾單元與 RTO

### Requirement: 供應鏈與安全重閘門（比例式）
於「驗證→穩定化→治理」與高風險分類，SHALL 掛載 OWASP LLM Top10 對映控制點、AI-BOM、SLSA build L2+、WCAG AA、依賴授權（禁 GPL）掃描；低風險/提案-實驗階段不掛載。所有標準引用標註「工程參考，非法律結論」（G10）。

#### Scenario: 治理階段掛重閘門
- **WHEN** 高風險能力進入「治理」階段
- **THEN** AI-BOM/SLSA/依賴授權掃描為必要 artifact，缺一不得放行

#### Scenario: 提案階段不掛重閘門
- **WHEN** 低風險能力仍在「提案」階段
- **THEN** 不要求 AI-BOM/SLSA，僅輕量三指標評測
