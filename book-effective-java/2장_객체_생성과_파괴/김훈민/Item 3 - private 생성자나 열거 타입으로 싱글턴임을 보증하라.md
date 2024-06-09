# Item 3 - private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 의미한다.

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려운데, 싱글턴 인스턴스를 가짜로 대체할 수 없기 때문이다.

싱글턴을 만드는 방법은 보통 둘 중 하나이다.

## 1. public static 멤버가 final 필드인 방식

```java
public class Printer {
    public static final Printer INSTANCE = new Printer();
    private Printer() { ... }

    public void print() { ... }
}
```

public, protected 생성자가 없으므로 클래스가 초기화될때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.

예외적으로, 권한이 있는 클라이언트는 리플렉션 API의 AccessibleObject.setAccessible 을 사용해 private 생성자를 호출할 수 있긴하다.

이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.

### 장점

1. 싱글턴임이 API에 명백히 드러난다.

⇒ 코드를 보면 public static 필드가 final 이니 절대로 다른 객체를 참조할 수 없다는 생각이 들게 된다.

1. 간결하다

## 2. 정적 팩토리 메서드를 public static 멤버로 제공한다.

```java
public class Printer {
    private static final Printer INSTANCE = new Printer();
    private Printer() { ... }
    public static Printer getInstance() { return INSTANCE; }

    public void print() { ... }
}
```

Printer.getInstance() 는 항상 같은 객체의 참조를 반환하니 싱글턴을 보장한다.

### 장점

1. API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점
2. 정적 팩토리를 제네릭 싱글턴 팩토리로 만들 수 있다는 점
3. 정적 팩토리의 메서드 참조를 공급자(supplier)로 사용할 수 있다는 점

위 장점들이 굳이 필요하지 않다면 public 필드 방식이 좋다고 한다.

### 유의할 점

그런데 1번과 2번 방식으로 만들어진 싱글턴 클래스를 직렬화 하려면 단순히 Serializable을 구현한다고 선언하는 것으로는 부족하다.

모든 인스턴스 필드에 transient를 선언하고, readResolve 메서드를 제공해야 한다.

만약 그렇게 하지 않으면, 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

직렬화(Serialization)와 역직렬화(Deserialization)는 객체를 바이트 스트림으로 변환하고, 바이트 스트림을 다시 객체로 복원하는 과정이고, 기본적으로, 역직렬화는 새로운 객체를 생성하는 과정이기 때문에, 역직렬화된 객체가 기존의 싱글턴 인스턴스와 다른 새로운 인스턴스가 될 수도 있다.

예시 코드를 보면

```java
import java.io.ObjectStreamException;
import java.io.Serializable;

public class Singleton implements Serializable {
    private static final long serialVersionUID = 1L;
    private static volatile Singleton instance;

    private Singleton() { ... }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

		// 싱글턴임을 보장해주는 readResolve 메서드
    private Object readResolve() throws ObjectStreamException 
		    // 진짜 Singleton을 반환하고, 가짜는 가바지 컬렉터에 맡긴다.
        return getInstance();
    }
}

```

위 과정에서 readResolve 메서드는 역직렬화된 객체를 대체할 객체를 반환한다.

이는 싱글턴 인스턴스가 역직렬화 과정에서 새로운 인스턴스로 생성되지 않고 기존의 인스턴스를 유지하도록 보장할수있게 한다.

---

그리고 싱글턴을 만드는 세번째 방법이 있다.

## 3. 열거 타입 방식의 싱글턴

저자는 이 방법을 추천한다.

```java
public enum Printer {
    INSTANCE;

    public void print() { ... }
}
```

앞의 예제들에 비해서 더 간결하고, 추가 노력 없이 직렬화도 할 수 있고, 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 막아준다.

**대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법**이고 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다는 단점이 있다.

이 방식에 대해서 더 찾아보았는데

Java의 열거형(enum)은 기본적으로 직렬화가 지원된다고 한다. 당연히 직렬화 역직렬화 과정에서 단일 인스턴스임을 보장하기도 하고 사용할때는

```java
public enum Singleton {
    INSTANCE;
    public void doSomething() {
        // 싱글턴 객체의 메서드
        System.out.println("Singleton is doing something.");
    }
}
```

이렇게 만들고 클라이언트 코드에서는

```java
public class Main {
    public static void main(String[] args) {
        Singleton singleton = Singleton.INSTANCE;
        singleton.doSomething();
    }
}
```

이렇게 사용하면 된다.