---
title: 'トッピクス強化学習: [1] はじめに'
date: 2025-03-08T00:00:00+00:00
params:
  math: true
---

昨今では深層ニューラルネットワークのアプリケーションが注目を集め、さまざまなサービスに組み込まれつつある。特に、ChatGPTを皮切りにした言語モデル（LLM）によるコードや文章の生成、意思決定は社会に大きな影響を与えている。

言語モデルは、自然言語やコード、ときには画像のトークン列を入力として、次に行うべきことやコマンドを生成する意思決定システムとしても使うことができる。実際に[Claude3.7がポケモンをプレイ](https://www.twitch.tv/claudeplayspokemon)できたりする。よく学習されたLLMは高度な知識を持ち、適切な計画を立てることができる。

強化学習（Reinforcement Learning; RL）は、機械学習における意思決定システムを作るための代表的なパラダイムである。RLは、ゲームやシミュレーションなどの外部システムと相互作用しながら、トライ＆エラーを通じて学習する。深層学習が登場する以前から研究されてきたが、[Deep Q-Network](https://arxiv.org/abs/1312.5602)以後、深層学習と組み合わせた深層強化学習（Deep Reinforcement Learning; DRL）の分野が発展し、さまざまな研究が行われている。LLMのPre-Trainingのように大規模データセットを使った学習とは異なり、RLは試行錯誤を通じてデータを生成する。つまり、自律的に賢くなるシステムである。

RLの応用例は色々とある。たとえば[自動運転](https://arxiv.org/abs/2311.01043)やロボットの行動決定を行うニューラルネットワークの学習がある。ゲームの自動プレイでは、StarCraft IIの[AlphaStar](https://www.nature.com/articles/s41586-019-1724-z)や、囲碁・将棋の[MuZero](https://arxiv.org/abs/1911.08265)などが有名である。また、LLMの追加学習手法としても利用され、[Reinforcement Learning from Human Feedback（RLHF）](https://arxiv.org/abs/1706.03741)として知られている。

この「トピックス強化学習」では、RLの基本的な説明を交えながら、個人的に気になっているトピックについて書いてみようと思う。厳密な定義や証明よりも、これからの可能性や他の手法との関わりなどをメインに扱う予定。

より厳密な定義や入門書を読んでみたければ、以下の本が参考になるかもしれない：

- [Reinforcement Learning: An Introduction](https://mitpress.mit.edu/9780262039246/reinforcement-learning/)：有名な教科書。著者名から「Sutton本」とも呼ばれる。
- [強化学習](https://www.kspub.co.jp/book/detail/5155912.html)：厳密な定義や公理について知りたいならおすすめ
- [これからの強化学習](https://www.morikita.co.jp/books/mid/088031)：少し古くなってしまったが具体的な事例も含まれており、入門書としておすすめ