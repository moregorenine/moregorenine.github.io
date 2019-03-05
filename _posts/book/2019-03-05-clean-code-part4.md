---
title: "Clean Code - 로버트 C.마틴 #4 주석"
#excerpt: "애자일 소프트웨어 장인 정신"
categories: 
  - 도서
tags: 
  - 도서
last_modified_at: 2019-03-05T12:00:00+09:00
toc: true
toc_sticky: true
---

## Info
책을 읽을 때 목차를 정리하고 목차의 구성원을 하나씩 하나씩 채워 나가는 재미가 있다. 이 책 또한 마찬가지였다. 구입한 책을 펼쳐 들고 각각의 주제들을 읽고 그 내용을 정리하고자 하였는데 이미 잘 정리된 프로젝트가 존재하였다.  
[Clean-Code 스터디 결과물 github](https://github.com/Yooii-Studios/Clean-Code "Clean-Code 스터디 결과물")  
이런 좋은 문서를 공개해주신 팀에 감사드린다. <q>1장 코드가 존재하리라</q> 주제를 정리하면서 위의 스터디 결과물의 내용만큼 깔끔하게 정리할 수 있을 거란 생각은 들지 않았다. 이번 책은 위 링크의 내용과 함께 읽어 나갈 생각이다. 스터디 모임에 참석해 본 경험이 없지만, 글은 같은 내용일지라도 개인의 경험에 따라 다양하게 해석되고 받아들여지리라 생각한다. 그런 다양한 견해들을 접할 기회와 여건이 된다면 얼마나 좋을까?

## 4장 주석
> 나쁜 코드에 주석을 달지 마라. 새로 짜라.     
 - 브라이언 W.커니핸, P.J.플라우거

주석은 필요악이다. 코드로 의도를 표현하지 못해, 실패를 만회하기 위해 쓰는 것이다. 주석은 언제나 실패를 의미한다. 주석 없이는 자신을 표현할 방법을 찾지 못해 할 수 없이 주석을 사용한다. 그래서 주석은 반겨 맞을 손님이 아니다.

주석을 무시하는 이유가 무엇이냐고? 주석이 오래될수록 코드에서 멀어져서 거짓말을 하게 될 가능성이 커지기 때문이다. 코드는 유지보수를 해도, 주석을 계속 유지보수하기란 현실적으로 불가능하기 때문이다.

### 주석은 나쁜 코드를 보완하지 못한다
코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나빠서이다. 깔끔하고 주석이 거의 없는 코드가, 복잡하고 어수선하며 주석이 많이 달린 코드보다 훨씬 좋다. 주석으로 설명하려 애쓰는 대신에 그 난장판을 깨끗이 치우는 데 시간을 보내라!

### 코드로 의도를 표현하라!
{% highlight java linenos %}
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다. 
if ((emplotee.flags & HOURLY_FLAG) && (employee.age > 65)
{% endhighlight %}

다음 코드는 어떤가?

{% highlight java linenos %}
if (employee.isEligibleForFullBenefits())
{% endhighlight %}

주석도 필요없이 함수 이름만으로 충분히 깔끔하게 표현되었다.

### 좋은 주석
정말로 좋은 주석은, 주석을 달지 않을 방법을 찾아낸 주석이나 아래는 글자 값을 한다고 생각하는 주석 몇 가지를 소개한다.

#### 법적인 주석
각 소스 파일 첫머리에 들어가는 저작권 정보와 소유권 정보 등
`// Copyright (C) 2003, 2004, 2005 by Object Montor, Inc. All right reserved.`  
`// GNU General Public License`

#### 정보를 제공하는 주석
{% highlight java linenos %}
// 테스트 중인 Responder 인스턴스를 반환
protected abstract Responder responderInstance();
{% endhighlight %}

  물론 이 주석도 함수 이름에 정보를 담아 responderBeingTested로 바꾸면 없앨 수 있다.    
  더 나은 예:

{% highlight java linenos %}
// kk:mm:ss EEE, MMM dd, yyyy 형식이다.
Pattern timeMatcher = Pattern.compile("\\d*:\\d*\\d* \\w*, \\w*, \\d*, \\d*");
{% endhighlight %}

#### 의도를 설명하는 주석
{% highlight java linenos %}
// 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다. 
for (int i = 0; i > 2500; i++) {
    WidgetBuilderThread widgetBuilderThread = 
        new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
    Thread thread = new Thread(widgetBuilderThread);
    thread.start();
}
{% endhighlight %}

#### 의미를 명료하게 밝히는 주석
{% highlight java linenos %}
assertTrue(a.compareTo(b) == 0); // a == b
assertTrue(a.compareTo(b) != 0); // a != b
assertTrue(a.compareTo(b) == -1); // a < b
assertTrue(a.compareTo(b) == 1); // a > b
{% endhighlight %}

#### 결과를 경고하는 주석
{% highlight java linenos %}
// 여유 시간이 충분하지 않다면 실행하지 마십시오.
public void _testWithReallyBigFile() {
}
{% endhighlight %}

#### TODO 주석
{% highlight java linenos %}
// TODO-MdM 현재 필요하지 않다.
// 체크아웃 모델을 도입하면 함수가 필요 없다.
protected VersionInfo makeVersion() throws Exception {
    return null;
}
{% endhighlight %}

#### 중요성을 강조하는 주석
{% highlight java linenos %}
String listItemContent = match.group(3).trim();
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다. 
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
{% endhighlight %}

#### 공개 API에서 Javadocs    
설명이 잘 된 공개 API는 참으로 유용하고 만족스럽다. 공개 API를 구현한다면 반드시 훌륭한 Javadocs 작성을 추천한다. 하지만 여느 주석과 마찬가지로 Javadocs 역시 독자를 오도하거나, 잘못 위치하거나, 그릇된 정보를 전달할 가능성이 존재하는 것 역시 잊으면 안 된다. 

### 나쁜 주석
대다수의 주석이 이 범주에 속한다. 일반적으로 대다수 주석은 허술한 코드를 지탱하거나, 엉성한 코드를 변명하거나, 미숙한 결정을 합리화하는 등 프로그래머가 주절거리는 독백에서 크게 벗어나지 못한다. 

#### 주절거리는 주석
특별한 이유 없이 달리는 주석이다. 

{% highlight java linenos %}
public void loadProperties() {
    try {
        String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
        FileInputStream propertiesStream = new FileInputStream(propertiesPath);
        loadedProperties.load(propertiesStream);
    } catch (IOException e) {
        // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다. 
    }
}
{% endhighlight %}

catch 블록에 있는 주석의 의미를 알아내려면 다른 코드를 뒤져보는 수밖에 없다. 이해가 안되어 다른 모듈까지 뒤져야 하는 주석은 제대로 된 주석이 아니다.

#### 같은 이야기를 중복하는 주석
코드 내용을 그대로 중복하는 주석이 있다. 전혀 필요없는 코드

{% highlight java linenos %}
// this.closed가 true일 때 반환되는 유틸리티 메서드다.
// 타임아웃에 도달하면 예외를 던진다. 
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if (!closed) {
        wait(timeoutMillis);
        if (!closed) {
            throw new Exception("MockResponseSender could not be closed");
        }
    }
}
{% endhighlight %}

#### 오해할 여지가 있는 주석
위 코드를 다시 보자. 중복이 많으면서도 오해할 여지가 살짝 있다. this.closed가 true로 변하는 순간에 메서드는 반환되지 않는다. this.closed가 true여야 메서드는 반환된다. 아니면 무조건 타임아웃을 기다렸다 this.closed가 그래도 true가 아니면 예외를 던진다. 주석에 담긴 '살짝 잘못된 정보'로 인해 어느 프로그래머가 경솔하게 함수를 호출해 자기 코드가 아주 느려진 이유를 못찾게 되는 것이다.

#### 의무적으로 다는 주석
모든 함수에 Javadocs를 달거나 모든 변수에 주석을 달아야 한다는 규칙은 어리석기 그지없다. 이런 주석은 코드를 복잡하게 만들며, 거짓말을 퍼뜨리고, 혼동과 무질서를 초래한다. 아래와 같은 주석은 아무 가치도 없다. 

{% highlight java linenos %}
/**
 *
 * @param title CD 제목
 * @param author CD 저자
 * @param tracks CD 트랙 숫자
 * @param durationInMinutes CD 길이(단위: 분)
 */
public void addCD(String title, String author, int tracks, int durationInMinutes) {
    CD cd = new CD();
    cd.title = title;
    cd.author = author;
    cd.tracks = tracks;
    cd.duration = durationInMinutes;
    cdList.add(cd);
}
{% endhighlight %}

#### 이력을 기록하는 주석
지금은 소스 코드 관리 시스템이 있으니 전혀 필요없다. 

{% highlight java linenos %}
* 변경 이력 (11-Oct-2001부터)
* ------------------------------------------------
* 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키징
* 05-Nov-2001: getDescription() 메소드 추가
* 이하 생략
{% endhighlight %}

#### 있으나 마나 한 주석

{% highlight java linenos %}
/*
 * 기본 생성자
 */
protected AnnualDateRule() {

}
{% endhighlight %}

#### 무서운 잡음
때로는 Javadocs도 잡음이다.

{% highlight java linenos %}
/** The name. */
private String name;

/** The version. */
private String version;
{% endhighlight %}

#### 함수나 변수로 표현할 수 있다면 주석을 달지 마라

{% highlight java linenos %}
// 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (module.getDependSubsystems().contains(subSysMod.getSubSystem()))
{% endhighlight %}

주석을 제거하고 다시 표현하면 다음과 같다.

{% highlight java linenos %}
ArrayList moduleDependencies = smodule.getDependSubSystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
{% endhighlight %}

#### 위치를 표시하는 주석
때때로 프로그래머는 소스 파일에서 특정 위치를 표시하려 주석을 사용한다. 예를 들어, 최근에 살펴보던 프로그램에서 다음 행을 발견했다. 

{% highlight java linenos %}
// Actions /////////////////////////////////////////////
{% endhighlight %}

이런 주석은 가독성만 낮추므로 제거해야 마땅하다. 특히 뒷부분에 슬래시로 이어지는 잡음은 제거하는 편이 좋다. 너무 자주 사용하지 않을때만 배너는 눈에 띄며 주위를 환기한다. 그러므로 반드시 필요할 때 아주 드몰게 사용하는 편이 좋다.

#### 닫는 괄호에 다는 주석
중첩이 심하고 장황한 함수라면 의미가 있을지도 모르지만 작고 캡슐화면 함수에는 잡음일 뿐이다. 그러므로 닫는 괄호에 주석을 달아야겠다는 생각이 든다면 대신에 함수를 줄이려 시도하자. 

#### 공로를 돌리거나 저자를 표시하는 주석
소스 코드 관리 시스템은 누가 언제 무엇을 추가했는지 귀신처럼 기억하기 때문에 저자 이름으로 코드를 오염시킬 필요가 없음. 

{% highlight java linenos %}
/* 릭이 추가함 */
{% endhighlight %}

#### 주석으로 처리한 코드
{% highlight java linenos %}
this.bytePos = writeBytes(pngIdBytes, 0);
//hdrPos = bytePos;
writeHeader();
writeResolution();
//dataPos = bytePos;
if (writeImageData()) {
    wirteEnd();
    this.pngBytes = resizeByteArray(this.pngBytes, this.maxPos);
} else {
    this.pngBytes = null;
}
return this.pngBytes;
{% endhighlight %}

주석으로 처리된 코드는 다른 사람들이 지우기를 주저한다. 소스관리 시스템이 우리를 대신해 코드를 기억해준다. 그냥 삭제하라.

#### HTML 주석
HTML 주석은 편집기/IDE에서조차 읽기가 어렵다.

#### 전역 정보
주석을 달아야 한다면 근처에 있는 코드만 기술하라. 시스템의 전반적인 정보를 기술하지 마라. 해당 시스템의 코드가 변해도 아래 주석이 변하리라는 보장은 전혀 없다. 그리고 심하게 중복된 주석도 확인하자. 

{% highlight java linenos %}
/**
 * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>.
 *
 * @param fitnessePort
 */
public void setFitnessePort(int fitnessePort) {
    this.fitnewssePort = fitnessePort;
}
{% endhighlight %}

#### 너무 많은 정보
주석에다 흥미로운 역사나 관련 없는 정보를 장황하게 늘어놓지 마라.

#### 모호한 관계
주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다.

{% highlight java linenos %}
/**
 * 모든 픽셀을 담을 만큼 충분한 배열로 시작한다(여기에 필터 바이트를 더한다).
 * 그리고 헤더 정보를 위해 200바이트를 더한다.
 */
this.pngBytes = new byte[((this.width + 1) * this.height * 3) + 200];
{% endhighlight %}

필터 바이트는 +1과 *3 중 어느 것과 관련이 있을까? 한 픽셀이 한 바이트인가? 200을 추가하는 이유는? 주석 자체가 다시 설명을 요구하는 경우다.

#### 함수 헤더
짧고 한 가지만 수행하며 이름을 잘 붙인 함수가 주석으로 헤더를 추가한 함수보다 훨씬 좋다.

#### 비공개 코드에서 Javadocs
공개 API는 Javadocs가 유용하지만 공개하지 않을 코드라면 Javadocs는 쓸모가 없다. 코드만 보기싫고 산만해질 뿐이다.

#### 예제

## 5장 형식 맞추기
### 형식을 맞추는 목적
### 적절한 행 길이를 유지하라
__ 신문 기사처럼 작성하라
__ 개념은 빈 행으로 분리하라
__ 세로 밀집도
__ 수직 거리
__ 세로 순서

### 가로 형식 맞추기
__ 가로 공백과 밀집도
__ 가로 정렬
__ 들여쓰기

### 가짜 범위
### 팀 규칙
### 밥 아저씨의 형식 규칙

## 6장 객체와 자료 구조
### 자료 추상화
### 자료/객체 비대칭
### 디미터 법칙
__ 기차 충돌
__ 잡종 구조
__ 구조체 감추기

### 자료 전달 객체
__ 활성 레코드

### 결론
### 참고 문헌

## 7장 오류 처리
### 오류 코드보다 예외를 사용하라
### Try-Catch-Finally 문부터 작성하라
### 미확인unchecked 예외를 사용하라
### 예외에 의미를 제공하라
### 호출자를 고려해 예외 클래스를 정의하라
### 정상 흐름을 정의하라
### null을 반환하지 마라
### null을 전달하지 마라
### 결론
### 참고문헌

## 8장 경계
### 외부 코드 사용하기
### 경계 살피고 익히기
### log4j 익히기
### 학습 테스트는 공짜 이상이다
### 아직 존재하지 않는 코드를 사용하기
### 깨끗한 경계
### 참고 문헌

## 9장 단위 테스트
### TDD 법칙 세 가지
### 깨끗한 테스트 코드 유지하기
__ 테스트는 유연성, 유지보수성, 재사용성을 제공한다

### 깨끗한 테스트 코드
__ 도메인에 특화된 테스트 언어
__ 이중 표준

### 테스트 당 assert 하나
__ 테스트 당 개념 하나

### F.I.R.S.T.
### 결론
### 참고 문헌

## 10장 클래스
### 클래스 체계
__ 캡슐화

### 클래스는 작아야 한다!
__ 단일 책임 원칙
__ 응집도Cohesion
__ 응집도를 유지하면 작은 클래스 여럿이 나온다

### 변경하기 쉬운 클래스
__ 변경으로부터 격리

### 참고 문헌

## 11장 시스템
### 도시를 세운다면?
### 시스템 제작과 시스템 사용을 분리하라
__ Main 분리
__ 팩토리
__ 의존성 주입

### 확장
__ 횡단(cross-cutting) 관심사

### 자바 프록시
### 순수 자바 AOP 프레임워크
### AspectJ 관점
### 테스트 주도 시스템 아키텍처 구축
### 의사 결정을 최적화하라
### 명백한 가치가 있을 때 표준을 현명하게 사용하라
### 시스템은 도메인 특화 언어가 필요하다
### 결론
### 참고 문헌

## 12장 창발성(創發性)
### 창발적 설계로 깔끔한 코드를 구현하자
### 단순한 설계 규칙 1: 모든 테스트를 실행하라
### 단순한 설계 규칙 2~4: 리팩터링
### 중복을 없애라
### 표현하라
### 클래스와 메서드 수를 최소로 줄여라
### 결론
### 참고 문헌

## 13장 동시성
### 동시성이 필요한 이유?
__ 미신과 오해

### 난관
### 동시성 방어 원칙
__ 단일 책임 원칙Single Responsibility Principle, SRP
__ 따름 정리corollary: 자료 범위를 제한하라
__ 따름 정리: 자료 사본을 사용하라
__ 따름 정리: 스레드는 가능한 독립적으로 구현하라

### 라이브러리를 이해하라
__ 스레드 환경에 안전한 컬렉션

### 실행 모델을 이해하라
__ 생산자-소비자Producer-Consumer
__ 읽기-쓰기Readers-Writers
__ 식사하는 철학자들Dining Philosophers

### 동기화하는 메서드 사이에 존재하는 의존성을 이해하라
### 동기화하는 부분을 작게 만들어라
### 올바른 종료 코드는 구현하기 어렵다
### 스레드 코드 테스트하기
__ 말이 안 되는 실패는 잠정적인 스레드 문제로 취급하라
__ 다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자
__ 다중 스레드를 쓰는 코드 부분을 다양한 환경에 쉽게 끼워 넣을 수 있게 스레드 코드를 구현하라
__ 다중 스레드를 쓰는 코드 부분을 상황에 맞게 조율할 수 있게 작성하라
__ 프로세서 수보다 많은 스레드를 돌려보라
__ 다른 플랫폼에서 돌려보라
__ 코드에 보조 코드instrument를 넣어 돌려라. 강제로 실패를 일으키게 해보라
__ 직접 구현하기
__ 자동화

### 결론
### 참고 문헌

## 14장 점진적인 개선
### Args 구현
__ 어떻게 짰느냐고?

### Args: 1차 초안
__ 그래서 멈췄다
__ 점진적으로 개선하다

### String 인수
### 결론

## 15장 JUnit 들여다보기
### JUnit 프레임워크
### 결론

## 16장 SerialDate 리팩터링
### 첫째, 돌려보자
### 둘째, 고쳐보자
### 결론
### 참고 문헌

## 17장 냄새와 휴리스틱

### 주석
__ C1: 부적절한 정보
__ C2: 쓸모 없는 주석
__ C3: 중복된 주석
__ C4: 성의 없는 주석
__ C5: 주석 처리된 코드

### 환경
__ E1: 여러 단계로 빌드해야 한다
__ E2: 여러 단계로 테스트해야 한다

### 함수
__ F1: 너무 많은 인수
__ F2: 출력 인수
__ F3: 플래그 인수
__ F4: 죽은 함수

### 일반
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

### 자바
__ J1: 긴 import 목록을 피하고 와일드카드를 사용하라
__ J2: 상수는 상속하지 않는다
__ J3: 상수 대 Enum

### 이름
__ N1: 서술적인 이름을 사용하라
__ N2: 적절한 추상화 수준에서 이름을 선택하라
__ N3: 가능하다면 표준 명명법을 사용하
__ N4: 명확한 이름
__ N5: 긴 범위는 긴 이름을 사용하라
__ N6: 인코딩을 피하라
__ N7: 이름으로 부수 효과를 설명하라

### 테스트
__ T1: 불충분한 테스트
__ T2: 커버리지 도구를 사용하라!
__ T3: 사소한 테스트를 건너뛰지 마라
__ T4: 무시한 테스트는 모호함을 뜻한다
__ T5: 경계 조건을 테스트하라
__ T6: 버그 주변은 철저히 테스트하라
__ T7: 실패 패턴을 살펴라
__ T8: 테스트 커버리지 패턴을 살펴라
__ T9: 테스트는 빨라야 한다

### 결론
### 참고 문헌

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
