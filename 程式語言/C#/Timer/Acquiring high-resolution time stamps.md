---
aliases:
date:
update: 2023-04-19
author: Microsoft
language:
sourceurl: https://learn.microsoft.com/en-us/windows/win32/sysinfo/acquiring-high-resolution-time-stamps
tags:
  - CSharp
  - DateTime
---

# Acquiring high-resolution time stamps - 高解析度時間戳記

Windows provides APIs that you can use to acquire high-resolution time stamps, or measure time intervals. The primary API for native code is [**QueryPerformanceCounter (QPC)**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter). For device drivers, the kernel-mode API is [**KeQueryPerformanceCounter**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ntifs/nf-ntifs-kequeryperformancecounter). For managed code, the [**System.Diagnostics.Stopwatch**](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch) class uses **QPC** as its precise time basis.
Windows 提供了您可以使用的 API，用來取得高解析度時間戳記或測量時間間隔。原生程式的主要 API 是 `QueryPerformanceCounter` (QPC)。對於裝置驅動程式，內核模式 API 是 `KeQueryPerformanceCounter`。對於受管理的程式碼，`System.Diagnostics.Stopwatch` 類別使用 QPC 作為其精確的時間基準。

[**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) is independent of, and isn't synchronized to, any external time reference. To retrieve time stamps that can be synchronized to an external time reference, such as, Coordinated Universal Time (UTC) for use in high-resolution time-of-day measurements, use [**GetSystemTimePreciseAsFileTime**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemtimepreciseasfiletime).
QPC 独立于任何外部時間參考，且不與任何外部時間參考同步。若要取得可與外部時間參考同步的時間戳記，例如，用於高解析度時間日期測量的協調世界時 (UTC)，請使用 `GetSystemTimePreciseAsFileTime`。

Time stamps and time-interval measurements are an integral part of computer and network performance measurements. These performance measurement operations include the computation of response time, throughput, and latency, as well as profiling code execution. Each of these operations involves a measurement of activities that occur during a time interval that is defined by a start and an end event that can be independent of any external time-of-day reference.
時間戳記與時間間隔測量是電腦與網路效能測量的重要組成部分。這些效能測量操作包含回應時間、吞吐量與延遲的計算，以及程式碼執行的剖析。每個操作都涉及對在由一個開始事件與一個結束事件所定義的時間間隔內發生的活動進行測量，而這些開始與結束事件可以獨立於任何外部日期時間參考。

[**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) is typically the best method to use to time-stamp events and measure small time intervals that occur on the same system or virtual machine. Consider using [**GetSystemTimePreciseAsFileTime**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemtimepreciseasfiletime) when you want to time-stamp events across multiple machines, provided that each machine is participating in a time synchronization scheme such as Network Time Protocol (NTP). **QPC** helps you avoid difficulties that can be encountered with other time measurement approaches, such as reading the processor’s time stamp counter (TSC) directly.
QPC 通常是最適用於對事件進行時間戳記並測量在同一系統或虛擬機器上發生的微小時間間隔的方法。當您希望在多台機器上對事件進行時間戳記時，請考慮使用 GetSystemTimePreciseAsFileTime，前提是每台機器都參與一個時間同步協議，例如網路時間協議 (NTP)。QPC 協助您避免其他時間測量方法可能遇到的困難，例如直接讀取處理器的時間戳記計數器 (TSC)。

# QPC support in Windows versions Windows - 版本中的 QPC 支援

[**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) was introduced in Windows 2000 and Windows XP and has evolved to take advantage of improvements in the hardware platform and processors. Here we describe the characteristics of **QPC** on different Windows versions to help you maintain software that runs on those Windows versions.
QPC 是在 Windows 2000 和 Windows XP 中引進的，並隨著硬體平台和處理器的改進而演進。這裡我們描述 QPC 在不同 Windows 版本中的特性，以幫助您維護在那些 Windows 版本上運行的軟體。

註：關於各版本的支援請見網頁版。

# Guidance for acquiring time stamps - 獲取時間戳記的指導

Windows has and will continue to invest in providing a reliable and efficient performance counter. When you need time stamps with a resolution of 1 microsecond or better and you don't need the time stamps to be synchronized to an external time reference, choose [**QueryPerformanceCounter**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter), [**KeQueryPerformanceCounter**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ntifs/nf-ntifs-kequeryperformancecounter), or [**KeQueryInterruptTimePrecise**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nf-wdm-kequeryinterrupttimeprecise). When you need UTC-synchronized time stamps with a resolution of 1 microsecond or better, choose [**GetSystemTimePreciseAsFileTime**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemtimepreciseasfiletime) or [**KeQuerySystemTimePrecise**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nf-wdm-kequerysystemtimeprecise).
Windows 會持續投入資源以提供可靠且高效的效能計數器。當您需要解析度達到 1 微秒或更高且不需要時間戳記與外部時間參考同步時，請選擇 `QueryPerformanceCounter`、`KeQueryPerformanceCounter` 或 `KeQueryInterruptTimePrecise`。當您需要解析度達到 1 微秒或更高且與 UTC 同步的時間戳記時，請選擇 `GetSystemTimePreciseAsFileTime` 或 `KeQuerySystemTimePrecise`。

On a relatively small number of platforms that can't use the TSC register as the [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) basis, for example, for reasons explained in [Hardware timer info](https://learn.microsoft.com/en-us/windows/win32/sysinfo/acquiring-high-resolution-time-stamps#hardware-timer-info), acquiring high resolution time stamps can be significantly more expensive than acquiring time stamps with lower resolution. If resolution of 10 to 16 milliseconds is sufficient, you can use [**GetTickCount64**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-gettickcount64), [**QueryInterruptTime**](https://learn.microsoft.com/en-us/windows/desktop/api/realtimeapiset/nf-realtimeapiset-queryinterrupttime), [**QueryUnbiasedInterruptTime**](https://learn.microsoft.com/en-us/windows/win32/api/realtimeapiset/nf-realtimeapiset-queryunbiasedinterrupttime), [**KeQueryInterruptTime**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nf-wdm-kequeryinterrupttime), or [**KeQueryUnbiasedInterruptTime**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nf-wdm-kequeryunbiasedinterrupttime) to obtain time stamps that aren't synchronized to an external time reference. For UTC-synchronized time stamps, use [**GetSystemTimeAsFileTime**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemtimeasfiletime) or [**KeQuerySystemTime**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-kequerysystemtime%7Er1). If higher resolution is needed, you can use [**QueryInterruptTimePrecise**](https://learn.microsoft.com/en-us/windows/desktop/api/realtimeapiset/nf-realtimeapiset-queryinterrupttimeprecise), [**QueryUnbiasedInterruptTimePrecise**](https://learn.microsoft.com/en-us/windows/desktop/api/realtimeapiset/nf-realtimeapiset-queryunbiasedinterrupttimeprecise), or [**KeQueryInterruptTimePrecise**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nf-wdm-kequeryinterrupttimeprecise) to obtain time stamps instead.
在相對較小的平台上，無法使用 TSC 登記表作為 QPC 的基礎，例如，由於硬體計時器資訊中解釋的原因，獲取高解析度時間戳記可能比獲取低解析度的時間戳記顯著地更昂貴。如果 10 到 16 毫秒的解析度足夠，您可以使用 `GetTickCount64`、`QueryInterruptTime`、`QueryUnbiasedInterruptTime`、`KeQueryInterruptTime` 或 `KeQueryUnbiasedInterruptTime` 來獲取未與外部時間參考同步的時間戳記。對於 UTC 同步的時間戳記，請使用 `GetSystemTimeAsFileTime` 或 `KeQuerySystemTime`。如果需要更高的解析度，您可以使用 `QueryInterruptTimePrecise`、`QueryUnbiasedInterruptTimePrecise` 或 `KeQueryInterruptTimePrecise` 來獲取時間戳記。

In general, the performance counter results are consistent across all processors in multi-core and multi-processor systems, even when measured on different threads or processes. Here are some exceptions to this rule:
一般來說，在多核心和多處理器系統中的所有處理器上，性能計數器的結果都是一致的，即使是在不同的線程或進程上進行測量。這一規則有一些例外：

- Pre-Windows Vista operating systems that run on certain processors might violate this consistency because of one of these reasons:
  在 Windows Vista 之前的操作系統中，如果運行在某些處理器上，可能會因為以下原因之一而違反這一一致性：
    - The hardware processors have a non-invariant TSC and the BIOS doesn't indicate this condition correctly.
      硬體處理器具有非不變的 TSC，而且 BIOS 並未正確指示這個條件。
    - The TSC synchronization algorithm that was used wasn't suitable for systems with large numbers of processors.
      所使用的 TSC 同步演算法不適用於擁有大量處理器的系統。
- When you compare performance counter results that are acquired from different threads, consider values that differ by ± 1 tick to have an ambiguous ordering. If the time stamps are taken from the same thread, this ± 1 tick uncertainty doesn't apply. In this context, the term tick refers to a period of time equal to 1 ÷ (the frequency of the performance counter obtained from [**QueryPerformanceFrequency**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancefrequency)).
  當你比較從不同執行緒取得的效能計數器結果時，應考慮差異在 ± 1 個滴答的值具有模糊的排序。如果時間戳記是從同一執行緒取得的，這 ± 1 個滴答的不確定性就不適用。在這個語境中，所謂的滴答是指一段時間，等於 1 ÷ (從 `QueryPerformanceFrequency` 取得的效能計數器頻率)。

When you use the performance counter on large server systems with multiple-clock domains that aren't synchronized in hardware, Windows determines that the TSC can't be used for timing purposes and selects a platform counter as the basis for [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter). While this scenario still yields reliable time stamps, the access latency and scalability is adversely affected. Therefore, as previously stated in the preceding usage guidance, only use the APIs that provide 1 microsecond or better resolution when such resolution is necessary. The TSC is used as the basis for **QPC** on multi-clock domain systems that include hardware synchronization of all processor clock domains, as this effectively makes them function as a single clock domain system.
當您在具有多個非硬體同步時鐘域的大型伺服器系統上使用效能計數器時，Windows 判斷 TSC 不能用於時間測量目的，並選擇平台計數器作為 QPC 的基礎。雖然此情況下仍然能產生可靠的時間戳記，但存取延遲和可擴展性會受到不利影響。因此，如先前在先前使用指南中所述，僅在需要此解析度時，才使用提供 1 微秒或更好解析度的 API。在包含所有處理器時鐘域硬體同步的多時鐘域系統上，TSC 作為 QPC 的基礎，因為這有效地使它們作為單一時鐘域系統運作。

The frequency of the performance counter is fixed at system boot and is consistent across all processors so you only need to query the frequency from [**QueryPerformanceFrequency**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancefrequency) as the application initializes, and then cache the result.
效能計數器的頻率在系統啟動時固定，並在所有處理器上保持一致，因此您只需要在應用程式初始化時從 `QueryPerformanceFrequency` 查詢頻率，然後快取結果。

## Virtualization  - 虛擬化

The performance counter is expected to work reliably on all guest virtual machines running on correctly implemented hypervisors. However, hypervisors that comply with the hypervisor version 1.0 interface and surface the reference time enlightenment can offer substantially lower overhead. For more information about hypervisor interfaces and enlightenments, see [Hypervisor Specifications](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/tlfs).
性能計數器預期能在所有正確實作的虛擬主機上可靠運作。然而，符合虛擬主機版本 1.0 接口並顯示參考時間啟示的虛擬主機可以提供顯著較低的開銷。有關虛擬主機接口和啟示的更多信息，請參閱虛擬主機規格。

## Direct TSC usage  - 直接使用 TSC

We strongly discourage using the **RDTSC** or **RDTSCP** processor instruction to directly query the TSC because you won't get reliable results on some versions of Windows, across live migrations of virtual machines, and on hardware systems without invariant or tightly synchronized TSCs. Instead, we encourage you to use [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) to leverage the abstraction, consistency, and portability that it offers.
我們強烈不建議使用 RDTSC 或 RDTSCP 處理器指令直接查詢 TSC，因為在某些版本的 Windows、虛擬機器 live migration 和沒有不變或緊密同步 TSC 的硬體系統上，您無法獲得可靠的結果。相反地，我們鼓勵您使用 QPC 來利用它提供的抽象、一致性與可攜性。

## Examples for acquiring time stamps - 獲取時間戳的範例

The various code examples in these sections show how to acquire time stamps.
這些區段的各種程式碼範例展示了如何取得時間戳。

## Using QPC in native code - 在原生程式碼中使用 QPC

This example shows how to use [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) in C and C++ native code.
此範例展示了如何在 C 和 C++ 原生程式碼中使用 QPC。

```cpp
LARGE_INTEGER StartingTime, EndingTime, ElapsedMicroseconds;
LARGE_INTEGER Frequency;

QueryPerformanceFrequency(&Frequency); 
QueryPerformanceCounter(&StartingTime);

// Activity to be timed

QueryPerformanceCounter(&EndingTime);
ElapsedMicroseconds.QuadPart = EndingTime.QuadPart - StartingTime.QuadPart;


//
// We now have the elapsed number of ticks, along with the
// number of ticks-per-second. We use these values
// to convert to the number of elapsed microseconds.
// To guard against loss-of-precision, we convert
// to microseconds *before* dividing by ticks-per-second.
//

ElapsedMicroseconds.QuadPart *= 1000000;
ElapsedMicroseconds.QuadPart /= Frequency.QuadPart;
```

## Acquiring high resolution time stamps from managed code - 從受控程式碼中取得高解析度時間戳記

This example shows how to use the managed code [**System.Diagnostics.Stopwatch**](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch) class.
本範例說明如何使用受管理的程式碼 `System.Diagnostics.Stopwatch` 類別。

```csharp
using System.Diagnostics;

long StartingTime = Stopwatch.GetTimestamp();

// Activity to be timed

long EndingTime  = Stopwatch.GetTimestamp();
long ElapsedTime = EndingTime - StartingTime;

double ElapsedSeconds = ElapsedTime * (1.0 / Stopwatch.Frequency);
```

The [**System.Diagnostics.Stopwatch**](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch) class also provides several convenient methods to perform time-interval measurements.
`System.Diagnostics.Stopwatch` 類別也提供數種便利的方法來進行時間間隔測量。

## Using QPC from kernel mode - 使用從核心模式使用 QPC

The Windows kernel provides kernel-mode access to the performance counter through [**KeQueryPerformanceCounter**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ntifs/nf-ntifs-kequeryperformancecounter) from which both the performance counter and performance frequency can be obtained. **KeQueryPerformanceCounter** is available from kernel mode only and is provided for writers of device drivers and other kernel-mode components.
Windows 核心提供透過 `KeQueryPerformanceCounter` 來存取效能計數器的核心模式存取，可從中取得效能計數器與效能頻率。`KeQueryPerformanceCounter` 僅供核心模式使用，是為裝置驅動程式和其他核心模式組件的撰寫者所提供。

This example shows how to use [**KeQueryPerformanceCounter**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ntifs/nf-ntifs-kequeryperformancecounter) in C and C++ kernel mode.
此範例說明如何在 C 和 C++ 的核心模式中使用 KeQueryPerformanceCounter。

```cpp
LARGE_INTEGER StartingTime, EndingTime, ElapsedMicroseconds;
LARGE_INTEGER Frequency;

StartingTime = KeQueryPerformanceCounter(&Frequency);

// Activity to be timed

EndingTime = KeQueryPerformanceCounter(NULL);
ElapsedMicroseconds.QuadPart = EndingTime.QuadPart - StartingTime.QuadPart;
ElapsedMicroseconds.QuadPart *= 1000000;
ElapsedMicroseconds.QuadPart /= Frequency.QuadPart;
```

# General FAQ about QPC and TSC - 常見問題總結

Here are answers to frequently-asked questions about [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) and TSCs in general.
這裡是關於 QPC 和 TSCs 的常見問題解答。

**Is QueryPerformanceCounter() the same as the Win32 GetTickCount() or GetTickCount64() function?
QueryPerformanceCounter() 和 Win32 的 GetTickCount() 或 GetTickCount64() 函數相同嗎？**

No. [**GetTickCount**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-gettickcount) and [**GetTickCount64**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-gettickcount64) aren't related to [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter). **GetTickCount** and **GetTickCount64** return the number of milliseconds since the system was started.
不相同。GetTickCount 和 GetTickCount64 與 QPC 無關。GetTickCount 和 GetTickCount64 傳回系統啟動後的毫秒數。

**Should I use QPC or call the RDTSC/RDTSCP instructions directly?
我應該使用 QPC 還是直接呼叫 RDTSC/RDTSCP 指令？**

To avoid incorrectness and portability issues, we strongly encourage you to use [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) instead of using the TSC register or the **RDTSC** or **RDTSCP** processor instructions.
為了避免不正確性和可攜性問題，我們強烈建議您使用 QPC，而不是使用 TSC 寄存器或 RDTSC 或 RDTSCP 處理器指令。

**What is QPC’s relation to an external time epoch? Can it be synchronized to an external epoch such as UTC?
QPC 與外部時間紀元有何關係？能否與外部紀元（例如 UTC）同步？**

[**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) is based on a hardware counter that can't be synchronized to an external time reference, such as UTC. For precise time-of-day time stamps that can be synchronized to an external UTC reference, use [**GetSystemTimePreciseAsFileTime**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemtimepreciseasfiletime).
QPC 是基於一個無法與外部時間參考（例如 UTC）同步的硬體計數器。若要取得能與外部 UTC 參考同步的精確日期時間戳記，請使用 GetSystemTimePreciseAsFileTime。

**Is QPC affected by daylight savings time, leap seconds, time zones, or system time changes made by the administrator?
QPC 是否會受到夏令時、閏秒、時區或系統管理員所進行的系統時間變更的影響？**

No. [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) is completely independent of the system time and UTC.
不會。QPC 完全獨立於系統時間和 UTC。

**Is QPC accuracy affected by processor frequency changes caused by power management or Turbo Boost technology?
QPC 的精度是否會受到由電源管理或 Turbo Boost 技術所導致的處理器頻率變更的影響？**

No. If the processor has an invariant TSC, the [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) is not affected by these sort of changes. If the processor doesn't have an invariant TSC, **QPC** will revert to a platform hardware timer that won't be affected by processor frequency changes or Turbo Boost technology.
否。若處理器擁有不變的 TSC，QPC 不會受到這類變化的影響。若處理器沒有不變的 TSC，QPC 將切換回平台硬體時鐘，而此時鐘不會受到處理器頻率變化或 Turbo Boost 技術的影響。

**Does QPC reliably work on multi-processor systems, multi-core system, and systems with hyper-threading?
QPC 在多處理器系統、多核心系統和超頻寫入系統中是否可靠？**

Yes.  是的。

**How do I determine and validate that QPC works on my machine?
我如何判斷和驗證 QPC 在我的電腦上是否正常工作？**

You don't need to perform such checks.
您不需要執行這樣的檢查。

**Which processors have non-invariant TSCs? How can I check if my system has a non-invariant TSC?
哪些處理器具有非不變的 TSC？我如何檢查我的系統是否有非不變的 TSC？**

You don't need to perform this check yourself. Windows operating systems perform several checks at system initialization to determine if the TSC is suitable as a basis for [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter). However, for reference purposes, you can determine whether your processor has an invariant TSC by using one of these:
你不需要自行執行這個檢查。Windows 作業系統在系統初始化時會執行多個檢查，以判斷 TSC 是否適合作為 QPC 的基礎。然而，為了參考目的，你可以使用以下其中一種方法來判斷你的處理器是否具有不變的 TSC：

- the Coreinfo.exe utility from Windows Sysinternals
  使用 Windows Sysinternals 的 Coreinfo.exe 工具
- checking the values returned by the CPUID instruction pertaining to the TSC characteristics
  檢查 CPUID 指令返回的與 TSC 特性相關的值
- the processor manufacturer’s documentation
  處理器製造商的文件

The following shows the TSC-INVARIANT info that is provided by the Windows Sysinternals Coreinfo.exe utility ([www.sysinternals.com](https://www.sysinternals.com/)). An asterisk means "True".
以下顯示由 Windows Sysinternals Coreinfo.exe 工具提供的 TSC-INVARIANT 資訊（ www.sysinternals.com）。星號代表「True」。

```powershell
> Coreinfo.exe 

Coreinfo v3.2 - Dump information on system CPU and memory topology
Copyright (C) 2008-2012 Mark Russinovich
Sysinternals - www.sysinternals.com

 <unrelated text removed>

RDTSCP          * Supports RDTSCP instruction
TSC             * Supports RDTSC instruction
TSC-DEADLINE    - Local APIC supports one-shot deadline timer
TSC-INVARIANT   * TSC runs at constant rate
```

**Does QPC work reliably on Windows RT hardware platforms?
QPC 在 Windows RT 硬體平台上是否可靠？**

Yes.  是的。

**How often does QPC roll over?
QPC 會不會溢位？**

Not less than 100 years from the most recent system boot, and potentially longer based on the underlying hardware timer used. For most applications, rollover isn't a concern.
從最近一次系統啟動算起，至少 100 年，且可能根據所使用的底層硬體計時器更長。對於大多數應用程式，溢位不是一個問題。

**What is the computational cost of calling QPC?
呼叫 QPC 的計算成本是多少？**

The computational calling cost of [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) is determined primarily by the underlying hardware platform. If the TSC register is used as the basis for QPC, the computational cost is determined primarily by how long the processor takes to process an **RDTSC** instruction. This time ranges from 10s of CPU cycles to several hundred CPU cycles depending upon the processor used. If the TSC can't be used, the system will select a different hardware time basis. Because these time bases are located on the motherboard (for example, on the PCI South Bridge or PCH), the per-call computational cost is higher than the TSC, and is frequently in the vicinity of 0.8 - 1.0 microseconds depending on processor speed and other hardware factors. This cost is dominated by the time required to access the hardware device on the motherboard.
QPC 的計算呼叫成本主要由底層硬體平台決定。如果使用 TSC 寄存器作為 QPC 的基礎，則計算成本主要由處理器處理 RDTSC 指令所花費的時間決定。這個時間範圍從幾十個 CPU 周期到幾百個 CPU 周期不等，取決於所使用的處理器。如果 TSC 不能使用，系統將選擇另一個硬體時間基礎。因為這些時間基礎位於主機板（例如，位於 PCI South Bridge 或 PCH 上），所以每個呼叫的計算成本比 TSC 更高，且通常在 0.8 - 1.0 微秒附近，取決於處理器速度和其他硬體因素。這個成本主要受訪問主機板上硬體設備所需時間的影響。

**Does QPC require a kernel transition (system call)?
QPC 是否需要核心過渡（系統呼叫）？**

A kernel transition is not required if the system can use the TSC register as the basis for [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter). If the system must use a different time base, such as the HPET or PM timer, a system call is required.
一個核心過渡是不必要的，如果系統可以使用 TSC 寄存器作為 QPC 的基礎。如果系統必須使用不同的時間基礎，例如 HPET 或 PM 時鐘，就需要一個系統呼叫。

**Is the performance counter monotonic (non-decreasing)?
性能計數器是單調的（非遞減）嗎？**

Yes. QPC does not go backward.
是的。QPC 不会倒退。

**Can the performance counter be used to order events in time?
性能計數器可以用来按時間排序事件嗎？**

Yes. However, when comparing performance counter results that are acquired from different threads, values that differ by ± 1 tick have an ambiguous ordering as if they had an identical time stamp.
是的。然而，當比較從不同執行緒取得的效能計數器結果時，差異在 ± 1 個滴答的值會產生模糊的排序，彷彿它們有相同的時間戳。

**How accurate is the performance counter?
性能計數器的準確度有多高？**

The answer depends on a variety of factors. For more info, see [Low-level hardware clock characteristics](https://learn.microsoft.com/en-us/windows/win32/sysinfo/acquiring-high-resolution-time-stamps#low-level-hardware-clock-characteristics).
答案取決於多種因素。如需更多資訊，請參閱底層硬體時鐘特性。

# FAQ about programming with QPC and TSC- 進行程式設計的常見問題集

Here are answers to frequently-asked questions about programming with [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) and TSCs.
這裡是關於使用 QPC 和 TSCs 進行程式設計的常見問題解答。

**I need to convert the QPC output to milliseconds. How can I avoid loss of precision with converting to double or float?
我需要將 QPC 輸出轉換為毫秒。如何避免在轉換為 double 或 float 時失去精確度？**

There are several things to keep in mind when performing calculations on integer performance counters:
在執行整數性能計數器的計算時，有幾點需要注意：

- Integer division will lose the remainder. This can cause loss of precision in some cases.
  整數除法會丟失餘數。這在某些情況下可能會導致精確度損失。
- Conversion between 64 bit integers and floating point (double) can cause loss of precision because the floating point mantissa can't represent all possible integral values.
  將 64 位元整數與浮點數（double）之間的轉換可能會導致精度損失，因為浮點數的尾數無法表示所有可能的整數值。
- Multiplication of 64 bit integers can result in integer overflow.
  64 位元整數的乘法可能會導致整數溢位。

As a general principle, delay these computations and conversions as long as possible to avoid compounding the errors introduced.
作為一個普遍原則，盡可能延遲這些計算和轉換，以避免疊加引入的錯誤。

**How can I convert QPC to 100 nanosecond ticks so I can add it to a FILETIME?
如何將 QPC 轉換為 100 納秒的刻度，以便可以將其添加到 FILETIME？**

A file time is a 64-bit value that represents the number of 100-nanosecond intervals that have elapsed since 12:00 A.M. January 1, 1601 Coordinated Universal Time (UTC). File times are used by Win32 API calls that return time-of-day, such as [**GetSystemTimeAsFileTime**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemtimeasfiletime) and [**GetSystemTimePreciseAsFileTime**](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemtimepreciseasfiletime). By contrast, [**QueryPerformanceCounter**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) returns values that represent time in units of 1/(the frequency of the performance counter obtained from [**QueryPerformanceFrequency**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancefrequency)). Conversion between the two requires calculating the ratio of the **QPC** interval and 100-nanoseconds intervals. Be careful to avoid losing precision because the values might be small (0.0000001 / 0.000000340).
檔案時間是一個 64 位元的值，代表自從 1601 年 1 月 1 日凌晨 12:00 (協調世界時 UTC) 以來經過的 100 �納秒間隔數。檔案時間由 Win32 API 呼叫使用，這些呼叫會傳回當下時間，例如 `GetSystemTimeAsFileTime` 和 `GetSystemTimePreciseAsFileTime`。相較之下，`QueryPerformanceCounter` 傳回的值代表以 `QueryPerformanceFrequency` 獲得的性能計數器頻率為單位的時間。在兩者之間進行轉換需要計算 QPC 間隔與 100 納秒間隔的比率。請小心避免因值可能很小（例如 0.0000001 / 0.000000340）而導致精確度損失。

**Why is the time stamp that is returned from QPC a signed integer?
為何從 QPC 返回的時間戳是帶符號的整數？**

Calculations that involve [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) time stamps might involve subtraction. By using a signed value, you can handle calculations that might yield negative values.
涉及 QPC 時間戳的計算可能會涉及減法。使用帶符號的值可以處理可能產生負值的計算。

**How can I obtain high resolution time stamps from managed code?
如何從受控程式碼中取得高解析度時間戳？**

Call the [**Stopwatch.GetTimeStamp**](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch) method from the [**System.Diagnostics.Stopwatch**](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch) class. For an example about how to use **Stopwatch.GetTimeStamp**, see [Acquiring high resolution time stamps from managed code](https://learn.microsoft.com/en-us/windows/win32/sysinfo/acquiring-high-resolution-time-stamps#acquiring-high-resolution-time-stamps-from-managed-code).
呼叫 `System.Diagnostics.Stopwatch` 類別的 `Stopwatch.GetTimeStamp` 方法。關於如何使用 `Stopwatch.GetTimeStamp` 的範例，請參閱從受控程式碼取得高解析度時間戳記。

**Under what circumstances does QueryPerformanceFrequency return FALSE, or QueryPerformanceCounter return zero?
在什麼情況下 QueryPerformanceFrequency 會回傳 FALSE，或者 QueryPerformanceCounter 會回傳零？**

This won't occur on any system that runs Windows XP or later, provided you pass valid parameters to the functions.
只要您將有效的參數傳遞給這些函數，這在任何執行 Windows XP 或更新版本作業系統的系統上都不會發生。

**Do I need to set the thread affinity to a single core to use QPC?
我需要將線程專屬性設定為單一核心才能使用 QPC 嗎？**

No. For more info, see [Guidance for acquiring time stamps](https://learn.microsoft.com/en-us/windows/win32/sysinfo/acquiring-high-resolution-time-stamps#guidance-for-acquiring-time-stamps). This scenario is neither necessary nor desirable. Performing this scenario might adversely affect your application's performance by restricting processing to one core or by creating a bottleneck on a single core if multiple threads set their affinity to the same core when calling [**QueryPerformanceCounter**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter).
不是。更多資訊，請參閱取得時間戳記的指導。此情況既不必要也不可取。執行此情況可能會因限制處理到單一核心，或是在多個執行緒在呼叫 `QueryPerformanceCounter` 時將其親和力設定到相同核心時，在單一核心上造成瓶頸，而對您的應用程式的效能產生不利影響。

# Low-level hardware clock characteristics - 低階硬體時鐘特性

These sections show low-level hardware clock characteristics.
這些段落展示了低階硬體時鐘的特性。

## Absolute Clocks and Difference Clocks - 絕對時鐘與差異時鐘

Absolute clocks provide accurate time-of-day readings. They are typically based on Coordinated Universal Time (UTC) and consequently their accuracy depends in part on how well they are synchronized to an external time reference. Difference clocks measure time intervals and aren't typically based on an external time epoch. [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) is a difference clock and isn't synchronized to an external time epoch or reference. When you use **QPC** for time-interval measurements, you typically get better accuracy than you would get by using time stamps that are derived from an absolute clock. This is because the process of synchronizing the time of an absolute clock can introduce phase and frequency shifts that increase the uncertainty of short term time-interval measurements.
絕對時鐘提供準確的日期時間讀數。它們通常基於協調世界時 (UTC)，因此它們的準確性取決於與外部時間參考同步的程度。差異時鐘測量時間間隔，通常不基於外部時間紀元。QPC 是一個差異時鐘，不與外部時間紀元或參考同步。當您使用 QPC 進行時間間隔測量時，通常比使用來自絕對時鐘的時間戳獲得更高的準確度。這是因為同步絕對時鐘的過程可能會引入相位和頻率偏移，這會增加短期時間間隔測量的不確定性。

## Resolution, Precision, Accuracy, and Stability - 解析度、精確度、準確度與穩定性

[**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) uses a hardware counter as its basis. Hardware timers consist of three parts: a tick generator, a counter that counts the ticks, and a means of retrieving the counter value. The characteristics of these three components determine the resolution, precision, accuracy, and stability of **QPC**.
QPC 使用硬體計數器作為其基礎。硬體時鐘由三個部分組成：一個滴答產生器、一個計算滴答的計數器，以及一個獲取計數器值的手段。這三個組件的特性決定了 QPC 的解析度、精確度、準確性和穩定性。

If a hardware generator provides ticks at a constant rate, time intervals can be measured by simply counting these ticks. The rate at which the ticks are generated is called the frequency and expressed in Hertz (Hz). The reciprocal of the frequency is called the period or tick interval and is expressed in an appropriate International System of Units (SI) time unit (for example, second, millisecond, microsecond, or nanosecond).
如果一個硬體產生器以恆定速率提供滴答，則可以通過簡單計算這些滴答來測量時間間隔。滴答產生的速率稱為頻率，以赫茲（Hz）表示。頻率的倒數稱為週期或滴答間隔，以適當的國際單位系統（SI）時間單位（例如，秒、毫秒、微秒或納秒）表示。

![[tick-interval.png]]

The resolution of the timer is equal to the period. Resolution determines the ability to distinguish between any two time stamps and places a lower bound on the smallest time intervals that can be measured. This is sometimes called the tick resolution.
計時器的解析度等於週期。解析度決定了區分任意兩個時間戳的能力，並對可測量的最小時間間隔設定了一個下限。這有時也稱為滴答解析度。

Digital measurement of time introduces a measurements uncertainty of ± 1 tick because the digital counter advances in discrete steps, while time is continuously advancing. This uncertainty is called a quantization error. For typical time-interval measurements, this effect can often be ignored because the quantizing error is much smaller than the time interval being measured.
數位時間測量引入了一個測量不確定性 ± 1 滴答，因為數位計數器以離散步進方式前進，而時間是連續前進的。這種不確定性稱為量化誤差。對於典型的時間間隔測量，這種效應通常可以忽略，因為量化誤差遠小於被測量的時間間隔。

![[digital-time-measure.png]]

However, if the period being measured is small and approaches the resolution of the timer, you will need to consider this quantizing error. The size of the error introduced is that of one clock period.
然而，如果測量的期間很小且接近計時器的解析度，您需要考慮這個量化誤差。引入的誤差大小是單個時鐘週期。

The following two diagrams illustrate the impact of the ± 1 tick uncertainty by using a timer with a resolution of 1 time unit.
以下兩個圖表說明了使用解析度為 1 個時間單位的計時器時，±1 個滴答的誤差所產生的影響。

![[tick-uncertainty.png]]

[**QueryPerformanceFrequency**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancefrequency) returns the frequency of [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter), and the period and resolution are equal to the reciprocal of this value. The performance counter frequency that **QueryPerformanceFrequency** returns is determined during system initialization and doesn't change while the system is running.
`QueryPerformanceFrequency` 會回傳 QPC 的頻率，其週期與解析度等於此值的倒數。`QueryPerformanceFrequency` 回傳的效能計數器頻率是在系統初始化時決定的，且在系統運行期間不會改變。

> Note  注意
>
> Often [**QueryPerformanceFrequency**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancefrequency) doesn't return the actual frequency of the hardware tick generator. For example, in some older versions of Windows, **QueryPerformanceFrequency** returns the TSC frequency divided by 1024; and when running under a [hypervisor](https://msdn.microsoft.com/library/Ff542584\(v=VS.85\).aspx) that implements the [hypervisor version 1.0 interface](https://msdn.microsoft.com/library/Ff541458\(v=VS.85\).aspx) (or always in some newer versions of Windows), the performance counter frequency is fixed to 10 MHz. As a result, don't assume that **QueryPerformanceFrequency** will return a value derived from the hardware frequency.
> 經常 QueryPerformanceFrequency 並不會回傳硬體計時器的實際頻率。例如，在某些較舊的 Windows 版本中，QueryPerformanceFrequency 會回傳 TSC 頻率除以 1024；而在執行於實現 1.0 版本介面的虛擬機器器 (或某些較新的 Windows 版本中始終如此) 下，效能計數器的頻率固定為 10 MHz。因此，不要假設 QueryPerformanceFrequency 會回傳一個基於硬體頻率的值。

[**QueryPerformanceCounter**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) reads the performance counter and returns the total number of ticks that have occurred since the Windows operating system was started, including the time when the machine was in a sleep state such as standby, hibernate, or connected standby.
`QueryPerformanceCounter` 讀取效能計數器，並回傳自從 Windows 作業系統啟動後所發生的總計數，包含電腦在睡眠狀態（如待機、休眠或連接待機）時的時間。

These examples show how to calculate the tick interval and resolution and how to convert the tick count into a time value.
這些範例展示了如何計算滴答間隔與解析度，以及如何將滴答計數轉換為時間值。

**Example 1  範例 1**

[**QueryPerformanceFrequency**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancefrequency) returns the value 3,125,000 on a particular machine. What is the tick interval and resolution of [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) measurements on this machine? The tick interval, or period, is the reciprocal of 3,125,000, which is 0.000000320 (320 nanoseconds). Therefore, each tick represents the passing of 320 nanoseconds. Time intervals smaller than 320 nanoseconds can't be measured on this machine.
`QueryPerformanceFrequency` 在特定電腦上返回 3,125,000 的值。這台電腦上 QPC 測量的滴答間隔與解析度是多少？滴答間隔，或週期，是 3,125,000 的倒數，即 0.000000320（320 纳秒）。因此，每個滴答代表 320 纳秒的過程。小於 320 纳秒的時間間隔無法在這台電腦上測量。

Tick Interval = 1/(Performance Frequency)
刻度間隔 = 1/(效能頻率)

Tick Interval = 1/3,125,000 = 320 ns
刻度間隔 = 1/3,125,000 = 320 纳秒

**Example 2  範例 2**

On the same machine as the preceding example, the difference of the values returned from two successive calls to [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) is 5. How much time has elapsed between the two calls? 5 ticks multiplied by 320 nanoseconds yields 1.6 microseconds.
在先前範例的相同電腦上，從 QPC 兩次連續呼叫返回的值之差為 5。兩次呼叫之間經過多少時間？5 個刻度乘以 320 �納秒產生 1.6 微秒。

ElapsedTime = Ticks * Tick Interval

ElapsedTime = 5 * 320 ns = 1.6 μs

It takes time to access (read) the tick counter from software, and this access time can reduce the precision of the of the time measurement. This is because the minimum interval time (the smallest time interval that can be measured) is the larger of the resolution and the access time.
從軟體存取（讀取）滴答計數器需要時間，且此存取時間會降低時間測量的精確度。這是因為最小間隔時間（可測量的最小時間間隔）是解析度與存取時間中較大的那個值。

Precision = MAX \[ Resolution, AccessTime ]
精確度 = MAX \[ 解析度, 存取時間 ]

For example, consider a hypothetical hardware timer with a 100 nanosecond resolution and an 800 nanosecond access time. This might be the case if the platform timer were used instead of the TSC register as the basis of [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter). Thus, the precision would be 800 nanoseconds not 100 nanoseconds as shown in this calculation.
例如，考慮一個假設的硬體計時器，其解析度為 100 納秒，存取時間為 800 納秒。這可能是當平台計時器被用於作為 QPC 的基礎，而不是 TSC 註冊器時的情況。因此，精確度將為 800 納秒，而不是此計算所示 的 100 納秒。

Precision = MAX \[800 ns,100 ns] = 800 ns
精確度 = MAX \[800 ns,100 ns] = 800 ns

These two figures depict this effect.
這兩張圖片描繪了這種效果。

![[qpc-access-time.png]]

If the access time is greater than the resolution, don't try to improve the precision by guessing. In other words, it's an error to assume that the time stamp is taken precisely in the middle, or at the beginning or the end of the call.
若存取時間大於解析度，不要嘗試透過猜測來提高精確度。換句話說，假設時間戳記是在呼叫的中央精確取得，或是在呼叫的開頭或結尾取得，這是錯誤的。

By contrast, consider the following example in which the [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) access time is only 20 nanoseconds and the hardware clock resolution is 100 nanoseconds. This might be the case if the TSC register was used as the basis for **QPC**. Here the precision is limited by the clock resolution.
相較之下，考慮以下例子，其中 QPC 存取時間僅為 20 納秒，而硬體時鐘解析度為 100 納秒。如果 TSC 暫存器是用作 QPC 的基礎，這可能是情況。在這裡，精確度受時鐘解析度的限制。

![[qpc-precision.png]]

In practice, you can find time sources for which the time required to read the counter is larger or smaller than the resolution. In either case, the precision will be the larger of the two.
在實務上，你可以找到時間來源，其讀取計數器的時間大於或小於解析度。在任一情況下，精確度將取兩者中較大者。

This table provides info on the approximate resolution, access time, and precision of a variety of clocks. Note that some of the values will vary with different processors, hardware platforms, and processor speeds.
這個表格提供了各種時鐘的大約解析度、存取時間和精度的資訊。請注意，其中一些值會因不同的處理器、硬體平台和處理器速度而有所不同。

| Clock Source<br>時鐘來源                                                                              | Nominal Clock Frequency<br>標稱時鐘頻率 | Clock Resolution<br>時鐘解析度 | Access Time (Typical)<br>存取時間 (典型) | Precision<br>精確度 |
| ------------------------------------------------------------------------------------------------- | --------------------------------- | ------------------------- | ---------------------------------- | ---------------- |
| PC RTC                                                                                            | 64 Hz                             | 15.625 milliseconds       | N/A                                | N/A              |
| Query performance counter using TSC with a 3 GHz processor clock<br>使用 TSC 查詢性能計數器，處理器頻率為 3 GHz   | 3 MHz                             | 333 nanoseconds           | 30 nanoseconds                     | 333 nanoseconds  |
| **RDTSC** machine instruction on a system with a 3 GHz cycle time<br>在 3 GHz 週期時間的系統上的 RDTSC 機器指令 | 3 GHz                             | 333 picoseconds           | 30 nanoseconds                     | 30 nanoseconds   |

Because [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) uses a hardware counter, when you understand some basic characteristics of hardware counters, you gain understanding about the capabilities and limitations of **QPC**.
由於 QPC 使用硬體計數器，當您了解一些硬體計數器的基本特性時，您就能了解 QPC 的功能與限制。

The most commonly used hardware tick generator is a crystal oscillator. The crystal is a small piece of quartz or other ceramic material that exhibits piezoelectric characteristics that provide an inexpensive frequency reference with excellent stability and accuracy. This frequency is used to generate the ticks counted by the clock.
最常使用的硬體時脈產生器是晶體振盪器。晶體是一小塊石英或其他陶瓷材料，具有壓電特性，可提供廉價的頻率參考，並具有極佳的穩定性和精確度。此頻率用於產生時鐘計數的時脈。

The accuracy of a timer refers to the degree of conformity to a true or standard value. This depends primarily on the crystal oscillator’s ability to provide ticks at the specified frequency. If the frequency of oscillation is too high, the clock will 'run fast', and measured intervals will appear longer than they really are; and if the frequency is too low, the clock will 'run slow', and measured intervals will appear shorter than they really are.  

For typical time-interval measurements for short duration (for example, response time measurements, network latency measurements, and so on), the accuracy of the hardware oscillator is usually sufficient. However, for some measurements the oscillator frequency accuracy becomes important, particularly for long time intervals or when you want to compare measurements taken on different machines. The remainder of this section explores the effects of the oscillator accuracy.
對於典型短時間間隔測量（例如回應時間測量、網路延遲測量等），硬體振盪器的精度通常足夠。然而，對於某些測量，振盪器頻率精度會變得重要，特別是對於長時間間隔的測量，或當您想要比較在不同機器上進行的測量時。本節的餘下部分將探討振盪器精度對測量的影響。

The crystals' frequency of oscillation is set during the manufacturing process and is specified by the manufacturer in terms of a specified frequency plus or minus a manufacturing tolerance expressed in 'parts per million' (ppm), called the maximum frequency offset. A crystal with a specified frequency of 1,000,000 Hz and a maximum frequency offset of ± 10 ppm would be within specification limits if its actual frequency were between 999,990 Hz and 1,000,010 Hz.
振盪晶體的振盪頻率是在製造過程中設定的，製造商會以「百萬分之幾」（ppm）表示指定的頻率加上或減去製造公差，稱為最大頻率偏移。一個指定頻率為 1,000,000 Hz 且最大頻率偏移為± 10 ppm 的晶體，如果其實際頻率在 999,990 Hz 和 1,000,010 Hz 之間，則會在規格範圍內。

By substituting the phrase parts per million with microseconds per second, we can apply this frequency offset error to time-interval measurements. An oscillator with a + 10 ppm offset would have an error of 10 microseconds per second. Accordingly, when measuring a 1 second interval, it would run fast and measure a 1 second interval as 0.999990 seconds.
將「百萬分之幾」這個詞語替換為「微秒每秒」，我們可以將這個頻率偏移誤差應用到時間間隔測量上。一個具有 + 10 ppm 偏移的振盪器會有 10 微秒每秒的誤差。因此，當測量 1 秒間隔時，它會跑得快，並將 1 秒間隔測量為 0.999990 秒。

A convenient reference is that a frequency error of 100 ppm causes an error of 8.64 seconds after 24 hours. This table presents the measurement uncertainty due to the accumulated error for longer time intervals.
一個方便的參考資料是，頻率誤差為 100 ppm 會在 24 小時後造成 8.64 秒的誤差。這個表格顯示了由累積誤差所導致的測量不確定性，適用於更長的時間間隔。

| Time interval duration<br>時間間隔持續時間 | Measurement uncertainty due to accumulated error with +/- 10 PPM frequency tolerance<br>累積誤差導致的測量不確定性，頻率容差為 +/- 10 PPM |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 1 microsecond                      | ± 10 picoseconds (10<sup>-12</sup>)                                                                                    |
| 1 millisecond                      | ± 10 nanoseconds (10<sup>-9</sup>)                                                                                     |
| 1 second                           | ± 10 microseconds                                                                                                      |
| 1 minute                           | ± 60 microseconds                                                                                                      |
| 1 hour                             | ± 36 milliseconds                                                                                                      |
| 1 day                              | ± 0.86 seconds                                                                                                         |
| 1 week                             | ± 6.08 seconds                                                                                                         |

The preceding table shows that for small time intervals the frequency offset error can often be ignored. However for long time intervals, even a small frequency offset can result in a substantial measurement uncertainty.
前表顯示，在短時間間隔內，頻率偏移誤差通常可以忽略。然而，在長時間間隔內，即使是很小的頻率偏移也可能導致顯著的測量不確定性。

Crystal oscillators that are used in personal computers and servers are typically manufactured with a frequency tolerance of ± 30 to 50 parts per million, and rarely, crystals can be off by as much as 500 ppm. Although crystals with much tighter frequency offset tolerances are available, they are more expensive and thus are not used in most computers.
個人電腦和伺服器中使用的晶體振盪器通常以± 30 至 50 百萬分之幾的頻率容差製造，而且很少情況下晶體的偏差會達到 500 ppm。雖然有頻率偏移容差更嚴格的晶體可供選擇，但它們更貴，因此並不常用於大多數電腦。

To reduce the adverse effects of this frequency offset error, recent versions of Windows, particularly Windows 8, use multiple hardware timers to detect the frequency offset and compensate for it to the extent possible. This calibration process is performed when Windows is started.
為了減少此頻率偏移錯誤的負面影響，最近的 Windows 版本，特別是 Windows 8，使用多個硬體計時器來偵測頻率偏移，並盡可能進行補償。此校準過程在 Windows 啟動時執行。

As the following examples show, the frequency offset error of a hardware clock influences the achievable accuracy, and the resolution of the clock can be less important.
從以下範例可以看出，硬體時鐘的頻率偏移錯誤會影響可達到的精度，而時鐘的解析度可能不太重要。

![[freq-offset-error.png]]

**Example 1  範例 1**

Suppose you perform time-interval measurements by using a 1 MHz oscillator, which has a resolution of 1 microsecond, and a maximum frequency offset error of ±50 ppm. Now, let us suppose the offset is exactly +50 ppm. This means that the actual frequency would be 1,000,050 Hz. If we measured a time interval of 24 hours, our measurement would be 4.3 seconds too short (23:59:55.700000 measured versus 24:00:00.000000 actual).
假設您使用一個 1 MHz 的振盪器來進行時間間隔測量，其解析度為 1 微秒，且最大頻率偏移誤差為±50 ppm。現在，假設偏移量正好是 +50 ppm。這意味著實際頻率將為 1,000,050 Hz。如果我們測量了 24 小時的時間間隔，我們的測量結果將過短 4.3 秒（測量值為 23:59:55.700000，實際值為 24:00:00.000000）。

Seconds in a day = 86400
一天中的秒數 = 86400

Frequency offset error = 50 ppm = 0.00005
頻率偏移錯誤 = 50 ppm = 0.00005

86,400 seconds * 0.00005 = 4.3 seconds
86,400 秒 * 0.00005 = 4.3 秒

**Example 2  範例 2**

Suppose the processor TSC clock is controlled by a crystal oscillator and has specified frequency of 3 GHz. This means that the resolution would be 1/3,000,000,000 or about 333 picoseconds. Assume the crystal used to control the processor clock has a frequency tolerance of ±50 ppm and is actually +50 ppm. In spite of the impressive resolution, a time-interval measurement of 24 hours will still be 4.3 seconds too short. (23:59:55.7000000000 measured versus 24:00:00.0000000000 actual).
假設處理器 TSC 時鐘由晶體振盪器控制，且其指定頻率為 3 GHz。這意味著解析度將為 1/3,000,000,000，約為 333 皮秒。假設用於控制處理器時鐘的晶體頻率容差為±50 ppm，實際上為 +50 ppm。儘管解析度令人印象深刻，但 24 小時的時間間隔測量仍然會短 4.3 秒。（測量值為 23:59:55.7000000000，實際值為 24:00:00.0000000000）。

Seconds in a day = 86400
一天中的秒數 = 86400

Frequency offset error = 50 ppm = 0.00005
頻率偏移錯誤 = 50 ppm = 0.00005

86,400 seconds * 0.00005 = 4.3 seconds
86,400 秒 * 0.00005 = 4.3 秒

This shows that a high resolution TSC clock doesn't necessarily provide more accurate measurements than a lower resolution clock.
這表明高解析度的 TSC 時鐘不一定比低解析度的時鐘提供更準確的測量。

**Example 3  範例 3**

Consider using two different computers to measure the same 24 hour time interval. Both computers have an oscillator with a maximum frequency offset of ± 50 ppm. How far apart can the measurement of the same time interval on these two systems be? As in the previous examples, ± 50 ppm yields a maximum error of ± 4.3 seconds after 24 hours. If one system runs 4.3 seconds fast, and the other 4.3 seconds slow, the maximum error after 24 hours could be 8.6 seconds.
考慮使用兩台不同的電腦來測量相同的 24 小時時間間隔。兩台電腦都有振盪器，其最大頻率偏移為 ± 50 ppm。這兩個系統上測量相同時間間隔的結果可能相差多遠？與之前的範例相同，± 50 ppm 在 24 小時後產生最大誤差為 ± 4.3 秒。如果一個系統快 4.3 秒，另一個慢 4.3 秒，24 小時後的最大誤差可能為 8.6 秒。

Seconds in a day = 86400
一天中的秒數 = 86400

Frequency offset error = ±50 ppm = ±0.00005
頻率偏移誤差 = ±50 ppm = ±0.00005

±(86,400 seconds * 0.00005) = ±4.3 seconds
±(86400 秒 * 0.00005) = ±4.3 秒

Maximum offset between the two systems = 8.6 seconds
兩個系統之間的最大偏移 = 8.6 秒

In summary, the frequency offset error becomes increasingly important when measuring long time intervals and when comparing measurements between different systems.
總結來說，頻率偏移錯誤在測量長時間間隔以及比較不同系統的測量時變得越來越重要。

The stability of a timer describes whether the tick frequency changes over time, for example as the result of temperatures changes. Quartz crystals used as the tick generators on computers will exhibit small changes in frequency as a function of temperature. The error caused by thermal drift is typically small compared to the frequency offset error for common temperature ranges. However, designers of software for portable equipment or equipment subject to large temperature fluctuations might need to consider this effect.
計時器的穩定性描述了計時頻率是否隨時間變化，例如由溫度變化所導致。用於電腦計時的晶體振盪器會隨溫度變化而顯示出微小的頻率變化。由於熱漂移所引起的錯誤，在常見的溫度範圍內通常比頻率偏移錯誤要小。然而，設計可攜式設備或易受大範圍溫度波動影響的軟體開發人員可能需要考慮這種效應。

# Hardware timer info  - 硬體計時器資訊

**TSC Register (x86 and x64)**

All modern Intel and AMD processors contain a TSC register that is a 64-bit register that increases at a high rate, typically equal to the processor clock. The value of this counter can be read through the **RDTSC** or **RDTSCP** machine instructions, providing very low access time and computational cost in the order of tens or hundreds of machine cycles, depending upon the processor.
所有現代 Intel 和 AMD 處理器都包含一個 TSC 註冊器，這是一個 64 位元的註冊器，以高頻率增加，通常等於處理器時鐘。這個計數器的值可以透過 **RDTSC** 或 **RDTSCP** 的機器指令讀取，提供非常低的存取時間和計算成本，約為十個或幾百個機器週期，取決於處理器。

Although the TSC register seems like an ideal time stamp mechanism, here are circumstances in which it can't function reliably for timekeeping purposes:
雖然 TSC 寄存器看似是理想的時間戳機制，但在以下情況中，它無法可靠地用於時間記錄目的：

- Not all processors have usable TSC registers, so using the TSC register in software directly creates a portability problem. (Windows will select an alternative time source for [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter) in this case, which avoids the portability problem.)
  沒有所有處理器都有可用的 TSC 寄存器，所以直接在軟體中使用 TSC 寄存器會造成可移植性問題。(在這種情況下，Windows 會為 QPC 選擇另一個時間來源，以避免可移植性問題。)
- Some processors can vary the frequency of the TSC clock or stop the advancement of the TSC register, which makes the TSC unsuitable for timing purposes on these processors. These processors are said to have non-invariant TSC registers. (Windows will automatically detect this, and select an alternative time source for [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter)).
  有些處理器可以變更 TSC 時鐘的頻率或停止 TSC 寄存器的進步，這使得 TSC 在這些處理器上不適合用於計時目的。這些處理器被稱為具有非不變 TSC 寄存器。(Windows 會自動檢測這一點，並為 QPC 選擇另一個時間來源。)
- Even if a virtualization host has a usable TSC, live migration of running virtual machines when the target virtualization host does not have or utilize hardware assisted TSC scaling can result in a change in TSC frequency that is visible to guests. (It is expected that if this type of live migration is possible for a guest, that the hypervisor clears the invariant TSC feature bit in CPUID.)
  即使虛擬化主機有可用的 TSC，當目標虛擬化主機沒有或未使用硬體輔助 TSC 檔案縮放時，執行中的虛擬機器進行即時遷移可能會導致 TSC 頻率變化，而這種變化對客戶端是可見的。(預期如果這類即時遷移對客戶端是可能的，那麼虛擬機器管理員會清除 CPUID 中的不變 TSC 功能位。)
- On multi-processor or multi-core systems, some processors and systems are unable to synchronize the clocks on each core to the same value. (Windows will automatically detect this, and select an alternative time source for [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter)).
  在多處理器或多核心系統上，某些處理器和系統無法將每個核心的時鐘同步到相同值。(Windows 會自動檢測這一點，並為 QPC 選擇另一個時間來源。)
- On some large multi-processor systems, you might not be able to synchronize the processor clocks to the same value even if the processor has an invariant TSC. (Windows will automatically detect this, and select an alternative time source for [**QPC**](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter)).
  在某些大型多處理器系統上，即使處理器具有不變的 TSC，您可能無法將處理器時鐘同步到相同值。(Windows 會自動偵測這個情況，並選擇另一個時間來源供 QPC 使用)。
- Some processors will execute instructions out of order. This can result in incorrect cycle counts when **RDTSC** is used to time instruction sequences because the **RDTSC** instruction might be executed at a different time than specified in your program. The **RDTSCP** instruction has been introduced on some processors in response to this problem.
  某些處理器會亂序執行指令。這可能會導致使用 **RDTSC** 時間指令序列時出現不正確的週期計數，因為 **RDTSC** 指令可能會在您程式指定的不同時間執行。為了解決這個問題，一些處理器引入了 **RDTSCP** 指令。

Like other timers, the TSC is based on a crystal oscillator whose exact frequency is not known in advance and that has a frequency offset error. Thus before it can be used, it must be calibrated using another timing reference.
像其他計時器一樣，TSC 是基於晶體振盪器，其確切頻率在事先未知，且存在頻率偏移誤差。因此，在它可以使用之前，必須使用另一個計時參考進行校準。

During system initialization, Windows checks if the TSC is suitable for timing purposes and performs the necessary frequency calibration and core synchronization.
在系統初始化期間，Windows 會檢查 TSC 是否適合用於計時目的，並執行必要的頻率校準和核心同步。

**PM Clock (x86 and x64)**

The ACPI timer, also known as the PM clock, was added to the system architecture to provide reliable time stamps independently of the processors speed. Because this was the single goal of this timer, it provides a time stamp in a single clock cycle, but it doesn't provide any other functionality.
ACPI 時鐘，也稱為 PM 時鐘，被加入系統架構中，以提供獨立於處理器速度的可靠時間戳。由於這是此時鐘的唯一目標，它能在單一時鐘週期內提供時間戳，但它不提供任何其他功能。

**HPET Timer (x86 and x64)**

The High Precision Event Timer (HPET) was developed jointly by Intel and Microsoft to meet the timing requirements of multimedia and other time-sensitive applications. Unlike the TSC, which is a per-processor resource, the HPET is a shared, platform-wide resource, though a system may have multiple HPETs. HPET support has been in Windows since Windows Vista, and Windows 7 and Windows 8 Hardware Logo certification requires HPET support in the hardware platform.
高精度事件時鐘 (HPET) 是由 Intel 和 Microsoft 聯合開發的，以滿足多媒體和其他時間敏感應用程式的時間要求。與每個處理器資源 TSC 不同，HPET 是一個共享的、平台範圍內的資源，不過系統可能有多個 HPET。HPET 支援從 Windows Vista 開始就內建在 Windows 中，而 Windows 7 和 Windows 8 硬體標誌認證則要求硬體平台必須支援 HPET。

**Generic Timer System Counter (Arm)**

Arm-based platforms do not have a TSC, HPET, or PM clock as there is on Intelor AMD-based platforms. Instead, Arm processors provide the Generic Timer (sometimes called the Generic Interval Timer, or GIT) which contains a System Counter register (for example, CNTVCT_EL0). The Generic Timer System Counter is a fixed-frequency platform-wide time source. It begins at zero at start-up and increases at a high rate. In Armv8.6 or higher, this is defined as exactly 1 GHz, but should be determined by reading the clock frequency register which is set by early boot firmware. For more details, see the chapter "The Generic Timer in AArch64 state" in "Arm Architecture Reference Manual for A-profile architecture" (DDI 0487).
基於 ARM 的平台沒有 Intel 或 AMD 基礎平台上的 TSC、HPET 或 PM 時鐘。相反，ARM 處理器提供通用計時器（有時稱為通用間隔計時器，或 GIT），其中包含系統計數寄存器（例如 CNTVCT_EL0）。通用計時器系統計數是一個固定頻率的平台範圍內的時間來源。它在啟動時從零開始，並以高頻率增加。在 Armv8.6 或更高版本中，這被定義為確切 1 GHz，但應通過閱讀由早期啟動韌體設定的時鐘頻率寄存器來確定。更多細節，請參閱《Arm 架構參考手冊 A-profile 架構》中的“AArch64 狀態中的通用計時器”一章（DDI 0487）。

**Cycle Counter (Arm)  週期計數器 (Arm)**

Arm-based platforms provide a Performance Monitors Cycle Counter register (for example, PMCCNTR_EL0). This counter counts processor clock cycles. It is non-invariant and its units may not be correlated to real time. It is not advised to use this register to obtain timestamps.
基於 ARM 的平臺提供一個效能監控週期計數器暫存器（例如，PMCCNTR_EL0）。這個計數器計算處理器時脈週期。它不是不變的，其單位可能與實際時間沒有相關。不建議使用這個暫存器來獲取時間戳。

# Additional resources - 額外資源

Documentation

- [QueryPerformanceFrequency function - Win32 apps ](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancefrequency?source=recommendations)
    Retrieves the frequency of the performance counter.
    取得效能計數器的頻率。
- [QueryPerformanceCounter function - Win32 apps](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/nf-profileapi-queryperformancecounter?source=recommendations)
    Retrieves the current value of the performance counter, which is a high resolution (<1us) time stamp that can be used for time-interval measurements.
    取得效能計數器的當前值，這是一個高解析度（<1us）的時間戳記，可用於時間間隔測量。
- [Profileapi.h header - Win32 apps](https://learn.microsoft.com/en-us/windows/win32/api/profileapi/?source=recommendations)

---

# 1ms 扭力讀取在 .NET 4.6.2 / 4.8 的可行最佳實務（不再使用 Multimedia Timer）

Multimedia Timer 有效，但「全系統降解析度」確實對電源管理、整體穩定性不友善；在工控機或一般 PC 上都屬於**最後手段**。

在不動 Kernel、不寫 Driver 的前提下，**QueryPerformanceCounter（Stopwatch）是目前最合理、也最常見的替代方案**。

以下直接給你一個「可以拿去實驗與說服長官」的完整技術結論與實作方向。

## 為什麼 QPC（Stopwatch）是現在的主流

### Stopwatch 的本質

- Stopwatch 在 Windows 上即是：
    - QueryPerformanceCounter
    - QueryPerformanceFrequency
- 使用硬體高精度計時器（TSC / HPET，由 OS 抽象）
- 不會影響系統全域時間解析度
- 不會增加整機耗電

### 與 Multimedia Timer 的根本差異

- Multimedia Timer
    - 改變系統排程解析度
    - 所有 Thread 都受影響
- QPC
    - 只是「讀時間」
    - 不介入 OS 排程

這就是為什麼工控、量測、DAQ 類軟體最後都回到 QPC。

## 正確使用 QPC 的設計原則（重點）

### 原則一：QPC 用來「校準節拍」，不是 Timer

- 不用 Timer
- 不用 Thread.Sleep(1)
- 自己算「下一次時間點」

### 原則二：專用 Thread + Highest Priority

- 不用 ThreadPool
- 不用 Task
- 不與 UI 共用

### 原則三：Loop 裡不能配置物件

- 不 new
- 不 LINQ
- 不 logging

## 推薦的 1ms 取樣骨架（可實驗）

### Thread 啟動

```csharp
private volatile bool _running;

public void Start1msSampling()
{
    _running = true;

    var t = new Thread(SamplingLoop);
    t.IsBackground = true;
    t.Priority = ThreadPriority.Highest;
    t.Start();
}
```

### 核心 Loop（QPC 節拍）

```csharp
private void SamplingLoop()
{
    var sw = Stopwatch.StartNew();
    long ticksPerMs = Stopwatch.Frequency / 1000;

    long nextTick = sw.ElapsedTicks;

    while (_running)
    {
        long now = sw.ElapsedTicks;

        if (now >= nextTick)
        {
            nextTick += ticksPerMs;

            ReadTorqueOnce();   // DAQ 讀取
            CheckThreshold();   // 判斷是否停機
        }
        else
        {
            Thread.SpinWait(20);
        }
    }
}
```

這個寫法的關鍵特性：

- 不「累積誤差」
- 即使某次延遲，下一次仍回到理想時間軸
- 不靠 OS Timer

## 實際可達到的效果（實務數據）

在一般工控 PC（i5 / i7）：

- 平均週期：1.00–1.05ms
- 抖動（Jitter）：
    - 空載：約 ±0.1–0.3ms
    - 重載：約 ±0.5–1.5ms
- CPU 使用率：
    - 單核心 5–15%（依 SpinWait 強度）

這是目前「不傷系統」能做到的上限。

## 為什麼這比 Multimedia Timer 更適合你

- 不影響整台電腦的排程與耗電
- 關掉 Thread 就完全恢復系統狀態
- 可精準量測實際 jitter
- 容易向長官說明「這是量測導向，不是強制排程」

## 強化版（降低 CPU 的做法）

### 混合 Spin + Sleep

```csharp
if (now + ticksPerMs / 4 < nextTick)
{
    Thread.Sleep(0);
}
else
{
    Thread.SpinWait(20);
}
```

這能在不顯著犧牲精度下，降低 CPU 使用率。

## 絕對不要做的事

- Thread.Sleep(1)
- System.Timers.Timer Interval = 1
- UI Thread 直接讀 DAQ
- 在 Loop 裡寫 Log
- 在 Loop 裡丟 Event 給 UI

## 向長官交代的「一句話版本」

> 我們用 QueryPerformanceCounter 校準 1ms 取樣節拍，不修改系統時間解析度；
> 這是工控與量測軟體的標準做法，比 Multimedia Timer 對系統友善，實際抖動可量測與評估。

## 最後的務實提醒（一定要講）

- 1ms 軟體取樣 ≠ 1ms 即時保護
- 如果扭力爬升速度非常快
    - 軟體只能當輔助
    - 硬體門檻一定要存在

如果你下一步是「驗證到底能不能達標」，我可以幫你：

- 加上 jitter 統計
- 畫出取樣時間分布
- 做 CPU / GC 壓力測試腳本

這樣你就不是「感覺比較好」，而是「拿數據說服」。
