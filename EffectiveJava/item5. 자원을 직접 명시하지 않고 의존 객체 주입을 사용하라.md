### item5. 자원을 직접 명시하지 않고 의존 객체 주입을 사용하라

> Effective Java 공부

<br><br>

많은 클래스가 하나 이상의 자원에 의존한다. 
예를 들어 `SpellChecker`(맞춤법 검사기)는 `dictionary`에 의존하는데, 이런 클래스를 정적 유틸리티 클래스나 싱글톤으로 구현한 모습을 드물지 않게 볼 수 있다.

<br>

- 정적 유틸리티 클래스  
  객체 상태 정보(인스턴스 메소드, 인스턴스 변수)가 없고 정적 메소드와 정적 변수만을 제공하는 클래스.   
  클래스의 본래 목적인 데이터와 데이터 처리를 위한 로직의 캡슐화를 실행하지 않고 `비슷한 기능의 메소드와 상수를 모아서 캡슐화`한 것이다.  

- 싱글톤   
  전역 변수를 사용하지 않고 객체를 하나만 생성 하도록 하며, 생성된 객체를 어디에서든지 참조할 수 있도록 하는 패턴

<br>

```java
//정적 유틸리티를 잘못 사용한 예
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() {}

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
```

```java
//싱글톤을 잘못 사용한 예
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChecker(...) {}
    private static SpellChecker INSTANCE = new SpellChecker(...);

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
```

두 방식 모두에서 `SpellChecker`가 `dictionary`를 단 하나만 사용한다고 가정한다는 점에서 그리 훌륭하지 않다. 사전 하나로 모든 쓰임에 대응하기 어렵다.   
사전이 언어별로 있을 수도 있고, 특수 어휘용이나 테스트용 사전도 필요할 수 있다.

<br><br>

여기서 `SpellChecker`가 여러 사전을 사용할 수 있게 변경해보자.
간단하게 `dictionary`에서 `final`을 제거하고 사전 교체 메소드를 추가할 수도 있지만 이는 어색하고 오류를 내기 쉬우며 멀티쓰레드 환경에서는 쓸 수 없다.  
**사용하는 자원에 따라 동작이 달라지는 클래스에서는 정적 유틸리티 클래스나 싱글톤 방식이 적합하지 않다.**


<br><br>

`SpellChecker`가 여러 자원 인스턴스를 지원해야 하며 클라이언트가 원하는 `dictionary`를 사용해야 한다. 이는 **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**의 패턴으로 해결할 수 있다.

이는 의존 객체 주입의 한 형태로 `SpellChecker`를 생성할 때 의존 객체인 `dictionary`를 넣어주면 된다.

<br>

```java
public class SpellChecker {
    private final Lexicon dictionary;

    // SpellChecker를 생성할 때 의존 객체인 dictionary를 넣어주면 된다.
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
```

<br>

### 의존 객체 주입 패턴
의존 객체 주입 패턴은 아주 단순하여 수많은 프로그래머가 이 방식에 이름이 있다는 사실도 모른 채 사용해왔다. 즉 `dictionary`라는 딱 하나의 자원만 사용하지만 자원이 몇 개든 의존 관계가 어떻든 잘 작동한다.
또한 불변을 보장하여 같은 자원을 사용하려는 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다. 이는 생성자, 정적팩토리, 빌더 모두에 똑같이 응용할 수 있다.


<br>


이의 변형으로 생성자에 자원 팩토리를 넘겨주는 방식이 있다. 
여기서 팩토리란 호출할 때 마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다. 즉, 팩토리 메소드 패턴을 구현한 것이다.


<br><br>

클래스가 내부적으로 하나 이상의 자원에 의존하고 그 자원이 클래스 동작에 영향을 준다면 싱글톤과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 또한 이 자원들을 클래스가 직접 만들게 해서도 안된다.

#### 대신 필요한 자원을 생성자에 넘겨주자. 
**의존 객체 주입** 이라 하는 이 방법은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.

<br>
