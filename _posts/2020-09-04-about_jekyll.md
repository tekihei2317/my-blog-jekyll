---
layout: posts
title: Jekyllのディレクトリ構成と、Jekyllでできること
category: Jekyll
---

## Jekyllとは

静的サイトを作るためのフレームワーク(Rubyのgem)です。記事を書くときにはmarkdown記法を使うことが出来ます。

以前Gatsbyという静的サイトジェネレータを少し使ってみたのですが、モダンフロントエンドはほとんど知らなかったので、ブラックボックスがたくさんあって辛かったです。

今回使ってみたJekyllはシンプルで、HTML/CSSの知識である程度理解できたので、HTML/CSSを知っていて、自分でブログを作ってみたいという人にはオススメかもしれません。

## Jekyllのディレクトリ構成

ディレクトリ構成は大体こんな感じになっています。

.  
├── _includes/  
│   ├── footer.html  
│   └── header.html  
├── _layouts/  
│   ├── default.html  
│   └── post.html  
├── _posts/  
│   ├── 2020-09-03-my_first_post.md  
│   └── 2020-09-05-my_second_post.md  
├── _site/  
└── index.html (or index.md)  


### 各ディレクトリの説明

* _includes

  複数のページで使う部品(ヘッダー、フッター等)を置く場所。

* _layouts

  ページの共通部分を定義する場所。

* _posts

  ブログの投稿を置く場所。フォーマットはYEAR-MONTH-DAY-title.mdにする。

* _site

  Jekyllが生成したサイトが置かれる場所。jekyll serveコマンドではこのディレクトリのファイルが公開される。

## Jekyllで出来ること

1. layout、includeを用いたページの共通化/部品化
1. markdownで記事を書ける
1. Jekyllに組み込まれた変数とLiquidを用いて(多分)色々出来る

<!-- dummy comment line for breaking list -->

### layout、includeを用いたページの共通化/部品化
ブログ記事を書く際、frontmatterでレイアウトファイルを指定すると、そこに記事の中身を挿入することが出来ます。
例えば\_layouts/default.htmlとmarkdownファイルが以下のようだったとします。

\_layouts/default.html
{% highlight html %}
{% raw %}

<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Jekyll Blog</title>
</head>
<body>
  {{ content }}
</body>
</html>

{% endraw %}
{% endhighlight %}

markdownファイル
{% highlight markdown %}
---
layout: default
---
1. ほげ
2. ふが
3. ほげふが
{% endhighlight %}

すると、Jekyllはビルド時に以下のようなファイルを生成します。
{% highlight html %}

<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Jekyll Blog</title>
</head>
<body>
  <ol>
  <li>ほげ</li>
  <li>ふが</li>
  <li>ほげふが</li>
</ol>
</body>
</html>

{% endhighlight %}
また\{\{ include ファイル名 \}\}とすることで、\_includes以下のファイルの中身を挿入することも出来ます。
### markdownで記事を書ける
Jekyllはmarkdownで記事を書くことができます。コードハイライトはLiquid(後述)を使うので、QrunchやCrieitにクロス投稿しようと思ったら、そこは修正しないといけなさそうです。



### Jekyllに組み込まれた変数とLiquidを用いて(多分)色々出来る

LiquidとはJekyllで使われるテンプレート言語です。Railsのerbのようにhtmlに制御文を埋め込むことが出来ます。
またLiquidからは、Jekyllに組み込まれた変数(site変数、page変数など)を使うことが出来ます。例えばsite.postsには全ての投稿の情報が日付の逆順に入っているので、以下のようにして投稿を表示することが出来ます。

コード
{% highlight liquid %}
{% raw %}
{% for post in site.posts limit:5 %}
  <li>{{ post.title }}</li>
{% endfor %}
{% endraw %}
{% endhighlight %}

実行結果
{% highlight liquid %}

{% for post in site.posts limit:5 %}
<li>{{ post.title }}</li>
{% endfor %}

{% endhighlight %}

## 感想

まだCSSはほとんど書いていませんが、記事が増えてきたらテーマとか参考にしながら書いてみようと思います(そのままテーマを使う気もします笑)。ブログ制作を通してレスポンシブ対応とかデプロイとか勉強できたらいいなと思います。

## 参考

* [公式サイト](http://jekyllrb-ja.github.io/)

  日本語に翻訳されています。

* [ドットインストール Jekyll入門](https://dotinstall.com/lessons/basic_jekyll)

  0から作っていくので分かりやすかったです。いきなり公式サイトを見てもあまり分からなかったので、こっちを先に見たほうがよかったなと思いました。