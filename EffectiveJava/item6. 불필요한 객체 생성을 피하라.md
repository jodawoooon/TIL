### item6. 불필요한 객체 생성을 피하라

> Effective Java 3/E 공부

<br><br>

똑같은 기능의 객체를 매번 생성하기 보다는 객체 하나를 재사용하는 것이 나을 때가 많다.

<br> 

극단적인 예를 들어보면,

```java
String str = new String("TEST");
```
이는 실행될 때 마다 인스턴스를 새로 만든다. 이 문장이 반복문이나 자주 호출되는 메소드 안에 있다면 쓸데없는 String 인스턴스가 수백만개 만들어 질 수 있다.

```java
String str = "TEST";
```
이와 같이 코드를 바꾸면 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다.  
또한 이 방식을 사용한다면 같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.

<br><br>

### 생성자 대신 정적 팩토리 메소드를 제공
생성자 대신 정적 팩토리 메소드를 제공하는 불변 클래스에서는 정적 팩토리 메소드를 사용해 불필요한 객체 생성을 피할 수 있다.   
즉, `Boolean(String)` 생성자 대신에 `Boolean.valueOf(String)` 팩토리 메소드를 사용하는 것이 좋다.  

=> 자바 9에서는 `Boolean()` 생성자가 사용 자제(deprecated) API로 지정되었다.

<br>

생성자는 호출할 때마다 새로운 객체를 만들지만, 팩토리 메소드는 그렇지 않다. 

불변 객체가 아니여도 사용 중에 변경되지 않을 것이라면 재사용할 수 있다. 

<br><br>

### 생성 비용이 비싼 객체
생성 비용이 아주 비싼 객체도 있다. 이러한 것들이 반복해서 필요한 경우에는 캐싱해서 재사용하길 권한다. 무엇이 비싼 객체인지는 명확하게는 알 수 없다.

예를 들어 주어진 문자열이 유효 로마 숫자인지 판단하는 메소드를 정규표현식을 활용해 작성해보면

```java
static boolean isRomanNumeral(String s) {
    return s.matches(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$"
    );
}
```
위와 같이 구현된다. 여기서 사용한 **`String.matched`는 정규 표현식으로 문자열 형태를 확인하는 가장 쉬운 방식이지만, 성능이 중요한 상황에서 반복해 사용하기는 적합하지 않다.**

`String.matched`가 내부에 만드는 정규 표현식용 `Pattern` 인스턴스는 한번 쓰고 버려지기 때문에 가비지 컬렉션 대상이 된다.  
Pattern은 정규표현식에 해당하는 유한 상태 머신을 만들기 때문에 인스턴스 생성 비용이 높다.

<br>

이를 개선하려면 `Pattern` 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고 나중에 위 메소드(`isRomanNumeral`)가 호출 될 때마다 이 인스턴스를 재사용해야한다.

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$"
    );

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

위와 같이 변경하면 `isRomanNumeral`이 호출 될 때 성능을 개선할 수 있다.  
또한 성능 뿐 아니라 `Pattern` 인스턴스를 `static final로 끄집어 내고 이름을 지어주었기 때문에 코드의 의미가 잘 드러나며 명확해 졌다.`
다만 개선된 `isRomanNumeral` 방식의 클래스가 초기화 된 후 이 메소드를 한번도 호출하지 않으면 쓸데없이 `ROMAN` 필드를 초기화한 것이다. 이러한 효율성 문제도 생각해봐야 할 것이다.


<br><br>

### 오토박싱

불필요한 객체를 만들어내는 예로 오토박싱이 있다.
오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어서 사용할 때 자동으로 상호 변환해주는 기술이다. 
이는 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주지는 않는다. 의미 상으로는 별 차이가 없지만 성능은 그렇지 않다.

```java
private static long sum() {
    Long sum = 0L;

    for (long i = 0 ; i <= Integer.MAX_VALUE; i++) 
        sum+= i;

    return sum;
}
```

위 코드는 모든 양의 정수의 총합을 구하는 메소드이다.   
이 프로그램은 정답을 출력하긴 하지만 매우 느리다.  
이 코드에는 **오타**가 있기 때문이다.

바로 `sum` 변수를 `long`이 아닌 `Long` 타입으로 선언하여 불필요한 `Long` 인스턴스가 약 2^31개나 만들어진다. 
단순하게 `sum`의 타입을 `long`으로 바꾸어만 줘도 훨씬 속도가 개선된다.


**박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 존재하지 않도록 주의해야 한다**

<br><br>

위에서 공부한 내용들을 객체 생성은 비싸므로 피하자!! 로 이해하면 안된다.  
**프로그램의 명확성, 간결성, 기능을 위해 객체를 생성하는 것은 일반적으로는 좋은 것이다.**  
반대로 단순히 객체 생성을 피하고자 나만의 자체 객체 풀을 만들면 안된다. 

```
객체 풀 ? 

객체를 매번 할당, 해제하지 않고 고정 크기 풀에 들어 있는 객체를 재사용함으로써 메모리 사용 성능을 개선한다.
재사용 가능한 객체들을 모아놓은 것이 객체 풀 클래스이다.
```

일반적으로 자체 객체 풀은 코드를 헷갈리게 하고 메모리 사용량을 늘리며 성능을 떨어트린다. 요즘 JVM의 가비지 컬렉터는 잘 최적화 되어 있어서 가벼운 객체용을 다룰 때에는 직접 만든 객체 풀보다 빠르다.