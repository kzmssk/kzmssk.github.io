---
title: 'トッピクス強化学習: [4] Advantage'  
date: 2025-03-23T00:00:00+00:00  
params:  
  math: true  
---

[前回の記事](/posts/rl/value-and-policy/)では状態価値 \(V(s_t)\) と行動価値 \( Q(s_t, a_t) \) の説明をしたが、強化学習（RL）ではもう1つ重要な価値がある。それはAdvantageだ。これは単に \(Q\) から \(V\) を引いた値である。

$$
A(s_t, a_t) = Q(s_t, a_t) - V(s_t)
$$

直観的には、「ある状態 \( s_t \) において行動 \( a_t \) を選択したときの、他の行動と比べたときの相対的な価値」だ。Advantageは[制御変量法(Control Variates)](https://en.wikipedia.org/wiki/Control_variates)的に \( Q \) や \( V \) に比べて分散が小さく、機械学習による推定に比較的向いている。

## Advantageの近似方法

Advantageの近似にはさまざまな方法がある。たとえば、\( Q \) の代わりに状態 \( s_t \) で行動 \( a_t \) を選択した場合の実際の報酬和 \(g(s_t, a_t) \) を使い、また状態価値をモンテカルロ法で推定した上で \( A(s_t, a_t) \approx g(s_t, a_t) - V(s_t; \phi) \) とし、重みづけを加えて方策の学習をするのが [Advantage Weighted Regression (AWR)](https://arxiv.org/abs/1910.00177) である。TD法による近似を考えると、AdvantageはTD誤差の推定量であるともみなせる。つまり \( A(s_t, a_t) \approx r_t + V(s_{t+1}) - V(s_t) \) である。モンテカルロ法とTD法の間をとったのが [Generalized Advantage Estimation (GAE)](https://arxiv.org/abs/1506.02438) である:

$$
A(s_t, a_t) \approx \sum_{l=0}^{T} \lambda^l \delta_{t+l}
$$

ただし、\( \delta_t = r_t + V(s_{t+1};\phi) - V(s_t;\phi) \) である（実際には割引率 \( \gamma \) が入るが省略してある）。\( V(s_t;\phi) \) はパラメータ \(\phi\) をもつニューラルネットワークによる価値をモンテカルロ推定した値である。パラメーター \( \lambda \) は0から1までの連続値なハイパーパラメーターで、\( \lambda = 0 \) の場合は\( A(s_t, a_t) \approx r_t + V(s_{t+1};\phi) - V(s_t;\phi) \)として近似することに対応する。一方、\( \lambda = 1 \) の場合は\( A(s_t, a_t) \approx g(s_t, a_t) - V(s_t; \phi) \)に対応する。\( \lambda \) を調整することで両者の間をとった近似ができるわけだ。GAEはRLの有名なアルゴリズム [Proximal Policy Optimization(PPO)](https://arxiv.org/abs/1707.06347) でも使われている。

これらのAdvantage近似は\( Q \) や \( V \) を精度よく近似できることが前提だが、これ自体の分散が大きいという問題がある。そこで直接Advantageをニューラルネットワークに推定させてしまおう、というのが [Direct Advantage Estimation (DAE)](https://arxiv.org/abs/2109.06093) だ。方策 \( \pi(a_t|s_t) \)で行動選択した場合のAdvantage \( A^\pi (s_t, a_t )\) を行動 \( a_t \) 上で積分すると0になる \(\pi\)-centered（\( \sum_{a_t} \pi(a_t|s_t) A^\pi (s_t,a_t) = 0\)になる）という性質を持つことに着目し、以下のような損失関数を提案している:

$$
L(\phi) = \frac{1}{N} \sum_{\tau=\tau_1,\cdots,\tau_N} \bigl(\sum_{t=0}^{T(\tau)}(r_t - A(s_t, a_t;\phi))\bigr)^2
$$

ただし、\( A(s_t, a_t;\phi) \)はエピソード \( \tau \) を集める方策について\(\pi\)-centeredであることが要求される。この要求はニューラルネットワークの出力ベクトルを方策 \( \pi \) について正規化する、つまり\(f_\phi(s_t, a_t)\)をネットワークの出力として \( A(s_t, a_t) = f_\phi(s_t, a_t) - \sum_{a_t} \pi(a_t|s_t) f_\phi(s_t, a_t) \)とする [Dueling Architecture](https://arxiv.org/abs/1511.06581)を使えばよい。

オンポリシーであるDAEをオフポリシーに拡張したのが [off-policy DAE](https://arxiv.org/abs/2402.12874)だ。この手法ではAdvantageを状態遷移確率に伴う成分と、DAEで扱ったような方策による行動推定にともなう成分の2つに分けた以下の定式化を行う:

$$
V(s_0) + \sum_{t=0}^{T}\bigl(A(s_t, a_t) + B(s_t, a_t, s_{t+1})\bigr) = G
$$

ここで \( A \) はAdvantageであり、DAEと同様に \( \pi \)-centered である必要がある。ただし、\( B \) は

$$
B(s_t, a_t, s_{t+1}) = V(s_{t+1}) - \mathbb{E}_{s' \sim p(\cdot|s_t, a_t)} \bigl[V(s')\mid s_t,a_t\bigr]
$$

であり、これは状態遷移関数を方策とみなしたときのAdvantageに対応している。つまり、\( A \)はエージェントの方策による価値(Skill)であり、\(B\) は行動選択の後の環境の変化による価値(Luck)であると解釈できる。Aと同様に\(B\)は状態遷移確率について\( \pi \)-centered である必要がある。つまり、\(\sum_{s'} B(s,a,s') p(s'|s,a) = 0\)を満たす必要がある。これはあり得る未来の状態すべてを求めなければ陽に求められない。そのため実際には[変分オートエンコーダー](https://arxiv.org/abs/1312.6114)などを使った状態遷移確率の近似が必要になる。

Advantageや行動価値を推定することは、フィードバックとして与えられる価値を行動に適切に割り当てるCredit Assignment Problemであるが、行動に対して学習しやすい形で報酬が与えられることは必ずしもない。たとえばサッカーをプレイしていて、とても下手なプレイをしているが味方がとてもうまかったり、カードゲームをしていて山から引いた札がよい札で都合が勝ててしまった場合には、エージェント（プレイヤー）の行動決定が巧みでない、むしろ悪い場合でも正の報酬が与えられてしまう。off-policy DAEはDAEの拡張という形で導出されるが、このようなラッキー由来の価値を明示的に分離できるアルゴリズムになっている。

## Baseline

AdvantageはREINFORCEにおける実際の利得 \( g(s_t, a_t) \) の代わりによく使われ、このときに \( V \) は分散を抑えるという意味で **Baseline** と呼ばれる。このBaselineは上記のGAEやAWRのようなモンテカルロ法による状態価推定以外にも近似が可能で、これを使ったのが [Group Relative Policy Optimization(GRPO)](https://arxiv.org/abs/2402.03300) だ。このアルゴリズムは大規模言語モデル(LLM)の追加学習法として提案された。既存の追加学習では状態価値をLLMの生成モデルと同様なたくさんのパラメーターをもつニューラルネットワークで近似していたが、メモリ消費が大きかった。そこでGRPOはBaselineを1つの質問文に対して生成した複数の結果のスコア（報酬）の平均をBaselineとしてみなすことで効率化を図っている。

もちろんバイアスを許容することが前提だが、Baselineは価値に相関する量であれば他にもさまざまな量で近似できる可能性がある。Baselineを \( V(s_t) \) とすることは、最適化を行うマルコフ過程(MDP)の全体を見渡した上での行動価値の相対化である。一方、GRPOのように複数の回答バリエーションの平均を使うことは、つまり具体的な質問文＝状態 \( s_t \) とありうる回答 \( a_t \) に対するローカルな組み合わせに絞った相対化であるとみなせる。現実世界のような広大で複雑な環境ではそもそも、全体を見渡した上での価値を推定すること自体が困難なため、何かしらの形で仮定を置きBaselineの範囲を絞り込まないと行動の相対化が難しいのではないか。そうだとしたらBaselineの定義は意思決定におけるある種の仮定を置くようなものであるのかもしれない。