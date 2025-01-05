---
{"dg-publish":true,"permalink":"/engineering/cloudflare-pages-obsidian/","metatags":{"og:title":"Cloudflare Pages と Obsidian でデジタルガーデンを作成する","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Cloudflare Pages と Obsidian でデジタルガーデンを作成する","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"noteIcon":"","created":"2024-12-18T00:45:58.255+09:00"}
---


#cloudflare #obsidian #DigitalGarden 

[Digital Garden](Digital%20Garden.md) を作成出来る Obsidian のプラグインを使いつつ初期設計では Vercel になっているので Cloudflare Pages に置き換える
# 利用する物
- [Cloudflare Pages](https://www.cloudflare.com/ja-jp/developer-platform/products/pages/)
- [obsidian-digital-garden](https://github.com/oleeskild/obsidian-digital-garden) 

# 1. obsidian-digial-garden を利用して Vercel にデプロイする

[Getting started](https://dg-docs.ole.dev/getting-started/01-getting-started/) を参考に Vercel にデプロイできるような状態にする

# 2. Cloudflare のアカウントの作成および Cloudflare Pages のプロジェクトの作成



# 3. Vercel と GitHub の連携を解除する
## Vercel 上から作成されたアプリを削除する
1.  `Overview` -> `Settings` で設定ページに移動
2.  `General` -> `Delete Project` の `Delete` を選択
	1. もし、Vercel の プロジェクトを残しておきたい場合(連携のみを解除するの)は `Git` の項目からリポジトリとの連携が解除出来る
`

## GitHub 側から Vercel アプリを削除する 
1. 項 1 で作成したリポジトリの `Settings` -> `Integrations` -> `GitHub Apps` に Vercel のアプリがあるはずなので、`Configure` を選択
2. `Danger zone` に、`Uninstall "Vercel"` があるので `Uninstall` を選択
	1. **この削除は自身の関連した全ての Vercel アプリが動かなくなる** ので、既に別件で利用している場合は、削除しないか、`Repository access` を設定するなどして対応が必要

# その他
- 必要があれば [Cloudflare Pages にカスタムドメインを設定する](Cludflare/Cloudflare%20Pages%20にカスタムドメインを設定する.md)

