---
{"dg-publish":true,"dg-metatags":{"og:title":"Obsidian のテーマを公開するとき","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Obsidian のテーマを公開するとき","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Obsidian/Obsidian のテーマを公開するとき/","metatags":{"og:title":"Obsidian のテーマを公開するとき","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Obsidian のテーマを公開するとき","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-08-05T00:33:03.562+09:00","updated":"2025-08-10T01:27:19.637+09:00"}
---

#obsidian 

# 1. サンプルプラグインをテンプレートとして利用
以下リポジトリをテンプレートとして利用してリポジトリを作成します
https://github.com/obsidianmd/obsidian-sample-plugin

# 2. 必要なファイルを置いたり、変えたり、消したり
 `theme.cs`, `manifest.json`, `package.json`, `README.md`, `LICENSE` を適当に設置したり変更したりします。

# 3. コミュニティに PR を出す
以下のリポジトリの [community-css-themes.json](https://github.com/obsidianmd/obsidian-releases/blob/master/community-css-themes.json) に作成したテーマプラグインの項目を追加して PR を出します。
https://github.com/obsidianmd/obsidian-releases

## 設定参考例と補足
```json
{
  "name": "Minimal",
  "author": "kepano",
  "repo": "kepano/obsidian-minimal",
  "screenshot": "dark-simple.png",
  "modes": ["dark", "light"]
},

```

> `name` と `author` は、プラグインがユーザーにどのように表示されるかを決定し、マニフェストの対応するプロパティと一致する必要があります。
`repo` は GitHub リポジトリのパスです。例えば、GitHub リポジトリが https://github.com/your-username/your-repo-name に配置されている場合、パスは your-username/your-repo-name になります。
`screenshot` はテーマのスクリーンショットのパスです。スクリーンショットは 16:9 のアスペクト比で最も見栄えが良くなります。推奨画像サイズ：512 x 288 ピクセル。
`modes` はテーマがサポートするカラーモードを列挙します。


小ネタ
- name に Obsidian や theme の類を設定すると冗長なためエラー扱いにされます。
![Pasted image 20250806014441.png](/img/user/Pasted%20image%2020250806014441.png)

- スクリーンショットサイズの推奨を無視するとこれまたエラー扱いにされます。
	- (幅 1000px または 高さ 500px を超えるサイズだとエラー扱いになります。)
プラグインの一覧として使用されるので 512x288 の画像(README.md などに挿入する画像はサイズ対象外)なので複数画像を用意するといいでしょう。 
![Pasted image 20250806014546.png](/img/user/Pasted%20image%2020250806014546.png)


参考:
https://docs.obsidian.md/Themes/App+themes/Submit+your+theme