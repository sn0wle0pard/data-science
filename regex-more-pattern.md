---
layout: page
title: 데이터 과학
subtitle: 추가 패턴
---

### 추가 패턴

정규표현식 동작원리를 알게 되어서, Notebook #3을 살펴보자:

~~~ {.output}
    Date Site Evil(mvad)
    May 29 2010 (Hartnell) 1029.3
    May 30 2010 (Hartnell) 1119.2
    June 1 2010 (Hartnell) 1319.4
    May 29 2010 (Troughton) 1419.3
    May 30 2010 (Troughton) 1420.0
    June 1 2010 (Troughton) 1419.8
    ⋮            ⋮           ⋮
~~~

상기 데이터는 날짜가 필드 세개로 구성되어 있고, 괄호에 장소명이 들어있고,
그리고 나서 측정값이 있다.
이런 형식을 갖는 날짜를 파싱하는 방법을 알고 있다.
필드는 공백으로 구분된다. 하지만, 괄호를 어떻게 매칭할 수 있을까?
지금까지 정규표현식에서 살펴본 괄호는 문자를 매칭하지 못한다: 괄호로 그룹집단을 생성했다.

이 문제를 해결하는 방식--즉, 문자 그대로 여는 괄호 '(' 혹은 닫는 괄호 ')'를 매칭하는 방법--은 괄호 앞에 역슬래쉬를 넣는 것이다. 이것이 [확장 비트열(escape sequence)](http://terms.naver.com/entry.nhn?docId=858903&cid=42346&categoryId=42346)의 또다른 사례다: 문자 그대로 탭문자를 표현하는데,
문자열에 `\t` 문자2개 시퀀스를 사용했듯이, 정규표현식에서 문자그대로 문자 '(' , ')'을 매칭하는데 문자2개 시퀀스로 `'\('` , `'\)'` 을 사용한다.

하지만, 문자열에서 역슬래쉬 '\\'을 추출하려면 이중으로 해서 *역슬래쉬* 를 빼내온다.
이것은 정규표현시과 아무 관련이 없다: 문자열에 역슬래쉬를 넣는 파이썬 규칙이다.
따라서 여는 괄호를 매칭하는 정규표현식 문자열 표현은 `'\\('`이 된다. 
혼동스러울 수 있다. 그래서 연관된 다양한 계층을 살펴보자.

프로그램 텍스트--즉, `.py` 파일로 저장된 것--는 다음과 같다:

~~~ {.python}
# find '()' in text
m = re.search('\\(\\)', text)
    ⋮    ⋮    ⋮
~~~

파일에 인용부호 내부에 역슬래쉬 두개, 여는 괄호, 역슬래쉬 두개, 닫는 괄호가 있다.
파이썬에서 상기 파일을 불러 읽어올 때, 문자2개 시퀀스 '\\\\'은 메모리 문자열에 단일 문자 
'\\'이 된다. 이것이 처음 빼내기한 결과가 된다.
'\\(\\)' 이 문자열을 정규표현식 라이브러리에 넘겨줄 때, 
정규표현식 라이브러리는 문자2개 시퀀스 '\\('을 건네받아서 문자 그대로 괄호를 매칭하는
유한상태기계에 화살표에 넣는다.

<img src="fig/regex-fsm-match-parentheses.png" alt="" width="50%" />

다른 말로 표현하면, 문자 그대로 괄호를 매칭하려면, 정규표현식 라이브러리에 '\\('을 전달해야만 된다.
문자열로 '\\('을 넣고자 한다면, `.py` 파일에 '\\\\('처럼 작성해야만 된다.

이를 잠시 옆으로 밀어 넣고서, Notebook #3 사례로 되돌아 가자.
각 레코드에서 필드 5개를 뽑아내는 정규표현식은 
`'([A-Z][a-z]+) ([0-9]{1,2}) ([0-9]{4}) \\((.+)\\) (.+)'` 이다.
이를 풀어보면 다음과 같다.

- 대문자 문자 하나 다음에 하나 혹은 그이상 소문자로 시작하는 단어
- 공백
- 1자리 혹은 2자리 숫자
- 4자리 숫자
- 또다른 공백 
- 역슬래쉬와 괄호와 관련된 패턴
- 또다른 공백
- 하나 혹은 그이상 문자로 구성된 측정값

괄호와 연관된 부분을 좀더 살펴보면, `'\\('` , `'\\)'` 이 데이터에 
존재하는 문자 그대로 여는 괄호 '(' , 닫는 괄호 ')' 문자를 매칭하는 정규표현식을 작성하는 방법이다.
앞에 역슬래쉬를 갖지 않는 두번째 내부 괄호는 그룹집단을 생성하지만, 어떤 문자 자체로 문자를 매칭하지는 않는다.
그룹을 생성해서 매칭되는 결과를 저장할 수 있다(이번 경우에, 장소명).

정규표현식에서 역슬래쉬로 작업하는 방식을 학습했기 때문에,
그자체로 축약되어 사용될 정도로 자주 활용되는 문자집합을 살펴보자.
정규표현식에서 `\d` 를 사용하면, 0에서 9까지 숫자를 매칭한다.
`\s` 를 사용하면, 화이트스페이스 문자, 공백, 탭, 개행, 개행복귀를 매칭한다.
`'\w'` 는 단어 문자를 매칭한다; 대문자, 소문자, 숫자, 밑줄에 대한 
`'[A-Za-z0-9_]'` 패턴집합과 동치다.
(사실 C 혹은 파이썬 같은 프로그래밍 언어 변수명에 나올 수 있는 문자 집합이다.)
다시, 파이썬 문자열로 정규표현식을 작성하려면, 이중 역슬래쉬를 사용해야 된다.


