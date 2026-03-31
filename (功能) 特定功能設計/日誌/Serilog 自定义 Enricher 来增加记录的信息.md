---
aliases:
date: 2019-08-08
update:
author: WeihanLi
language:
sourceurl: https://www.cnblogs.com/weihanli/p/custom-serilog-enricher-to-record-more-info.html
tags:
---

# Serilog 自定义 Enricher 来增加记录的信息

## Intro

Serilog 是 .net 里面非常不错的记录日志的库，结构化日志记录，而且配置起来很方便，自定义扩展也很方便

> Serilog is a diagnostic logging library for .NET applications. It is easy to set up, has a clean API, and runs on all recent .NET platforms. While it's useful even in the simplest applications, Serilog's support for structured logging shines when instrumenting complex, distributed, and asynchronous applications and systems.

Serilog 是.NET 应用程序的诊断日志库。 它易于设置，具有干净的 API，并可在所有最新的.NET 平台上运行。 虽然它在最简单的应用程序中也很有用，但 Serilog 对结构化日志记录的支持在处理复杂，分布式和异步应用程序和系统时仍然很有用。

之前一直使用 log4net 来记录日志，使用 serilog 之后觉得 serilog 比 log4net 好用很多，很灵活，配置方式多种多样，支持许多不同的输出，详细参考 [https://github.com/serilog/serilog/wiki/Provided-Sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks)

最近打算把之前基于 log4net 的日志迁移到 serilog， 我自定义的一套 logging 组件也增加了对 Serilog 的支持。 [https://www.nuget.org/packages/WeihanLi.Common.Logging.Serilog](https://www.nuget.org/packages/WeihanLi.Common.Logging.Serilog) 现在还没有发布正式版，不过我已经在用了，在等 serilog 发布 2.9.0 正式版，因为 2.8.x 版本的 netstandard2.0 版本还依赖了一个 `System.Collections.NonGeneric`，这个依赖只在 netstandard1.3 的时候需要引用，netstandard2.0 已经不需要了，详细信息可以参考 PR: [https://github.com/serilog/serilog/pull/1342](https://github.com/serilog/serilog/pull/1342)

## 自定义 Enricher

自定义 Enricher 很简单很方便，来看示例：

```csharp
public class RequestInfoEnricher : ILogEventEnricher
{
    public void Enrich(LogEvent logEvent, ILogEventPropertyFactory propertyFactory)
    {
        var httpContext = DependencyResolver.Current.GetService<IHttpContextAccessor>()?.HttpContext;
        if (null != httpContext)
        {
            logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("RequestIP", httpContext.GetUserIP()));
            logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("RequestPath", httpContext.Request.Path));

            logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("Referer", httpContext.Request.Headers["Referer"]));
        }
    }
}
```

这个示例会尝试获取请求的 RequestIP/RequestPath/Referer 写入到日志中，这样可以方便我们的请求统计

为了方便使用我们可以再定义一个扩展方法：

```csharp
public static class EnricherExtensions
{
    public static LoggerConfiguration WithRequestInfo(this LoggerEnrichmentConfiguration enrich)
    {
        if (enrich == null)
            throw new ArgumentNullException(nameof(enrich));

        return enrich.With<RequestInfoEnricher>();
    }
}
```

配置 Serilog ：

```csharp
loggingConfig
    .WriteTo.Elasticsearch(Configuration.GetConnectionString("ElasticSearch"), $"logstash-{ApplicationHelper.ApplicationName.ToLower()}")
    .Enrich.FromLogContext()
    .Enrich.WithRequestInfo()
```

完整代码示例参考：[https://github.com/WeihanLi/ActivityReservation/blob/e68ab090f8b7d660f0a043889f4353551c8b3dc2/ActivityReservation/SerilogEnrichers/RequestInfoEnricher.cs](https://github.com/WeihanLi/ActivityReservation/blob/e68ab090f8b7d660f0a043889f4353551c8b3dc2/ActivityReservation/SerilogEnrichers/RequestInfoEnricher.cs)

## 验证

来看一下我们使用了我们自定义的 `RequestInfoEnricher` 之后的日志效果

[![](https://img2018.cnblogs.com/blog/489462/201908/489462-20190809094910687-1646011001.png)](https://img2018.cnblogs.com/blog/489462/201908/489462-20190809094910687-1646011001.png)

可以看到我们自定义的 Enricher 中添加的请求信息已经写到日志里了，而且我们可以根据 RequestIP 去筛选，为了方便查询 IP 统计，在 kibana 中加了一个可视化面板，效果如下图所示：

[![Request IP Summary](https://img2018.cnblogs.com/blog/489462/201908/489462-20190809094843333-1430022901.png)](https://img2018.cnblogs.com/blog/489462/201908/489462-20190809094843333-1430022901.png)

## Reference

- [https://github.com/serilog/serilog](https://github.com/serilog/serilog)
- [https://github.com/WeihanLi/ActivityReservation/blob/e68ab090f8b7d660f0a043889f4353551c8b3dc2/ActivityReservation](https://github.com/WeihanLi/ActivityReservation/blob/e68ab090f8b7d660f0a043889f4353551c8b3dc2/ActivityReservation)

作者：weihanli

出处：[https://www.cnblogs.com/weihanli/p/custom-serilog-enricher-to-record-more-info.html](https://www.cnblogs.com/weihanli/p/custom-serilog-enricher-to-record-more-info.html)

版权：本作品采用「[署名-非商业性使用-相同方式共享 4.0 国际](https://creativecommons.org/licenses/by/4.0)」许可协议进行许可。

本文版权归作者和博客园共有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文连接，否则保留追究法律责任的权利。
