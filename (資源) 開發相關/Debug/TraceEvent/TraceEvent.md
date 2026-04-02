---
aliases:
date:
update:
author:
language:
sourceurl: https://github.com/Biswa96/TraceEvent
tags:
---

Trace Events with real time sessions using (un)documented Windows APIs and NT APIs.
使用（未）公開的 Windows API 和 NT API 透過即時會話追蹤事件。

# What is Event Tracing  什麼是事件追蹤？

See this Microsoft Documentation: [Event Tracing](https://docs.microsoft.com/en-us/windows/desktop/etw/event-tracing-portal)
請參閱此 Microsoft 文件： [事件追蹤](https://docs.microsoft.com/en-us/windows/desktop/etw/event-tracing-portal)

# How to build  如何建造

Clone this repository. Open the solution (.sln) or project (.vcxproj) file in Visual Studio and build it. Alternatively, run Visual Studio developer command prompt, go to the cloned folder and run `msbuild` command. You can also build with mingw-w64 toolchain. Go to the folder in terminal run `make` command for mingw-w64/msys2.
克隆此倉庫。在 Visual Studio 中開啟解決方案 (.sln) 或專案 (.vcxproj) 檔案並進行建置。或者，執行 Visual Studio 開發人員命令提示符，進入複製的資料夾並執行 `msbuild` 命令。您也可以使用 mingw-w64 工具鏈進行建置。在終端機中進入該資料夾，然後執行 mingw-w64/msys2 的 `make` 命令。

# How to use  如何使用

Download the executable from [Release Page](https://github.com/Biswa96/TraceEvent/releases). Run this program as administrator every time. Here are the options.
從 [發布頁面](https://github.com/Biswa96/TraceEvent/releases) 下載可執行檔。每次執行此程式時，請務必以管理員身分執行。以下是運行選項。

```text
Usage: TraceEvent.exe [--] [option] [argument]
Options:

    -E,  --enumguidinfo                      Enumerate registered trace GUIDs with all PID and Logger ID. 
    -e,  --enumguid                          Enumerate registered trace GUIDs. 
    -g,  --guid        <ProviderGUID>        Add Event Provider GUID with trace session. 
    -L,  --list                              List all registered trace sessions with details. 
    -l,  --log         <LoggerName>          Log events in real time. 
    -q,  --query       <LoggerName>          Query status of <LoggerName> trace session. 
    -S,  --start       <LoggerName>          Starts the <LoggerName> trace session. 
    -s,  --stop        <LoggerName>          Stops the <LoggerName> trace session. 
    -h,  --help                              Display this usage information. 
```

## Start a session  開始會話

Run this command as administrator: `TraceEvent.exe --start <Session Name> --guid <Event Provider GUID>`. Always use an unique session name otherwise this will show error. Event provider GUIDs can be found from this Powershell cmdlet: `Get-EtwTraceProvider`. **Always use curly brackets** to specify GUID strings. Find more GUIDs in [**Event Providers list**](https://github.com/Biswa96/TraceEvent/blob/master/docs/Event_Providers.md). For example: `TraceEvent.exe --start MyTrace --guid {12345678-1234-1234-1234-123457890ABCD}`
請以管理員身分執行此命令： `TraceEvent.exe --start <Session Name> --guid <Event Provider GUID>` 。務必使用唯一的會話名稱，否則會顯示錯誤。事件提供者 GUID 可透過以下 PowerShell cmdlet 取得： `Get-EtwTraceProvider` 。 **請始終使用花括號**指定 GUID 字串。更多 GUID 請參閱 [**事件提供者清單**](https://github.com/Biswa96/TraceEvent/blob/master/docs/Event_Providers.md) 。例如： `TraceEvent.exe --start MyTrace --guid {12345678-1234-1234-1234-123457890ABCD}`

## Log events  日誌事件

Run this command as administrator: `TraceEvent.exe --log <Session Name>`. Only use session names which are started previously. If CPU usage becomes high then redirect output to a file. e.g. `TraceEvent.exe --log MyTrace > FileName.txt`
以管理員身分執行此命令： `TraceEvent.exe --log <Session Name>` 。僅使用先前已啟動的會話名稱。如果 CPU 使用率過高，則將輸出重新導向至檔案。例如： `TraceEvent.exe --log MyTrace > FileName.txt`

## Stop a session  停止會話

Run this command as administrator: `TraceEvent.exe --stop <Session Name>`. Stop only the previously opened tracing session. Using an already stopped session will show error. For example user this command to stop previously opened 'MyTrace' session: `TraceEvent.exe --stop MyTrace`.
以管理員身分執行此命令： `TraceEvent.exe --stop <Session Name>` 。僅停止先前開啟的追蹤會話。使用已停止的會話將顯示錯誤。例如，使用下列指令停止先前開啟的「MyTrace」會話： `TraceEvent.exe --stop MyTrace` 。

# Project Overview  項目概述

Here are the overview of source files according to their dependencies:
以下是根據依賴關係整理的來源文件概覽：

```text
TraceEvent\
    |
    +-- WinInternal: Crafted TRACE_CONTROL_FUNCTION_CLASS and NT API's definitions
    +-- PrintProperties: Display Event session details and it's security properties
    +-- CallBacks: Callback functions to log events messages
        |
        |   +-- Log: Helper functions to Log status and convert GUID to string
        |   +-- Helpers: Helper/Auxiliary functions for SecHost functions
        |   +-- SecHost: Internal functions from SecHost.dll, Advapi32.dll etc.
        |   |
        +-- TraceEvent: Functions to start, stop, log and other tasks
            |
            |    +-- wgetopt: Converted from Cygwin getopt file for wide characters
            |    |
            +-- main: Main function with option processing
```

# Further Readings  延伸閱讀

- [Event Tracing for Windows (ETW)  
    Windows 事件追蹤 (ETW)](https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/event-tracing-for-windows--etw-)
- [Retrieving Event Data Using TDH  
    使用 TDH 檢索事件數據](https://docs.microsoft.com/en-us/windows/desktop/etw/retrieving-event-data-using-tdh)
- [Configuring and Starting an Event Tracing Session  
    配置和啟動事件追蹤會話](https://docs.microsoft.com/en-us/windows/desktop/etw/configuring-and-starting-an-event-tracing-session)

# Acknowledgments  致謝

Thanks to:  由於：

- ProcessHacker's collection of [native API header file](https://github.com/processhacker/processhacker/tree/master/phnt)
- wbenny's [pedbex](https://github.com/wbenny/pdbex) tool
- RedPlait Blog: [NtTraceControl](https://redplait.blogspot.com/2011/02/nttracecontrol.html)
- Geoff Chappell: [NtTraceControl](http://www.geoffchappell.com/studies/windows/km/ntoskrnl/api/etw/traceapi/control/index.htm)

# License

This project is licensed under [GPLv3+](https://github.com/Biswa96/TraceEvent/blob/master/LICENSE). This program comes with ABSOLUTELY NO WARRANTY. This is free software, and you are welcome to redistribute it under certain conditions.
本項目採用 [GPLv3+](https://github.com/Biswa96/TraceEvent/blob/master/LICENSE) 授權協議。本程序不提供任何形式的擔保。本軟體為自由軟體，歡迎您在特定條件下進行再分發。

```text
TraceEvent -- (c) Copyright 2018-19 Biswapriyo Nath

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
```
