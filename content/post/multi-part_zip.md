---
title: "Multi-Part zipをどうやってlinuxで解凍するか？についての解答"
date: 2022-10-13T19:39:00+09:00
draft: true
categories:
    - tech
---

# Multi-Part zipをどうやって解凍するか？
研究でドデカデータセットをダウンロードすることは多々あります。<br />
で、唐突にこいつがやってきました。multi-part zipくん。<br />
~~~
hoge.zip hoge.z01 hoge.z02
~~~
なんだこいつ。<br />
こいつをいい感じに解凍する方法を調べたところwindowsでアプリ使おう！みたいなのが多くてムカついたのでLinuxコマンドでやる方法を調べたのでまとめます。

## その解答
以下のコマンドでまずたくさんのzipファイルをまとめます。<br />
~~~
cat hoge.z* > ./hogehoge.zip
~~~
すると複数の謎z0Xなどのファイルが一つにまとまってドデカZipファイルになります。<br />
であとはこれを解凍するだけです。<br />
~~~
unzip hogehoge.zip
~~~
以上でした。

## 参考サイト
https://unix.stackexchange.com/questions/40480/how-to-unzip-a-multipart-spanned-zip-on-linux
