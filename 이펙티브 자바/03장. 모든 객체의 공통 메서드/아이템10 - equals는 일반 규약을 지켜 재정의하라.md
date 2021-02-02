# equals는 일반 규약을 지켜 재정의하라

- equals 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 있어 자칫하면 끔찍한 결과를 초래한다.
- 이 문제를 해결하려면 가장 쉬운 방법은 아예 재정의하지 않는 것이다.
	- 그냥 두면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.

## 재정의할 필요 없는 경우

1. 각 인스턴스가 본질적으로 고유하다
	- 값을 표현하는 것이 아닌 동작하는 개체를 표현하는 클래스가 해당된다.
	- Thread가 좋은 예로, Object의 equals 메서드는 이러한 클래스에 딱 맞게 구현되었다.

2. 인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없다
	- 객체 본질적으로 같은지를 판단하는 것이 아니라 논리적으로 같은지를 판단하는 경우는 equals를 재정의 해야 하지만 그렇지 않다면 재정의할 필요가 없다.

3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 경우
	- 예컨대 대부분의 `Set` 구현체는 `AbstractSet`이 구현한 equals를 상속받아 쓰고, `List` 구현체들은 `AbstractList`로부터, `Map` 구현체들은 `AbstractMap`으로부터 상속받아 그대로 쓴다.

4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없을 경우.

## equals를 재정의해야 하는 경우
- `객체 식별성(object identity)`이 아니라 `논리적 동치성`을 확인해야할 경우 + 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되어 있지 않을 경우이다.

	> 객체 식별성: 두 객체가 물리적으로 같은가

- 주로 `값 클래스`들이 여기에 해당한다.

	> 값 클래스: Integer와 String처럼 값을 표현하는 클래스를 말한다. 두 값 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아니라, "값"이 같은지를 알고 싶어 할 것이다.

- 값 클래스라고 하더라도 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 경우는 equals를 재정의하지 않아도 된다. 아래가 그에 해당한다.
	1. 인스턴스 통제 클래스(아이템 1)
	2. Enum(아이템 34)

## equals 메서드를 재정의할 때 반드시 지켜야할 일반 규약

다음은 Object 명세에 적힌 규약이다.

equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.
- 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
 - 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals.(x)도 true다.
- 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)가 true이면 x.equals(z)도 true이다.
- 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

> 동치관계: 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산이다. 이 부분집합을 동치류(equivalence class; 동치 클래스)라 한다. equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.