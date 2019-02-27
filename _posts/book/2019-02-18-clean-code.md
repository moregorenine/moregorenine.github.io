---
title: "Clean Code - 로버트 C.마틴"
#excerpt: "애자일 소프트웨어 장인 정신"
categories: 
  - 도서
tags: 
  - 도서
last_modified_at: 2019-02-18T00:00:00+09:00
toc: true
toc_sticky: true
---

## Info
책을 읽을 때 목차를 정리하고 목차의 구성원을 하나씩 하나씩 채워 나가는 재미가 있다. 이 책 또한 마찬가지였다. 구입한 책을 펼쳐 들고 각각의 주제들을 읽고 그 내용을 정리하고자 하였는데 이미 잘 정리된 프로젝트가 존재하였다.  
[Clean-Code 스터디 결과물 github](https://github.com/Yooii-Studios/Clean-Code "Clean-Code 스터디 결과물")  
이런 좋은 문서를 공개해주신 팀에 감사드린다. <q>1장 코드가 존재하리라</q> 주제를 정리하면서 위의 스터디 결과물의 내용만큼 깔끔하게 정리할 수 있을 거란 생각은 들지 않았다. 이번 책은 위 링크의 내용과 함께 읽어 나갈 생각이다. 스터디 모임에 참석해 본 경험이 없지만, 글은 같은 내용일지라도 개인의 경험에 따라 다양하게 해석되고 받아들여지리라 생각한다. 그런 다양한 견해들을 접할 기회와 여건이 된다면 얼마나 좋을까?


## 1장 깨끗한 코드
### 코드가 존재하리라
<q>AI의 발달로 몇 년 후 사라질 직업&#8230;</q> 이란 기사가 심심찮게 보인다. 프로그래머 또한 모델이나 요구사항에 집중해야 한다고 생각할지 모르겠다. 하지만 이에 대한 반론을 제기한다. 코드는 요구사항을 상세히 표현하는 수단이다. 요구사항을 모호하게 줘도 우리 의도를 정확히 꿰뚫어 프로그램을 완벽하게 실행하는 그런 기계가 나올 수 있을까? 모호함에서 정밀한 표현의 단계로 옮겨가기 위한 그 자체가 코드이기 때문이다.

### 나쁜 코드
엔트로피는 증가한다. 이는 자연의 법칙이고 코드 또한 이 법칙에서 벗어날 수 없다고 생각한다. 증가하는 엔트로피는 제때에 리팩토링 하지 않으면 우리는 혼돈의 카오스를 맞이하게 될 것이다.
  
### 나쁜 코드로 치르는 대가
나쁜 코드가 쌓일 수록 그 팀의 생산성은 떨어지고 이윽고 0에 수렴한다.
  
- 원대한 재설계의 꿈
  - 나쁜 코드가 쌓여 더는 리팩도링도 수정도 불가한 상태에 이르러 재디자인을 마련한다.

- 태도
나쁜 코드의 원인이 무리한 요구사항과 일정이라 때문이라 어쩔 수 없다고 생각할 수도 있다.
하지만 일정에 쫓기더라도 대다수 관리자는 좋은 코드를 원한다.
관리자들이 일정과 요구사항을 강력하게 밀어붙이는 이유는 그것이 그들의 책임이기 때문이다.
그리고 좋은 코드를 사수하는 일은 바로 우리 프로그래머들의 책임이다.

- 원초적 난제
기한을 맞추기 위해 더러운 코드를 짜나 더러운 코드가 결국은 기한에 맞추는 데 방해 요소가 된다. 즉 언 발에 오줌 누는 꼴이다. 빨리 가기 위한 유일한 방법은, 언제나 코드를 최대한 깨끗하게 유지하는 습관이다.

- 깨끗한 코드라는 예술?
잘 그린 그림을 구분하는 능력이 그림을 잘 그리는 능력은 아니다. 깨끗한 코드를 작성하려면 '청결'이라는 힘겹게 습득한 감각을 활용해 자잘한 기법들을 적용하는 절제와 규율이 필요하다.

- 깨끗한 코드란?
<details>
  <summary>
    <b> Bjarne Stroustrup, inventor of C++ and author of The C++ Programming Language </b>
  </summary>

> *I like my code to be elegant and efficient. The logic should be straightforward to make it hard for bugs to hide, the dependencies minimal to ease maintenance, error handling complete according to an articulated strategy, and performance close to optimal so as not to tempt people to make the code messy with unprincipled optimizations. Clean code does one thing well.*
</details>

  - 코드는 즐겁게 읽혀야 한다.
  - 효율적인 코드라야 한다. 이는 성능적 측면 뿐만 아니라 나쁜 코드는 난장판을 더 키우기 때문이다.(깨진 유리창 이론)
  - 에러 핸들링, 메모리 누수, 경쟁상태, 일관되지 않은 네이밍 등 디테일을 신경쓰라.
  - 나쁜 코드는 여러가지 일을 하려고 한다. 나쁜 코드는 애매한 의도와 모호한 목적을 포함한다. 클린코드는 한 가지에 집중한다. **클린코드는 한 가지 일을 잘 한다.**


<details>
  <summary>
    <b> Grady Booch, author of Object Oriented Analysis and Design with Applications </b>
  </summary>

> *Clean code is simple and direct. Clean code reads like well-written prose. Clean code never obscures the designer’s intent but rather is full of crisp abstractions and straightforward lines of control.*
</details>

- 클린코드는 하나의 잘 쓰여진 산문처럼 읽혀야 한다. 소설의 기승전결처럼 문제를 제시하고 명쾌한 해답을 제시해야 한다.
- 명백한 추상: 코드는 추측 대신 실제를 중시, 필요한 것만 포함하며 독자로 하여금 결단을 내렸다고 생각하게 해야 한다.


<details>
  <summary>
    <b> “Big” Dave Thomas, founder of OTI, godfather of the Eclipse strategy </b>
  </summary>

> *Clean code can be read, and enhanced by a developer other than its original author. It has unit and acceptance tests. It has meaningful names. It provides one way rather than many ways for doing one thing. It has minimal dependencies, which are explicitly defined, and provides a clear and minimal API. Code should be literate since depending on the language, not all necessary information can be expressed clearly in code alone.*
</details>

- 다른 이가 수정하기 쉬워야 한다.
- 테스트를 해야 한다.
- 코드는 간결할 수록 좋다.(Smaller is better)
- 코드는 세련되어야 한다.


<details>
  <summary>
    <b> Michael Feathers, author of Working Effectively with Legacy Code </b>
  </summary>

> *I could list all of the qualities that I notice in clean code, but there is one overarching quality that leads to all of them. Clean code always looks like it was written by someone who cares. There is nothing obvious that you can do to make it better. All of those things were thought about by the code’s author, and if you try to imagine improvements, you’re led back to where you are, sitting in appreciation of the code someone left for you—code left by someone who cares deeply about the craft.*
</details>

- 코드를 **care**하라.(주의, 관심을 가지고 작성하라)


<details>
  <summary>
    <b> Ron Jeffries, author of Extreme Programming Installed and Extreme Programming Adventures in C# </b>
  </summary>

> *In recent years I begin, and nearly end, with Beck’s rules of simple code. In priority order, simple code:*  
>   *- Runs all the tests;*  
>   *- Contains no duplication;*  
>   *- Expresses all the design ideas that are in the system;*  
>   *- Minimizes the number of entities such as classes, methods, functions, and the like.*  
> &nbsp;&nbsp;&nbsp;&nbsp;*Of these, I focus mostly on duplication. When the same thing is done over and over, it’s a sign that there is an idea in our mind that is not well represented in the code. I try to figure out what it is. Then I try to express that idea more clearly. Expressiveness to me includes meaningful names, and I am likely to change the names of things several times before I settle in. With modern coding tools such as Eclipse, renaming is quite inexpensive, so it doesn’t trouble me to change.*  
> &nbsp;&nbsp;&nbsp;&nbsp;*Expressiveness goes beyond names, however. I also look at whether an object or method is doing more than one thing. If it’s an object, it probably needs to be broken into two or more objects. If it’s a method, I will always use the Extract Method refactoring on it, resulting in one method that says more clearly what it does, and some submethods saying how it is done.*  
> &nbsp;&nbsp;&nbsp;&nbsp;*Duplication and expressiveness take me a very long way into what I consider clean code, and improving dirty code with just these two things in mind can make a huge difference. There is, however, one other thing that I’m aware of doing, which is a bit harder to explain.*  
> &nbsp;&nbsp;&nbsp;&nbsp;*After years of doing this work, it seems to me that all programs are made up of very similar elements. One example is “find things in a collection.” Whether we have a database of employee records, or a hash map of keys and values, or an array of items of some kind, we often find ourselves wanting a particular item from that collection. When I find that happening, I will often wrap the particular implementation in a more abstract method or class. That gives me a couple of interesting advantages.*  
> &nbsp;&nbsp;&nbsp;&nbsp;*I can implement the functionality now with something simple, say a hash map, but since now all the references to that search are covered by my little abstraction, I can change the implementation any time I want. I can go forward quickly while preserving my ability to change later.*  
> &nbsp;&nbsp;&nbsp;&nbsp;*In addition, the collection abstraction often calls my attention to what’s “really” going on, and keeps me from running down the path of implementing arbitrary collection behavior when all I really need is a few fairly simple ways of finding what I want.*  
> &nbsp;&nbsp;&nbsp;&nbsp;*Reduced duplication, high expressiveness, and early building of simple abstractions. That’s what makes clean code for me.*
</details>

- 중복을 없애라
- 클래스/메서드는 한 가지 일만 하게 하라
- 메서드의 이름 등으로 코드가 하는 일을 명시하라
- (메서드 등을) 일찍 추상화해서 프로젝트를 빠르게 진행할 수 있게 하라


<details>
  <summary>
    <b> Ward Cunningham, inventor of Wiki, inventor of Fit, coinventor of eXtreme Programming. Motive force behind Design Patterns. Smalltalk and OO thought leader. The godfather of all those who care about code. </b>
  </summary>

> *You know you are working on clean code when each routine you read turns out to be pretty much what you expected. You can call it beautiful code when the code also makes it look like the language was made for the problem.*
</details>

- 읽고, 끄덕이고, 다음으로 넘어갈 수 있는 코드를 작성하라.
- 당신이 사용하는 언어를 탓하지 말라. 코드를 아름답게 만드는 것은 프로그래머이다.

### 우리들 생각
저자와 저자의 동료들이 생각하는 깨끗한 코드는 수십 년에 걸친 경험과 반복적인 시행착오로 습득한 교훈과 기법이며 절대적으로 옳다고 주장할 의도는 없음을 말한다.

### 우리는 저자다
Javadoc에서 @author 필드는 저자를 소개한다. 저자에게는 독자가 있다. 코드를 읽는 시간대 짜는 시간 비율이 10대 1을 훌쩍 넘는다. 새 코드를 짜면서 우리는 **끊임없이** 기존 코드를 읽는다.

### 보이스카우트 규칙
> 캠프장은 처음 왔을 때보다 더 깨끗하게 해놓고 떠나라.

시간이 지날수록 코드가 **좋아지는** 프로젝트에서 작업한다고 상상해보라!

### 프리퀄과 원칙
여러모로 봐서 이 책은 내가 지난 2002년에 쓴 책 Agile Software Development: Principles, Patterns, and Practices (PPP) 의 프리퀄이다. SRP, OCP, DIP등 PPP에서 설명하는 객체지향 디자인의 원칙과 실제에 대한 설명이 종종 나오기 때문에 같이 읽어보면 좋을 것이다.

### 결론
이 책은 당신을 예술가로 만들어줄 수는 없다. 다만, 다른 아티스트가 사용했던 툴, 기술, 사고방식 등을 전달해줄 수 있을 뿐이다.  
예술 교본과 마찬가지로 이 책은 당신에게 많은 (좋은/나쁜)코드를 보여줄 것이다. 나쁜 코드들이 좋은 코드로 바뀌는 것도 볼 것이다. 많은 휴리스틱, 규율, 태크닉을 볼 것이다. 예제, 그리고 더 많은 예제들을 볼 것이다. 그 다음은 당신의 몫이다.  
공연에 지각한 콘서트 바이올리니스트에 대한 옛 농담을 기억하는가? 그는 코너에 있던 한 노인을 불러 세우고는 카네기 홀 까지의 길을 물었다. 노인은 그와 그의 바이올린을 쳐다보고는 말했다. **"연습해 젊은이. 연습!"**

## 2장 의미 있는 이름
### 의도를 분명히 밝혀라
- 변수(혹은 함수나 클래스)의 존재 이유?, 수행 기능?, 사용 방법은? 따로 주석이 필요하다면 의도를 분명히 드러내지 못했다는 말이다.
- 의미를 함축하거나 독자(코드를 읽는 사람)가 사전지식을 가지고 있다고 가정하지 말자.
{% highlight java linenos %}
public List<int[]> getFlaggedCells() {
    List<Cell> flaggedCells = new ArrayList<Cell>();
    for (Cell cell : gameBoard) {
        if (cell.isFlagged()) {
            flaggedCells.add(cell);
        }
    }
    return flaggedCells;
}
{% endhighlight %}
### 그릇된 정보를 피하라
### 의미 있게 구분하라
### 발음하기 쉬운 이름을 사용하라
### 검색하기 쉬운 이름을 사용하라
### 인코딩을 피하라
#### 헝가리식 표기법
#### 멤버 변수 접두어
#### 인터페이스 클래스와 구현 클래스
### 자신의 기억력을 자랑하지 마라
### 클래스 이름
### 메서드 이름
### 기발한 이름은 피하라
### 한 개념에 한 단어를 사용하라
### 말장난을 하지 마라
### 해법 영역에서 가져온 이름을 사용하라
### 문제 영역에서 가져온 이름을 사용하라
### 의미 있는 맥락을 추가하라
### 불필요한 맥락을 없애라
### 마치면서

## 3장 함수
작게 만들어라!
__ 블록과 들여쓰기

한 가지만 해라!
__ 함수 내 섹션

함수 당 추상화 수준은 하나로!
__ 위에서 아래로 코드 읽기: 내려가기 규칙

Switch 문
서술적인 이름을 사용하라!
함수 인수
__ 많이 쓰는 단항 형식
__ 플래그 인수
__ 이항 함수
__ 삼항 함수
__ 인수 객체
__ 인수 목록
__ 동사와 키워드

부수 효과를 일으키지 마라!
__ 출력 인수

명령과 조회를 분리하라!
오류 코드보다 예외를 사용하라!
__ Try/Catch 블록 뽑아내기
__ 오류 처리도 한 가지 작업이다.
__ Error.java 의존성 자석

반복하지 마라!
구조적 프로그래밍
함수를 어떻게 짜죠?
결론
참고 문헌

## 4장 주석
주석은 나쁜 코드를 보완하지 못한다
코드로 의도를 표현하라!
좋은 주석
__ 법적인 주석
__ 정보를 제공하는 주석
__ 의도를 설명하는 주석
__ 의미를 명료하게 밝히는 주석
__ 결과를 경고하는 주석
__ TODO 주석
__ 중요성을 강조하는 주석
__ 공개 API에서 Javadocs

나쁜 주석
__ 주절거리는 주석
__ 같은 이야기를 중복하는 주석
__ 오해할 여지가 있는 주석
__ 의무적으로 다는 주석
__ 이력을 기록하는 주석
__ 있으나 마나 한 주석
__ 무서운 잡음
__ 함수나 변수로 표현할 수 있다면 주석을 달지 마라
__ 위치를 표시하는 주석
__ 닫는 괄호에 다는 주석
__ 공로를 돌리거나 저자를 표시하는 주석
__ 주석으로 처리한 코드
__ HTML 주석
__ 전역 정보
__ 너무 많은 정보
__ 모호한 관계
__ 함수 헤더
__ 비공개 코드에서 Javadocs
__ 예제

참고 문헌

## 5장 형식 맞추기
형식을 맞추는 목적
적절한 행 길이를 유지하라
__ 신문 기사처럼 작성하라
__ 개념은 빈 행으로 분리하라
__ 세로 밀집도
__ 수직 거리
__ 세로 순서

가로 형식 맞추기
__ 가로 공백과 밀집도
__ 가로 정렬
__ 들여쓰기

가짜 범위
팀 규칙
밥 아저씨의 형식 규칙

## 6장 객체와 자료 구조
자료 추상화
자료/객체 비대칭
디미터 법칙
__ 기차 충돌
__ 잡종 구조
__ 구조체 감추기

자료 전달 객체
__ 활성 레코드

결론
참고 문헌

## 7장 오류 처리
오류 코드보다 예외를 사용하라
Try-Catch-Finally 문부터 작성하라
미확인unchecked 예외를 사용하라
예외에 의미를 제공하라
호출자를 고려해 예외 클래스를 정의하라
정상 흐름을 정의하라
null을 반환하지 마라
null을 전달하지 마라
결론
참고문헌

## 8장 경계
외부 코드 사용하기
경계 살피고 익히기
log4j 익히기
학습 테스트는 공짜 이상이다
아직 존재하지 않는 코드를 사용하기
깨끗한 경계
참고 문헌

## 9장 단위 테스트
TDD 법칙 세 가지
깨끗한 테스트 코드 유지하기
__ 테스트는 유연성, 유지보수성, 재사용성을 제공한다

깨끗한 테스트 코드
__ 도메인에 특화된 테스트 언어
__ 이중 표준

테스트 당 assert 하나
__ 테스트 당 개념 하나

F.I.R.S.T.
결론
참고 문헌

## 10장 클래스
클래스 체계
__ 캡슐화

클래스는 작아야 한다!
__ 단일 책임 원칙
__ 응집도Cohesion
__ 응집도를 유지하면 작은 클래스 여럿이 나온다

변경하기 쉬운 클래스
__ 변경으로부터 격리

참고 문헌

## 11장 시스템
도시를 세운다면?
시스템 제작과 시스템 사용을 분리하라
__ Main 분리
__ 팩토리
__ 의존성 주입

확장
__ 횡단(cross-cutting) 관심사

자바 프록시
순수 자바 AOP 프레임워크
AspectJ 관점
테스트 주도 시스템 아키텍처 구축
의사 결정을 최적화하라
명백한 가치가 있을 때 표준을 현명하게 사용하라
시스템은 도메인 특화 언어가 필요하다
결론
참고 문헌

## 12장 창발성(創發性)
창발적 설계로 깔끔한 코드를 구현하자
단순한 설계 규칙 1: 모든 테스트를 실행하라
단순한 설계 규칙 2~4: 리팩터링
중복을 없애라
표현하라
클래스와 메서드 수를 최소로 줄여라
결론
참고 문헌

## 13장 동시성
동시성이 필요한 이유?
__ 미신과 오해

난관
동시성 방어 원칙
__ 단일 책임 원칙Single Responsibility Principle, SRP
__ 따름 정리corollary: 자료 범위를 제한하라
__ 따름 정리: 자료 사본을 사용하라
__ 따름 정리: 스레드는 가능한 독립적으로 구현하라

라이브러리를 이해하라
__ 스레드 환경에 안전한 컬렉션

실행 모델을 이해하라
__ 생산자-소비자Producer-Consumer
__ 읽기-쓰기Readers-Writers
__ 식사하는 철학자들Dining Philosophers

동기화하는 메서드 사이에 존재하는 의존성을 이해하라
동기화하는 부분을 작게 만들어라
올바른 종료 코드는 구현하기 어렵다
스레드 코드 테스트하기
__ 말이 안 되는 실패는 잠정적인 스레드 문제로 취급하라
__ 다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자
__ 다중 스레드를 쓰는 코드 부분을 다양한 환경에 쉽게 끼워 넣을 수 있게 스레드 코드를 구현하라
__ 다중 스레드를 쓰는 코드 부분을 상황에 맞게 조율할 수 있게 작성하라
__ 프로세서 수보다 많은 스레드를 돌려보라
__ 다른 플랫폼에서 돌려보라
__ 코드에 보조 코드instrument를 넣어 돌려라. 강제로 실패를 일으키게 해보라
__ 직접 구현하기
__ 자동화

결론
참고 문헌

## 14장 점진적인 개선
Args 구현
__ 어떻게 짰느냐고?

Args: 1차 초안
__ 그래서 멈췄다
__ 점진적으로 개선하다

String 인수
결론

## 15장 JUnit 들여다보기
JUnit 프레임워크
결론

## 16장 SerialDate 리팩터링
첫째, 돌려보자
둘째, 고쳐보자
결론
참고 문헌

## 17장 냄새와 휴리스틱

주석
__ C1: 부적절한 정보
__ C2: 쓸모 없는 주석
__ C3: 중복된 주석
__ C4: 성의 없는 주석
__ C5: 주석 처리된 코드

환경
__ E1: 여러 단계로 빌드해야 한다
__ E2: 여러 단계로 테스트해야 한다

함수
__ F1: 너무 많은 인수
__ F2: 출력 인수
__ F3: 플래그 인수
__ F4: 죽은 함수

일반
__ G1: 한 소스 파일에 여러 언어를 사용한다
__ G2: 당연한 동작을 구현하지 않는다
__ G3: 경계를 올바로 처리하지 않는다
__ G4: 안전 절차 무시
__ G5: 중복
__ G6: 추상화 수준이 올바르지 못하다
__ G7: 기초 클래스가 파생 클래스에 의존한다
__ G8: 과도한 정보
__ G9: 죽은 코드
__ G10: 수직 분리
__ G11: 일관성 부족
__ G12: 잡동사니
__ G13: 인위적 결합
__ G14: 기능 욕심
__ G15: 선택자 인수
__ G16: 모호한 의도
__ G17: 잘못 지운 책임
__ G18: 부적절한 static 함수
__ G19: 서술적 변수
__ G20: 이름과 기능이 일치하는 함수
__ G21: 알고리즘을 이해하라
__ G22: 논리적 의존성은 물리적으로 드러내라
__ G23: If/Else 혹은 Switch/Case 문보다 다형성을 사용하라
__ G24: 표준 표기법을 따르라
__ G25: 매직 숫자는 명명된 상수로 교체하라
__ G26: 정확하라
__ G27: 관례보다 구조를 사용하라
__ G28: 조건을 캡슐화하라
__ G29: 부정 조건은 피하라
__ G30: 함수는 한 가지만 해야 한다
__ G31: 숨겨진 시간적인 결합
__ G32: 일관성을 유지하라
__ G33: 경계 조건을 캡슐화하라
__ G34: 함수는 추상화 수준을 한 단계만 내려가야 한다
__ G35: 설정 정보는 최상위 단계에 둬라
__ G36: 추이적 탐색을 피하라

자바
__ J1: 긴 import 목록을 피하고 와일드카드를 사용하라
__ J2: 상수는 상속하지 않는다
__ J3: 상수 대 Enum

이름
__ N1: 서술적인 이름을 사용하라
__ N2: 적절한 추상화 수준에서 이름을 선택하라
__ N3: 가능하다면 표준 명명법을 사용하
__ N4: 명확한 이름
__ N5: 긴 범위는 긴 이름을 사용하라
__ N6: 인코딩을 피하라
__ N7: 이름으로 부수 효과를 설명하라

테스트
__ T1: 불충분한 테스트
__ T2: 커버리지 도구를 사용하라!
__ T3: 사소한 테스트를 건너뛰지 마라
__ T4: 무시한 테스트는 모호함을 뜻한다
__ T5: 경계 조건을 테스트하라
__ T6: 버그 주변은 철저히 테스트하라
__ T7: 실패 패턴을 살펴라
__ T8: 테스트 커버리지 패턴을 살펴라
__ T9: 테스트는 빨라야 한다

결론
참고 문헌

부록A 동시성 II

클라이언트/서버 예제
__ 서버
__ 스레드 추가하기
__ 서버 살펴보기
__ 결론

가능한 실행 경로
__ 경로 수
__ 가능한 순열 수 계산하기
__ 심층 분석
__ 결론

라이브러리를 이해하라
__ Executor 프레임워크
__ 스레드를 차단하지 않는non blocking 방법
__ 다중 스레드 환경에서 안전하지 않은 클래스

메서드 사이에 존재하는 의존성을 조심하라
__ 실패를 용인한다
__ 클라이언트-기반 잠금
__ 서버-기반 잠금

작업 처리량 높이기
__ 작업 처리량 계산 - 단일스레드 환경
__ 작업 처리량 계산 - 다중 스레드 환경

데드락
__ 상호 배제Mutual Exclusion
__ 잠금 & 대기Lock & Wait
__ 선점 불가No Preemption
__ 순환 대기Circular Wait
__ 상호 배제 조건 깨기
__ 잠금 & 대기 조건 깨기
__ 선점 불가 조건 깨기
__ 순환 대기 조건 깨기
__ 다중 스레드 코드 테스트
__ 스레드 코드 테스트를 도와주는 도구

결론
자습서: 전체 코드 예제
__ 클라이언트/서버 - 단일스레드 버전
__ 클라이언트/서버 - 다중 스레드 버전

부록B org.jfree.date.SerialDate

부록C 휴리스틱의 교차 참조 목록

에필로그
용어 대역표
약어 목록
찾아보기
