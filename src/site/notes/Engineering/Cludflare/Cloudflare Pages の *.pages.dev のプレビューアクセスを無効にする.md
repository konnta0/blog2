---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"Cloudflare Pages の *.pages.dev のプレビューアクセスを無効にする","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Cloudflare Pages の *.pages.dev のプレビューアクセスを無効にする","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/Cludflare/Cloudflare Pages の *.pages.dev のプレビューアクセスを無効にする/","metatags":{"og:title":"Cloudflare Pages の *.pages.dev のプレビューアクセスを無効にする","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Cloudflare Pages の *.pages.dev のプレビューアクセスを無効にする","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-01-31T12:15:31.742+09:00"}
---

#cloudflare 
## Cloudflare Access を設定する
1. `ダッシュボード`から`Workers & Pages` を選択
2. `Overview` を選択
3. `Settings` タブの `General` の項目の `Access policy` を `Enable` に変更する
	1. ![](/img/user/Engineering/Cludflare/Cloudflare-Pages-.pages.dev-access.png)
4. 設定が完了するとプレビューページ(`*.{project}.pages.dev`)へのアクセスはメールでのコード認証になる
	1. ![](/img/user/Engineering/Cludflare/Cloudflare-Pages-.pages.dev-access-01.png)

## \*.pages.dev へのアクセスをカスタムドメインへリダイレクトする
1. `Account Home` -> `Websites` から設定を行いたいサイトを選択する
2. `Rules` -> `Redirect Rules` -> `Bulk Redirects` から `+ Create rule` を選択する
	1. ![](/img/user/Engineering/Cludflare/Cloudflare-Pages-.pages.dev-access-03.png)
3. `Source URL` に `page.dev` の URL を設定し、 `Target URL` にリダイレクトさせたい URL を設定する。また、`Subpath matching`にチェックを入れておく。必要に応じて `Preserve query string`, `Preserve path suffix` にもチェックを入れておく
	1. ![](/img/user/Engineering/Cludflare/Cloudflare-Pages-.pages.dev-access-02.png)


参考:
https://developers.cloudflare.com/pages/configuration/custom-domains/#disable-access-to-pagesdev-subdomain

https://developers.cloudflare.com/pages/configuration/preview-deployments/#customize-preview-deployments-access

https://developers.cloudflare.com/rules/url-forwarding/bulk-redirects/

https://dev.classmethod.jp/articles/cloudflare-pages-access/