# 데이터 과학



> ## 학습 목표 {.objectives}
>
> * 20년전 보고서 작성 작업흐름과 R기반 보고서 작성 자동화 과정을 비교한다.
> * 보고서 작성에 필요한 툴체인(toolchain)을 구축한다.

## 1. 보고서 작성 과정 비교

데이터를 바탕으로 보고서를 작성하는 과정은 데이터를 데이터를 분석할 수 있는 자료형으로 변환하여 SPSS/SAS/미니탭 같은 
통계팩키지가 읽어들일 수 있는 형태로 변환을 우선해야만 된다. 데이터를 불러 읽어들일 때 엑셀을 사용해서 데이터 전처리 과정을 흔히 거친다.
그리고 나서, 통계팩키지의 데이터 불러오기 혹은 가져오기(Import) 기능을 활용하여 SPSS/SAS/미니탭 파일 형태로 변환하여 저장하고 나서
데이터 마사지, 기술통계, 탐색적 자료분석 후에 회귀분석, 군집분석, 주성분분석, 요인분석 등 다양한 통계분석을 수행하고 나서
다양한 통계출력결과물과 그래프를 보고서에 붙여넣는 방식을 취한다. 최종보고서에 사용되는 소프트웨어는 아래한글이 사용되고,
발표자료를 만들 경우 파워포인트가 추가로 동원되기도 한다.

<img src="fig/ds-old-report-workflow.png" alt="기존 보고서 작성 작업흐름" width="57%" />

### 1.1. 보고서 작성 자동화

보고서를 작성할 때, 메모장, 엑셀, 통계 팩키지, 워드프로세서 혹은 슬라이드쇼 같은 여러 소프트웨어를 
독립적으로 각기 이용하는 것과 R을 활용하여 보고서를 작성하는 과정은 R을 기반으로 깔고, 보고서 작성에 필요한
여러 팩키지를 조합하여 사용하고 마지막으로 니트질(knitting)하여 워드, HTML, PDF 등 최종 보고서를 뽑아내는 과정을 갖는다.

<img src="fig/ds-new-report-workflow.png" alt="보고서 작성 자동화 작업흐름" width="57%" />

한 걸음 더 들어가면 세부적으로 다음과 같은 보고서 작성 파이프라인을 갖추게 된다.

1. `readr`: 데이터 불러오기
1. `tidyr`: 깔끔한 데이터 만들기
1. `dplyr`: 데이터 변환과 정제
1. `leaps`: 회귀분석, 변수 선택을 통한 최적모형
1. `ggplot2`: 시각화와 그래프 생성
1. `stargazer` : 출판품질 수준 고품질 표 작성
1. `rmarkdown`, `knitr`: 다양한 보고서 출력

<img src="fig/ds-report-automation.png" alt="보고서 작성 자동화 개요" width="77%" />

## 2. 보고서 자동화 사례 -- `mtcars` 연비예측 회귀분석 보고서

### 2.1. 툴체인 설치

문서화 자동화를 위한 툴체인을 구축한다. 데이터 불러와서, 전처리하고, 회귀모형 구축하고,
시각화하고 나서, 출력문서로 내보내는데 필요한 팩키지를 설치한다.



### 2.2. 데이터 불러오기

`readr` 팩키지에 포함된 기능을 활용하여 자료형을 식별하고 데이터를 불러온다.
R을 설치하게 되면 자동차 연비예측을 위한 회귀분석 및 시각화 예제로 많이 사용되는 `mtcars` 데이터가 포함되어 있다.
`read_csv` 함수를 사용하게 되면 자동으로 로컬 파일이든, 인터넷에 위치한 파일이든 `.csv` 파일이면 알아서 
불러와서 코딩이 훨씬 간결해진다.


~~~{.r}
# 2. 데이터 불러오기---------------------
data(mtcars)

spec_csv("https://gist.githubusercontent.com/seankross/a412dfbd88b3db70b74b/raw/5f23f993cd87c283ce766e7ac6b329ee7cc2e1d1/mtcars.csv")
~~~



~~~{.output}
cols(
  model = col_character(),
  mpg = col_double(),
  cyl = col_integer(),
  disp = col_double(),
  hp = col_integer(),
  drat = col_double(),
  wt = col_double(),
  qsec = col_double(),
  vs = col_integer(),
  am = col_integer(),
  gear = col_integer(),
  carb = col_integer()
)

~~~



~~~{.r}
df <- read_csv("https://gist.githubusercontent.com/seankross/a412dfbd88b3db70b74b/raw/5f23f993cd87c283ce766e7ac6b329ee7cc2e1d1/mtcars.csv", 
               col_names = TRUE, 
               cols(
                 model = col_character(),
                 mpg = col_double(),
                 cyl = col_integer(),
                 disp = col_double(),
                 hp = col_integer(),
                 drat = col_double(),
                 wt = col_double(),
                 qsec = col_double(),
                 vs = col_integer(),
                 am = col_integer(),
                 gear = col_integer(),
                 carb = col_integer()))
glimpse(df)
~~~



~~~{.output}
Observations: 32
Variables: 12
$ model <chr> "Mazda RX4", "Mazda RX4 Wag", "Datsun 710", "Hornet 4 Dr...
$ mpg   <dbl> 21.0, 21.0, 22.8, 21.4, 18.7, 18.1, 14.3, 24.4, 22.8, 19...
$ cyl   <int> 6, 6, 4, 6, 8, 6, 8, 4, 4, 6, 6, 8, 8, 8, 8, 8, 8, 4, 4,...
$ disp  <dbl> 160.0, 160.0, 108.0, 258.0, 360.0, 225.0, 360.0, 146.7, ...
$ hp    <int> 110, 110, 93, 110, 175, 105, 245, 62, 95, 123, 123, 180,...
$ drat  <dbl> 3.90, 3.90, 3.85, 3.08, 3.15, 2.76, 3.21, 3.69, 3.92, 3....
$ wt    <dbl> 2.620, 2.875, 2.320, 3.215, 3.440, 3.460, 3.570, 3.190, ...
$ qsec  <dbl> 16.46, 17.02, 18.61, 19.44, 17.02, 20.22, 15.84, 20.00, ...
$ vs    <int> 0, 0, 1, 1, 0, 1, 0, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1,...
$ am    <int> 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1,...
$ gear  <int> 4, 4, 4, 3, 3, 3, 3, 4, 4, 4, 4, 3, 3, 3, 3, 3, 3, 4, 4,...
$ carb  <int> 4, 4, 1, 1, 2, 1, 4, 2, 2, 4, 4, 3, 3, 3, 4, 4, 4, 1, 2,...

~~~


### 2.3. 탐색적 자료분석

본격적인 모형개발을 추진하기 전에 탐색적 데이터 분석을 통해 변수들간의 관계를 일별하여 친숙해지는 과정이 꼭 필요하다.

`library(GGally)` 팩키지에 내장된 `ggpairs` 함수를 통해 `mtcars` 데이터셋에 포함된 변수 관계를 짝지어 시각화한다.


~~~{.r}
line_fn <- function(data, mapping, ...){
  p <- ggplot(data = data, mapping = mapping) + 
    geom_point() + 
    geom_smooth(method=loess, fill="red", color="red", ...) +
    geom_smooth(method=lm, fill="blue", color="blue", ...)
  p
}

df_g  <-  ggpairs(df, columns = 2:12, lower = list(continuous = line_fn), warning=FALSE, message=FALSE)
df_g
~~~

<img src="fig/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

기술통계량도 [`stargazer`](https://cran.r-project.org/web/packages/stargazer/) 팩키지를 활용하면 기술통계량도 출판품질로 변환이 가능하다.


~~~{.r}
stargazer(mtcars, type="html", notes.align = "l")
~~~


<table style="text-align:center"><tr><td colspan="6" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Statistic</td><td>N</td><td>Mean</td><td>St. Dev.</td><td>Min</td><td>Max</td></tr>
<tr><td colspan="6" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">mpg</td><td>32</td><td>20.091</td><td>6.027</td><td>10.400</td><td>33.900</td></tr>
<tr><td style="text-align:left">cyl</td><td>32</td><td>6.188</td><td>1.786</td><td>4</td><td>8</td></tr>
<tr><td style="text-align:left">disp</td><td>32</td><td>230.722</td><td>123.939</td><td>71.100</td><td>472.000</td></tr>
<tr><td style="text-align:left">hp</td><td>32</td><td>146.688</td><td>68.563</td><td>52</td><td>335</td></tr>
<tr><td style="text-align:left">drat</td><td>32</td><td>3.597</td><td>0.535</td><td>2.760</td><td>4.930</td></tr>
<tr><td style="text-align:left">wt</td><td>32</td><td>3.217</td><td>0.978</td><td>1.513</td><td>5.424</td></tr>
<tr><td style="text-align:left">qsec</td><td>32</td><td>17.849</td><td>1.787</td><td>14.500</td><td>22.900</td></tr>
<tr><td style="text-align:left">vs</td><td>32</td><td>0.438</td><td>0.504</td><td>0</td><td>1</td></tr>
<tr><td style="text-align:left">am</td><td>32</td><td>0.406</td><td>0.499</td><td>0</td><td>1</td></tr>
<tr><td style="text-align:left">gear</td><td>32</td><td>3.688</td><td>0.738</td><td>3</td><td>5</td></tr>
<tr><td style="text-align:left">carb</td><td>32</td><td>2.812</td><td>1.615</td><td>1</td><td>8</td></tr>
<tr><td colspan="6" style="border-bottom: 1px solid black"></td></tr></table>

### 2.3. 최적모형 회귀모형 선정 [^best-subset-reg]

[^best-subset-reg]: [All subset regression with leaps, bestglm, glmulti, and meifly](https://rstudio-pubs-static.s3.amazonaws.com/2897_9220b21cfc0c43a396ff9abf122bb351.html)

탐색적 데이터분석을 통해 예측하려는 종속변수(`mtcars`)와 설명변수에 대한 탐색이 완료되면 회귀분석을 본격적으로 실시한다.
회귀분석을 통해 최종 예측모형을 만들어낼 때, 적절한 변수 선택이 중요한데 이유는 다음과 같다.
모형의 설명력을 높이려면 변수를 많이 사용하면 $R^2$ 값이 올라가서 극단적으로는 **1** 에 수렴하게 된다. 하지만, 이런 경우 
회귀모형의 일반성이 훼손되어 예측하려는 새로운 표본이 들어왔을 경우 간결한, 극단적으로 간단한 모형보다 예측력이 떨어지는 경우가 쉽게 관찰된다.
따라서, 모형의 복잡성이 높아지면 이에 대한 패널티를 부여하든지, $R^2$ 은 다소 희생하더라도 변수를 최소화하는 **변수 제거법** 등 다양한 방법이 개발되어 
활용되고 있다.

전진변수 선택법(forward selection)을 통해 회귀모형을 구축하는 경우, 먼저 상수항으로 적합시킨 회귀모형을 만들어 내고 `df_null`, 이를 바탕으로 
변수를 하나씩 집어 넣어 순차적으로 최적 모형을 찾아나간다. 후방변수 제거법(backward elimination)은 꽉찬 모형, `df_full`에서 변수를 순차적으로 하나씩
제거하면 최적 모형을 찾아내는 것이고, 양방향 `direction=both`인 경우 상황에 따라 변수를 추가 제거하는 과정을 반복하면서 최적 모형을 찾아낸다.


~~~{.r}
# 4. 회귀분석---------------------
# 단계별 회귀분석: 전진선택법, 양방향 선택/제거법
df_null <- lm(mpg ~1, data=df[,-1])
df_full <- lm(mpg ~., data=df[,-1])

df_forward <- step(df_null, scope=list(lower=df_null, upper=df_full), direction="forward", trace=FALSE)
df_backward <- step(df_null, scope=list(lower=df_null, upper=df_full), direction="both")
~~~



~~~{.output}
Start:  AIC=115.94
mpg ~ 1

       Df Sum of Sq     RSS     AIC
+ wt    1    847.73  278.32  73.217
+ cyl   1    817.71  308.33  76.494
+ disp  1    808.89  317.16  77.397
+ hp    1    678.37  447.67  88.427
+ drat  1    522.48  603.57  97.988
+ vs    1    496.53  629.52  99.335
+ am    1    405.15  720.90 103.672
+ carb  1    341.78  784.27 106.369
+ gear  1    259.75  866.30 109.552
+ qsec  1    197.39  928.66 111.776
<none>              1126.05 115.943

Step:  AIC=73.22
mpg ~ wt

       Df Sum of Sq     RSS     AIC
+ cyl   1     87.15  191.17  63.198
+ hp    1     83.27  195.05  63.840
+ qsec  1     82.86  195.46  63.908
+ vs    1     54.23  224.09  68.283
+ carb  1     44.60  233.72  69.628
+ disp  1     31.64  246.68  71.356
<none>               278.32  73.217
+ drat  1      9.08  269.24  74.156
+ gear  1      1.14  277.19  75.086
+ am    1      0.00  278.32  75.217
- wt    1    847.73 1126.05 115.943

Step:  AIC=63.2
mpg ~ wt + cyl

       Df Sum of Sq    RSS    AIC
+ hp    1    14.551 176.62 62.665
+ carb  1    13.772 177.40 62.805
<none>              191.17 63.198
+ qsec  1    10.567 180.60 63.378
+ gear  1     3.028 188.14 64.687
+ disp  1     2.680 188.49 64.746
+ vs    1     0.706 190.47 65.080
+ am    1     0.125 191.05 65.177
+ drat  1     0.001 191.17 65.198
- cyl   1    87.150 278.32 73.217
- wt    1   117.162 308.33 76.494

Step:  AIC=62.66
mpg ~ wt + cyl + hp

       Df Sum of Sq    RSS    AIC
<none>              176.62 62.665
- hp    1    14.551 191.17 63.198
+ am    1     6.623 170.00 63.442
+ disp  1     6.176 170.44 63.526
- cyl   1    18.427 195.05 63.840
+ carb  1     2.519 174.10 64.205
+ drat  1     2.245 174.38 64.255
+ qsec  1     1.401 175.22 64.410
+ gear  1     0.856 175.76 64.509
+ vs    1     0.060 176.56 64.654
- wt    1   115.354 291.98 76.750

~~~

최상부분집합선택법(Best Subset) 회귀모형 구축방법은 `leaps` 팩키지 `regsubsets` 함수를 사용하면 최대 10개까지 변수를 
조합하여 최상부분집합을 선택한다. 전진변수 선택 혹은 후방변수 제거법과 달리 회귀모형의 복잡성에 대해 패널티를 가하는 방법으로 
AIC/BIC 등이 사용되는데 AIC가 다소 변수를 과대선택하는 경향이 알려져있어 BIC 기준으로 최소가 되는 모형을 최적모형으로 선정한다.

이를 위해서 `regsubsets` 함수로 최상부분집합 적합을 시킨 결과에서 BIC 값을 뽑아내고 그중 `min_bic`에 최소값을 저장시킨다.
그리고 변수가 몇개일 때 최소가 되는지 


~~~{.r}
# 최상부분집합선택법
# noquote(paste0(names(mtcars), "+"))
subset_mod <- regsubsets(mpg ~ ., data = mtcars, nvmax=NULL, 
                         force.in = NULL, force.out = NULL, 
                         method="exhaustive")

subset_mod_out <- summary(subset_mod)

min_bic <- which.min(subset_mod_out$bic)

plot(subset_mod_out$bic, ylab = "BIC", type="b")
points(min_bic, subset_mod_out$bic[min_bic], cex=2, col="blue", pch=20)
ibest <- seq(along = subset_mod_out$bic)[subset_mod_out$bic == min_bic]

foo <- subset_mod_out$which[ibest, ]
form <- names(foo)[foo][-1]
form <- paste(form, collapse = " + ")
form <- paste("mpg ~", form)

## 최적 모형
mtcars_best <- lm(as.formula(form), data=df)

summary(mtcars_best)
~~~

### 2.4. 회귀식 표현 [^convert-lm-objects]

[^convert-lm-objects]: [function to convert lm model to LaTeX equation](https://stat.ethz.ch/pipermail/r-help/2009-October/408317.html)

최적의 회귀식을 구축했으면 이제 남은 단계는 이를 문서화하는 것이다.


~~~{.r}
# 5. 회귀식 표현---------------------
latex_lm <- function(object, file="", math.env=c("$","$"),
                     estimates="none", abbreviate = TRUE, abbrev.length=8, digits=3) {
  # Get and format IV names
  co <- c("Int", names(object$coefficients)[-1])
  co.n <-  gsub("p.*)", "", co)
  if(abbreviate == TRUE) {
    co.n <- abbreviate(gsub("p.*)", "", co), minlength=abbrev.length)
  }
  # Get and format DV
  m.y <- strsplit((as.character(object$call[2])), " ~ ")[[1]][1]
  # Write coefficent labels
  b.x <- paste("\\beta_{", co.n ,"}", sep="")
  # Write error term
  e <- "\\epsilon_i"
  # Format coefficint x variable terms
  m.x <- sub("}Int","}", paste(b.x, co.n, " + ", sep="", collapse=""))
  # If inline estimates convert coefficient labels to values
  if(estimates == "inline") {
    m.x <- sub("Int", "",
               paste(round(object$coefficients,digits=digits), co.n, " + ", sep="",
                     collapse=""))
    m.x <- gsub("\\+ \\-", "-", m.x)
  }
  # Format regression equation
  eqn <- gsub(":", " \\\\\\times ", paste(math.env[1], m.y, " = ",
                                          m.x, e, sep=""))
  # Write the opening math mode tag and the model
  cat(eqn, file=file)
  # If separae estimates format estimates and write them below the model
  if(estimates == "separate") {
    est <- gsub(":", " \\\\\\times ", paste(b.x, " = ",
                                            round(object$coefficients, digits=digits), ", ", sep="", collapse=""))
    cat(", \\\\ \n \\text{where }", substr(est, 1, (nchar(est)-2)), file=file)
  }
  # Write the closing math mode tag
  cat(math.env[2], "\n", file=file)
}

# 회귀식
mtcars_best <- lm(mpg~wt+qsec+am, data=df[,-1])

latex_lm(mtcars_best)
~~~

$mpg = \beta_{Int} + \beta_{wt}wt + \beta_{qsec}qsec + \beta_{am}am + \epsilon_i$ 


~~~{.r}
# 최종회귀모형 상세
stargazer(mtcars_best, df_forward, df_backward, type="html", notes.align = "l")
~~~


<table style="text-align:center"><tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="3"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="3" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="3">mpg</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td><td>(3)</td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">wt</td><td>-3.917<sup>***</sup></td><td>-3.167<sup>***</sup></td><td>-3.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.711)</td><td>(0.741)</td><td>(0.741)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">qsec</td><td>1.226<sup>***</sup></td><td></td><td></td></tr>
<tr><td style="text-align:left"></td><td>(0.289)</td><td></td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">am</td><td>2.936<sup>**</sup></td><td></td><td></td></tr>
<tr><td style="text-align:left"></td><td>(1.411)</td><td></td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">cyl</td><td></td><td>-0.942<sup>*</sup></td><td>-0.942<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.551)</td><td>(0.551)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">hp</td><td></td><td>-0.018</td><td>-0.018</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(0.012)</td><td>(0.012)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>9.618</td><td>38.752<sup>***</sup></td><td>38.752<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(6.960)</td><td>(1.787)</td><td>(1.787)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>32</td><td>32</td><td>32</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.850</td><td>0.843</td><td>0.843</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.834</td><td>0.826</td><td>0.826</td></tr>
<tr><td style="text-align:left">Residual Std. Error (df = 28)</td><td>2.459</td><td>2.512</td><td>2.512</td></tr>
<tr><td style="text-align:left">F Statistic (df = 3; 28)</td><td>52.750<sup>***</sup></td><td>50.171<sup>***</sup></td><td>50.171<sup>***</sup></td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="3" style="text-align:left"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

이를 위해 다음과 같이 데이터에서 최상부분집합선택법에서 바로 최적 회귀모형을 생성시키도록 `latex_lm` 함수를 작성하고 이를 문서와 함께 
자동화한다. 그리고 나서 `stargazer` 함수를 통해 최적 모형에 대한 회귀식 문서를 완성시키고 모형에 대한 간략한 설명을 붙여 회귀분석 보고서를 마무리한다.
 
