---
title: "StorageのSATA関連の設定でUbuntuが起動しなかった話"
date: 2022-09-20T12:54:45+09:00
draft: true
categories:
    - Tech
tags:
    - Linux
    - Tips
    - Storage
---

# 何が起こったか
研究室のノートPCでやたら定期的にOSが起動しなくなるPCが見つかった。<br />
実験で使うPCで非常に安定性がアレで困っていたので原因を調査することにした。<br />

# PCに起こっていた現象の詳細
どこまで起動するか、についていうとUEFIまでは起動する。<br />
で，どのブートローダを選ぶ画面に突っ込んでしまい、肝心のUbuntu本体を選んでも以下のエラーが出てきて起動しないといった感じです.

~~~
Gave up waiting for root device. Common problems:
 - Boot args (cat /proc/cmdline)
   - Check rootdelay= (did the system wait long enough?)
   - Check root= (did the system wait for the right device?)
 - Missing modules (cat /proc/modules; ls /dev)
ALERT! /dev/disk/by-uuid/XXXX does not exist. Dropping to a shell!
~~~

どうやら起動時にrootデバイスのがちゃんと立ち上がるのを確認出来なかった？みたいです. <br />
これはなんでなんだろう...?と思ったので調べました.<br />

# 解決法
結論としてはSATAストレージのインターフェースの設定をAHCIという新しいモノに切り替えれば解決シマス.<br />
やることは単純で，BIOSのSATAのインターフェーズのところをAHCIに変更するだけです. <br />

# なんでこうなったの？
これはStack Overflowで見かけただけなんですが、一定期間起動しないとPC側で自動的にSATAのインターフェース設定がIDEという汎用性が高いものの古いものに変更されることが原因のようです．<br />
AHCIとIDEについては<a href='https://www.partitionwizard.jp/clone-disk/ahci-vs-ide.html'>このサイト</a>を参考にするとわかりやすいです. <br />
つまり，AHCIは新しめの規格で新しいSATAの機能もりもりで新しいデバイスにも対応できるようです．（合ってる？）<br />
AHCIでOSがインスコされているのに，IDEで起動しようとするとストレージの速度が出ずLinux側が待ちきれなくなってUbuntuをうまく起動出来なかったのではと思っています。<br />
（運動会で張り切って全力疾走して体に気持ちがついてこなかった結果コケちゃうお父さん現象？）

