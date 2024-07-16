---
layout: single
title: "Transformer"
categories : LLM
tag: [Transformer, google]
---

Transformer 분석

### Transformer

2017년에 구글이 발표한 "Attention is all you need"라는 논문에서 제안된 모델

기존의 seq2seq 구조를 따르면서도 이름처럼 어텐션(Attention) 메커니즘만을 사용하여 구현

RNN을 사용하지 않고도 인코더-디코더 구조를 설계한 이 모델은 번역 성능에서 RNN보다 우수한 결과를 보임

기존의 seq2seq 모델은 인코더-디코더 구조로 이루어져 있었으며, 인코더는 입력 시퀀스를 하나의 벡터 표현으로 압축하고, 디코더는 이 벡터 표현을 기반으로 출력 시퀀스를 생성한다.

### 기존 seq2seq 모델

- RNN : Recurrent Neural Network
- LSTM : Long Short-Term Memory
- GRU : Gated Recurrent Unit

    문제점
    - Gradient Exploding & Vanishing
    - Long-term Dependency

- Attention RNN
  - 중요한 부분이 어디인지 계산하여, 중요한 부분에 집중
  - 사라져 가는 정보에 Attention 연산을 수행하는 문제점

### Attention

기본 아이디어는 디코더가 출력 단어를 예측하는 각 시점(time step)마다 인코더에서의 전체 입력 문장을 다시 한 번 참고

모든 입력 단어를 동일한 비율로 고려하는 것이 아니라, 해당 시점에서 예측해야 할 단어와 관련 있는 입력 단어 부분에 더 집중(attention)

어텐션 함수

    Attention(Q, K, V) = Attention Value

    '쿼리(Query)'에 대한 모든 '키(Key)'와의 유사도를 개별적으로 계산하고, 이러한 유사도를 해당 키와 연결된 각 '값(Value)'에 반영

    유사도가 반영된 '값(Value)'을 모두 합산하여 반환

    Q: 모든 시점의 디코더 셀에서의 은닉 상태들
    K: 모든 시점의 인코더 셀의 은닉 상태들
    V: 모든 시점의 인코더 셀의 은닉 상태들

셀프 어텐션은 Q, K, V가 전부 동일한 것을 의미

Transformer에서는 Encoder Self-attention, Masked Decoder Self-attention, Encoder-Decoder Attention의 세 가지의 어텐션이 사용

두 self-attention은 자신의 동일한 Q, K, V를 가지지만 세번째 Encoder-Decoder Attention에서는 Query가 디코더의 벡터이고 Key와 Value는 인코더의 벡터이다.

<img src="https://wikidocs.net/images/page/31379/transformer_attention_overview.PNG">

Transformer에 대한 구글 AI 블로그 포스트에서 발취

The animal didn't cross the street because it was too tired.에서 그것(it)에 해당하는 것은 길(street)일까? 동물(animal)일까?

우리는 피곤한 주체가 동물이라는 것을 아주 쉽게 알 수 있지만 기계는 그렇지 않다. 하지만 셀프 어텐션은 입력 문장 내의 단어들끼리 유사도를 구하므로서 그것(it)이 동물(animal)과 연관되었을 확률이 높다는 것을 찾아낸다.

<img src="https://wikidocs.net/images/page/31379/transformer10.png">

### Q, K, V 벡터 얻기
 
셀프 어텐션은 입력 문장의 단어 벡터를 기반으로 작동한다. 그러나 실제로는 셀프 어텐션을 수행하기 위해 인코더의 초기 입력인 단어 벡터들로부터 Q벡터, K벡터, V벡터를 얻는 과정이 필요하다.

이러한 Q벡터, K벡터, V벡터는 초기 입력의 차원과는 달리 더 낮은 차원을 가지며, Transformer에서는 초기 입력으로부터 512차원의 각 단어 벡터를 64차원의 Q벡터, K벡터, V벡터로 변환했다.

위의 64라는 값은 Transformer의 다른 Hyperparameter인 num_heads로 결정된다. Transformer에서는 num_heads 값을 8로 설정하였고, 이로인해 512/8=64차원의 Q벡터, K벡터, V벡터로 변환해야 한다.

<img src="https://wikidocs.net/images/page/31379/transformer11.PNG">

student라는 단어의 512차원 임베딩 벡터에 512 X 64 차원의 크기를 가진 Q 가중치, K 가중치, V 가중치 행렬을 각각 곱하여 Q, K, V 벡터를 얻어낸다.

Q, K, V 벡터를 얻었다면 지금부터는 기존의 어텐션 메커니즘과 동일하다.

각 Q벡터는 모든 K벡터에 대해서 어텐션 스코어를 구하고, 어텐션 분포를 구한 뒤에 이를 사용하여 모든 V벡터를 가중합하여 어텐션 값 또는 컨텍스트 벡터를 구한다. 그리고 이를 모든 Q벡터에 대해서 반복한다.

### Transformer 논문에서 사용된 주요 Hyperparameter

<mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" display="true" tabindex="0" ctxtmenu_counter="0" style="font-size: 113.1%; position: relative;"><mjx-math display="true" class="MJX-TEX" aria-hidden="true" style="margin-left: 0px; margin-right: 0px;"><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45A TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45C TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D452 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D459 TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msub><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c3D"></mjx-c></mjx-mo><mjx-mn class="mjx-n" space="4"><mjx-c class="mjx-c35"></mjx-c><mjx-c class="mjx-c31"></mjx-c><mjx-c class="mjx-c32"></mjx-c></mjx-mn></mjx-math><mjx-assistive-mml unselectable="on" display="block"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><msub><mi>d</mi><mrow data-mjx-texclass="ORD"><mi>m</mi><mi>o</mi><mi>d</mi><mi>e</mi><mi>l</mi></mrow></msub><mo>=</mo><mn>512</mn></math></mjx-assistive-mml></mjx-container>

Transformer의 인코더와 디코더에서의 정해진 입력과 출력의 크기를 의미함. 임베딩 벡터의 차원 또한 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="1" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45A TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45C TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D452 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D459 TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msub></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><msub><mi>d</mi><mrow data-mjx-texclass="ORD"><mi>m</mi><mi>o</mi><mi>d</mi><mi>e</mi><mi>l</mi></mrow></msub></math></mjx-assistive-mml></mjx-container>이며, 각 인코더와 디코더가 다음 층의 인코더와 디코더로 값을 보낼 때에도 이 차원을 유지한다. 논문에서는 512.

<mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" display="true" tabindex="0" ctxtmenu_counter="2" style="font-size: 113.1%; position: relative;"><mjx-math display="true" class="MJX-TEX" aria-hidden="true" style="margin-left: 0px; margin-right: 0px;"><mjx-mtext class="mjx-n"><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c75"></mjx-c><mjx-c class="mjx-c6D"></mjx-c><mjx-c class="mjx-c5F"></mjx-c><mjx-c class="mjx-c6C"></mjx-c><mjx-c class="mjx-c61"></mjx-c><mjx-c class="mjx-c79"></mjx-c><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c72"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c3D"></mjx-c></mjx-mo><mjx-mn class="mjx-n" space="4"><mjx-c class="mjx-c36"></mjx-c></mjx-mn></mjx-math><mjx-assistive-mml unselectable="on" display="block"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><mtext>num_layers</mtext><mo>=</mo><mn>6</mn></math></mjx-assistive-mml></mjx-container>

Transformer에서 하나의 인코더와 디코더를 층으로 생각하였을 때, Transformer 모델에서 인코더와 디코더가 총 몇 층으로 구성되었는지를 의미. 논문에서는 인코더와 디코더를 각각 총 6개 쌓았다.  

<mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" display="true" tabindex="0" ctxtmenu_counter="3" style="font-size: 113.1%; position: relative;"><mjx-math display="true" class="MJX-TEX" aria-hidden="true" style="margin-left: 0px; margin-right: 0px;"><mjx-mtext class="mjx-n"><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c75"></mjx-c><mjx-c class="mjx-c6D"></mjx-c><mjx-c class="mjx-c5F"></mjx-c><mjx-c class="mjx-c68"></mjx-c><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c61"></mjx-c><mjx-c class="mjx-c64"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c3D"></mjx-c></mjx-mo><mjx-mn class="mjx-n" space="4"><mjx-c class="mjx-c38"></mjx-c></mjx-mn></mjx-math><mjx-assistive-mml unselectable="on" display="block"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><mtext>num_heads</mtext><mo>=</mo><mn>8</mn></math></mjx-assistive-mml></mjx-container>

Transformer에서는 어텐션을 사용할 때, 한 번 하는 것 보다 여러 개로 분할해서 병렬로 어텐션을 수행하고 결과값을 다시 하나로 합치는 방식을 택했다. 이때 이 병렬의 개수를 의미함.  

<mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" display="true" tabindex="0" ctxtmenu_counter="4" style="font-size: 113.1%; position: relative;"><mjx-math display="true" class="MJX-TEX" aria-hidden="true" style="margin-left: 0px; margin-right: 0px;"><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D453 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D453 TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msub><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c3D"></mjx-c></mjx-mo><mjx-mn class="mjx-n" space="4"><mjx-c class="mjx-c32"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c34"></mjx-c><mjx-c class="mjx-c38"></mjx-c></mjx-mn></mjx-math><mjx-assistive-mml unselectable="on" display="block"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><msub><mi>d</mi><mrow data-mjx-texclass="ORD"><mi>f</mi><mi>f</mi></mrow></msub><mo>=</mo><mn>2048</mn></math></mjx-assistive-mml></mjx-container>

Transformer 내부에는 피드 포워드 신경망이 존재하며 해당 신경망의 은닉층의 크기를 의미. 피드 포워드 신경망의 입력층과 출력층의 크기는 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="5" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45A TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45C TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D452 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D459 TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msub></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><msub><mi>d</mi><mrow data-mjx-texclass="ORD"><mi>m</mi><mi>o</mi><mi>d</mi><mi>e</mi><mi>l</mi></mrow></msub></math></mjx-assistive-mml></mjx-container>임.

### Scaled dot-product Attention

Transformer에서는 어텐션 챕터에 사용했던 내적만을 사용하는 어텐션 함수 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="29" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D460 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D450 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45C TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45F TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D452 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c28"></mjx-c></mjx-mo><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45E TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c2C"></mjx-c></mjx-mo><mjx-mi class="mjx-i" space="2"><mjx-c class="mjx-c1D458 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c29"></mjx-c></mjx-mo><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c3D"></mjx-c></mjx-mo><mjx-mi class="mjx-i" space="4"><mjx-c class="mjx-c1D45E TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n" space="3"><mjx-c class="mjx-c22C5"></mjx-c></mjx-mo><mjx-mi class="mjx-i" space="3"><mjx-c class="mjx-c1D458 TEX-I"></mjx-c></mjx-mi></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>s</mi><mi>c</mi><mi>o</mi><mi>r</mi><mi>e</mi><mo stretchy="false">(</mo><mi>q</mi><mo>,</mo><mi>k</mi><mo stretchy="false">)</mo><mo>=</mo><mi>q</mi><mo>⋅</mo><mi>k</mi></math></mjx-assistive-mml></mjx-container>가 아니라 여기에 특정값으로 나눠준 어텐션 함수인 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="30" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D460 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D450 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45C TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45F TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D452 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c28"></mjx-c></mjx-mo><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45E TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c2C"></mjx-c></mjx-mo><mjx-mi class="mjx-i" space="2"><mjx-c class="mjx-c1D458 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c29"></mjx-c></mjx-mo><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c3D"></mjx-c></mjx-mo><mjx-mi class="mjx-i" space="4"><mjx-c class="mjx-c1D45E TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n" space="3"><mjx-c class="mjx-c22C5"></mjx-c></mjx-mo><mjx-mi class="mjx-i" space="3"><mjx-c class="mjx-c1D458 TEX-I"></mjx-c></mjx-mi><mjx-texatom texclass="ORD"><mjx-mo class="mjx-n"><mjx-c class="mjx-c2F"></mjx-c></mjx-mo></mjx-texatom><mjx-msqrt><mjx-sqrt><mjx-surd><mjx-mo class="mjx-n"><mjx-c class="mjx-c221A"></mjx-c></mjx-mo></mjx-surd><mjx-box style="padding-top: 0.281em;"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45B TEX-I"></mjx-c></mjx-mi></mjx-box></mjx-sqrt></mjx-msqrt></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>s</mi><mi>c</mi><mi>o</mi><mi>r</mi><mi>e</mi><mo stretchy="false">(</mo><mi>q</mi><mo>,</mo><mi>k</mi><mo stretchy="false">)</mo><mo>=</mo><mi>q</mi><mo>⋅</mo><mi>k</mi><mrow data-mjx-texclass="ORD"><mo>/</mo></mrow><msqrt><mi>n</mi></msqrt></math></mjx-assistive-mml></mjx-container>를 사용한다.

<img src="https://wikidocs.net/images/page/31379/transformer13.PNG">

어텐션 스코어에 소프트맥스 함수를 사용하여 어텐션 분포(Attention Distribution)을 구하고, 각 V벡터와 가중합하여 어텐션 값(Attention Value)을 구한다.

이를 단어 I에 대한 어텐션 값 또는 단어 I에 대한 컨텍스트 벡터(context vector)라고도 할 수 있다. am에 대한 Q벡터, a에 대 Q벡터, student에 대한 Q벡터에 대해서도 모두 동일한 과정을 반복하여 각각에 대한 어텐션 값을 구한다.

<img src="https://wikidocs.net/images/page/31379/transformer14_final.PNG">

### 행렬 연산으로 일괄 처리하기

각 단어 벡터마다 일일히 가중치 행렬을 곱하는 것이 아니라 문장 행렬에 가중치 행렬을 곱하여 Q행렬, K행렬, V행렬을 구한다.

<img src="https://wikidocs.net/images/page/31379/transformer12.PNG">

행렬 연산을 통해 어텐션 스코어는 Q행렬을 K행렬을 전치한 행렬과 곱해준다. 이렇게 되면 각각의 단어의 Q벡터와 K벡터의 내적이 각 행렬의 원소가 되는 행렬이 결과로 나온다.

<img src="https://wikidocs.net/images/page/31379/transformer15.PNG">

행렬의 값에 전체적으로 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="36" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-msqrt><mjx-sqrt><mjx-surd><mjx-mo class="mjx-n"><mjx-c class="mjx-c221A"></mjx-c></mjx-mo></mjx-surd><mjx-box style="padding-top: 0.082em;"><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D458 TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msub></mjx-box></mjx-sqrt></mjx-msqrt></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><msqrt><msub><mi>d</mi><mrow data-mjx-texclass="ORD"><mi>k</mi></mrow></msub></msqrt></math></mjx-assistive-mml></mjx-container>를 나누어주면 이는 각 행과 열이 어텐션 스코어 값을 가지는 행렬이 된다.

예를 들어 I 행과 student 열의 값은 I의 Q벡터와 student의 K벡터의 어텐션 스코어 값이 된다. 이걸 어텐션 스코어 행렬이라 하면. 어텐션 스코어 행렬을 구하였다면 남은 것은 어텐션 분포를 구하고, 이를 사용하여 모든 단어에 대한 어텐션 값을 구하는 일이다. 이는 간단하게 어텐션 스코어 행렬에 소프트맥스 함수를 사용하고, V행렬을 곱하는 것으로 해결된다.

<img src="https://wikidocs.net/images/page/31379/transformer16.PNG">

Transformer 논문에 기재된 수식과 정확하게 일치하는 식<br>
<mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" display="true" tabindex="0" ctxtmenu_counter="37" style="font-size: 113.1%; position: relative;"><mjx-math display="true" class="MJX-TEX" aria-hidden="true" style="margin-left: 0px; margin-right: 0px;"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D434 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D461 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D461 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D452 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45B TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D461 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D456 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45C TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45B TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c28"></mjx-c></mjx-mo><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D444 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c2C"></mjx-c></mjx-mo><mjx-mi class="mjx-i" space="2"><mjx-c class="mjx-c1D43E TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c2C"></mjx-c></mjx-mo><mjx-mi class="mjx-i" space="2"><mjx-c class="mjx-c1D449 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c29"></mjx-c></mjx-mo><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c3D"></mjx-c></mjx-mo><mjx-mi class="mjx-i" space="4"><mjx-c class="mjx-c1D460 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45C TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D453 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D461 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45A TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D44E TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D465 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c28"></mjx-c></mjx-mo><mjx-texatom texclass="ORD"><mjx-mfrac><mjx-frac type="d"><mjx-num><mjx-nstrut type="d"></mjx-nstrut><mjx-mrow><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D444 TEX-I"></mjx-c></mjx-mi><mjx-msup><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D43E TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: 0.363em; margin-left: 0.052em;"><mjx-mi class="mjx-i" size="s"><mjx-c class="mjx-c1D447 TEX-I"></mjx-c></mjx-mi></mjx-script></mjx-msup></mjx-mrow></mjx-num><mjx-dbox><mjx-dtable><mjx-line type="d"></mjx-line><mjx-row><mjx-den><mjx-dstrut type="d"></mjx-dstrut><mjx-texatom texclass="ORD"><mjx-msqrt><mjx-sqrt><mjx-surd><mjx-mo class="mjx-n"><mjx-c class="mjx-c221A"></mjx-c></mjx-mo></mjx-surd><mjx-box style="padding-top: 0.082em;"><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-mi class="mjx-i" size="s"><mjx-c class="mjx-c1D458 TEX-I"></mjx-c></mjx-mi></mjx-script></mjx-msub></mjx-box></mjx-sqrt></mjx-msqrt></mjx-texatom></mjx-den></mjx-row></mjx-dtable></mjx-dbox></mjx-frac></mjx-mfrac></mjx-texatom><mjx-mo class="mjx-n"><mjx-c class="mjx-c29"></mjx-c></mjx-mo><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D449 TEX-I"></mjx-c></mjx-mi></mjx-math><mjx-assistive-mml unselectable="on" display="block"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><mi>A</mi><mi>t</mi><mi>t</mi><mi>e</mi><mi>n</mi><mi>t</mi><mi>i</mi><mi>o</mi><mi>n</mi><mo stretchy="false">(</mo><mi>Q</mi><mo>,</mo><mi>K</mi><mo>,</mo><mi>V</mi><mo stretchy="false">)</mo><mo>=</mo><mi>s</mi><mi>o</mi><mi>f</mi><mi>t</mi><mi>m</mi><mi>a</mi><mi>x</mi><mo stretchy="false">(</mo><mrow data-mjx-texclass="ORD"><mfrac><mrow><mi>Q</mi><msup><mi>K</mi><mi>T</mi></msup></mrow><mrow data-mjx-texclass="ORD"><msqrt><msub><mi>d</mi><mi>k</mi></msub></msqrt></mrow></mfrac></mrow><mo stretchy="false">)</mo><mi>V</mi></math></mjx-assistive-mml></mjx-container>

### Multi-head Attention

Transformer 연구진은 한 번의 어텐션을 하는 것보다 여러번의 어텐션을 병렬로 사용하는 것이 더 효과적이라고 판단함. <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="59" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45A TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45C TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D452 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D459 TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msub></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><msub><mi>d</mi><mrow data-mjx-texclass="ORD"><mi>m</mi><mi>o</mi><mi>d</mi><mi>e</mi><mi>l</mi></mrow></msub></math></mjx-assistive-mml></mjx-container>의 차원을 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="60" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-mtext class="mjx-n"><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c75"></mjx-c><mjx-c class="mjx-c6D"></mjx-c><mjx-c class="mjx-c5F"></mjx-c><mjx-c class="mjx-c68"></mjx-c><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c61"></mjx-c><mjx-c class="mjx-c64"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><mtext>num_heads</mtext></math></mjx-assistive-mml></mjx-container>개로 나누어 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="61" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45A TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D45C TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D451 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D452 TEX-I"></mjx-c></mjx-mi><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D459 TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msub><mjx-mtext class="mjx-n"><mjx-c class="mjx-c2F"></mjx-c><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c75"></mjx-c><mjx-c class="mjx-c6D"></mjx-c><mjx-c class="mjx-c5F"></mjx-c><mjx-c class="mjx-c68"></mjx-c><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c61"></mjx-c><mjx-c class="mjx-c64"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><msub><mi>d</mi><mrow data-mjx-texclass="ORD"><mi>m</mi><mi>o</mi><mi>d</mi><mi>e</mi><mi>l</mi></mrow></msub><mtext>/num_heads</mtext></math></mjx-assistive-mml></mjx-container>의 차원을 가지는 Q, K, V에 대해서 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="62" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-mtext class="mjx-n"><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c75"></mjx-c><mjx-c class="mjx-c6D"></mjx-c><mjx-c class="mjx-c5F"></mjx-c><mjx-c class="mjx-c68"></mjx-c><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c61"></mjx-c><mjx-c class="mjx-c64"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><mtext>num_heads</mtext></math></mjx-assistive-mml></mjx-container>개의 병렬 어텐션을 수행합니다. 논문에서는 하이퍼파라미터인 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="63" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-mtext class="mjx-n"><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c75"></mjx-c><mjx-c class="mjx-c6D"></mjx-c><mjx-c class="mjx-c5F"></mjx-c><mjx-c class="mjx-c68"></mjx-c><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c61"></mjx-c><mjx-c class="mjx-c64"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><mtext>num_heads</mtext></math></mjx-assistive-mml></mjx-container>의 값을 8로 지정하였고, 8개의 병렬 어텐션이 이루어지게 된다.

어텐션이 8개로 병렬로 이루어지게 되는데, 이때 각각의 어텐션 값 행렬을 어텐션 헤드라고 부른다. 이때 가중치 행렬 <mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" tabindex="0" ctxtmenu_counter="64" style="font-size: 113.1%; position: relative;"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-msup><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D44A TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: 0.363em; margin-left: 0.055em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D444 TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msup><mjx-mo class="mjx-n"><mjx-c class="mjx-c2C"></mjx-c></mjx-mo><mjx-msup space="2"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D44A TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: 0.363em; margin-left: 0.055em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D43E TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msup><mjx-mo class="mjx-n"><mjx-c class="mjx-c2C"></mjx-c></mjx-mo><mjx-msup space="2"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D44A TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: 0.363em; margin-left: 0.055em;"><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D449 TEX-I"></mjx-c></mjx-mi></mjx-texatom></mjx-script></mjx-msup></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><msup><mi>W</mi><mrow data-mjx-texclass="ORD"><mi>Q</mi></mrow></msup><mo>,</mo><msup><mi>W</mi><mrow data-mjx-texclass="ORD"><mi>K</mi></mrow></msup><mo>,</mo><msup><mi>W</mi><mrow data-mjx-texclass="ORD"><mi>V</mi></mrow></msup></math></mjx-assistive-mml></mjx-container>의 값은 8개의 어텐션 헤드마다 전부 다름.

병렬 어텐션으로 얻을 수 있는 효과는? 그리스로마신화에는 머리가 여러 개인 괴물 히드라나 케로베로스가 나온다. 이 괴물들의 특징은 머리가 여러 개이기 때문에 여러 시점에서 상대방을 볼 수 있다는 것이다. 이렇게 되면 시각에서 놓치는 게 별로 없을테니까 이런 괴물들에게 기습을 하는 것이 굉장히 힘이 들것이다. 멀티 헤드 어텐션도 똑같이 어텐션을 병렬로 수행하여 다른 시각으로 정보들을 수집하겠다는 것이다.

<img src="https://wikidocs.net/images/page/31379/transformer17.PNG">