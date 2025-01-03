---
{"dg-publish":true,"permalink":"/obsidian/obsidian-digital-garden/twitter-card/","metatags":{"og:title":"Twitter card の出し方","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Twitter card の出し方","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"noteIcon":"","created":"2024-12-30T01:45:34.151+09:00"}
---


#obsidian #twitter #X #obsidial-digial-garden

## 対応方法
Obsidian の Property に `dg-metatags` を追加して Twitter Card を表示するうえで必要な meta タグを設定していくことで、Twitter card に対応できる。

```markdown
---
dg-publish: true
dg-home: false
dg-metatags:
 "twitter:card": "summary_large_image"
 "twitter:title": "This is title"
 "twitter:image": "https://path/to/image.jpg"
 "twitter:site": "@konnta0"
---
```

`twitter:card` などの細かい指定内容について[ドキュメント](https://developer.x.com/ja/docs/tweets/optimize-with-cards/guides/getting-started)を参照

このページでは以下のような設定をしている。
```markdown
dg-publish: true
dg-home: false
dg-metatags:
 "og:title": "Twitter card の出し方"
 "og:image": "https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg"
 "twitter:card": "summary"
 "twitter:title": "Twitter card の出し方"
 "twitter:image": "https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg"
 "twitter:site": "@konnta0"
```

表示としては以下のようになる。
![](/img/user/Obsidian/obsidian-digital-garden/Twitter-card.png)


### 蛇足... OGP の対応
尚、OGP 画像に対応する場合は `og:image` を設定する必要があるが、`dg-metatags` は設定されている key / value で `<meta name="{{ key }}" content="{{ value }}">` で出力されるため本来の OGP の設定上必要な `<meta property...` ではないため、テンプレートの修正が必要そう。
また、`head` タグに `og: https://ogp.me/ns#` もデフォルトではついていないため、そちらも修正が必要そう。

修正箇所は以下 ... 
#### meta タグの修正箇所
https://github.com/oleeskild/digitalgarden/blob/1.61.3/src/site/_includes/components/pageheader.njk#L33-L37

雑に修正した感じは以下で動きそうだった
```
{% if metatags %}
    {% for name, content in metatags %}
        {%- if name and name | replace("og:", "") != name %}
            <meta property="{{ name }}" content="{{ content }}">
        {% else %}
            <meta name="{{ name }}" content="{{ content }}">
        {%endif%}
    {% endfor %}
{% endif %}
```

#### head タグの修正箇所
https://github.com/oleeskild/digitalgarden/blob/1.61.3/src/site/_includes/layouts/note.njk#L6
https://github.com/oleeskild/digitalgarden/blob/1.61.3/src/site/_includes/layouts/index.njk#L3
