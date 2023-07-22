---
title: draft
date: 2022/02/28
category:
tags:
toc: true
---

### MLE 기반 접근법의 문제점 2. 언어의 역사성
두 번째로 살펴볼 성질은 언어의 역사성입니다. 언어의 형태는 앞서 살펴본 문법 등에 따라 변화할 뿐만 아니라, 시간이 흐름에 따라서도 변화합니다. 예를 들어, [‘covidiocy’](https://trends.google.com/trends/explore?date=today%205-y&q=covidiocy)라는 단어는 ‘바이러스의 확산을 통제하는 데에 필요한 공중 보건 조치를 거부하거나 무시하는 현상’을 가리킵니다. 이 단어는 2020년 이전에는 사용되지 않다가 2020년 3월 즈음에 새롭게 생겨난 신조어입니다. 또 다른 예로, [‘돈쭐’ 혹은 ‘돈쭐내다’](https://trends.google.com/trends/explore?date=today%205-y&q=%EB%8F%88%EC%AD%90)는 ‘돈으로 혼쭐을 내다’의 줄임말로, ‘몰래 한 선행의 대가로 큰 돈을 벌게 해주다’를 일종의 반어법을 이용해 나타낸 표현입니다. 해당 표현 또한 사용 빈도가 거의 없다가, 2021년 2-3월 즈음에 인기를 얻어 사용 빈도가 급격히 증가하였습니다. 앞서 살펴본 언어의 변동성에서와 마찬가지로, 우리가 수집한 데이터가 이러한 신조어에 대한 내용을 담고있지 못하다면 우리가 만든 언어 모델은 해당 단어들의 발생 확률을 모두 0으로 추정하게 되는 문제가 있습니다.

## Word Level
이러한 문제를 완화하기 위해서,

두 번째로는

이러한 문제를 조금 완화하기 위해, 우리는 단어 레벨

$$ P(w\_1=오늘은) \times P(w\_2=좀|w\_1=오늘은) \times P(w\_3=쉬려고|w\_1=오늘은,\ w\_2=좀) $$

## Conditional Language Modelling — with Markov Assumption
## N-grams


---- 
ERNIE (Zhang et al., ACL 2019)
KGLM (Logan et al., ACL 2019)
KNN-LM (Khandelwal et al., ICLR 2020)
WKLM (Xiong et al., ICLR 2020)
ERNIE (Sun et al., arXiv 2019), Salient Span Masking (Guu et al., ICML 2020 & Roberts et al., EMNLP 2020)


