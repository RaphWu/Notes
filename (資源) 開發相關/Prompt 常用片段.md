# Prompt 常用片段

依原本的格式
X 生成程式碼指令風格
產生可給 Github Copilot 用於生成程式碼的 Prompt
使用 Markdown 語法風格 (程式碼區塊使用 csharp；powershell 指令用 bash；一般文字區塊使用 text)，圖表使用 Mermaid，標題之間不要使用分隔線

---

因為 NuGet 的引用版本改為中央管理，請移除所有專案檔中的 NuGet 版本號
套件版本是由中央管理，專案檔中勿加入版本號
檢查 XXX 專案檔，採用「最小外部依賴面（Minimal Surface）」原則，為不需要傳遞給依賴專案的套件加上 PrivateAssets="all"

---

為 XXX 的所有公開型別提供完整 XML Summary（正體中文）
添加適當的 #region 做程式碼區隔

---

只編譯 XXX，其它專案不可變更、不須編輯，亦忽略其它專案的編譯錯誤

---

請依我的規範，新增你的建議，再給我新一版的規範，
採用原本格式，不要刪除原有條件(除非確定是多餘的)

---

使用 #file:'Calin.Logging.csproj' 專案做為日誌

Copyright ©2026 佳凌科技股份有限公司 Calin Technology Co.,Ltd.

---

請收集所有專案的 README.md 檔：

- 撰寫一個簡介，用來介紹 #CALIN.sln 方案
- 各所有 README.md 都移到所屬專案的根目錄：
  - 依 NuGet README 模板撰寫簡介，放置於所屬專案的根目錄的 README.md
  - 將完整的功能說明、API，放置於所屬專案的根目錄的 Document.md
- 完成撰寫後，刪除不在各專案根目錄的 README.md

---

重構 #file:'R:\OneDrive\MyCodes\Projects\ScrewFastening\Calin\Calin.csproj' ：

- 改用新的 #file:'R:\OneDrive\MyCodes\Projects\ScrewFastening\Calin.Logging\Calin.Logging.csproj' 專案做為日誌
- 只編譯 #file:'R:\OneDrive\MyCodes\Projects\ScrewFastening\Calin\Calin.csproj' ，其它專案不可變更、不須編輯，亦忽略其它專案的編譯錯誤。

- Dirty Flag 使用 #file:'R:\OneDrive\MyCodes\Projects\ScrewFastening\src\Calin\FA\README.md' 中所示範的方法。
- NuGet 套件採用中央版本管理，在 .csproj 專案檔中勿加入版本號。
- Logging 優先使用 Microsoft.Extensions.Logging，次選 LoggingBridge。
- 請依 #file:'R:\OneDrive\MyCodes\Projects\ACM\Prompt.md' 規劃生成 Calin.MotionControl.Advantech.Core
