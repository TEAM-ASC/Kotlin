### Chapter6. 코틀린 타입 시스템

#### 1. 널 가능성(Nullability)

* 물음표 기호 ' ? '를 사용하여 Null이 될 수 있는 여부를 컴파일러가 미리 감지하게 함.

* NPE처리를 위해 Nullable타입을 명시적으로 지원

  * ```Java
    public void strLen(@NotNull String s1, @Nullable String s2) {...}
    ```

  * ```kotlin
    fun strLen(s1 : String, s2 : String?) {...}
    ```



Ex)

```kotlin
fun strLen(s : String) = s.length
fun strLenSafe(s : String?) = s!!.length	//모순...
fun strLenSafe2(s : String?) = if(s != null) s.length else 0

fun main(args: Array<String>) {
    println(strLen("abc"))			// 3
    println(strLenSafe("abc"))		// 3
    println(strLenSafe2("abc"))		// 3
    
    val x: String? = null
    println(strLenSafe(x))			// ? 
}
```

> **Kotlin에서는 NPE가 일어나지 않는가?**
>
> - NullPointerException을 상속한 KotlinNullPointerException이 발생
> - 인자로 넘겨준 값이 Nullable이라면 함수내에서도 Nullable 통일!
>
> ```kotlin
> public open class KotlinNullPointerException : NullPointerException {
>     constructor()
> 
>     constructor(message: String?) : super(message)
> }
> ```



#### 1.1 Safe Call 연산자

* Safe Call 연산자는 ' .? ' 을 붙여 사용
* Null검사와 메소드 호출을 한 번의 연산으로 수행

Ex.1) Safe Call 연쇄시키기1

```kotlin
fun printAllCaps(s: String?) {
    val allCaps: String? = s?.toUpperCase()
    println(allCaps)
}

fun main(args: Array<String>) {
    printAllCaps("abc")			//ABC
    printAllCaps(null)			//null
}
```

Ex.2) Safe Call 연쇄시키기2

```kotlin
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
   val country = this.company?.address?.country
   return if (country != null) country else "Unknown"
}

fun main(args: Array<String>) {
    val person = Person(name = "Dmitry", company = null)
    println(person.countryName())
}
```

> **생각해보자!**
>
> 위의 **Safe Call 연산자**를 보고 처음엔 읽기 불편했지만 아래의 예시를 보고 느낌이 왔다.
>
> ```Kotlin
> val country = this.company?.address?.country
> return if (country != null) country else "Unknown"
> ```
>
> 저 부분만 Java로 변환하면
>
> ```Java
> if(this.getCompany() != null 
>    && this.getCompany().getAddress() != null 
>    && this.getCompany().getAddress().getCountry() != null)
> {
>     String country = this.getCompany().getAddress().getCountry();
>     return country;
> }else{
>     return "Unknown";
> }
> ```



#### 1.2 Elvis 연산자

* **널복합 연산자**(Null coalescing) 라고도 함
* 이항연산자로 좌항을 계산한 값이 Null인 경우, 특정 값(우항)으로 값을 할당
* 예외 처리에 유용
* 삼항 연산자 + Null 처리(Kotlin에서는 삼항 연산자를 지원하지 않음)

Ex .1)  Safe Call 연산자 vs Elvis 연산자

```Kotlin
//Safe Call 연산자
fun Person.countryName(): String {
   val country = this.company?.address?.country
   return if (country != null) country else "Unknown"
}

//Elvis 연산자
fun Person.countryName() = this.company?.address?.country ?: "Unknown"
```

Ex.2) Elvis 연산자를 이용한 예외 처리

```Kotlin
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun printShippingLabel(person: Person) {
    val address = person.company?.address
      ?: throw IllegalArgumentException("No address")
    with (address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}

fun main(args: Array<String>) {
    val address = Address("Elsestr. 47", 80687, "Munich", "Germany")
    val jetbrains = Company("JetBrains", address)
    val person = Person("Dmitry", jetbrains)
    printShippingLabel(person)					//Elsestr. 47
    										  //80687 Munich, Germany
    printShippingLabel(Person("Alexey", null))	  //java.lang.IllegalArgumentException: No address
}
```



#### 1.3  Type Cast 연산자  : as 와 Safe Cast :as?

* as?는 변환 가능한 타입인지 검사 후, 아니면 Null값 반환

  ```kotlin
  class Person(val firstName: String, val lastName: String) {
     override fun equals(o: Any?): Boolean {
        val otherPerson = o as? Person ?: return false
  
        return otherPerson.firstName == firstName &&
               otherPerson.lastName == lastName
     }
  
     override fun hashCode(): Int =
        firstName.hashCode() * 37 + lastName.hashCode()
  }
  
  fun main(args: Array<String>) {
      val p1 = Person("Dmitry", "Jemerov")
      val p2 = Person("Dmitry", "Jemerov")
      println(p1 == p2)					//true
      println(p1.equals(42))				//false
      
      println("p1 HashCode : " + p1.hashCode())
      println("p1 HashCode : " + p2.hashCode())
  }
  ```

  > **심화학습..**	//혼자 다 설명할 수 없음...
  >
  > 위의 코드를 보면, 뜬금없이 hashCode를 재정의한 것을 볼 수 있는데
  >
  > 그 이유를 간략히 설명하자면, Effective Java에서는 다음과 같이 설명한다.
  >
  > > `equals를 재정의한 클래스에서는 hashcode도 재정의 해야한다.`
  > > 그렇지 않으면 hash를 사용하는 HashMap, HashSet과 같은 컬렉션의 원소로 사용될 때 문제가 발생할 것이다.
  > >
  > > 따라서, eqauls를 재정의 하는 경우 다음 3가지 규약을 지켜야한다.
  > >
  > > - equals비교에 사용되는 정보가 변경되지 않았다면, 객체의 hashcode 메서드는 몇번을 호출해도 항상 일관된 값을 반환해야 한다.
  > >   (단, Application을 다시 실행한다면 값이 달라져도 상관없다. (메모리 소가 달라지기 때문))
  > > - equals메서드 통해 두 개의 객체가 같다고 판단했다면, 두 객체는 똑같은 hashcode 값을 반환해야 한다.
  > > - equals메서드가 두 개의 객체를 다르다고 판단했다 하더라도, 두 객체의 hashcode가 서로 다른 값을 가질 필요는 없다. (Hash Collision)
  > >   단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.
  >
  > 참고
  >
  > 1. HashCode() 재정의 이유:
  >
  >    https://jaehun2841.github.io/2019/01/12/effective-java-item11/#%EC%84%9C%EB%A1%A0
  >
  > 2. 37을 곱한 이유:
  >
  >    https://d2.naver.com/helloworld/831311



#### 1.4 널 아님 단언(Not-null Assertion)

* 널 아님 단언은 ' !! '로 표현
* 컴파일러에게 "나는 이 값이 null이 아님을 잘 알고있으며, 예외가 발생해도 감수하겠다"라는 표현

Ex.1) Swing(프레임워크)에서 널 아님 단언 사용

```kotlin
class CopyRowAction(val list: JList<String>): AbstractAction(){
    override fun isEnabled(): Boolean = list.selectedValue != null
    override fun actionPerformed(e : ActionEvent){
        val value = list.selectedValue!!
    }
}
```

* 한 줄에 나란히 널 아님 단언이 쓰였을 경우, 어떤 식에서 발생한 예외인지 알 수 없으니 나란히 사용은 피하자

  ```Kotlin
  person.company!!.address!!.country
  ```



#### 1.5 let함수

* 널이 될 수 있는 값(Nullable)을 널이 아닌 값(NotNull)만 인자로 받을 때, 용이하게 사용
* 여러 값에 대한 null 체크 시, let을 중첩해서 사용하면 코드가 복잡해지므로 피하는 것이 좋다.
  - (if (something != null) 사용을 권장)

Ex.1) let을 사용해 Null이 아닌 인자로 함수 호출하기

```kotlin
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

fun main(args: Array<String>) {
    var email: String? = "yole@example.com"
    email?.let { sendEmailTo(it) }		//Sending email to yole@example.com
    email = null
    email?.let { sendEmailTo(it) }
}
```

> let을 호출하면 람다의 인자인 it은 널이 될 수 있는 타입으로 추론됨

#### 1.6 lateinit 변경자

* lateinit 변경자를 사용하면 Nullable, NotNull을 난잡하게 사용하지않고 프로퍼티를 나중에 초기화 할 수 있다.
* 프로퍼티가 val인 경우는 적용할 수 없고, 항상 var여야만 한다.

```Kotlin
import org.junit.Before
import org.junit.Test
import org.junit.Assert

class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private var myService: MyService? = null
    //lateinit 변경자 선언 시
	private lateinit myService : MyService
    @Before fun setUp() {
        myService = MyService()
    }

    @Test fun testAction() {
        Assert.assertEquals("foo", myService!!.performAction())
        //lateinit 변경자로 선언 시,
        Assert.assertEquals("foo", myService.performAction())
    }
}
```

* lateinit에 대한 초기화진행 여부를 확인하기 위해서는 ex) ::myService.isInitialized 형태로 분기처리하여 확인할 수 있다

#### 1.7 널 가능 타입의 확장

* null이 될 수 있는 타입에 대한 확장함수를 정의하면, Safe Call을 하지않아도 된다.

Ex.1) Null이 될 수 있는 수신 객체에 대해 확장 함수 호출

```kotlin
fun verifyUserInput(input: String?) {
    if (input.isNullOrBlank()) {	//<- input이 Nullable이지만 Safe Call 연산자를 쓰지않음
        println("Please fill in the required fields")
    }
}

fun main(args: Array<String>) {
    verifyUserInput(" ")		//Please fill in the required fields
    verifyUserInput(null)		//Please fill in the required fields
}

//String에서의 확장함수 정의
public inline fun CharSequence?.isNullOrBlank(): Boolean {
    contract {
        returns(false) implies (this@isNullOrBlank != null)
    }
    return this == null || this.isBlank()
}
```



#### 1.8 타입 파라미터의 널 가능성

* Kotlin에서는 함수나 클래스의 모든 타입 파라미터는 기본적으로 널이 될 수 있다.(t를 Any 타입으로 추론하기 때문)
* 타입 파라미터는 타입 상한을 지정하지 않는다면 Null이 될 수 있다.(t를 Any?의 타입으로 추론하기 때문)

Ex.)  널이 될 수 있는 타입파라미터와 상한 지정

```Kotlin
//널이 될 수 있는 타입 파라미터
fun <T> printHashCode(t: T) {
    println(t?.hashCode())
}

fun main(args: Array<String>) {
    printHashCode(null)		//null
}

//상한을 지정한 타입 파라미터
fun <T : Any> printHashCode(t: T) {
    println(t.hashCode())
}

fun main(args: Array<String>) {
    printHashCode(null)
    //Type parameter bound for T in fun <T : Any> printHashCode(t: T): Unit is not satisfied
}
```

참고: https://stackoverflow.com/questions/54760309/why-i-cant-assign-nullable-type-to-any-in-kotlin



#### 1.9 플랫폼 타입

* 플랫폼 타입은 Kotlin이 널 관련 정보를 알 수 없는 Java 타입을 지칭
* Kotlin은 보통 NotNull타입의 값에 대해 널 안정성을 검사하지만 플랫폼 타입의 값에 대해서는 경고하지않음

```java
//Java
public class Person{
    private final String name;
    public Person(String name){
        this.name = name;
    }
    public String getName(){
        return name;
    }
}
```

```Kotlin
//Kotlin
fun yellAtSafe(person: Person) {
    println(person.name.toUpperCase() + "!!!")
    //toUpperCase()의 수신객체의 person.name이 non-null인데 null이라 컴파일 에러 
    println((person.name ?: "Anyone").toUpperCase() + "!!!")
    //ANYONE!!!
}

fun main(args: Array<String>) {
    yellAtSafe(Person(null))
}
```

> **전부 다 널이 될 수 있는 타입으로 하면 되지않을까?**
>
>  모든 자바 타입을 널이 될 수 있는 타입으로 처리하기엔 널 안정성 검사로 인해 얻는 이익보다 쓸데없는 비용이 늘어나 프로그래머에게 타입의 널 가능성의 책임을 부여함.



#### 1.10 상속

* 자바클래스를 코틀린에서 상속할때, 파라미터나 반환타입의 Null처리를 결정해야함

```java
//Java
interface StringProcessor{
    void process(String value);
}
```

```kotlin
//Kotlin
class StringPrinter : StringProcessor{
    override fun process(value: String){		// notnull 선언
        println(value)		
    }
}

class NullableStringPrinter : StringProcessor{
    override fun process(value: String?){		//nullable 선언
        value?.let{println(it)}
    }
}
```



#### 2. 코틀린의 원시 타입

* Kotlin은 원시타입과 참조타입을 구분하지 않고, 항상 같은 타입을 사용
* 대부분, Kotlin의 Int타입은 Java의 int 타입으로 컴파일 되고, Int를 타입인자로 넘기는 경우 Integer로 컴파일된다.



#### 2.1 null이 될 수 있는 원시타입 : Int?, Boolean? 등

* 코틀린에서 null이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일 된다.

Ex.) 

```kotlin
data class Person(val name: String, val age: Int? = null) {
    fun isOlderThan(other: Person) : Boolean? {
        if (age == null || other.age == null)
            return null
        return age > other.age    
    }
}

fun main(args: Array<String>) {
    println(Person("Sam", 35).isOlderThan(Person("Amy", 42))) //false
    println(Person("Sam", 35).isOlderThan(Person("Jane")))	  //null
}
```

* age는 컴파일러에 의해 널 안정성 검사를 마친 후 값을 비교가 허용되므로, age의 프로퍼티값은래퍼 타입으로 저장된다.



#### 2.2 숫자 변환

* Kotlin은 Boolean을 제외한 모든 원시 타입에 대한 변환 함수를 제공

* Kotlin은 한 타입의 숫자를 다른 타입으로 자동변환 하지 않음.

* 명시적으로 변환 메소드를 사용해야함

  ```kotlin
  val i = 1
  val l = Long = i  // "Error: type mismatch" 컴파일 오류 발생
  
  val i = 1
  val l: Long = i.toLong()	// 명시적으로 변환 메소드 사용
  
  ---------------------------명시 이유---------------------------
  val x = 1  // Int
  val list = listOf(1L, 2L, 3L) // Long 리스트 
  x in list // 묵시적 타입 변환으로 인해 false
  
  val x = 1
  println(x.toLong() in listOf(1L, 2L, 3L)) //true
  
  fun foo(l: Long) = pinrtln(l)
  
  val b: Byte = 1 // 상수 값는 적절한 타입으로 해석 된다.
  val l = b + 1L  // +는 Byte와 Long을 인자로 받을 수 있다.
  foo(42)         // 함수 인자이므로 42를 Long으로 해석한다.
  ```



#### 2.3 Any, Any? : 최상위 타입

* Java는 Object, Kotlin은 Any가 널이 될 수 없는 타입의 최상위타입
* 내부적으로 Any는 자바의 java.lang.Object로 컴파일 된다.
* 모든 코틀린 클래스에는 toString, equals, hashCode 라는 3개의 메소드가 있다. 하지만 java.lang.Object에 있는 다른 메소드(wait나 notify 등)는 Any에서 사용할 수 없다. 그런 메소드를 사용하고 싶다면 java.lang.Object로 캐스트해야 한다.



#### 2.4 Unit 타입 : Kotlin의 void

* Kotlin의 Unit타입은 Java의 void와 같은 기능을 함.

```kotlin
fun f(): Unit{...}
fun f(){...}		//fun의 기본 선언은 void라고 생각
```

Ex.) void 와Unit의 차이

* 반환 타입으로 Unit을 반환가능
* 타입 매개변수로 Unit을 쓸 수 있다

```kotlin
interface Processor<T> {
    fun process(): T
}

//반환 타입으로 Unit을 묵시적으로 반환 가능
class NoResultProcessor : Processor<Unit> {
    override fun process() {
        // 업무 처리 코드
        
    }
}

class ResultProcessor : Processor<Integer> {
    override fun process(): Int {
        // 업무 처리 코드
        return someInt
    }
}
```

> **Kotlin은 왜 void가 아닌 Unit을 만들었을까?**
>
> Kotlin의 Unit은 Java의 void와 다르게 두 가지 특징을 가진다
>
> 1. Unit은 싱글톤 인스턴스이다. 그래서 Kotlin에서 Unit이라는 키워드는 타입이면서도 동시에 객체
>
>    ```kotlin
>    val unit : Unit = Unit
>    ```
>
> 2. Unit은 객체이기도 하기때문에 Any를 상속하는 서브 클래스이다.
>
>    ```kotlin
>    val unit : Any = Unit
>    ```
>
>    



#### 2.5 Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다.

* 함수가 정상적으로 끝나지 않는 사실을 표현할 때 사용
* Nothing 타입은 아무 값도 포함하지 않음 = return이라는 행위 자체를 하지않음

Ex. 1) 함수가 리턴 될 일이 없을 경우

```kotlin
fun infiniteLoop(): Nothing {
    while (true) {
        println("Hi there!")
    }
}
```

Ex. 2) 예외를 던지는(throw Exception)함수의 리턴 타입

```kotlin
fun throwException(): Nothing {
    throw IllegalStateException()
}
```

Ex. 3) 함수의 리턴 타입이 Nothing? 일 경우

```kotlin
fun mayThrowAnException(throwException: Boolean): Nothing? {
    return if (throwException) {
        throw IllegalStateException()
    } else {
        println("Exception not thrown :)")
        null
    }
}

fun main() {
    val result = mayThrowAnException(true)
    if (result == null) { // Always true
        println("Ignored code")
    }
}
```



#### 3. 컬렉션과 배열



#### 3.1 널 가능성과 컬렉션

Ex.1)null이 될 수 있는 값으로 이뤄진 컬렉션

```kotlin
fun readNumbers(reader: BufferedReader): List<Int?> {
    val result = ArrayList<Int?>()
    for (line in reader.lineSequence()) {
        try {
            val number = line.toInt()
            result.add(number)
        }
        catch(e: NumberFormatException) {
            result.add(null)
        }
    }
    return result
}

fun addValidNumbers(numbers: List<Int?>) {
    var sumOfValidNumbers = 0
    var invalidNumbers = 0
    for (number in numbers) {
        if (number != null) {
            sumOfValidNumbers += number
        } else {
            invalidNumbers++
        }
    }
    println("Sum of valid numbers: $sumOfValidNumbers")
    println("Invalid numbers: $invalidNumbers")
}

fun main(args: Array<String>) {
    val reader = BufferedReader(StringReader("1\nabc\n42"))
    val numbers = readNumbers(reader)
    addValidNumbers(numbers)		//Sum of valid numbers: 43
								  //Invalid numbers: 1
    
}
```

* List<Int?> 과 List<Int>?과 List<Int?>? 잘 구분해서 쓸 것!



#### 3.2 읽기 전용과 변경 가능한 컬렉션

* Kotlin 컬렉션은 자바와 다르게 읽기 전용 컬렉션(Collection<T>)과 변경가능 컬렉션(MutableCollection<T>)이 분리되어있다.

  Ex) Collection(size, iterator(), contains())    /    MutableCollection(add(), remove(), clear())

  ```kotlin
  fun <T> copyElements(source: Collection<T>,
                       target: MutableCollection<T>) {
      for (item in source) {
          target.add(item)
      }
  }
  
  fun main(args: Array<String>) {
      val source: Collection<Int> = arrayListOf(3, 5, 7)
      val target: MutableCollection<Int> = arrayListOf(1)
      copyElements(source, target)
      println(target)		//[1, 3, 5, 7]
  }
  ```

  

  Ex) Thread-safe하지않은 읽기전용 Collection

  ```kotlin
  //MutableCollection이 Collection 참조
  val collection: Collection<Int> = arrayListOf(1, 2, 3)
  val mutableCollection: MutableCollection<Int> = collection
  error: type mismatch: inferred type is Collection<Int> but MutableCollection<Int> was expected
  
  
  // Collection이 MutableCollection 참조
  val mutableCollection: MutableCollection<Int> = arrayListOf(1, 2, 3)
  val collection: Collection<Int> = mutableCollection //가능 
  >>> collection
  [1, 2, 3]
  >>> mutableCollection
  [1, 2, 3]
  >>> mutableCollection.add(5)
  >>> mutableCollection
  [1, 2, 3, 5]
  >>> collection
  [1, 2, 3, 5]    // mutalbeCollection에 5를 추가했지만 collection에도 영향을 미친다
  			   // 같은 객체를 다른 타입의 참조들이 가리키고 있다.
  			   
  ```

  * 따라서, ConcurrentModificationException등의 오류가 발생할 수 있으므로, 다중 스레드 환경에서 데이터를 다루는 경우 그 데이터를 적절히 동기화 하거나 동시 접근을 허용하는 데이터 구조를 활용해야한다.



#### 3.3 코틀린 컬렉션과 자바

* 코틀린에서 읽기 전용인 Collection으로 선언된 객체라도 자바코드에서는, 이를 구분하지 않으므로 수정가능
* 코틀린 컴파일러가 컬렉션이 어떤 일을 하는지 정확한 분석이 어려움
* 어차피 ImmutableList가 나와도 자바로 가면 변경이 가능하니 차라리 변경 가능한 컬렉션을 넘기고 명시를 정확히 해두는 것이 좋다.

```java
//Java
public class CollectionUtils {
    public static List<String> uppercaseAll(List<String> items) {
        for (int i = 0; i < items.size(); i++) {
            items.set(i, items.get(i).toUpperCase());
        }
        
        return items;
    }
}
```

```kotlin
//Kotlin
fun printInUppercase(list: List<String>) {
    println(CollectionUtils.uppercaseAll(list))
    println(list.first())
}

fun main(args: Array<String>) {
    val list = listOf("a", "b", "c")
    printInUppercase(list)			//[A, B, C]
    							   //A
}
```



#### 3.4 자바의 콜렉션 타입을 플랫폼 타입으로 다루기

* 자바의 콜렉션 타입은 읽기전용이나 변경 가능으로 다룰 수 있음
* 시그니처에서 콜렉션 타입을 사용한 자바 메서드를 오버라이드할 경우 코틀린 타입 선택 필요
  - 선택 사항
    - 컬렉션이 널이 될 수 있는가?
    - 컬렉션의 원소가 널이 될 수 있는가?
    - 오버라이드하는 메서드가 컬렉션을 변경할 수 있는가?
  - 자바 인터페이스나 클래스가 어떤 맥락에서 사용되는지 정확히 알아야한다.



#### 3.5 객체의 배열과 원시타입의 배열

* 코틀린에서 배열을 만드는 방법

  * arrayOf : 함수에 원소를 넘김

    ```kotlin
    fun main(args: Array<String>) {
        val strings = listOf("a", "b", "c")
        println("%s/%s/%s".format(*strings.toTypedArray()))		// a/b/c
    }
    ```

  * arrayOfNuls : 원소타입이 널이 될 수 있는 타입인 경우에만 호출할 수 있고, 인자에 배열의 크기를 넘김

    ```kotlin
    val nullStorableArray = arrayOfNulls<String>(3)
    ```

  * Array() : 배열의 크기와 람다를 인자로 받아, 람다를 호출하여 각 배열 원소를 초기화

    ```kotlin
    fun main(args: Array<String>) {
        val squares = IntArray(5) { i -> (i+1) * (i+1) }
        println(squares.joinToString())		//1, 4, 9, 16, 25
    }
    ```

    



