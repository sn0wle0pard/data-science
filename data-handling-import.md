---
title: "데이터 과학"
output:
  html_document: 
    keep_md: yes
  pdf_document:
    latex_engine: xelatex
subtitle: 데이터 다루기
layout: page
mainfont: NanumGothic

---



> ## 학습 목표 {.objectives}
>
> * R로 다양한 원천자료를 가져온다.
> * 로컬디스크, 데이터베이스, 웹 인터넷 데이터 출처를 이해한다.
> * 활성도가 높은 R 팩키지를 취사선택하여 데이터 가져오기 프로세스를 정형화한다. 



<img src="fig/rstudio-data-import.png" alt="RStuio 데이터 가져오기" width="50%" />


### 공공데이터 출처

공공데이터는 과거 로컬디스크에서 `. csv`, `.xlsx`, SAS, SPSS, 미니탭 등 형식 파일로 제공되었으나, SQL 데이터베이스에
ODBC, JDBC, DBI 등 인터페이스를 통해 직접 접근하여 데이터를 뽑아올 수도 있고,
인터넷 웹을 통해 XML, JSON 등의 형식으로 받아올 수도 있다.


> ### 공공데이터 출처 형식 {.callout}
> 
> - 로컬 디스크: `. csv`, `.xlsx`, SAS, SPSS, 미니탭
> - 데이터베이스: SQL
> - 웹 인터넷: XML, JSON

#### 1. 로컬 데이터

전통적으로 많이 활용하는 방식으로 `. csv`, `.xlsx`, SAS, SPSS, 미니탭 등 다양한 파일 형식에 담긴 정보를 R에서 `readr`, `readxl`, `haven`
팩키지를 활용하여 R에서 분석할 수 있는 R 데이터프레임으로 불러온다. 


> ### 로컬 디스크 파일 R로 가져오기(Ingest) {.callout}
> 
> - `readr`: `. csv`, `.txt`, 등 텍스트 파일
> - `readxl`:  `.xls`, `.xlsx` 엑셀 파일
> - `haven`: `sas7bdat`, `.sav` SAS, SPSS 파일

#### 2. 데이터베이스(SQL)

데이터베이스에서 R로 데이터를 불러오는 방식은 기본적으로 `DBI`, `RODBC` 등 팩키지를 불러와서 적재한다.
목표 데이터베이스에 연결하고, SQL 쿼리를 데이터베이스에 던져 결과를 얻어와서 데이터프레임에 저장한다.
데이터 정합성 검사를 하고 모든 것이 정상적이면 데이터베이스 연결을 끊어 자원을 반납한다.

##### 데이터베이스 연결 작업흐름


~~~{.r}
# 1) DBI 팩키지 불러오기
library(DBI)
# 2) 특정 데이터베이스 연결
db <- dbConnect(RPostgres::Postgres(), user, pass, ...)
db <- dbConnect(RMySQL::MySQL(), user, pass, ...)
db <- dbConnect(RSQLite::SQLite(), path)
# 3) SQL 쿼리
dbGetQuery(db, "SELECT * FROM mtcars")
# 4) 연결 끊기
dbDisconnect(db)
~~~

##### rstats-db

GitHub에 [rstats-db](http://github.com/rstats-db/) 저장소에는 *MySQL*, *Postgres*, *SQLite* 등
다양한 데이터베이스에 연결해서 R로 데이터를 가져오는 팩키지가 개발되고 있다.
`library` 함수보다 `devtools` 함수를 사용해서 연관된 팩키지를 설치하여 데이터를 가져올 필요가 있다.



~~~{.r}
# http://github.com/rstats-db/
devtools::install_github("rstats-db/DBI")
devtools::install_github("rstats-db/RPostgres")
devtools::install_github("rstats-db/RMySQL")
devtools::install_github("rstats-db/RSQLite")
~~~


#### 3. 웹 인터넷 데이터

최근에 많은 공공데이터가 웹인터넷을 통해 배포되고 있다. XML, JSON이 많이 사용되고 있다.
특히, XML, JSON은 API를 통해 제공되는 정형화된 데이터로 통용되며, 그렇지 않고 데이터가 제공되는 경우
HTML 페이지를 웹스크래핑해서 데이터를 퍼온다.

XML `libxml2`에 대응되는 `xml2` 팩키지를 사용하고, JSON의 경우, `jsonlite` 팩키지를 사용한다.
API를 통해 데이터가 제공되지 않는 경우, `rvest` 팩키지를 통해 HTML 페이지 데이터를 긁어온다.

> ### 웹 인터넷 R 팩키지 {.callout}
> 
> - XML : `library(xml2)`, 활성도가 높지 않는 `rjson`, `RJSONIO`는 권장하지 않는다.
> - JSON: `library(jsonlite)`, 과거 `XML` 팩키지에 대한 대안
> - HTML: `library(rvest)`, `rvest`는 파이썬 뷰티풀숩(Beautiful Soup)에 해당


### 참고자료

- [Getting your data into R](https://www.rstudio.com/resources/webinars/getting-your-data-into-r/)