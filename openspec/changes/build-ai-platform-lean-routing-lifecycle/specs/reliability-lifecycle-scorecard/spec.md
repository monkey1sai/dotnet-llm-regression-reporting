## ADDED Requirements

### Requirement: 十指標定義（每項附公式 + 資料來源型態，方法登記於 metric registry）
reliable/innovative/stable SHALL 以十項指標定義，每項在 spec 中**同時附「計算公式」與「資料來源型態」**，並依 G5 層1 登記於 metric registry（G4/G5）；缺任一者不得列入錦標賽評分。metric registry SHALL 為本 capability 內的**版本化設定表**（沿用既有關聯式儲存，非新增常駐 service）；引用未登記方法之量測，系統 SHALL 拒絕該次評分並視為無效。十指標（成本維運優先加權）：
1. **每次評測成本 USD**：公式＝(模型 API 費用 + 批次/快取節省後淨額 + 依 evaluation 執行次數攤提之運算費) ÷ 該次評測筆數；來源＝雲端帳單 API 匯出 + OTel span cost 屬性。
2. **每月維運人力工時（人月）**：公式＝(事故回應工時 + 閘門人工複核工時 + 依賴/版本升級工時 + 高風險常駐評測維運工時 + 路由刷新批次維運工時) 換算人月；來源＝工單系統時間戳記 + on-call 排班紀錄。
3. **MTTD 平均偵測回歸時間**：公式＝Σ(偵測時間 − 引入時間)/事件數；來源＝RegressionEvent 時間戳記 + OTel。
4. **MTTR 平均恢復時間**：公式＝Σ(恢復時間 − 偵測時間)/事件數；來源＝工單 + 回滾紀錄。
5. **誤路由/誤判率（含 Regression Escape Rate）**：公式＝(逃逸至生產的回歸數 + 誤路由數)/總決策數；來源＝每日對帳 + RegressionEvent。
6. **閘門延遲 P95（PR 到判定）**：公式＝P95(判定時間 − PR 觸發時間)；來源＝CI 產出 + lifecycle-gates 事件。
7. **供應商鎖定風險（可替換性評分）**：公式＝依替代候選數與遷移成本計分（越低鎖定分越高）；來源＝ontology-lite Provider/Route + 依賴清單。
8. **新能力上線前置時間（提案→穩定化）**：公式＝穩定化時間 − 提案時間；來源＝LifecycleStage 時間戳記。
9. **基礎設施元件數量（新增常駐 service 數）**：公式＝本 change 新增常駐 service 計數（越少越好；含高風險常駐評測單元）；來源＝部署清單/AI-BOM。
10. **事故成本影響（近 90 天，含 Blast Radius）**：公式＝Σ(事故影響金額)；Blast Radius＝受影響路由/分類佔比；來源＝事故報告 + 帳單。
- **加權**：指標 1、2、9 合計 40%（成本維運優先關鍵裁決），其餘 60% 平均分配。**tie-break**：以指標 1（每次評測成本）為準。

#### Scenario: 依公式判定勝出方
- **WHEN** 候選 A、B 十指標分數相近，A 每次評測成本較低但每月維運工時高 30%、新增 service 數多 3 個
- **THEN** 在 40% 權重下 A 總分落後，判定 B 勝出

#### Scenario: 缺公式或方法未登記不計分
- **WHEN** 某指標未同時附計算公式與資料來源型態，或引用未登記於 metric registry 的量測方法
- **THEN** 該指標不計入評分，該次評分若已引用則視為無效

### Requirement: 回收期（payback period）評分框架與自我一致性
錦標賽評分 SHALL 採**回收期框架**而非靜態單點快照：SHALL 明訂計算累積工時節省/NPV 的時間窗（illustrative default，如 6–12 個月，G2/G3）；SHALL **明文承認任何新增能力的 change（含本 change 自己）於指標 9 上線初期必然劣勢**，並以回收期而非單點分數判定，以避免十指標對所有提案系統性不利。

#### Scenario: 本 change 對純手動拼接的回收期推演
- **WHEN** 以十指標比較「本 change（新增路由/對帳/生命週期治理）」vs「純沿用既有工具手動拼接」兩候選
- **THEN** 單點快照下本 change 於指標 9（新增 service）與短期指標 2 落後，但在時間窗內累積的維運工時節省使回收期為正，據回收期判定本 change 勝出；t0 結論標記 `estimated-pre-operational`

### Requirement: t0 自舉——估計值評分與強制校準
在平台上線前（t0），錦標賽 SHALL 允許以估計值/代理來源（供應商定價頁、業界 benchmark、spike 量測）計算指標**數值**，但每個估計數值 SHALL 標記 `estimated-pre-operational`；累積真實營運資料達門檻（illustrative default，如 N 週，G2）後 SHALL 強制重新評分覆蓋估計值。t0 錦標賽結論 SHALL 標記為「初始估計裁決，待校準」，非終局定案。

#### Scenario: t0 以估計值跑錦標賽
- **WHEN** 平台尚無雲端帳單/on-call/事故等營運資料，需在 t0 選定初始平台設計
- **THEN** 以標記 `estimated-pre-operational` 的估計值跑十指標錦標賽並產出「初始估計裁決」，且方法已登記於 metric registry（層1 有效），不因層2 為估計而視為無效

#### Scenario: 營運資料達門檻強制覆蓋
- **WHEN** 累積真實營運資料達門檻（如 N 週）
- **THEN** 系統以真實值強制重新評分，覆蓋 t0 估計裁決

### Requirement: 校準翻盤後的平台級後果
若強制重新評分後校準翻盤（回收期轉負，或原落敗的純手動拼接候選反勝），系統 SHALL 觸發平台級退場/回滾**提案**（對映 design.md 反 lock-in 承諾），MUST NOT 令 t0 決策事實上不可逆而使校準淪為留痕劇場；翻盤判定本身 SHALL 走多 agent 對抗複核並於 `ontology-lite` Decision 實體留痕。

#### Scenario: 校準後回收期轉負
- **WHEN** N 週真實資料使本 change 的回收期轉負或手動拼接候選反勝
- **THEN** 系統建立平台級退場/回滾提案，該翻盤判定經對抗複核並寫入 Decision

### Requirement: 改版偵測假陽性率監控與假陽性預算
scorecard SHALL 監控供應商改版偵測（無標頭統計檢定法）的**假陽性率**並設定**月假陽性預算**（illustrative default，G2/G3），不得只監控假陰性率；超過月假陽性預算 SHALL 觸發門檻校準覆核。

#### Scenario: 假陽性超預算觸發校準
- **WHEN** 某月統計檢定法的誤觸發（供應商未改版卻判定改版）次數超過月假陽性預算
- **THEN** 系統觸發統計顯著門檻/連續 N 天參數的校準覆核

### Requirement: generate-then-filter 錦標賽候選下限
錦標賽 SHALL 產生至少三個彼此有實質差異的候選（G6），依十指標評分並記錄評分軌跡於 ontology-ledger。

#### Scenario: 候選不足三個
- **WHEN** 錦標賽候選少於三個或候選間無實質差異
- **THEN** 拒絕進入評分，要求補足具實質差異之候選
