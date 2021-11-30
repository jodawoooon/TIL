# item3. private 생성자나 열거 타입으로 싱글톤임을 보증하라
> Effective Java 공부

<br>

## 싱글톤(Singleton)이란 ?
인스턴스를 오직 하나만 생성할 수 있는 클래스

<br>

**클래스를 싱글톤으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어렵다.**   
생성 방식이 제한적이기 때문에 Mock(가짜) 객체로 대체하기 어려우며 동적으로 객체를 주입하기도 어렵다.  
테스트가 개발의 핵심인데, 테스트 코드를 작성하기 어렵다는 점은 큰 단점이 된다.

<br><br>

보통 싱글턴을 만드는 방식은 둘 중 하나이다.
두 방식 모두 생성자는 private으로 감춰두고 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 둔다.

<br>

## 1. public static final 필드 방식의 싱글톤

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}

    public void leaveTheBuilding() {...}
```
private 생성자는 `public static final` 필드인 `Elvis.INSTANCE`를 초기화할 때 딱 한 번만 호출된다.
public 또는 protected 생성자가 없으므로 Elvis 클래스가 초기화 될 때 만들어진 인스턴스가 전체에서 하나 뿐임이 보장된다.

<br>

다만 예외로 권한이 있는 경우 리플렉션 API인 `AccessibleObject.setAccessible`을 사용해 private 생성자를 호출할 수 있다.   
이를 방어하기 위해 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {
        if(INSTANCE != null) throw new RuntimeException("Exceptions .... ");
    }

    public void leaveTheBuilding() {...}
```

<br><br>

## 2. 정적 팩토리 방식의 싱글톤

정적 팩토리 메소드를 `public static` 멤버로 제공한다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}
    
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {...}
```

`Elvis.getInstance`는 항상 같은 객체의 참조를 반환하므로 마찬가지로 인스턴스가 전체 시스템에서 하나 뿐임이 보장된다.   
다만 1과 같이 리플렉션 API를 통한 예외는 똑같이 적용된다.

<br><br>

1번 방식(`public static final 필드 방식`)의 경우의 장점은 해당 클래스가 싱글턴임이 API에 명백하게 드러난다는 것이다. `public static`이 `final`이니 절대로 다른 객체를 참조할 수 없게 된다. 또한 간결하다는 장점도 크다.

<br>

2번 방식(`정적 팩토리 방식`)의 경우에는 API를 바꾸지 않고도 싱글톤이 아니게 변경할 수 있다는 점이 장점이다. 유일한 인스턴스를 반환하던 팩토리 메소드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다. 
그 외에도 정적 팩토리를 `제네릭 싱글톤 팩토리`로 만들 수 있다는 것과 정적 팩토리의 메소드 참조를 `Supplier`로 사용할 수 있다는 것이다.

(`Supplier`: get메서드로 아무 type이나 리턴할 수 있는 인터페이스.)  
ex)
```java
Supplier<Elvis> elvisSupplier = Elvis::getInstance;
Elvis elvis = elvisSupplier.get();
```
이와 같은 장점들이 필요 없다면 1번 방식(`public static final 필드 방식`)이 좋다.

<br><br>

다만 위 방법으로 만든 싱글턴 클래스를 직렬화하려면 단순히 `Serializable`을 구현한다고 선언하는 것으로는 부족하다. 모든 인스턴스를 `transient`라고 선언하고 readResolve 메소드를 제공해야 한다. 
```java
private Obejct readResolve() {
	return INSTANCE;
}
```
이러지 않으면 직렬화된 인스턴스를 역직렬화 할 때마다 새로운 인스턴스가 생성된다. 즉 가짜 인스턴스가 생긴다는 것이다.

<br><br>

## 3. 열거 타입 방식의 싱글톤

```java
public enum Elvis {
	INSTANCE; 

	public void leaveTheBuilding() { ... }
}
```

원소가 하나인 Enum 타입을 선언하는 것이다.
1번 방식과 비슷하지만 더 간결하고 추가적 노력없이 직렬화 할 수 있다. 또한 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽하게 막아준다.

**대부분 상황에서는 원소가 하나뿐인 Enum타입이 싱글톤을 만드는 가장 좋은 방법이다.**   
다만 만들려는 싱글톤이 Enum 외에 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.