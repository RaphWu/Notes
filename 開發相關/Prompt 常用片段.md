# Prompt 常用片段

依原本的格式
產生可給 Github Copilot 用於生成的 Prompt
使用 Markdown 語法風格 (程式碼區塊使用 csharp；powershell 指令用 bash；一般文字區塊使用 text)，圖表使用 Mermaid，標題之間不要使用分隔線

套件版本是由中央管理，專案檔中勿加入版本號

只編譯 XXX，其它專案不可變更、不須編輯，亦忽略其它專案的編譯錯誤

改用新的 #file:'R:\OneDrive\MyCodes\Projects\ScrewFastening\Calin.Logging\Calin.Logging.csproj' 專案做為日誌

檢查 XXX 專案檔，為不需要傳遞給依賴專案的套件加上 PrivateAssets="all"

Copyright ©2026 佳凌科技股份有限公司 Calin Technology Co.,Ltd.

重構 #file:'R:\OneDrive\MyCodes\Projects\ScrewFastening\Calin\Calin.csproj' ：

- 改用新的 #file:'R:\OneDrive\MyCodes\Projects\ScrewFastening\Calin.Logging\Calin.Logging.csproj' 專案做為日誌
- 只編譯 #file:'R:\OneDrive\MyCodes\Projects\ScrewFastening\Calin\Calin.csproj' ，其它專案不可變更、不須編輯，亦忽略其它專案的編譯錯誤。

---

Calin.MotionControl.Advantech
│
├─ Calin.MotionControl.Advantech.Core
├─ Calin.MotionControl.Advantech.Runtime
├─ Calin.MotionControl.Advantech.Configuration
├─ Calin.MotionControl.Advantech.Diagnostics
│
├─ Calin.MotionControl.Advantech.ACM
├─ Calin.MotionControl.Advantech.MotionNavi
├─ Calin.MotionControl.Advantech.EtherCAT
│
├─ Calin.MotionControl.Advantech.Simulation
│
└─ Calin.MotionControl.Advantech.Controller

我要建立一個輕量化 Motion Framework（Level 5 工控架構）的程式庫，Root namespace 為 `Calin.MotionControl.Advantech`，所有規範均在 #file:'R:\OneDrive\MyCodes\Projects\ACM\Prompt.md' 中。

- Dirty Flag 使用 #file:'R:\OneDrive\MyCodes\Projects\ScrewFastening\src\Calin\FA\README.md' 中所示範的方法。
- NuGet 套件採用中央版本管理，在 .csproj 專案檔中勿加入版本號。
- Logging 優先使用 Microsoft.Extensions.Logging，次選 LoggingBridge。
- 請依 #file:'R:\OneDrive\MyCodes\Projects\ACM\Prompt.md' 規劃生成 Calin.MotionControl.Advantech.Core

以下是 Calin.Communication.SerialPort 架構規劃（工控 LEVEL 5） ，產生可給 Github Copilot 用於生成的 Prompt，使用 Markdown 語法風格 (程式碼區塊使用 csharp；powershell 指令用 bash；一般文字區塊使用 text)，圖表使用 Mermaid，標題之間不要使用分隔線
