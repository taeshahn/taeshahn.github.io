---
title: "[Taes' R #2]() '좋은 형태의 데이터'와 'Tidiness'에 대하여"
categories:
  - Taes' R
tags:
  - Taes' R
date: 2019-11-16  
---

## 1. 지난 이야기
[지난 글]()[1]()에서는 일관된 인터페이스(Consistent Interface)에 대해 살펴보았다. 오늘의 주제에 대해 이야기 하기에 앞서, 잠시 분석 프로세스를 최대한 멀리서 한 번 바라보도록 하자. R을 이용한 데이터 분석을 최대한 간단하게 나타낸다면, 다음과 같이 '데이터'를 분석을 위한 '함수'에 넣어서 '분석 결과'를 도출하는 과정으로 표현할 수 있을 것 같다.

![analysis_process]()(analysis_process.jpg)

이전 글에서 일관된 인터페이스를 지닌 함수에 대해 살펴보았다면, 이번 글에서 관심을 갖고 살펴볼 대상은 또 하나의 핵심 요소인 '데이터' 그 자체이다.

## 2. 내가 생각한 플롯은 이게 아닌데...?
간단한 데이터 시각화 예시로부터 이야기를 시작해보자. 우리는 시계열 데이터와 R의 기본 그래픽 함수 plot()을 이용해서 두 가지 플롯을 그려볼 것이다. 하나는 시간의 흐름에 따른 값의 변화를 나타내는 라인 그래프가 될 것이고, 다른 하나는 해당 기간동안의 값의 분포를 보여주는 히스토그램이 될 것이다.

샘플 데이터로는 ggplot2 패키지의 economics 데이터를 이용해보도록 하겠다. 해당 데이터셋은 미국의 경제 관련 지표들에 대한 시계열 데이터를 담고 있고, 우리는 이 중에서 psavert라는 항목만을 이용할 것이다.


	`R
library(tidyverse)
data(economics)
head(economics)

data_ts = economics %\>% pull(pce)
	`

`plot()`함수의 Document를 확인해보면, 다음과 같이 플롯의 타입을 지정해 줄 수 있는 것을 확인할 수 있다.

> `type`: what type of plot should be drawn. Possible types are
> 
> * `"l"` for lines
> * `"h"` for ‘histogram’ like (or ‘high-density’) vertical lines

우선 라인 그래프 먼저 그려보자. 미리 말해두자면, 이 친구에게는 아무런 문제가 없다.


	`R
plot(x=data_ts, type='l')
	`



![png]()(output_7_0.png)



다음은 type='h' 옵션을 이용한 히스토그램의 차례다.


	`R
plot(x=data_ts, type='h')
	`



![png]()(output_9_0.png)



...? 그냥 위에 있는 그래프 아래 면적 색칠한 것 같은데?~~ 우리가 원했던 히스토그램이 아닌, 주어진 데이터 값을 높이로 하는 vertical bar 플롯이 생성되었다. 사실 type='h' 옵션을 주면서 히스토그램을 그리고 싶다면, 원본 데이터가 아니라 원본 데이터의 도수 분포표를 생성해서 입력값으로 넣어줘야 한다.

우리가 원했던 형태의 플롯을 얻기위해서 데이터를 8개의 등구간으로 분할한 도수분포표를 구한 후, 이를 통해 히스토그램을 그리는 코드는 다음과 같다.


	`R
br = seq(min(data_ts), max(data_ts), length.out=9)
freq_table = table(cut(data_ts, breaks=br))
names(freq_table) = paste("≤", round(br[-1](), 0))
plot(freq_table, type='h', lwd=60, lend=1)
	`



![png]()(output_11_0.png)



왜 처음에 예상과 다른 그래프가 그려졌을까? `plot()` 함수가 요구하는 데이터의 형태(도수분포표)가 내가 생각했던 데이터의 형태(원본 데이터)와 달랐기 때문이다. 지난 글에서와 비슷한 문제가 여기에도 존재한다. '우리는 그래프마다 정해져있는 데이터의 포멧을 또 모두 외워야하는가?'에 대한 문제이다.

사실 Plotting을 위한 함수와 데이터를 정의하는 데에는 다음과 같은 두 가지 방법이 있다.

* 데이터의 표준형을 정해두고 Plotting 함수를 데이터에 맞춰 정의하는 방법
* Plotting 함수마다 다르게 정의된 데이터 형태에 맞추어 데이터 포멧을 변경하는 방법

두 방법 중 어떤 방법이 더 효율적일까? 복잡하게 따져볼 것도 없이 전자의 방법이 훨씬 더 효율적인 방법이다. 데이터 분석 과정에서 대부분이 시간이 데이터 처리에 소요된다는 것은 널리 알려진 사실이다. 그만큼 데이터의 형태를 바꾸는 것은 상당한 작업과 시간이 소요되는 작업이다. 따라서 데이터를 표준적인 형태로 한 번만 변형해놓고, 해당 데이터셋을 계속 이용할 수 있도록 함수를 정의하는 것이 훨씬 더 효율적인 방법이라고 볼 수 있다. 이러한 이유로 전자의 방법을 따르기로 했다면, 자연스럽게 따라오는 질문이 하나 있다. 

> 데이터의 어떤 형태가 좋은 형태인가? 어떤 데이터 포멧을 표준형으로 정해야하나?

R 생태계의 절대자~~Hadley Wickham 교수는 이러한 질문에 대한 답변으로 Tidy Data라는 개념을 제안한다. 이 글에서는 Tidy Data의 개념을 포함하여 다음과 같은 총 세 가지 관점에서 데이터의 형태에 대해 이야기 해 볼 예정이다.

* Raw Data vs Summary Data (Full Data vs Aggregated Data)
* Long Form vs Wide Form
* Tidy Data vs Messy Data

## 3.데이터의 형태를 바라보는 다양한 관점

#### (1) Raw Data vs Summary Data (Full Data vs Aggregated Data)

앞서 economics 데이터셋에서 살펴보았던 원본 데이터와 도수분포표의 관계가 바로 이 첫번째 경우에 해당한다. 이 둘을 구분하는 기준은 '데이터 형태 변화에 따른 정보량의 감소 여부'이다. 다르게 표현하자면 '데이터의 형태 변환이 Invertible한가?'라고 이야기 할 수도 있겠다.

`data_ts`가 주어지면, 라인 그래프와 (데이터 변환이 필요하기는 하지만) 히스토그램을 모두 생성할 수 있다. 그러나 반대로 `freq_table`만 주어지는 경우, 히스토그램은 그릴 수 있지만 라인 그래프는 생성할 수가 없다. `freq_table`은 `data_ts`가 Summary된 형태의 데이터이고, Sumamry 과정에서 일부 정보가 소실되었기 때문이다. 이러한 형태의 데이터 변환(연산)은 굉장히 쉽게 찾아볼 수 있는데, 대표적으로 우리가 매우 흔하게 접하는 평균을 구하는 연산 또한 그 중 하나이다.

이를테면 어느 회사의 모든 구성원의 연봉 정보를 알 수 있다면 평균 연봉을 구할 수 있지만, 평균 연봉을 안다고 해서 개인별 연봉 정보를 다시 복원할 수는 없다. 이는 전체 데이터인 '개별 연봉 정보'가 대표값인 '평균 연봉'으로 집계/요약(Aggregation/Summary)되면서 정보량이 감소했기 때문이다. 또 다른 대표적인 예로는 SQL에서의 group by aggregation을 생각해 볼 수도 있겠다.

다시 원래의 주제로 돌아와서, 어떤 형태가 표준형에 적합한지에 대해 이야기해보자면 Sumamry Data로는 표현할 수 있는 정보에 한계가 있기 때문에 Full Data가 상대적으로 적합한 형태라고 볼 수 있겠다.

#### (2) Long Form vs Wide Form

앞서 살펴본 (1)번의 경우는 데이터 형태에 따라 정보량에 차이가 발생하게 되는 구분이었다. 이와 달리, 정보량 자체에는 차이가 없더라도 데이터의 형태는 다를 수 있다. 두번째로는 이러한 케이스에 해당하는 Long Form과 Wide Form에 대해서 살펴보도록 하자.

Long Form과 Wide Form은 각각 다음과 같은 형태를 의미한다.
  - Wide Form: 서로 다른 속성을 나타내는 값들이 각각 하나의 컬럼을 이루는 형태
  - Long Form: 속성의 종류와 해당 속성의 값을 나타내는 컬럼이 각각 하나씩 존재하는 형태

설명만으로는 충분히 와닿지 않을 수 있으니 데이터 샘플을 통해 형태를 확인해보도록 하자. 예시에는 앞서 활용했던 economics 데이터셋을 그대로 이용하도록 하겠다.


	`R
economics_row3 = economics %\>% head(3)
economics_row3
	`


\<table class="dataframe"\>
\<caption\>A tibble: 3 × 6\</caption\>
\<thead\>
\<tr\>\<th scope=col\>date\</th\>\<th scope=col\>pce\</th\>\<th scope=col\>pop\</th\>\<th scope=col\>psavert\</th\>\<th scope=col\>uempmed\</th\>\<th scope=col\>unemploy\</th\>\</tr\>
\<tr\>\<th scope=col\>&lt;date&gt;\</th\>\<th scope=col\>&lt;dbl&gt;\</th\>\<th scope=col\>&lt;dbl&gt;\</th\>\<th scope=col\>&lt;dbl&gt;\</th\>\<th scope=col\>&lt;dbl&gt;\</th\>\<th scope=col\>&lt;dbl&gt;\</th\>\</tr\>
\</thead\>
\<tbody\>
\<tr\>\<td\>1967-07-01\</td\>\<td\>506.7\</td\>\<td\>198712\</td\>\<td\>12.6\</td\>\<td\>4.5\</td\>\<td\>2944\</td\>\</tr\>
\<tr\>\<td\>1967-08-01\</td\>\<td\>509.8\</td\>\<td\>198911\</td\>\<td\>12.6\</td\>\<td\>4.7\</td\>\<td\>2945\</td\>\</tr\>
\<tr\>\<td\>1967-09-01\</td\>\<td\>515.6\</td\>\<td\>199113\</td\>\<td\>11.9\</td\>\<td\>4.6\</td\>\<td\>2958\</td\>\</tr\>
\</tbody\>
\</table\>



편의를 위해 economics에서 가장 상위의 3개 행만 뽑아서 살펴보면, 해당 데이터는 date라는 컬럼을 기준으로 해당 일자의 pce, pop, psavert, unempmed, unemploy 라는 5가지 속성을 담고있는 데이터라는 것을 알 수 있다. 각 행을 유니크하게 식별할 수 있는 Unique Key는 Date이며, **5개의 속성은 각각의 컬럼으로 표현**되고 있으므로 이 데이터는 **Wide Form**이라고 볼 수 있다.

이 데이터를 long form으로 변환하면 다음과 같은 형태가 된다.


	`R
economics_row3 %\>% 
  pivot_longer(cols=c(pce, pop, psavert, uempmed, unemploy))
	`


\<table class="dataframe"\>
\<caption\>A tibble: 15 × 3\</caption\>
\<thead\>
\<tr\>\<th scope=col\>date\</th\>\<th scope=col\>name\</th\>\<th scope=col\>value\</th\>\</tr\>
\<tr\>\<th scope=col\>&lt;date&gt;\</th\>\<th scope=col\>&lt;chr&gt;\</th\>\<th scope=col\>&lt;dbl&gt;\</th\>\</tr\>
\</thead\>
\<tbody\>
\<tr\>\<td\>1967-07-01\</td\>\<td\>pce     \</td\>\<td\>   506.7\</td\>\</tr\>
\<tr\>\<td\>1967-07-01\</td\>\<td\>pop     \</td\>\<td\>198712.0\</td\>\</tr\>
\<tr\>\<td\>1967-07-01\</td\>\<td\>psavert \</td\>\<td\>    12.6\</td\>\</tr\>
\<tr\>\<td\>1967-07-01\</td\>\<td\>uempmed \</td\>\<td\>     4.5\</td\>\</tr\>
\<tr\>\<td\>1967-07-01\</td\>\<td\>unemploy\</td\>\<td\>  2944.0\</td\>\</tr\>
\<tr\>\<td\>1967-08-01\</td\>\<td\>pce     \</td\>\<td\>   509.8\</td\>\</tr\>
\<tr\>\<td\>1967-08-01\</td\>\<td\>pop     \</td\>\<td\>198911.0\</td\>\</tr\>
\<tr\>\<td\>1967-08-01\</td\>\<td\>psavert \</td\>\<td\>    12.6\</td\>\</tr\>
\<tr\>\<td\>1967-08-01\</td\>\<td\>uempmed \</td\>\<td\>     4.7\</td\>\</tr\>
\<tr\>\<td\>1967-08-01\</td\>\<td\>unemploy\</td\>\<td\>  2945.0\</td\>\</tr\>
\<tr\>\<td\>1967-09-01\</td\>\<td\>pce     \</td\>\<td\>   515.6\</td\>\</tr\>
\<tr\>\<td\>1967-09-01\</td\>\<td\>pop     \</td\>\<td\>199113.0\</td\>\</tr\>
\<tr\>\<td\>1967-09-01\</td\>\<td\>psavert \</td\>\<td\>    11.9\</td\>\</tr\>
\<tr\>\<td\>1967-09-01\</td\>\<td\>uempmed \</td\>\<td\>     4.6\</td\>\</tr\>
\<tr\>\<td\>1967-09-01\</td\>\<td\>unemploy\</td\>\<td\>  2958.0\</td\>\</tr\>
\</tbody\>
\</table\>



Wide Form에서 하나의 행으로 표현되던 1967년 7월 1일의 데이터는 Long Form에서는 총 5개의 행으로 변환되어 표현되고 있다. 5개의 컬럼으로 표현되던 각각의 속성이, 속성의 이름과 값을 담고있는 5개의 행으로 변환되었으며, 이에 따라 전체 행의 개수는 기존의 3개에서 3(기존 행)\*5(속성의 수)=15개로 증가한 것을 살펴볼 수 있다. 하나의 행이 여러 개의 행으로 분리되어 표현되었므로 더 이상 date 컬럼만으로는 각 행을 유니크하게 식별할 수 없으며, Unique Key가 date + name으로 변경된 것 또한 확인할 수 있다.

이미 눈치챈 사람들도 있겠지만, 데이터의 Long/Wide Form 전환은 우리에게 엑셀 혹은 SQL을 통해 이미 친숙한 개념인 Pivot과 매우 연관이 있다. 다시 말해, 우리는 Pivot을 통해 Long Form의 데이터를 데이터를 Wide Form으로, 혹은 그 반대로 변환할 수 있다.

이번 절에서 우리는 데이터의 Long/Wide Form이란 무엇이고, 각 형태 사이의 변환이 어떤 의미를 지니는지에 대해 살펴보았다. 어떤 형태의 데이터를 의미하는 것인지는 알겠는데, 도대체 이런 형태 변환을 왜 하는지에 대한 의문이 드는 것은 어쩌면 자연스러운 일일 것이다. 이러한 의문을 해소하고 조금 더 실용적인 Long/Wide Form의 변환 예시를 확인하고 싶다면 [다음 글]()[2]()을 살펴보도록 하자.

#### (3) Tidy Data vs Messy Data
앞서 언급했듯이 Tidy Data는 Hadley Wickham 교수가 2014년 8월에 Journal of Statistical Software에 기고한 그의 [논문]()[3]()에서 제안한 개념으로, 다음과 같은 세 가지 조건을 만족하는 데이터셋을 의미한다.

1. 각 변수는 하나의 열(column)을 구성한다. (Each variable forms a column.)
2. 각 관측치는 하나의 행(row)를 구성한다. (Each observation forms a row.)
3. 각 관측 기준의 데이터는 하나의 테이블을 구성한다. (Each type of observational unit forms a table.)


정의 자체는 어렵지 않지만, 왜 이러한 조건들이 좋은 데이터의 기준이 되는지는 잘 와닿지 않을 수도 있다. 만약 데이터베이스에 대한 약간의 지식이 있다면 Tidy Data의 개념을 가장 쉽게 이해할 수 있는 방법은 이를 정규형(Codd's Normal Form)과 연관지어 생각해보는 것이다. 결론부터 이야기하자면 Tidy Data는 데이터베이스에서의 '제3정규형'을 의미한다. 두 개념이 어떻게 연관되는지가 궁금하다면, [다음 페이지]()[4]()가 Tidy Data와 제 3정규형의 동치에 대해 간략한 내용을 담고 있으므로 도움이 될 것으로 보인다.

> This is Codd’s 3rd normal form (Codd 1990), but with the constraints framed in statistical language, and the focus put on a single dataset rather than the many connected datasets common in relational databases. Messy data is any other arrangement of the data.

데이터 정규화의 정의나 필요성, 그리고 그 유용함에 대해서는 이미 잘 정리되어 있는 글들이 많으니 설명을 줄이고자 한다. 다만 여기에서 이야기하고자 하는 Tidiness의 개념은, 데이터베이스에서 추구해온 Efficiency와는 다소 다르다는 점에 유의할 필요가 있다. Tidy Data는 다소간의 중복 등을 포함하더라도 최대한 정보량을 보존하면서, 다른 형태로의 규칙적인 전환이 쉬운 데이터이며, 효율적인 저장과 처리보다는 분석 목적의 활용에 보다 초점이 맞춰져있는 데이터의 형태라고 이해하면 될 것 같다.

## 4. 마치면서
데이터 과학과 관련된 공부를 하다보면, Tidy Data와 Codd's Normal Form처럼 서로 다른 두 분야가 영향을 주면서 발전하거나, 독립적으로 발전했음에도 불구하고 결국에는 같은 결론에 도달한 경우를 자주 볼 수 있다. 이를 보며 문득 베이즈 정리가 떠올라 관련된 이야기로 글을 마무리해보고자 한다. 베이지안 추론에 대한 하나의 해석은 다음과 같이 표현된다.


$$lim_{Experiences \to \infty}(Inference\ Based\ on\ Experiences) = Truth$$

다시 말해, 베이즈 정리의 간단한 수식에는 우리가 현상에 대한 점점 더 많은 정보(혹은 경험, 혹은 데이터)를 모을수록 진리에 한 걸음 더 가까이 다가갈 수 있다는 베이지안 학파의 수학적, 철학적 관점이 반영되어 있다. 각각 독립적인 분야에서 시작된 연구들이 어느 정도 수준에 이르러서 비슷한 결론으로 수렴한다는 사실은 이러한 해석을 다시 한번 떠올려보게 하며, 매우 흥미로운 관점이 아닐 수 없다.

[1](): https://taes.me/TaesR01/
[2](): https://taes.me/TaesR03/
[3](): https://www.jstatsoft.org/article/view/v059i10/v59i10.pdf
[4](): https://datascience.stackexchange.com/questions/43184/equivalence-of-tidy-data-and-third-normal-form