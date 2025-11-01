해당 내용은 Kotln IN ACTION 5장 내용 기반으로 작성되었습니다.

# 람다
- 다른 함수에 넘길 수 있는 작은 코드 조각
- 함수를 값처럼 다룰 수 있는 익명 함수

"이벤트가 발생하면 어떤 핸들러를 실행시켜아지" 동작을 하는 코드를 작성하고 싶을 때 어떤 방법이 있을가요? <br>
예전 자바8 이전에서는 익명 내부 클래스를 사용했는데요 람다와 어떤 차이가 있을가요?

```kotlin
// 메서드가 하나뿐인 인터페이스(SAM)
public interface OnClickListener {
    void onClick(View var1);
}

// 자바8 이전 익명 내부 클래스 방식
button.setOnClickListener(object: OnClickListener {
    override fun onClick(v: View) {
    println("clicked")
    }
})

// 자바8 이후 람다
button.setOnClickListener {
    println("clicked")
}
```
이처럼 람다는 메서드가 하나뿐인 익명 객체 대신 사용할 수 있고 함수를 값처럼 다룰 수 있습니다.

<br>
<br>

## 람다식 문법


### 함수 타입은 람다의 파라미터 타입과 반환 타입을 지정한다
람다를 인자로 받는 함수를 정의하기 위해 람다 파라미터의 타입을 어떻게 선언할 수 있는지 알아야 합니다. <br>
람다를 로컬 변수에 대입하는 경우를 보겠습니다.

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
val action: () -> Unit = { println(42) }
```
이처럼 함수 타입을 정의하기 위해선 함수 파라미터의 타입을 괄호 안에 넣고 그 뒤에 화살표를 추가한 다음, 함수의 반환 타입을 지정하면 됩니다.
```kotln
  파라미터 타입    반환 타입
(Int, String) -> Unit
```

함수 타입에서도 반환 타입을 `Null`이 될 수 있는 타입으로도 지정할 수 있습니다.
```kotlin
var canReturnNull: (Int, Int) -> Int? = { x, y -> null }
```
<br>

`Null`이 될 수 있는 함수 타입 변수 또한 가능합니다.
```kotlin
var funOrNull: ((Int, Int) -> Int)? = null
```

<br>

### 인자로 전달 받은 함수 호출
위에서 함수 타입을 선언하는 방법을 살펴보았습니다. <br>
이번에는 고차 함수를 작성하는 방법에 대해 알아보겠습니다.

```kotlin
fun mySum(operation: (Int, Int) -> Int) {
  val result = operation(2, 3)
  println("result = $result")
}

fun main() {
  mySum { a, b -> a + b }
}
```
만약 파라미터에 이름을 붙이고 싶다면 아래와 같이 가능합니다.
```kotlin
fun mySum(operation: (operandA: Int, operandB: Int) -> Int) {
  val result = operation(2, 3)
  println("result = $result")
}

fun main() {
  mySum { operandA, operandB -> operandA + operandB }
  mySum { a, b -> a + b } // 그냥 다른 이름 붙여도 됨
}
```

우리가 흔히 쓰는 `filter` 내부 코드를 같이 확인해보겠습니다.
```kotlin
fun main() {
    val a = "hello LAMDA"
    val b= a.filter { it.isLowerCase() }
    println(b) // "hello"
}

public inline fun String.filter(predicate: (Char) -> Boolean): String {
    return filterTo(StringBuilder(), predicate).toString()
}

public inline fun <C : Appendable> CharSequence.filterTo(destination: C, predicate: (Char) -> Boolean): C {
    for (index in 0 until length) {
        val element = get(index)
        if (predicate(element)) destination.append(element)
    }
    return destination
}
```
`predicate` 파라미터는 문자를 파라미터로 받고 Boolean 결괏값을 반환합니다.<br>
filter 함수가 돌려주는 결과 문자열에 인자로 받은 문자가 남아 있기를 바라면 true를 반환하고, 문자열에서 거르고 싶다면 false를 반환하면 원하는 결과를 얻을 수 있습니다.
