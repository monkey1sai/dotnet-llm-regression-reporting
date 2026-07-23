## MODIFIED Requirements

> 相依 Phase 0（design-enterprise archive 進 `openspec/specs/` base）；未完成前 MODIFIED 語意不套用至 base，僅結構驗證通過。以下兩條 MODIFIED 各對映既有 `endpoint-profiles` 之原文 requirement，非憑空撰寫。

### Requirement: Secret references only
系統 MUST 拒絕 committed endpoint-profile 欄位中的明文 API key 或 authorization token，並 SHALL 僅透過**部署無關、OS-agnostic 的 host secret 邊界**解析憑證：MUST NOT 依賴 Windows DPAPI、Windows 機碼或任何單一 OS 綁定的密鑰儲存機制，改以外部注入的環境變數或掛載 secret 參照（具體掛載機制為 design.md illustrative，非本 requirement 規範），且跨 Windows/Linux/macOS 行為一致。

#### Scenario: Environment secret is resolved cross-platform
- **WHEN** 一個 profile 在任一支援 OS 上參照被允許的環境變數且該變數存在
- **THEN** 系統將憑證交給 transport 而不持久化其值，且解析行為跨 OS 一致

#### Scenario: Plaintext credential is detected
- **WHEN** 一個 profile 欄位含被分類為憑證而非 secret 參照的值
- **THEN** 系統拒絕該 profile 並於診斷中遮蔽被偵測的值

#### Scenario: OS-bound secret store is rejected
- **WHEN** 一個 profile 要求以 Windows DPAPI 或機碼作為憑證來源
- **THEN** 系統拒絕該設定並要求改用部署無關的 secret 參照

### Requirement: Versioned endpoint profile
系統 SHALL 從版本化 schema 載入端點 profile（含 logical endpoint id、base URI、model id、secret 參照、timeouts、retry policy、declared capabilities），並為該次執行建立不可變的正規化 profile；該正規化 profile 的 canonical JSON 序列化 SHALL 在 Windows/Linux/macOS 上產出**位元相同的 SHA256 設定雜湊**（強制 LF、InvariantCulture、正斜線路徑 canonical 化、單一 Unicode 正規化形式）。

#### Scenario: Valid profile is accepted
- **WHEN** operator 提供符合支援 schema 版本的 profile
- **THEN** 系統建立不可變正規化 profile 供該次執行使用

#### Scenario: Unknown profile version is rejected
- **WHEN** profile 宣告不支援的 schema 版本
- **THEN** 系統在網路存取前停止並回報設定錯誤

#### Scenario: Cross-OS canonical hash is identical
- **WHEN** 在不同 OS 分別載入同一 profile
- **THEN** 序列化後的 canonical JSON SHA256 相同
