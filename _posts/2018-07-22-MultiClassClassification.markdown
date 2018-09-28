---
layout: post
title:  "ViDi / Color Classification & Muli-Class Classification"
date:   2018-07-22 23:18:00
author: Alex Choi
categories: Deep-Learning
---

------
## 1. `Found = best_tag`? `tag![TAG_NAME]`?
Green Tool에서 분류된 결과를 Red Tool에서 Tag 별로 불러오는 과정에서 생긴 이슈였습니다.

Green Tool에서 찾은(Found) Tag의 수와 Display Filter를 통해 걸러낸 이미지(정확히 말하면 Views)의 개수가 일치하지 않는 경우가 있습니다.

아래 이미지와 같이 Table 상에서 Tag '7'은 Found가 30개이며, 이를 더블클릭하면 Display Filter에 자동으로 `best_tag = '7'` 입력됨과 동시에 이에 해당하는 Views의 개수를 알 수 있는데 34개의 Views를 표시하고 있는 것을 확인할 수 있습니다 (이와 같은 상황은 Feature Size를 상식 이하로 크게 잡고 Epochs 수를 매우 낮추어 제대로 분류하지 못하도록 하면 쉽게 만들 수 있습니다. 간혹 Feature Size를 너무 크게 잡으면 수치불안정으로 진행이 안 될 수 있으니 이보다는 크게 잡아야 합니다).

즉, 이미 아시는 분은 아시겠지만, `Found for Tag '7' ≠ best_tag = '7'`입니다.

<br/><br/>
<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/01.png">
<br/><br/>

그러면, 왜 `Found for Tag '7' ≠ best_tag = '7'`일까요? ViDi의 기능을 잘 아시는 분들은 '이것도 질문이라고 하냐?'고 하실 수 있겠지만, 아쉽게도 이 부분이 ViDi 공식 도큐먼트에는 `best_tag`에 대한 설명 전혀 없으므로 ViDi 처음 입문자에게는 다소 혼란을 줄 수 있을 것이라고 생각됩니다. 사실 고객이 도큐먼트에 설명이 없는 것을 우리가 어떻게 아느냐고 따지면 별달리 해 줄 수 있는 말이 없기도 합니다 (개인적으로는 ViDi 공식 도큐먼트 아직은 아쉬운 부분이 많습니다. 실제로 ViDi 공식 도큐).

ViDi 3.1이 기본경로에 설치되어 있는 분들은 아래 링크에서 Display Filter에 대한 내용을 확인할 수 있는데 best_tag에 대한 설명은 어떠한 언급도 없습니다.

[ViDi 3.1 Display Filter 도큐먼트 바로가기](http://13.125.95.237/vidi_documentation/Default.htm#ViDi_Topics/Concepts/images_display_filters_sort.htm%3FTocPath%3DCognex%2520ViDi%2520Suite%2520Concepts%7CImages%7C_____2)

심지어 `best_tag를` 검색창에 입력하여 검색해 보면,
<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/02.png">
<br/><br/>

아니 이럴수가????!!!!! ㅎ_ㅎ

그러면, `best_tag`는 어떤 Filter이며, 왜 Found의 개수와 다른지 설명드리도록 하겠습니다 (이미 아시는 분은 패쑤~~).

일단, 결론부터 말씀드리면 `best_tag`는 ViDi가 찾은 Tag 중 가장 점수가 높은 녀석들을 의미합니다. 즉, Classifier 특성 상 [Information Theory](https://en.wikipedia.org/wiki/Information_theory)의 [Cross Entropy](https://en.wikipedia.org/wiki/Cross_entropy) - 이것은 두 Class 사이의 정보 차이에 의한 Information Loss를 계산하는 지표가 됩니다. 즉, 두 Class(Ground Truth Label과 Predicted Label)가 일치하면 Loss가 0가 되며, 이 Loss를 최소화하는 방향으로 뉴럴 네트워크의 파라미터를 결정하게 됩니다 - 등의 개념을 도입해 확률적인 점수를 매길 것입니다.

그런데, Green Tool의 Processing 파라미터 중 Threshold 라는 것이 있습니다. 이것은 찾아낸 Tag의 스코어가 다른 Tag보다 높더라도 원하는 스코어 이상을 넘기지 못하면 버리는 역할을 합니다.

아래 예를 통해 살펴보도록 하겠습니다.
<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/03.png">
<br/><br/>

위의 경우, 해당 이미지에 대한 `best_tag`는 'Dog'이 됩니다. `Found`도 'Dog'이 되겠죠.

하지만 아래의 경우에는,
<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/04.png">
<br/><br/>

Tag 중 그 어느 것도 Threshold를 넘지 못하기 때문에 best_tag는 'Dog'이 되지만 Found에는 해당되지 않습니다. 바로 이런 경우가 `best_tag`와 `Found` 간의 차이를 일으키게 됩니다. 즉, `best_tag = 'TAG_NAME' ≥ Found for TAG_NAME`가 성립됩니다.

따라서, `Found`와 동일한 수의 Views를 검출하려면 score가 Threshold를 넘는 조건을 추가하면 됩니다 : `best_tag = 'TAG_NAME' and score > threshold`. 곧, 이 조건이 `Found`와 동일한 수를 뽑아내는 조건이 되겠습니다.

이제 아래 이미지와 같이 Tag '7'에 대하여 Found의 수와 Display Filter에 의해 표시되는 Views의 개수가 일치함을 볼 수 있습니다.
<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/05.png">
<br/><br/>

또 한가지 언급할 점은, 원래 Green Tool에서는 이미지에 대한 마스킹을 적용하려면 전체 이미지에 동일하게 적용하여 사용해야 하는데, 이미지마다 다른 마스킹을 적용하였고 마스킹도 일부 이미지에만 적용을 하여 사용한 경우가 있습니다 (추후 이런 식으로 접근하는 고객이 있다면 즉시 말려야 할 것입니다). 이런 경우에는 `best_tag = 'TAG_NAME' and score > threshold Display Filter` 조건을 적용한다고 하더라도 Found와 Views 개수가 일치하지 않았습니다.

------
## 2.  Green Tool로 단순 컬러 분류 가능?
가끔 고객이 저한테 묻는 질문 중 하나입니다. "Green Tool로 단순히 컬러만 다른 이미지 분류가 가능한가?"

ViDi가 CNN(회선신경망; Convolutional Neural Networks)을 기반으로 한다면 이 질문은 어리석을 정도로 심플한 것입니다. 왜냐하면, CNN에서는 컬러 채널별로 서로 다른 회선처리를 하여 Feature Map을 생성하기 때문입니다. 그렇기 때문에 당연히 이미지의 단순 컬러 분류가 가능합니다.

현재 ViDi의 Green Tool은 4개의 채널을 지원하기 때문에 당연히 채널별로 Pixel Intensity를 별도로 처리할 수 있음을 의미하고, 이는 당연히 ViDi Green Tool에서 단순히 컬러가 다른 이미지를 구분할 수 있다는 의미가 됩니다.

그래도 혹시나 해서 테스트 해보았습니다 (시간 낭비 일수도 있지만...).

테스트를 위해 4가지 컬러별로 121개씩의 이미지를 생성했습니다. 동일한 Tag에 속하는 이미지라도 적당히 색의 Variation을 주었으며, 이미지마다 서로 다른 Noise도 추가했습니다.

아래 이미지는 각 Tag에 대하여 3개의 대표 이미지 예시입니다.

<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/06.png">
<br/><br/>

결과는 예상대로 100% 분류율을 보이고 있습니다.
<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/07.png">
<br/><br/>

결론적으로, Green Tool을 이용한 이미지의 단순 컬러 분류가 가능하냐는 고객이 있다면 자신있게 그럴 수 있다고 대답하시면 됩니다!

------
## 3. Green Tool Multi-class 분류 기능은 어떻게?
ViDi 3.X로 업데이트 되면서 Green Tool의 새로운 기능이 추가된 것이 있는데, Multi-class 분류 기능입니다.

그런데, 문제는 이 또한 ViDi 3.1 공식 도큐먼트에는 사용법이 전혀 나와 있지 않습니다.

3.X의 가장 막강한 기능인 Blue Read에 너무 집중한 탓인지 (Blue Read에 대한 설명은 무려 12 페이지를 차지하고 있습니다), 아니면 설명할 필요없을 정도로 간단해서인지 Green Tool의 새 기능에 대한 설명을 빠뜨렸나 봅니다.

어쨌든 알아낸 사용법은 간단하지만, 사용법이 없으므로 몇 번의 삽질로 사용법을 알아냈습니다.
<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/08.png">
<br/><br/>

위의 이미지와 같이 Green Tool의 파라미터 중 Exclusive라는 것이 새로 추가된 것을 알 수 있습니다. 이에 대한 기본값은 Checked 상태이며, 이 상태는 단일 Class만 분류하겠다는 것입니다.

Checked 상태를 해제하면 Multi-class를 분류할 수 있는 옵션으로 전환한 것입니다.

그렇다면 Multi-class에 대한 Labeling은 어떻게 하느냐가 중요한 포인트입니다.

가령, 아래 이미지와 같이 이미지 내에 'lego'와 'pig'가 있다고 하면,
<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/09.png">
<br/><br/>

Labeling은 단순히 두 개의 Class를 ','로 분리해서 `lego, pig`와 같이 넣으면 됩니다.

Multi-class를 테스트할만한 이미지를 보유하고 있지 않으므로, 컴퓨터 그래픽스 도구를 이용하여 각 모델의 위치, 크기, 회전 등에 랜덤 노이즈를 적용하여 시퀀스로 렌더링하여 테스트 이미지를 생성하였습니다.

몇가지 테스트 이미지 예시는 다음 이미지와 같습니다.
<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/10.png">
<br/><br/>

Multi-class 특성 상 결과로 Confusion Matrix를 구성하지는 않으며, 아래와 같이 Table 형태로 결과를 확인할 수 있습니다.
<br/><br/>

<img src="{{ site.baseurl }}/assets/posts/2018-07-22-MultiClassClassification/11.png">
<br/><br/>

참고하고 싶어 하실 분이 혹시라도 계실 것 같아 HTML Report 링크 공유 드립니다.

- [Multi-class HTML Report](http://13.125.95.237/vidi/Multi-Class%20-%20Classify.html)
