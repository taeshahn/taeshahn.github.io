---
title: "Hexo 블로그로의 전환과 Vexo 테마 수정기"
categories:
	- Others
tags:
	- Others
date: 2021-05-08
---

이 글은 오늘은 또 무슨 딴짓을 하고 살았는지에 대한 기록이다.

\<center\>

![img]()(https://w.namu.la/s/8432f174b7725019735b726c6a8c48b8674d93794cf54d5fb60b10781cef8987cf3259451222879d3b3a84c5195f7143d32454f1b00ce4478cba8e27f3b4e15d2bf4f63993a23468d788e257a8efd55a11f82f41d7e75a8f0041fbc12711b9a8)

 하라는 데이터 공부는 안하고
 \</center\>

# 1. 글을 조금 작성해보려하면 찾아오시는 그 분

고질병이 다시 찾아와서 블로그 테마를 바꿨다. 사실 테마만 바꾼 수준이 아니라 Static Site Generator 자체를 Jekyll에서 Hexo로 변경했다. 변경한 이유는 여러 가지가 있지만, Jupyter로 생성한 이미지 파일들을 마크다운에서 경로 수정없이 바로 참조할 수 있게 되었다는 점이 가장 좋은 것 같다. 이외에도 포스팅 관련 파일과 블로그 관련 파일을 별도로 버전 관리할 수 있게 되었다는 점도 마음에 든다.

Jekyll에서는 [Minimal-Mistakes]()[1]()라는 테마를 사용했었다. Minimal-Mistakes는 7.9K Star, 14.5K Fork를 자랑할 뿐만 아니라 Contributor만도 225명이나 되는 굉장히 인기있는 테마다. 이미 어지간한 기능들은 테마에 다 포함되어 있기 때문에, 옵션만 조금 조정해주면 어렵지 않게 커스터마이징해서 쓸 수 있는 강력한 테마였다.

한편 Hexo로 넘어오면서 선택한 테마는 [Vexo]()[2]()였다. Hexo의 개발자가 대만 출신이라서 관련 생태계에서 중화권의 비율이 꽤 높은 것으로 알고 있는데, GitHub 프로필을 참고해보면 이 테마 또한 알리페이에서 근무하는 중국 개발자분이 만드신 것 같다. 다른 부분은 별로 고민하지 않았고 단순히 샘플 웹페이지의 디자인이 마음에 들어서 선택했다.

미적 감각이라고는 없는 내가 이런 멋진 테마를 무료로 사용할 수 있다는 것은 정말 감사한 일이지만, 항상 그렇듯이 직접 기획해서 만든 게 아니라면 아쉬운 부분들이 눈에 들어오기 마련이다. 대충 적응해서 써보려고 했지만, 미적인 부분들이 아니라 기능적으로 크리티컬한 부분들이 자꾸 눈에 들어와서 결국 한번 작정하고 수정해보기로 했다.

# 2. 그래서 어디를 고치겠다고?

그래서 일단 수정해야할 부분들을 List-up 해보았다. 개인적으로 수정할 부분들은 아래 리스트 이외에도 조금 더 있었지만, 추가적으로 필요하거나 현재 문제가 있는 기능들은 5개 내외 정도로 정리해 볼 수 있었다.

1. 글의 Table of Contents 링크가 정상적으로 작동하지 않는 문제 
2. Tags 레이아웃에서 각 태그 레이블의 링크가 정상적으로 작동하지 않는 문제
3. `Utterances`를 댓글 시스템으로 사용할 수 있도록 추가
4. 블로그 내 검색 기능 추가
5. Tags 레이아웃 이용 시, 태그 및 포스팅이 정렬되지 않는 문제
6. 기타 자잘한 수정사항들
	- 하드코딩된 한자 제거
	- 깨진 URL/이미지 제거 및 대체
	- Minor한 CSS 수정


# 3. 하나씩 차례대로!

지금부터는 앞서 언급한 리스트의 각 항목들이 어떤 맥락에서 수정이 필요했고, 또 어떻게 수정을 했는지 하나씩 기록해보도록 하겠다.

#### (1) 글의 Table of Contents 링크가 정상적으로 작동하지 않는 문제  
정확히 이야기하자면 링크되는 단락에 '한글이 포함되어 있는 경우'에만 이동이 정상적으로 이루어지지 않았다. 예를 들자면, `Taes' R` 시리즈 첫번째 글의 Table of Contents에서, 첫번째 단락인 `0. 시작하기에 앞서`을 눌러도 아무런 반응이 없었다. 확인해보니 다음과 같이 링크는 정상적으로 생성이 되었는데, 한글에 해당하는 링크의 뒷부분이 정상적으로 디코딩되지 않아서 발생하는 문제였다. 링크 생성 시 URL을 디코딩하도록 수정해줌으로써 간단히 해결할 수 있었다.

> \< 다음: 아래의 두 링크의 내용은 동일하다. \>
> 
> `https://taes.me/TaesR01/#0-시작하기에-앞서`
> 
> `https://taes.me/TaesR01/#0-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0%EC%97%90-%EC%95%9E%EC%84%9C`

#### (2) Tags 레이아웃에서 각 태그 레이블의 링크가 정상적으로 작동하지 않는 문제
이 문제도 정확히 이야기하자면 태그에 특수문자(공백 포함)가 포함되어 있는 경우에만 발생했다. 예를 들자면 Tag가 `TaesR`인 경우에는 정상적으로 링크가 작동해서 스크롤링이 되었지만, `Taes' R`이나 `Taes R` 처럼 특수문자가 포함된 경우에는 클릭 시에도 아무런 반응이 없었다.  도대체 왜 반응이 없는지 찾느라 시간이 조금 걸리긴 했지만, 정규표현식으로 특수문자들을 Escaping 처리해줌으로써 이 문제도 어렵지 않게 해결할 수 있었다.

#### (3) `Utterances`를 댓글 시스템으로 사용할 수 있도록 추가
Vexo 테마에 `Utterances`와 유사한 `Gitment`를 댓글 시스템으로 이용할 수 있는 기능이 포함되어 있긴 했다. 처음에는 그냥 `Gitment`를 이용하려고 했는데, 내가 설정을 잘못했는지, 아니면 GitHub Pages를 private repository에서 호스팅하고 있어서 그런지 모르겠지만 도저히 정상적으로 작동하지를 않았다. 그래서 결국 Utterances를 삽입하는 스크립트를 추가하고, 테마의 `_config.yml` 파일에서 선호하는 댓글 시스템을 설정할 수 있도록 수정하였다.

#### (4) 블로그 내 검색 기능 추가
사실 이 부분이 가장 어려웠는데... 수정을 하고 있긴 하지만 내가 웹 개발도 Hexo의 구조도 잘 모르는 상태에서 거의 거의 눈치로 때려맞추는 수준으로 고치고 있는 것이 문제였다. `hexo-generator-searchdb` 모듈을 사용하면 될 것 같긴한데... 사실 이 모듈 [설명 페이지]()[3]()에도 대충 `search view`, `search script`를 작성한 후에 둘을 `connect` 해주면 된다고만 되어있고, 나처럼 배경지식이 없는 사람을 위한 친절한 설명까지는 없었다.

자세한 내용은 NexT 테마의 소스 코드를 참고하라고 하길래, 일단 NexT 테마를 들고와서 구조를 뜯어봤다. `Local Search`와 관련된 소스들과, 해당 소스를 참조하는 소스가 다음과 같은 구조로 되어 있는 것을 확인할 수 있었다.

	`
layout/\_layout.swig
layout/\_partial/header/index.swig
layout/\_partial/header/menu.swig -- insert search button
layout/\_partial/search/index.swig
layout/\_partial/search/localsearch.swig -- search view
layout/\_third-party/index.swig
layout/\_third-party/search/localsearch.swig -- connect
source/js/local\_search.js -- search script
	`

아, 잠깐만... swig요...? vexo는 ejs 쓰던데...?\~\~

대충 복사/붙여넣기 해서 돌려볼 생각이었던 나는 확장자가 다른 것을 확인한 순간 내 실력으로 수정하기에는 글러먹었구나 싶었다. 그래도 조금 찾아보니까 둘 다 템플릿 엔진의 한 종류이고, 어떤 식으로 작동하는지는 정도는 확인할 수 있었다. 한번만 해보고 안되면 포기할 생각으로 수정을 시작했다. 일단 참조하는 테마와 유사하게 다음과 같이 구조를 잡았다.

	`
layout/layout.ejs
layout/\_partial/header.ejs -- insert search button
layout/\_partial/search/localsearch\_button.ejs -- search button
layout/\_partial/search/localsearch.ejs -- search view
layout/\_partial/head.ejs
layout/\_third-party/localsearch.ejs -- connect
source/js/local-search.js -- search script
	`

중간에 삽질을 많이 하긴 했지만, 조금만 더 하면 될 것 같아서 대충 눈치로 수정을 거듭하다보니 동작을 했다. 다만, 기존 소스에서는 설정할 수 있는 Hexo의 변수를 자바스크립트(`local-search.js`)로 넘겨서 처리하는 부분이 있었는데, 어떻게 넘기는건지 잘 모르겠어서 `local-search.js`에 그냥 하드코딩으로 박아버렸다. (...) 

어쨌든 이리저리 수정을 하다보니 정상적으로 검색이 되는 수준까지는 수정을 할 수 있었다. 물론 기능은 작동을 한 후에도 검색 팝업창의 디자인은 조금 이상했지만, 크롬 개발자 도구 통해서 class 속성이나 css를 확인해가면서 적당히 수정해주었다.


*(View가 모바일로 변경되면서 search 버튼이 사라지는 문제가 있어서 현재는 재수정한 상태이다.)*

### (5) Tags 레이아웃 이용 시, 태그 및 포스팅이 정렬되지 않는 문제
이건 사실 버그라기보다는, 원래 기획된 기능과 내가 바라는 기능이 달랐던 부분인 것 같다. Tag를 이용하여 글들을 Grouping할 때에는 굳이 포스팅이나 태그에 대한 정렬이 필요하지 않을 수도 있다. 하지만 나는 [Velog]()[4]()에서의 시리즈처럼 글들을 특정 주제로 묶고, 일관된(랜덤하지 않은) 순서로 보여줄 수 있는 레이아웃이 있었으면 했다. 마침 포스팅에 지정할 수 있는 category를 이용한 레이아웃이 없길래, `post.categories` 변수를 이용해서 시리즈 기능을 할 수 있는 레이아웃을 만들어봤다. Hexo에서 제공하는 변수들과 자바스크립트 자체에 익숙하지 않아서 시행 착오가 조금 있긴 했지만, 그래도 블로그 내 검색 기능 추가하는 것보다는 수월하게 추가할 수 있었다.

### (6) 기타 자잘한 수정사항들
이외에도 이미지나 URL이 깨져있다거나, 한자가 하드코딩 되어있어서 페이지에 보인다거나, CSS 스타일에 조금 조정이 필요하다거나 하는 부분들에 대한 사소한 수정들을 진행했다.

# 4. PULL REQUEST

수정을 해놓고 보니 내게 맞춰서 커스터마이징한 부분들만 있는 것이 아니라, 기능적으로 추가되거나 버그를 수정한 부분들이 꽤 많았다. 그래서, 받아들여질지는 모르겠지만 수정한 내용을 정리를 좀 해서 원본 테마 Repository에 PR를 해보기로 마음먹었다. 이왕 수정한 소스 남들도 같이 사용하면 좋을 것 같고, 솔직히 말하자면 내가 수정에 쏟아부은 하루의 시간이 아깝기도 했다.

그래서 어떤 기능을 추가/수정을 했고, 그러한 작업을 위해서 어느 소스 코드를 수정했는지 아래와 같이 정리하고, `README.md`에 새로운 기능들을 설정할 수 있는 방법들도 조금 작성해서 [Pull Request]()[5]()를 날려보았다.

### \< The List of New/Updated Sources \>

| File Path/Name                                      | ChangeType | Utterances | Local Search | Series (Category Layout) | Broken Link                                   | Hard-coded Chinese                            | Symbols Escaping for Tags/Categories Hyperlink | URL Decoding for ToC HiperLink | CSS                                           | [\\\\\\\\\\\\\\#]() NOTE                          |
| --------------------------------------------------- | ---------- | :--------: | :----------: | :----------------------: | :-------------------------------------------: | :-------------------------------------------: | :--------------------------------------------: | :----------------------------: | :-------------------------------------------: | ------------------------------------------------- |
| \\\_config.yml                                      | Modified   | O          | O            |                          | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> |                                               |                                                |                                |                                               | Removed                                           |
| layout/page.ejs                                     | Modified   | O          |              |                          |                                               |                                               |                                                |                                |                                               |                                                   |
| source/js/local-search.js                           | New        |            | O            |                          |                                               |                                               |                                                |                                |                                               |                                                   |
| source/css/\\\_partial/search.styl                  | New        |            | O            |                          |                                               |                                               |                                                |                                |                                               |                                                   |
| layout/\\\_partial/head.ejs                         | Modified   |            | O            |                          |                                               |                                               |                                                |                                |                                               |                                                   |
| layout/\\\_partial/header.ejs                       | Modified   |            | O            |                          |                                               |                                               |                                                |                                |                                               |                                                   |
| source/css/style.styl                               | Modified   |            | O            | O                        |                                               |                                               |                                                |                                | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> | [modify]() display: inline-grid                   |
| layout/\\\_third-party/localsearch.ejs              | New        |            | O            |                          |                                               |                                               |                                                |                                |                                               |                                                   |
| layout/\\\_partial/search/localsearch\\\_button.ejs | New        |            | O            |                          |                                               |                                               |                                                |                                |                                               |                                                   |
| layout/\\\_partial/search/localsearch\\\_view.ejs   | New        |            | O            |                          |                                               |                                               |                                                |                                |                                               |                                                   |
| \\\_source/series/                                  | New        |            |              | O                        |                                               |                                               |                                                |                                |                                               |                                                   |
| layout/series.ejs                                   | New        |            |              | O                        |                                               |                                               |                                                |                                |                                               |                                                   |
| source/css/\\\_partial/categories.styl              | New        |            |              | O                        |                                               |                                               |                                                |                                |                                               |                                                   |
| source/js/script.js                                 | Modified   |            |              | O                        |                                               |                                               | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\>  | O                              |                                               | Escaping Special symbols in Tag                   |
| layout/project.ejs                                  | Modified   |            |              |                          |                                               | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> |                                                |                                |                                               | Replace `开源项目` with `GitHub Open Source Projects` |
| layout/tags.ejs                                     | Modified   |            |              |                          |                                               | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> |                                                |                                |                                               | Replace `标签检索` with `Tag Retrieval`               |
| layout/archive.ejs                                  | Modified   |            |              |                          |                                               | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> |                                                |                                |                                               | Replace `文章归档` with `Archives`                    |
| source/css/images/error\\\_icon.png                 | New        |            |              |                          | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> |                                               |                                                |                                |                                               | Replaced                                          |
| source/css/\\\_partial/about.styl                   | Modified   |            |              |                          |                                               |                                               |                                                |                                | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> | [remove]() opacity: 0.25                          |
| source/css/\\\_partial/catalog.styl                 | Modified   |            |              |                          |                                               |                                               |                                                |                                | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> | [modify]() ToC margin/padding                     |
| source/css/\\\_partial/footer.styl                  | Modified   |            |              |                          |                                               |                                               |                                                |                                | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> | [modify]() max-width: 960px                       |
| source/css/\\\_partial/header.styl                  | Modified   |            |              |                          |                                               |                                               |                                                |                                | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> | [remove]() border-bottom                          |
| source/css/\\\_partial/markdown.styl                | Modified   |            |              |                          |                                               |                                               |                                                |                                | O\\\\\\\<sup\\\\\\\>\\\\#\\\\\\\</sup\\\\\\\> | [modify]() overflow-x, font-size                  |

# 5. PR의 결과는?

![img]()(https://i.imgur.com/RV0JZhl.png)

수정한 내용을 정리한 정성 덕분이었는지, 소스의 퀄리티에도 불구하고 감사하게도 하루만에 PR이 승인되어 master 브랜치로 병합되었다. 이렇게 `뿌듯+1`을 추가하며 오늘의 딴짓도 보람차게 마무리했다.



[1](): https://github.com/mmistakes/minimal-mistakes
[2](): https://github.com/yanm1ng/hexo-theme-vexo
[3](): https://github.com/theme-next/hexo-generator-searchdb
[4](): https://velog.io/
[5](): https://github.com/yanm1ng/hexo-theme-vexo/pull/128
