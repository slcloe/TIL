# Item 23 - 태그 달린 클래스보다는 클래스 계층구조를 활용하라

#### 태그 클래스의 단점
* 태그 클래스란 final shape처럼 필드를 사용하여 타입을 표현하는 클래스이다.
```java
// 태그 클래스
static class Figure {
	enum Shape { RECTANGLE, CIRCLE };

	// 태그 필드 - 현재 모양을 나타낸다.
	final Shape shape;

	// 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
	double length;
	double width;

	// 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
	double radius;

	// 원용 생성자
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

	// 사각형용 생성자
	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}

	double area() {
		switch(shape) {
			case RECTANGLE -> {
				return this.length * this.width;
			}
			case CIRCLE ->  {
				return Math.PI * (radius * radius);
			}
			default -> {
				throw new AssertionError(shape);
			}
		}
	}
}
1. 열거 타입, 태그 필드 등 쓸데 없는 코드가 많다.
2. 다른 의미를 위한 코드가 클래스 내부에 저장되어 메모리가 많이 사용된다.
3. 필드를 final로 선언하려면 해당 의미에 쓰이지 않는 필드까지 생성자에서 초기화 해야한다.
4. 엉뚱한 필드를 초기화해도 런타임시에 문제를 알 수 있다.
5. 새로운 태그를 추가하기 위해 코드를 수정하면 런타임시에 문제를 알 수 있다.
6. 인스턴스 타입만으로는 현재 나타내는 의미를 알 수 없다.

```
#### 계층구조로 변경하기

```java
abstract static class Figure {
    abstract double area();
}

static class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

static class Rectangle extends Figure {
    final double width;
    final double length;

    public Rectangle(double width, double length) {
        this.width = width;
        this.length = length;
    }

    @Override
    double area() {
        return width * length;
    }
}

static class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```
* 각 클래스에서 필요요소, 공통요소를 분리하여 추상클래스, 구현클래스로 나눌 수 있다.
* 만약 태그클래스를 사용하고 있다면, 리팩토링을 통해 계층 구조로 만들어보자