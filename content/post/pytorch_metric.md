---
title: "Pytorch Metric Learningでのバッチサイズの話"
date: 2022-07-14T13:51:12+09:00
draft: true
categories:
- tech

tags:
- Pytorch
- Metric Learning
---

# Pytorch Metric Learningとは？
端的に言えば距離学習と呼ばれる機械学習タスクを扱うPytorchラッパーだ.<br>
操作感はかなりPytorch Lightningと近く、距離学習が簡単に実行できる. <br>
こいつはかなり楽でいいんだけれど、イマイチ設定周りは試行錯誤がいる、というのが実際だ.<br>
細かい説明は省くがそもそも距離学習は概念自体はわかりやすいが、様々な手法が存在するし、パラメータに大きく依存するところが大きく学習は結構大変だ. <br>
そこで色々試した結果わかったことのメモ書きと現状の謎を書いておこうと思う. <br>
随時付け足していく. <br>

# 基本的指針
- batch size は基本的に大きい方が良い<br>
	これはTripletの選択の存在が大きいと考えている. <br>
	つまり、多クラスの場合Tripletを作成する際にBatch Sizeが小さいとバリエーションが少ない可能性があるという話だ. <br> 
	20クラスと仮定すると、全てのクラス同士の比較をするTripletの組み合わせは単純に考えて190個ある.
	それを考慮するとBatch Sizeは最低でも570は必要である. <br>
	このことからすべてのtripletの組み合わせを一つのミニバッチに入れておきたいのであれば大きくした方が良いといえる. <br>
	これは少し違う分野だがContractive Learningだと良く行われているようで概ね間違っていないのではないかと考えている. <br>
	なお, Pytorch Metric LearningではBatch Sizeを大きくする際にエラーが出ることがある. <br>
	<a href="https://github.com/KevinMusgrave/pytorch-metric-learning/issues/367">ここ</a>にあるように、バッチサイズを小さくする以外に対処方がないようである.(2022/07/14現在)
	現状自分のデータセットであるとBatch Size=1024までは動作できていることを確認した. <br>
	なお、ミニバッチのサイズが大きくなると学習は進みにくなるので、epochは大きくすることを勧める.<br>


