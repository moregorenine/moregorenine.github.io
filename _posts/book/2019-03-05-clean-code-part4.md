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

## 3장 함수

### 작게 만들어라!
- 함수를 만드는 첫번째 규칙은 '작게!'다.
- 함수를 만드는 둘째 규칙은 '더 작게!'다.

{% highlight java linenos %}
public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite) throws Exception {
	boolean isTestPage = pageData.hasAttribute("Test"); 
	if (isTestPage) {
		WikiPage testPage = pageData.getWikiPage(); 
		StringBuffer newPageContent = new StringBuffer(); 
		includeSetupPages(testPage, newPageContent, isSuite); 
		newPageContent.append(pageData.getContent()); 
		includeTeardownPages(testPage, newPageContent, isSuite); 
		pageData.setContent(newPageContent.toString());
	}
	return pageData.getHtml(); 
}
{% endhighlight %}

위 코드도 길다. 되도록 한 함수당 3~5줄 이내로 줄이는 것을 권장한다

{% highlight java linenos %}
public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite) throws Exception { 
   if (isTestPage(pageData)) 
   	includeSetupAndTeardownPages(pageData, isSuite); 
   return pageData.getHtml();
}
{% endhighlight %}

### 블록과 들여쓰기
중첩구조(if/else, while문 등)에 들어가는 블록은 한 줄이어야 한다. 각 함수 별 들여쓰기 수준이 2단을 넘어서지 않고, 각 함수가 명백하다면 함수는 더욱 읽고 이해하기 쉬워진다.

### 한 가지만 해라!
- 함수는 한가지를 해야 한다. 그 한가지를 잘 해야 한다. 그 한가지만을 해야 한다.
- 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 하는 것이다.
- 함수를 여러 섹션으로 나눌 수 있다면 그 함수는 여러작업을 하는 셈이다.

### 함수 당 추상화 수준은 하나로!
함수가 ‘한가지’ 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 된다.  
만약 한 함수 내에 추상화 수준이 섞이게 된다면 읽는 사람이 헷갈린다.

#### 위에서 아래로 코드 읽기: 내려가기 규칙
코드는 위에서 아래로 이야기처럼 읽혀야 좋다.  
함수 추상화 부분이 한번에 한단계씩 낮아지는 것이 가장 이상적이다.(내려가기 규칙)

### Switch 문
{% highlight java linenos %}
public Money calculatePay(Employee e) throws InvalidEmployeeType {
	switch (e.type) { 
		case COMMISSIONED:
			return calculateCommissionedPay(e); 
		case HOURLY:
			return calculateHourlyPay(e); 
		case SALARIED:
			return calculateSalariedPay(e); 
		default:
			throw new InvalidEmployeeType(e.type); 
	}
}
{% endhighlight %}

- 문제점
  - 함수가 길다.
  - '한 가지' 작업만 수행하지 않는다.
  - SRP(Single Responsibility Principle)를 위반한다. -> 코드를 변경할 이유가 여럿이기 때문이다.
    - 단일책임 원칙
    - [참조 - 권영재님 블로그 Nesoy](https://nesoy.github.io/articles/2017-12/SRP "Single Responsibility Principle")  
  - OCP(Open Closed Principle)를 위반한다. -> 새 직원 유형을 추가할 때마다 코드를 변경하기 때문이다.
    - 개방폐쇄 원칙
    - [참조 - 권영재님 블로그 Nesoy](https://nesoy.github.io/articles/2018-01/OCP "Open Closed Principle")  

{% highlight java linenos %}
public abstract class Employee {
	public abstract boolean isPayday();
	public abstract Money calculatePay();
	public abstract void deliverPay(Money pay);
}
-----------------
public interface EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType; 
}
-----------------
public class EmployeeFactoryImpl implements EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
		switch (r.type) {
			case COMMISSIONED:
				return new CommissionedEmployee(r) ;
			case HOURLY:
				return new HourlyEmployee(r);
			case SALARIED:
				return new SalariedEmploye(r);
			default:
				throw new InvalidEmployeeType(r.type);
		} 
	}
}
{% endhighlight %}

- 개선점
  - EmployeeFactoryImpl는 switch 문을 사용해 적절한 Employee 파생 클래스의 인스턴스를 생성한다.
  - Employee의 추상 메소드들은 인터페이스를 거쳐 호출된다. 그러면 다형성으로 인해 실제 파생 클래스의 함수가 실행된다.

### 서술적인 이름을 사용하라!
> “코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다” - 워드

작은 함수는 그 기능이 명확하므로 이름을 붙이기가 더 쉬우며, 일관성 있는 서술형 이름을 사용한다면 코드를 순차적으로 이해하기도 쉬워진다.
- 한 가지만 하는 작은 함수에 좋은 이름을 붙이자.
- 길고 서술적인 이름이 짧고 어려운 이름보다 좋다.
- 길고 서술적인 이름이 길고 서술적인 보다 좋다.
- 서술적인 이름 사용시 설계가 뚜려해지므로 코드를 개선하기 쉬워진다.
- 이름은 일관성이 있어야 한다.(같은 문구, 명사, 동사 사용) 문체가 비슷하면 이야기를 순차적으로 풀어가기도 쉬워진다.

### 함수 인수
함수에서 이상적인 인수 개수는 0개(무항).
인수는 코드 이해에 방해가 되는 요소이므로 최선은 0개이고, 차선은 1개뿐인 경우이다.
출력인수(함수의 반환 값이 아닌 입력 인수로 결과를 받는 경우)는 이해하기 어려우므로 왠만하면 쓰지 않는 것이 좋겠다.

- 많이 쓰는 단항 형식(아래 3가지 유형이 아니라면 단항 함수는 가급적 피하는 것이 좋다.)
  - 인수에 질문을 던지는 경우
    - `boolean fileExists(“MyFile”);`  
  - 인수를 뭔가로 변환해 결과를 변환하는 경우
    - `InputStream fileOpen(“MyFile”);`  
  - 이벤트 함수일 경우
    - `passwordAttemptFailedNtimes(int attempts)`
    - 입력 인수만 있다. 출력 인수는 없다.
    - 함수 호출을 이벤트로 해석해 입력 인수로 시스템 상태를 변경한다.
    - 이벤트라는 사실이 코드에 명확하게 드러나야 하며 조심해서 사용한다.
- 플래그 인수
  - 플래그 인수는 추하다. 쓰지마라. bool 값을 넘기는 것 자체가 그 함수는 한꺼번에 여러가지 일을 처리한다고 공표하는 것과 마찬가지다.
  - `render(boolean isSuite)`는 `renderForSuite()`와 `renderForSingleTest()`라는 함수로 나누는게 낫다.
- 이항 함수
  - 단항 함수보다 이해하기가 어렵다.
  - 2개의 인수간의 자연적인 순서가 있어야함
  - Point 클래스의 경우에는 이항 함수가 적절하다. `Point p = new Point(x,y);`
  - 무조건 나쁜 것은 아니지만, 인수가 2개이니 만큼 이해가 어렵고 위험이 따르므로 가능하면 단항으로 바꾸도록
- 삼항 함수
  - 이항 함수보다 이해하기가 훨씬 어려우므로, 위험도 2배 이상 늘어난다.
  - 삼항 함수를 만들 때는 신중히 고려하라.
  - `assertEquals(1.0, amount, .001)`은 그리 음험하지 않은 삼항 함수다. 단 부동소수점 비교가 상대적이라는 사실은 언제든 주지할 중요한 사항이다.
- 인수 객체
  - 인수가 많이 필요할 경우, 일부 인수를 독자적인 클래스 변수로 선언할 가능성을 살펴보자
  - x,y를 인자로 넘기는 것보다 Point를 넘기는 것이 더 낫다.
  -`Circle makeCircle(double x, double y, double radius);` 보단 `Circle makeCircle(Point center, double radius);`가 낫다.
- 인수 목록
  - 때로는 String.format같은 함수들처럼 인수 개수가 가변적인 함수도 필요하다.
  - String.format의 인수는 List형 인수이기 때문에 이항함수라고 할 수 있다.
  - `public String format(String format, Object... args)`
- 동사와 키워드
  - 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야한다.
    - `writeField(name);`
  - 함수이름에 키워드(인수 이름)을 추가하면 인수 순서를 기억할 필요가 없어진다.
    - `assertExpectedEqualsActual(expected, actual);`  

### 부수 효과를 일으키지 마라!
부수효과는 거짓말이다. 함수에서 한가지를 하겠다고 약속하고는 **남몰래** 다른 짓을 하는 것이므로, 한 함수에서는 딱 한가지만 수행할 것!

아래 코드에서 `Session.initialize();`는 함수명과는 맞지 않는 부수효과이다.
{% highlight java linenos %}
public class UserValidator {
	private Cryptographer cryptographer;
	public boolean checkPassword(String userName, String password) { 
		User user = UserGateway.findByName(userName);
		if (user != User.NULL) {
			String codedPhrase = user.getPhraseEncodedByPassword(); 
			String phrase = cryptographer.decrypt(codedPhrase, password); 
			if ("Valid Password".equals(phrase)) {
				Session.initialize();
				return true; 
			}
		}
		return false; 
	}
}
{% endhighlight %}

여기서, 함수가 일으키는 부수 효과는 `Session.initialize();` 호출이다. `checkPassword` 함수는 이름만 봐서는 세션을 초기화한다는 사실이 드러나지 않는다. 이럴 경우 `checkPasswordAndInitializeSession`이라는 이름이 훨씬 좋다. 물론 함수가 '한 가지'만 한다는 규칙을 위반하지만.

- 출력 인수
  - 일반적으로 출력 인수는 피해야 한다.
  - 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택하라.

### 명령과 조회를 분리하라!
함수는 뭔가 객체 상태를 변경하거나, 객체 정보를 반환하거나 둘 중 하나다. 둘 다 수행해서는 안 된다.  
`public boolean set(String attribute, String value);`같은 경우에는 속성 값 설정 성공 시 true를 반환하므로 아래와 같은 괴상한 코드가 작성된다.  
`if(set(“username”, “unclebob”))...` 그러므로 명령과 조회를 분리해 혼란을 주지 않도록 한다.

### 오류 코드보다 예외를 사용하라!
try/catch를 사용하면 오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해 진다.
- Try/Catch 블록 뽑아내기
{% highlight java linenos %}
if (deletePage(page) == E_OK) {
	if (registry.deleteReference(page.name) == E_OK) {
		if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
			logger.log("page deleted");
		} else {
			logger.log("configKey not deleted");
		}
	} else {
		logger.log("deleteReference from registry failed"); 
	} 
} else {
	logger.log("delete failed"); return E_ERROR;
}
{% endhighlight %}

정상 작동과 오류 처리 동작을 뒤섞는 추한 구조이므로 if/else와 마찬가지로 블록을 별도 함수로 뽑아내는 편이 좋다.

{% highlight java linenos %}
public void delete(Page page) {
	try {
		deletePageAndAllReferences(page);
  	} catch (Exception e) {
  		logError(e);
  	}
}
{% endhighlight %}

위에서 `delete` 함수는 모든 오류를 처리한다. 그래서 코드를 이해하기 쉽다. 실제로 페이지 제거하는 함수는 아래의 `deletePageAndAllReferences`다. `deletePageAndAllReferences` 함수는 예외를 처리하지 않는다. 이렇게 정상 동작과 오류 처리 동작을 분리하면 코드를 이해하고 수정하기가 쉬워진다.

{% highlight java linenos %}
private void deletePageAndAllReferences(Page page) throws Exception { 
	deletePage(page);
	registry.deleteReference(page.name); 
	configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) { 
	logger.log(e.getMessage());
}
{% endhighlight %}

- 오류 처리도 한가지 작업이다.

- Error.java 의존성 자석
  - 오류를 처리하는 곳곳에서 오류코드를 사용한다면 enum class를 쓰게 되는데 이런 클래스는 의존성 자석이므로, 새 오류코드를 추가하거나 변경할 때 코스트가 많이 필요하다.
  - 그러므로 예외(새 예외는 Exception 클래스에서 파생된다.)를 사용하는 것이 더 안전하다.
{% highlight java linenos %}
public enum Error { 
	OK,
	INVALID,
	NO_SUCH,
	LOCKED,
	OUT_OF_RESOURCES, 	
	WAITING_FOR_EVENT;
}
{% endhighlight %}

### 반복하지 마라!
중복은 모든 소프트웨어에서 모든 악의 근원이므로 늘 중복을 없애도록 노력해야한다.

### 구조적 프로그래밍
다익스크라의 구조적 프로그래밍의 원칙을 따르자면 모든 함수와 함수 내 모든 블록에 입구와 출구가 하나여야 된다. 즉, 함수는 return문이 하나여야 되며, **루프 안에서 break나 continue를 사용해선 안된며 goto는 절대로, 절대로 사용하지 말자.** 함수가 클 경우에만 상당 이익을 제공하므로, 함수를 작게 만든다면 오히려 여러차례 사용하는 것이 함수의 의도를 표현하기 쉬워진다.

그런데 구조적 프로그래밍의 목표와 규율은 공감하지만 함수가 작다면 위 규칙은 별 이익을 제공하지 못한다. 함수가 아주 클 때만 상당한 이익을 제공한다. 그러므로 함수를 작게 만든다면 간혹 return, break, continue를 사용해도 괜찮다. 오히려 때로는 단일 입/출구 규칙보다 의도를 표현하기 쉬워진다.

### 함수를 어떻게 짜죠?
처음에는 길고 복잡하고, 들여쓰기 단계나 중복된 루프도 많다. 인수목록도 길지만, 이 코드들을 빠짐없이 테스트하는 단위 테스트 케이스도 만들고, 
코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다. 이 와중에도 코드는 항상 단위 테스트를 통과한다. 처음부터 탁 짜지지는 않는다.

### 결론
대가<sup>master</sup> 프로그래머는 시스템을 (구현할) 프로그램이 아니라 (풀어갈) 이야기로 여긴다.

## 4장 주석
### 주석은 나쁜 코드를 보완하지 못한다
### 코드로 의도를 표현하라!
### 좋은 주석
__ 법적인 주석
__ 정보를 제공하는 주석
__ 의도를 설명하는 주석
__ 의미를 명료하게 밝히는 주석
__ 결과를 경고하는 주석
__ TODO 주석
__ 중요성을 강조하는 주석
__ 공개 API에서 Javadocs

### 나쁜 주석
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

### 참고 문헌

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
