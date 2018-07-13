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

が書き出された。--formats で明示的に拡張子を指定した場合は、 ```% eleventy``` でデフォルトで書き出される html や md ファイルは書き出されない。
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

## [test-4] コンパイル元ファイル指定 --input オプション

諸々のファイルはリポジトリ直下に設置していたので、規定のディレクトリ配下に移動し、編集するファイルはその配下に限定したい。  
index.html と img ディレクトリを src 配下へ移動し、

```zsh
% eleventy --formats=gif,jpg,html --output=dist --input=src

```

--input オプションで想定通りの挙動となった。

