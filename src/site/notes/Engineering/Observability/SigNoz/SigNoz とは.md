---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"description":"","og:title":"SigNoz とは","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"SigNoz とは","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/Observability/SigNoz/SigNoz とは/","metatags":{"description":"","og:title":"SigNoz とは","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"SigNoz とは","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-01-08T00:52:56.195+09:00"}
---

#signoz #observability 

## SigNoz とは
OpenTelemetry をサポートしている Observability ツールです。
以下のような o11y ツールとしては定番の機能が提供されています。
- APM (Application Performance Monitoring)
- Distributed Tracing
- Log Management
- Metrics & Dashboards
- Exceptions
- Alerts


また、公式サイトを見る限り SAMSUNG や salesforce など日本でも知られている企業で導入されています。
![](/img/user/Engineering/Observability/SigNoz/20241002010404.png)

### SigNoz の個人的にいいところ
#### コスト
まず、コスト面が優秀かなと思います。SigNoz 曰くは他の o11y ツールに比べてコストが低めです。
SigNoz Cloud はユーザー毎の課金や Container や Node 単位での課金は現時点では無いため企業で導入しても利用者が増えても負担になりにくい傾向にあるようです。
また、SigNoz は Datadog や Grafana などの先発な o11y ツール類を意識していること印象が多く、コスト面や機能面の優位性を開発初期はアピールしていたように思います。
https://signoz.io/product-comparison/signoz-vs-datadog/
https://signoz.io/blog/pricing-comparison-signoz-vs-datadog-vs-newrelic-vs-grafana/

![](/img/user/Engineering/Observability/SigNoz/20241001014003.png)

#### ホスティングの選択
2つ目にホスティングを選ぶことができることも良い点かなと思います。
SigNoz Cloud, Self-Host, On-pre の Managed SigNoz が利用できるため様々な環境に取り入れやすいと思います。

---

これら2つの良いところを組み合わせると例えば共有で利用する開発環境やステージング、本番環境では SigNoz Cloud を利用して各自の手元での開発は Self-Host 版を利用しても大きくコストがかかる心配も薄いです。
何より普段の開発から同一の o11y を用いることができるので、あまりこの手のツールに馴染みが無い組織においての導入のハードルが下げれますし、「本番環境だけ入っていて実はあまり使われてない(使い方がわからない)」≒「o11y ツール高い」みたいな少し勿体ない解釈も減るんじゃなかろうかと感じています。