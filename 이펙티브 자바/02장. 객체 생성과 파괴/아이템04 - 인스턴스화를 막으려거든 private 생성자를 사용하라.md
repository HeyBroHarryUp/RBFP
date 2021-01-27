# 인스턴스화를 막으려거든 private 생성자를 사용하라

## 사용 목적

- 인스턴스를 생성하지 않고 `정적 멤버`만을 제공하여 사용하도록 하기 위함이다.
  > `정적 멤버`란 static이 붙은 필드나 메서드를 말한다.

## 예시

```java
public class DateUtility {
    private static String FULL_DATE_FORMAT = "yyyy-MM-dd HH:mm:ss.SSS";
	/*
	    !!생성자 없음!!
	*/
    public static String convertDateToString(Date date) {
        return new SimpleDateFormat(FULL_DATE_FORMAT).format(date);
    }
}
```

- 생성자를 명시하지 않으면 **컴파일러가 자동으로 기본 생성자를 만들어 준다.**
- 개발자는 사용자가 convertDateToString 메서드를 통하여 사용하기를 바란다. 하지만 사용자는 아래처럼 인스턴스를 생성한 후 사용할 수 있다.

```java
public void someMethod() {
	DateUtility dateUtility = new DateUtility();
    String formattedToday = dateUtility.convertDateToString(new Date());
    ...
}
```

- 우리가 기대한 사용 방법은 아래와 같다.

```java
public void someMethod() {
	String formattedToday = DateUtility.convertDateToString(new Date);
	...
}
```

## 인스턴스화를 막는 방법

- 인스턴스화를 막는 방법은 간단하다. 단지 **private 생성자**를 추가하면 된다.
- 이 방식은 **상속을 불가능**하게 하는 효과도 있다. (모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데, 이를 private으로 선언 했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막혀버린다.)

```java
public class DateUtility {
    ...
	// 생성자 생성을 막기 위함!
	private DateUtility() {
		throw new AssertionError();
	}

	...
}
```

- 하지만 생성자가 존재하는데 호출할 수 없다니 직관적이지 않다. 따라서 위의 코드처럼 **적절한 주석을 달아주는 것이 좋다.**
- 꼭 AssertionError를 던질 필요는 없지만, 클래스 안에서 실수라도 생성자를 호출하지 않도록 해준다.
