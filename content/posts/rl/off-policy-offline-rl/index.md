---
title: 'トピックス強化学習: [6] オフポリシー強化学習とオフライン強化学習'
date: 2025-04-12T00:00:00+00:00
params:
    math: true
---

## オフポリシー強化学習

深層学習を用いた価値および方策の近似に基づく強化学習（RL）は、多様かつ大量のデータサンプルを必要とする。一方、分散かつ非同期の探索は計算効率が高く、スケーラビリティに優れている。このため、データ収集時に用いる方策と最適化対象の方策が異なる条件設定はオフポリシーRLと呼ばれ、実際の探索ではBehavior Policy、学習時にはTarget Policyが用いられる。これら2つの方策間の乖離（Policy Gap）は、Target Policyの学習の妨げとなる。

Policy Gapを求める手法として重点サンプリング (Importance Sampling) が利用される。例えば、ある確率分布 \( \pi(x) \) の期待値 \( \mathbb{E}_\pi [x] \) を、別の分布 \( b(x) \) に基づいて近似すると以下のようになる:

$$
\mathbb{E}_\pi[x] \simeq \sum x \pi(x) = \sum x \frac{b(x)}{b(x)} \pi(x) = \mathbb{E}_b \Bigl[\frac{\pi(x)}{b(x)} x\Bigr]
$$

Target Policy の代わりに Behavior Policy でサンプリングし、その違いを後で \( \pi(x) \) により補正する。この手法では方策比を用いて乖離を埋めるが、Policy Gapが大きい場合は分散が増大するため、分散低減のための各種手法が提案されている。たとえば、分散低減を狙った方策勾配法を用いるIMPALAで採用されるRetraceでは、方策比に一定のクリップを加えて安定化を図る。他にも軌道全体の比率を利用する方法がある。

Policy Gapの緩和策として、更新量を制限する保守的な方策更新がある。代表例としてTrust Region Policy Optimization（TRPO）が挙げられる。TRPOは、データサンプルにおける価値最大化とBehavior PolicyとのKL距離の最小化という2つの目的を組み合わせた損失を用いる。この考えをシンプルにしたのがProximal Policy Optimization (PPO)で、方策比 \( \pi(a|s) / b(a|s) \) が一定範囲に収まるかどうかで損失の形状を調整する。なお、TRPOおよびPPOは主にオンポリシーRLに分類されるが、オンラインRLにおいてもPolicy Gapの影響は避けがたい問題である。実際、PPOの安定性は大規模言語モデルの追加学習手法である [Deep reinforcement learning from human preferences (HFRL)](https://arxiv.org/abs/1706.03741) にも寄与している。

## オフライン強化学習

オフラインRLは、さらに大きなPolicy Gapを伴う手法で、Behavior Policy による探索を行わず、事前に収集したデータセットを用いたオフライン学習を行う。これにより、探索コストが極めて高いロボティクスや医療分野などへの応用が期待される。

静的なデータセットの既存状態を入力とし、それに伴う方策（例：人間の行動）を教師信号とするBehavior Cloning (BC) は、オフラインな学習手法の一例である。BCはシンプルな手法だが、最適な場合でもエピソード長の二乗に比例してデータセットを構築する方策との乖離が生じることが示されている。直感的には、ある時点での小さな誤差が次時刻の状態や行動推定の誤差へと波及するためであり、この乖離はDistributional Shiftと呼ばれる。オフラインRLでは、報酬を組み合わせることでこのDistributional Shiftを低減することが試みられている。

オフラインRLは、データセット \( D \) を作成した未知のBehavior Policy \( b(a|s) \) を模倣するBC-Objectiveと、多少の乖離を許容してでも報酬を獲得するReward-Maximization (RM) Objectiveという2つの目的を同時に達成する必要がある。たとえば、[Advantage Weighted Regression (AWR)](https://arxiv.org/abs/1910.00177) は、Policy Searchに着想を得た重み付けBC損失を用いる手法であり、BCの制約付き方策勾配法とみなすことができる:

$$
L(\phi) = - \log \pi (a|s;\theta) \exp \frac{1}{\beta} A(s, a)
$$

大規模言語モデル（LLM）の追加学習手法として提案された [Direct Preference Optimization (DDP)](https://arxiv.org/abs/2305.18290) は、BC-ObjectiveとRM-Objectiveを組み合わせた最適化問題として定式化される:

$$
\max_{\pi_\theta} \mathbb{E} [r(s, a)] - \beta \text{KL}\Bigl[\pi(a|s;\theta) \,\big|\big|\, \pi_{ref}(a|s)\Bigr]
$$

ここで論文では \( \pi_{ref} \) を事前学習済み言語モデルとしているが、これはBCモデルに相当するとみなせる。最適な方策は次式を満たす:

$$
\pi^*(a|s) = \frac{1}{Z(s)} \pi_{ref}(a|s) \exp \Bigl(\frac{1}{\beta} r(s, a)\Bigr)
$$

ただし、分配関数 \( Z(s) = \int_a \pi_{ref} (a|s) \exp(\frac{1}{\beta}r(s, a)) \) は、例えばLLMのように次元数が大きい場合、計算が困難となる。そこで、2つの行動（LLMであれば単語予測または回答文）を比較する分布として[Bradlley-Trerry Model](https://en.wikipedia.org/wiki/Bradley%E2%80%93Terry_model)を導入し、\( Z(s) \) を打ち消すことで、シンプルな損失関数を導出する。DPOは言語モデル向けの手法だが、[Behavior Preference Regression (BPR)](https://arxiv.org/abs/2503.00930) はこれを連続行動の方策に適用している。

また、データセットのサンプル分布から逸脱する（Out-Of-Distribution, OOD）予測に対して、ペナルティとしてRM-Objectiveに改良を加える手法も研究されている。[Conservative Q-Learning (CQL)](https://arxiv.org/abs/2006.04779) は、オフライン学習時のQ-Learningにおいて、データ内の行動に対する価値と比べ、OODな行動の価値を低く評価する保守的な損失を提案する:

$$
L(\phi) = \Bigl\| r(s, a) + \max_{a'} Q(s',a';\phi) - Q(s, a;\phi) \Bigr\|_2  + \Bigl(\mathbb{E}_{a \sim \mu} Q(s, a;\phi) - \mathbb{E}_{a \sim \pi } Q(s, a)\Bigr)
$$

1項目は一般的なQ-LearningのTD誤差、2項目は正則化項である。ただし、ここで \( \pi \) はデータセット作成に用いた（すなわちBCに相当する）方策、\( \mu \) はオフラインRLで学習される方策を意味する。この期待値部分は直接計算が難しいため、実際にはlogsumexp (\( =\log \sum_a \exp Q(s, a) \)) を用いて正則化される。[Implicit Q-Leaning](https://arxiv.org/abs/2110.06169) は、TD誤差の \( \max \) 項がOODな行動にも適用されることで過大評価を招く問題に着目し、BC方策が選択した行動から最大のQ値を教師信号として用いる。ただし、データセットに存在しない行動の評価は直接できないため、Expectile回帰でQ値の期待値を近似する。

BC-ObjectiveとRM-Objectiveを同時に達成する第3の方法として、データセット内に存在する複数の方策を組み合わせるアプローチがある。例えば、「鍵を拾って扉を開ける」というタスクでは、データセットに直接該当するデータがなくとも、「鍵を拾って歩く」と「起き上がってドアを開ける」といったデータを組み合わせることで目的を達成できる可能性がある。こうしたエピソードの組み合わせはTrajectory Stichingと呼ばれる。[前回](/posts/rl/model-based-rl-and-planning/)触れた[Diffusion Forcing](https://arxiv.org/abs/2407.01392) は、未来の行動系列を部分的に不確実なまま推論できるため、Trajectory Stichingに直接対応できるだろう。

[Decision Transformer](https://arxiv.org/abs/2106.01345) は、エピソード報酬和を条件とする自己回帰モデルを用いたBCにより、従来のオフラインRLを上回る性能を示したが、Trajectory Stichingが苦手という問題があった。[Elastic Decision Transformer](https://arxiv.org/abs/2307.02484) は、エピソード報酬和の最大値を推定する損失項を加え、行動選択時に異なる系列長のパターンから最も高いReturnを得られるものを採用することで性能向上を図っている。また、オフラインRLにTransformerを組み合わせる試みとして [Q-Transformer](https://arxiv.org/abs/2309.10150) なども提案されている。

## オフライン学習モデルのオンラインRL追加学習

オフラインで得た方策・価値モデルをオンラインで追加学習することで、更なる性能向上が期待される。しかし、損失関数やデータ分布の変化により学習が不安定になったり、破滅的忘却が生じるリスクがある。この問題に対しては、オフライン学習時にOODな行動価値を単に過小評価するのではなく、[信頼度を導入する手法](https://arxiv.org/abs/2212.04607)や、[破滅的忘却を防ぐ正則化手法](https://arxiv.org/abs/2402.02868)、[学習率の調整](http://arxiv.org/abs/2301.07302)、[BCモデルとのKL距離を用いた正則化](https://arxiv.org/abs/2206.11795)、[価値のスケール調整手法](http://arxiv.org/abs/2303.05479)など、各種工夫が試されている。

また、報酬ラベルが存在しない場合など、価値モデルがオフライン学習で獲得できないケースもある。[Jump-Start Reinforcement Leanring](https://arxiv.org/abs/2204.02372) は、エピソード前半はBC方策、後半はオンラインRL用の別モデルで学習し、両者の切り替えタイミングをスケジューリングすることで、最終的にオンラインRLモデル単独で行動選択を可能にする。この手法は、エピソード途中からの開始という観点で、重要な部分のみを探索する [Go-Explore](https://www.nature.com/articles/s41586-020-03157-9) を連想させる。