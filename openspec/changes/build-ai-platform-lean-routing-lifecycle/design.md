## Context

本 change 在既有 `design-enterprise-dotnet-regression-reporting`（五 capability）之上向上擴展為完整 AI Platform（AIP）。核心假說：**凡能用既有 OSS/雲端服務達成的執行層，一律不重造**；AIP 只自建「治理契約 + 資料整合」。裁決主軸為**成本維運優先**——新元件的存在，舉證責任在於證明自建的邊際維運成本 < 沿用既有 OSS/雲端服務。

研究基礎（confirmed，供本文件引用依據）：
- 市場缺口：promptfoo/DeepEval/Ragas 缺生產監控、集中式實驗追蹤、協作/治理介面。
- Palantir 對標：Ontology 被設計為「代表企業**決策**而非資料」；Foundry/Ontology 為閉源，須以 schema registry + policy engine 自建可匯出、模型無關、開放 schema 之等價物，設計非目標＝不複製鎖定護城河。
- 技術基座：編排採 Microsoft Agent Framework（Semantic Kernel v1.71.0 + AutoGen，Process Framework Q2 2026 GA）；分類路由參考 LiteLLM；.NET 原生評測用 Microsoft.Extensions.AI.Evaluation 10.8.0。控制平面（.NET 編排/路由/報表/治理）／資料平面（容器化 Python 評測 DeepEval+Promptfoo 雙軌，經 HTTP/gRPC + LiteLLM 介接）分工，不用 C# 重寫指標庫。可觀測性映射 OTel GenAI 語意慣例（OTel 2026-05-21 CNCF 畢業，惟 GenAI 欄位多仍 experimental）→ 以 adapter/反腐層 + 契約版本鎖定。Promptfoo 於 2026-03-09 被 OpenAI 收購（維持開源）。
- 跨平台驅動：.NET Aspire 官方定位為本地/dev 編排、非生產編排器；生產須 publish 至容器；CI Linux-first（GitHub Actions Linux $0.006、Windows $0.010＝2x、macOS $0.062＝10x/min，confirmed）。
- 生命週期治理標準：NIST AI RMF（Govern/Map/Measure/Manage）+ GenAI Profile、ISO/IEC 42001:2023、EU AI Act Art.53「重大變更觸發風險等級重評估」；OWASP LLM Top10 2025 約束路由選型與供應鏈；依賴授權僅 MIT/Apache、禁 GPL（.NET Foundation 政策）。

## 全域格式規則（GLOBAL — 適用本 change 所有 delta 檔）

- **G1（Scenario 文法）**：每條 Requirement 情境採 `#### Scenario:` + `- **WHEN** … / - **THEN** …` 兩行式，不得散文。
- **G2（示意值揭露）**：任何**非研究簡報 confirmed** 的具體數值門檻（canary 上限、偵測窗口、RTO、ontology-lite「3 層 join + P95>500ms×2 週」、批次視窗、統計顯著門檻、假陽性預算、回收期時間窗、拆分倍數等）一律標註「**illustrative default — 可由治理設定覆寫，非已驗證數值，須排入校準排程**」。已 confirmed 的公開數值（GitHub Actions CI 費率）不加此標註。
- **G3（數值即治理參數）**：所有數值型閾值必須是**可設定的治理參數**，不得隱藏於程式碼常數；spec 需標明設定位置與預設值。
- **G4（度量可稽核性）**：任何量測/評分類需求引用之指標，**必須同時附「計算公式」與「具體資料來源（型態）」**（OTel span 屬性、雲端帳單 API 匯出、工單/on-call 時間戳記）；缺任一者不得列入評分或驗收。
- **G5（指標登記兩層制）**：
  - **層1（方法登記，硬約束）**：所有評分指標的量測「方法」須登錄於**可版本化、可稽核的 metric registry**，附「計算公式」與「資料來源型態」。任何評分若引用「未登記方法」的量測，系統必須拒絕該次評分並視為無效。
  - **層2（數值信賴，允許自舉）**：指標「數值」在平台上線前（t0）**允許使用估計值/代理來源**（供應商定價頁、業界 benchmark、spike 量測），但**強制標記 `estimated-pre-operational`**；累積真實營運資料達門檻（illustrative default，如 N 週，G2）後**強制重新評分覆蓋估計值**。
- **G6（候選數量下限）**：凡採 generate-then-filter 者，候選池**至少三個且彼此有實質差異**；不得用「若干個」等不可驗證量詞。
- **G7（抽象化守門）**：任何新增抽象層/契約/擴充點，提出時**必須已知至少兩個具體實作候選**；否則不建契約，只做單一具體實作，待第二個實作出現再抽象化。
- **G8（能力更名可追溯 — 純人讀，非工具遷移）**：任何 capability 更名/收斂，必須提供「舊 id → 新 id 對照表」與逐條「保留/擴充/棄用」標註。此對照表**僅為人讀可追溯文件，不是 `openspec validate` 解析執行的 machine-readable 遷移指令**。跨資料夾的 requirement 收斂一律以「新 capability 內 `ADDED` 重述（明文承認重複）」達成，**不得對他 capability 資料夾的 requirement 下 `MODIFIED`**。
- **G9（範疇可稽核）**：每個排除項逐一列於 scope_out 並在 tasks.md 對應標註 future work + 排除理由。任何 requirement 若找不到對應使用者旅程節點，視為 scope 蔓延並於本輪排除。
- **G10（合規標註）**：所有 NIST AI RMF / ISO/IEC 42001 / EU AI Act / OWASP / WCAG 引用一律標註「**工程參考，非法律結論；法遵定案留待法務複核**」。
- **G11（複雜度總帳 — 第二道客觀複雜度量）**：除「≤5 新 capability 硬上限」外，本 change 於下方附錄發布**複雜度總帳**，記錄每個 capability 的 `ADDED`+`MODIFIED` Requirement 數量與「可獨立部署/測試單元」數；當任一 capability 的 Requirement 數超過其餘四者平均值的既定倍數（illustrative default，見附錄）時，SHALL 觸發「拆分評估」提案。

## 裁決原則
- **舉證倒置**：新元件預設「不建」，除非證明自建維運成本 < 沿用 OSS。
- **抽象化守門（G7）**：擴充點提出前須有 ≥2 具體實作候選，否則只做具體實作。
- **比例式治理**：重閘門只掛「驗證→穩定化→治理」階段與高風險分類。

## config.yaml 與部署禁令邊界聲明
- **本 change 不變更 `config.yaml` 的 proposal 部署禁令**（"Keep deployment, MCP, browser automation, and credential changes out of this repository"）。所有 spec 內執行邊界一律以**部署無關的可觀測行為**敘述；K8s CronJob / 容器 / Azure Container Apps / 容器 secret 掛載等**具體部署選型僅為本節 illustrative 實作建議**，不進入 `specs/` 的 normative requirement。
- 本 change **唯一觸碰**的 `config.yaml` 編輯為 context 兩處 `Windows-first`→cross-platform/OS-agnostic，及 design 規則 `Windows portability`→`cross-platform portability (Windows/Linux/macOS parity)`（tasks 0.1）。此為對「所有」未來 change 生效的全域規則層編輯，故限最小外科手術、逐字界定，落在 `openspec/changes/` 之外。

## G8 對照表性質聲明
「舊 id→新 id 對照表」**僅為人讀可追溯文件**，OpenSpec 無原生 rename、validate 不會據此做遷移；跨資料夾搬移 requirement 在 validate 眼中就是 `REMOVED`+`ADDED`。本 change 因此**不對 `baseline-policy-gates` 下 `MODIFIED`**，四態判定改由 `lifecycle-gates` 以 `ADDED` 重述並明文承認重複。

## 前置相依與 validate 語意（時序）
`openspec validate --strict` 已證實只檢查 delta **結構**、不檢查 `MODIFIED` 的 base 是否存在（base 存在性為 `openspec archive` 階段檢查）。故本 change 現階段即可通過 strict；但 `MODIFIED`/四態重述的**語意生效**相依 Phase 0（design-enterprise archive 進 `openspec/specs/`）。Phase 0 由 tasks.md 0.0 顯性阻塞，未完成前所有 Modified/收斂宣稱以條件式語氣閱讀。

## 擴充點與 SemVer 治理（G7 修正後）

依 R2Q3 收斂裁決，**移除 Ontology-Adapter 抽象層**：`ontology-lite` 此刻只有單一關聯式具體實作候選，依 G7 不建 adapter 契約；圖資料庫僅為 future-work 觸發條件，待第二實作真出現再抽象化。修正後四類擴充點各滿足 G7（≥2 confirmed 候選）：

| 擴充點類別 | Confirmed 實作候選（≥2 滿足 G7） | 契約治理 |
|---|---|---|
| Provider（供應商端點） | 官方 OpenAI .NET SDK、protocol-compatible fallback | 既有 `endpoint-profiles` |
| Router-Strategy | LiteLLM（薄封裝）＋ 自訂效用函式策略 | `category-router` |
| Evaluator-Backend | DeepEval、Promptfoo（雙軌）；.NET 側 Microsoft.Extensions.AI.Evaluation 10.8.0 | `eval-backend-bridge` |
| Lifecycle-Gate | 輕量三指標閘門、重量閘門（血緣/AI-BOM/SLSA/WCAG） | `lifecycle-gates` |
| ~~Ontology-Adapter~~（移除） | 只有關聯式 DB 一個候選 → 依 G7 不建抽象層，只做具體關聯式實作 | `ontology-lite`（單一具體實作） |

**SemVer 規則**：擴充點契約以版本化 schema（HTTP/gRPC）固定；**破壞性變更（major）須有向後相容遷移路徑與明確退場流程，禁止未經遷移腳本直接上線**；minor/patch 不得破壞既有契約而不觸發相容性回歸。OTel GenAI 欄位多仍 experimental → 以 adapter/反腐層 + 契約版本鎖定，內部維持穩定自有語義。

## 風險分類治理（R2Q1，折入 `lifecycle-gates`）
比例式治理全掛在「高風險分類」開關上，故該開關本身必須是受治理、可稽核、有 fail-closed 預設的一等決策：
- **產出主體**：分類為受治理決策——由治理指派，或由 `ontology-lite` 的 Route/EvalSuite 屬性依明訂 rubric 推導；**不接受純自我宣告不留痕**。
- **預設值（fail-closed）**：未分類/分類缺失時**預設高風險**（走全套重閘門），與全份 fail-closed 哲學一致；不得 fail-open 靜默走輕量閘門。
- **變更治理**：風險分類的變更本身是一次需寫入 `ontology-lite` Decision 實體、可被多 agent 對抗複核的受治理決策。
- 殘留：分類 rubric（哪些訊號→高風險）待首次治理委員會校準（Open Questions R2Q1）。

## 高風險評測基座的常駐成本揭露（誠實揭露）
`eval-backend-bridge` 主軸為「無狀態、用完即釋放、零常駐」以壓成本。但**高風險分類的即時評測需求與該零常駐主軸直接衝突**：高風險偵測不能等批次。裁決為**比例式例外**——高風險分類的評測工作單元例外於批次限制，以**低延遲、隨時可用的執行模式**（常駐評測單元或同步呼叫）運行。**本節誠實承認此打破「用完即釋放、零常駐」成本主軸**，並定位為比例式治理下高風險分類**必然的較高常駐成本**；此常駐成本計入 `reliability-lifecycle-scorecard` 指標 2/9，在回收期框架下與「延誤高風險偵測」的事故成本權衡。

## 路由分數新鮮度維持（R2Q5，折入 `eval-backend-bridge`）
category-router 依「最近一次回歸批次品質分數」，但批次僅由 regression 事件驅動；穩定生產、近期無變更的 `(model,category)` 分數會單調老化至全部 stale。裁決：新增**治理式、可設定頻率的排程刷新批次**（非常駐，仍為批次執行單元），其固定運算成本**誠實揭露**並計入 scorecard 指標 1/2（與高風險常駐評測同樣揭露張力）；若治理選擇停用刷新，則 stale 分數須於 `enterprise-reporting` 明示為預期行為。此為 eval-backend-bridge「零常駐」主軸的第二個誠實例外。

## grill-me 型多 agent 對抗式決策複核（治理程序，折入 `lifecycle-gates`，非新 capability）
直接對應使用者 req(3)。**不新增第 6 個 capability**，改定義為 `lifecycle-gates` 內「驗證→穩定化→治理」階段與高風險分類的**必要治理程序**：
- **觸發**：僅高風險分類、或生命週期進入「驗證/穩定化/治理」時觸發；低風險不觸發（成本裁決）。
- **程序**：指派 ≥2 個 agent 以 grill-me 技能對候選設計/評分結果進行**相互對抗式交叉問答**。
- **收斂判定（決定性、可稽核）**：收斂與否由一個**指定的、不參與對抗的第三方仲裁角色**依**結構化 checklist**（本輪是否新增：反駁證據？新假設？新方法？——三項皆布林值）判定；判定依據（三布林值 + 理由）寫入 `ontology-lite` Decision 實體，供稽核者以同一 checklist 重現。
- **終止與死結預設（fail-closed）**：同一問題最多兩輪對抗；**兩輪跑完仍未收斂 SHALL fail-closed**——閘門 block 並自動升級人工治理委員會，**不得靜默放行**。人工事後覆寫 SHALL 於 Decision 留審批紀錄。

## 供應鏈與安全（折入 `lifecycle-gates` 重閘門，非新 capability）
- **OWASP LLM Top10 2025 逐項對映**：定義可稽核控制點列表；約束路由選型與供應鏈（G10 標註工程參考）。
- **候選前置篩查**：路由候選（Router-Strategy）於**產生階段**即須通過供應鏈/LLM 應用安全最低篩查，把風險攔在候選產生而非事後審查。
- **依賴授權閘門**：僅允許 MIT/Apache，**禁 GPL**（.NET Foundation 政策，confirmed）；依賴授權掃描為合併閘門。
- **輸入淨化/輸出過濾 + agent 最小權限**：掛於高風險分類。
- **AI-BOM / SLSA build L2+ / WCAG AA**：按風險分級掛載，只掛驗證→穩定化→治理階段（比例式）。

## Fail-safe 設計（折入 `category-router` + `lifecycle-gates`，非常駐 incident-containment 服務）
- **circuit breaker + 人工 kill-switch**：作為 `category-router` 的**設定式**能力（低固定成本），非常駐監控服務。
- **canary 流量上限 + 觀察窗口 + 自動凍結/回滾**：作為 `lifecycle-gates` 高風險分類的閘門要求（canary 上限為 illustrative default，G2）。
- **可回滾單元 + RTO**：每個高風險閘門判定必須定義「可回滾單元」與「最大回滾時限」，且回滾流程可獨立演練驗證。
- **事後報告回寫**：事故覆盤寫回 `ontology-lite` 的 Decision/Action + Lineage。

## metric registry 歸屬（R2Q2）
metric registry 定為 `reliability-lifecycle-scorecard` 內的**版本化設定表**，沿用 `enterprise-reporting`/`ontology-lite` 既有關聯式儲存，**非新增常駐 service**——故不違反「新元件預設不建」舉證倒置；其版本化與稽核查詢以既有 DB 的 schema 版本治理達成，其「拒絕未登記方法評分」能力為 scorecard 評分邏輯的一部分，不需獨立部署。殘留：是否需跨 capability 共用 registry 待第二使用者出現再評估（G7）。

## 十指標錦標賽與回收期框架（成本維運優先加權）
指標本身以既有可觀測性資料（OTel、CI 產出、雲端帳單、工時紀錄）計算，不新建量測基礎設施；每項須登記於 metric registry（G5 兩層）並附公式+資料來源型態（G4）。**評分模型由「靜態單點快照」改為「損益平衡/回收期（payback period）框架」**：
- 明訂計算累積工時節省/NPV 的**時間窗**（illustrative default，如 6–12 個月，G2）。
- **明文承認設計偏誤並修正**：任何「新增能力」的 change（**包含本 change 自己**）於指標 9（新增 service 數）上線初期**必然劣勢**，指標 2（維運工時）短期亦不占優；若以單點快照評分，十指標會對「所有新提案」系統性不利，等於自我否決。故以**回收期而非單點分數**判定。
- t0 階段的錦標賽結論（含本 change vs 純手動拼接）依 G5 層2 標記為 `estimated-pre-operational`、「初始估計裁決，待校準」，非終局定案。
- **校準翻盤後果（R2Q6）**：若 N 週後校準使回收期轉負、或原落敗的純手動拼接候選反勝，SHALL 觸發平台級退場/回滾**提案**（對映反 lock-in 承諾），翻盤判定本身走多 agent 對抗複核並於 Decision 留痕。

## 複雜度總帳（G11 附錄）

以本 change 實際 delta 需求數重算（取代草案早期不一致的估計）：

| capability | ADDED Req 數 | MODIFIED Req 數 | 可獨立部署/測試單元（概估） |
|---|---|---|---|
| `category-router` | 8 | 0 | 效用路由 / 品質 provenance / 獨立覆核 / 可觀測失敗 / failover+對帳 / 批次快取 / 前置篩查 / circuit-breaker |
| `eval-backend-bridge` | 4 | 0 | 無狀態批次 / 高風險即時例外 / 版本化契約 / 路由新鮮度刷新批次 |
| `lifecycle-gates` | 7 | 0（四態改 ADDED 重述） | 四態重述 / 五階段 / 對抗複核 / 改版偵測 / RTO / 供應鏈重閘門 / 風險分類治理 |
| `ontology-lite` | 3 | 0 | 八實體 schema / 遷移治理 / 圖 DB 觸發條件 |
| `reliability-lifecycle-scorecard` | 6 | 0 | 十指標 registry / 回收期 / t0 自舉 / 假陽性監控 / 錦標賽候選下限 / 校準翻盤後果 |
| endpoint-profiles（既有，待 Phase 0） | 0 | 2 MODIFIED（Secret references only / Versioned endpoint profile） | 部署無關可攜密鑰 / 跨 OS canonical 雜湊 |
| regression-execution（既有，待 Phase 0） | 2 ADDED | 1 MODIFIED（Deterministic execution controls） | Linux-first CI 矩陣 / 五類位元一致性測試 / 決定性跨 OS 擴充 |
| performance-measurement（既有，待 Phase 0） | 1 ADDED | 0 | 成本可觀測欄位 |
| enterprise-reporting（既有，待 Phase 0） | 1 ADDED | 0 | 路由決策與成本追蹤欄位 |
| baseline-policy-gates（既有） | 0 | 0（四態改由 lifecycle-gates ADDED 重述） | 原檔維持原樣，不被本 change 改動 |

- **拆分倍數**：illustrative default **1.75x**（G2，待校準）。各 capability 對「其餘四者平均」比值：`category-router` 8 vs 平均 (4+7+3+6)/4=5.0 → **1.60x**（最高，未達 1.75x）；`lifecycle-gates` 7 vs (8+4+3+6)/4=5.25 → 1.33x；其餘更低。**現況無 capability 越過 1.75x 拆分線**，故不強制拆分；但規則成立且 `category-router` 為最接近之離群點，**tasks 7.2 強制交付一次拆分評估**（非僅納入驗收）以在真實需求數穩定後重檢。此第二道量納入驗收準則，證明防蔓延不靠 capability 計數玩弄。

## 風險與緩解
- **前置 archive 未完成 → 本 change delta 語意無法生效**（strict 仍可過，但 Modified/四態不套 base）→ Phase 0 為顯性阻塞前提；未完成前以條件式語氣。
- **外部依賴維護節奏/授權變更**（LiteLLM、Agent Framework）→ 契約層與實作解耦，記錄替代方案清單，每半年一次依賴健康度覆核。
- **ontology-lite 關係查詢複雜度上升後效能不足**，且量化觸發門檻本身未經實測 → 需求明訂該門檻為「待校準」（G2），排入第一次季度覆核強制重新確認。
- **批次評測延誤高風險偵測時效** → 高風險分類強制排除批次、走即時常駐/同步（誠實揭露成本）；僅低風險走批次/快取。
- **路由分數單調老化 → 準冷啟動** → 治理式刷新批次（誠實揭露成本）或 stale-with-reporting（R2Q5）。
- **CI 抽測讓跨平台回歸延遲最多 24h** → 僅適用中低風險；路徑/密鑰/序列化/Unicode 相關變更以 PR 標籤強制觸發全 OS 矩陣。
- **治理閘門預設輕量被質疑合規不足** → 升級路徑（輕→重）與觸發條件為顯性規則，對映 NIST/EU AI Act「重大變更觸發重評估」，比例式合規非規避（G10）。
- **AIP/Palantir 命名爭議** → 內部 generic 用語，對外品牌另定，附差異聲明，法務複核後才對外發布。
- **供應商靜默改版偵測的非決定性誤報** → LLM 輸出即使 temperature=0 也因供應商 batching/硬體 nondeterminism/MoE 路由而非決定性，「雜湊精確比對連兩次不同即觸發」會保證每日誤報。改為**統計檢定**（語意嵌入餘弦相似度分布或輸出長度分布，管制圖/CUSUM）＋統計顯著門檻＋連續 N 天皆超過才判定改版；並**同時監控假陽性率與月假陽性預算**。

## scope_out（逐項排除 + 理由，對應 tasks.md future work）
- 不建自研圖資料庫/本體引擎 — 除非達 `ontology-lite` 明訂量化觸發條件（future work）。
- 不建常駐多 agent 即時 consensus 服務 — grill-me 對抗僅在高風險/驗證階段觸發，非常駐（future work）。
- 不建常駐 incident-containment 服務 — 僅折入設定式 circuit breaker/kill-switch（future work）。
- 不追求每 PR 全 OS 矩陣 — Linux 每 PR + Win/macOS 每日抽測。
- 不將重閘門套用於所有階段/風險分類 — 比例式觸發。
- 不自建路由引擎/評測指標庫 — 沿用 LiteLLM/DeepEval/Promptfoo/Ragas/M.E.AI.Evaluation。
- 不在本 change 定案對外品牌/商標 — 留待法務複核。
- 不做即時（sub-second）誤路由對帳 — 每日批次對帳。
- 不變更 `config.yaml` 部署禁令、不在 `specs/` 綁定具體部署選型 — 部署選型僅 design.md illustrative。
- 不建 Ontology-Adapter 抽象層（R2Q3）— 只有單一關聯式候選，依 G7 待第二實作再抽象化（future work）。
- 不將 EU AI Act Annex III / GDPR / WCAG 版本目標作為本 change 法遵定案依據 — 留待法務複核（G10）。
