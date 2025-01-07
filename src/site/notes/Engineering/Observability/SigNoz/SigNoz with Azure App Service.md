---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"SigNoz with Azure App Service","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"SigNoz with Azure App Service","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/Observability/SigNoz/SigNoz with Azure App Service/","metatags":{"og:title":"SigNoz with Azure App Service","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"SigNoz with Azure App Service","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-01-08T01:22:53.322+09:00"}
---

#observability #azure #signoz #appservice


Observability ツールの [[SigNoz とは \|SigNoz]] に Azure, AWS, GCP などパブリッククラウドの設定方法についてドキュメントが記載されいて心が高まったので作業ログです。

## 検証の登場人物
以下の SizNoz の実装ガイドを元に構築を進めます。
- Application 部分は Azure App Service (Web Apps)を利用します。
- Event Hub, Azure Monitor を利用します。
- Central Collector は Azure VM を利用します。
- SigNoz Backend は Azure VM 上に Self-Host したものを利用します。 
![](/img/user/Engineering/Observability/SigNoz/20240926025057.png)
https://signoz.io/docs/azure-monitoring/bootstrapping/strategy/#implementation-guide


