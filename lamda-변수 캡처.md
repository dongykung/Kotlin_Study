람다를 함수 안에서 정의하면 함수의 파라미터뿐 아니라 람다 정의보다 앞에 선언된 로컬 변수까지 람다에서 모두 사용할 수 있습니다. <br>
이에 대한 간단한 예시로 forEach문을 살펴보겠습니다.
```kotlin
fun printMessages(messages: List<String>) {
    var messageCount = 0
    messages.forEach {
        messageCount++
        println("$messageCount: $it")
    }
}

fun main() {
    val myMessages = listOf("hello", "kotlin")
    printMessages(myMessages)
}
/**
messageCount = 1: hello
messageCount = 2: kotlin
*/
```
`messageCount`와 같이 람다 안에서 접근할 수 있는 외부 변수를 `람다가 캡처한 변수`라고 부릅니다.

<br><br>

## 함수와 로컬 변수의 생명주기
보통 함수 안에 정의된 로컬 변수의 생명주기는 함수가 반환되면 끝납니다. 
<br> <br>
함수안에서 생성되는 로컬 변수의 경우 JVM의 `Stack 영역`에 저장되며 함수가 종료되면 Stack Frame이 사라지면서 같이 사라지며 함께 제거되는 것이 일반적입니다.

> 자신의 로컬 변수를 캡처한 람다를 반환하거나 다른 변수에 저장한다면 로컬 변수의 생명주기와 함수의 생명주기가 달라질 수 있습니다.
```kotlin
fun counterFactory(): () -> Int {

    var count = 0
    
    val counterLambda = {
        count++
        println("캡처된 count 값: $count")
        count
    }

    return counterLambda
}

fun main() {
    val myCounter = counterFactory()
    myCounter() // 캡처된 count 값: 1
    myCounter() // 캡처된 count 값: 2
    myCounter() // 캡처된 count 값: 3
    
    val myCounter2 = counterFactory()
    myCounter2() // 캡처된 count 겂: 1
}
```
counterFactory 함수의 로컬 변수인 `count`는 함수가 종료되어도 캡처한 변수를 읽거나 쓸 수 있는 것을 확인했습니다. <br>
도대체 어떻게 이게 가능한 걸가요?

<br><br>

## 변경 가능한 변수 캡처하기: 내부 구현
위에서 함수의 로컬 변수는 Stack 메모리에 저장되며 함수가 종료되며 같이 사라진다고 했습니다. <br>
하지만 위 결과를 보았을 때 캡처한 변수를 함수가 종료되어도 계속 사용할 수 있었습니다.
<br><br>

이를위해, 변경 가능한 변수에 대한 참조를 저장하는 `클래스`를 `내부적`으로 선언합니다. <br>
위 `counterFactory()` 함수의 디컴파일된 코드를 함께 살펴보겠습니다.
```kotlin
public static final Function0 counterFactory() {
      Ref.IntRef count = new Ref.IntRef();
      Function0 counterLambda = MainActivityKt::counterFactory$lambda$0;
      return counterLambda;
}
```
count가 `Ref`라는 인스턴스로 바뀌었는데요 이게 뭘까요?
```kotlin
public class Ref {
    private Ref() {
    }
    ...
    public static final class IntRef implements Serializable {
        public int element;

        public String toString() {
            return String.valueOf(this.element);
        }
    }
}
```
이는 컴파일러가 생성한 래퍼 클래스로 감싸 `Heap 영역`에 저장되게 합니다. <br>
함수 또한 `static final`로 지정되어 
