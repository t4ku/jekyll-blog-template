--- 
layout: post
title: "ブログをJekyll(on Heroku)にしてみる"
tags: 
- Programming
- Jekyll
- heroku
- rack
status: publish
type: post
published: true
---

気づけば半年以上ブログを更新してませんでした。

なんでかな〜と考えると、まあ仕事で結構いっぱいいっぱいだったのも少しありつつ、Twitterっていうまとまってないアイデアをだだ漏れさせるサービスが出てしまったからというのが大きいかなと思います。

iPhoneからツイートボタン押すっていうのに比べると、ある程度言いたい事まとめてWordPressのゴミゴミしたWYSIWYGエディタ(が使えないからHTML入力画面)に打ちこんでいく作業ってどう考えてもヨッコラッショ感が違いすぎる。。。

ただ、やっぱ昭和生まれとしてはTwitterもFacebookもどうしても情報の消費者感が否めない気もするので、
ストレスフリーでブログ書ける環境を整えてみました。

## Jekyllとは?

GithubのCoFounderのmjomboこと[Tom Preston-Werner](https://github.com/mojombo)さんによる、シンプルな静的サイトジェネレーターで、
下記のような特徴があります。

 * [Liquid](http://liquidmarkup.org/) (erbみたいなもんです)を使用して記事やページのレイアウトを定義できる。
 * Markdown/Textileで書かれたファイルは自動的にHTMLへの変換してくれる。
 * カテゴリやタグ、使用するレイアウトなどのメタデータは、ファイル先頭にYAMLで記載([Yaml Front Matter](https://github.com/mojombo/jekyll/wiki/yaml-front-matter))。
 * jekyllコマンドでhtml変換だけじゃなく、プレビュー用のWebサーバー起動してくれる。

要は記事をtextileやmarkdownで書いたら、コマンド一発でブログっぽいHTML構成にして
吐き出してくれますよーというツールです。(設定によって返られますが)

## ホスティング環境

HTMLファイルをホストするサーバーはどこでもいいんですが、ftpしたりブラウザでアクセスしたりするのはイラっとするので、
gitでpushしたらリリースできるとこがよいかと思います。

自分の場合はherokuが使いやすいので、herokuでの手順を書いておきます。

## Jekyllのインストール

{% prism clike %}
gem install jekyll
{% endprism %}

コマンド入れなきゃ始まらないので、インストール。
bundlerの人はGemfileに追加&&bundle install

## プロジェクトの雛形をclone

一からレイアウトを作っていってもいいのですが、それなりに面倒なので
下記のような雛形プロジェクトから作成します。

[Jekyll Bootstrap](http://jekyllbootstrap.com/)  
[krisb/jekyll-template](https://github.com/krisb/jekyll-template)

ちなみに自分は、CSSにSASS(Compass)を使用しているのが好みだったので、後者のプロジェクトから
拝借しました。

{% prism clike %}
git clone https://github.com/krisb/jekyll-template.git
cd jekyll-template
{% endprism %}

## ディレクトリ構成

下記が典型的なJekyllを使用したプロジェクトのディレクトリ構成です。

{% prism clike %}
.
|-- _config.yml
|-- _includes
|-- _layouts
|   |-- default.html
|   `-- post.html
|-- _posts
|   |-- 2007-10-29-why-every-programmer-should-play-nethack.textile
|   `-- 2009-04-26-barcamp-boston-4-roundup.textile
|-- _site
`-- index.html
{% endprism %}

jekyllはjekyllコマンドを実行したディレクトリ配下に、_siteディレクトリを作成してhtmlを吐き出すだけで、
ほとんどと行ってよいほど決まり事はありませんが、下記のような規約があります。

* _config.yaml
  * 設定ファイルですね。デフォルトではgaのアカウントやサイトのタイトルなどがありますが、YAMLなので他ファイルから参照したい設定があれば簡単に追加出来ます
* _includes
  * フォームとか共通で使い回したいHTMLを置いとけば、テンプレートからインクルードできます。
* _layout
  * HTML&amp;Liquidで作成するレイアウト定義
* _posts
  * 記事の元ネタ。yyyy-mm-dd-title.markup(markdown/textile/html)という形で、ファイルを追加していきます。
* _site
  * 変換後のHTMLファイル一式の出力先


## レイアウトとデザイン作成

ブログに必要な最低限のレイアウトをとりあえず作ります。
まずデフォルトのレイアウトを作成します。


_layout/default.html
{% prism markup %}
{% raw %}
<!DOCTYPE html>
<html>
  <head>
    <title>{% if page.title %}{{ page.title }} - {% endif %}{{ site.title }}</title>
    {% if page.description %}
    <meta name="description" content="{{ page.description }}" />
    {% endif %}
    ...
  </head>
  <body>
       <div id="header">
         <div id="header-item-area">
          <div id="logo-area"><a href="/">WYSAWYG</a></div>
          <nav>
          <ul id="menu" class="clearfix">
             ....
          </ul>
          </nav>
         </div>
      </div>
        {{ content }}
    </div>
    ....
  </div>
  </body>
</html>
{% endraw %}
{% endprism %}

変わってるのはLiquidのタグだけかと思います。[Liquid for Designers](https://github.com/shopify/liquid/wiki/liquid-for-designers)に書いてますが、

 * {% raw %}{{ }}{% endraw %}はテキスト出力あり
 * {% raw %}{{% %}}{% endraw %}はコード評価するだけ

です。

またjekyllで定義している変数として、contentがありこれはサブビューの内容を取得します。  


次に、下記個別ページおよび、レイアウトを作成します。

* インデックスページ
* 記事ごとのページレイアウト

まずはインデックスページ。

<img width="480px" src="/img/wysawyg.png" /><br/>

./index.html
{% prism markup %}
{% raw %}
---
layout: default
description: A description about my blog homepage
---
<div id="wrapper">
  <div id="content">
    <div id="posts">
      <h2>Blog Posts</h2>
      <ul>
        {% for post in site.posts %}
        <li><span class="postdate">{{ post.date | date_to_string }} &gt;&gt; </span><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endfor %}
      </ul>
    </div>

    <div id="pages">
      <h2>Pages</h2>
      <ul>
        {% for page in site.html_pages %}
          {% if page.title %}
            <li><a href="{{ page.url }}">{{ page.title }}</a></li>
          {% endif %}
        {% endfor %}
      </ul>
    </div>
  </div>
</div>
{% endraw %}
{% endprism %}

先ほど_layout/default.html内で使用している変数contentは「サブビューの内容を取得します」と書きましたが、
jekyllではファイルの先頭の定義部分で"layout:
default"などと指定する事によりそのファイルが使用するレイアウトを設定できます。  

そして、参照されている側のファイルからは、変数contentから内容を取得できます。

最後に、記事ごとのページレイアウトを作成します。  

_layout/posts.html
{% prism markup %}
{% raw %}
---
layout: default
---
<div id="wrapper">
  <div id="content">
    <div class="post">
      <div class="post-title">
    {% if page.title %}<h1>{{ page.title }}</h1>{% endif %}
    {% if page.date %}<div class="postdate">Posted on {{ page.date | date_to_string }}{% if page.last_updated %}, last updated on {{ page.last_updated | date_to_string }}{% endif %}</div>{% endif %}
      </div>
    {{ content }}
    ...
    </div>
  </div>
</div> 
...
{% endraw %}
{% endprism %}

インデックスページとの違いを見てみると、変数contentがあるところですが、
理由は個別記事のレイアウト指定で使用したいからです("layout:posts")。

 * _ layout/default.html -&gt; (content) -&gt; index.html
 * _ layout/default.html -&gt; (content) -&gt; _layout/post.html -&gt; (content) -&gt; 個別記事

## ローカルで記事作成/プレビュー

これまでの手順は初回のみ必要になるものでしたが、実際に記事を追加する際の手順は
_posts/以下にファイル追加して、生成されたhtmlをリリースするだけです。

下記コマンドを実行すると、http://localhost:4000で最終的なHTMLが確認できますし、
Markdown/Textileファイルに変更があるたびにHTML生成し直してくれます。

{% prism clike %}
jekyll --pygments --auto --server
{% endprism %}

ちなみにこの記事は、こんな感じです。

_posts/2012-03-22-blogging-jekyll-on-heroku.md
{% prism clike %}
{% raw %}
--- 
layout: post
title: "ブログをJekyll(on Heroku)にしてみる"
tags: 
- Programming
- Jekyll
- heroku
- rack
status: publish
type: post
published: true
---
気づけば半年以上ブログを更新してませんでした。

なんでかな〜と考えると、まあ仕事で結構いっぱいいっぱいだったのも少しありつつ、Twitterっていうまとまってないアイデアをだだ漏れさせるサービスが出てしまったからというのが大きいかなと思います。

iPhoneからツイートボタン押すっていうのに比べると、ある程度言いたい事まとめてWordPressのゴミゴミしたWYSIWYGエディタ(が使えないからHTML入力画面)に打ちこんでいく作業ってどう考えてもヨッコラッショ感が違いすぎる。。。

ただ、やっぱ昭和生まれとしてはTwitterもFacebookもどうしても情報の消費者感が否めない気もするので、
ストレスフリーでブログ書ける環境を整えてみました。

## Jekyllとは?

GithubのCoFounderのmjomboこと[Tom Preston-Werner](https://github.com/mojombo)さんによる、シンプルな静的サイトジェネレーターで、
下記のような特徴があります。

....
{% endraw %}
{% endprism %}

## Herokuへリリース

Herokuは一応アプリのプラットフォームなんで静的ファイルをpushしてもなんにもおきません。
ということで、静的ファイルを返すアプリ(というか単なるRackup)をpushするリポジトリに追加します。

config.ru
{% prism ruby %}
require "rack"
require "rack/contrib/try_static"
require "./try_static_patch.rb"

use ::Rack::TryStatic,:root => "_site",
                      :urls => %w[/],
                      :try  => ['.html','index.html','/index.html']

run lambda { [404, {'Content-Type' => 'text/html'}, ['Not Found']]}
{% endprism %}

静的ファイル返すかつDirectoryIndexっぽい事ができそうなRackミドルウェア(TryStatic)を使っていますが、
なぜかHerokuだと、無効なパスを指定した際に500エラーになる問題があったので、モンキーパッチしてるのが3行目。

※Rackでrewriteエンジンっぽい事をできるミドルウェアがあったので、そっちの方がきれいかも。。。  
 [rack-rewrite](https://github.com/jtrupiano/rack-rewrite)

追加できたら、_site以下をリポジトリに反映してpushするだけ。パチパチ!
(あ、下記コマンドはそもそもherokuアプリを作成していて、git
remoteにherokuがあることが前提です。)

{% prism clike %}
git push heroku
{% endprism %}

ちなみに、このブログのリポジトリ自体は下記で見れます。  
[t4ku/jekyll-template](https://github.com/t4ku/jekyll-template)

## あとがき

[github pages](http://pages.github.com/)のユーザーであれば、"username.github.com"というリポジトリを作成してindex.htmlを置けば、http://username.github.comでアクセスした際に、そのhtmlが表示されるという仕掛けがあります。また、github pagesはhtmlだけではなくmarkdown/textileもちゃんとhtmlに変換して表示してくれます。

実はjekyllはそもそもgithub pageを動かすライブラリなので、jekyllで作ったサイトはhtml変換前のもの(_site以外)をgithub pagesのリポジトリにpushするだけで使用できます。
