---
title: "Earth Mover's Distanceの2次元行列版をPytorchで実装した"
date: 2022-05-28T15:50:46+09:00
draft: true
categories:
    - Tech
Tags:
    - Pytorch
---

# EMDについて
Earth Mover's Distance(EMD)は分布間の距離を表す指標の一つです。<br>
KL距離などと似ていますが、EMDは重み、という概念を用いる部分が肝ですね。重みと距離を表すものですので、MLの分野におけるLossとして用いることができそうですね。<br>
例えば、GANでは積極的に用いられており、W-GANでは連続値版のEMDである、Wassernstein Lossという名前で用いられています。<br>

# なぜ今回2次元行列版を作ったか
研究で尤度マップをモデルに生成させており、その学習に用いてみようと思い立ったためです。<br>
単純に考えれば、正解ラベルの尤度マップとのMSELoss(二乗誤差損失)でも問題ないようにも思いますが、肝心の尤度の分布のズレ等は反映されない問題があります。

# 実装
結構単純です<br>
~~~python
def EMDLoss(self,y_pred, y_true):
	integral_y_true = torch.cumsum(torch.cumsum(y_true, dim=3), dim=2) 
	integral_y_pred = torch.cumsum(torch.cumsum(y_pred, dim=3), dim=2)
	square = torch.square(integral_y_true - integral_y_pred)
	square_sum = torch.sum(torch.sum(square, dim=3), dim=2)
	mean = torch.div(square_sum, len(y_pred[0][0])**2)
	distance = torch.mean(mean)
	return distance 

~~~
二次元行列の積分画像的なものを作成し、その差分を取り、二乗和を取ります。あとはそのBatchの平均を取ってLossとします。
(間違いがあれば教えて下さい。)<br>


# 現状
計算量はGPU上で行ってもそこそこ重めです。速度の計測はしていませんがそこそこ遅いので我慢してください。
ただし、結構分布としては近づくように学習は進んでいるので意図通りのものはできたのかなと思います。

# 参考文献
https://discuss.pytorch.org/t/implementation-of-squared-earth-movers-distance-loss-function-for-ordinal-scale/107927
