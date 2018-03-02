title: "Druid SQL 语法支持更多的查询时聚合时间粒度"
date: 2017-12-12 19:51:12
categories:
- druid
tags:
- druid sql
---


## 背景

时序数据库的一个常用场景，是实时导入明细数据，在查询时按照用户需要的时间粒度进行聚合计算。

Druid 本质上也是一个时序数据库，支持多种查询时聚合方式，包括：

- Simple Granularity
  - Simple granularities are specified as a string and bucket timestamps by their UTC time.
  - Supported granularity strings are: `all`, `none`, `second`, `minute`, `fifteen_minute`, `thirty_minute`, `hour`, `day`, `week`, `month`, `quarter` and `year`.
  - `all` buckets everything into a single bucket.
  - `none` does not bucket data (it actually uses the granularity of the index - minimum here is `none` which means millisecond granularity). 
- Duration Granularity
  - Duration granularities are specified as an exact duration in milliseconds and timestamps are returned as UTC.
  - They also support specifying an optional origin, which defines where to start counting time buckets from (defaults to 1970-01-01T00:00:00Z).
- Period Granularity
  - Period granularities are specified as arbitrary period combinations of years, months, weeks, hours, minutes and seconds (e.g. P2W, P3M, PT1H30M, PT0.750S) in [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) format.
  - They support specifying a time zone which determines where period boundaries start as well as the timezone of the returned timestamps.
  - Origin is optional (defaults to 1970-01-01T00:00:00 in the given time zone).

以上三种查询时聚合方式，都可以通过 Druid 原生的 JSON 查询使用。

不过目前我们使用的是 Druid 的 SQL 模块，通过 SQL 查询 Druid 的数据，所以没办法直接使用 JSON 格式的查询描述。

Druid 的 SQL 模块支持 `FLOOR(__time TO <granularity>)` 函数，封装了 JSON 查询的 granularity 设置，但是经测试不支持 `FIVE_MINUTE`, `FIFTEEN_MINUTE` 等 JSON 查询可以支持的时间粒度。

<!-- more -->

##调研

Druid 的 SQL 模块通过 Calcite 提供 SQL 语法解析和验证，然后转化为 JSON 格式的查询提交给 Druid Broker Node 执行。

Calcite 默认支持 `YEAR`, `MONTH`, `DAY`, `HOUR`, `MINUTE`, `SECOND` 等时间关键字，不包括 `FIVE_MINUTE`, `FIFTEEN_MINUTE` 等 Druid 内置的时间粒度关键字，所以这些关键字无法通过 SQL 语法校验逻辑。

若想使Calcite 支持所需的时间粒度，就需要扩展 Calcite 的时间粒度，比在解析、校验相关部分增补对应逻辑。

## 开发

看了一下代码，发现 Calcite 的时间粒度是在 Avatica 子项目中维护的，其 TimeUnit 类和 TimeUnitRange 类包括了时间粒度相关的枚举定义。（Avatica 是 Calcite 的子项目， 按照官方定义是一个开发数据库 Driver 的框架，定义了client 和 server 之间通信的 api。Avatica Client 是 JDBC Driver，通过 HTTP 使用 JSON 或 Protobuf 的 API 和 Avatica Server 之间通信。所以我比较好奇为什么时间粒度是在这个子项目中定义的呢……）

TimeUnit 类声明了时间粒度的 `separator`, `multiplier` 和 `limit` ，描述了时间粒度枚举值的语法和时间基准。

calcite-core 模块的 SqlIntervalQualifier 类基于 TimeUnit 和 TimeUnitRange 提供了SQL 规范中 Interval Qualifier 语法的实现，在 FLOOR 函数参数校验的时候，会检查参数的类型。

Calcite 基于 JavaCC 生成了 SqlParserImpl 以及相关的 Token、确定有限状态机等。所以需要修改 SqlParser 的模板文件增加新增时间粒度的 Token 定义、解析等规则。

calcite-core 模块的 StandardConvertletTable 提供了 SQL expression 到 `RexNode` 的转化，其中也需要增加对新增的时间粒度的支持。包括`convertExtract`， `getFactor`, `TimestampDiffConverlet` 等。

此外，还有 SQL 解析相关的测试类 SqlParserTest 、SqlOperatorBaseTest 等需要增加新时间粒度对应的测试用例。

ps：Calcite 打包的时候会和 `site/_docs/reference.md` 文档中的 Token 列表进行校验，所以增加了时间粒度以后不要忘了更新文档 : p

最后，druid-sql 模块的 `TimeUnits` 也需要增加相关的 TimeUnit 常量，否则会影响RexNode 到 DruidRel的解析。

## 后续

目前只是在 Druid 社区 0.10.0 版本的基础上实现了常用时间粒度常量解析的支持，和 Druid 原生的 json 查询时聚合能力相比还有一定的差距，后续有空还需要再熟悉下 Calcite 的 SQL 解析和校验实现，扩展 FLOOR 函数语法，支持 `FLOOR(__time TO 'ISO8601 Duration')` 的调用方式，解析为 Druid 原生的 period Granularity 表示形式，这样就可以在 SQL 语法中支持任意时间粒度的聚合了。不过 Druid 社区 0.11 版本已经新增了 TIME_FLOOR 函数，支持任意的 ISO8601 period，所以这次魔改能够支持升级社区 0.11 版本之前的过渡期就可以了。
