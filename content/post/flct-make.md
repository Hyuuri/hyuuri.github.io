---
title: "Flct Make"
date: 2022-06-13T16:49:17+09:00
draft: true
---

# File Count commandをとりあえず作る
今いるディレクトリのファイルの数をカウントしたい時がある。<br>
調べても既存コマンドの組み合わせしか出てこなくてウザい、いちいちオプションとか覚えてないので。<br>
とりあえずこれ打てばファイル数がわかるCLIのツールを作ってみた。

~~~
#!/bin/sh

while getopts r OPT; do
    case $OPT in
        r ) FLG_R=true ;;
    esac
done

if [ "${FLG\_R}" ]; then
	find . -type f|wc -l
else
	ls -U1 | wc -l
fi
~~~
動作としてはオプション指定ナシでカレントディレクトリのファイル数を表示（サブディレクトリ内のファイル数はカウントしない)<br>
-r オプションでサブディレクトリ内のファイル数まで再帰的に探索して表示する感じ<br>

これだけでとりあえずは動くので良しとする。<br>

あとは適当にパーミッションを777で設定して、.bashrcとか.bash\_profileとかにPATH通せばよろし。<br>

