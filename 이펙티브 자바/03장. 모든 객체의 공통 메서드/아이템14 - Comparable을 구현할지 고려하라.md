## [사전 지식] Comparable과 Comparator의 차이점

Comparable, Comparator 모두 자바에서 정렬을 할 때 사용한다. 하지만 이 두개는 명확한 차이를 가진다. 

- Comparable - 기준을 설정하고 정렬. 즉, 특정한 기준 1가지를 가지고 정렬 Ex) 학점을 기준으로 오름차순 or 내림차순

-  Comparator - Comparable과 다르게 요구사항에서 주어진 특정 기준을 가지고 정렬 Ex) 학점이 같다면 이름순으로 정렬

### Comparable

Comparable은 compareTo 메서드를 오버라이드한다.
| 메서드 | 설명 |
|--|--|
| int compareTo(T o) | 해당 객체와 전달된 객체의 순서를 비교함. |

Comparable을 구현한 클래스는 기본적으로 오름차순을 기준으로 정렬을 수행한다. 또한 **1가지** 기준점을 가지고 그에 대해 정렬을 하고 싶을 때 사용한다.

### Comparator

Comparator는 compare 메서드를 오버라이드 한다.
| 메서드 | 설명 |
|--|--|
| int compare(T o1, T o2) | 두 개의 인자를 받아 순서를 정한다. |

Comparator를 구현한 클래스는 일반적인 정렬기준을 넘어 사용자 정의 조건을 규칙으로 정렬을 하고 싶을 때 사용한다.

[Docs: Java8 Comparator](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html)



# Comparable을 구현할지 고려하라

- `Comparable` 인터페이스는 `compareTo` 메서드를 구현하라고 한다.
- compareTo는 `Object`의 메서드가 아니다.
- 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입(아이템 34)이 Comparable을 구현했다.
- 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

## Object equals와 차이점

- compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있다.
- 제네릭하다.
- Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서(natural order)가 있음을 뜻한다.
- Comparable을 구현한 객체들의 배열은 다음처럼 쉽게 정렬할 수 있다.

	```java
	Arrays.sort(a);
	```
	
## compareTo 메서드의 일반 규약

- 이 객체와 주어진 객체의 순서를 비교한다.
- 이 객체가 주어진 객체보다 작으면 음수를 반환한다.
- 같다면 0을 반환한다.
- 크다면 양의 정수를 반환한다.
- 이 객체와 비교할 수 없는 타입의 객체가 주어지면 `ClassCastException`을 던진다.

### 첫 번째 규약
`(x.compareTo(y) < 0)`  이라면  `(y.compareTo(x) > 0)`  이다.  따라서  `x.compareTo(y)`가 Exception을 발생시킨다면 `y.compareTo(x)`  또한 Exception을 발생시켜야 한다.

> 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다는 얘기다.
    
### 두 번째 규약
`x.compareTo(y) < 0`  이고  `y.compareTo(z) < 0`  이라면  `x.compareTo(z) < 0`  이다. (삼단 논법과 같다.)

> 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야한다.
    
### 세 번째 규약
크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야한다. `x.compareTo(y) == 0`  으로 인해 동치가 두 객체의 크기가 같다면, `x.compareTo(z)`  와  `y.compareTo(z)`  는 같아야한다.

> 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다는 뜻이다.

### 추가적 규약
-   별도로 equals 메서도와 일관되게 작성하는 것이 좋다.
- 만약 일관되지 않게 구현을 했다면 그렇다는 것을 명시해 주도록 하자.
- 정렬된 컬렉션의 경우 equals가 아닌 compareTo를 이용하여 정렬을 시도한다.

## 구현

- compareTo는 타입이 다른 객체를 신경쓰지 않아도 된다.
	- 타입이 다른 객체가 주어지면 간단히 `ClassCastException`을 던져도 된다.(대부분 그렇게 한다.)
- 물론 다른 타입 사이의 비교도 허용하는데, 보통은 배교할 객체들이 구현한 공통 인터페이스를 매개로 이루어진다.

### 기본 타입 필드가 여러 개 일때 비교
```java
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(this.areaCode,pn.areaCode);
	if(result == 0) {
		result = Short.compare(this.prefix, pn.prefix);
		if(result == 0) {
			result = Short.compare(this.lineNum, pn.lineNum);  
		}  
	}  
}
```

- 필드의 정렬 우선순위를 정해 같은 값이 있을 때 마다 조건을 추가한다.

### 위 코드를 Comparator로 구현하는 방법
```java
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode)
	.thenComparingInt(pn -> pn.prefix)
	.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPARTOR.compare(this, pn);
}
```



## compareTo 안티 패턴

- compareTo 메서드에서 관계연산자 (`<` 와 `>`)를 사용하지 말아야 한다.
	- 대신 Type.compare(T t1, T t2)를 사용하여 비교하는 것이 좋다.
- 해쉬코드의 차를 이용해 비교하지 말자.

### 안티 패턴 코드

#### 1

```java
public int compareTo (Object obj) {  
 return this.x < obj.x ? 1 : (this.x == obj.x) ? 0 : -1;  
}
```

개선 방법

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	(Object o1, Object o2) -> Integer.compare(o1.hashCode(), o2.hashCode())  
}
```

#### 2

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {  
 (Object o1, Object o2) -> o1.hashCode() - o2.hashCode();  
}
```

- hashCode의 차로 구현한 위의 코드 처럼 한다면, 정수 overflow를 일으키거나 IEEE754 부동소수점 계산방식에 따른 오류를 발생 시킬 수 있다.

개선 방법

```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```