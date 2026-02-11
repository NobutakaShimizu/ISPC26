---
theme: neversink
transition: none
layout: cover
title: 埋め込みクリーク問題に見る計算限界と統計限界のギャップ
info: |
  ## 埋め込みクリーク問題に見る計算限界と統計限界のギャップ
  STOC2025

author: Nobutaka Shimizu
mdc: true
css: unocss
style: |
  @import './styles/custom.css';
addons:
  - slidev-addon-rabbit
rabbit:
  slideNum: true 
fonts:
  sans: 'Roboto'
  mono: 'Fira Code'
  weights: '400,500,700'
  italic: true
favicon: 'https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6/svgs/solid/book.svg'
themeConfig:
  primary: '#1976d2'

---

# 埋め込みクリーク問題に見る<br>計算限界と統計限界のギャップ

<div class="grid grid-cols-2 gap-4 place-items-center h-56">

<div>

[**清水 伸高**](https://sites.google.com/view/nobutaka-shimizu/home) (Institute of Science Tokyo)

</div>
<div>

  <QRCode value="https://nobutakashimizu.github.io/ISPC26/1" :size="120" render-as="svg"/>

<div class="text-center text-blue-500 text-sm">

  $\uparrow$ [slide (github page)](https://nobutakashimizu.github.io/ISPC26/1)

  [論文のリンク](https://dl.acm.org/doi/10.1145/3618260.3649751)

</div>
</div>

</div>

:: note ::
<div class="text-slate-500">
  
  @[ISPC2026](https://sites.google.com/view/mimaizumi/event/ispcworkshop2026)

</div>

---
layout: top-title
color: amber-light
---

::title::

# 簡単な自己紹介

::content::

清水 伸高
- 分野：計算量理論
- 知ってること：
  - **平均時計算量（ランダムなインスタンスの計算量）**
  - （高次元）エクスパンダー（東北大数学科での[集中講義](https://nobutakashimizu.github.io/lecture_May2024/main.pdf)）
  - PCP定理 (RIMSで[集中講義](https://nobutakashimizu.github.io/PCP_lecture_2025/main.pdf))
  - 離散時間のマルコフ連鎖 (mixing timeまわり)

<div class="topic-box">

複雑性クラスなどの抽象的な議論より、具体的な議論が好き

</div>

---
layout: top-title
color: amber-light
---

::title::

# 今日の内容：埋め込みクリーク問題

::content::

- 高次元統計学(high-dimensional statistics)で考えられている問題
  - この文脈では計算困難性の仮定のスタート地点（NP困難性におけるSATの立ち位置）

- 論文：Planted Clique Conjectures are Equivalent

  - 平原 秀一さん（次の講演者）との共著
  - 2024年の**STOC** (Symposium on Theory of Computing) に採択
  - **理論計算機科学全体で最もレベルが高い**国際会議
  - 東京**工業**大学の**最後**のSTOC論文！

<div class="topic-box" v-click>

この講演は、技術的な話よりも、背景の紹介を優先。

</div>



---
layout: top-title
color: amber-light
---

::title::

# 埋め込みクリーク問題とは？

::content::

<div class="grid grid-cols-2 gap-6 items-start">

<div>

- $\ER(n)$ : Erdős--Rényiグラフ
  - $n$個の頂点を用意する
  - 各頂点ペアに対し、独立に確率$0.5$で辺を張る

<v-click>

- $\PC(n,k)$ : 埋め込みクリークグラフ
  - $\ER(n)$ を用意する
  - 相異なる$k$頂点$c_1,\dots,c_k$を一様ランダムに選ぶ
  - 全ての$c_i,c_j$間に辺を張る
  - この$k$頂点を**隠されたクリーク**という

</v-click>

</div>
<div>

<ErdosRenyiGraph
  :n="100"
  :p="0.06"
  :size="420"
  caption="見やすさのため辺確率を 0.06 としている。頂点数は100"
/>

</div>

</div>

---
layout: top-title
color: amber-light
---

::title::

# 埋め込みクリーク問題とは？

::content::

<div class="flex justify-center">
  <ErdosRenyiGraphSlider
    :n="100"
    :p="0.06"
    :size="420"
  />
</div>

---
layout: top-title
color: amber-light
---

::title::

# 埋め込みクリーク問題（探索版）

::content::

<div class="question">

入力として$\PC(n,k)$が与えられたとき、隠されたクリークを見つけよ。

</div>

- $\ER(n)$の最大クリークは高確率でおよそ **$2\log_2 n$**
  - $k \ll 2\log_2 n$ $\Rightarrow$ **統計的に**解けない（他のクリークと区別できない）
  - $k \gg 2\log_2 n$ $\Rightarrow$ **統計的に**解ける（$n^k$通り全てを列挙して最尤推定）

<v-click>

- **焦点**: $k\gg 2\log_2 n$ の範囲で**多項式時間で**クリークを見つけられるか？

<div class="flex justify-center">
  <img src="./images/baby_button_denchi.png" alt="赤ちゃんとボタンと電池" class="h-28 object-contain" />
</div>

</v-click>

---
layout: top-title
color: amber-light
---

::title::

# 埋め込みクリーク問題（探索版）

::content::

<div class="definition">

アルゴリズム$A$が **成功確率$\theta$で解く** とは、以下を満たすことをいう：

$$
\Pr_{G\sim \PC(n,k)}[A(G)=\text{隠されたクリーク}] \ge \theta.
$$

$\theta=2/3$のとき、単に「解く」という。

</div>

<v-clicks>

- $k=\Omega(\sqrt{n\log n})$なら[次数を見れば](https://nobutakashimizu.github.io/nobunote/docs/planted_clique/Kucera%E3%81%AE%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0/)解ける 
<a class="cite-reference" href="https://www.sciencedirect.com/science/article/pii/0166218X9400103K?via%3Dihub">\[Kučera (1995)]</a>

- $k=\Omega(\sqrt{n})$なら[固有ベクトルに基づく手法](https://nobutakashimizu.github.io/nobunote/docs/planted_clique/AKS%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0/)で解ける <a class="cite-reference" href="https://onlinelibrary.wiley.com/doi/10.1002/(SICI)1098-2418(199810/12)13:3/4%3C457::AID-RSA14%3E3.0.CO;2-W">Alon, Kurivelevich, Sudakov (1998)</a>
  - 高速化 <a class="cite-reference" href="https://dmtcs.episciences.org/2802">\[Feige, Ron (2010)]</a> や成功確率の改善 <a class="cite-reference" href="https://www.cambridge.org/core/journals/combinatorics-probability-and-computing/article/abs/finding-hidden-cliques-in-linear-time-with-high-probability/88A05D6B07878D54E62433DA72190623">\[Dekel, Gurel-Gurevich, Peres (2014)]</a>


<div class="topic-box">

現在では、$k=n^{1/2-\varepsilon}$では多項式時間で解けないと予想されている（**埋め込みクリーク予想**）

</div>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::

# 埋め込みクリーク問題（判定版）

::content::

<div class="question">

入力として$n$頂点のグラフ$G$が与えられる。このグラフは
- $\ER(n)$
- $\PC(n,k)$

のいずれかからサンプリングされたグラフである。どちらの分布かを当てよ。

</div>

- 判定するアルゴリズムは$G$を受け取って$0$または$1$を出力

<div class="definition" v-click>

アルゴリズム$A$は**アドバンテージ$\theta$で判定する**とは、

$$
\left| \Pr_{G\sim \PC(n,k)}[A(G)=1] - \Pr_{G\sim \ER(n)}[A(G)=1] \right| \ge \theta.
$$


</div>


---
layout: top-title
color: amber-light
---

::title::

# 統計限界と計算限界のギャップ

::content::

<div class="k-numberline-wrap">
  <div class="k-axis-labels k-axis-top">
    <span class="k-tick-label"><InlineMath formula="k \le 2\log_2 n" /></span>
    <span class="k-tick-label"><InlineMath formula="2\log_2 n \ll k \ll \sqrt{n}" /></span>
    <span class="k-tick-label"><InlineMath formula="k \ge \sqrt{n}" /></span>
  </div>
  <div class="k-numberline">
    <div class="k-seg k-zone-stat"></div>
    <div class="k-seg k-zone-comp"></div>
    <div class="k-seg k-zone-poly"></div>
  </div>
  <div class="k-axis-ticks">
    <span class="k-tick"></span>
    <span class="k-tick"></span>
    <span class="k-tick"></span>
    <span class="k-tick"></span>
  </div>
  <div class="k-axis-labels k-axis-bottom">
    <span class="k-desc">統計限界</span>
    <span class="k-desc">計算限界（予想）</span>
    <span class="k-desc">多項式時間で解ける</span>
  </div>
</div>

- $2\log_2 n \ll k \ll \sqrt{n}$は最尤推定で解けるが効率的に解けなさそう
  - **統計限界と計算限界のギャップ (computational-statistical gap)** という

<v-click>

- 様々なタイプのアルゴリズムに対して、$k=n^{1/2-\varepsilon}$は不可能である
  - Metropolis process <a class="cite-reference" href="https://onlinelibrary.wiley.com/doi/abs/10.1002/rsa.3240030402">\[Jerrum (1992)]</a><a class="cite-reference" href="https://onlinelibrary.wiley.com/doi/abs/10.1002/rsa.21274">\[Chen, Mossel, Zadik (2023)]</a>
  - Lovász-Schrijver semidefinite hierarchy <a class="cite-reference" href="https://epubs.siam.org/doi/10.1137/S009753970240118X">\[Feige, Krauthgamer (2023)]</a>
  - statistical query model <a class="cite-reference" href="https://dl.acm.org/doi/10.1145/3046674">\[Feldman, Grigorescu, Reyzin, Vempala, Xiao (2017)]</a>
  - low-degree polynomials <a class="cite-reference" href="https://epubs.siam.org/doi/10.1137/17M1138236">\[Barak, Hopkins, Kelner, Kothari, Moitra, Potechin (2019)]</a>

</v-click>

---
layout: top-title
color: amber-light
---

::title::

# なぜこの問題は重要なのか？

::content::

<div class="topic-box">

埋め込みクリーク問題の計算困難性 $\Rightarrow$ **様々な問題**の計算困難性

</div>

<v-clicks>

- 圧縮センシング <a class="cite-reference" href="https://ieeexplore.ieee.org/document/6837515">\[Koiran, Zouzias (2014)]</a>
- 最密部分グラフ問題 <a class="cite-reference" href="https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ITCS.2021.10">\[Manurangsi, Rubinstein, Schramm (2021)]</a>
- 主成分分析 <a class="cite-reference" href="https://proceedings.mlr.press/v30/Berthet13.html">\[Berthet, Rigollet (2013)]</a><a class="cite-reference" href="https://proceedings.mlr.press/v75/brennan18a.html">\[Brennan, Bresler, Huleihel (2018)]</a>
- ナッシュ均衡 <a class="cite-reference" href="https://epubs.siam.org/doi/10.1137/090766991">\[Hazan, Krauthgamer (2011)]</a><a class="cite-reference" href="https://theoryofcomputing.org/articles/v009a003/">\[Austin, Braverman, Chlamtac (2013)]</a>
- コミュニティ検出 <a class="cite-reference" href="https://proceedings.mlr.press/v40/Hajek15.html">\[Hajek, Wu, Xu (2015)]</a>
- 暗号学的プリミティブ <a class="cite-reference" href="https://link.springer.com/article/10.1023/A:1008374125234">\[Juel, Peinado (2000)]</a><a class="cite-reference" href="https://dl.acm.org/doi/10.1145/1806689.1806715">\[Applebaum, Barak, Wigderson (2010)]</a>
- 性質検査 <a class="cite-reference" href="https://dl.acm.org/doi/10.1145/1250790.1250863">\[Alon, Andoni, Kaufman, Matulef, Rubinfeld, Xie (2007)]</a>

</v-clicks>

---
layout: top-title
color: amber-light
---

::title::

# ランダムな入力上での計算量（平均時計算量）

::content::

<div class="timeline-table-wrap">

<table>
<thead>
<tr>
  <th>時期</th>
  <th>出来事</th>
</tr>
</thead>
<tbody>
<tr v-click>
  <td>50s–60s</td>
  <td>具体的なアルゴリズムのランダムな入力に対する挙動の解析<br>クイックソート<a class="cite-reference" href="https://dl.acm.org/doi/10.1145/366622.366644">[Hoare (1961)]</a>, 二分探索木<a class="cite-reference" href="https://dl.acm.org/doi/10.1145/321105.321108">[Hibbard (1962)]</a></td>
</tr>
<tr v-click>
  <td>1971–73</td>
  <td><InlineMath formula="\mathsf{NP}" />完全性の理論（<strong>最悪時計算量</strong>）<br><a class="cite-reference" href="https://dl.acm.org/doi/10.1145/800157.805047">[Cook (1971)]</a> <a class="cite-reference" href="https://www.mathnet.ru/php/archive.phtml?wshow=paper&jrnid=ppi&paperid=914&option_lang=eng">[Levin (1973)]</a> <a class="cite-reference" href="https://link.springer.com/chapter/10.1007/978-1-4684-2001-2_9">[Karp (1972)]</a></td>
</tr>
<tr v-click>
  <td>1976</td>
  <td>ランダムな入力上での計算量の議論の本格化<br>・ランダムな入力上でTSPや<strong>クリークの近似</strong>が解ける <a class="cite-reference" href="https://www2.eecs.berkeley.edu/Pubs/TechRpts/1976/28848.html">[Karp (1976)]</a><br>・離散対数の困難性に基づく<strong>公開鍵暗号方式</strong> <a class="cite-reference" href="https://ieeexplore.ieee.org/document/1055638">[Diffie, Hellman (1976)]</a></td>
</tr>
<tr v-click>
  <td>1983</td>
  <td>平均時困難性の開拓<br>・暗号学的擬似乱数生成器への応用 <a class="cite-reference" href="https://epubs.siam.org/doi/10.1137/0213053">[Blum, Micali (1984)]</a><br>・<strong>平均時計算複量クラス</strong> <InlineMath formula="\mathsf{avgP}" />, <InlineMath formula="\mathsf{distNP}" /> <a class="cite-reference" href="https://epubs.siam.org/doi/10.1137/0215020">[Levin (1986)]</a></td>
</tr>
<tr v-click>
  <td>1989</td>
  <td>脱乱択化への応用 <a class="cite-reference" href="https://www.sciencedirect.com/science/article/pii/S0022000005800431?via%3Dihub">[Nisan, Wigderson (1994)]</a></td>
</tr>
</tbody>
</table>

</div>

---
layout: top-title-two-cols
color: amber-light
---

::title::

# 平均時計算量の現在

::left::


### アルゴリズムの平均的な性能

<div class="topic-box">

**特定の問題**が成功確率$1-o(1)$で解けるか?

</div>


- 高次元統計学
  - ノイズが乗った高次元データから構造を抽出
  - 埋め込みクリークに帰着 <a class="cite-reference" href="https://proceedings.mlr.press/v75/brennan18a.html">\[Brennan, Bresler, Huleihel (2018)]</a>
- 制限されたクラスのアルゴリズムの限界
  - low-degree polynomial <a class="cite-reference" href="https://projecteuclid.org/journals/annals-of-statistics/volume-50/issue-3/Computational-barriers-to-estimation-from-low-degree-polynomials/10.1214/22-AOS2179.full">\[Schramm, Wein (2022)]</a>
- 様々な問題を **定性的（解けるかどうか）** に評価

::right::

<v-click>

### 平均時困難性の応用

<div class="topic-box">

成功確率$o(1)$の問題を構成したい (**困難性増幅**)

</div>

  - 最悪時困難性よりも強い要請
    - $\forall$効率的なアルゴリズム, $\exists$ 間違える入力
    - $\forall$効率的なアルゴリズムはほぼ全ての入力で間違える
  - 暗号学的プリミティブや脱乱択化への応用
    - 耐量子暗号
    - 公開鍵暗号方式
  - 成功確率やアドバンテージを**定量的に**評価

</v-click>

---
layout: top-title
color: amber-light
---

::title::

# Planted Clique Conjectures are Equivalent

::content::

<div class="question">

  埋め込みクリーク問題に対する多項式時間アルゴリズムの**成功確率**や**アドバンテージ**の限界はどこ？

</div>


- 文献によって、埋め込みクリーク問題の設定が異なる
  - $\forall$多項式時間アルゴリズムは**探索**問題の成功確率が **$1/3$** 以下
  - $\forall$多項式時間アルゴリズムは**判定**問題のアドバンテージが **$o(1)$**

<div class="topic-box" v-click>

我々の動機：これを統一化できないか？

</div>



---
layout: top-title
color: amber-light
---

::title::

# 主結果1



::content::

  <div class="theorem">
  
  成功確率 **$1/n^c$** で $\PC(n,n^{1/2-\alpha})$ 上で探索が多項式時間で解ける

  $\Downarrow$
  
  成功確率 **$1-\exp(-n^{c'})$** で $\PC(n, n^{1/2-\Theta(\alpha)})$ 上で探索が多項式時間で解ける
  
  </div>

- $k$の値が少しだけ変えるだけで、成功確率を劇的に増幅できる

<div class="corollary" v-click>

成功確率は **$1/\mathrm{poly}(n)$** から **$1-\exp(-\mathrm{poly}(n))$** の間のどの値に設定しても**探索問題**に対する「埋め込みクリーク予想」は同値

</div>

---
layout: top-title
color: amber-light
---

::title::

# 主結果2

::content::

<div class="theorem">

アドバンテージ $\frac{k^2}{n}\cdot$ **$n^\varepsilon$** で$\PC(n,k)$と$\ER(n)$を多項式時間で判定できる

$\Downarrow$

成功確率 **$1- \exp(-n^c)$** で $\PC(n,n^{1/2-\alpha})$ 上で探索が多項式時間で解ける

</div>

<v-clicks>

- $\PC(n,k)$ と $\ER(n)$ は、**辺を数えれば**アドバンテージ **$\Omega(\frac{k^2}{n})$** で判定できる
  - $\ER(n)$の辺の本数 $= \frac{1}{2}\binom{n}{2} \underbrace{\pm O(n)}_{\text{標準偏差}}$
  - $\PC(n,k)$の辺の本数 $= \frac{1}{2}\binom{n}{2} + \Theta(k^2) \pm O(n)$

- 我々の結果：辺数え上げより**ちょっと良いアドバンテージ**が出せれば、クリークを見つけられる
  
- (厳密には$\PC(n,k)$とはちょっと異なるグラフを考えている)

</v-clicks>



---
layout: top-title
color: amber-light
---

::title::

# 証明のコア

::content::

次の帰着アルゴリズム $\mathsf{Shrink}$ を考える：

<div class="topic-box">

1. 成功確率$1/\mathrm{poly}(n)$ のアルゴリズムを$A$とし、入力を $G\sim\PC(n,n^{1/2-\alpha})$ とする。

<v-clicks>

2. ランダムに $n'=n^{1-\varepsilon}$ 個の頂点を選び、その集合からなる <strong> 誘導部分グラフ $H$ </strong> をとる
3. $A(H)$ が$H$内のクリークを見つけたら、そこから$G$のクリークを復元する
4. ステップ2-3を $\mathrm{poly}(n)$ 回繰り返す

</v-clicks>

</div>

<div class="flex justify-center mt-4">
  <img src="./images/shrinking.svg" alt="shrinking" class="w-[50%] max-w-5xl h-auto" />
</div>

---
layout: top-title
color: amber-light
---

::title::

# 証明のコア

::content::

<div class="lemma">

元のアルゴリズムの成功確率が$1/\mathrm{poly}(n)$なら、適切な$\varepsilon$を選ぶことで成功確率が$2/3$にできる。

</div>

- 直感：1回の成功確率が$1/n^c$なら、$n^c$回実行すれば、期待値的には$1$回は成功する
- それぞれの試行は独立ではないが、この直感はある程度は正当化できる

<div class="topic-box" v-click>

- 残念ながら、これだけだと（現状の解析では）成功確率をもっと上げることはできない
- そこで、別の帰着が必要

</div>

---
layout: top-title
color: amber-light
---

::title::

# 証明のコア

::content::


次の帰着アルゴリズム $\mathsf{Embed}$ を考える：

<div class="topic-box">

1. 成功確率$2/3$ のアルゴリズムを$A$とし、入力を $G\sim\PC(N,\ell)$ とする。

<v-clicks>

2. $\ER(O(N\log N))$を用意し、このグラフにランダムに$G$を埋め込んで得られるグラフを$G'$とする
3. $A(G')$ がクリークを見つけたら、そこから$G$のクリークを復元する
4. ステップ2-3を $\mathrm{poly}(n)$ 回繰り返す

</v-clicks>

</div>

<div class="flex justify-center mt-4">
  <img src="./images/embedding.svg" alt="shrinking" class="w-[60%] max-w-5xl h-auto" />
</div>

---
layout: top-title
color: amber-light
---

::title::

# 証明のコア

::content::

<div class="lemma">

元のアルゴリズムの成功確率が$2/3$ならば、$\mathsf{Embed}$の成功確率は$1-\exp(-\mathrm{poly})$になる。

</div>

- 直感：1回の成功確率が$2/3$なら、$n^c$回実行すれば、全て失敗する確率は $(1/3)^{n^c}$ になる
  - それぞれの試行は独立ではないが、この直感もまた、ある程度は正当化できる

---
layout: top-title
color: amber-light
---

::title::

# 証明のコア

::content::

最終的な帰着は $\mathsf{Shrink}$ と $\mathsf{Embed}$ を組み合わせる

<div class="flex justify-center mt-4">
  <img src="./images/reduction.svg" alt="shrinking" class="w-[92%] max-w-5xl h-auto" />
</div>

<v-click>

<div class="topic-box">

左のグラフの成功確率 **$\le 1-1/\mathrm{poly}(n)$** $\Rightarrow$ 右のグラフの成功確率 **$\le \exp(-\mathrm{poly}(n))$**

つまり、困難性が増幅している

</div>

</v-click>

---
layout: top-title
color: amber-light
---

::title::

# 証明のコア

::content::

- 探索から判定への帰着：<a class="cite-reference" href="https://dl.acm.org/doi/10.1145/1250790.1250863">\[Alon, Andoni, Kaufman, Matulef, Rubinfeld, Xie (2007)]</a> の結果を使う

<div class="lemma">

アドバンテージ **$1-\frac{1}{10n}$** で$\PC(n,k)$と$\ER(n)$を多項式時間で判定できる

$\Downarrow$

成功確率 **$0.9$** で $\PC(n,k)$ 上で探索が多項式時間で解ける

</div>

- アドバンテージを**増幅**させればよい
  - アドバンテージ $\frac{k^2}{n}\cdot n^\varepsilon$で解ける $\Rightarrow$ アドバンテージ $1-\frac{1}{10n}$ で解ける
- $\mathsf{Shrink}$ によってこの増幅が可能

---
layout: top-title
color: amber-light
---

::title::

# まとめ

::content::

- **埋め込みクリーク問題**：$2\log_2 n \ll k \ll \sqrt{n}$ に**統計限界と計算限界のギャップ**がある
- 文献ごとに困難性仮定（成功確率・アドバンテージの限界）が異なる → **統一化**を目指した
- **証明のコア**
  - $\mathsf{Shrink}$ ... 頂点数を減らして成功確率を増幅（クリークサイズがちょっと変わる）
  - $\mathsf{Embed}$ ... グラフを埋め込んで成功確率を増幅

<div class="topic-box">

**メッセージ**

- 平均時計算量は、暗号などへの応用といった文脈でも研究されている
- しかし、この文脈では特定の問題に焦点は当てず、問題を変形して**困難性を増幅**している
- 困難性増幅の手法が**埋め込みクリーク問題**に役立った

</div>