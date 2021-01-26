# private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 싱글턴(singleton)이란?

### 정의

- 인스턴스를 오직 하나만 생성할 수 있는 클래스

### 장점

- 한번의 객체 생성으로 재사용이 가능하여 메모리 낭비를 방지할 수 있다.
- 전역성을 갖기 때문에 다른 객체와 공유가 용이히다.

### 단점

- 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려워질 수 있다.

## 싱글턴을 만드는 방법

1. public static final 필드 방식
2. 정적 팩터리 방식
3. 열거 타입 방식

## 1. public static final 방식

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {...}

}
```

- private 생성자는 Elvis.INSTANCE를 초기화할 때 딱 한번만 호출된다.
- public이나 protected 생성자가 없으므로 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임을 보장된다.
- 다만 리플렉션 API인 `AccessibleObject.setAccessible`을 사용해 private 생성자를 호출할 수 있다. 이러한 공격을 방지하고자 한다면, 생성자에서 두 번 객체를 생성하려고 할 때 예외를 던지게 하면 된다.

### 장점

1. 해당 클래스가 싱글턴임이 API에 명백히 들어난다. (public static 필드가 final임으로 절대로 다른 객체를 참조할 수 없다.)
2. 간결하다.

## 2. 정적 팩터리 방식

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elivs();
	private Elvis() {...}

	public static Elivs getInstance() {
		return this.INSTANCE;
	}

}
```

- Elvis.getInstance는 항상 같은 객체의 참조를 반환함으로 제 2의 Elvis 인스턴스란 결코 만들어지지 않는다.
- 단 리플렉션을 통한 예외는 똑같이 적용된다.

### 장점

- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
  - 호출하는 스레드별로 다른 인스턴스를 넘기게 하는 등 ...
- 정적 패터리를 제네릭 싱글턴 팩터리로 만들 수 있다. (아이템 30)
- 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다는 점 (아이템 43, 44)
  - `Printer::getInstance` -> `Supplier<Printer>`

> [Supplier Interface](https://m.blog.naver.com/zzang9ha/222087025042): 매개변수를 받지 않고 단순히 무엇인가를 반환하는 함수형 인터페이스 -> Lazy Evaluation

#### 장점

- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
  - 호출하는 스레드별로 다른 인스턴스를 넘기게 하는 등 ...
- 정적 패터리를 제네릭 싱글턴 팩터리로 만들 수 있다. (아이템 30)
- 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다는 점 (아이템 43, 44)
  - `Printer::getInstance` -> `Supplier<Printer>`

> [Supplier Interface](https://m.blog.naver.com/zzang9ha/222087025042): 매개변수를 받지 않고 단순히 무엇인가를 반환하는 함수형 인터페이스 -> Lazy Evaluation

## 유의점

- 1번과 2번 방식으로 만들어진 싱글턴 클래스를 `직렬화`하려면 단순히 `Serializable`을 구현한다고 선언하는 것만으로는 부족하다.

- 모든 인스턴스 필드에 `transient`를 선언하고, `readResolve` 메서드를 제공해야 한다.(아이템 89)
- `역직렬화`시에 새로운 인스턴스가 만들어지는 것을 방지할 수 있다.
- 만약 이렇게 하지 않으면 초기화해둔 인스턴스가 아닌 다른 인스턴스가 반환된다.

```java
private Object readResolve() {
	// 진짜 Elvis를 반환하고,가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```

## 3. 열거 타입 방식

```java
public enum Elvis {
	INSTANCE;

	public void leaveTheBuilding() {...}
}
```

### 장점

1. public 필드 방식과 비슷하지만, 더 간결하다.
2. 추가 노력 없이 직렬화할 수 있다.
3. 아주 복잡한 직렬화 상황이나 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.
4. **대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.**

### 주의점

- 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
