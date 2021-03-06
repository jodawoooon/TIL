### item7. 다 쓴 객체 참조를 해제하라

> Effective Java 3/E 공부

<br><br>

### 가비지 컬렉터 언어의 메모리 누수

Java의 **가비지 컬렉터**는 다 쓴 객체를 알아서 회수해가기 때문에 메모리 관리에 신경 쓰지 않아도 된다고 오해할 수 있다. 그러나 이는 명백한 오류이다.

아래 코드는 스택을 간단하게 구현한 코드이다.

```java
public class Stack {
    private Object[] elements;
    private int size;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e){
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop(){
        if(size==0)
            throw new EmptyStackException();
        return elements[--size];
    }

    private void ensureCapacity() {
        /**
         * 원소를 위한 공간을 적어도 하나 이상 확보. 
         * 배열 크기를 늘려야 할 때마다 대략 2배씩 늘린다
         */
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

<br>


이 코드엔 **메모리 누수**가 존재한다. 스택이 커졌다가 줄어들 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않고있다. 이 스택이 그 객체들의 **다 쓴 참조**를 가지고 있다.    

> **다 쓴 참조**?  
> 앞으로 다시 쓰지 않을 참조를 뜻한다. 

<br>

위 코드에서는 `elements` 배열의 **활성 영역** 밖의 참조들이 모두 여기에 해당한다.  

> **활성 영역**?  
> 인덱스가 size보다 작은 원소들의 영역

<br>

가비지 컬렉션 언어에서는 **의도치 않게 객체를 살려두는 메모리 누수를 찾기 까다롭다.**  
객체 참조 하나를 살려두면 **가비지 컬렉터**는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체(그리고 또 그 객체들이 참조하는 모든 객체들)을 회수해가지 못한다. 그래서 단 몇 개의 객체가 매우 많은 객체를 회수되지 못하게 할 수 있고 잠재적으로 성능에 악영향을 줄 수 있다.

<br><br>

### 객체 참조 해제

메모리 누수의 해법은 간단하다. 해당 참조를 다 썼을 때 `null` 처리하면 된다.

위 예시 코드에서 각 원소의 참조가 더 이상 필요없어지는 때는 언제일까?

<br>

바로 스택에서 꺼내질 때, `pop` 과정이다.
이에 따라 `pop` 메소드를 다시 구현해보면,

```java
    public Object pop(){
        if(size==0)
            throw new EmptyStackException();
        Object res = elements[--size];
        elements[size] = null; // 다 쓴 참조 객체 해제
        return res;
    }
```

다 쓴 참조를 `null` 처리하게 되면 메모리상의 이점뿐만 아니라 오류도 막을 수 있다.

`null` 처리한 참조를 실수로 사용하려 하면 프로그램은 즉시 `NullPointerException`을 던진다. 이는 즉, `null` 처리를 하지 않았다면 뭔가 잘못된 일이 Exception 처리 되지 않았을 것이라는 뜻이다...

<br><br>


### 그러면 다 쓴 모든 객체를 null 처리 해야 할 까?

답은 '그럴 필요는 없다'이다!  
이는 프로그램을 필요 이상으로 지저분하게 만들 뿐이다. **객체 참조를 `null` 처리 하는 일은 예외적인 경우여야 한다.**  
다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것이다. 변수의 범위를 최소가 되게 정의했다면 이는 자연스럽게 이뤄진다. 

<br><br>

### 그렇다면 null 처리는 언제 해야 할까?

`Stack` 클래스 왜 메모리 누수에 취약할까?  
바로 스택은 자신의 메모리를 직접 관리하기 때문이다. 스택은 `elements` 배열로 저장소 풀을 만들어 원소를 관리한다. 배열에 **활성 영역**에 속한 원소들이 사용되고 **비활성 영역**은 쓰이지 않는다. 여기서 문제는 **가비지 컬렉터가 이 사실을 알 수 없다는 것이다.**

<br>

**가비지 컬렉터** 입장에서는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체이다. 비활성 영역에 객체가 필요 없다는 것은 프로그래머만 알 수 있다. **따라서 프로그래머는 비활성 영역이 되는 순간 `null` 처리를 해서 가비지 컬렉터에 알려야 한다!**

<br><br>

### 또 다른 메모리 누수의 주범, 캐시

캐시 역시 메모리 누수를 일으키는 주범이다.
객체 참조를 캐시에 넣고 이를 까먹은 채 객체를 다 쓴 뒤에도 그냥 놔두눈 경우를 자주 접할 수 있다.

이 문제의 해법은 여러 가지이다.

#### 먼저 캐시 외부에서 Key를 참조하는 동안만 캐시가 필요한 상황이라면?  
`WeakHashMap`을 사용해 캐시를 만든다.
다 쓴 엔트리가 자동으로 제거된다. 단, 이러한 상황에서만 유용한 방식이다.

#### 보통은 캐시 엔트리에 유효 기간을 정확히 정의하기 어렵다.
때문에 시간이 지날수록 엔트리의 가치를 떨어트리는 방식을 흔히 사용한다. 이 때 쓰지않는 엔트리를 이따금 청소해야한다. (`ScheduledThreadPoolExecutor` 같은) 백그라운드 스레드를 사용하거나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법이 있다.  
`LinkedHashMap`은 `removeEldestEntry`를 써서 후자의 방식을 사용한다.  


<br><br>

### 리스너 혹은 콜백
메모리 누수의 세번째 주범은 리스너 혹은 콜백이라 부르는 것이다.  
클라이언트가 콜백을 등록만 하고 명확히 해지하지 않으면 콜백은 계속 쌓여간다. 이럴 때 **콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거한다.** 예를 들어 `WeakHashMap`에 키로 저장하면 된다.

<br><br>

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 이용해야만 발견되기도 한다.
따라서 이는 예방법을 익혀 두는 것이 가장 중요하다!