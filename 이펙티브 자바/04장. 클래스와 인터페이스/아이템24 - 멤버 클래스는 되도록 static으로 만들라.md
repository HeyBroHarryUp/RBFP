# 멤버 클래스는 되도록 static으로 만들라

## 중첩 클래스란?
- `중첩 클래스(nested class)`란 다른 클래스 안에 정의된 클래스를 말한다.
- 중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야한다.
	- 그외 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

## 중첩 클래스 종류

1. 정적 멤버 클래스
2. (비정적) 멤버 크래스
3. 익명 클래스
4. 지역 클래스


## 1. 정적 멤버 클래스

- 다른 클래스 안에서 선언된다.
- 바깥 클래스의 private 멤버에도 접근할 수 있다. 

> 이를 제외하고는 일반 클래스와 똑같다.

### 1.1. 정적 멤버 클래스 예제 코드
```java
public  class Calculator {
	public  enum Operator{ // 열거타입도 정적 멤버 클래스이다
		PLUS(),MINUS()
	}
	public  static  class Operator {
		// 대부분 이처럼 선언 
	}
}
```

## 2. 비정적 멤버 클래스

- 정적 멤버 클래스와 비정적 멤버 클래스의 구문상 차이는 단지 `static`이 붙어 있고 없고의 뿐이다.
	- 하지만 의미상의 차이는 꽤 크다.
- 의미상의 차이는 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.
	- 따라서 비정적 멤버 클래스의 인스턴스는 메서드에서 `정규화된 this`를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.
	- `정규화된 this`란 *클래스명*.this 형태로 바깥 클래스의 이름을 명시하는 용법을 말한다.
- 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없다.
- 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다며 정적 멤버 클래스로 만들어야 한다.

### 2.1. 비정적 멤버 클래스 예제 코드

```java
public  class NestedNonStaticExample {

	private  final String name;
	
	public NestedNonStaticExample(String name) {
		this.name = name;
	} 

	public String getName() {
		// 비정적 멤버 클래스와 바깥 클래스의 관계가 확립되는 부분
		NonStaticClass nonStaticClass = new NonStaticClass("nonStatic : ");
		return nonStaticClass.getNameWithOuter();
	}

	private  class NonStaticClass {
		private  final String nonStaticName;
		public NonStaticClass(String nonStaticName) {
			this.nonStaticName = nonStaticName;
		}

		public String getNameWithOuter() {
			// 정규화된 this 를 이용해서 바깥 클래스의 인스턴스 메서드를 사용할 수 있다. 
			return nonStaticName + NestedNonStaticExample.this.getName();
		}
	}
}
```

### 2.2. 비정적 멤버 클래스의 쓰임
- 어댑터를 정의할 때 자주 쓰인다.

### 2.3. 정리

- **멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조껀 static을 붙여서 정적 멤버클래스로 만들자**

-   숨은 외부 참조를 갖게되며, 이 참조를 저장하기 위한 시간과 공간이 소비된다.

-   GC가 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다.

## 3. 익명 클래스

-   이름이 없다(익명이니까..!)
-   바깥 클래스의 멤버도 아니다.
-   쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
-   비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스 참조가 가능하다
-   상수 정적변수 (`static final`) 외에는 정적 변수를 가질 수 없다.


### 3.1 익명 클래스 예제 코드

```java
public clas AnonymousExample {

	private double x;
	private double y;
	
	public double operate() {
		Oerator operator = new Operator() {
			@Override
			public double plus() {
				return x + y;
			}
		};

		return operator.plus();
	}
	
}

interface Oeperator {
	double plus();
	double minus();
}
```

### 3.2 익명 클래스 제약사항

-   선언한 지점에서만 인스턴스를 만들수 있다.
-   Instanced 검사, 클래스 이름이 필요한 검사를 할 수 없다.
-   여러 인터페이스를 구현할 수 없다. 구현과 동시에 다른클래스 상속도 불가능하다.
-   익명 클래스 사용 클라이언트는 사용하는 익명 클래스가 상위타입에서 상속한 멤버외에는 호출이 불가능하다
-   표현식 중간에 등장해 10줄이 넘어가면 가독성이 좋지 않다.

## 4. 지역 클래스

-   가장 드물게 사용된다.
-   지역변수를 선언할 수 있는 곳이면 어디서든 선언할 수 있다
-   지역변수와 유효범위가 같다
-   **멤버 클래스 유사도**  : 이름이 있고 반복해서 사용이 가능하다
-   **익명 클래스 유사도**  : 비정적 문맥에서 사용할 때만 바깥 인스턴스 참조가 가능하다
-   **익명 클래스 유사도**  : 정적 멤버는 가질 수 없고, 가독성을 위해 짧게 작성한다.

### 4.1. 지역 클래스 예제 코드

```java
public class LocalExample {
	private int number;
	public LOcalExample(int number) {
		this.number = nuber;
	}
	public void foo() {
		class LocalClass {
			private String name;
			public LocalClass(String name) {
				this.name = name;
			}
			public void print() {
				System.out.println(number + name);
			}
		}
		LocalClass localClass = new LocalClass("local");
		localClass.print();
	}
}
```
  
    
  
#### 참고 자료
[쟈미의 devlog](https://jyami.tistory.com/86)