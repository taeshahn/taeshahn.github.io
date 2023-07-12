---
title: "나를 위해 기록하는 Jupyter - HTML/PDF 변환 프로세스"
date: 2021-02-06
categories:
  - Others
tags:
  - Others
toc: true
---

노트북(.ipynb) 파일은 물론 [Reproducible Research][1]의 개념에 매우 잘 어울리는 포멧이다. 그러나 우리는 블로그 작성을 위해서라거나, Reproducibility에는 전혀 관심이 없을 분들을 위한 최종 보고서 작성을 위해서라거나, 파일 수신자가 노트북 파일을 실행할 수 없는 환경에 있다는 등의 다양한 이유로 노트북 파일을 보다 일반적인 형태의 문서로 변환해야하는 상황과 마주친다.

편리하게도 Jupyter Lab은 다양한 형태로 문서를 내보내는 기능을 제공한다. 노트북 파일을 HTML이나 LaTeX 포멧으로 내보낼수도 있고, 내부적으로 pandoc을 이용하여 pdf로 변환하는 기능도 제공하고 있다. 이러한 과정에서 사용자가 포멧을 직접 설정할 할 수 있는 기능들이 일부 존재하기도 하지만, 아무래도 변환 도구들을 직접 사용하는 것이 아니라 Jupyter를 통해서 접근하는 것이기 때문에 상대적으로 번거롭고 자유도도 떨어진다. 이를 테면 지정된 위치에 custom.css 파일을 작성해놓으면 그 파일을 이용하여 HTML로 내보내준다거나, Pandoc 관련된 옵션을 수동으로 수정할 수 있다거나 하는 식이다.

몇 가지 방법을 시도해봤으나 뭔가 조금씩 마음에 안들었던 나는, 결국 Jupyter Lab에서 마크다운으로 컨텐츠를 내보낸 후, 마크다운을 편집할 수 있는 에디터를 이용하여 내가 원하는 레이아웃으로 최종 변환하는 방법을 선택했다. 이 글에는 나의 삽질의 과정을 기록하고자 한다.

## 1. Notebook to Markdown

노트북 파일을 마크다운으로 변환하는 방법은 매우 간단하다. Jupyter Lab에서 노트북 파일을 오픈한 후에, `File > Export Notebook As ... > Export Notebook As Markdown` 버튼을 차례로 누르면 Markdown 파일과 실행 셀의 결과로 출력된 이미지 파일들이 모두 포함된 압축 파일의 다운로드가 시작된다.

## 2. Markdown Editor and Preview Configuration

이 글에서는 마크다운 에디터로 [Visual Studio Code][2]를 이용할 것이다. Code에는 기본적으로 마크다운 프리뷰 기능이 포함되어 있으며, 프리뷰 렌더링에 사용하는 별도의 css 파일이 존재한다. 해당 파일을 수정하면 Markdown Preview 화면의 출력 결과를 원하는대로 수정할 수 있는데, 여기에는 폰트나 줄간격, (데이터프레임의 출력 결과를 포함하는) 테이블의 표현 방식, 이미지의 정렬 방식 등이 모두 포함된다.

Windows 버전 1.45.1 기준으로 렌더링에 사용하는 css 파일의 기본 경로는 다음과 같다.
> C:\Users\\[Username]\AppData\Local\Programs\Microsoft VS Code\resources\app\extensions\markdown-language-features\media\markdown.css

이 파일을 어떤 식으로 수정할 것인지에 대해서는 [4. GitHub Style CSS][3]를 참조하자. 

## 3. Extension Configuration

다음으로는 마크다운을 HTML과 PDF로 내보내는 기능을 위해 [Markdown PDF](https://github.com/yzane/vscode-markdown-pdf)라는 Extension을 추가로 설치하여 이용할 것이다. 물론 이 Extension에도 Code의 프리뷰 렌더링 css 파일처럼 HTML과 PDF 파일 생성에 이용하는 CSS 파일들이 존재한다. 각각의 CSS 파일의 경로는 다음과 같다.

- Markdown to HTML
    > C:\Users\\[Username]\\.vscode\extensions\yzane.markdown-pdf-1.4.4\styles\markdown.css

- Markdown to PDF
    > C:\Users\\[Username]\\.vscode\extensions\yzane.markdown-pdf-1.4.4\styles\markdown-pdf.css

지금까지 언급한 세 개의 파일을 모두 동일하게 설정할 경우, Code에서 프리뷰로 보는 문서와 동일한 레이아웃의 HTML과 PDF를 외부로 내보낼 수 있다.

## 4. GitHub Style CSS
앞서 언급한 3개의 CSS 파일들을 어떻게 수정할 지는 개인 취향이다. 미적 감각 같은 건 진작에 포기한 나는 GitHub Style Markdown CSS를 가져다가 취향에 맞게 조금만 수정해서 쓰기로 했다. 잠깐 검색을 해보니 두 개 정도를 찾을 수 있었는데, 나는 그 중 하나를 기반으로 다음과 같은 사항들을 추가적으로 수정하여 이용하고 있다. 개인적으로 수정한 css 파일은 [이 페이지][4]에서 확인 및 다운받을 수 있다.

- 기억나는 수정 사항
    - 한글 폰트 추가 설정
    - 코드 영역 폰트 기본 색상 설정
    - 테이블 내부 폰트 사이즈 조정
    - 테이블 전체적인 레이아웃 조정
    - H1 ~ H6 BOLD 해제
    - H1 border-bottom: 1px solid #333;
    - table border-collapse: collapse;

내가 수정하기 이전의 원본 css파일은 [여기][5]와 [저기][6]에서 확인할 수 있다.

## 5. With VS Code Setting

3, 4번 항목에서 각 CSS 파일의 위치를 명시하고 해당 파일을 덮어쓰는 방식으로 진행했지만, 사실 Code와 Extension은 설정 화면에서 custom css 파일을 지정할 수 있는 옵션을 제공하고 있다. 내가 설정을 제대로 못한 것인지, 아니면 Code에 버그가 있는 것인지 상대경로가 아닌 절대경로로 입력하면 제대로 설정이 안되는 상태이긴 하지만...

그래도 참고용으로 설정 화면에서 CSS 파일을 지정할 수 있는 Setting ID를 기록해 놓는다.

- Setting ID 및 Json (설정 화면에서 'markdown styles'로 검색 시 두 옵션 모두 조회 가능)
    - "markdown.styles": ["TaesHahn.css"]
    - "markdown-pdf.styles": ["TaesHahn.css"]

추가로, 설정 화면에서 `Markdown-pdf: Include Default Styles` 옵션은 체크 해제하는 것을 권한다. 아래 경로에 있는 파일이 우리가 직접 지정한 CSS 파일과 중복 적용되는 것을 방지하기 위한 설정이다.

* Setting ID 및 Json
    * "markdown-pdf.includeDefaultStyles": false
    > C:\Users\\[Username]\\.vscode\extensions\yzane.markdown-pdf-1.4.4\node_modules\jsdom\lib\jsdom\browser\default-stylesheet.js # 미사용 설정

## At the end ...
이것으로 모든 설정을 마무리하였다. 프로그래밍을 하거나 분석 코드를 작성하는 것만큼, 작성한 글이 읽는 이에게 어떤 형태로 전달되는 지는 매우 중요하다. 이 글이 누군가에게는 도움이 되길 바라면서 글을 마친다.

Happy Coding! Happy Editing!

## References
[Visual Studio Code 에서 깃헙 스타일 마크다운 사용하기][7]

[1]: https://en.wikipedia.org/wiki/Reproducibility
[2]: https://code.visualstudio.com/
[3]: #4-github-style-css
[4]: https://github.com/tshahn/
[5]: https://github.com/nicolashery/markdownpad-github
[6]: https://gist.github.com/andyferra/2554919
[7]: https://blog.aliencube.org/ko/2016/07/06/markdown-in-visual-studio-code/