
# equals를 재정의하려거든 hashCode도 재정의하라

- **equals를 재정의한 클래스 모두에서 `hashCode`도 재정의해야 한다.**
- 그렇지 않으면 hashCode 일반 규약을 어기게되어 해당 클래스의 인스턴스를 `HashMap`이나 `HashSet`같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

### Object 명세에서 발췌한 규약
- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야한다.
- equals(Object)가 두 객체를 다르게 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성는이 좋아진다.

- 재정의를 잘못했을 때 크게 문제가 되는 조항은 두번 째로, 논리적으로 같은 객체는 같은 해시코드를 반환해야한다.

### hashCode를 재정의 하지 않을 경우 발생하는 문제

```java
Map<PhoneNumber, Stirng> m = new HashMap<>();
m.put(new PhoneNumber(010, 1234, 1234), "해리");
m.get(new PhoneNumber(010, 1234, 1234)) // null
```

- `get`메서드를 실행하면 해리가 나와야 하지만 null이 반환된다.
- 그 이유는 PhoneNumber의 hashCode를 재정의 하지 않았기 때문이다.

## 최악의 해시코드 구현

```java
@Override
public int hashCode() { return 42; }
```

# 해시코드 재정의 방법

- equals 비교에 사용되지 않은 필드는 반드시 제외해야한다.
- 성능을 높이기위해 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
	- 속도야 조금 빨라지겠지만, 해시 품질이 나빠져 해시 테이블의 성능을 심각하게 떨어트린다.
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.
	- 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

## 1. Objects 클래스가 재공하는 hash 메서드

	```java
	@Override
	public int hashCode() {
		Objects.hash(first, middle, last);
	}
	```

	- 구현하기 가장 쉬운 방법이지만 속도가 살짝 아쉽다.

## 2. 좋은 hashCode를 작성하는 간단한 요령
	
### 1. int 변수 result를 선언한 후 값을 c로 초기화한다. 이때 c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드다.

### 2. 해당 객체의 나머지 핵심 필드 f 각각에 다음 작업을 수행한다.

#### 2.1. 해당 필드의 해시코드 c를 계산한다.

##### 2.1.1. 기본 타입 필드라면, *tpye*.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.
			
##### 2.1.2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형(canonical representation)을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다(다른 상수도 괜찮지만 전통적으로 0을 사용한다).

##### 2.1.3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.2 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.

##### 2.2. 단계 2.1에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다. 
```java
result = 31 * result + c;
```
		
### 3. result를 반환한다.

```java
@Override
public int hashCode() {
	int result = Short.hashCode(first);
	result = 31 * result + Short.hashCode(middle);
	result = 31 * result + Short.hashCode(last);
	return result;
}
```

## 클래스가 불변이고 해시코드를 계산하는 비용이 클 때

- 캐싱하는 방법을 고려한다.

```java
private int hashCode;

@Override
public int hashCode() {
	int result = hashCode;
	if (hashCode == 0) {
		int result = Short.hashCode(first);
		result = 31 * result + Short.hashCode(middle);
		result = 31 * result + Short.hashCode(last);
		hashCode = result;
	}
	return result;
}
```