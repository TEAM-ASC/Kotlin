### Chapter2. 코틀린 기초



#### 함수

```kotlin
// 블록이 본문인 함수
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
// 식이 본문인 함수
fun max2(a: Int, b: Int): Int = if (a > b) a else b
// 식입 본문인 함수는 return type 생략 가능 (feat. 타입 추론)
fun max3(a: Int, b: Int) = inf (a > b) a else b
```

* 함수 선언은 `fun` 키워드로 시작

* fun 다음에 `함수명`을 명시

* 함수 이름 뒤에 괄호 안에 파라미터들 명시

  * 변수 선언과 마찬가지로 파라미터 뒤에 `:`을 통해 타입을 명시

* 본문

  * 블록이 본문인 함수: 중괄호로 본문을 감싼 형태

  * 식이 본문인 함수: 중괄호 대신 등호와 식을 이용한 형태

    > **TODO - 알고가자!**
    >
    > 문(statement)과 식(expression)의 차이
    >
    > * 식은 **값을 만들어** 내며 다른 식의 하위 요소로 계산에 참여할 수 있으나 문은 자신을 둘러싸고 있는 가장 안쪽 블록의 최상위 요소로 존재하며 **아무런 값도 만들어내지 않는다**



#### 변수

```kotlin
// 타입 표기 생략 (feat. 타입 추론)
val question = "TEAM-Android Study Coworker !"
val answer1 = 42
// 타입 명시
val answer2: Int = 42
// 초기화를 하지 않는 경우 타입 명시 필수!
val answer3: Int
answer3 = 42
```

* 코틀린은 변수를 선언할 때에 변수명 뒤에 `:`을 통해 명시

  > **TODO - 알고가자 !**
  >
  > 변수명을 뒤에 명시하는 이유?
  >
  > * 타입을 생략할 경우 식과 변수 선언이 구별을 할 수 없기 때문에 변수명 뒤에 명시하거나 생략하도록 설계되었다.

* 자바와 마찬가지로 부동소수점 사용시 `Double` 타입이 된다.

변경 가능한 변수와 변경 불가 함수

* `val`: **변경 불가능한 값**을 저장하는 변수. 일단 초기화하면 재대입이 불가.

  * 딱 1번만 초기화가 가능
  * 자바의 `final 변수`에 해당
  * val 참조 자체가 불변일지라도 그 참조가 가리키는 객체의 내부 값은 변경될 수 있다. 

  *하기 코드는 올바른 코드이다.*

  ```kotlin
  val languages = arrayListOf("java") // 불변 참조 선언
  languages.add("kotlin") // 참조가 가리키는 객체 내부를 변경
  
  // message를 한번만 초기화한다는 것을 컴파일러가 알 수 있어 올바른 코드이다.
  val message: String
  if (canPerformOperation()) {
      message = "Success"
  } else {
      message = "Failed"
  }
  ```

  > **TODO - 의논해보자!**
  >
  > val 객체의 내부 값이 변경 가능한 이유는 무엇일까?
  >
  > * 태형's Think
  >   * 코틀린은 자바와 동일하게 기본적으로 `call by value` 이며 객체 전달 시 메모리 주소를 전달하므로 객체 내부를 변경하여도 메모리 값이 변하는 것이 아니기 때문에 변경이 가능하다.
  >
  > **TODO - 고민해보자!**
  >
  > 호출되는 순서를 고민해보자.
  >
  > ``` Kotlin
  > object Main {
  >     @JvmStatic
  >     fun main(args: Array<String>) {
  >         callByValue(funA())
  >         // funA
  >         // callByValue
  > 
  >     }
  > 
  >     fun callByValue(b: Boolean): Boolean {
  >         println("callByValue")
  >         return b
  >     }
  > 
  >     val funA: () -> Boolean = {
  >         println("funA")
  >         true
  >     }
  > }
  > ```
  >
  > ```kotlin
  > object Main {
  >     @JvmStatic
  >     fun main(args: Array<String>) {
  >         callByName(funA)
  >         // callByName
  >         // funA
  > 
  >     }
  > 
  >     fun callByName(f: () -> Boolean): Boolean {
  >         println("callByName")
  >         return f()
  >     }
  > 
  >     val funA: () -> Boolean = {
  >         println("funA")
  >         true
  >     }
  > }
  > ```
  >
  > 
  >
  > *callByValue정답: funA() -> callByValue()*
  >
  > *callByName 정답: callByName() -> funA()*
  >
  > 
  >
  >
  > callByName의 이점?
  >
  > ```kotlin
  > object Main {
  >     @JvmStatic
  >     fun main(args: Array<String>) {
  >         val condition = false
  > 
  >         callByValue(condition, doSomething())
  >         //doSomething
  >         //callByValue
  > 
  >     }
  > 
  >     fun callByValue(condition: Boolean, value: Int) {
  >         println("callByValue")
  > 
  >         if (condition) {
  >             println(value)
  >         }
  >     }
  > 
  >     val doSomething: () -> Int = {
  >         // 굉장히 오래 걸리는 연산
  >         println("doSomething")
  >         1
  >     }
  > }
  > ```
  >
  > ```kotlin
  > object Main {
  >     @JvmStatic
  >     fun main(args: Array<String>) {
  >         val condition = false
  > 
  >         callByName(condition, doSomething)
  >         //callByName
  >     }
  > 
  >     fun callByName(condition: Boolean, value: () -> Int) {
  >         println("callByName")
  > 
  >         if (condition) {
  >             println(value())
  >         }
  >     }
  > 
  >     val doSomething: () -> Int = {
  >         // 굉장히 오래 걸리는 연산
  >         println("doSomething")
  >         1
  >     }
  > }
  > ```
  >
  > * callByValue의 경우 condition에 상관 없이 doSomething()이 실행되며 비효율적으로 동작하게 되지만, callByName을 사용하는 경우 condition 값에 따라 doSomething()이 실행되므로 효율적인 측면에서 더 뛰어나다.

* `var`: 변경 가능한 참조. 변수 타입은 고정.

  * 자바의 일반 변수에 해당

  * 타입은 변환시킬 수 없음

    ```kotlin
    var answer = 42
    answer = "no answer" // "error: type mismatch" 컴파일 오류 발생
    ```



* 문자열 템플릿

  ```kotlin
  val name ="TEAM-ASC"
  println("Hello, $name")
  println("Hello, ${name}")
  println("\$name의 값 = $name") // \$ 탈출문자 사용
  println("max(1, 2) = ${max(1, 2)}") // 중괄호 안에서 식 사용
  println("args: ${if (args.isEmpty()) "empty" else args[0]}") // 식에서 큰 따옴표 사용
  ```

  

#### 클래스

```kotlin
// kotlin
class Person(val name: String)
```

```java
// java
public class Person {
    private final String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    public String getString() {
        return name;
    }
}
```

* 코틀린 클래스의 기본 접근지정자가 **public** 으로 생략 가능
* getter/setter 메소드를 기본적으로 제공하여 생략 가능



#### 프로퍼티

자바의 필드와 접근자 메소드(getter/setter 메소드)를 완전히 대신함

```kotlin
class Person(
	val name: String, // 읽기 전용(val) 프로퍼티
    var isMarried: Boolean // 변경 가능(var) 프로퍼티
)
```

```kotlin
Person p = Person("Bob", false)
println(p.name)
println(p.isMarried)
p.isMarried = true;
```

* val 프로퍼티: 읽기 전용 프로퍼티로 **private 필드**와 필드를 읽는 **pulbic getter()** 를 생성함(backing 필드)

  > **TODO - 찾아보자 !**
  >
  > 프로퍼티에 접근지정자 (private)을 지정했을 경우에 private 필드 + private getter()가 되어 접근이 안되는 것인가? 아니면 접근 지정자 지정을 안했을 때에 default가 public 필드인가?



* var 프로퍼티: 읽고 쓰기가 가능한 프로퍼티로 **private 필드**와 **public getter()/public setter()** 를 생성함(backing 필드)
* 뿐만 아니라 **생성자가 필드를 초기화 하는 구현**이 내부적으로 구현되어 있음

> **TODO - 알고가자!**
>
> backing 필드 ?
>
> * 프로퍼티의 값을 저장하기 위한 비공개 필드
>
> 프로퍼티 이름이 is로 시작할 경우
>
> * 프로퍼티 이름과 동일한 getter() 생성: 예, isMarried()



#### 커스텀 접근자

```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
    	get() { // 프로퍼티 getter 선언, 블록 사용
            return height == width;
        }
    
    val size: Int
    	get() = height * width // 식 사용
}
```

* 위와 같이 사용 시 해당 프로퍼티에 접근할 때마다 getter가 프로퍼티 값을 매번 다시 계산함



#### 소스코드 구조

* 파일의 맨 앞에 package 문 사용해서 패키지 지정
* 파일의 모든 선언 (클래스, 함수, 프로퍼티 등)이 해당 패키지에 속함
* 디렉토리 구조와 패키지 구조가 일치할 필요 없음
* 같은 패키지에 속해있다면 다른 파일에서 임포트 없이 정의한 선언 사용 가능
* 다른 패키지에서 사용하려면 `import` 키워드로 사용할 선언을 임포트해야 함



#### enum

`enum` 키워드를 사용하여 **열거타입** 지정

```kotlin
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

* 자바는 `enum`, 코틀린은 `enum class`
* 코틀린에서의 enum은 **소프트 키워드(soft keyword)** 라고 부름
* class 앞에 붙여질 경우 특별한 의미를 지니지만 다른 곳에서는 이름에 사용할 수 있음(예약어처럼 사용이 불가하지 않음)

프로퍼티와 메소드 선언 가능 (메소드 선언시 마지막 열거 값 뒤에 세미콜론 **필수**)

```kotlin
enum class Color(val r: Int, val g: Int, val b: Int) {
    READ(255, 0, 0), ORANGE(255, 165, 0), YELLOW(255, 255, 0),
    BLUE(0, 0, 255), VIOLET(238, 130, 238);
    
    fun rgb() = (r * 256 + g) * 256 + b
}

println(Color.BLUE.rgb()) // 결과: 255
```



#### when

자바의 `switch`와 유사하며 코틀린의 `when`은 `if`와 마찬가지로 **값을 만들어내는 식** 임

```kotlin
fun getColorString(color: Color) =
	when (color) {
        Color.RED -> "It is RED !"
        Color.ORANGE -> "It is ORANGE !"
        Color.YELLOW -> "It is YELLOW !"
        Color.GREEN, Color.BLUE -> "It is GREEN OR BLUE !"
        else -> "What is Color..?"
    }

println(getColorString(Color.ORANGE)) // 결과: It is ORANGE
```

* 각 분기에 `break` 키워드 필요 없음
* `,`를 통해 여러 매치 패턴을 지정할 수 있음
* 모든 분기 식에 만족하지 않으면 else 분기가 실행됨



when 식은 객체의 동등성 사용

```kotlin
fun mix(c1: Color, c2: Color) =
	when (setOf(c1, c2)) {
        setOf(RED, YELLOW) -> ORANGE
        setOf(YELLOW, BLUE) -> GREEN
        setOf(BLUE, VIOLET) -> INDIGO
        else -> throw Exception("Dirty color")
    }
```

* `setOf()`: 자바로 치면 set을 만들어 주는 메소드로 HashSet과 비슷하다고 생각하면 됨
* c1, c2가 들어오는 순서에 상관이 없음



인자 없는 when 식

```kotlin
fun mixOptimized(c1: Color, c2: Color) = 
    when {
        (c1 == RED && c2 == YELLOW) ||
        (c1 == YELLOW && c2 == RED) -> 
            ORANGE
        (c1 == YELLOW && c2 == BLUE) ||
        (c1 == BLUE && c2 == YELLOW) -> 
            GREEN
        (c1 == BLUE && c2 == VIOLET) ||
        (c1 == VIOLET && c2 == BLUE) ->
            INDIGO
        else -> throw Exception("Dirty color")
    }

println(mixOptimized(BLUE, YELLOW)) //결과: GREEN
```

* when에 인자가 없으려면 각 분기의 조건이 Boolean 결과를 계산하는 식이어야 함

> **TODO - 생각해보자!**
>
> * 불필요한 인스턴스를 생성하지 않아 불필요한 가비지 객체가 늘어나지 않는 장점이 있으나 가독성이 매우 떨어지는 코드가 될 수 있어 주의하여 사용하는 것이 좋을 것 같다.



#### 스마트 캐스트

Object의 타입 확인과 변환을 한번에 해주는 기능

```kotlin
fun eval(e: Expr): Int {
    if (e is Num) {
        val n = e as Num // 명시적 형변환, 스마트 캐스트로 사실상 필요 없음
        return n.value
    }
    
    if (e is Sum) {
        return eval(e.left) + eval(e.right)
    }
    throw IllegalArgumentException("Unknown expression")
}
```

```kotlin
fun eval(e: Expr): Int {
    when (e) {
        is Num -> {
            println("num: ${e.value}")
            e.value // 블록에서는 마지막 식이 반환값이 됨
        }
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression")
    }
}
```

* `is` 연산자를 통해 변수 타입 검사
  * 검사한 이후에는 명시적인 캐스팅 없이 해당 타입으로 바로 사용 가능
* 블록으로 되어있는 경우 마지막 식이 반환값이 되어 반환됨



#### while 루프

자바의 `while 루프`와 동일하게 사용된다.

```kotlin
while (조건) {
    // TODO 조건이 참인 경우 반복 실행
}

do {
    // TODO 최초 1번 실행 후 조건이 참인 경우 반복 실행
} while (조건)
```



#### for 루프

* 범위(CloseRange 인터페이스): 두 값으로 이루어진 구간
* 수열(Progression): 범위에 속한 값을 일정한 순서로 이터레이션
* 예시
  * `1 rangeTo 10 step 2` 또는 `1..10 step 2`: 1~10 까지 2씩 증가하며 이터레이션
  * `100 downTo 1 step 2`: 100부터 1로 줄어들며 2씩 감소하며 이터레이션
  * `0 until 10`: 0부터 10까지 이터레이션(단, 10은 미포함)

자바의 `for (int i = 0; i < length; i++)`에 해당하는 루프가 없다. 대신 `범위`를 사용한다.

```kotlin
// 범위 1~100 (100 포함)
val oneToTen = 1..100
```



#### 맵, 리스트에 대한 이터레이션

```kotlin
val binaryReps = TreeMap<Char, String>()
for (c in 'A'..'F') {
    val binary = Integer.toBinaryString(c.toInt())
    binaryReps[c] = binary; // 자바의 put(), get() 대신에 이와 같이 사용함
}

// 맵에 대한 이터레이션
// letter에는 키, binary에는 값(2진 표현)이 들어감
for ((letter, binary) in binaryReps) {
    println("$letter = $binary")
}

// 결과: A = 1000001
//      B = 1000010
//      C = 1000011
//      D = 1000100
//      E = 1000101
//      F = 1000110

val list = arrayListOf("10", "11", "1001")
for ((index, element) in list.withIndex()) {
    print("$index = $element")
}

// 결과: 0 = 10
//      1 = 11
//      2 = 1001
```



#### `in`을 통해 값이 범위에 속하는지 검사하기

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'

println(isLetter('q')) // 결과: true
println(isNotDigit('x')) // 결과: true

// Comparable 구현 클래스
println("Kotlin" in "Java".."Scala")
println("Kotlin" in setOf("Java", "Scala", "Kotlin")) // 결과: true
```

* `in`은 `contains`와 동일



#### 익셉션(Exception)

자바나 다른 언어의 예외 처리와 비슷하다. 즉, 함수 실행 중 오류가 발생하면 예외를 던질(throw) 수 있고 함수를 호출하는 쪽에서는 그 예외를 잡아 처리(catch)할 수 있다. 예외에 대해 처리를 하지 않은 경우 함수 호출 스택을 거슬러 올라가면서 예외를 처리하는 부분이 나올 때까지 예외를 다시던진다(rethrow).

```kotlin
if (percentage !in 0..100) {
    throw IllegalArgumentException("message: $percentage")
}
```

```kotlin
fun readNumber(reader: BufferedReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    } catch (e: NumberFormatException) {
        return null
    } finally {
        reader.close()
    }
}

val reader = BufferedReader(StringReader("239"))
println(readNumber(reader)) // 결과: 239
```

* 자바와의 가장 큰 차이점은 함수명 뒤에 `throws` 절이 없다는 것임

  * 자바에서 상기 코드는 함수 뒤에 throws IOException을 붙여야 함

  * IOException이 **체크 예외**이기 때문에 자바는 명시적으로 표현해야 함

    > **TODO - 알고가자!**
    >
    > Java에서의 체크 예외
    >
    > * ClassNotFoundException
    > * CloneNotSupportedException
    > * InstantiationException
    > * IOException
    >
    > Java에서의 언체크 예외
    >
    > * RuntimeException을 상속받는 Exception
    >   * ArithmeticException
    >   * IllegalArgumentException
    >   * IndexOutOfBoundsException



#### try는 식

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return // null로 수정 시 null을 반환하게 된다.
    } finally {
        reader.close
    }
    
    println(number) // Exception 발생 시 호출되지 않음
}

val reader = BufferedReader(StringReader("not a number"))
readNumber(readNumber(reader)) // 결과: 아무것도 출력되지 않음.
```







###### 참조

* [코틀린 인 액션 스터디 정리 자료 :: 자바캔(Java Can Do IT)](https://javacan.tistory.com/entry/Kotlin-in-action-study-summary)
* [탤로우 월드#Kotlin](https://taehyungk.github.io/tag/#Kotlin)

