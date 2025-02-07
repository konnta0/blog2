---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":".NET Aspire で SigNoz を動かしてみる","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":".NET Aspire で SigNoz を動かしてみる","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/-.NET/-.NET Aspire/NET Aspire で SigNoz を動かしてみる/","metatags":{"og:title":".NET Aspire で SigNoz を動かしてみる","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":".NET Aspire で SigNoz を動かしてみる","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-02-04T00:19:39.093+09:00"}
---


#dotnet #csharp #observability #signoz 

[.NET Aspire](https://learn.microsoft.com/ja-jp/dotnet/aspire/get-started/aspire-overview) で オープンソース オブザーバビリティツールの [SigNoz](https://signoz.io/)を動かしてみた時の備忘録です。
.NET Aspire ってなんだっけ？や
SigNoz ってなに？というのはここでは割愛します。

# 前提条件と申し送り
macOS での検証となります。[Windows では SigNoz が動かない可能性](https://signoz.io/docs/install/docker/)があります(悲しい)
SigNoz は SigNoz Cloud を利用せずにセルフホストしたものを利用します。
Aspire のコードの書き方としてもう少しスマートのやり方があるかもしれません。
「あくまで動かしてみること」を優先した実装サンプルとなります。
Log, Trace, Metrics が確認できることがひとまずのゴールとしています。
# .NET Aspire プロジェクトを作成
.NET Aspire のプロジェクトの作成方法は [こちら](https://learn.microsoft.com/ja-jp/dotnet/aspire/get-started/build-your-first-aspire-app?pivots=visual-studio) を参考に行えば問題ないです。
# コンテナの起動処理などを作成する
`AppHost` のプロジェクトに SigNoz のコンテナを立ち上げる処理を `Program.cs` に追加していきます。
`BindMount` しているファイル群は、[こちら](https://github.com/SigNoz/signoz/blob/v0.68.0/deploy/docker/clickhouse-setup/) を参考にします。
また、`WithEndpoint`, `WithArgs` などなどの設定については [こちら](https://github.com/SigNoz/signoz/blob/v0.68.0/deploy/docker/clickhouse-setup/docker-compose-minimal.yaml) の compose ファイルを参考にします。
```cs
var builder = DistributedApplication.CreateBuilder(args);  
  
// https://github.com/SigNoz/signoz/blob/v0.68.0/deploy/docker/clickhouse-setup/docker-compose-minimal.yaml  
var zookeeper = builder.AddContainer("zookeeper-1", "bitnami/zookeeper", "3.7.1")  
    .WithEndpoint(2181, name: "zookeeper-2181", isProxied: false)  
    .WithEndpoint(2888, name: "zookeeper-2888", isProxied: false)  
    .WithEndpoint(3888, name: "zookeeper-3888", isProxied: false)  
    .WithBindMount("../../.data/zookeeper", "/bitnami/zookeeper")  
    .WithEnvironment("ALLOW_ANONYMOUS_LOGIN", "yes")  
    .WithEnvironment("ZOO_SERVER_ID", "1")  
    .WithEnvironment("ZOO_AUTOPURGE_INTERVAL", "1");  
  
var clickhouse = builder.AddContainer("clickhouse", "clickhouse/clickhouse-server", "24.1.2-alpine")  
    .WithEndpoint(8123, name: "clickhouse-8123", isProxied: false)  
    .WithEndpoint(9000, name: "clickhouse-9000", isProxied: false)  
    .WithEndpoint(9181, name: "clickhouse-9181", isProxied: false)  
    .WithBindMount("Container/config/clickhouse/clickhouse-config.xml", "/etc/clickhouse-server/config.xml")  
    .WithBindMount("Container/config/clickhouse/clickhouse-users.xml", "/etc/clickhouse-server/users.xml")  
    .WithBindMount("Container/config/clickhouse/custom-function.xml", "/etc/clickhouse-server/custom-function.xml")  
    .WithBindMount("Container/config/clickhouse/clickhouse-cluster.xml", "/etc/clickhouse-server/config.d/cluster.xml")  
    .WithBindMount("Container/config/clickhouse/user_scripts/", "/var/lib/clickhouse/user_scripts/")  
    .WithBindMount("../../.data/clickhouse/", "/var/lib/clickhouse/")  
    .WaitFor(zookeeper);  
  
var otelMigratorSync = builder.AddContainer("otel-collector-migrator-sync", "signoz/signoz-schema-migrator", "0.111.23")  
    .WithArgs("sync", "--dsn=tcp://clickhouse:9000", "--up=")  
    .WithReference(clickhouse.GetEndpoint("clickhouse-9000"))  
    .WaitFor(clickhouse);  
  
var queryService = builder.AddContainer("query-service", "signoz/query-service", "0.68.0")  
    .WithArgs("-config=/root/config/prometheus.yml", "--use-logs-new-schema=true",  
        "--use-trace-new-schema=true")  
    .WithBindMount("Container/config/queryservice/prometheus.yml", "/root/config/prometheus.yml")  
    .WithBindMount("Container/config/queryservice/dashboards", "/root/config/dashboards")  
    .WithBindMount("../../.data/queryservice/", "/var/lib/signoz/")  
    .WithEnvironment("ClickHouseUrl", "tcp://clickhouse:9000")  
    .WithEnvironment("ALERTMANAGER_API_PREFIX", "http://alertmanager:9093/api/")  
    .WithEnvironment("SIGNOZ_LOCAL_DB_PATH", "/var/lib/signoz/signoz.db")  
    .WithEnvironment("DASHBOARDS_PATH", "/root/config/dashboards")  
    .WithEnvironment("STORAGE", "clickhouse")  
    .WithEnvironment("GODEBUG", "netdns=go")  
    .WithEnvironment("TELEMETRY_ENABLED", "true")  
    .WithEnvironment("DEPLOYMENT_TYPE", "docker-standalone-amd")  
    .WithHttpEndpoint(8085, name: "query-service-8085", isProxied: false)  
    .WithReference(clickhouse.GetEndpoint("clickhouse-9000"))  
    .WaitFor(clickhouse)  
    .WaitFor(otelMigratorSync);  
  
var frontend = builder.AddContainer("frontend", "signoz/frontend", "0.68.0")  
    .WithHttpEndpoint(3301, name: "frontend-3301", isProxied: false)  
    .WithBindMount("Container/config/frontend/nginx-config.conf", "/etc/nginx/conf.d/default.conf")  
    .WithReference(queryService.GetEndpoint("query-service-8085"))  
    .WaitFor(queryService);  
  
var alertManager = builder.AddContainer("alertmanger", "signoz/alertmanager", "0.23.7")  
    .WithArgs("--queryService.url=http://query-service:8085", "--storage.path=/data")  
    .WithEndpoint(9093, name: "alertmanager-9093", isProxied: false)  
    .WithBindMount("../../.data/alertmanager", "/data")  
    .WithReference(queryService.GetEndpoint("query-service-8085"))  
    .WaitFor(queryService)  
    .WaitFor(clickhouse);  
  
queryService.WithReference(alertManager.GetEndpoint("alertmanager-9093"));  
  
var otelCollector = builder.AddContainer("otel-collector", "signoz/signoz-otel-collector", "0.111.23")  
    .WithContainerRuntimeArgs("--user=0")  
    .WithArgs("--config=/etc/otel-collector-config.yaml",  
        "--manager-config=/etc/manager-config.yaml",  
        "--copy-path=/var/tmp/collector-config.yaml",  
        "--feature-gates=-pkg.translator.prometheus.NormalizeName")  
    .WithBindMount("Container/config/otel-collector/otel-collector-config.yaml", "/etc/otel-collector-config.yaml")  
    .WithBindMount("Container/config/otel-collector/otel-collector-opamp-config.yaml", "/etc/manager-config.yaml")  
    .WithBindMount("/var/lib/docker/containers", "/var/lib/docker/containers", true)  
    .WithBindMount("/", "/hostfs", true)  
    .WithEnvironment("OTEL_RESOURCE_ATTRIBUTES", "host.name=signoz-host,os.type=linux")  
    .WithEnvironment("LOW_CARDINAL_EXCEPTION_GROUPING", "false")  
    .WithHttpEndpoint(port: 4317, targetPort: 4317, name: "grpc")  
    .WithHttpEndpoint(port: 4318, targetPort: 4318, name: "http")  
    .WithEndpoint(port: 14268, targetPort: 4318, name: "jaeger-thrift-http")  
    .WithHttpEndpoint(port: 55679, targetPort: 55679, name: "zpages")  
    .WithReference(clickhouse.GetEndpoint("clickhouse-9000"))  
    .WithReference(queryService.GetEndpoint("query-service-4320"))  
    .WaitFor(clickhouse)  
    .WaitFor(queryService)  
    .WaitForCompletion(otelMigratorSync);  
  
var apiService = builder.AddProject<Projects.AspireApp1_ApiService>("apiservice")  
    .WithEnvironment("OTEL_SERVICE_NAME", "api-service")  
    .WithEnvironment("OTEL_EXPORTER_OTLP_ENDPOINT", otelCollector.GetEndpoint("grpc"))  
    .WaitFor(otelCollector);  
  
builder.AddProject<Projects.AspireApp1_Web>("webfrontend")  
    .WithExternalHttpEndpoints()  
    .WithReference(apiService)  
    .WithEnvironment("OTEL_SERVICE_NAME", "webfrontend")  
    .WithEnvironment("OTEL_EXPORTER_OTLP_ENDPOINT", otelCollector.GetEndpoint("grpc"))  
    .WaitFor(apiService)  
    .WaitFor(frontend)  
    .WaitFor(otelCollector);  
  
builder.Build().Run();
```

次に、適当なログを出力する処理を追記します。
`ApiSerice` プロジェクトの  `Program.cs` にいくつか処理を追加します。
最終的には以下のような実装にします。
```cs
using System.Diagnostics.Metrics;  
using Microsoft.AspNetCore.Mvc;  
  
var builder = WebApplication.CreateBuilder(args);  
  
// Add service defaults & Aspire client integrations.  
builder.AddServiceDefaults();  
builder.Services.AddOpenTelemetry().WithLogging();  
  
// Add services to the container.  
builder.Services.AddProblemDetails();  
  
// Learn more about configuring OpenAPI at https://aka.ms/aspnet/openapi  
builder.Services.AddOpenApi();  
const string meterName = "MyCustomMetrics";  
builder.Services.AddOpenTelemetry().WithMetrics(static metrics =>  
{  
    metrics.AddMeter(meterName);  
});  
  
var app = builder.Build();  
  
// Configure the HTTP request pipeline.  
app.UseExceptionHandler();  
  
if (app.Environment.IsDevelopment())  
{  
    app.MapOpenApi();  
}  
  
string[] summaries = ["Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"];  
var meter = new Meter(meterName, "1.0.0");  
var requestCounter = meter.CreateCounter<long>("myapp_requests", "requests", "Number of requests handled");  
app.MapGet("/weatherforecast", ([FromServices]ILogger<Program> logger) =>  
{  
    // カウンターの加算
    requestCounter.Add(1);

    // ログの出力
    logger.LogInformation("Getting weather forecast. Information!");  
    logger.LogWarning("Getting weather forecast. Warning!");  
    logger.LogError("Getting weather forecast. Error!");  
  
    var forecast = Enumerable.Range(1, 5).Select(index =>  
        new WeatherForecast  
        (  
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),  
            Random.Shared.Next(-20, 55),  
            summaries[Random.Shared.Next(summaries.Length)]  
        ))  
        .ToArray();  
    return forecast;  
})  
.WithName("GetWeatherForecast");  
  
app.MapDefaultEndpoints();  
  
app.Run();  
  
record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)  
{  
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);  
}
```


> [!tips]
> builder.Services.AddOpenTelemetry().WithLogging();  
> を、 `Program.cs` で呼び出していますが、Aspire プロジェクトをテンプレートから作成した際に実装されている、`ServiceDefaults` プロジェクト内で `UseOtlpExporter` 関数内で `WithLogging()` は呼び出されているのですが、検証時は期待通りの挙動にならなかったため `WithLogging()` を `ApiSerivce` プロジェクト内で呼び出すようにしています。
# 動作確認
SigNoz のフロントエンドへは `localhost:3301` でアクセスできます。
.NET Aspire で起動した `webfrontend` サービス(`localhost:5272`) へアクセスして画面を触った後に情報が SigNoz へ送信されているか確認します。
## Log
![NET Aspire で SigNoz を動かしてみる-5.png](/img/user/Engineering/-.NET/-.NET%20Aspire/NET%20Aspire%20%E3%81%A7%20SigNoz%20%E3%82%92%E5%8B%95%E3%81%8B%E3%81%97%E3%81%A6%E3%81%BF%E3%82%8B-5.png)

## Trace 

![NET Aspire で SigNoz を動かしてみる-1.png](/img/user/Engineering/-.NET/-.NET%20Aspire/NET%20Aspire%20%E3%81%A7%20SigNoz%20%E3%82%92%E5%8B%95%E3%81%8B%E3%81%97%E3%81%A6%E3%81%BF%E3%82%8B-1.png)


![NET Aspire で SigNoz を動かしてみる.png](/img/user/Engineering/-.NET/-.NET%20Aspire/NET%20Aspire%20%E3%81%A7%20SigNoz%20%E3%82%92%E5%8B%95%E3%81%8B%E3%81%97%E3%81%A6%E3%81%BF%E3%82%8B.png)

![NET Aspire で SigNoz を動かしてみる-2.png](/img/user/Engineering/-.NET/-.NET%20Aspire/NET%20Aspire%20%E3%81%A7%20SigNoz%20%E3%82%92%E5%8B%95%E3%81%8B%E3%81%97%E3%81%A6%E3%81%BF%E3%82%8B-2.png)

![NET Aspire で SigNoz を動かしてみる-3.png](/img/user/Engineering/-.NET/-.NET%20Aspire/NET%20Aspire%20%E3%81%A7%20SigNoz%20%E3%82%92%E5%8B%95%E3%81%8B%E3%81%97%E3%81%A6%E3%81%BF%E3%82%8B-3.png)

## Metrics
![NET Aspire で SigNoz を動かしてみる-4.png](/img/user/Engineering/-.NET/-.NET%20Aspire/NET%20Aspire%20%E3%81%A7%20SigNoz%20%E3%82%92%E5%8B%95%E3%81%8B%E3%81%97%E3%81%A6%E3%81%BF%E3%82%8B-4.png)



> [!tips] トラブルシューティング
> 
> [[Engineering/Observability/SigNoz/SigNoz Troubleshooting\|OpenTelemetry Collector に情報をくれるか試すツール]] で要求が送れてそうか
[[Engineering/Observability/OpenTelemetry/zPages を使ったデバッグ\|zPages を使って]] OpenTelemetry Collector 側でデータを受信できてそうか
を、確認するとよいでしょう
> 
> 
> 
> 
> 

---
ひとまず動くようになったら、`ServiceDefaults` プロジェクトの設定を色々変更したり、カウンター以外のメトリクスを追加し検証してもいいかもしれません。
