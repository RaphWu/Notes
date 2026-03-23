---
aliases:
date: 2017-01-19
update:
author: Jeffrey,黑暗執行緒
language: C#
sourceurl: https://blog.darkthread.net/blog/autofac-notes-6/
tags:
  - CSharp
  - IoC
  - Autofac
  - ASPdotNET
---

# Autofac 筆記 6-Resolve 時依參數傳回不同型別

好久沒寫 [Autofac 筆記](http://blog.darkthread.net/blogs/darkthreadtw/archive/tags/Autofac/default.aspx)，記錄一則最近遇到的小需求。系統中針對介面（例如：IBlah）實作了多個型別，`Resolve<IBlah>()` 時希望透過參數指定傳回不同型別。

依據 [官方文件](http://docs.autofac.org/en/latest/advanced/keyed-services.html)，實現這類需求的最簡單做法是使用 `Named Service`（具名服務）或 `Keyed Service`（鍵值對應服務）， `Register<T>()` 後不使用 `As<IBlah>()`，而改用 `Named<IBlah>("服務名稱")` 或 `Keyed<IBlah>(列舉值)` 註冊。之後呼叫端改用 `ResolveNamed<IBlah>("服務名稱")` 或 `ResolveKeyed<IBlah>(列舉值)` 即可取得不同執行個體（Instance）。如此我們可為 IBlah 註冊多種型別，彼此以服務名稱字串或列舉值區隔，`Resolve` 時透過服務名稱字串或列舉值可取得指定型別的執行個體。`Named` 與 `Keyed` 功能相同，差異在於使用字串或列舉當參數，我傾向使用列舉以充分享受強型別的優勢。

講了一堆，其實用起來蠻簡單的，來段程式範例大家就明白了：

```csharp
using Autofac;
using System;
 
namespace AutofacLab
{
    public interface ISensor
    {
        string Detect();
    }
    public class TemperatureeSensor : ISensor
    {
        public string Detect()
        {
            return "It's hot";
        }
    }
    public class SoundSensor: ISensor
    {
        public string Detect()
        {
            return "It's noisy";
        }
    }
 
    public enum SensorType
    {
        Temperature,
        Sound
    }
 
    class Program
    {
        static void _Main(string[] args)
        {
            ContainerBuilder builder = new ContainerBuilder();
            //http://docs.autofac.org/en/latest/advanced/keyed-services.html
            builder.RegisterType<SoundSensor>().Keyed<ISensor>(SensorType.Sound);
            builder.Register((c) =>
            {
                return new TemperatureeSensor();
            }).Keyed<ISensor>(SensorType.Temperature);
            IContainer container = builder.Build();
 
            var ss = container.ResolveKeyed<ISensor>(SensorType.Sound);
            Console.WriteLine(ss.Detect());
            var ts = container.ResolveKeyed<ISensor>(SensorType.Temperature);
            Console.WriteLine(ts.Detect());
            Console.Read(); 
        }
    }
}
```
