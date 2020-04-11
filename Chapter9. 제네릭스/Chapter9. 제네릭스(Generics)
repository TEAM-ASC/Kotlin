### Chapter9 제네릭스(Generics)



#### 1. 제네릭 타입 파라미터

* 제네릭스를 사용하기 위해서는 **타입 파라미터(type parameter)**를 받는 타입을 정의하고, 인스턴스를 만들어 타입파라미터를 구체적인 **타입 인자(type argument)**로 치환해야한다. (Ex. Map<K, V>  = >  Map<String, Person>)

> **짚고넘어가자!**
>
> *매개변수와 전달인자의 차이?*
>
> ```Kotlin
> fun sum(a : Int, b : Int) : Int = a + b // (a : Int, b : Int) 매개'변수'
> 
> sum(10, 20) // (10, 20) '전달'인자
> ```
>
> * 매개변수 : 전달된 인자를 받아들이는 **변수**
> * 전달인자: 함수에 전달되는 **값**

* Kotlin은 Java와 달리 타입 추론이 가능한 경우 타입 매개변수를 명시하지 않아도 된다.

  ```Kotlin
  val authors = listOf("Dmitry", "Svetlanan")
  val readers : MutableList<String> = mutableListOf()
  ```

> **알고가자!**
>
> *'Java와 Kotlin의 제네릭 차이?'*
>
> **Java는 뒤늦게 1.5부터 제네릭을 지원**했기 때문에 하위 버전과의 호환성을 위해 **타입인자가 없는 제네릭(raw 타입)을 허용**한다. Kotlin도 Java와 마찬가지로 제네릭을 지원하지만, **Kotlin은 처음부터 제네릭을 도입**했기 때문에 raw타입을 지원하지않고, **반드시 제네릭 타입의 타입 인자를 정의(추론 or 명시)해야 한다.**



#### 1.1 제네릭 함수와 프로퍼티

* 제네릭 함수 기본선언

  ```Kotlin
  fun <T> List<T>.slice(indices: IntRange) : List<T>
  ```

  * 함수의 타입파라미터 **T**는 **수신객체**와 **반환 타입**에 쓰인다.

* Kotlin은 타입파라미터를 명시적으로 넣어도 되지만 넣지 않더라도 컴파일러가 추론한다.

  ```Kotlin
  val letters = ('a'..'z').toList()	//List<Char>
  println(letters.slice<Char>(0..2))	//[a b, c]
  println(letters.slice(10..13))		//[k, l, m, n]
  ```

* 제네릭 함수를 정의할 때와 마찬가지 방법으로 제네릭 확장 프로퍼티를 선언할 수 있다.

  ```Kotlin
  val <T> List<T>.second: T
      get() = this[size - 2]
  
  fun main(args: Array<String>) {
      println(listOf(1, 2, 3, 4).second)	//3
      									//List<Int>
  }
  ```

> **주의!** 
>
> *''확장 프로퍼티만 제네릭하게 만들 수 있다''*
>
> 일반 프로퍼티는 타입 파라미터를 가질 수 없다. 클래스 프로퍼티에 여러 타입의 값을 저장할 수는 없으므로 제네릭한 일반 프로퍼티는 말이 되지 않는다. 일반 프로퍼티를 제네릭하게 정의하면 컴파일 오류가 난다.



#### 1.2 제네릭 클래스 선언

* Java와 마찬가지로 Kotlin도 타입 파라미터를 넣은 꺾쇠 기호(Angle Bracket)를 클래스 이름 뒤에 붙이면 클래스를 제네릭하게 만들 수 있다.

* 기반 타입의 제네릭 파라미터를 넘길 수도 있고, 하위 클래스도 제네릭 클래스라면 타입 파라미터로 받은 타입을 넘길 수도 있다.

  ```Kotlin
  interface List<T>{
      operator fun get(index: Int) : T
  }
  
  class StringList : List<String>{
      override fun get(index : Int) String = ... 
      //상위클래스 List<String>의 함수를 오버라이드하가위해, 하위 클래스 StringList는 상위 클래스 			 List<String>의 타입인자T인 String으로 치환.
  }
  class ArrayList<T> : List<T> {
      override fun get(index : Int) : T = ...
      //ArrayList<T>의 T는  자신만의 타입파라미터 T를 가지고 List<T>의 T와는 다른 파라미터
  }
  class ArrayList<S> : List<Int>{
      override fun get(index : Int) : Int = ...
  }
  ```

* 클래스가 자기 자신을 타입 인자로 참조할 수도 있다.

  ```Kotlin
  interface Comparable<T>{
      fun compareTo(other : T) : Int
  }
  class String : Comparable<String>{
      override fun compareTo(other: String) : Int = ...
  }
  ```



#### 1.3 타입 파라미터 제약

* **타입 파라미터 제약(type parameter constraint)**은 클래스나 함수에 사용할 수 있는 타입 인자의 상한타입(Upper bound)을 설정하는 기능이다.

* Java에서는 **extends**나 **super**를 사용하여 사용한 타입을 제한할 수 있고, Kotlin에서는 **' : '(콜론)**을 사용한다.

* 어떤 타입이 제네릭 타입의 타입 파라미터에 대한 상한으로 지정되었다면, 해당 제네릭 타입의 파라미터는 반드시 그 상한 타입이거나 그 상한타입의 하위타입이어야 한다.

  ```Java
  //Java
  <T extends Number> T oneHalf(T value)
  ```

  ```Kotlin
  //Kotlin
  fun <T : Number> oneHalf(value: T): Double {
      return value.toDouble() / 2.0
  }
  
  fun main(args: Array<String>) {
      println(oneHalf(3))		//1,5
  }
  ```

  ```kotlin
  fun <T: Comparable<T>> max(first: T, second: T): T {
      return if (first > second) first else second
  }
  
  fun main(args: Array<String>) {
      println(max("kotlin", "java"))
  }
  ```

* 두 개 이상의 제약을 걸어야 하는 경우 **where** 연산자를 사용한다.

  Ex)

  ```Kotlin
  import java.time.Period
  
  fun <T> ensureTrailingPeriod(seq: T)
          where T : CharSequence, T : Appendable {
      if (!seq.endsWith('.')) {
          seq.append('.')
      }
  }
  
  fun main(args: Array<String>) {
      val helloWorld = StringBuilder("Hello World")
      ensureTrailingPeriod(helloWorld)
      println(helloWorld)	//Hello World. 
  }
  ```



#### 1.4 타입 파라미터를 널이 될 수 없는 타입으로 한정

* 제네릭 클래스나 함수를 정의하고 그 타입을 인스턴스화할 때는 널이 될 수 있는 타입을 포함하는 어떤 타입으로 타입 인자를 지정해도 타입 파라미터를 치환할 수 있다.

  ```kotlin
  class Processor <T> {
      fun process(value: T) {
          println(value?.hashCode()) 
      } 
  } 
  fun main(args: Array<String>) {
      val processor = Processor<String?>() 
      processor.process(null) //null
  }
  ```

  * 명시를 <T : Any>처럼 따로 해두지 않으면, 제네릭의 상한 타입(upper bound)는 Any?이므로 널이 될 수 있다.



#### 2. 실행 시 제네릭스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터

* JVM의 제네릭스는 보통 **타입 소거(type erasure)**를 사용해 구현되어, 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지않다.
* Kotlin **타입 소거**의 실용성과 inline을 이용한 제약 우회로 인해 얻을 수 있는 이점을 알아보자.



#### 2.1 실행 시점의 제네릭: 타입 검사와 캐스트

* 제네릭 클래스 인스턴스가 생성될 때 인스턴스 생성에 쓰인 타입 인자에 대한 정보를 유지하지 않는다.

* 객체를 선언해도 객체의 타입 정보까지는 저장하지 않기 때문에 객체에 대한 정보만 가진다.

  ```Kotlin
  val list1 : List<String> = listOf("a", "b")		//List를 가진 객체로만 인식
  val list2 : List<Int< = listOf(1, 2, 3)
  ```

* 원소의 타입까지는 저장하지 않으면서 메모리적인 이득을 얻지만, 그 떄문에 실행 시점에 타입 인자를 검사를 할 수 없어 다음과 같은 컴파일 오류가 발생될 수 있다.

  ```Kotlin
  if(value is List<String>){...}
  //Error : Cannot check for instance of erased type
  ```

  > **주의!** 
  >
  > http://play.kotlinlang.org/ 같은 REPL에서 실행하면 컴파일 에러 없이 정상적으로 돌아가니 조심...

* 따라서, 인자를 알 수 없는 제네릭 타입을 표현할 때,  **' * '(스타 프로젝션)**을 사용하면 된다.

  ```kotlin
  fun printSum(c: Collection<*>) {
      val intList = c as? List<Int>	//Warning : Unchecked cast: List<*> to ListInt>
              ?: throw IllegalArgumentException("List is expected")
      println(intList.sum())	//없는 함수?
  }
  
  fun main(args: Array<String>) {
      printSum(listOf(1, 2, 3))
  }
  ```

  * 컴파일 시점에 타입 정보가 진 경우에는 is 검사를 수행하게 허용

    ```Kotlin
    fun printSum(c : Collection<Int>){
        if(c is List<Int>){
            println(c.sum())
        }
    }
    ```



#### 2.2 실체화한 타입 파라미터를 사용한 함수 선언

* inline 함수의 타입 파라미터는 실체화되므로 실행 시점에 inline함수의 타입 인자를 알 수 있다.

  ```kotlin
  fun <T> isA(value: Any) = value is T
  //Error : Cannot check for instance of erased type : T
  
  inline fun <reified T> isA(value : Any) = value is T
  
  fun main(args : Array<String>){
      println(isA<String>("abc")) 		//true
      println(isA<String>(123))			//false
  }
  ```

  * 함수를 inline함수로 만들고 타입 파라미터를 reified로 지정하면 실행시점에 타입정보를 알 수 있다.

  ```kotlin
  public inline fun <reified R> Iterable<*>.filterIsInstance(): List<@kotlin.internal.NoInfer R> {
      return filterIsInstanceTo(ArrayList<R>())
  }
  public inline fun <reified R, C : MutableCollection<in R>> Iterable<*>.filterIsInstanceTo(destination: C): C {
      for (element in this) if (element is R) destination.add(element)
      return destination
  }
  
  val items = listOf("one", 2, "three")
  println(items.filterIsInstance<String>())	//[one, three]
  ```

> **인라인 함수에서만 실체화한 타입 인자를 쓸 수 있는 이유?**
>
> *''왜 일반함수에서는 element is T를 쓸 수 없고 inline함수에서만 쓸 수 있는걸까?''*
>
> inline함수의 본문이 구현된 바이트코드가 그 함수가 호출되는 모든 지점에 삽입이 되고 결과적으로 
>
> ```kotlin
> for(element in this){
> 	if(element is String){
>         destination.add(element)
>     }
> }
> ```
>
> 형태로 입력되어 실행시점에 벌어지는 타입 소거의 영향을 받지 않아서 가능하다.
>
> 또한, Java에서는Kotlin인라인 함수를 다른 보통 함수처럼 호출하기 때문에 reified 타입 파라미터를 사용하는 inline 함수를 호출할 수 없다. 

* 추가적으로, 8장에서 언급했듯이 람다를 인자로 받는 inline 함수는 해당 인자를 함께 인라이닝함으로써 얻는 이익이  큼으로 inline 함수로 만들라고했지만, inline함수의 내용이 길다면 실체화가 필요없는 부분은 따로 함수로 뽑아내는 편이 좋다.



#### 2.3 실체화한 타입 파라미터로 클래스 참조 대신

* Java를 쓰다보면 java.lang.Class를 참조로 얻는 방법이 있는데, Kotlin에서는 ::class.java구문이 그러하다.
* service를 start할떄, activity를 start할 때, intent에 class명을 직접 입력해서 사용하는데, reified를 이용하면 간단하게 표현 가능하다.

```kotlin
inline fun <reified T : Activity> Context.startActivity(){
    val intent = Intent(this, T::class.java)
    startActivity(intent)
}
startActivity<DetailActivity>()
```



#### 2.4 실체화한 타입 파라미터의 제약

- **실체화한 타입 파라미터 사용가능 case**

- - 타입 검사와 캐스팅 (is, !is, as, as?)
  - 추후 언급되는 코틀린 리플렉션API(::class)
  - 코틀린타입에 대응하는 java.lang.Class 얻기 (::class.java)
  - 다른 함수를 호출할 대 타입 인자로 사용

- **실체화한 타입 파라미터 사용 불가 case**

- - 타입 파라미터 클래스의 인스턴스 생성
  - 타입 파라미터의 companion object method 호출
  - 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
  - 클래스, property, inline 함수가 아닌 함수의 타입 파라미터를 reified로 지정하기
    * **실체화한 타입 파라미터를 inline 함수에만 사용**할 수 있으므로 실체화한 타입 파라미터를 사용하는 함수는 자신에게 전달되는 **모든 람다와 함께 인라이닝** 된다. **성능 혹은 람다를 인라이닝 할 수 없는 문제**로 인라이닝을 할 수 없는 경우 **noinline변경자를 함수 타입 파라미터에 붙여서 인라이닝을 금지**할 수 있다.



#### 3. 변성: 제네릭과 하위 타입

* **변성(variance)**의 개념: List<String>과 List<Any>의 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념
* 타입 안정성을 보장하는 API를 만들 수 있다.



#### 3.1 변성이 있는 이유: 인자를 함수에 넘기기

* String클래스는 Any를 확장하므로 List<Any> 타입의 파라미터를 받는 함수에 List<String> 을 넘겨도 안전성이 보장된다. 하지만, MutableList<Any> 등 Any와 String이 List 인터페이스의 타입 인자로 들어가게 되면 안전하다고 할 수 없다.

* Ex) 리스트에 대한 추가변경이 일어나서 컴파일 에러가 뜨는 경우

  ```Kotlin
  //안정성 보장
  fun printContents(list: List<Any>) {
      println(list.joinToString())	
  }
  
  //안정성 보장 X
  fun addAnswer(list : MutableList<Any>){
      list.add(42)
  }
  
  fun main(args: Array<String>) {
      printContents(listOf("abc", "bac"))	//abc, bac
      
      val strings = mutableListOf("abc", "bac")
      addAnswer(strings)					//Type mismatch 먼저 뜨는데..
      //addAnswer(strings.toMutableList())	//abc
      println(strings.maxBy{ it.length })	//ClassCastException : Integer cannot be cast to String
  }
  
  ```



#### 3.2 클래스, 타입, 하위 타입

* **제네릭 클래스가 아닌 경우**: 클래스 이름을 바로 타입(String, Person, Int etc..)으로 쓸 수 있다. 그리고 **Kotlin의 모든 클래스는 적어도 Non-null, Nullable 이 두 가지 타입**을 가진 클래스이다.
* **제네릭 클래스의 경우**: 타입인자를 가지는 제네릭 클래스인 List<T>의 경우, 타입 인자를 치환한 List<Int>, List<String?>, List<List<String>> 등 무수히 많은 타입을 가지고 있다.
* 리스코프 치환법칙: 어떤 타입 A의 값이 필요한 모든 장소에 어떤 타입 B의 값을 넣어도 아무 문제가 없다면 타입B는 타입 A의 하위 타입이다.
* 예를 들면, Int는 Number의 하위 타입이지만 String의 하위 타입은 아니다. 또한, 모든 타입이 자신의 하위 타입이라고도 할 수 있다.

|   A   | Number  |   Int   | String  |  Int?   |
| :---: | :-----: | :-----: | :-----: | :-----: |
|   ↑   |    ↑    |    ↑    | ↑**x**  |    ↑    |
| **B** | **Int** | **Int** | **Int** | **Int** |

```kotlin
fun test(i : Int){
    val n : Number = i
    
    fun f(s: String){/* ... */}
    f(i)	//Type mismatch
}
```

* 따라서, List<String>은 List<Any>의 하위타입이지만 MutableList<String>은 MutableList<Any>의 하위 타입관계가 성립하지않는다. 이를 무공변이라고 한다.
* 만약, A가 B의 하위타입이면 List<A>는 List<B>의 하위 타입이다. 이런 클래스나 인터페이스를 **공변적(covariant)**이라고 한다.

> **함께 알아보자!** 
>
> *"가변성의 3가지 유형"*
>
>  모던 랭귀지들은 타입바운드 개념을 제공하며, 타입바운드는 **무공변성(invariant)**, **공변성(covariant)**, **반공변성(contravariant)** 3가지로 분류할 수 있다. 
>
> (* **타입바운드(Type bound)**란? 타입 매개변수와 타입 변수에 제약을 거는 행위를 말하며, 타입에 대해 안전하게 코딩을 하기위해 사용)
>
> * 무공변성(Invariant) : 상속 관계에 상관없이, 자기 타입만 허용하는 것을 말한다.
>
>   ```Kotlin
>   fun main(args: Array<String>) {
>   
>       val language: Some<Language> = object : Some<Language> {
>           override fun something() {
>               println("language")
>           }
>       }
>       val jvm: Some<JVM> = object : Some<JVM> {
>           override fun something() {
>               println("jvm")
>           }
>       }
>       val kotlin: Some<Kotlin> = object : Some<Kotlin> {
>           override fun something() {
>               println("kotlin")
>           }
>       }
>       test(language)  //error
>       test(jvm)       //ok
>       test(kotlin)    //error
>   }
>   interface Some<T> {fun something()}
>   fun test(value: Some<JVM>) {}
>   
>   open class Language
>   open class JVM : Language()
>   class Kotlin : JVM()
>   ```
>
> * 공변성(covariant): **out 키워드**를 사용하며, 타입생성자에게 리스코프 치환법칙을 허용한다는 의미(자신과 자식객체만 허용)
>
>   ```kotlin
>   fun test(value: Some<out JVM>) {}
>   
>   test(language)  //error
>   test(jvm)       //ok
>   test(kotlin)    //ok
>   ```
>
> * 반공변성(contravariant): **in 키워드**를 사용하며, 공변성의 반대개념으로 자신과 부모 객체만 허용하는 것을 말함.
>
>   ```kotlin
>   fun test(value: Some<in JVM>) {}
>   
>   test(language)  //ok
>   test(jvm)       //ok
>   test(kotlin)    //error
>   ```
>
>   출처: [https://medium.com/@lazysoul/%EA%B3%B5%EB%B3%80%EA%B3%BC-%EB%B6%88%EB%B3%80-297cadba191](https://medium.com/@lazysoul/공변과-불변-297cadba191)



#### 3.3 공변성: 하위 타입 관계를 유지

* 공변적으로 만들면 안전하지 못한 클래스도 있기 때문에 **모든 클래스를 공변적으로 만들 수는 없다**.

* **타입 파라미터를 공변적으로 지정(out T)**하면 클래스 내부에서 그 **파라미터를 사용하는 방법을 제한**한다.

* 클래스 멤버를 선언할 때, 타입 파라미터를 사용할 수 있는 지점은 모두 **인(in)**과 **아웃(out)** 위치로 나뉜다.

  ```Kotlin
  interface Transformer<T> {
      fun transform(t: T) :  T
                  //in위치  //out위치
  }
  ```

  * 함수 파라미터 타입은 in위치, 함수 반환 타입은 out 위치에 있다.
  * out 키워드가 붙으면 클래스 안에서 T를 사용하는 메소드가 아웃 위치에서만 T를 사용하게 허용하고 in위치에서는 T를 사용하지 못하게 막는다.
  * out위치: T타입의 값을 생산한다.
  * in위치 : T타입의 값을 소비한다.(반공변성)



#### 3.4 반공변성: 뒤집힌 하위 타입 관계

* 반공변성은 공변성을 뒤집은 상이라 할 수 있다. 따라서, **하위타입 관계가 공변 클래스의 경우와 반대**
* 타입 인지의 하위 타입 관계가 제네릭 타입에서 뒤집힌다.
* T타입의 값을 함수안에서 소비하기만 한다.



#### 3.5 사용 지점 변성: 타입이 언급되는 지점에서 변성 지정

* 클래스를 **선언하면서 변성을 지정**하는 것을 **선언지점 변성**이라고 한다.

* 자바의 와일드카드 타입 out 프로젝션은 <? extends T>, in 프로젝션은 <? super T>와 같다.

  Ex) 타입 파라미터가 두 개인 함수

  ```kotlin
  fun <T: R, R> copyData(source: MutableList<T>,
                         destination: MutableList<R>) {
      for (item in source) {
          destination.add(item)
      }
  }
  
  fun main(args: Array<String>) {
      val ints = mutableListOf(1, 2, 3)
      val anyItems = mutableListOf<Any>()
      copyData(ints, anyItems)
      println(anyItems)
  }
  ```

  Ex) out-프로젝션 타입 파라미터를 사용하는 데이터 복사 함수

  ```kotlin
  fun <T> copyData(source: MutableList<out T>,
                   destination: MutableList<T>) {
      for (item in source) {
          destination.add(item)
      }
  }
  ```

  * Int는 Any의 하위 타입이므로 out프로젝션을 사용하여 더 간단히 제한할 수 있다.

  Ex) in 프로젝션 타입 파라미터를 사용하는 데이터 복사 함수

  ```kotlin
  fun <T> copyData(source: MutableList<T>,
                   destination: MutableList<in T>) {
      for (item in source) {
          destination.add(item)
      }
  }
  ```

  * Any는 Int의 상위 타입이므로 in 프로젝션을 사용하여 표현할 수도 있다.

3.6 스타프로젝션: 타입 인자 대신 * 사용

* 제네릭 타입 인자 정보가 없을 경우 **스타 프로젝션**을 사용
