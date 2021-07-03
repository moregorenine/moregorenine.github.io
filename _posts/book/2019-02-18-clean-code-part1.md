---
title: "Clean Code - 로버트 C.마틴 #1 깨끗한 코드"
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
