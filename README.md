# test static page/site generator "eleventy" test

## init

```zsh
% npm install
% npm install -g @11ty/eleventy
```

## memo

- [Official documentation](https://www.11ty.io/docs/)
- https://github.com/11ty/eleventy-base-blog
- https://medium.com/@11ty/making-a-simple-web-site-with-the-simplest-static-site-generator-level-1-7fc6febca1
- https://www.evoworx.co.jp/blog/11ty-static-site-generator/
- [temlate engine Liquid](https://shopify.github.io/liquid/)

## [test-1] just run % eleventy

ルートに index.html を設置し以下を run

```zsh
% eleventy
```

yaml front matter で指定した変数、配列は \{\}、\{% for 変数名 in 配列名 %\} で、/_site/index.html としてコンパイル。  
ただし README.md も /_site/README/index.html としてコンパイルされた。
img/ フォルダ配下の画像郡は特に何もされず。

## [test-2] 変換対象ファイル指定 --formats オプション

--formats オプションがあるので、

```zsh
% eleventy --formats=gif,jpg
```

とすると、今度は

- /_site/img/a.jpg
- /_site/img/b.jpg
- /_site/img/c.gif

が書き出された。--formats で明示的に拡張子を指定した場合は、 ```% eleventy``` でデフォルトで書き出される html や md ファイルは書き出されない。
今回は html ファイルは書き出してほしいわけで、

```zsh
% eleventy --formats=gif,jpg,html
```

とすると、

- /_site/index.html
- /_site/img/a.jpg
- /_site/img/b.jpg
- /_site/img/c.gif

想定どおり書き出された。

## [test-3] 書き出し先指定 --output オプション

指定なしだと書き出し先が _site になってしまう。  
書き出し先を指定したい場合は --output を指定する

```zsh
% eleventy --formats=gif,jpg,html --output=dist
```

- /dist/index.html
- /dist/img/a.jpg
- /dist/img/b.jpg
- /dist/img/c.gif

これで、dist ディレクトリに書き出せる。

## [test-4] コンパイル元ファイル指定 --input オプション

諸々のファイルはリポジトリ直下に設置していたので、規定のディレクトリ配下に移動し、編集するファイルはその配下に限定したい。  
index.html と img ディレクトリを src 配下へ移動し、

```zsh
% eleventy --formats=gif,jpg,html --output=dist --input=src
```

--input オプションで想定通りの挙動となった。

## [test-5] ベースとなる html 一元管理のための layout

Static page/site generator を利用する大きな利用として、共通する html を一元管理できる点があります。
他の主な Static page/site generator と同様、ベースとなる html を layout ファイルとして用意し、ユニークな html 部分を書き出したい html ファイルとして用意します。

eleventy ではデフォルトで _incudes ディレクトリが layout ディレクトリとして使えるので、

__src/_includes/base.html__

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>{{ siteTitle }}</title>
</head>
<body>
<h1>{{ siteTitle }}</h1>
  <ul>
    {% for filename in images %}<li><a href="img/{{ filename }}">{{ filename }}</a></li>
    {% endfor %}
  </ul>
{{ content }}
</body>
</html>
```

上のように共通部分の html が入ったファイルを作成、
次に

__src/index.html__

```html
---
siteTitle: Giffleball
images:
  - img/a.jpg
  - img/b.jpg
  - img/c.gif
layout: base.html
---
<h2>index.html</h2>
<p>may some text here.</p>
```

先程 /dist/index.html として書き出されれた src/index.html を、上のように編集します。

src/_includes/base.html の {{ content }} は予約語で、layout として呼び出されたファイルの html を挿入する為の記述ですが、{{ siteTitle }} や {% for filename in images %} の images は、src/index.html の yaml front matter で指定した値を入れることができます。

先程と同様のコマンドでコンパイルすると、

```zsh
% eleventy --formats=gif,jpg,html --output=dist --input=src
```

以下のような html が書き出されます。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Giffleball</title>
</head>
<body>
  <h1>Giffleball</h1>
  <ul>
    
    <li><a href="img/img/a.jpg">img/a.jpg</a></li>
    
    <li><a href="img/img/b.jpg">img/b.jpg</a></li>
    
    <li><a href="img/img/c.gif">img/c.gif</a></li>
    
  </ul>
<h2>index.html</h2>
<p>may some text here.</p>
</body>
</html>
```

最終的に html は minify すると思うので関係ないかもしれませんが、li タグの開業やインデントは {% %} を、{%- -%} のようにすることで whitespace control できます。

## [test-6] 共通の html を管理する layout

ベースとなる src/_includes/base.html に src/index.html でユニークな部分を書き込む以外に、ページの一部の html を一元管理したいケースでは、 ```{% include partial/**.**** %}``` を利用します。

設置ファイル場所は _includes 配下ならどこでも良いですが、管理上わかり易くする為に partial ディレクトリを作成し、そこに設置することにします。

__src/_includes/partial/ad.html__

```ad.html
<aside>this is ad</aside>
```

__src/_includes/base.html__

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>{{ siteTitle }}</title>
</head>
<body>
<h1>{{ siteTitle }}</h1>
  <ul>
    {%- for filename in images -%}<li><a href="img/{{ filename }}">{{ filename }}</a></li>
    {%- endfor -%}
  </ul>
{{ content }}
```{% include partial/ad.html %}```
</body>
</html>
```

いつものコマンドで

```zsh
% eleventy --formats=gif,jpg,html --output=dist --input=src
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Giffleball</title>
</head>
<body>
  <h1>Giffleball</h1>
  <ul>
    <li><a href="img/img/a.jpg">img/a.jpg</a></li><li><a href="img/img/b.jpg">img/b.jpg</a></li><li><a href="img/img/c.gif">img/c.gif</a></li>
  </ul>
<h2>index.html</h2>
<p>may some text here.</p>
<aside>this is ad</aside>
</body>
</html>
```

ad.html の <aside /> タグが追加されました。

