---
title: Learning Frameworks in NLP (1/2)
date: 2023/07/16
category:
  - NLP
tag:
  - NLP
toc: true
---

# Intro

## LIMA and KoLIMA

최근에 KoLIMA라는 사이드 프로젝트를 하나 시작했습니다. [LIMA: Less Is More for Alignment](https://arxiv.org/abs/2305.11206)는 Meta AI에서 2023년 5월에 발표한 논문으로, 소규모의 엄선된 instruction dataset 만으로도 pre-training objective와 final objective 사이의 mismatch를 충분한 수준에서 align 할 수 있다는 내용을 다루고 있습니다.

우리의 최종 목표인 'LM이 Input으로 주어지는 사람의 지시*instruction*에 따르게 한다'와 Pre-training 단계의 학습 목표인 'Masked/Causal Language Modelling의 Loss를 낮춘다' 사이에는 Mismatch (혹은 Gap)이 존재하고, 이러한 상이한 학습 목표를 어떻게 Align 할 것인가는 최근 NLP 분야에서 많은 관심을 받아온 연구 주제였습니다. 이를테면, ChatGPT의 전신이자 Sibling 모형인 InstructGPT도 해당 문제를 다루고 있고, LLaMA에 Self-Instruct를 활용하여 instruction dataset을 augmentation해서 instruction-tuning을 적용한 Stanford Alpaca도 이러한 Alignment에 관심을 집중하고 있습니다.

아시다시피, ChatGPT의 핵심은 RLHF를 통한 Human Feedback을 학습 과정에 반영하는 것이지만, 해당 프로세스는 꽤 많은 비용을 필요로 합니다. 물론 최대한 자동화된 프로세스를 만들기 위해서 Human Feedback을 예측하는 모형을 만들고, 일정 시점 이후부터는 사람이 직접 LM의 output에 대한 Feedback을 annotation할 필요성이 줄어들긴 하지만, 그럼에도 불구하고 RLHF는 전체적으로 Costly한 과정일 수 밖에 없을 뿐더러, OpenAI는 학습에 사용한 데이터셋을 공개하지 않았기 때문에 데이터/모형의 재활용을 통한 비용 절감 또한 기대할 수 없는 상황이죠.

LIMA는 '그게... 그렇게까지 할 일인가?'라는 질문에서부터 출발합니다. [(Xie et al., 2021)](https://arxiv.org/abs/2111.02080)과 [(Ahuja et al., 2023)](https://arxiv.org/abs/2306.04891)을 비롯한 여러 논문들의 실험 결과가 암시하듯이, In-context learning ability를 비롯한 LLM의 대부분의 지식은 이미 Pre-training 단계에서 학습된 것이며, instruction-tuning은 단지 사람과 LM이 상호 작용하는 방식을 학습하는 간단한 과정일 수 있습니다. 따라서 'RLHF처럼 복잡한 과정 없이도, 엄선된 1k 규모의 instruction dataset만으로도 만족할만한 성능의 alignment를 수행할 수 있다'라는 내용이 LIMA의 저자들이 주장하고자 했던 핵심적인 내용이라고 볼 수 있습니다.

KoLIMA는 LIMA에서 주장하는 내용이 한국어 언어 모형에서도 동일하게 적용되는지 확인해보기 위한 프로젝트입니다. 우선 LIMA dataset을 DeepL API를 활용하여 한국어로 번역한 KoLIMA dataset을 생성하고, 사전 학습된 한국어 언어 모형인 Polyglot-ko을 백본 모형으로 instruction-tuning을 적용하여 모형의 성능을 평가해보고자 합니다. 현재 데이터셋의 번역은 완료된 상태이고, Polyglot-ko 1.3B, 3.8B 모형의 학습에 이어서 5.8B, 12.B 모형의 학습을 진행 중에 있습니다.

## This article covers:

이번 글에서는 지금까지 언급했던 내용들인 pre-training, fine-tuning, instruction-tuning, transfer learning 등의 개념을 이해하기 위한 배경 지식에 대해 다룹니다. 이전 글들에서 다루었던 CNNs, RNNs, Transformers 등의 개념이 LM을 구성하는 아키텍쳐 측면에서의 구성 요소였다면, 이번 글에서는 LM을 '어떻게 학습시킬 것인가'에 대한 내용을 다룹니다. 다시 말해, 최근 몇 년간의 NLP 분야에서의 연구 트랜드를 Learning Framework의 관점에서 정리해보도록 하겠습니다. 또한, 이 글은 [Lena Viota](https://lena-voita.github.io/nlp_course/transfer_learning.html)와 [Sebastian Ruder](https://www.ruder.io/state-of-transfer-learning-in-nlp/)의 블로그를 참고하고, 많은 영감을 받았음을 미리 밝힙니다.


# Learning Frameworks in NLP

## Supervised Learning with Task-specific Models

머신러닝을 공부하기 시작하면 가장 먼저 배우게 되는 개념들 중 하나가 바로 지도 학습과 비지도 학습에 대한 내용입니다. 지도 학습은 레이블이 붙어있는 학습 데이터셋을 활용하여 모형이 특정 Task를 수행하는 방법을 가르치는 훈련 방법이죠. 그리고 얼마 전까지만해도 대부분의 NLP Task들은 해당 Task를 수행하기 위한 구체적인 모형이 있고, 해당 모형에 지도 학습 적용하여 훈련시키는 방법을 사용했었습니다. 예를 들어, 텍스트 요약, 주제 분류, 번역과 같은 3개의 구체적인 task가 있다고 할 때, 각각의 task를 위한 세 개의 모형이 별도로 존재하는 형태였죠. 그림으로 표현해보자면 다음과 같습니다

![[Pasted image 20230716173622.png]]


## Pre-trained Models for General Knowledge

그런데, 텍스트 요약과, 주제 분류, 번역이라는 세 가지 Task를 잘 하기 위해서 필요한 공통적인 지식이 있을 수 있지 않을까요? 예를 들어, 한국어를 할 줄 모르는 사람에게 한글로된 뉴스 기사를 요약하고, 주제를 분류하고, 영어로 번역하는 것을 가르치는 것보다, 한국어를 이미 알고 있는 사람에게 동일한 내용을 가르치는 것이 훨씬 더 효율적일 것입니다.

![[Pasted image 20230716201512.png]]

이렇게 특정 Task *Specific Task*에 관계없이 일반적으로 활용될 수 있는 언어에 대한 지식을 일반 지식*General Knowledge*이라고 하고, 여기에는 개별 단어의 의미, 품사, 문장 내에서의 역할, 문법 구조 등이 포함됩니다. 이러한 일반 지식을 어떤 형태로 개별 모형들에게 전달할 수 있을까요?

가장 일반적인 방법은 단어 임베딩*Word Embeddings*에 이러한 정보들을 담아 개별 모형의 입력값으로 활용하는 것입니다. 혹은 [지난 글](https://taes.me/Knowledge%20Integration%20in%20Language%20Model/)에서 살펴봤던 것처럼 개체 임베딩*Entity Embeddings*을 활용할 수도 있습니다. 이러한 임베딩을 생성하기 위한 다양한 방법들이 연구되어 왔으며, 대표적으로는 word2vec, ELMo, BERT 등이 있습니다.

각각의 방법론들이 어떤 차이점을 지니고 있는지 Learning Framework의 관점에서 살펴보도록 하겠습니다

### word2vec: Static Word Embeddings

word2vec은 '함께 등장하는 단어 집합이 유사한 단어들은 서로 비슷한 의미를 지닌다'라는 간단한 가정을 기반으로 단어를 수치 벡터로 표현하는 방법입니다. 많은 분들께서 익숙하실 `vector("King") - vector("Man") + vector("Woman") = vector("Queen")`의 예시가 바로 word2vec의 [original paper](https://arxiv.org/abs/1301.3781)에서 처음 등장했습니다. 

![[Pasted image 20230716222738.png]]

다만 word2vec은 주변에 함께 등장한 단어를 고려하지 못하고 하나의 단어에 고정된 하나의 임베딩만을 할당한다는 단점이 있습니다. 다시 말해, word2vec을 통해 생성한 임베딩을 통해서는 [이 글](https://taes.me/Dependency%20in%20Languages/)에서 다루었던 여러 의미를 지닐 수 있는 '배'를 각각 다르게 표현할 수 없으며, 이러한 형태의 embedding을 우리는 static embedding 이라고 부릅니다.

### ELMo: Contextualised Word Embeddings

(아래에서 언급될 BERT와 동일하게) ELMo는 이러한 static embedding의 단점을 개선한 contextualised embedding을 생성하는 방법론입니다. 아래의 예시에서, static embedding이 단어 'cat'의 의미를 flying cat이든 the cat on the mat의 cat이든, the cat by the door의 cat이든 동일하게 표현할 수 밖에 없는 데에 반해, ELMo를 비롯한 Contextualised Embedding 기법들에서는 각각의 cat을 주변 맥락*Context*를 고려한 임베딩으로 나타낼 수 있습니다.

![[Pasted image 20230716223431.png]]
(Figure from Lena Voita)

지금까지 살펴본 것처럼 word2vec과 ELMo는 단어를 표현할 때 주변 맥락을 고려할 수 있는지 없는지에 따른 차이가 있긴 하지만, 모형에서 활용되는 방법에 있어서는 차이가 없습니다. 다시 말해, 아래의 그림에서 살펴볼 수 있는 것처럼 각각의 pre-trained model은 단어 집합을 입력으로 받아 각각의 단어에 대한 pre-trained embeddings을 생성하고, 이를 다시 task-specific한 모형에 전달하는 형태로 전체 모형에서 활용되게 됩니다.

![[Pasted image 20230716223735.png]]

### BERT: Single Model for ALL NLP Tasks with Contextualised Word Embeddings

반면 BERT는 다양한 Downstream Task에 대한 접근법이 다소 다릅니다. 여러 종류의 task-specific model이 별도로 존재하고, general knowledge는 pre-trained model로부터 생성된 embedding을 통해 개별 모형에 전달되던 기존 구조와 달리, BERT는 모든 downstream task를 BERT 하나만으로 수행해도 기존 개별 모형 대비 더 높은 성능을 달성할 수 있다는 사실을 보였습니다. 

![[Pasted image 20230716224255.png]]

# Outro

이번 글에서는 Sequential Transfer Learning의 기본적인 개념인 Pre-training과 Fine-tuning에 대해 살펴보았습니다. 다음 글에서는 In-context Learning, Prompting, Few/Zero-shot Learning, Instruction-tuning의 개념에 대해 살펴보도록 하겠습니다.


---


### Pre-trained Model -> Task-specific Model (Transfer Learning, fine-tuning)

언어에 대한 일반 지식을 task-specific model의 input으로 넣어주면 모형의 성능이 높아지지 않을까?
embedding 자체도 학습을 해야하고, random하게 init되던 상황
embedding에 대한 일종의 initialisation

word2vec (Static)
ELMo (Contextualised, Dynamic, Context 반영)

### Pre-trained Model -> Fine-tuning (fine-tuned model); BERT, GPT, ...

BERT
BERT로 얻은 임베딩 위에 레이어 한두개 얹어서 다양한 Task 수행


Pre-trained Model -> Multi-task fine-tuning; T5

### In-context learning, Prompting, Few-shot learning

### Instruction-tuning

다시 fine-tuning으로. 다만 task-specific한 fine-tuning이 아니라, instruct 따른다, 는 제너럴한 태스크를 위한 fine-tuning
InstructGPT, ChatGPT

Alignment


### Distillation, Augmentation, Adaptation

GPT-1: Improving Language Understanding by Generative Pre-Training (Jun 2018)
GPT-2: Language Models are Unsupervised Multitask Learners (Feb 2019)
GPT-3: Language Models are Few-Shot Learners (Jun 2020)
GPT-4: ? (Mar 2023)

It's Not Just Size That Matters: Small Language Models Are Also Few-Shot Learners
[What Does BERT Learn about the Structure of Language? - ACL Anthology](https://aclanthology.org/P19-1356/)



# Transfer Learning

최근 주목받고 있는 언어 모형들은 단일 모형으로 여러 종류의 Downstream NLP Tasks를 수행할 수 있습니다. 예를 들어, ChatGPT는 주어지는 Input에 따라 텍스트 요약, 분류, 번역 등 다양한 작업을 수행합니다. 하지만 불과 몇 년 전까지만 하더라도 이것은 일반적이지 않았습니다. 분류에는 분류를 위한 모형이, 요약에는 요약을 위한 모형이, 번역에는 또 번역을 위한 모형이 필요했죠. 각각의 모형들은 아키텍쳐도 달랐고, 훈련을 위해 사용되는 데이터셋도 달랐습니다. 


---


이렇게 특정 Task *Specific Task*에 관계없이 일반적으로 활용될 수 있는 언어에 대한 지식을 일반 지식*General Knowledge*이라고 하고, 여기에는 개별 단어의 의미, 품사, 문장 내에서의 역할, 문법 구조 등이 포함됩니다. 이러한 일반 지식을 어떤 형태로 개별 모형들에게 전달할 수 있을까요?

가장 일반적인 방법은 단어 임베딩*Word Embeddings*에 이러한 정보들을 담아 개별 모형의 입력값으로 활용하는 것입니다. 혹은 [지난 글](https://taes.me/Knowledge%20Integration%20in%20Language%20Model/)에서 살펴봤던 것처럼 개체 임베딩*Entity Embeddings*을 활용할 수도 있습니다. 이러한 임베딩을 생성하기 위한 다양한 방법들이 연구되어 왔으며, 대표적으로는 word2vec, ELMo, BERT 등이 있습니다.

각각의 방법론들이 어떤 차이점을 지니고 있는지 Learning Framework의 관점에서 살펴보도록 하겠습니다.




Learning Framework, 즉 어떻게 모형을 훈련시킬 것인가


![img](https://lena-voita.github.io/resources/lectures/transfer/intro/idea-min.png)


Transfer Learning

- Static embeddings - word2vec, glove, fasttext, elmo
- contextualised embeddings - BERT



Pretrained Language Models: 좌측의 모형에 관심이 집중
BERT - T5: Multi-task learning

GPT

LLaMA

Alpaca

LIMA

Polyglot



---

### 2.3.?? Scaling Law

GPT-1
GPT-2
GPT-3

### 2.3.2 Multi-taskers

GPT-2: Multi-task learner
Massive Multi-task Learning
T5: Multi-task fine-tuning
ExT5

### 2.3.3 Few-shot Learners & Prompting

Few-shot learning
Zero-shot learning
Prompting

### 2.3.4 Animal Kingdom

LLaMA / Alpaca / KoAlpaca
Gopher / Chinchilla
Dolly / Vicuna / RedPajama





%%### 2.2.1 word2vec: Static Word Embeddings

word2vec은 '함께 등장하는 단어 집합이 유사한 단어들은 서로 비슷한 의미를 지닌다'라는 간단한 가정을 기반으로 단어를 수치 벡터로 표현하는 방법입니다. 많은 분들께서 익숙하실 `vector('King') - vector('Man') + vector('Woman') = vector('Queen')`의 예시가 바로 word2vec의 [original paper](https://arxiv.org/abs/1301.3781)에서 처음 등장했습니다. 

![img3](https://i.imgur.com/gA6KUG8.png)

다만 word2vec은 주변에 함께 등장한 단어를 고려하지 못하고 하나의 단어에 고정된 하나의 임베딩만을 할당한다는 단점이 있습니다. 다시 말해, word2vec을 통해 생성한 임베딩을 통해서는 [이 글](https://taes.me/Dependency%20in%20Languages/)에서 다루었던 여러 의미를 지닐 수 있는 '배'를 각각 다르게 표현할 수 없으며, 이러한 형태의 embedding을 우리는 static embedding 이라고 부릅니다.

### 2.2.2 ELMo: Contextualised Word Embeddings



![Image to be digitised](https://i.imgur.com/pZYbdhH.png)

## 2.3 BERT: Single Model for ALL NLP Tasks with Contextualised Word Embeddings

반면 BERT는 다양한 Downstream Task에 대한 접근법이 다소 다릅니다. 여러 종류의 task-specific model이 별도로 존재하고, 언어에 대한 일반 지식은 사전 학습 모형으로부터 생성된 임베딩을 통해 개별 모형에 전달되던 기존 구조와 달리, BERT는 모든 downstream task를 BERT 하나만으로 수행해도 기존 개별 모형 대비 더 높은 성능을 달성할 수 있다는 사실을 보였습니다.

![iImage to be digitised](https://i.imgur.com/wexruys.png)%%