---
title: "Effective Java"
categories:
    - 도서
tags:
    - 도서
last_modified_at: 2020-01-08T00:00:00+09:00
toc: true
toc_sticky: true
---

# 객체의 생성과 삭제

-   객체 생성 시점, 방법
-   객체 생성을 피해야 하는 경우, 방법
-   객체 삭제 보장 방법
-   객체 삭제 전 청소 작업 관리

## <a name="r1">**규칙1**</a> : 생성자 대신 정적 팩터리 메서드를 사용할 수 없는지 생각해 보라

{% highlight java linenos %}
public static Boolean valueOf(boolean b) {
return b ? Boolean.TRUE : Boolean.FALSE;
}
{% endhighlight %}

-   장점1 : 생성자와는 달리 정적 팩터리 메서드에는 이름(name)이 있다.
-   장점2 : 생성자와는 달리 호출할 때마다 새로운 객체를 생성할 필요는 없다.
    -   변경 불가능 클래스(**규칙15**)라면 이미 만들어 둔 객체를 활용할 수도 있고,
        만든 객체를 캐시(cache) 해놓고 재사용하여 같은 객체가 불필요하게 거듭 생성되는 일을 피할 수도 있다.
    -   개체 수를 제어하면 싱글턴 패턴을 따르도록 할 수 있다([**규칙3**](#r3))
    -   객체 생성이 불가능한 클래스를 만들 수도 있다(**규칙4**)
    -   변경이 불가능한 클래스의 경우(**규칙15**) 두 개의 같은 객체가 존재하지 못하도록 할 수도 있다.
        -   즉 a == b 일 때만 a.equals(b)가 참이 되도록 만들 수 있다.
        -   이렇게 구현된 클래스는 equals(Object) 대신 == 연산자를 사용하여 비교할 수 있으므로 성능이 향상된다.
        -   열거 자료형(**규칙30**)이 기법을 사용한다.
-   장점3 : 생성자와는 달리 반환값 자료형의 하위 자료형 객체를 반환할 수 있다.
    -   이는 public으로 선언되지 않은 클래스의 객체를 반환하는 API를 만들 수 있다.
    -   이 기법은 인터페이스 기반 프레임워크 구현에 적합(**규칙18**)
        -   인터페이스는 정적 메서드를 가질 수 없으므로, 관습상 반환값 자료형이 Type이라는 이름의 인터페이스인 정적 팩터리 메서드는
            Types라는 이름의 객체 생성 불가능 클래스(**규칙4**) 안에 둔다.
        -   클라이언트 코드는 반환된 객체의 실제 구현 세부사항이 아니라 인터페이스만 보고 작성하게 되는데, 일반적으로 바람직한 습관이다(**규칙52**)
    -   JDK 1.5부터 도입된 java.util.EnumSet(**규칙32**)에는 public으로 선언된 생성자가 없으며, 정적 팩터리 메서드뿐이다.
-   장점4 : 형인자 자료형(parameterized type) 객체를 만들 때 편하다.

-   단점1 : public이나 protected로 선언된 생성자가 없으므로 하위 클래스를 만들 수 없다.
-   단점2 : 정적 팩터리 메서드가 다른 정적 메서드와 확연히 구분되지 않는다.

## <a name="r3">**규칙3**</a> : private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라

# 열거형(enum)과 어노테이션

자바 1.5에는 새로운 참조 자료형(reference type)이 추가되었다. 열거형(enum type)이라 불리는 새로운 종류의 클래스와, 어노테이션 자료형이라 불리는 새로운 종류의 인터페이스가 그것이다.

## <a name="r30">**규칙30**</a> : int 상수 대신 enum을 사용하라

{% highlight java linenos %}
// int를 사용한 enum 패턴
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
{% endhighlight %}

{% highlight java linenos %}
// enum 자료형
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
{% endhighlight %}

자바의 enum 자료형 이면에 감춰진 기본적 아이디어는 단순하다. 열거 상수(enumeration constant)별로 하나의 객체를 public static final 필드 형태로 제공하는 것이다.

enum 자료형은 실질적으로는 final로 선언된 것이나 마찬가지인데, 클라이언트가 접근할 수 있는 생성자가 없기 때문이다. 즉 enum 자료형은 싱글턴 패턴을 일반화한 것으로([**규칙3**](#r3)), 싱글턴 패턴은 본질적으로 보면 열거 상수가 하나뿐인 enum과 같다.

enum 자료형은 컴파일 시점 형 안전성(compile-time type safety)을 제공한다. Apple 형의 인자를 받는다고 선언한 메서드는 반드시 Apple 값 세 개 가운데 하나만 인자로 받는다.

{% highlight java linenos %}
// 데이터와 연산을 구비한 enum 자료형
public enum Planet {  
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 킬로그램 단위
    private final double radius;         // 미터 단위
    private final double surfaceGravity; // m / s^2

    // 중력 상수 in m^3 / kg s^2
    private static final double G = 6.67300E-11;

    // Constructor
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    } 
}
{% endhighlight %}