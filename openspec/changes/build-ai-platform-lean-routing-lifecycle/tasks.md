## 0. 前置相依（Phase 0 — 必要前提，先於一切 MODIFIED/四態重述 delta 生效）

> 本階段任務落在 `openspec/changes/` 之外（`openspec/config.yaml`、`openspec archive` 寫入 `openspec/specs/`），為本 change 的顯性前置，不由本 change 自身檔案承載。

- [ ] 0.0 前置——確認 `design-enterprise-dotnet-regression-reporting` 達可 archive 完成度，執行 `openspec archive` 使其五個 capability 進入 `openspec/specs/` base，作為本 change 一切 MODIFIED/四態重述 delta **語意生效**的必要前提。**若尚未達可 archive 完成度，本 change 全部 Modified/收斂宣稱維持條件式語氣（strict 仍可過，但 delta 未套 base）。**
- [ ] 0.1 `config.yaml` 最小外科手術編輯：context 兩處 `Windows-first`→cross-platform/OS-agnostic；design 規則 `Windows portability`→`cross-platform portability (Windows/Linux/macOS parity)`。**不觸碰 proposal 部署禁令及其他規則。**

## 1. 跨平台化既有五能力（Modified 優先，低風險先落地；相依 Phase 0）

- [ ] 1.1 endpoint-profiles：MODIFIED `Secret references only` 去 DPAPI/機碼，改部署無關可攜密鑰參照；MODIFIED `Versioned endpoint profile` 加跨 OS canonical 雜湊測試
- [ ] 1.2 regression-execution：MODIFIED `Deterministic execution controls` 決定性跨 OS 擴充；ADDED CI 改 Linux-first + Win/macOS 每日抽測；ADDED 路徑/大小寫/CRLF/locale/Unicode 正規化（NFC，含 combining character 測試）五類位元一致性測試
- [ ] 1.3 performance-measurement / enterprise-reporting：ADDED 成本與路由追蹤欄位（含品質 provenance 標記），沿用既有 redaction

## 2. 垂直切片：category-router（先驗證擴充點契約可用）

- [ ] 2.1 薄封裝 LiteLLM，實作效用函式（品質下限 + 成本/延遲懲罰）
- [ ] 2.2 品質訊號 provenance：取最近批次分數 + 新鮮度上限/stale 標記 + 冷啟動保守預設模型
- [ ] 2.3 可獨立覆核之路由決策紀錄 + 可觀測失敗訊號（不可依賴 HTTP 200）
- [ ] 2.4 failover + 每日批次對帳 + routing-collapse 告警
- [ ] 2.5 設定式 circuit breaker + kill-switch；候選前置安全篩查（OWASP/GPL）

## 3. eval-backend-bridge（無狀態批次 + 高風險例外 + 路由新鮮度刷新）

- [ ] 3.1 DeepEval/Promptfoo 無狀態批次執行單元化，用完釋放運算資源
- [ ] 3.2 高風險分類即時評測例外（低延遲、隨時可用）；成本計入 scorecard 指標 2/9
- [ ] 3.3 路由新鮮度刷新批次（治理式、可設定頻率、成本揭露）；停用時於 enterprise-reporting 明示 stale 為預期
- [ ] 3.4 HTTP/gRPC 版本化契約 + 相容性回歸測試

## 4. lifecycle-gates（四態以 ADDED 重述，不跨資料夾 MODIFY baseline-policy-gates）

- [ ] 4.1 五階段進退場條件 + 輕量預設；風險分層開關；四態判定以 ADDED 重述（承認重複）
- [ ] 4.2 風險分類治理：產出主體（治理指派/屬性推導）+ fail-closed 預設高風險 + 分類變更寫入 Decision 且可對抗複核
- [ ] 4.3 多 agent 對抗式決策複核：第三方仲裁角色 + 布林 checklist 可重現 + 兩輪未收斂 fail-closed 升級人工
- [ ] 4.4 供應商改版偵測：有標頭指紋比對；無標頭改語意嵌入/長度分布統計檢定 + 顯著門檻 + 連續 N 天
- [ ] 4.5 高風險可回滾單元/RTO；供應鏈重閘門（AI-BOM/SLSA/WCAG/GPL）比例式掛載

## 5. ontology-lite（單一關聯式具體實作，依 G7 不建 Ontology-Adapter 抽象層）

- [ ] 5.1 八實體（含 Decision/Action）關聯式 schema + JSON Schema registry + JSON/CSV 匯出
- [ ] 5.2 schema 遷移治理（禁破壞性未遷移）
- [ ] 5.3 圖資料庫升級觸發條件（標註待校準）+ 排入首次季度覆核

## 6. reliability-lifecycle-scorecard

- [ ] 6.1 十指標公式 + 資料來源型態登記於 metric registry（G5 層1，scorecard 內版本化設定表）
- [ ] 6.2 回收期評分框架 + t0 估計值（estimated-pre-operational）與強制校準
- [ ] 6.3 校準翻盤後果：翻盤觸發平台級退場/回滾提案 + 對抗複核 + Decision 留痕
- [ ] 6.4 改版偵測假陽性率監控 + 月假陽性預算
- [ ] 6.5 錦標賽規則（≥3 實質差異候選、加權、tie-break）+ 評分軌跡回寫 ontology-ledger

## 7. 複雜度總帳與稽核

- [ ] 7.1 發布複雜度總帳（design.md 附錄）：各 capability ADDED+MODIFIED Req 數 + 可獨立部署/測試單元數
- [ ] 7.2 交付前強制執行一次「拆分評估」：以真實需求數重算各 capability 對其餘四者平均之比值，對照 1.75x（illustrative default）拆分線，記錄結論（拆或不拆及理由）；非僅將規則納入驗收

## 8. 其餘擴充點第二實作與退場（future work，降低初期交付風險）

- [ ] 8.1（future work）Ontology 圖資料庫具體實作——僅達量化觸發條件才啟動，屆時才依 G7 抽象化為 Ontology-Adapter
- [ ] 8.2（future work）常駐 incident-containment / 即時 consensus——僅事故頻率證明 ROI 才評估
- [ ] 8.3（future work）平台級退場/回滾演練——校準翻盤提案觸發後定義回滾單元與 RTO
- [ ] 8.4（future work）對外品牌/商標定案——法務複核後

## 9. 驗收準則對照（自檢）

- [ ] 9.1 `openspec validate build-ai-platform-lean-routing-lifecycle --strict` 通過（結構層）；Phase 0 完成後 delta 語意方生效
- [ ] 9.2 新增 capability ≤5；每個可回答「為何不用既有 OSS 元件直接滿足」；任何新抽象/契約已有 ≥2 具體實作候選（G7，Ontology-Adapter 已依 R2Q3 移除）
- [ ] 9.3 十指標每項附公式 + 資料來源型態、方法登記於 metric registry；t0 允許 estimated-pre-operational 且強制校準；回收期框架 + 校準翻盤後果齊備
- [ ] 9.4 每個 Windows-first 殘留假設（路徑/大小寫/CRLF/locale/Unicode）對應至少一個跨 OS 位元一致性測試，Unicode 含 combining character
- [ ] 9.5 供應商版本偵測含「有標頭」與「無標頭（統計檢定）」兩情境；scorecard 同時監控假陽性率與月假陽性預算
- [ ] 9.6 風險分類治理（fail-closed 預設高風險）、對抗複核（第三方仲裁 + 兩輪 fail-closed 升級）落入 lifecycle-gates
- [ ] 9.7 每個 scope_out 排除項於 tasks future work 標註理由（G9）
