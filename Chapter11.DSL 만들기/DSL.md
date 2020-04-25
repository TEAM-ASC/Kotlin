## 11장

### Overviews

- 영역 특화 언어 만들기
- 수신 객체 지정 람다 사용
- `invoke` 관례 사용
- 기존 코틀린 DSL 예제

> 들어가기 전에

영역 특화 언어(DSL, Domain-Specific Language)는 특정 주제에 특화된 언어를 의미하며, 데이터 베이스에 접근하기 위한 SQL이 대표적인 DSL이다.

이 밖에도 HTML 생성, 테스트, 빌드 스크립트 작성, 안드로이드 UI 레이아웃 정의 등 여러 작업에서 사용할 수 있다.

코틀린에서는 고차함수와 람다식의 특징을 이용하여 읽기 좋고 간략한 코드를 만들 수 있다.



### 11.1 API에서 DSL로

라이브러리 개발자 뿐만 아니라 모든 개발자는 깔끔한 API 작성해야 할 책임이 있다.

##### 깔끔한 API란?

* 코드를 읽는 사람들이 어떤 일이 벌어질지 명확하게 이해할 수 있어야 한다.
  * e.g.) 이름 잘붙이기, 적절한 개념 사용 등..
* 불필요한 구문이나 번잡한 준비 코드가 적고 코드가 간결해야 한다.

##### 코틀린에서 깔끔한 API를 작성하도록 도와주는 기능

| 일반 구문                   | 간결한 구문                                                  | 사용한 언어 특성               |
| --------------------------- | ------------------------------------------------------------ | ------------------------------ |
| StringUtil.capitalizes(s)   | s.capitalize()                                               | 확장 함수                      |
| 1.to("one")                 | 1 to "one"                                                   | 중위 호출                      |
| set.add(2)                  | set += 2                                                     | 연산자 오버로딩                |
| map.get("key")              | map["key"]                                                   | get 메소드에 대한 관례         |
| file.use({ f -> f.read() }) | file.use { it.read() }                                       | 람다를 괄호 밖으로 빼내는 관례 |
| sb.append("yes")            | with (sb) {<br />    append("yes")<br />    append("no")<br />} | 수신 객체 지정 람다            |



#### 11.1.1 영역 특화 언어라는 개념

프로그래밍 언어는 컴퓨터로 풀 수 있는 모든 문제를 충분히 풀 수 있는 기능을 제공하는 **범용 프로그래밍 언어** 와 특정 과업 또는 영역에 초점을 맞추고 그 영역에 필요하지 않은 기능을 없앤 **영역 특화 언어** 로 구분할 수 있다.

* 범용 프로그래밍 언어
  * **명령적**인 특징을 가지고 있다.
    * ex) 어떤 연산을 완수하기 위해 필요한 각 단계를 순서대로 정확히 기술

* 영역 특화 언어

  * SQL과 정규식과 같이 제공하는 기능을 스스로 제한함으로써 오히려 더 효율적으로 목표를 달성할 수 있 도록 하는 특징을 가진 언어
  * **선언적**인 특징을 가지고 있다.
    * 원하는 결과를 기술하기만 하고 그 결과를 위한 세부 실행은 언어를 해석하는 엔진에 맡김
  * 특정 영역에 특화되어 자체 문법이 있기 때문에 범용 언어로 만든 애플리케이션과 조합하기가 어렵다.
    * ex) 내부 DB 사용 시 쿼리문 작성 <-- 컴파일 시점에 검증 불가 등..

  > 태형 Think!
  >
  > * 물론 최근에는 Room 등 DB 라이브러리를 이용하면 어노테이션을 통해 컴파일 시점에도 알 수 있다.

#### 11.1.2 내부 DSL

내부 DSL은 범용 언어로 작성된 프로그램의 일부며, 범용 언어와 동일한 문법을 사용한다. 그렇기 때문에 내부 DSL은 다른 언어가 아니라 DSL의 핵심 장점을 유지하면서 주 언어를 특별한 방법으로 사용하는 것이다.

> 반면, 외부 DSL은?
>
> 주 언어와는 독립적인 문법 구조를 가진다. 예로는 XML, MakeFile 등이 있다.

```sql
// SQL을 이용한 쿼리문 작성 (외부 DSL)
SELECT Country.name, COUNT(Customer.id) FROM Country
	JOIN Customer ON Country.id = Customer.country_id
GROUP BY Country.name
ORDER BY COUNT(Customer.id) DESC LIMIT 1
```

```kotlin
// 코틀린 익스포즈드를 사용해 쿼리문 작성 (내부 DSL)
(Country join Customer)
	.slice(Coutry.name, Count(Customer.id))
	.selectAll()
	.groupBy(Country.name)
	.orderBy(Count(Customer.id), isAsc = false)
	.limit(1)
```

* [코틀린 익스포즈드](https://github.com/JetBrains/Exposed) 이용 시 결과를 코틀린 객체로 반환해준다. (내부 DSL인 이유)

#### 11.1.3 DSL의 구조

DSL과 일반 API를 구분하는 방법

* DSL은 구조 또는 문법을 **독립적**으로 가진다.

* 위의 예제에서 여러 함수 호출을 조합해서 연산을 만드는 것 또한 내부 DSL 적인 특징이다.

DSL 구조의 장점

* 같은 문맥을 함수 호출 시마다 반복하지 않고도 재사용할 수 있다.

```kotlin
// gradle에서 디펜던시 설정 <-- 람다 중첩을 이용해 구조를 만듦
dependencies {
	compile("junit:junit:4.11")
	compile("com.google.inject:guice:4.1.0")
}

// 일반 명령-질의 API를 통해 디펜더시 설정
project.dependencies.add("compile", "junit:junit:4.11")
project.dependencies.add("compile", "com.google.inject:guice:4.1.0")

// 코틀린테스트를 이용한 테스트 코드 호출 <-- 메소드 호출 연쇄를 통해 구조를 만듦
str should startWith("kot")

// 일반 JUnit API 사용
assertTrue(str.startsWith("kot"))
```

* 일반 명령-질의 API 사용 시 중복 코드가 늘어나는 것을 볼 수 있다.

> 태형 Think!
>
> 개인적으로 [코틀린테스트](https://github.com/kotlintest/kotlintest) 의 메소드 호출 연쇄 문법 보다는 일반 JUnit이 코드를 보았을 때 어떠한 결과일지 예측이 더 쉬운 것 같다.
>
> 물론 익숙해진다면 일반 언어와 동일하게 사용하는 코틀린테스트 문법이 더 편해질 수 있지만 적어도 지금은 아닌 것 같다.



#### 11.1.4 내부 DSL로 HTML 만들기

[kotlinx.html 라이브러리](https://github.com/Kotlin/kotlinx.html)을 이용하여 HTML 페이지를 생성해보자.

```kotlin
fun createSimpleTable() = createHtml().
	table {
    tr {
      td { +"cell" }
    }
  }

// 생성되는 html 결과
//<table>
//	<tr>
//		<td>cell</td>
//	</tr>
//</table>
```

* `createSimpleTable()`은 HTML 조각이 들어있는 문자열을 반환한다.
* 솔직하게 위 함수가 HTML 생성이 편하지는 않다.

코틀린 코드로 HTML을 만들려는 이유

* 타입 안정성을 보장한다.
  * 위 코드에서 td는 tr 내에서만 사용할 수 있다. 그렇지 않은 경우 컴파일이 되지 않는다.
* 코틀린 코드를 원하는대로 사용할 수 있다.
  * 맵에 들어있는 원소에 따라 동적으로 표의 칸을 생성할 수 있다.

```kotlin
fun createAnotherTable() = createHTML().table {
  val numbers = mapOf(1 to "one", 2 to two)
  for ((num, string) in numbers) {
    tr {
      td { +"$num" }
      td { +string }
    }
  }
}

// 생성되는 html 결과
//<table>
//	<tr>
//		<td>1</td>
//    <td>one</td>
//	</tr>
//	<tr>
//		<td>2</td>
//    <td>two</td>
//	</tr>
//</table>
```



### 구조화된 API 구축: DSL에서 수신 객체 지정 DSL 사용

#### 11.2.1 수신 객체 지정 람다와 확장 함수 타입

```kotlin
// 람다를 인자로 받는 buildString() 정의
fun buildString(buiderAction: (StringBuilder) -> Unit): String {
  val sb = StringBuilder()
  builderAction(sb)
  return sb.toString()
}

val s = buildString { 
  it.append("Hello, ") 
	it.append("World!")
}
println(s)
// 결과: Hello, World!

// 수신 객체 지정 람다를 사용해 buildString() 정의
fun buildString(builderAction: StringBuilder.() -> Unit): String { // 수신 객체가 있는 함수 타입(확장 함수 타입)의 파라미터 선언
  val sb = StringBuilder()
  sb.builderAction() // 수신 객체로 넘김.
  return sb.toString()
}

val s = buildString {
  this.append("Hello, ")
  append("World!")
}
println(s)
// 결과: Hello, World
```

* 수신 객체 지정 람다를 인자로 넘기므로 람다 내에서 `it`를 사용하지 않아도 된다.
* 확장 함수 타입 선언은 `<수신 객체 타입>.(param1, param2) -> <반환타입>` 형태로 명시한다.

```kotlin
// apply를 이용한 buildString() 정의
fun buildString(builderActon: StringBuilder.() -> Unit): String = StringBuilder().apply(builderAction).toString()
```

* `apply` 함수는 인자로 받은 람다나 함수를 호출하면서 자신의 수신 객체를 람다나 함수의 묵시적 수신 객체로 사용한다.

> 이쯤에서 다시보는 `apply` 함수의 구현과 `with` 함수의 구현

```kotlin
inline fun <T> T.apply(block: T.() -> Unit): T {
	block()
  return this
}

inline fun <T, R> with(receiver: T, block: T.() -> R): R = recevier.block()
```

* `apply`함수
  * 수신 객체 타입에 대한 확장 함수로 선언됐기 때문에 수신 객체의 메소드처럼 불린다.
  * 수신 객체를 묵시적 인자(this)로 받게 된다.
  * **수신 객체를 다시 반환한다.**
* `with` 함수
  * 수신 객체를 첫번째 파라미터로 받는다.
  * **람다를 호출해 얻은 결과를 반환한다.**



#### 11.2.2 수신 객체 지정 람다를 HTML 빌더 안에서 사용

> 들어가기 전에

##### HTML 빌더란?

* HTML을 만들기 위한 코틀린 DSL을 뜻한다.
* 타입 안전한 빌더(type-safe builder)의 대표적인 예다.
* 객체 계층 구조를 선언적으로 정의할 수 있다.
* 책의 저자의 말에 의하면 코틀린 빌더는 사용하기 편하면서도 안전하다고 한다.

```kotlin
fun createSimpleTable() = createHTML().
	table { 
    tr { // == (this@table).tr
      td { +"cell" } // == (this@tr).td
    }
  }
```

* 사용된 `table`, `tr`, `td` 는 고차 함수로 수신 객체 지정 람다를 인자로 받는 형태이다.
* 각 수신 객체 지정 람다가 이름 결정 규칙을 바꾸는 방식으로 되어있다.
  * `table` 함수에 넘겨진 람다에서는 `tr` 함수를 사용해 \<tr> HTML 태그를 먼들 수 있지만 그 람다 밖에서는 `tr` 함수를 찾을 수 없다.
  * `table`에 전달된 수신 객체는 TABLE 이라는 특별한 타입이며, 그 안에 tr 메소드 정의가 있다.  
* 수신 객체 지정 람다가 다른 수신 객체 지정 람다 안에 들어가면 내부 람다에서 외부 람다에 정의된 수신 객체를 사용할 수 있지만 `코틀린 1.1` 부터는 `@DslMarker` 어노테이션을 사용해 중첩된 람다에서 외부 람다의 수신 객체를 접근하지 못하게 제한할 수 있다.

```kotlin
// 간단한(?) HTML 빌더의 구현
open class Tag(val name: String) {
  private val children = mutableListOf<Tag>() // 모든 중첩 태그를 저장
  protected fun <T: Tag> doInit(child: T, init: T.() -> Unit) {
    child.init() // 자식 태그 초기화
    children.add(child) // 자식 태그에 대한 참조 저장
  }
  override fun toString() = "<$name>${children.joinToString("")}</$name>" // 결과 HTML을 문자열로 반환
}
fun table(init: TABLE.() -> Unit) = TABLE().apply(init)
class TABLE: Tag("table") {
  fun tr(init: TR.() -> Unit) = doInit(TR(), init) // TR태그 인스턴스를 새로 만들고 초기환 후 TABLE 태그의 자식으로 등록
}
class TR: Tag("tr") {
  fun td(init TD.() -> Unit) = doInit(TD(), init) // TD 태그의 새 인스턴스를 만들어서 TR 태그의 자식으로 등록
}
class TD: Tag("td")

fun createTable() = 
	table {
    tr {
      td {
        
      }
    }
  }
println(createTable())
// 결과: <table><tr><td></td></tr></table>
```

* 모든 중첩 태그를 저장하는 리스트는 각 태그를 저장해두고 재귀로 호출하여 닫는 태그를 추가하기 위해 사용한다.
* 부모 태그가 가진 자식 목록에 추가하므로 동적으로 태그를 만들 수 있다.
* 전체 구현을 보려면 [kotlinx.html 라이브러리](https://github.com/Kotlin/kotlinx.html) 을 참고하자.

#### 11.2.3 코틀린 빌더: 추상화와 재사용을 가능하게 하는 도구

내부 DSL을 사용하면 일반 코드와 마찬가지로 반복되는 내부 DSL 코드 조각을 새 함수로 묶어서 재사용할 수 있다.

이번 예제는 [부트스트랩 라이브러리](http://getbootstrap.com) 에서 제공하는 드롭다운 메뉴가 있는 HTML 페이지를 코틀린 빌더를 통해 생성해보자.

```kotlin
fun dropdownExample() = createHTML().dropdown {
  dropdownButton { +"Dropdown" }
  dropdownMenu {
    item("#", "Action")
    item("#", "Another action")
    divider()
    dropdownHeader("Header")
    item("#", "Separated link")
  }
}
```

* `item` 함수

  * href 주소(첫번째 파라미터)와 메뉴 원소의 이름(두번째 파라미터)로 구현되어 있다.

  * `li { a (href) { +nam } }`라는 원소를 새로 추가한다.

  * 해당 함수를 UL 태그의 확장 함수로 구현하여 li 태그를 추가할 수 있도록 구현한다.

    `fun UL.item(href: String, name: String) = li { a (href) { +name } }`

만들어야 할 HTML 코드(?) 나 중간 단계의 코틀린 코드, 개선한 함수에 대한 코드는 생략하고 예시로 `item` 에 대한 설명만 추가하였으니 책을 통해 꼭 한번 보도록 하자.



### 11.3 invoke 관례를 사용한 더 유연한 블록 중첩

> 들어가기 전에

* **invoke 관례**를 사용하면 객체를 함수처럼 호출할 수 있다.

* **관례**를 사용하면 특별한 이름이 붙은 함수를 일반 메소드 호출 구문으로 호출하지 않고 더 간단한 다른 구문으로 호출 할 수 있게 한다.

#### 11.3.1 invoke 관례: 함수처럼 호출할 수 있는 객체

```kotlin
class Greeter(val greeting: String) {
  operator fun invoke(name: String) {
    println("$greeting, $name!")
  }
}
val bavarianGreeter = Greeter("Servus")
bavarianGreeter("Dmitry")
// 결과: Servus, Dmitry!
```

* **`operator` 변경자가 붙은 invoke 메소드 정의가 들어있는 클래스의 객체**는  함수처럼 부를 수 있다.
* `bavarianGreeter("Dmitry")` 는 내부적으로 `bavarianGreeter.invoke("Dmitry")`로 컴파일 된다.
* 원하는대로 파라미터 개수나 타입을 지정할 수 있다.
* 여러 파라미터 타입을 지원하기 위해 오버로딩도 가능하다.

#### 11.3.2 invoke 관례와 함수형 타입

* 인라인하는 람다를 제외한 모든 람다는 함수형 인터페이스(Function1 등)를 구현하는 클래스로 컴파일된다.
* 람다를 함수처럼 호출하면 이 관례에 따라 `invoke` 메소드 호출로 변환된다.

```kotlin
data class Issue(val id: String, val project: String, val type: String, val priority: String, val description: String)

class ImportantIssuesPredicate(val project: String): (Issue) -> Boolean {
  override fun invoke(issue: Issue): Boolean {
    return issue.project == project && issue.isImportant()
  }
  
  private fun Issue.isImportant(): Boolean {
    return type == "Bug" &&
            (priority == "Major" || priority == "Critical")
  }
}

val i1 = Issue("IDEA-154446", "IDEA", "Bug", "Major", "Save settings failed")
val i2 = Issue("KT-12183", "Kotlin", "Feature", "Normal", "Intention: convert serveral ~~")

val predicate = ImportantIssuesPredicate("IDEA")
for (issue in listOf(i1, i2).filter(predicate)) { // 술어를 filter에 넘김
  println(issue.id)
}
// 결과: IDEA-154446
```

* 람다를 함수 타입 인터페이스를 구현하는 클래스로 변환하고 그 클래스의 `invoke` 메소드를 오버라이드하면 복잡한 람다가 필요한 구문을 리팩토링할 수 있다.
* 위와 같이 리팩토링할 경우 람다 본문에서 따로 분리해낸 메소드가 영향을 끼치는 영역을 최소화할 수 있다는 장점이 있다.
  * 예제에서는 오직 술어 클래스 내부에서만 람다에서 분리해낸 메소드를 볼 수 있다.

#### 11.3.3 DSL의 invoke 관례: 그레이들에서 의존관계 정의

```kotlin
class DependencyHandler {
  // 일반적인 명령형 API 정의
  fun compile(coordinate: String) {
    println("Added dependency on $coordinate")
  }
  // invoke를 정의해 DSL 스타일 API를 제공
  operator fun invoke(body: DependencyHandler.() -> Unit) {
    body() // == this.body()
  }
}
val dependencies = DependencyHandler()
dependencies.compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
// 결과: Added dependecy on org.jetbrains.kotlin:kotlin-stdlib:1.0.0
dependencies {
  compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
}
// 결과: Added dependecy on org.jetbrains.kotlin:kotlin-stdlib:1.0.0
```

* 두번째 호출은 결과적으로 아래와 같이 변환되게 된다.

  ```kotlin
  dependencies.invoke({
    this.compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
  })
  ```

  * dependencies를 함수처럼 호출하면 람다를 인자로 넘기게 된다.
  * 람다의 타입은 **확장 함수 타입(수신 객체를 지정한 함수 타입)** 이다.
  * 지정한 수신 객체 타입은 DpendencyHandler이다.
  * `invoke` 메소드는 이 수신 객체 지정 람다를 호출한다.
  * `invoke` 메소드가 DependencyHandler의 메소드이므로 이 메소드 내부에서 `this`는 DependencyHandler 객체이다.
  * 따라서 `invoke` 안에서 DependencyHandler 타입의 객체를 따로 명시하지 않고 `compile()` 을 호출할 수 있다.

### 11.4 실전 코틀린 DSL

#### 11.4.1 중위 호출 연쇄: 테스트 프레임워크의 should

```kotlin
// should 함수 구현
infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)

// Matcher 선언
interface Matcher<T> {
  funt test(value: T)
}
class startWith(val prefix: String): Matcher<String> {
  override fun test(value: String) {
    if (!value.startsWith(prefix)) {
      throw AssertionError("String $value does not start with $prefix")
    }
  }
}

"kotlin" should start with("kot")

object start
infix fun String.should(x: start): StartWrapper = StartWrapper(this)
class StartWrapper(val value: String) {
  infix fun with(prefix: String) = if (!value.startsWith(prefix)) {
    throw AssertionError("String $value does not start with $prefix")
  } else {
    Unit
  }
}
```

* 중위 호출로 호출된 should를 일반 메소드로 바꾸면 아래와 같은 코드가 된다.
  * `"kotlin".should(start).with("kot")`
* `start`는 `should` 메소드의 인자이며 (싱글턴) 객체 선언을 참조하며 `should`와 `with`는 중위 호출 구문으로 쓰이는 함수이다.
  * 싱글턴 객체는 인스턴스가 단 하나밖에 없기 때문에 그 객체를 인자로 넘기지 않아도 직접 그 인스턴스에 접근할 수 있기 때문
  * 또한 여기서 start 객체는 함수에 데이터를 넘기기 위해서가 아니라 DSL의 문법을 정의하기 위해 사용된다.
  * start를 인자로 넘김으로서 should를 오버로딩한 함수 중에서 적절한 함수를 선택할 수 있고 그 함수를 호출한 결과로 StartWrapper 인스턴스를 받을 수 있게 된다.

```kotlin
// start 이외에 코틀린테스트 라이브러리에서 지원하는 Matcher
"kotlin" should end with "in" // "in" 으로 끝나는지
"kotlin" should have substring "otl" // "otl" 이라는 부분 문자열이 있는지
```

> 태형 Think!
>
> 개인적으로 이번 챕터는 이해가 잘 가지 않았다. 팀원들과 함께 내용을 공유하며 더 이해해보도록 하자..!

#### 11.4.2 원시 타입에 대한 확장 함수 정의: 날짜 처리

자바 8의 java.time API와 코틀린을 사용해 `1.days.ago` , `1.days.fromNow` 와 같은 API를 구현해보자

```kotlin
import java.time.Period
import java.time.LocalDate

val Int.days: Period
	get() = Period.ofDays(this) // this는 상수의 값을 가리킴

val Period.ago: LocalDate
	get() = LocalDate.now() - this

val Period.fromNow: LocalDate
	get() = LocalDate.now() + this

println(1.days.ago)
// 결과: 2020-04-18

println(1.days.fromNow)
// 결과: 2020-04-20
```

* Period 클래스는 두 날짜 사이의 시간 간격을 나타내는 JDK 8 타입이다.
* 위에 쓰인 `-, +` 는 코틀린에서 제공하는 확장함수가 아닌 LocalData라는 JDK 클래스에 있는 관례에 의해 minus, plus 메소드가 호출되는 것이다.
  * 코틀린의 -, + 연산자와 일치할 경우 호출해주는 관례가 들어있다.

#### 11.4.3 멤버 확장 함수: SQL을 위한 내부 DSL

정의한 확장 함수나 확장프로퍼티는 그들이 선언된 클래스의 멤버인 동시에 그들이 확장하는 다른 타입의 멤버이기도 하다. 이러한 함수나 프로퍼티를 **멤버 확장(Member Extensions)** 이라고 부른다.

```kotlin
// 코틀린 익스포즈드에서 테이블 선언
object Country: Table() {
  val id = integer("id").autoIncrement().primaryKey()
  val name = varchar("name", 50)
}

// 테이블 생성
SchemaUtils.create(Country)
```

* `autoIncrement()` , `primaryKey()` 는 해당 칼럼의 속성을 지정하는 메소드이다.
  * 각 메소드는 확장 함수로 정의되어 있으며 자신의 수신 객체를 다시 반환하므로 메소드 연쇄 호출이 가능하다.
  * Table 내부에 멤버로 확장 함수를 정의하므로서 Table 외부에서는 해당 메소드를 사용할 수 없게 된다. (확장 함수의 특징 중 메소드 적용 범위 제한에 속함)
  * ex) Int 타입이 아닌 경우 autoIncrement() 속성 적용 시 컴파일 실패

> 멤버 확장은 **확장성이 떨어진다**는 단점이 있다.
>
> * 어떤 클래스의 내부에 속해 있기 때문에 기존 클래스를 수정하지 않고는 새로운 멤버 확장을 추가할 수 없다.

#### 11.4.4 안코: 안드로이드 UI를 동적으로 생성하기

[Anko 라이브러리](https://github.com/Kotlin/anko) 는 JetBrain에서 개발한 라이브러리로 코드 작성을 편리하게 도와주는 라이브러리이다.

이번 챕터에서는 안드로이드 애플리케이션의 UI를 구성할 때 어떻게 도움이 되는지 살펴보도록 하자.

```kotlin
// Anko를 사용해 안드로이드 경고창 표시하기
fun Activity.showAreYouSureAlert(process: () -> Unit) {
  alert(title = "Are you sure?", message = "Are you really sure?") {
    positiveButton("Yes") { process() }
    negativeButton("No") { cancel() }
  }
}
```

* 이 코드에서는 `alert` 함수의 3번째 인자, `positiveButton`, `negativeButton` 의 2번째 인자들 총 3개의 람다가 사용되었다.
* `alert ` 함수에서 람다의 수신 객체는 AlertDialogBuilder 타입이다.
  * 코드상 나타나있진 않지만 람다 안에서 해당 빌더의 멤버에 접근해 사용하고 있다.

```kotlin
// alert API의 선언
fun Context.alert(message: String, title: String, init: AlertDialogBuilder.() -> Unit)
class AlertDialogBuilder {
  fun positiveButton(text: String, callback: DialogInterface.() -> Unit)
  fun negativeButton(text: String, callback: DialogInterface.() -> Unit)
}
```

* 내부 DSL을 사용하면 UI와 비즈니스 로직을 다른 컴포넌트로 분리할 수 있지만 모든 컴포넌트를 여전히 코틀린 코드로 작성할 수 있다.

### 11.5 요약

* 내부 DSL은 여러 메소드 호출로 구성된 구조를 더 쉽게 표현할 수 있게 해주는 API를 설계할 때 사용할 수 있는 패턴이다.
* 수신 객체 지정 람다는 람다 본문 안에서 메소드를 결정하는 방식을 재정의함으로써 여러 요소를 중첩시킬 수 있는 구조를 만들 수 있다.
* 수신 객체 지정 람다를 파라미터로 받은 경우 그 람다의 타입은 확장 함수 타입이다. 람다를 파라미터로 받아서 사용하는 함수는 람다를 호출하면서 람다에 수신 객체를 제공한다.
* 외부 템플릿이나 마크업 언어 대신 코틀린 내부 DSL을 사용하면 코드를 추상화하고 재활용할 수 있다.
* 중위 호출 인자로 특별히 이름을 붙인 객체를 사용하면 특수 기호를 사용하지않는 실제 영어처럼 보이는 DSL을 만들 수 있다.
* 원시 타입에 대한 확장을 정의하면 날짜 등의 여러 종류의 상수를 더 읽기 좋게 만들 수 있다.
* invoke 관례를 사용하면 객체를 함수처럼 다룰 수 있다.
* kotlinx.html 라이브러리는 HTML 페이지를 생성하기 위한 내부 DSL을 제공한다. 그 내부 DSL을 확장하면 여러 프론트엔드 개발 프레임워크를 지원하게 만들 수 있다.
* 코틀린테스트 라이브러리는 단위 테스트에서 읽기 쉬운 단언문을 지원하는 내부 DSL을 제공한다.
* 익스포즈드 라이브러리는 데이터베이스를 다루기 위한 내부 DSL을 제공한다.
* 안코 라이브러리는 안드로이드 개발에 필요한 여러 도구를 제공한다. 그런 도구 중에는 UI 레이아웃을 정의하기 위한 내부 DSL도 있다.

### 참고
* [우아한 형제들 - 'Gradle Kotlin DSL' 이야기](https://woowabros.github.io/tools/2019/04/30/gradle-kotlin-dsl.html)

