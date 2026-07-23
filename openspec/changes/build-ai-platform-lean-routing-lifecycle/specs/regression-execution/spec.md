## MODIFIED Requirements

> 相依 Phase 0。以下 MODIFIED 對映既有 `regression-execution` 原文 requirement「Deterministic execution controls」，將其決定性保證擴充為跨 OS 位元一致，非憑空撰寫。

### Requirement: Deterministic execution controls
系統 SHALL 記錄並遵守 run seed、scenario order、concurrency limit、warm-up count、measured repetition count、output-token limit；且相同 seed 與設定的重跑 SHALL 產生相同的排程與**跨 Windows/Linux/macOS 位元相同的 canonical 輸出**（強制 LF、InvariantCulture、正斜線路徑 canonical 化、單一 Unicode 正規化形式），使決定性不因 OS 差異而破壞。

#### Scenario: Repeated run uses the same schedule
- **WHEN** operator 以相同 suite、seed 與設定重跑
- **THEN** 系統產生相同的規劃 scenario order 與 concurrency schedule

#### Scenario: Same baseline regenerated across OS is bit-identical
- **WHEN** 同一 baseline JSON 在三個 OS 上以相同 seed 重新產生
- **THEN** canonical 輸出的位元內容（與其 SHA256）相同，diff 為空

## ADDED Requirements

### Requirement: Linux-first CI 矩陣與抽測
CI 執行矩陣 SHALL 預設 Linux-only 每 PR 跑，Win/macOS 每日一次抽測（成本裁決依 confirmed 費率 Linux $0.006 / Windows $0.010 / macOS $0.062 每分鐘；風險分層觸發為顯性規則，G3）。

#### Scenario: PR 只跑 Linux，每日跨 OS 抽測
- **WHEN** PR 觸發
- **THEN** 僅 Linux runner 執行完整回歸；每日排程在 Win/macOS 各跑一次，差異寫入報表但不阻擋 merge

#### Scenario: 敏感變更強制全矩陣
- **WHEN** PR 變更涉及路徑/密鑰/序列化/Unicode 正規化相關程式碼並帶對應標籤
- **THEN** 強制觸發全 OS 矩陣

### Requirement: 跨 OS 位元一致性回歸測試（五類，含 Unicode 正規化）
所有路徑組合、檔名大小寫比對、換行序列化、locale 格式化、**Unicode 正規化**（canonical 序列化前對所有字串欄位強制單一正規化形式，建議 NFC，與 .NET `string.Normalize` 對齊）SHALL 通過跨 OS 位元一致性回歸測試（五類各至少一案例，其中 Unicode 案例 SHALL 含 combining character 字串）。Unicode 一項依 G2 標註「工程盡職調查建議，非研究簡報 confirmed，illustrative 待驗證」。

#### Scenario: 三 OS 重生 baseline diff 為空
- **WHEN** 同一 baseline JSON 在三個 OS 上重新產生
- **THEN** diff 為空

#### Scenario: combining character 跨 OS 位元一致
- **WHEN** canonical JSON 含使用者提供之 combining character 字串（如模型名/標籤/prompt id），在 macOS（傾向 NFD）與 Linux/Windows（傾向 NFC）分別序列化
- **THEN** 經強制 NFC 正規化後三 OS 的 SHA256 相同

#### Scenario: 檔名大小寫在 canonical 比對強制敏感
- **WHEN** 兩個僅大小寫不同的 fixture 檔名在 Windows（不敏感）與 Linux（敏感）被比對
- **THEN** canonical 比對強制大小寫敏感，兩者視為不同，跨 OS 判定一致
