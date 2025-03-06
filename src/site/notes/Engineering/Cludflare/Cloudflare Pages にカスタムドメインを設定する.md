---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"Cloudflare Pages にカスタムドメインを設定する","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Cloudflare Pages にカスタムドメインを設定する","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/Cludflare/Cloudflare Pages にカスタムドメインを設定する/","metatags":{"og:title":"Cloudflare Pages にカスタムドメインを設定する","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Cloudflare Pages にカスタムドメインを設定する","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-01-31T12:15:31.741+09:00"}
---


#cloudflare #cloudflaredns #dns 

ダッシュボードから `Workers & Pages` を選択
カスタムドメインを設定したいプロジェクトを選択する
`Custom Domains` のタブから設定を開始する
![](/img/user/Engineering/Cludflare/0.png)

設定したいカスタムドメインを入力して `Continue`
![](/img/user/Engineering/Cludflare/1.png)

今回は Cloudflare DNS を利用する
`Begin DNS transfer` を選択
![](/img/user/Engineering/Cludflare/2.png)

作成済みのドメインを入力して `Continue`
尚、ドメイン自体はお名前.com で作成した
![](/img/user/Engineering/Cludflare/3.png)

Free を選択して `Continue`
![](/img/user/Engineering/Cludflare/4.png)

DNS の設定を追加する
サブドメインとして運用したかったので `CNAME` の設定を追加して `Save` しつつ `Continue to activation`
![](/img/user/Engineering/Cludflare/5.png)

以下の Update your nameservers の 2項目をコピーする
![](/img/user/Engineering/Cludflare/7.png)

お名前.com の管理画面でコピーした値を貼り付ける
![](/img/user/Engineering/Cludflare/6.png)

再び Cloudflare のダッシュボードで `Continue` すると検証が始まり、検証時には 5分程度で設定が有効化された


再度、`Workers & Pages` -> `Custom domains` より設定したいドメインを入力し `Active domain` を選択
![](/img/user/Engineering/Cludflare/8.png)

成功すると以下の様に有効になり設定したドメイン経由で Cloudflare Pages にアクセスできるようになる(体感は 2,3 分で設定が完了した)
![](/img/user/Engineering/Cludflare/9.png)