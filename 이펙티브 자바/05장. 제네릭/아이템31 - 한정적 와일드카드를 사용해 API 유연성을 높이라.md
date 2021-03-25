# 한정적 와일드카드를 사용해 API 유연성을 높이라
- 매개변수화 타입은 불공변(invariant)이다.
- `List<Stinrg>`은 `List<Object>`의 하위타입이 아니라는 뜻이다.
- `List<Object>`에는 어떤 객체든 넣을 수 있지만 `List<String>`에는 문자열만 넣을 수 있다.
- 즉 `List<String>`은 `List<Object>`이 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다.
  - 리스코프 치환 원칙에 어긋난다. (아이템 10)

## 때론 불공변 방식보다 유연한 무언가가 필요하다
- 아이템 29의 Stack 클래스
  
```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```
  
- 여기에 일련의 원소를 스택에 넣는 메서드를 추가한다고 보자.
```java
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```
  
- 이 메서드는 컴파일되지만 완벽하지 않다.
- `Iterable src`의 원소타입이 `Stack`의 원소 타입과 일치한다면 잘 동작한다.
- 하지만 `Stack<Number>`로 선언한 후 `pushAll(intVal)`을 호출하면 Integer는 Number의 하위타입이 잘 동작할 것 같지만
- 실제로는 에러가 발생한다.
- 매개변수화 타입이 불공변이기 때문이다.

### pushAll 해결 방법
- 자바에서는 이런 상황에 대처할 수 있도록 `한정적 와일드 카드 타입`을 지원한다.
- pushAll의 매개변수 타입은 E의 Iterable이 아니라 **E의 하위 타입 Iterable**이어야 한다.
  - 사실 extends라는 키워드는 이 상황에서 딱 어울리지는 않는다.
  - 하위 타입이란 자기 자신도 포함하지만, 그렇다고 자신을 확장(extends)한 것은 아니기 때문이다.
  
```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```
  
- 위의 코드로 변경하면 잘 동작한다.

### 이번에는 popAll을 해결해보자
- 와일드 카드 타입을 사용하지 않은 popAll 메서드 -> 결함 존재!
  
```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```
  
- 이번에도 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 일치한다면 전혀 문제 없다.
- 하지만 다르다면 에러가 발생한다.
- `Stack<Number>`의 원소를 `Object`용 컬렉션에 옮긴다고 해보자.
  
```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> Objects = ...;
numberStack.popAll(objects);
```
  
- 위 코드를 컴파일하게되면 에러가 발생한다.
- 에러 내용은 `Collection<Object>`는 `Collection<Number>`의 하위 타입이 아니다라는 내용이다.
- 이번에도 와일드카드 타입으로 해결할 수 있다.
- 이번에는 E의 Collection이 아니라 E의 상위 타입의 Collection이어야 한다.
  
```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

## PECS 공식
- **Producer(생산자)의 경우에는 extends**
- **Consumer(소비자)의 경우에는 super**
- Stack 예에서 pushAll의 src 매개변수는 Stack이 사용할 E 인스턴스를 생산하므로 src의 적절한 타입은 `extends`이다.
- popAll의 dst 배개변수는 Stack으로부터 E 인스턴스를 소비하므로 dst의 적절한 타입은 `super`이다