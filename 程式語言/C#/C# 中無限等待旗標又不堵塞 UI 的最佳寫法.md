---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
  - CSharp
  - 非同步程式設計
---

# C# 中無限等待旗標又不堵塞 UI 的最佳寫法

以下整理在 WinForm / WPF / MAUI 等 UI 情境下，不阻塞 UI Thread 的標準作法。

## 核心觀念

使用非同步 await + TaskCompletionSource 或輪詢結合 Task.Delay。避免 Thread.Sleep、WaitHandle.WaitOne 等阻塞式 API。

## 最佳解法：TaskCompletionSource（事件驅動）

適用：旗標狀態由某事件、回呼或其他執行緒「主動通知」時。

### 說明

TaskCompletionSource (TCS) 允許你建立一個可以 await 的 Task，旗標成立時只需呼叫 SetResult，不會阻塞 UI Thread，也不需要無限輪詢。

### 範例

```csharp
private TaskCompletionSource<bool> _flagTcs = new TaskCompletionSource<bool>();

public async Task WaitForFlagAsync()
{
    await _flagTcs.Task; // 不阻塞 UI Thread
    // 旗標成立後執行後續邏輯
}

// 某處旗標完成時呼叫
public void SetFlag()
{
    _flagTcs.TrySetResult(true);
}
```

## 次佳解法：非同步輪詢（適用沒有事件時）

適用：旗標只能被輪詢，沒有事件通知。

### 說明

以 async/await 搭配 Task.Delay 進行輕量輪詢，不會佔用任何 ThreadPool Thread，也不阻塞 UI。

### 範例

```csharp
public async Task WaitForFlagAsync(Func<bool> flagGetter)
{
    while (!flagGetter())
    {
        await Task.Delay(50); // 不阻塞 UI
    }
}
```

## 不建議的寫法（會阻塞 UI 或 Thread Pool）

1. Thread.Sleep
2. Task.Wait / Result
3. ManualResetEvent.WaitOne
4. SpinWait / while(true) 無休眠
5. Dispatcher.Invoke 同步等待

## 建議選擇方式

1. 有事件或能改寫流程 → 使用 TaskCompletionSource
2. 無法改寫旗標來源 → 使用非同步輪詢＋Task.Delay
3. 需要可取消 → 加上 CancellationToken

### 加上取消的版本

```csharp
public async Task WaitForFlagAsync(Func<bool> flagGetter, CancellationToken token)
{
    while (!flagGetter())
    {
        token.ThrowIfCancellationRequested();
        await Task.Delay(50, token);
    }
}
```
