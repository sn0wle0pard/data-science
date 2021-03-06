---
layout: page
title: 데이터 과학
subtitle: AWS RStudio 서버
output:
  html_document: 
    keep_md: yes
  pdf_document:
    latex_engine: xelatex
mainfont: NanumGothic
---


> ## 학습 목표 {.objectives}
>
> * AWS 데이터과학 서버를 생성한다.
> * AWS RStudio 서버를 준비하여 데이터 분석을 클라우드 환경에서 병렬처리한다.



## 1. AWS 클라우드 데이터 과학 서버 [^aws-rstudio-part-1] [^aws-rstudio-part-2]

[^aws-rstudio-part-1]: [RStudio in the Cloud I - Amazon Web Services](http://strimas.com/r/rstudio-cloud-1/)
[^aws-rstudio-part-2]: [RStudio in the Cloud II - Syncing Code & Data with AWS](http://strimas.com/r/rstudio-cloud-2/)


[아마존 웹서비스(Amazon Web Services, AWS)](http://aws.amazon.com/)는 다양한 컴퓨팅 자원을 클라우드 인터넷을 통해 제공하고 있다.
데이터 과학을 위해 필요한 컴퓨팅 자원을 사용한만큼 비용을 지불하고 서비스를 받을 수 있다.
고가 PC 혹은 노트북을 사용해서 초기 부담을 많이 부담하고 나서 거의 전기료만 지불하고 데이터 과학을 수행하던 것과 비교하여 
꼭 필요한 부분을 정확히 이해하지 않고 AWS 클라우드 서비스를 사용하는 경우 낭패를 볼 수도 있으니, 사전에 면밀히 혹은 충분한 사전 학습을 하고서
활용하는 것을 권장한다. 

### 1.1. 왜 AWS 클라우드 환경을 사용해야 하나.

AWS 혹은 IBM 소프트레이어, MS 애져 클라우드 서비스, 구글 클라우드를 사용해서도 대동소이하다. 즉, 클라우드에 RStudio를 올려 사용할 경우 다음과 같은 
좋은 점을 기대해도 좋다.

1. **장소에 구애받지 않고 데이터 과학 작업 수행:** 인터넷에 연결만 된다고 하면 AWS 클라우드에 가상 컴퓨터에 RStudio 서버를 설치하여 언제든 장소에 구애받지 않고
자유로이 데이터 과학 분석작업을 수행할 수 있다.
1. **컴퓨팅 자원:** 데이터 과학을 수행할 경우 컴퓨팅 자원을 특정 모형, 예를 들어 `Random Forest`를 규모가 있는 데이터에 적용할 경우 상당한 컴퓨팅 자원이 소요된다.
이런 경우 멀티코어 EC2 가상 컴퓨터를 주문하여 사용하고 나서 컴퓨터를 종료시키게 되면 사용한 시간만큼만 비용을 지불하게 된다. 이런 경우 두가지 장점이 있는데,
아주 고성능 고가의 컴퓨터를 사용한 측면과 더불어, 더이상 무거운 하드웨어를 관리하지 않아도 된다는 편리성이 있고, 특히 아주 저렴하고 편리하게 대여할 수 있다는 점이 
상당한 매력이다. 경우에 따라서는 빅데이터를 빠르게 병렬, 분산처리를 수행하는 경우 다른 유형의 컴퓨터 필요할 수 있다. 따라서 필요에 따라 작업성격에 맞추어 
컴퓨팅 자원을 잘 활용할 수 있다는 점이 클라우드에 RStudio를 올려 사용하는 매력이다.

## 2. EC2 인스턴스(가상 컴퓨터) 설치

AWS 클라우드에 EC2 가상컴퓨터를 생성하는 과정은 다음과 같이 3단계로 구분된다.

- [아마존 웹서비스(Amazon Web Services, AWS)](http://aws.amazon.com/) 계정 생성과 신용카드 등록.
- 사전 준비과정으로 가상컴퓨터를 배포할 지역(region) 선정, 보안그룹 생성, SSH 인증키 생성.
- EC2 가상컴퓨터 사양선정: 가상컴퓨터 하드웨어 사양 선정, 운영체제와 데이터과학 소프트웨어는 AMI RStudio 이미지로 대체.
- 컴퓨터 접속: 직접 `ssh` 로그인 혹은 웹브라우져 RStudio 서버 접속.

<img src="fig/aws-ec2-rstudio-ami-creation.png" alt="AWS EC2 생성과정" width="77%" />

### 2.1. 지역(Region) 선택.

[AWS 콘솔](http://console.aws.amazon.com)에서 로그인한 후에 **Compute** 아래 **EC2** 아이콘을 클릭한다.
가상컴퓨터를 설치할 지역(Region)을 선정한다. 지역은 아마존 실제 컴퓨터 클러스터가 위치한 지역으로 
어느 지역이나 가능하지만 물리적인 한계를 고려하여 실제 가상컴퓨터를 실행시킬 수 있는 지역을 선정한다.
한국에서는 **Asia Pacific (Seoul)**을 선정한다.

<img src="fig/aws-region-dropdown.png" alt="AWS 지역 설정" width="57%" />

### 2.2. 보안 그룹(Security Group) 설정

[보안 그룹(Security Group)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)은 
**가상 방화벽(Virtual Firewalls)** 역할을 하여 AWS 인스턴스(가상컴퓨터)에 대한 원격제어를 수행한다.
가상 컴퓨터에 접근이 허용된 IP 대역과 가능한 서비스(SSH, HTTP) 등을 명세한다.
데이터 과학을 위해서는 가상컴퓨터에 SSH 접근과 RStudio 서버를 위한 HTTP 접근만 허용되면 된다.

1. Type: **SSH**, Source: **Anywhere** 를 설정한다.
1. Type: **HTTP**, Source: **Anywhere** 를 설정한다.

주의할 점은 `Source`에 임의 IP 주소 범위를 허용했다. 만약 보안에 좀더 신경을 쓴다면 특정 IP 주소로 접속을 제한하는 것도 가능하다.

<img src="fig/aws-security-group.png" alt="AWS 보안그룹 설정" width="77%" />

### 2.3. SSH 인증키

AWS 가상컴퓨터에 접속하는데 비밀번호 대신에 [인증키(Key pair)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html?console_help=true)를 
사용해서 로그인해야만 된다. **인증키(Key pair)**는 공개키(public key)와 비밀키(private key)로 구성되는데 공개키로 가상컴퓨터를 걸어잠구고,
비밀키를 통해 가상컴퓨터를 풀어 로그인하는 구조를 갖게 된다. 따라서, 비밀키를 잃어버리게 되는 경우 가상컴퓨터에 로그인을 할 수 없게 되고,
비밀키를 다른 누군가 갖게 되면 비밀키를 갖는 누구나 가상컴퓨터에 접속할 수 있는 권한을 갖게 된다.

<img src="fig/aws-key-pair.png" alt="AWS 인증키" width="57%" />

EC2 가상컴퓨터에 대한 인증키를 생성시키려면, 좌측 패널에 **Key Pairs**를 선택하고 나서, **Create key Pair** 버튼을 생성한다.
`.pem` 확장자를 갖는 파일을 다운로드하게 되는데 이것이 가상컴퓨터에 걸린 자물쇠(공개키)를 풀 수 있는 열쇠(개인키)가 된다. 
따라서 이 개인키를 안전한 곳에 잘 보관해야만 된다.

### 2.4. 가상컴퓨터(인스턴스) 생성 

EC2 가상컴퓨터를 생성하려면, 좌측 패널에 **Instance** 버튼을 클릭하고 나서, **Launch Instance** 버튼을 클릭한다.
용산이나 인터넷 쇼핑몰에서 컴퓨터를 주문할 경우 하드웨어 사양과 더불어 운영체제, 오피스 등 소프트웨어를 선정해야 한다.
데이터 과학에서 하드웨어 못지않게 중요한 것이 소프트웨어라 **아마존 컴퓨터 이미지(Amazon Machine Images, AMIs)**를 
사용하여 처리하면 별도 번거로운 설정없이도 바로 데이터과학을 위한 가상컴퓨터를 손에 넣을 수 있다.

#### 2.4.1. 아마존 컴퓨터 이미지(Amazon Machine Images, AMIs)

아마존 컴퓨터 이미지(Amazon Machine Images, AMIs)를 사용하면 운영체제부터 특정 컴퓨터 작업에 필요한 모든 소프트웨어가
잘 설치되어 있고, 환경까지 맞춰져 있어 컴퓨터가 망가지더라도 다시 추가 작업없이 활용가능한 장점이 있다.

 [Louis Aslett](http://www.louisaslett.com/RStudio_AMI/)님이 RStudio AMI를 잘 만들어 주셨다. 
 이를 활용하여 우분투부터 RStudio까지 한방에 설치가 가능하다. **Community AMIs** &rarr; `rstudio aslett`을 검색하여 이를 선택한다.

 #### 2.4.2. 가상컴퓨터(인스턴스) 사양선정

 가상 컴퓨터지만 실제 컴퓨터와 마찬가지로 하드웨어 사양을 선정해야 된다. 코어가 40, 메모리가 160GB를 갖는 컴퓨터 생성도 가능하지만,
 무료로 사용가능한 `Free Tier (t2.micro)` 가상컴퓨터를 골라 생성시킨다.

<img src="fig/aws-instance-type.png" alt="AWS Free Tier 가상컴퓨터 사양선정" width="77%" /> 

#### 2.4.3. 가상컴퓨터 환경설정(추가옵션)

기본 설정 사항은 그대로 두고, `Advanced section`으로 가서 **User Data** 텍스트 상자에 쉘스크립트를 복사해서 넣거나 입력하여 
가상컴퓨터가 생성될 때 필요한 소프트웨어를 설치한다.

<img src="fig/aws-shell-script.png" alt="AWS 쉘스크립트 설정" width="77%" />


#### 2.4.4. 저장장치 추가설치

대용량 저장소 혹은 별도 저장장치가 필요한 경우 선택하여 붙인다. 물론 추가 요금이 발생될 수 있다.

#### 2.4.5. 태그 추가

가상컴퓨터에 태그 이름을 붙여 가독성을 부여한다. 예를 들어 `rstudio-for-data-science`.

#### 2.4.6. 보안그룹(Security Group) 환경설정

지금까지 가상컴퓨터를 AWS 내부적으로 생성시켰다. 이제 가상컴퓨터를 현실세계와 연결시키는 것이 필요한데,
보안그룹 환경설정을 통해 연결작업을 완성한다. 앞서 생성한 보안그룹을 선택하여 가상컴퓨터에 적용시키면 연결작업은 완료된다.

<img src="fig/aws-sg-select.png" alt="AWS 보안그룹 설정" width="77%" />

마지막으로 가상컴퓨터에 로그인하는데 필요한 보안키를 설정하라는 안내가 나오면 앞서 생성한 인증키를 선택한다.

<img src="fig/aws-kp-select.png" alt="AWS 로그인 보안키 적용" width="57%" />


### 2.5. 가상컴퓨터 생성 확인

AWS가 사용자가 주문한 모든 주문사항을 확인하고 이상이 없으면 가상컴퓨터를 생성시키는 작업을 수행한다.
몇분정도 소요가 되고 나면 **EC2 Console** &rarr; **Instances** 화면에 가상컴퓨터(인스턴스)가 녹색불로 바뀌어 **running**으로 표시된 것이 확인이 될 것이다.
가상 컴퓨터 **Public DNS**를 확인하고 이 정보를 활용하여 SSH 로그인 혹은 HTTP를 통해 RStudio 서버로 접속한다.

## 3. EC2 가상컴퓨터 로그인 

### 3.1. SSH 로그인

**Public DNS** 주소를 바탕으로, 앞서 다운로드받은 `.pem` 비밀키를 사용하여 가상컴퓨터에 로그인한다.


~~~{.r}
ssh -i ~/aws.pem ubuntu@ec2-52-36-52-70.us-west-2.compute.amazonaws.com
~~~

만약 다음과 같은 오류가 생기면 다른 사람이 비밀키에 접근해서 조작을 할 수 있기 때문에 `chmod 400 ~/aws.pem` 명령어로 보안설정을 바꿔 놓고 상기 명령어로 다시 접속한다.
`ubuntu`가 사용자 명이 되고, AWS 가상컴퓨터에는 비밀키(`.pem`)를 갖고 로그인한다.


~~~{.r}
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0640 for 'aws.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "aws.pem": bad permissions
Permission denied (publickey).
~~~

### 3.2. RStudio 로그인

앞서 설치된 AWS AMI에는 [RStudio](https://www.rstudio.com/products/rstudio/)가 내장되어 있다. 또한 RStudio 서버 사용자로 `rstudio`가 
생성되어 있어서 비밀번호만 변경하면 된다. 혹은 RStudio 서버 IDE가 열리면 `RStudioAMI::passwd()` 명령어로 비밀번호를 변경한다.


~~~{.r}
sudo passwd rstudio
~~~

**Public DNS** 주소를 웹브라우져에 입력하고 엔터를 치면 로그인 인증화면이 나오는데 기본설정으로 사용자명: `rstudio`, 비밀번호: `rstudio`가 
설정되어 있고, **Public DNS** 주소를 통해 접속이 되지 않는 경우 포트번호를 `:8787`을 붙여 RStudio 서버에 접속한다.

<img src="fig/aws-sign-in.png" alt="AWS RStudio 로그인" width="57%" />
