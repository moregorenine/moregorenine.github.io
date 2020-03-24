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

## <a name="r1">규칙1</a> : 생성자 대신 정적 팩터리 메서드를 사용할 수 없는지 생각해 보라

{% highlight java linenos %}
public static Boolean valueOf(boolean b) {
return b ? Boolean.TRUE : Boolean.FALSE;
}
{% endhighlight %}

-   장점1 : 생성자와는 달리 정적 팩터리 메서드에는 이름(name)이 있다.
-   장점2 : 생성자와는 달리 호출할 때마다 새로운 객체를 생성할 필요는 없다.
    -   변경 불가능 클래스(**규칙15**)라면 이미 만들어 둔 객체를 활용할 수도 있고,
        만든 객체를 캐시(cache) 해놓고 재사용하여 같은 객체가 불필요하게 거듭 생성되는 일을 피할 수도 있다.
    -   개체 수를 제어하면 싱글턴 패턴을 따르도록 할 수 있다([규칙3](#r3))
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

## <a name="r3">규칙3</a> : private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라

싱글턴은 객체를 하나만 만들 수 있는 클래스다.
JDK 1.5 이전에는 싱글턴을 구현하는 방법이 두 가지였다.
두 방법 다 생성자는 private 로 선언하고, 싱글턴 객체는 정적(static) 멤버를 통해 이용한다.

- 첫 번째 방법의 경우, 정적 멤버는 final로 선언한다.
{% highlight java linenos %}
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
{% endhighlight %}

생성자를 private 로 선언하고 public static ClassType getInstance() { return INSTANCE; }
위의 두 방법 모두 private 생성자를 reflection 통해 부를 수 있음에 주의해야 한다.

성능에 대한 것은, 두번째 방법이 method를 이용하니까 첫번째 방법의 성능이 더 좋다고 생각할 수 있지만, 최신 JVM은 static fiactory method호출을 거의 항상 inline으로 처리하기 때문에 성능상의 차이는 거의 없다고 봐도 된다.

위의 두 방법으로 구현한 싱글턴 클래스를 직렬화하려면 implements Serializable을 추가하는 것으로는 부족. 모든 필드를 transient로 선언, readResolve method 추가해야함. (그렇게 하지 않으면 deserialize할 때 새로운 객체가 생기게 된다.)

원소가 하나뿐이 enum
JDK 1.5부터는 싱글턴 구현할 때 새로운 방법을 사용할 수 있는데 바로 원소가 하나뿐인 enum 자료형을 쓰는 것!
직렬화가 자동으로 처리되고, 리플렉션을 통한 공격에도 안전하다고.
저자는 원소가 하나뿐인 enum 자료형이야 말로 싱글턴을 구현하는 가장 좋은 방법이라고 주장한다.

# 열거형(enum)과 어노테이션

자바 1.5에는 새로운 참조 자료형(reference type)이 추가되었다. 열거형(enum type)이라 불리는 새로운 종류의 클래스와, 어노테이션 자료형이라 불리는 새로운 종류의 인터페이스가 그것이다.

## <a name="r30">규칙30</a> : int 상수 대신 enum을 사용하라

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

자바의 enum 자료형 이면에 감춰진 기본적 아이디어는 단순하다.
열거 상수(enumeration constant)별로 하나의 객체를 public static final 필드 형태로 제공하는 것이다.

enum 자료형은 실질적으로는 final로 선언된 것이나 마찬가지인데, 클라이언트가 접근할 수 있는 생성자가 없기 때문이다.
즉 enum 자료형은 싱글턴 패턴을 일반화한 것으로([**규칙3**](#r3)), 싱글턴 패턴은 본질적으로 보면 열거 상수가 하나뿐인 enum과 같다.

enum 자료형은 컴파일 시점 형 안전성(compile-time type safety)을 제공한다.
Apple 형의 인자를 받는다고 선언한 메서드는 반드시 Apple 값 세 개 가운데 하나만 인자로 받는다.

{% highlight java linenos %}
// 풍부한 기능을 갖춘 enum 자료형 예제
// 데이터와 연산을 구비한 enum 자료형
public enum Planet {  
 MERCURY(3.302e+23, 2.439e6),
VENUS (4.869e+24, 6.052e6),
EARTH (5.975e+24, 6.378e6),
MARS (6.419e+23, 3.393e6),
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

**enum 상수에 데이터를 넣으려면 객체 필드(instance field)를 선언하고 생성자를 통해 받은 데이터를 그 필드에 저장하면 된다.**
enum은 원래 변경경 불가능하므로(immutable) 모든 필드는 final로 선언되어야 한다([규칙15](#r15)). 필드는 public으로 선언할 수도 있지만, private로 선언하고 public 접근자(accessor)를 두는 편이 더 낫다.([규칙14](#r14))

- 모든 enum 상수를 선언된 순서대로 저장하는 배열을 반환하는 static values 메서드가 기본적으로 정의되어 있다.
- enum 상수 이름을 쉽게 출력할 수 있도록 하는 toString 메서드도 이미 갖추어져 있다.

enum 상수에 가능한 연산 가운데는 enum이 정의된 클래스나 패키지 안에서만 사용되어야 하는 것이 있을 수 있다.
그런 연산은 private나 package-private 메서드로 선언하는 것이 최선이다.
그런 메서드들은 해당 enum이 포함된 클래스나 패키지가 입력으로 받은 enum 상수를 적절히 처리하기 위해서만 쓰인다. 일반 클래스와 마찬가지로, enum에 정의한 메서드를 클라이언트에게까지 공개할 특별한 이유가 없다면 private나 package-private로 선언하라([규칙13](#r13))

일반적으로 유용하게 쓰일 enum이라면, 최상위\*(top-level) public 클래스로 선언해야 한다.
특정한 최상위 클래스에서만 쓰이는 enum이라면 해당 클래스의 멤버 클래스로 선언해야 한다.([규칙22](#r22))

{% highlight java linenos %}
// 상수들이 제각기 다른 방식으로 동작 enum 자료형 예제
// 자기 값에 따라 분기하는 Enum 자료형
public enum Operation {
PLUS, MINUS, TIMES, DIVIDE;

       // 'this' 상수가 나타내는 산술 연산 실행
       double apply(double x, double y) {
           switch(this) {
               case PLUS:   return x + y;
               case MINUS:  return x - y;
               case TIMES:  return x * y;
               case DIVIDE: return x / y;
           }
           throw new AssertionError("Unknown op: " + this);
       }

}
{% endhighlight %}

-   단점
    -   throw 문 없이는 컴파일이 되지 않을 것
    -   깨지기 쉬운 코드, 새로운 enum 상수를 추가할 때 switch문에 case를 추가하지 않아도 이 코드는 컴파일 된다.

상수별 메서드 구현이란 더 좋은 방법이 있다.

{% highlight java linenos %}
// 상수별 메서드 구현을 이용한 enum 자료형
public enum Operation {
    PLUS { double apply(double x, double y){return x + y;} },
    MINUS { double apply(double x, double y){return x - y;} },
    TIMES { double apply(double x, double y){return x \* y;} },
    DIVIDE { double apply(double x, double y){return x / y;} };
    
    abstract double apply(double x, double y);
}
{% endhighlight %}

이 enum에 새로운 상수를 추가할 때는 apply 메서드 구현을 잊을 가능성이 거의 없다.
설사 잊더라도 컴파일러가 오류를 내 줄 것이다. enum 자료형의 abstract 메서드는 모든 상수가 반드시 구현해야 하기 때문이다.

상수별 메서드 구현은 상수별 데이터와도 혼용될 수 있다.
예를 들어, 아래의 Operation은 toString을 재정의하여 연산을 나타내는 기호가 반환될 수 있도록 하고 있다.

{% highlight java linenos %}
// 상수별로 클래스 몸체와 별도 데이터를 갖는 enum 자료형
public enum Operation {
   PLUS("+") {
       double apply(double x, double y) { return x + y; }
   },
   MINUS("-") {
       double apply(double x, double y) { return x - y; }
   },
   TIMES("*") {
       double apply(double x, double y) { return x * y; }
   },
   DIVIDE("/") {
       double apply(double x, double y) { return x / y; }
   };
   private final String symbol;
   Operation(String symbol) { this.symbol = symbol; }
   @Override public String toString() { return symbol; }
   abstract double apply(double x, double y);
}
{% endhighlight %}

toString을 재정의하면 아래처럼 쓸모 있을 때가 있다.

{% highlight java linenos %}
public static void main(String[] args) {
   double x = Double.parseDouble(args[0]);
   double y = Double.parseDouble(args[1]);
   for (Operation op : Operation.values())
       System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
{% endhighlight %}

enum 자료형에는 자동 생성된 valueOf(String) 메서드가 있는데, 이 메서드는 상수의 이름을 상수 그 자체로 변환하는 역할을 한다.
enum 자료형의 toString 메서드를 재정의할 경우에는 fromString 메서드를 작성해서
toString이 뱉어낸느 문자열을 다시 enum 상수로 변환할 수단을 제공해야 할지 생각해 봐야 한다.

{% highlight java linenos %}
// enum 자료형에 대한 fromString 메서드 구현
private static final Map<String, Operation> stringToEnum
   = new HashMap<String, Operation>();
static { // 상수 이름을 실제 상수로 대응시키는 맵 초기화
   for (Operation op : values())
       stringToEnum.put(op.toString(), op);
}
// 문자열이 주어지면 그에 대한 Operation 상수 반환, 잘못된 문자열이면 null 반환.
public static Operation fromString(String symbol) {
   return stringToEnum.get(symbol);
}
{% endhighlight %}

enum 상수끼리 공유하는 코드가 필요할 때
{% highlight java linenos %}
// enum 상수에 따라 분기하는 switch 문을 이용해서 코드 공유 - 좋은 방법인가?
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    private static final int HOURS_PER_SHIFT = 8;
    
    double pay(double hoursWorked, double payRate) {
        double basePay = hoursWorked * payRate;
        
        double overtimePay; // 초과근무수당 계산
        switch(this) {
            case SATURDAY: case SUNDAY:
                overtimePay = hoursWorked * payRate / 2;
                break;
            default: // Weekdays
                overtimePay = hoursWorked <= HOURS_PER_SHIFT ?
                    0 : (hoursWorked - HOURS_PER_SHIFT) * payRate / 2; 
        }
        
        return basePay + overtimePay;
    }
}
{% endhighlight %}

간결하지만 유지보수 관점에선 위험한 코드다.
enum에 새로운 상수(휴가 등)를 추가했을 때, case 추가하는 것을 잊어도 컴파일 오류가 나지 않기 때문이다.
좋은 방법은 다음과 같다. (strategy enum pattern)

{% highlight java linenos %}
// 정책 enum 패턴
enum PayrollDay {
    MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY), THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;
    PayrollDay(PayType payType) { this.payType = payType; }
    
    double pay(double hoursWorked, double payRate) {
        return payType.pay(hoursWorked, payRate);
    }
    
    // 정책 enum 자료형
    private enum PayType {
        WEEKDAY {
            double overtimePay(double hours, double payRate) {
                return hours <= HOURS_PER_SHIFT ? 0 :
                    (hours - HOURS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            double overtimePay(double hours, double payRate) {
                return hours * payRate / 2;
            }
        };
        
        private static final int HOURS_PER_SHIFT = 8;
        
        abstract double overtimePay(double hrs, double payRate);
        
        double pay(double hoursWorked, double payRate) {
            double basePay = hoursWorked * payRate;
            return basePay + overtimePay(hoursWorked, payRate);
        }
    }
}
{% endhighlight %}

enum에서 switch 문을 사용해 상수별로 다르게 동작하는 코드를 만드는 것이 바람직하지 않다면,
switch 문은 대체 어디에 적합한가?
외부(external) enum 자료형 상수별로 달리 동작하는 코드를 만들어야 할 때는 enum 상수에 switch 문을 적용하면 좋다.

{% highlight java linenos %}
// 기본 enum 자료형에 없는 메서드를 switch 문을 사용해 구현한 사례
public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS:      return Operation.MINUS;
        case MINUS:     return Operation.PLUS;
        case TIMES:     return Operation.DIVIDE;
        case DIVIDE:    return Operation.TIMES;
        default:        throw new AssertionError("Unknown op: " + op);
    }
}
{% endhighlight %}



{% highlight java linenos %}
{% endhighlight %}