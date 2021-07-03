---
title: "Clean Code - 로버트 C.마틴 #2 의미 있는 이름"
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
- 흡사한 이름은 사용하지 않도록 주의한다. XYZControllerForEfficientHandlingOfStrings, XYZControllerForEfficientStorageOfStrings라는 이름을 사용한다면? 차이를 알아챘는가?

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
            thereAreManyLetters(count);
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
위 처럼 접두어를 붙이는 것은 모듈의 재사용 관점에서도 좋지 못하다. 재사용하려면 이름을 바꿔야 한다.(eg, `GSDAccountAddress` 대신 `Address`라고만 해도 충분하다.)  

### 마치면서
> 두려워하지 말고 서로의 명명을 지적하고 고치자. 그렇게 하면 이름을 외우는 것에 시간을 빼앗기지 않고 "자연스럽게 읽히는 코드"를 짜는 데에 더 집중할 수 있다.
