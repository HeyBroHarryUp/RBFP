# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 캡슐화의 이점을 갖지 못하는 클래스
```java
class Point {
	public double x;
	public double y;
}
```

- x, y가 `public`으로 노출되어 외부에서 접근이 가능하다.

## 개선
```java
class Point {  
  private double x;
  private double y;
  public Point(double x, double y) {  
    this.x = x;  
    this.y = y;  
  }
  public double getX() { return x; }  
  public double getY() { return y; }
  public void setX(double x) { this.x = x; }  
  public void setY(double y) { this.y = y; }  
}
```

## 불변 필드라 해도 노출하지 말자

- API를 변경하지 않고는 표현 방식을 바꿀 수 없다.
- 필드를 읽을 때 부수 작업을 수행할 수 없다.
	- 부수작업: 단순 값만 가져오는 것이 아니라 메서드 내에서 추가 작업을 한 후 return