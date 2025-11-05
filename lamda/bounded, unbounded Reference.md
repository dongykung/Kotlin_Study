# callable reference
함수나 프로퍼티를 값으로 다룰 수 있게 해주는 기능<br>

<br><br>
::을 사용하는 식을 `멤버 참조`라고 부르며 한 메서드를 호출하거나, 한 프로퍼티에 접근하는 함수 값을 만들어 줍니다.

## 생성자 참조
생성자 참조(constructor reference)를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있습니다.
```kotlin
fun main() {
  val createPerson = ::Person
  val p = createPerson("Dongkyung", 21)
  println(p)
  // Person(name = Dongkyung, age = 21)
}
```
## unbounded reference
특정 인스턴스에 연결되지 않은 참조입니다. 특정 클래스의 멤버를 참조하지만, 어떤 객체의 멤버인지는 지정하지 않습니다.
```kotlin
data class Person(
    val name: String,
    var age: Int
)

fun main() {
    val dk = Person("Dongkyung", 25)
    val personName: (Person) -> String = Person::name
    val personAge: (Person) -> Int = Person::age


    println(personName(dk))
    println(personAge(dk))
}

=======================================================

public final class MainActivityKt {
   public static final void main() {
      Person dk = new Person("Dongkyung", 25);
      Function1 personName = (Function1)null.INSTANCE;
      Function1 personAge = (Function1)null.INSTANCE;
      System.out.println(personName.invoke(dk));
      int var3 = ((Number)personAge.invoke(dk)).intValue();
      System.out.println(var3);
   }

   // $FF: synthetic method
   public static void main(String[] args) {
      main();
   }
}
```
- `수신 객체(Receiver)`가 첫 번째 인자로 추가됩니다.
- map, filter, maxBy등 고차함수를 사용할 때 유용합니다.
```kotlin
data class Person(
    val name: String,
    var age: Int
)

fun main() {
    val people = listOf(Person("Alice", 31), Person("Bob", 21))
    val sortedList = people.sortedBy(Person::age)

    println(sortedList)
}
```
즉 수신객체를 파라미터로 받기 때문에 Collection 에서 제공하는 고차함수를 사용할 때 유용하게 사용할 수 있습니다.

<br><br>

## bounded reference
특정 인스턴스에 이미 연결된 참조입니다. 호출할 때 인스턴스를 전달할 필요가 없습니다. <br> <br>
여기서 중요한건 특정 인스턴스와 연결되어 있으며 내부적으로 수신 객체를 저장하고 있습니다. <br>레퍼런스를 호출할 때 저장된 수신 객체가 메서드를 호출하도록 합니다.
```kotlin
data class Person(
    val name: String,
    val age: Int
)

fun main() {
    val alice = Person("Alice", 25)
    val aliceName = alice::name 
    val aliceAge = alice::age    

    println(aliceName())
    println(aliceAge())
}

==================================================================

public final class MainActivityKt {
   public static final void main() {
      Person dk = new Person("DongKyung", 25);
      Function0 dkName = (Function0)(new PropertyReference0Impl(dk) {
         public Object get() {
            return ((Person)this.receiver).getName();
         }
      });
      Function0 dkAge = (Function0)(new PropertyReference0Impl(dk) {
         public Object get() {
            return ((Person)this.receiver).getAge();
         }
      });
      System.out.println(dkName.invoke());
      int var3 = ((Number)dkAge.invoke()).intValue();
      System.out.println(var3);
   }

   // $FF: synthetic method
   public static void main(String[] args) {
      main();
   }
}
```
- 콜백 전달할 때 많이 사용합니다. (ex: viewModel::updateTitle)
- 수신 객체를 receiver 필드에 저장하기 위해 새로운 Function 객체를 new로 생성하고 있습니다.
- 말 그대로 특정 인스턴스의 멤버 참조가 필요할 때 사용합니다.

## 정리
즉, 어떤 것을 사용해야 된다가 아닌 상황에 따라 맞게 사용하면 됩니다. <br> 
Collection의 고차함수를 사용할 때 가독성 측면에서 unbounded reference를 사용할 수 있습니다. <br>
특정 인스턴스의 멤버 참조가 필요한 경우 bounded reference를 사용하면 됩니다.

<br>

마무리 하기 전 재밌는 퀴즈가 있습니다. <br>
아래 코드의 출력은 어떻게 될가요? 이는 unbounded가 어떻데 디컴파일 되는지 알면 쉽게 알 수 있습니다.
```kotlin
fun main() {
    var dk = Person("DongKyung", 25)
    val dkName: () -> String = dk::name
    val dkAge: () -> Int = dk::age

    dk = Person("NewDongKyung", 20)

    println(dkName())
    println(dkAge())
}
```
[멤버 참조와 람다](https://github.com/dongykung/Kotlin_Study/blob/main/%EB%A9%A4%EB%B2%84%20%EC%B0%B8%EC%A1%B0(%3A%3A)%EC%99%80%20%EB%9E%8C%EB%8B%A4.md) 글을 확인해보시면 해당 문제를 이해하는데 도움을 받으실 수 있습니다. <br>
정답은 Dongkyung, 25가 출력됩니다.
