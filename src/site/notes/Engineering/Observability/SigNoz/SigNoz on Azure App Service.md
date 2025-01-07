---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"SigNoz on Azure App Service","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"SigNoz on Azure App Service","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/Observability/SigNoz/SigNoz on Azure App Service/","metatags":{"og:title":"SigNoz on Azure App Service","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"SigNoz on Azure App Service","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-01-08T01:22:53.322+09:00"}
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

# 環境を構築してみる
以降手順になります。
Azure の Subscription の作成は省略します。

## Step1 サンプルアプリ(.NET)の作成
ASP.NET でサンプルアプリを作成します。
参考: https://opentelemetry.io/docs/languages/net/getting-started/

## Step2 Azure App Service (Web Apps) を作成
Step1 で作成した ASP.NET のアプリを Azure App Service 上に展開します。
1. App Service Plan の作成
	1. 検証用で Plan は Free としています。
2. Web Apps の作成
	1. 今回は OS を Windows にしています。
3. デプロイ出来ているか確認
```shell
$ curl -X 'GET' \
  'https://signoz-azure-b5hgavavexfhemdg.japaneast-01.azurewebsites.net/WeatherForecast' \
  -H 'accept: text/plain'

[{"date":"2024-09-25","temperatureC":39,"temperatureF":102,"summary":"Hot"},{"date":"2024-09-26","temperatureC":-3,"temperatureF":27,"summary":"Chilly"},{"date":"2024-09-27","temperatureC":40,"temperatureF":103,"summary":"Freezing"},{"date":"2024-09-28","temperatureC":11,"temperatureF":51,"summary":"Chilly"},{"date":"2024-09-29","temperatureC":53,"temperatureF":127,"summary":"Bracing"}]%
```

## Step3 Azure Event Hubs の作成

### Event Hubs namespace の作成
今回はサンプルとして動かすだけなので最小構成で作成します。
価格レベル: Basic, スループットユニット: 1ユニットで作成しました。
	なお、これらの値に関しては後から変更が可能です。