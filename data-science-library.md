---
layout: page
title: 데이터 과학
subtitle: R 팩키지와 라이브러리
output:
  html_document: 
    keep_md: yes
  pdf_document:
    latex_engine: xelatex
mainfont: NanumGothic
---




> ## 학습 목표 {.objectives}
>
> * R 팩키지와 라이브러리를 이해한다.

### 라이브러리 설정 

`.libPaths()` 명령어로 현재 팩키지가 설치된 디렉토리를 확인할 수 있다.


~~~{.r}
.libPaths()
~~~



~~~{.output}
FALSE [1] "/Library/Frameworks/R.framework/Versions/3.2/Resources/library"

~~~

`[1] "C:/Users/KwangChun/Documents/R/win-library/3.2"` 디렉토리는 
사용자가 개별적으로 설치한 팩키지 목록이 나와 있고,
`[2] "C:/Program Files/R/R-3.2.3/library"` 디렉토리에는 
기본 팩키지와 유틸리티가 나타나 있다.


`.Rprofile` 파일에 `.libPaths("C:/Users/xwMOOC/Rpackages")`를 추가해서 팩키지를 한 곳에 몰아 관리할 수도 있다. 



### 참고자료
