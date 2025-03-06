---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"zPages を使ったデバッグ","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"zPages を使ったデバッグ","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/Observability/OpenTelemetry/zPages を使ったデバッグ/","metatags":{"og:title":"zPages を使ったデバッグ","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"zPages を使ったデバッグ","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-01-31T12:15:31.742+09:00"}
---

#opentelemetry #observability 
zPages を利用することで Collector のリアルタイムの内部状態を確認できます。

```yaml
extensions:  
  zpages:  
    endpoint: 0.0.0.0:55679  
  
service:  
  extensions: [zpages]
```

zPages にアクセスする
`http://<collector-host>:55679/debug/tracez`
![zPages を使ったデバッグ.png](/img/user/Engineering/Observability/OpenTelemetry/zPages%20%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%87%E3%83%90%E3%83%83%E3%82%B0.png)