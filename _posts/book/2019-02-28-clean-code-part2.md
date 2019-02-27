---
title: "Clean Code - 로버트 C.마틴 #2장"
#excerpt: "애자일 소프트웨어 장인 정신"
categories: 
  - 도서
tags: 
  - 도서
last_modified_at: 2019-02-28T00:00:00+09:00
toc: true
toc_sticky: true
---

## Info
책을 읽을 때 목차를 정리하고 목차의 구성원을 하나씩 하나씩 채워 나가는 재미가 있다. 이 책 또한 마찬가지였다. 구입한 책을 펼쳐 들고 각각의 주제들을 읽고 그 내용을 정리하고자 하였는데 이미 잘 정리된 프로젝트가 존재하였다.  
[Clean-Code 스터디 결과물 github](https://github.com/Yooii-Studios/Clean-Code "Clean-Code 스터디 결과물")  
이런 좋은 문서를 공개해주신 팀에 감사드린다. <q>1장 코드가 존재하리라</q> 주제를 정리하면서 위의 스터디 결과물의 내용만큼 깔끔하게 정리할 수 있을 거란 생각은 들지 않았다. 이번 책은 위 링크의 내용과 함께 읽어 나갈 생각이다. 스터디 모임에 참석해 본 경험이 없지만, 글은 같은 내용일지라도 개인의 경험에 따라 다양하게 해석되고 받아들여지리라 생각한다. 그런 다양한 견해들을 접할 기회와 여건이 된다면 얼마나 좋을까?

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
- 중의적으로 해석될 수 있는 이름 지양하기.
- 개발자에게는 특수한 의미를 가지는 단어(List 등)는 실제 컨테이너가 List가 아닌 이상 accountList라 명명하지 않는다. 차라리 accountGroup, bunchOfAccounts, accounts등으로 명명하자
- 흡사한 이름은 사용하지 않도록 주의한다. XYZControllerForEfficientHandlingOfStrings, XYZControllerForEfficientStorageOfStringsfksms라는 이름을 사용한다면? 차이를 알아챘는가?

### 의미 있게 구분하라
- 연속된 숫자를 덧붙이거나 불용어를 추가하는 방식은 지양 (a1, a2, …), 아무런 정보를 제공하지 못하는, 저자의 의도가 전혀 드러나지 않는 명명이다.
- 클래스 이름에 Info, Data와 같은 불용어를 붙이지 말자. 정확한 개념 구분이 되지 않음
  - Product라는 클래스가 있다고 가정하자. ProductInfo, ProductData라 부른다면 개념은 구분하지 않은 채 이름만 달리한 경우다.
- 불용어는 중복이다.
  - Name VS NameString
  - getActiveAccount() VS getActiveAccounts() VS getActiveAccountInfo() (이들이 혼재할 경우 서로의 역할을 정확히 구분하기 어렵다.)
  - money VS moneyAmount
  - message VS theMessage

### 발음하기 쉬운 이름을 사용하라
- 동료간 코드의 Object를 지칭하여 의사소통을 할 경우를 떠올려라.
  - "홍길동님, 이 레코드 좀 보세요. 'Generation Timestamp'값이 내일 날짜입니다.! 어떻게 이렇죠?"

### 검색하기 쉬운 이름을 사용하라
- 상수는 static final과 같이 정의해 쓰자.
- 변수 이름의 길이는 변수의 범위에 비례해서 길어진다.
- IDE 검색 고려

### 인코딩을 피하라

#### 헝가리식 표기법
예전 C언어 개발시 개발속도를 단축해주는 유용한 수단이였다 하지만 객체는 강한 타입이며 IDE는 비약적으로 발전해 오히려 해독과 리팩토리에 방해가 된다.
변수명에 해당 변수의 타입(str, int 등)을 적지 말자

#### 멤버 변수 접두어
클래스와 함수는 접두어가 필요 없을 정도로 작아야 한다. 그리고 사실 인간이 변수를 읽을땐 접두어(또는 접미어)를 무시하고 해독하는 편이다.

#### 인터페이스 클래스와 구현 클래스
인터페이스 클래스와 구현 클래스를 나눠야 한다면 구현 클래스의 이름에 정보를 인코딩하자.  
ShapeFactory 인터페이스와 이를 구현한 ShapeFactoryImp...

### 자신의 기억력을 자랑하지 마라
독자가 머리속으로 한번 더 생각해 변환해야 할만한 변수명을 쓰지 말라.(eg, URL에서 호스트와 프로토콜을 제외한 소문자 주소를 r이라는 변수로 명명하는 일 등)
똑똑한 프로그래머와 전문가 프로그래머를 나누는 기준 한가지는 "Clarity(명료함)"이다.

### 클래스 이름
- 명사 혹은 명사구를 사용하라.(Customer, WikiPage, Account, AddressParser)
- Manager, Processor, Data, Info와 같은 단어는 피하자
- 동사는 사용하지 않는다.
- 클래스를 하나의 객체(Object)로 인식하자.

### 메서드 이름
- 동사 혹은 동사구를 사용하라.(postPayment, deletePayment, deletePage, save 등)
- 접근자, 변경자, 조건자는 get, set, is로 시작하자. (추가: should, has 등도 가능)
- 생성자를 오버로드할 경우 정적 팩토리 메서드를 사용하고 메서드는 인수를 설명하는 이름을 사용한다.
{% highlight java linenos %}
Complex fulcrumPoint = Comlex.FromRealNumber(23.0);

위 코드가 아래 코드보다 좋다.

Complex fulcrumPoint = new Complex(23.0);
{% endhighlight %}

- 생성자 사용을 제한하려면 해당 생성자를 private으로 선언한다.

### 기발한 이름은 피하라
- 특정 문화에서만 사용되는 재미있는 이름보다 의도를 분명히 표현하는 이름을 사용하라
  - HolyHandGrenade → DeleteItems
  - whack() → kill()

### 한 개념에
한 단어를 사용하라
- 추상적인 개념 하나에 단어 하나를 사용하자.
  - 똑같은 메서드를 클래스마다 fetch, retrieve, get으로 제각각 부르면 혼란스럽다.
  - 마찬가지로 controller, manager, driver를 섞어 쓰면 혼란스럽다.
- 일관성 있는 어휘를 사용하자.

### 말장난을 하지 마라
한 단어를 두 가지 목적으로 사용하지 말자. 아래와 같은 경우에는 아래 메서드명을 append 혹은 insert로 바꾸는게 옳겠다.
{% highlight java linenos %}
public static String add(String message, String messageToAppend)  
public List<Element> add(Element element)
{% endhighlight %}

### 해법 영역에서 가져온 이름을 사용하라
개발자라면 당연히 알고 있을 JobQueue, AccountVisitor(Visitor pattern)등을 사용하지 않을 이유는 없다. 전산용어, 알고리즘 이름, 패턴 이름, 수학 용어 등은 사용하자.

### 문제 영역에서 가져온 이름을 사용하라
적절한 프로그래머 용어(위 13)가 없거나 문제영역과 관련이 깊은 용어의 경우 문제 영역 용어를 사용하자.

### 의미 있는 맥락을 추가하라
- 클래스, 함수, namespace등으로 감싸서 맥락(Context)을 표현하라
- 그래도 불분명하다면 접두어를 사용하자.

{% highlight java linenos %}
// Bad
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {  
        number = "no";  
        verb = "are";  
        pluralModifier = "s";  
    }  else if (count == 1) {
        number = "1";  
        verb = "is";  
        pluralModifier = "";  
    }  else {
        number = Integer.toString(count);  
        verb = "are";  
        pluralModifier = "s";  
    }
    String guessMessage = String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );

    print(guessMessage);
}
{% endhighlight %}

{% highlight java linenos %}
// Good
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreM
            anyLetters(count);
        }
    }

    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }

    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
{% endhighlight %}

### 불필요한 맥락을 없애라
- `Gas Station Delux` 이라는 어플리케이션을 작성한다고 해서 클래스 이름의 앞에 GSD를 붙이지는 말자. G를 입력하고 자동완성을 누를 경우 모든 클래스가 나타나는 등 효율적이지 못하다.  
위 a처럼 접두어를 붙이는 것은 모듈의 재사용 관점에서도 좋지 못하다. 재사용하려면 이름을 바꿔야 한다.(eg, `GSDAccountAddress` 대신 `Address`라고만 해도 충분하다.)  

### 마치면서
> 두려워하지 말고 서로의 명명을 지적하고 고치자. 그렇게 하면 이름을 외우는 것에 시간을 빼앗기지 않고 "자연스럽게 읽히는 코드"를 짜는 데에 더 집중할 수 있다.

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
