## ADDED Requirements

> 對既有 `enterprise-reporting` 的 additive 擴充（以 ADDED 新增欄位需求，非重寫既有 requirement，故不用 MODIFIED，避免憑空 Modified）。相依 Phase 0 後與 base 並存；不改動既有 redaction 與 canonical 語意。

### Requirement: 路由決策與成本追蹤欄位
系統 SHALL 新增「每次評測成本(USD/eval)」與「路由決策追蹤」欄位（含候選集合、效用分數、選中理由、品質分數 provenance 與新鮮度/cold-start 標記），供第三方獨立覆核；此追蹤欄位 SHALL 沿用既有 redaction 規則，MUST NOT 洩漏憑證或受限 payload。

#### Scenario: 報表含可覆核路由紀錄
- **WHEN** 產生報表
- **THEN** 每筆路由決策附候選集合、各候選效用分數、選中項與品質分數 provenance（含 stale/cold-start 標記），稽核者可獨立重算

#### Scenario: 路由追蹤欄位遵守 redaction
- **WHEN** 路由候選或選中理由含被分類為敏感的 payload
- **THEN** 報表以既有 redaction 標記取代該值，不寫入原文
