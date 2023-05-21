# 6. 객체와 자료 구조

## 자료 추상화

아래 코드에서 Point 클래스는 내부를 그대로 노출한다.
해당 클래스 변수를 private으로 수정하고 내부에 getX(), getY() 같은 함수를 추가한다고 해도 구현을 숨길 수 없다.

```swift
public class Point {
    public var x: Double
    public var y: Double
}
```

아래와 같이 추상 인터페이스를 사용하도록해서 구현을 감출 수 있다.

```swift
public protocol Point {
    public func getX() -> Double
    public func getY() -> Double
}
```

추상 인터페이스를 통해서 사용자가 구현을 모른채 자료의 핵심을 조작할 수 있어야 한다.

다만, 위 인터페이스는 값 자체를 그대로 노출시키므로 좋은 코드는 아니다.
자료를 세세하게 공개하기보다는 추상적인 개념으로 표시하는 편이 좋다.

```swift
public protocol Point {
    /// 원점으로부터 거리를 반환하는 함수
    public func distanceFromOrigin() -> Double
    /// 원점으로부터 기울기를 반환하는 함수
    public func slopeFromOrigin() -> Double
}
```

## 자료/객체 비대칭

- 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.
- 자료구조는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.

위 두가지 정의는 서로 상반된다.

**절차적인 도형**

각 도형 클래스는 간단한 자료구조이며 아무런 메소드도 제공하지 않는다.
도형이 동작하는 방식은 `Geometry` 클래스에서 구현한다.

```swift
public class Square {
    public var center: Point
    public var side: Double
}

public class Rectangle {
    public var topLeft: Point
    public var height: Double
    public var width: Double
}

public class Circle {
    public var center: Point
    public var radius: Double
}

public class Geometry {
    public var pi: Double = 3.141592653589793
    
    public func area(of shape: AnyObject) -> Double? {
        if let square = shape as? Square {
            return square.side * square.side
        } else if let rect = shape as? Rectangle {
            return rect.height * rect.width
        } else if let circle = shape as? Circle  {
            return circle.radius * circle.radius * pi
        }
    }
}
```

객체지향의 관점에서 위와 같은 절차적 코드는 좋지않다고 비판할 수 있다.

- 다만, 도형의 둘레를 구하는 함수를 추가하고 싶다면?
    - 각 도형 클래스는 아무런 영향도 받지 않고 `Geometry`클래스에 둘레를 구하는 함수를 구현하면 된다.
- 반대로, 새로운 도형을 추가하고 싶다면?
    - `Geometry`내에 구현된 모든 함수를 수정해야 한다.

**다형적인 도형**

객체 지향적인 도형 클래스를 구현한다.
다형성의 특성을 사용하므로 Geometry와 같은 별도의 클래스는 필요하지 않다.

```swift
public class Square: Shape {
    private var center: Point
    private var side: Double
    
    public func area() -> Double {
        return side * side
    }
}

public class Rectangle: Shape {
    private var topLeft: Point
    private var height: Double
    private var width: Double
    
    public func area() -> Double {
        return height * width
    }
}

public class Circle: Shape {
    private var center: Point
    private var radius: Double
    private var pi: Double = 3.141592653589793
    
    public func area() -> Double {
        return radius * radius * pi
    }
}
```

위와 같이 새로운 타입이 필요한 경우에는 객체지향 기법이 적절하고,
새로운 함수가 필요한 경우에는 절차지향 기법이 적절하다.

때로는 단순한 자료구조와 절차지향적인 방법이 최선인 경우도 있다.

## 디미터 법칙

모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다.
객체는 자기 자신의 자료를 숨기고 함수를 공개한다. 즉, 조회 함수로 내부 구조를 드러내서는 안된다.

**클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다**

- 클래스 C(자기 자신)
- f가 생성한 객체
- f의 parameter로 넘어온 객체
- C 인스턴스 변수에 저장된 객체

### 기차 충돌(Train wreck)

```swift
var outputDir: String = ctxt.getOptions().getScratchDir().getAbsolutePath()
```

위 코드는 getOptions()가 반환하는 객체의 getScratchDir()를 호출하고, 다시 그 함수가 반환하는 객체의 getAbsolutePath()를 호출하므로 디미터의 법칙에 위배된다.

위와 같은 코드를 기차 충돌(Train wreck)이라고 부른다.

일반적으로 조잡하다고 여겨지므로 아래와 같이 나누는 것이 좋다.

```swift
var options = ctxt.getOption()
var dir = options.getScratchDir()
var path = dir.getAbsolutePath()
```

다만, 위와 같이 나누어진 코드도 ctxt 가 options를 포함하고 options가 다시 ScratchDir을 포함하며, ScratchDir은 AbsolutePath를 포함한다.
위와 같은 코드가 있는 함수는 수많은 객체를 탐색하게 되는것이다.

위 예제가 디미터 법칙을 위반하는지 여부는 ctxt, Options, ScratchDir이 객 체인지 아니면 자료 구조인지에 달렸다.
객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다.
반면, 자료 구조라면 당연히 내부 구조를 노출 하므로 디미터 법칙이 적용되지 않는다.

조회 함수를 사용하지 말고 아래와 같이 변수로 구성하면 디미터의 법칙을 거론할 이유가 없어진다.

```swift
var outputDir: String = ctxt.options.scratchDir.absolutePath
```

### 잡종 구조

이런 혼란들로 인해서 절반은 객체, 절반은 자료구조 형태의 잡종구조가 나온다.

이런 잡종 구조는 객체와 자료구조 양쪽의 단점만 모아놓은 구조이므로 피하는 것이 좋다.

### 구조체 감추기

ctxt, options, scratchDir이 실제 객체라면 위와 같이 줄줄이 엮어서는 안된다.

ctxt가 객체라면 뭔가를 하라고 말해야지 속을 드러내라고 말하면 안 된다.
절대 경로를 얻는 이유가 임시 파일을 저장하기 위해서라면 ctxt에게 임시 파일을 저장하도록 명령하면 된다.

```swift
ctxt.saveTempFile()
```

ctxt는 내부 구조를 드러내지 않으며, 해당 함수를 콜하는 함수는 불필요하게 여러 객체를 탐색할 필요가 없다.

## 자료 전달 객체

자료 전달 객체의 전형적인 예시는 공개 변수만 있고 함수가 없는 클래스이다.

## 결론

- 객체는 동작을 공개하고, 자료를 숨긴다.
→ 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다.
- 자료 구조는 별다른 동작 없이 자료를 노출한다.
→ 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 주가하기는 어렵다.

훌륭한 개발자는 편견에 사로잡히지 않고, 문제 해결을 위해 최적의 방법을 선택한다.