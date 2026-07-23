## Why

現有 `design-enterprise-dotnet-regression-reporting`（五 capability：`endpoint-profiles`、`regression-execution`、`performance-measurement`、`baseline-policy-gates`、`enterprise-reporting`）只解決「單一 .NET LLM 端點的回歸/報表」，缺三件事：

1. **多模型/多供應商下的任務路由與成本治理**。promptfoo/DeepEval/Ragas 類 Python 評測工具已驗證缺生產監控、集中式實驗追蹤與協作/治理介面。
2. **橫跨提案→汰換的生命週期與比例式治理**。
3. **統一語義層讓「決策」（非僅資料）可被追蹤與匯出**（對標 Palantir Ontology 的 confirmed 定位：「代表企業決策而非資料」）。

若不填補，企業將被迫在多套零散工具間手動拼接，長期維運成本（人力、稽核、事故排查）會遠高於工具授權費本身。既有 change 的 Windows-first 假設是隱性維運成本（雙 OS 維護、IIS 相依、CI 費率差 2x–10x），必須移除；移除目的不是工程炫技式跨平台，而是**降低長期維運人力與 CI 帳單**。

**裁決主軸（成本維運優先，不得稀釋）**：任何新元件的存在，舉證責任在於「證明自建的邊際維運成本 < 買/沿用既有 OSS/雲端服務的邊際維運成本」，否則預設沿用既有元件（LiteLLM、Microsoft Agent Framework、Microsoft.Extensions.AI.Evaluation、DeepEval/Promptfoo、OTel）組裝，不重造執行引擎。AIP 的價值來自「治理契約 + 資料整合語意」，非「自建執行層」。

## What Changes

- 將既有五個 capability 全數保留為 AIP 的**執行層子能力**（非取代）；僅對含 Windows-first 執行邊界的段落做 `MODIFIED`（部署無關的無狀態執行 + Linux-first CI）。
- 新增**最多 5 個**新 capability（硬上限，並以複雜度總帳作第二道稽核），全部以「契約 + 既有 OSS 組裝」達成，刻意不新增「圖資料庫/本體引擎」「自研路由引擎」「常駐多 agent consensus 服務」「常駐 incident-containment 服務」等重資產：
  - `category-router`：分類路由**契約**（薄封裝 LiteLLM）。
  - `eval-backend-bridge`：Python 評測後端以**無狀態、用完即釋放的批次執行單元**執行（非常駐；高風險分類與路由新鮮度刷新為誠實揭露之例外）。
  - `lifecycle-gates`：五階段生命週期 + 比例式治理閘門 + 風險分類治理；以 `ADDED` 方式**重述**既有 `baseline-policy-gates` 四態判定語意（明文承認重複），**不對 baseline-policy-gates 做跨資料夾 `MODIFIED`**。
  - `ontology-lite`：關聯式 DB + JSON Schema registry 取代圖資料庫。
  - `reliability-lifecycle-scorecard`：reliable/innovative/stable 十指標 + generate-then-filter 錦標賽規則（回收期框架）。
- Modified：`endpoint-profiles`、`regression-execution` 的 Windows-first 執行邊界改寫為**部署無關的無狀態執行描述**；`performance-measurement`、`enterprise-reporting` 原樣保留並新增成本/路由可觀測性欄位；`baseline-policy-gates` **原檔維持原樣、不被本 change 改動**。
- **全域治理檔編輯（唯一範圍，見 tasks Phase 0）**：`config.yaml` context 兩處 `Windows-first`→cross-platform/OS-agnostic、design 規則 `Windows portability`→`cross-platform portability (Windows/Linux/macOS parity)`；**不觸碰 `config.yaml` 的 proposal 部署禁令及其他規則**。此編輯為全域規則層變更，落在 `openspec/changes/` 之外，屬本 change 的顯性前置任務，非本 change 自身檔案。

### 驗收狀態聲明與前置相依（時序硬約束）

本 change 的所有 `MODIFIED` 與 `lifecycle-gates` 四態重述 delta **硬性相依於先將 `design-enterprise-dotnet-regression-reporting` archive 進 `openspec/specs/` base**（見 tasks.md Phase 0）。經查證 `openspec/specs/` 目前為空、`openspec/changes/archive/` 為空、design-enterprise 尚未 archive。

- `openspec validate --strict` 只檢查 delta **結構**（operation header、每 Requirement 至少一 `#### Scenario:`、WHEN/THEN 文法），**不檢查 `MODIFIED` 的 base 是否存在**（base 存在性是 `openspec archive` 階段才檢查）。因此本 change **現階段即可通過 `openspec validate --strict`**。
- 但 `MODIFIED`/四態重述的**語意生效**（真正套用到 base spec）相依 Phase 0 archive。在 Phase 0 完成前，本 change 對既有五能力的所有「保留/Modified/收斂」宣稱一律以**條件式語氣**（「待 design-enterprise archive 後生效」）閱讀，不得當既定事實。

## 與既有 change 的關係（納入為子能力，非取代、非並存孤立；delta 生效相依 Phase 0）

| 既有 capability | 處置（待 Phase 0 archive 後語意生效） | 具體改寫段落 |
|---|---|---|
| `endpoint-profiles` | Modified（2 MODIFIED delta，各對映 base 原文 requirement） | `Secret references only`：密鑰儲存去 Windows DPAPI/機碼 → 部署無關的可攜密鑰參照；`Versioned endpoint profile`：canonical 設定雜湊跨 OS 位元一致 |
| `regression-execution` | Modified（1 MODIFIED + 2 ADDED） | MODIFIED `Deterministic execution controls`：決定性擴充為跨 OS 位元一致；ADDED：Linux-first CI 矩陣（Linux 每 PR + Win/macOS 每日抽測）、路徑/大小寫/CRLF/locale/**Unicode 正規化**五類跨 OS 位元一致性測試 |
| `performance-measurement` | Modified（additive，1 ADDED 可選欄位需求，非重寫既有 requirement） | 新增可選欄位 cost-per-eval、cache-hit-rate |
| `baseline-policy-gates` | **原檔維持原樣（0 delta，不跨資料夾 MODIFIED）** | 四態判定 pass/warn/fail/inconclusive 由 `lifecycle-gates` 以 `ADDED` **重述**（明文承認重複），並疊加「風險分層開關」；baseline-policy-gates 自身不被本 change 改動 |
| `enterprise-reporting` | Modified（additive，1 ADDED 欄位需求，非重寫既有 requirement） | 新增「每次評測成本(USD/eval)」「路由決策追蹤」欄位 |

**能力更名/收斂對照表（純人讀，非工具遷移）**：`baseline-policy-gates`（舊 id）→ `lifecycle-gates` 的四態語意**以 `ADDED` 重述**，逐條標註：四態判定=**重述保留（承認與原文重複）**、風險分層開關=**擴充**、無任何**棄用**項。此對照表**僅為人讀可追溯文件，不是 `openspec validate` 解析執行的 machine-readable 遷移指令**；OpenSpec 無原生 rename，validate 仍會將 baseline-policy-gates 與 lifecycle-gates 視為兩個獨立 capability。本 change **不對 baseline-policy-gates/spec.md 下任何 `MODIFIED` delta**。

## 跨平台化對 Windows-first 假設的具體修改點

研究簡報 confirmed 的四類 + 工程盡職調查建議的第五類（Unicode）須移除/釘死的假設，各對應至少一個跨 OS 位元一致性回歸測試（驗收硬約束）：

1. **路徑組合**（`Path.Combine`／分隔符）→ 統一以正斜線 canonical 化。
2. **檔名大小寫敏感**（Windows 不敏感 / Linux 敏感）→ canonical 比對強制大小寫敏感。
3. **CRLF/換行**對權威 canonical JSON 的位元穩定性 → 強制 LF、序列化位元固定。
4. **locale/文化相依格式化**（數字、日期）→ 強制 InvariantCulture。
5. **Unicode 正規化（NFC/NFD）**〔工程盡職調查建議，非研究簡報 confirmed，illustrative 待驗證〕→ canonical 序列化前對所有字串欄位（模型名、標籤、prompt id 等使用者可控輸入）強制套用單一正規化形式（建議 NFC，與 .NET `string.Normalize` 對齊），消除 macOS（傾向 NFD）與 Linux/Windows（傾向 NFC）之間的 SHA256 差異。

**執行邊界（部署無關）**：不得假設 IIS/Windows Service 等 OS 綁定常駐宿主，改以**部署無關的無狀態執行單元**描述可觀測行為；具體容器/K8s/Azure Container Apps 等選型為 design.md 的 illustrative 實作建議、非本 repo 定案，且**不變更 `config.yaml` 部署禁令**。CI：Linux-first 每 PR、Win/macOS 每日抽測（成本裁決依 confirmed 官方費率：GitHub Actions Linux $0.006 / Windows $0.010 / macOS $0.062 每分鐘）。

## PLTR 對標的具體借鏡與差異

**借鏡（confirmed 概念）**：
- Ontology 定位為「代表企業**決策**而非資料」→ `ontology-lite` 除七個資料實體外，**新增 Decision/Action 一等實體**，使決策可被追蹤/匯出。
- AIP Logic / Evals 在 Ontology 上建生產級 agent → 本方案以 Microsoft Agent Framework（Process Framework，Q2 2026 GA）承接生命週期編排，`reliability-lifecycle-scorecard` + `lifecycle-gates` 對映 AIP Evals 的治理角色。
- 統一語義 + kinetic 元素（actions/functions/models/dynamic security）→ 以 `ontology-lite` 的 GovernanceGate + Decision/Action 實體 + policy engine 達成等價治理語意。

**差異（硬約束，設計非目標＝不複製 Palantir 鎖定護城河）**：
- Foundry/Ontology 為**閉源**；本方案以「關聯式 DB + JSON Schema registry + policy engine」自建**可匯出、模型無關、開放 schema**的等價物。
- 明訂**遷移/退場**路徑，反 lock-in。
- 不自建圖資料庫（Palantir 級重資產），僅在達明訂量化觸發條件時才提案升級。

## 命名/IP

內部「AIP」= generic AI Platform 描述性用語；對外品牌另定（不單獨用「AIP」）；所有 Palantir 引用限名義性合理使用並附差異聲明。商標定案**留待法務複核**，不在本 change 定案。

## Impact

- 建立 AIP 治理契約層的可驗證 spec；既有五能力被納為執行層子能力並跨平台化。
- 新依賴：LiteLLM（路由）、Microsoft Agent Framework（生命週期編排）、DeepEval/Promptfoo（評測資料平面）、OTel GenAI（可觀測性，以 adapter/反腐層鎖定契約版本）。
- 不部署模型、不改 LLM 伺服器、不註冊 MCP、不在 `specs/` 綁定具體部署選型；部署選型僅 design.md illustrative。
- 需要一次全域治理檔（`config.yaml`）最小外科手術編輯以移除 Windows-first 措辭（Phase 0 前置，落在 `openspec/changes/` 之外）。

## Open Questions（殘留未解決詰問）

以下為第二輪對抗詰問（grill-me）後仍未完全收斂之點。標「已採方向」者本 change 已於 specs/design 落入對應需求以維持內部一致，但殘留子問題仍待後續對抗循環或營運資料校準定案；未落入者純列為待決。所有數值門檻均為 illustrative default，待校準（見 design.md GLOBAL 規則 G2）。

- **R2Q1 風險分類本身的治理（已採方向，殘留待定）**：全份設計的比例式治理（高風險常駐評測、多 agent 對抗複核、供應鏈重閘門、canary/RTO/kill-switch）全掛在「高風險分類」開關上，但誰、依何準則、在哪個生命週期節點判定高/低風險，草案原未定義。**已採方向**：於 `lifecycle-gates` 新增「風險分類治理」需求——(a) 分類為受治理決策（治理指派或由 ontology-lite 屬性推導，非純自我宣告）；(b) **未分類/分類缺失時 fail-closed 預設高風險**（與全份 fail-closed 哲學一致）；(c) 分類變更本身為需寫入 Decision 實體、可被對抗複核的受治理決策。**殘留**：分類準則的具體 rubric（哪些訊號→高風險）尚未定案，待首次治理委員會校準。
- **R2Q2 metric registry 的 owner 與舉證倒置（已採方向，殘留待定）**：G5 層1 使 metric registry 成為能「拒絕評分」的強制閘門，但草案未指派其 owner capability，也未對其套用舉證倒置。**已採方向**：metric registry 定為 `reliability-lifecycle-scorecard` 內的**版本化設定表**（沿用 `enterprise-reporting`/`ontology-lite` 既有關聯式儲存，**非新增常駐 service**），故不違反「新元件預設不建」；其版本化與稽核查詢能力以既有 DB 的 schema 版本治理達成。**殘留**：是否需獨立於 scorecard 的跨 capability 共用 registry（若未來 lifecycle-gates 亦需登記閘門指標）待第二使用者出現再評估（G7）。
- **R2Q3 G7 與 Ontology-Adapter 抽象層矛盾（已收斂並修正）**：G7 硬約束「新增抽象層須已知 ≥2 具體實作候選」，但 design.md 擴充點表的 Ontology-Adapter 第二候選明文為「未來/僅達觸發條件才啟動」，正是 G7 要禁止的投機性第二實作。**已收斂裁決**：採方向 (a)——**移除 Ontology-Adapter 抽象層**，`ontology-lite` 此刻只做單一關聯式具體實作，圖資料庫僅為 future-work 觸發條件（非現在就建 adapter 契約）；**G7 不放寬**，對其他擴充點約束力不被稀釋。design.md 擴充點表已據此修正。
- **R2Q4 G11 拆分倍數待定 + capability 複雜度離群（已採方向，殘留待定）**：G11 拆分觸發門檻「超過其餘四能力平均值的既定倍數」本身「待定」，使「客觀量」無法客觀判定。**已採方向**：design.md 複雜度總帳以真實需求數重算（見附錄），設 illustrative 倍數並計算各 capability 對「其餘四者平均」之比值；**強制交付一次拆分評估**（tasks 7.2，非僅「納入驗收」）。**殘留**：倍數具體值仍為 illustrative default 待校準，故此第二道量在校準前尚非完全客觀；已如實標註。
- **R2Q5 路由分數新鮮度的刷新來源（已採方向，殘留待定）**：category-router 依「最近一次回歸批次品質分數」，但 eval-backend-bridge 批次僅由 regression 事件驅動；穩定生產、近期無變更的 `(model,category)` 分數會單調老化至全部 stale，路由退化成準冷啟動。**已採方向**：於 `eval-backend-bridge` 新增「路由新鮮度刷新批次」需求——治理式、可設定頻率的排程刷新批次，其固定運算成本**誠實揭露**並計入 scorecard 指標 1/2（與高風險常駐評測 Q6 同樣揭露張力）；若治理選擇停用刷新，則 stale 分數須於 `enterprise-reporting` 明示為預期行為。**殘留**：刷新頻率/成本上限的最佳點待營運資料校準。
- **R2Q6 校準翻盤後的平台級後果（已採方向，殘留待定）**：t0 以估計值選定平台設計、N 週後強制重新評分，但草案只規定「重新評分」動作，未規定「若校準後回收期轉負、或原落敗的純手動拼接候選反勝，對已建成平台做什麼」。**已採方向**：於 `reliability-lifecycle-scorecard` 新增「校準翻盤後果」需求——翻盤 SHALL 觸發平台級退場/回滾**提案**（對映 design.md 反 lock-in 承諾），翻盤判定本身 SHALL 走多 agent 對抗複核並於 Decision 留痕。**殘留**：回滾單元是否為「整個 AIP change」及其 RTO 尚未量化，待首次治理覆核定案。
