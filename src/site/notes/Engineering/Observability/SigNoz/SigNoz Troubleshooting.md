---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"SigNoz Troubleshooting","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"SigNoz Troubleshooting","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/Observability/SigNoz/SigNoz Troubleshooting/","metatags":{"og:title":"SigNoz Troubleshooting","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"SigNoz Troubleshooting","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-01-27T01:32:08.758+09:00"}
---


#signoz 

SigNoz の Open Telemetry Collector に接続できるか。等を検証できるツール
https://github.com/SigNoz/troubleshoot

```bash
./troubleshoot checkEndpoint --endpoint=localhost:4317
```

などで実行できる

コンテナイメージも提供されているため以下でも実行可能
```bash
docker run -it --rm signoz/troubleshoot checkEndpoint --endpoint=172.17.0.1:4317
```

実行すると以下のようにデータを送ったログが出力される
```shell
docker run -it --rm signoz/troubleshoot checkEndpoint -e host.docker.internal:4317
2025-01-26T16:30:05.492Z        INFO    troubleshoot/main.go:28 STARTING!
2025-01-26T16:30:05.497Z        INFO    checkEndpoint/checkEndpoint.go:41       checking reachability of SigNoz endpoint
2025-01-26T16:30:05.558Z        INFO    troubleshoot/main.go:46 Successfully sent sample data to signoz ...
```