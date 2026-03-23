---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 工控級 SerialPort 安全等級分級表

Level 1 – 基本可靠
Level 2 – 企業級
Level 3 – 工控級
Level 4 – 高可靠容錯級

---

# LineBuffer 是什麼（工控通訊脈絡）

LineBuffer 是一種 **增量式資料組裝緩衝機制（Incremental Line Assembly Buffer）**，用於：

- 將多次 Read 取得的零碎 byte 資料累積
- 根據指定 Terminator 判斷是否形成完整「一行」
- 只在行完整時才對外觸發事件
- 避免頻繁字串建立與 GC 壓力

在工控 Serial 通訊中，它是 **避免假性斷線、誤判 Fault、資料殘缺** 的核心設計。

## 為什麼不能直接用 ReadLine()

`RJCP.SerialPortStream.ReadLine()` 有幾個問題：

1. 會阻塞直到遇到 NewLine
2. 內部使用字串分配
3. NewLine 只能單一設定
4. 若設備尾碼異常會卡死

工控 24/7 服務不能承受阻塞式設計，因此必須自己實作 LineBuffer。

## 工控實務問題

實際設備常見狀況：

- 尾碼可能是 `\r\n`
- 有些是 `\n`
- 有些無尾碼
- 有些回傳會分兩段
- 有些回傳中途會插入雜訊
- 有些只回傳 ACK (0x06)

若沒有 LineBuffer：

- 會誤判 TransmissionVerification 失敗
- 會誤判 Heartbeat Timeout
- 會因殘缺資料造成 ProtocolError

## LineBuffer 設計目標

1. 不阻塞 Read Task
2. 不假設每次 Read 都是完整行
3. 不頻繁配置字串
4. 支援自訂 Terminator
5. 支援 Terminator 為空字串（raw mode）
6. 可長時間運作不造成 GC 壓力

## 核心設計概念

### 1. 內部使用 byte[] 緩衝

```csharp
private byte[] _buffer;
private int _length;
```

- `_buffer`：累積資料
- `_length`：目前有效資料長度

### 2. 每次 Read 時 Append

```csharp
public void Append(ReadOnlySpan<byte> data)
```

- 將新資料拷貝到 buffer 尾端
- 若空間不足，成長為原來 2 倍
- 避免每次 new byte[]

### 3. 搜尋 Terminator

若 Terminator 為：

- `\r\n`
- `\n`
- `\r`
- 任意 byte[] 組合

則用 Span 搜尋：

```csharp
int index = bufferSpan.IndexOf(terminatorSpan);
```

找到後：

- 取出完整行
- 將剩餘資料往前搬移（Buffer Compact）
- 更新 _length

### 4. 若 Terminator = ""

代表：

- 不做 line-based 拆分
- 直接 raw data 模式
- 每次 Read 直接觸發 DataReceived

LineBuffer 只做 passthrough。

## 避免 GC 的關鍵技巧

### 1. 不立即轉 string

不要這樣：

```csharp
Encoding.ASCII.GetString(...)
```

應：

- 傳 byte[] 給外部
- 或延後轉換
- 或用 ArrayPool

### 2. ArrayPool 避免配置

```csharp
var rented = ArrayPool<byte>.Shared.Rent(size);
```

- 用完歸還
- 不可洩漏

### 3. 不使用 StringBuilder

StringBuilder 在高頻資料下仍會產生 GC 壓力。

## Read Loop 與 LineBuffer 整合

典型結構：

```csharp
while (!ct.IsCancellationRequested)
{
    int read = await stream.ReadAsync(tempBuffer, 0, tempBuffer.Length, ct);
    if (read > 0)
    {
        lineBuffer.Append(tempBuffer.AsSpan(0, read));

        while (lineBuffer.TryExtractLine(out ReadOnlyMemory<byte> line))
        {
            DispatchData(line);
        }
    }
}
```

特性：

- Read 永遠存在
- 不阻塞
- 不假設一行一次回來
- 可支援任何尾碼

## TryExtractLine 設計重點

```csharp
public bool TryExtractLine(out ReadOnlyMemory<byte> line)
```

行為：

1. 搜尋 terminator
2. 若找到：
    
    - 複製該行到 pool buffer
    - Compact 內部 buffer
    - 回傳 true
        
3. 若找不到：
    
    - 回傳 false

## Buffer Compact 機制

假設：

```csharp
HELLO\r\nWORLD\r\n
```

第一行取出後：

- 將 WORLD\r\n 往前搬移
- 更新 _length

避免重新配置新陣列。

## 進階：零複製版本

更工控等級做法：

- 不複製 line
- 直接回傳 internal segment
- 使用 ref-count 控制生命週期

但：

- 複雜度高
- 容易出現 race condition
- 不利維護

企業級建議：

使用 ArrayPool 複製，穩定性較高。

## 與 Transmission Verification 的關係

TransmissionVerification 會：

- 發送測試字串
- 等待回應

LineBuffer 確保：

- 即使回應被分段
- 或心跳與資料交錯
- 仍可正確組裝完整回應

否則會誤判：

- Timeout
- TransmissionVerificationFailed
- Fault

## 與 Heartbeat 的關係

心跳與一般資料共享 Read Loop。

LineBuffer 確保：

- 不會因資料混在一起造成判斷錯誤
- 不會因尾碼不同卡死

## 工控級穩定性考量

必須避免：

1. Terminator 設錯導致無限累積 buffer
2. 裝置噴大量資料造成記憶體爆炸
3. 未限制最大行長度

建議加入：

- MaxLineLength 設定
- 若超過：
    - 記錄 ProtocolError
    - 清空 buffer
    - 不進 Fault（除非連續發生）

## 總結

LineBuffer 是：

- 解決資料分段問題
- 解決尾碼差異問題
- 降低 GC 壓力
- 避免阻塞
- 提升 24/7 穩定性的關鍵設計

在你的 SerialPort 架構中，它是：

穩定性 > 相容性 > 效能

這三者的平衡核心。

---
