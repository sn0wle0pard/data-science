---
layout: page
title: 데이터 과학
subtitle: R로 전자우편 자동 전송
---

> ## 학습 목표 {.objectives}
>
> * API를 사용해서 자동으로 전자우편을 전송한다.
> * 구글 전자우편 Gmail API를 데이터과학 R과 연동한다.

### 1. R과 전자우편 만남 [^email-jenny]

R에서 전자우편을 자동으로 보낼 수 있도록 가능하게 된 것은 최근 [gmailr](https://cran.r-project.org/web/packages/gmailr/index.html) 팩키지가 [gmailr: Access the Gmail RESTful API](https://github.com/jimhester/gmailr) 이름으로 GitHub과 CRAN에 등록되었고, 구글 전자우편 사용법을 [How to send a bunch of email from R](https://github.com/jennybc/send-email-with-r) 통해 자세한 설명이 되어 누구나 손쉽게 그 오래전 **메일 머지(여러 사람의 이름, 주소 등이 들어 있는 '데이터 파일(data file)'과 '서식 파일(form letter file)'을 결합함(merging)으로써, 이름이나 직책, 주소 부분 등만 다르고 나머지 내용이 같은 수십, 수백 통의 편지지를 한꺼번에 만드는 기능)** 같은 고급 기능을 손쉽게 구현할 수 있게 되었다.

[^email-jenny]: [How to send a bunch of email from R](https://github.com/jennybc/send-email-with-r)

또다른 강력한 사용례는 데이터 과학을 통해 분석결과를 정기적으로 자동화해서 전달하는데 있다. 결국 정기적인 주간보고, 월간보고 등 다양한 보고서가 있을 것이고 데이터가 바뀌면 기존에 작성해 놓은 R 스크립트를 실행하고 나서, 전자우편 API를 통해 전송하게 된다.

> ### 사전 준비물 {.prereq}
> 
> * Gmail 전자우편 계정
> * Jim Hester가 개발한 `gmailr` R 팩키지

### 2. 구글 전자우편 Gmail API

1. [https://console.developers.google.com/project](https://console.developers.google.com/project) 가서 `New Project`를 생성한다. 명칭은 원하는 것으로 정한다. 예를 들어 `spammer-r` 로 정한다.
1. `Gmail API`를 활성화: 먼저 `Google APIs`에서 `Gmail API`를 선택하고 나서, `Enable`을 클릭한다. API Manager &rarr; Library &rarr; `Google Apps APIs` &rarr; `Gmail API`.
1. `API Manager` 좌측 메뉴의 열쇠모양 `Credentials`를 선택한다. 
    * `Create credentials` 를 선택하고 나서 `OAuth 2.0 client IDs`를 생성시킨다.
    * 명칭은 상기 `spammer-r`과 동일하게 한다.
    * `Download JSON` 을 클릭해서 client_id, project_id, auth_uri, token_uri, auth_provider_x509_cert_url, client_secret 등이 포함된 JSON 파일을 다운로드 한다.
        * `client_secret_<영문자-숫자-조합>.apps.googleusercontent.com.json` 파일을 적절한 명칭으로 바꾸고 작업 디렉토리로 이동하여 붙여넣는다.

상기 절차를 모두 마치게 되면 구글 전자우편 서버쪽과 연결을 위한 사전 준비가 다 되었다. 

### 3. R 전자우편을 위한 준비

> ### `gmailr` R 팩키지 설치 {.prereq}
> 
> CRAN에 등록된 R 팩키지를 설치하거나, GitHub에 등록된 최신 개발 팩키지를 설치한다.
>
> * `install.packages("gmailr")`
> * `devtools::install_github("jimhester/gmailr")`

`dryrun.R` R 스크립트를 하나 작성해서 전자우편이 제대로 전송되는지 확인한다.

먼저 `library(gmailr)` 라이브러리를 적재시키고, Gmail API를 연동하기 위한 인증작업을 `use_secret_file("spammer-r.json")`을 사용해서 진행한다.
다운로드 받는 `client_secret_<영문자-숫자-조합>.apps.googleusercontent.com.json` 파일명이 아주 길어서 `spammer-r.json`로 변경했다.

그리고 나서 전송할 전자우편 본문을 작성한다. **From = ** 에는 구글 메일 전자우편주소를 사용한다.

~~~ {.r}
suppressPackageStartupMessages(library(gmailr))

use_secret_file("spammer-r.json")

first_dryrun_email <- mime(
  To = "xwmooc@webzen.com",
  From = "kcl@gmail.com",
  Subject = "Spammer List Email...",
  body = "All right... 여기에... 스패머 리스트가 있어요...")
send_message(first_dryrun_email)
~~~

처음 전자우편을 전송하게 되면 다음과 같은 화면이 뜨게 되는데, 
모두 클릭하게 되면 다음부터 웹브라우져가 뜨지 않고 `.httr-oauth`를 통해 자동으로 전자우편이 전송된다.

~~~ {.output}
> send_message(first_dryrun_email)
Use a local file ('.httr-oauth'), to cache OAuth access credentials between R sessions?

1: Yes
2: No

Selection: 
~~~

<img src="fig/spammer-r-google-auth.png" alt="구글 Gmail 인증" width="50%" />

상기 과정이 확인이 되면 다음부터 전자우편을 전송하는 명령어를 실행시키면 다음과 같이 깔끔하게 전송된다.

~~~ {.output}
> first_dryrun_email <- mime(
+   To = "xwmooc@webzen.com",
+   From = "kcl@gmail.com",
+   Subject = "Spammer List Email...",
+   body = "All right... 여기에... 스패머 리스트가 있어요...")
> send_message(first_dryrun_email)
Id: 15624cefdfsf988a
To: 
From: 
Date: 
Subject: 
~~~

### 4. 윈도우 환경에서 자동화

윈도우 환경에서 자동화에 대한 사항은 R-블로거 사이트에 게시된 글을 참조한다.

[Scheduling R Markdown Reports via Email](http://www.r-bloggers.com/scheduling-r-markdown-reports-via-email/)