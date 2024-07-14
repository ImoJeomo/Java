## 아이템 10. `equals`는 일반 규약을 지켜 재정의하라

> 꼭 필요한 경우가 아니면 `equals`를 재정의하지 말자. 해야한다면 5가지 규약을 꼭 지키자.

### `equals`를 재정의하지 않는 상황들

#### 1. 각 인스턴스가 본질적으로 고유하다.

- 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 이에 해당한다.
  - `Thread`

#### 2. 인스턴스의 논리적 동치성(logical equality)를 검사할 일이 없다.

#### 3. 상위 클래스에서 정의한 `equals`가 하위 클래스에도 딱 들어맞을 때

#### 4. 클래스가 `private`이거나 `package-private`이고 `equals`를 호출할 일이 없을 때

- `equals`의 호출을 막기 위해서 다음과 같이 코드를 작성해도 좋다.

```java
@Override
public boolean equals(Object o) {
    throw new AssertionError();
}
```

### `equals`를 재정의해야 하는 상황들

- 객체 식별성(object identity; 두 객체가 물리적으로 같은지)이 아니라 논리적 동치성을 확인해야하는 필요가 있을 때
  - 주로 값 클래스
  - `Integer`, `String` ...
  - 값이 같은 인스턴스가 둘 이상 만들어지지 않음이 보장되는 인스턴스 통제 클래스는 제외

### `equals` 재정의 규약

#### 반사성(reflexivity)

> null이 아닌 모든 참조 값 x에 대해, `x.equals(x)`는 `true`이다.

#### 대칭성(symmetry)

> null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)`가 `true`면 `y.equals(x)`도 `true`다.

#### 추이성(transitivitiy)

> null이 아닌 모든 참조 값 x, y, z에 대해, `x.equals(y)`가 `true`고 `y.equals(z)`가 `true`면 `x.equals(z)`도 `true`다.

#### 일관성(consistency)

> null이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`를 반복해서 호출하면 항상 `true`를 반환하거나 항상 `false`를 반환한다.

- `equals`의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.
- `equals`는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.

#### null 아님

> null이 아닌 모든 참조 값 x에 대해, `x.equals(null)`은 `false`다.

- `instanceof`에서 `null`도 걸러진다.

### `equals` 구현 절차

#### 1. `==` 연산자를 사용하여 입력이 자기 자신의 참조인지 확인

- 성능 최적화용

#### 2. `instanceof` 연산자로 입력이 올바른 타입인지 확인

- 클래스가 구현한 특정 인터페이스일 수 있다.

#### 3. 입력을 올바른 타입으로 형변환

- 앞서 타입 확인을 했기 때문에 무조건 성공

#### 4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

- `float`과 `double`은 각각 정적 메서드인 `Float.compare(float, float)`, `Double.compare(double, double)`로 비교
  - 부동 소수점
- `null`을 정상값으로 취급하는 경우 `Objects.equals(Object, Object)`를 사용하여 `NullPointerException` 예방

### 고려해야 할 점

- 어떤 필드를 먼저 비교하느냐가 `equals`의 성능을 좌우할 때가 있다.
  - 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하는 것이 좋다.
- 동기화용 락(lock) 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다.
- `equals`를 재정의했다면 단위 테스트를 해보자.
- `hashCode`도 같이 재정의해라.
- `Object` 외의 타입을 매개변수로 받는 `equals`는 선언하지 말자.
  - 재정의가 아니라 다중정의이다.
- 구글의 오픈소스 프레임워크 AutoValue를 사용하면 어노테이션 하나로 메서드들을 작성해준다. 또는 IDE에 맡겨도 좋다.
