---
title: "Earth Mover's Distanceの2次元行列版をPytorchで実装した"
date: 2022-05-28T15:50:46+09:00
draft: true
---

# EMDについて
Earth Mover's Distance(EMD)は分布間の距離を表す指標の一つです。<br>
KL距離などと似ていますが、EMDは重み、という概念を用いる部分が肝ですね。重みと距離を表すものですので、MLの分野におけるLossとして用いることができそうですね。<br>
例えば、GANでは積極的に用いられており、W-GANではWassernstein Lossという名前で用いられています。<br>

# なぜ今回2次元行列版を作ったか
研究で尤度マップをモデルに生成させており、その学習に用いてみようと思い立ったためです。<br>
単純に考えれば、正解ラベルの尤度マップとのMSELoss(二乗誤差損失)でも問題ないようにも思いますが、肝心の尤度の分布のズレ等は反映されない問題があります。

# 実装
結構単純です。
'''
    def EMDLoss(self,y_pred, y_true):
        distance = torch.mean(torch.sum(torch.sum(torch.square(torch.cumsum(torch.cumsum(y_true, dim=3), dim=2) - torch.cumsum(torch.cumsum(y_pred, dim=3),dim=2)),dim=3), dim=2))
        return distance
'''
二次元行列の積分画像的なものを作成し、その差分を取り、二乗和を取ります。あとはそのBatchの平均を取ってLossとします。
(間違いがあれば教えて下さい。)<br>

# 現状
計算量はGPU上で行ってもそこそこ重めです。速度の計測はしていませんがそこそこ遅いので我慢してください。
ただし、結構分布としては近づくように学習は進んでいるので意図通りのものはできたのかなと思います。
