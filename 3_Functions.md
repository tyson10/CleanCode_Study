# 3. 함수

## 작게 만들어라!

> 함수를 만드는 첫째 규칙은 ‘작게!’다. 함수를 만드는 두번째 규칙은 ‘더 작게’다.
> 

### 블록과 들여쓰기

if/else, while 문 등에 들어가는 블록은 한 줄 이어야 한다.(함수를 호출)

그렇게 하면 바깥을 감싸는 함수가 작아지고, 코드를 이해하기에도 쉬워진다.

함수에서 들여쓰기 수준은 2단을 넘으면 안된다.

## 한 가지만 해라!

> 함수는 한 가지를 해야한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.
> 

추상화 수준이 하나인 단계만 수행한다면, 그 함수는 한 가지만 수행하는 함수로 볼 수 있다.

한 가지 더 쉬운 방법은, 단순히 다른 표현이 아니라 의미 있는 다른 이름으로 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 것이다.

### 함수 내 섹션

한 가지 작업만 하는 함수는 자연스럽게 여러개의 섹션으로 나누기가 어렵다.

```swift
// 추상화 수준이 하나의 단계만 수행하는 코드가 아님
func presentViewController() {
    let vc = ViewController()
    vc.makeConstaints()
    vc.addTargets()
    self.present(vc)
}
-----------------------------------------------------------------
// 추상화 수준이 하나의 단계만 수행하는 코드
func presentViewController() {
    let vc = makeViewController()
    self.present(vc)
}

// 추상화 수준이 하나의 단계만 수행하는 코드
func makeViewController() -> UIViewController {
    let vc = ViewController()
    vc.makeConstaints()
    vc.addTargets()
    return vc
}
```

## 함수 당 추상화 수준은 하나로!

> 함수가 확실히 한 가지 작업만 하려면 함수내의 모든 문장의 추상화 수준이 같아야 한다.
> 

그렇지 않으면 어떤 표현이 근본 개념인지, 세부 사항인지 판단하기가 어려워 진다.

그렇게 되면 깨진 유리창 효과로 함수에 세부 사항을 미친듯이 추가하기 시작한다.

### 위에서 아래로 코드 읽기: 내려가기 규칙

코드는 위에서 아래로 이야기처럼 읽혀야 좋다. 아래로 코드를 읽어 내려가다 보면 함수의 추상화 수준이 한 번에 한 단계씩 낮아진다.

이것을 ‘내려가기’ 규칙이라고 부르기로 한다.

이것을 하기 위한 핵심은 짧고, 한 가지만 하는 함수를 만드는 것.

## Switch 문

switch 문은 작게 만들기 어렵다.

```swift
func calculatePay(e: Employee) throws -> Money {
    switch e {
        case .commissioned:
            return calculateCommissionedPay(e)
        case .hourly:
            return calculateHourlyPay(e)
        case .salaried:
            return calculateSalariedPay(e)
        default:
            throw Error.invalidEmployee
    }
}
```

위 함수에는 몇 가지 문제가 있다.

1. 함수가 길다. 새 직원 유형을 추가하면 더 길어진다.
2. ‘한 가지’ 작업만 하지 않는다.
3. SRP(Single Responsibility Principle) 을 위배한다. 코드를 변경할 이유가 여럿이다.
4. OCP(Open Closed Principle) 을 위배한다. 새 직원 유형을 추가할 때마다 코드를 변경한다.
5. 위 함수와 동일한 구조의 함수가 무한정 존재할 수 있다.(switch 문)

이를 해결한 코드는 아래와 같다.(Swift)

```swift
public protocol Employee {
    func isPayday() -> Bool
    func calculatePay() -> Money
    func deliverPay(pay: Money)
}

public enum EmployeeError: Error {
    case invalidEmployee
}

public protocol EmployeeFactory {
    func makeEmployee(r: EmployeeRecord) throws -> Employee
}

public extension EmployeeFactory {
    func makeEmployee(r: EmployeeRecord) throws -> Employee {
        switch r {
        case .commissioned:
            return CommissionedEmployee.init(with: r)
        case .hourly:
            return HourlyEmployee.init(with: r)
        case .salaried:
            return SalariedEmployee.init(with: r)
        default:
            throw EmployeeError.invalidEmployee
        }
    }
}
```

다형성을 이용해서 switch 문을 딱 한 번만 허용한다.

## 서술적인 이름을 사용하라!

> 길고 서술적인 이름이 짧고 어려운 이름보다 좋다. 길고 서술적인 이름이 길고 서술적인 주석보다 좋다.
> 

함수 이름을 정할땐 여러 단어가 쉽게 읽히는 명명법을 사용한다.

이름을 붙일때는 일관성이 있어야 한다. 모듈내 함수에서 동일한 동사, 명사를 사용해서 네이밍한다.

## 함수 인수

> 함수에서 가장 이상적인 인수 개수는 0개(무항)이다. 그 다음은 1개(단항), 그 다음은 2개(이항) 이다.
> 
> 
> 3개(삼항) 이상은 피하는게 좋으며 4개(사항)부터는 무슨일이 있더라도 사용하지 말아야 한다!
> 

인수가 많아지면 읽기도 힘들어지고, 테스트 케이스도 너무 많아지는 문제가 있다.

최선은 입력 인수가 없는 경우이며, 차선은 1개인 경우이다.

### 많이 쓰는 단항 형식

1. 인수에 질문을 던지는 경우
    1. func fileExists(”MyFile”) → Bool
2. 인수를 뭔가로 변환해서 반환하는 경우
    1. func fileOpen(”MyFile”) → File
3. 이벤트 형식
    1. func passwordAttemptFailedNtimes(attempts: Int)
    2. 이벤트 함수의 경우 이벤트란것이 명확히 드러나게 표현해야 한다.

위 3가지가 아니라면 단항 함수는 가급적 피하는 것이 좋다. 또한 입력 인수를 변환하는 함수라면 변환 결과를 반환값으로 받는 형식이 좋다.

### 플래그 인수

함수로 Bool 값을 넘기는 것은 끔찍(?)하다. 함수가 한꺼번에 여러 가지를 수행하겠다고 대놓고 표현한 것이기 때문..

```swift
// 대놓고 여러 가지를 수행하겠다고 공표한 끔찍한 함수
func render(isSuite: Bool) {
    if isSuite {
        // Do something
    } else {
        // Do something
    }
}

// 위 함수를 이렇게 두개로 나누는 것이 맞다
func renderForSuite()
func reanderForTestSingleTest()
```

### 이항 함수

인수가 2개인 함수는 1개인 함수보다 이해하기 어렵다.

인수간에 자연적인 순서가 없기 때문에 사용할 때 실수하기가 쉽다.(Swift 에서는 인수명을 명시할 수 있기 때문에 어느정도 해결은 가능할 듯 하다.)

물론 불가피하게 이항 함수를 사용해야 할 때가 있기에 이러한 위험이 따른다는 사실을 인지하고 사용해야 한다.

그리고 왠만하면 단항 함수로 바꾸려고 노력해야한다.

### 삼항 함수

인수가 3개인 함수는 2개인 함수보다 이해하기 어렵다.

이항 함수보다 생길 수 있는 문제의 수가 두배 이상 늘어난다. 더더욱 조심해서 사용하라.

### 인수 객체

인수가 2~3개 가 필요하다면, 독자적인 클래스 변수로 선언할 것을 고려한다.

```swift
// x, y 값을 원자값 형태로 받음
func makeCircle(x: CGFloat, y: CGFloat, radius: Double) -> Circle

// x, y 를 Point 라는 별도 클래스 변수로 받음
func makeCircle(at point: Point, radius: Double)
```

### 인수 목록

때로는 인수 갯수가 가변적인 경우도 필요하다. String.init(format: **String**, **CVarArg...**) 가 대표적.

이것도 사실은 다항 함수다.

이런 경우에도 삼항을 넘어가도록 사용하는 것을 지양해야 한다.

### 동사와 키워드

단항 함수는 함수와 인수가 동사/명사 쌍을 이루어야 한다.

ex) write(name)

이항 함수의 경우 인수에 대한 키워드를 추가하는 방법으로 인수의 순서를 외울 필요가 없도록 한다.

ex) assertExpectedEqualsActual(expected, actual)

<aside>
💡 Swift 에서는 assertEquals(expected: expected, actual: actual) 과 같이 인수명을 명시 하는 형식으로 사용하면 될 것 같다.

</aside>

## 부수 효과를 일으키지 마라!

```swift
func checkPassword(password: String) -> Bool {
    if password.isValid {
        self.session.initialize()
    }
    return password.isValid
}
```

위 코드는 단순히 패스워드의 유효성을 체크하는 것이 아니라 유효한 경우 세션을 초기화 시키는 부수 효과를 일으킨다.

이름만 봐서는 세션을 초기화 한다는 것을 예상할 수 없다.

이런 함수는 세션을 초기화해도 되는 일부 경우에만 호출할 수 있는 함수가 돼버린다.(시간적 결합을 초래)

### 출력 인수

인수가 출력 인수인지 아닌지는 선언부를 봐야 명확진다. 함수 선언부를 찾아본다는 것은 인지적으로 거슬린다.

<aside>
💡 Swift 에서는 함수의 인자가 변수가 아닌 상수이므로 함수 선언부까지 가지 않더라도 확인이 가능하다.

</aside>

함수에서 상태를 변경해야 한다면 함수가 속한 객제 상태를 변경하는 방식을 선택한다.

ex) report.appendFooter()

## 명령과 조회를 분리하라!

> 함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야한다.
> 

```swift
// 이름이 attibute 인 속성을 찾아서 값을 value 로 설정하며, 성공했을 때 true 를 반환하는 함수이다.
func set(attribute: String, value, String) -> Bool

// 그래서 아래와 같은 코드가 탄생한다.
if set("username", "unclebob")...
```

위 코드는 set 함수의 동작을 보기전에는 도대체 뭘 하는 코드인지 알 수 없다.

명령과 조회를 분리해서 애초부터 혼란을 만들지 말자.

```swift
if attributeExists("username") {
    setAttribute("username", "unclebob")
}
```

## 오류 코드보다 예외를 사용하라!

함수에서 오류 코드를 반환하면 호출자는 해당 오류를 곧바로 처리해야 한다는 문제에 부딪힌다.

오류 코드를 반환하는 대신 예외를 사용하라!(try/catch)

```swift
// 함수에서 에러를 반환
if deletePage(page) == .ok {
    if deleteRef(ref) {
        ...
    } else {
        ...
    }
} else {
    print("deletePage error")
}

// 예외를 사용한 처리
// 더 깔끔하다!

do {
    try deletePage(page)
    try deleteRef(ref)
} catch {
    print(error)
}
```

### Try/Catch 블록 뽑아내기

try/catch 블록은 원래 추하다. 그러므로 try/catch 을 뽑아내는 편이 좋다.

아래와 같이 정상 동작과 오류 처리 동작을 분리하면 코드를 이해하고 처리하기가 쉬워진다.

```swift
func delete(page: Page) {
    do {
        try deletePageAndRef(page)
    } catch {
        print(error)
    }
}
```

### 오류 처리도 한 가지 작업이다.

함수는 ‘한 가지’ 작업만 해야한다. 오류 처리도 ‘한 가지’ 작업에 속한다. 그러므로 오류 처리를 하는 함수는 오류 처리만 해야 마땅하다.

## 반복하지 마라!

> 중복은 소프트웨어에서 모든 악의 근원이다. 많은 원칙과 기법이 중복을 없애거나 제어할 목적으로 나왔다.
> 

E. F. 커드E. F. Codd는 자료에서 중복을 제거할 목적으로 관계형 데이터베이스에 정규 형식을 만들었다.

객체 지향 프로그래밍은 코드를 부모 클래스로 몰아 중복을 없앤다.

하위 루틴을 발명한 이래로 소프트웨어 개발에서 지 금까지 일어난 혁신은 소스 코드에서 중복을 제거하려는 지속적인 노력으로 보인다.

## 구조적 프로그래밍

Dijkstra 는 모든 함수와 함수 내 모든 블록에 입구(entry)와 출구(exit)가 하나만 존재해야 한다고 말했다.

즉, 함수는 return 문이 하나여야 한다는 말이다. 또한 루프 안에서 break나 continue를 사용해선 안 되며 goto는 절대로 안 된다.

다만, 함수가 작으면 위 규칙은 별 이익을 제공하지 못한다.

함수가 작다면 return, break, continue를 여러 차례 사용해도 괜찮다.

## 함수를 어떻게 짜죠?

> 소프트웨어를 짜는 행위는 여느 글짓기와 비슷하다.
> 

초안은 대체로 서투르고 어수선하게 작성되나, 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거하고, 함수를 줄이고 순서도 바꾼다. 때로는 전체 클래스를 쪼개기도 한다. 이 와중에도 코드는 항상 단위테스트를 통과한다.

처음부터 탁 짜내는 사람은 없다.

## 결론

> 함수는 그 언어에서 동사이며 클래스는 명사다.
> 

대가(Master) 프로그래머는 시스템을 (구현할) 프로그램이 아닌 (풀어갈) 이야기로 여긴다.

함수가 분명하고 정확한 언어로 깔끔하게 맞아 떨어져야 이야기를 풀어가기 쉬워진다.

