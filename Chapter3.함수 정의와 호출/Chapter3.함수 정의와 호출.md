### Chapter3. 함수 정의와 호출

**코틀린 컬렉션**

```kotlin
val set = hashSetOf(1, 7, 53)
val list = listOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")

fun main(args: Array<String>) {
    println(set.javaClass)	//class java.util.HashSet
    println(list.javaClass)  //class java.util.ArrayList
    println(map.javaClass)  //class java.util.HashMap
}
```

* 자바 컬렉션을 활용한 **코틀린 컬렉션**

  

* 자바보다 더 많은 기능을 쓸 수 있음

  ```kotlin
  val strings = listOf("first", "second", "fourteenth")
  println(strings.last())	//fourteenth
  val numbers = setOf(1, 14, 2)
  println(numbers.max())	//14
  ```

  - `last()`, `max()` 는 코틀린에서 편의를 위해 추가한 **확장함수**

**직관적인 함수 호출**

* toString()함수 응용한 함수

  ```kotlin
  fun <T> joinToString(
          collection: Collection<T>,
          separator: String,
          prefix: String,
          postfix: String
  ): String {
  
      val result = StringBuilder(prefix)
  
      for ((index, element) in collection.withIndex()) {
          if (index > 0) result.append(separator)
          result.append(element)
      }
  
      result.append(postfix)
      return result.toString()
  }
  
  fun main(args: Array<String>) {
      val list = listOf(1, 2, 3)
      val alphabets = listOf("a", "b", "c")
      println(joinToString(list, separator = "; ", prefix = "(", postfix = ")"))
      println(joinToString(alphabets, separator = "; ", prefix = "(", postfix = ")"))
      //(1; 2; 3)
      //(a; b; c)
  }
  ```

  * 인자 이름 & 기본 파라미터 값

    - 인자의 이름을 명시 가능

      **Java**

      ```java
      joinToString(collection, /* separator */ "", /* prefix */ "", /*postfix */ ".");
      ```

      **Kotlin**

      ```kotlin
      joinToString(collection, prefix = "(", separator = ";", postfix = ")")
      ```

      > **차이점?** 파라미터 이름이 같다면 순서를 바꿔도 가능!
      >
      > **주의!** 자바에서 작성한 코드를 코틀린에서 호출한 경우에는 인자 이름 명시 불가능!  
      >
      > ​		함수 파라미터에 정보를 넣는 것은 자바 8이후, 코틀린은 JDK 6과 호환!

      

    - 기본 파라미터 값 초기화

      ```kotlin
      fun <T> joinToString(
          	collection : Collection<T>,
              separator: String = ", ",
              prefix: String = "",
              postfix: String = ""
      ): String {
         ...
      }
      
      fun main(args: Array<String>) {
          val list = listOf(1, 2, 3)
          println(joinToString(list))	
          //1, 2, 3
          println(joinToString(list, separator = ";"))	
          //1;2;3
          println(joinToString(list, separator = "; ", prefix = "("))	
          //(1; 2; 3
          println(joinToString(list, separator = "; ", prefix = "(", postfix = ")"))
          //(1; 2; 3;)
      }
      ```

      - 값을 지정하지 않은 모든 인자는 디폴트 값을 적용

    > **주의!**   자바에는 디폴트 파라미터 값이라는 개념이 없기때문에 코틀린 함수를 자바에서 호출하는 경우 
    >
    > ​          @JvmOverloads 애노테이션을 추가하면 컴파일러가 자동으로 오버로딩 함수 생성

**최상위 함수와 프로퍼티**

* 최상위 함수

  * Java

    ```java
    package strings;
    
    public class JoinKt{
    	public static String joinToString(...){...}
    }
    ```

  * Kotlin

    ```kotlin
    package strings
    
    fun joinToString(...){...}
    ```

    - 자바에서 함수 호출

      ```java
      import strings.JointKt; 
      ... 
      
      JoinKt.joinToStirng(list,",","","");
      ```

    - 파일에 대응하는 클래스 이름 변경

      ```kotlin
      //코틀린
      @file:JvmName("StringFunctions")	//패키지명 선언보다 이전에 위치
      package strings
      ```

      ```java
      //자바에서 호출
      import strings.StringFunctions;
      StringFunctions.joinToString(list, ",", "", "");
      ```

> ~~UtilClass를 대신해서 사용할 수 있다는 장점이 있다고 했는데, Util이 더 편하지않나...?~~

* 최상위 프로퍼티

  ```kotlin
  var cnt;		//최상위 프로퍼티
  
  fun performOperation(){	//최상위 함수
  	cnt++
  }
  ...
  ```

  - 프로퍼티를 클래스보다 더 이전에 위치
  - 최상위 프로퍼티도 접근자 메소드를 통해 접근가능
  - `const` 변경자를 추가하면 `public static final` 필드로 만들 수 있다.



**확장함수와 확장 프로퍼티**

* 클래스의 멤버 메서드처럼 호출되지만 클래스 밖에 호출되는 함수

* 확장함수를 만들기 위해서는 **수신객체타입**(`Receiver Type`)과 **수신객체**(`Receiver Object`)가 필요

  ```kotlin
  fun String.lastChar() : Char = this.get(this.length - 1)
  println("Kotlin".lastChar())
  //n
  ```

  - 수신객체타입(`Receiver Type`): 확장이 정의될 클래스 ex) String

  - 수신객체(`Receiver Object`): 위 클래스의 인스턴스 객체 ex)"Kotlin", this

  - 확장함수 안에서는 `private`나 `protected` 접근 불가능, 해당 클래스의 `public`함수에만 접근 가능

    

* 확장함수의 Import

  - 확장함수를 사용하기 위해서는 import를 통해서 사용

    ```kotlin
    import strings.lastChar		
    //import strings.*
    val c = "Kotlin".lastChar()
    ```

    **주의!** : 한 클래스 내의 같은 이름의 확장함수가 둘 이상일 경우 이름이 충돌할 수 있으니 `as`를 사용

    ```kotlin
    import strings.lastChar as last
    val c = "Kotlin".last()
    ```

* joinToString함수를 이용한 확장함수

  ```kotlin
  fun <T> Collection<T>.joinToString(
          separator: String = ", ",
          prefix: String = "",
          postfix: String = ""
  ): String {
      val result = StringBuilder(prefix)
  
      for ((index, element) in this.withIndex()) {	//this는 수신객체
          if (index > 0) result.append(separator)
          result.append(element)
      }
  
      result.append(postfix)
      return result.toString()
  }
  
  //문자열의 컬렉션만 받음
  fun Collection<String>.join(
          separator: String = ", ",
          prefix: String = "",
          postfix: String = ""
  ) = joinToString(separator, prefix, postfix)
  
  fun main(args: Array<String>) {
      println(listOf("one", "two", "eight").join(" "))	//one two eight
  }
  ```

  * 내부적으로 확장함수는 **수신객체**를 **첫 번째 인자**로 받음
  * 자바에서 호출시 **수신객체**를 첫 번째 인자로 넘겨줘야함

**확장함수의 오버라이드**

* 멤버함수 오버라이드

  ```kotlin
  open class View {
      open fun click() = println("View clicked")	//멤버함수
  }
  
  class Button: View() {
      override fun click() = println("Button clicked")
  }
  
  fun main(args: Array<String>) {
      val view: View = Button()
      view.click()	//Button Clicked
  }
  ```

* 확장 함수가 정적 메소드와 같은 특징을 가지므로 확장함수를 하위 클래스에서 오버라이드 할 수 없다.

  ```kotlin
  open class View {
      open fun click() = println("View clicked")	//멤버함수
  }
  
  class Button: View() {
      override fun click() = println("Button clicked")
  }
  
  fun View.showOff() = println("I'm a view!")		//확장함수
  fun Button.showOff() = println("I'm a button!")	//확장함수
  
  fun main(args: Array<String>) {
      val view: View = Button()
      view.showOff()	//I'm a view!
  }
  ```

  * 확장 함수는 클래스의 밖에 선언되며 확장함수 호출 시, 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 확장함수가 올 지 결정되는 것이지, 변수의 객체에 저장된 동적인 타입에 의해 확장 함수가 결정되지 않는다.

    <그림 3.2 참고>

  > TODO - 알고가자!
  >
  > 1. `open class` : 코틀린에서의 모든 클래스는 암시적으로 상위 클래스로 `Any`클래스를 가지는데, 
  >
  > ​                         명시적으로 상위클래스를 지정하기 위해서는 `open`을 명시
  >
  > 2. 멤버메서드? 정적메서드? 확장메서드?
  >
  > 3. 특정 클래스에 추가된 확장함수가 멤버함수와 이름이 같다면 우선순위는 **멤버함수 > 확장함수**

**확장 프로퍼티**

```kotlin
val String.lastChar : Char
	get() = get(length - 1)		//getter만

var StringBuilder.lastChar : Char
	get() = get(length - 1)
	set(value : Char){
        this.setChartAt(length - 1, value)
    }
```

* 일반 프로퍼티와 같은데 대신 **수신객체클래스**가 추가 되었을 뿐
* 확장 프로퍼티는 `backing field`를 가지고 있지않으니 최소한 getter()는 구현해야 함.



**컬렉션 처리**

* 가변인자함수

  ```kotlin
  val numbers = listOf(1,2,3,4,5)
  val strings = listOf("1", "2", "3")
  
  fun listOf<T>(vararg var: T) : List<T>{...}
  ```

  - `vararg`를 변경자로 선언타입 앞에 붙임

    

    ```kotlin
    fun printNumbers(vararg numbers: Int) {
        for (number in numbers) {
            println(number)
        }
    }
    
    val numbers = intArrayOf(1, 2, 3)
    printNumbers(*numbers)
    printNumbers(10, 20, *numbers, 30, 40)
    ```

    + 자바의 `스프레드 연산자`: `...` / 코틀린의 `스프레드 연산자` `*`

    + 배열(Array)을 함수에 전달할 수 있다.

      

* 중위 호출

  ```kotlin
  1 to "one"		//중위함수 호출
  1.to("one")		//확장함수 호출
  ```

  - 중위함수 to

    ```kotlin
    infix fun Any.to(other : Any) = Pair(this, other)    // to는 중위함수이자 확장함수
    ```

* 구조 분해 선언(7장에서 자세히 다룸)

  - data class 형태로 나오는 여러 값을 동시에 반환할 수 있다.
  - 자동으로 componentN함수가 생성됨.

  ```kotlin
  for ((index, element) in this.withIndex()) {	//this는 수신객체 Collection<Int> / (1, 2, 3)
       println("$index: $element")	//0: 1
      							  //1: 2
      							  //2: 3
  }
  ```

  ```kotlin
  data class Size(val width : Int, val height : Int)
  
  fun main(){
      val size = Size(10, 20)
      val (w, h) = size
      val a = size.component1()
      val b = size.component2()
      
      println(a) // 10
      println(b) // 20
  }
  ```



**문자열과 정규식**

* 문자열 나누기 / split()

  ```kotlin
  //String.kt
  public inline fun CharSequence.split(regex: Regex, limit: Int = 0): List<String> = regex.split(this, limit)
  public fun CharSequence.split(vararg delimiters: String, ignoreCase: Boolean = false, limit: Int = 0): List<String> {...}
  ```

  ```kotlin
  println("12.345-6.A".split("\\.|-".toRegex()))		//정규식을 파라미터로 받는 함수
  //[12, 345, 6, A]
  println("12.345-6.A".split(".", "-"))			    //String을 파라미터로 받는 함수
  //[12, 345, 6, A]
  ```

  - 마침표 `.` 를 문자로 읽기위해 `\\.`을 사용하여 escape 시킴

  - 코틀린에서는 split()의 확장함수를 제공함으로써 Java의 와일크카드 문자(?)로 인한 혼동을 피함.
  - 정규식 표현 https://regexr.com/,  https://ihateregex.io/playground에서 찾으면 쉽다.

* 정규식과 3중 따옴표

  ```kotlin
  fun parsePath(path: String) {
      val regex = """(.+)/(.+)\.(.+)""".toRegex()
      //group#1: /Users/yole/kotlin-book
      //group#2: chapter
      //group#3: adoc
      val matchResult = regex.matchEntire(path)
      if (matchResult != null) {
          val (directory, filename, extension) = matchResult.destructured
          // val directory = matchResult.destructured.component1()
          // val filename = matchResult.destructured.component2()
          // val extension = matchResult.destructured.component3()
          println("Dir: $directory, name: $filename, ext: $extension")
      }
  }
  
  fun main(args: Array<String>) {
      parsePath("/Users/yole/kotlin-book/chapter.adoc")
  }
  ```

  * 3중 따옴표를 사용하면, `\`, `\n`등을 포함한 어떤 문자도 따로 escape  시킬 필요가 없다.

  > **주의!** : 역슬래쉬(`\`)나 줄바꿈(`\n`)등을 3중 따옴표 안에 넣으면 문자로 인식해서 그냥 출력된다.

  

**중첩함수(Local method)**

* 함수에서 추출한 함수를 원 함수 내부에 중첩시키는 것을 말함.

* 코드중복을 보여주는 예제  //아직 중첩함수 X

  ```kotlin
  class User(val id: Int, val name: String, val address: String)
  
  fun saveUser(user: User) {
      if (user.name.isEmpty()) {					//-----------검증 중복
          throw IllegalArgumentException(
              "Can't save user ${user.id}: empty Name")
      }
  
      if (user.address.isEmpty()) {				//------------검증 중복
          throw IllegalArgumentException(
              "Can't save user ${user.id}: empty Address")
      }
  
      // Save user to the database
  }
  
  fun main(args: Array<String>) {
      saveUser(User(1, "", ""))
      //Can't save user ${user.id}: empty Name
  }	
  ```

  - 중첩함수를 이용한 중복 줄이기

    ```kotlin
    class User(val id: Int, val name: String, val address: String)
    
    fun saveUser(user: User) {
        fun validate(user: User,				//중첩함수 validate(...)선언
                     value: String,
                     fieldName: String) {
            if (value.isEmpty()) {
                throw IllegalArgumentException(
                    "Can't save user ${user.id}: empty $fieldName")
            }
        }
    
        validate(user, user.name, "Name")
        validate(user, user.address, "Address")
    
        // Save user to the database
    }
    
    fun main(args: Array<String>) {
        saveUser(User(1, "", ""))
    }
    ```

    + 중첩함수는 자신이 속한 바깥함수의 모든 파라미터와 변수를 참조 가능

    + 따라서 validate의 파라미터인 user까지 제거

      ```kotlin
      class User(val id: Int, val name: String, val address: String)
      
      fun saveUser(user: User) {
          fun validate(value: String, fieldName: String) {	//불필요한 user파라미터 제거
              if (value.isEmpty()) {
                  throw IllegalArgumentException(
                      "Can't save user ${user.id}: " +
                          "empty $fieldName")
              }
          }
      
          validate(user.name, "Name")							//user.name만 전달
          validate(user.address, "Address")					//user.address만 전달
      
          // Save user to the database
      }
      
      fun main(args: Array<String>) {
          saveUser(User(1, "", ""))
      }
      ```

      + 확장함수를 이용해서 검증로직 개선

        ```kotlin
        class User(val id: Int, val name: String, val address: String)
        
        fun User.validateBeforeSave() {			//User클래스의 확장함수
            fun validate(value: String, fieldName: String) {	//중첩함수
                if (value.isEmpty()) {
                    throw IllegalArgumentException(
                       "Can't save user $id: empty $fieldName")	//{$user.id}를 쓸 필요 X, $id O 
                }
            }
        
            validate(name, "Name")
            validate(address, "Address")
        }
        
        fun saveUser(user: User) {
            user.validateBeforeSave()
        
            // Save user to the database
        }
        
        fun main(args: Array<String>) {
            saveUser(User(1, "2", ""))
        }
        ```

        > TODO - 생각해보자!
        >
        > 좋은 구조인가?? 
        >
        > > **과유불급**이라고 중첩된 함수의 깊이가 깊어지면 깊어질수록 코드 읽기가 어려우니
        > >
        > > 일반적으로는 **한 단계**만 함수를 중첩시키는 것을 권장한다.

