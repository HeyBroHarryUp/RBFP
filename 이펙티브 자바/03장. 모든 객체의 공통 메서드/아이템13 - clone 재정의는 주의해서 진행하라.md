## [사전 지식] 얕은 복사 vs 깊은 복사

### 얕은 복사(shallow copy)

- 객체의 참조값(주소값)을 복사한다.
- 객체를 복사할 때 , 객체 자체만 복사하여 새 객체를 생성한다.
- 복사된 객체의 인스턴스 변수는 원본 객체의 인스턴스 변수와 같은 메모리 주소를 참조한다.
- 따라서 해당 메모리 주소의 값이 변경되면 원본 객체와 인스턴스 변수 값 모두 변경된다.

### 깊은 복사(deep copy)

- 객체를 복사할 때, 해당 객체와 인스턴스 변수까지 복사한다.
- 객체 자체 뿐만 아니라 객체 내부에있는 필드도 전부 복사해 새 주소에 담기 때문에 참조를 공유하지 않는다.
- 원본 객체 자체만 `Clonable` 인터페이스만 구현하면 안되고 복사할 원본 객체가 가지고 있는 필드가 기본타입이 아니고 **객체일 경우 그 필드의 객체 또한 `clonable`을 구현해야한다.**

## 복사할 원본 객체가 기본타입 필드만 가지고 있을 경우 깊은 복사 구현 방법

```java
public  class  PhysicalInformation  implements  Cloneable{

	int height;
	int weight;
	
	@Override
	public  Object  clone()  throws CloneNotSupportedException {
		return  super.clone();
	} 
}  
  
```

## 복사할 원본 객체가 객체 필드를 가지고 있을 경우 깊은 복사 구현 방법

```java
public  class  PhysicalInformation  implements  Cloneable {

	int height;
	int weight;
	
	@Override
	public  Object  clone()  throws CloneNotSupportedException {
		return  super.clone();
	} 
} 

public class Person implements Cloneable {
	
	String name;
	int age;
	PhysicalInformation physicalInformation;
	
	@Override  
	public Person clone() throws CloneNotSupportedException {  
		Person person = (Person) super.clone();
		person.physicalInformation = (PhysicalInformation) physicalInformation.clone();
		return person;
	}

}
```

- Person의 PhysicalInformation 객체 필드 또한 clone이 구현되어 있어야한다.
- clone 메서드는 기본적으로 `protected`로 되어있다 이것을 `public`으로 고쳐줘야 다른 패키지에서도 clone 메서드 사용이 가능하다.
- 반환타입도 기본적으로 `Object`로 되어있는데 이를 복사할 원본 객체 타입으로 반환해주어 반환받는 곳에서 형변환하지 않아도 되게끔 해주자.

[참고자료 - 얕은복사 vs 깊은복사](https://rok93.tistory.com/entry/%EC%96%95%EC%9D%80%EB%B3%B5%EC%82%AC-VS-%EA%B9%8A%EC%9D%80%EB%B3%B5%EC%82%AC)

# clone 재정의는 주의해서 진행하라

`Cloneable`은 복제해도 되는 클래스임을 병시하는 용도의 믹스인 인터페이스다. 하지만 아쉽게도 의도한 목적을 제대로 이루지 못했다.
가장 큰 문제는 `clone` 메서드가 선언된 곳이 `Cloneable`이 아닌 `Object`이고, 그마저도 `protected`라는 데 있다.
따라서 `Cloneable`을 구현하는 것 만으로는 외부 객체에서 `clone` 메서드를 호출할 수 없다.

> 리플렉션(아이템 65)를 사용한다면 가능하지만 100% 성공하는 것도 아니다. 해당 객체가 접근이 허용된 clone 메서드를 제공한다는 보장이 없기 때문이다.

## Cloneable은 무슨 일을 할까

- Object의 protected 메서드인 clone의 동작 방식을 결정한다.
- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사해서 객체를 반환한다.
- 그렇지 않은 클래스의 인스턴스에서 clone을 호출하면 `CloneNotSupportedException`을 던진다.
- 이 경우는 상당히 이례적으로 사용한 예이니 따라 사용하지 말자.

> 일반적으로 인터페이스를 구현한다는 것은 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위다. 그런데 Cloneable의 경우에는 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이다.

- 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.