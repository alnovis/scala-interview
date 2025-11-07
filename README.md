# План подготовки к собеседованию Senior Scala Developer

## 📋 Общая структура (4-6 недель)

**Неделя 1-2**: Основы Scala + Функциональное программирование  
**Неделя 3-4**: Продвинутые темы + Экосистема  
**Неделя 5-6**: Системный дизайн + Mock интервью

---

## 📑 Оглавление

### [🎯 Неделя 1: Основы Scala](#-неделя-1-основы-scala)

#### [День 1-2: Базовый синтаксис и концепции](#день-1-2-базовый-синтаксис-и-концепции)
1. [Collections (List, Map, Set, Vector, Array)](#1-collections-list-map-set-vector-array)
2. [Immutability vs Mutability](#2-immutability-vs-mutability)
3. [Class, Object, Trait, Sealed Trait](#3-class-object-trait-sealed-trait)
4. [Case Classes vs Classes](#4-case-classes-vs-classes)
   - [Structural Equality vs Referential Equality](#41-structural-equality-vs-referential-equality)
5. [Pattern Matching](#5-pattern-matching)
6. [For-Comprehensions](#6-for-comprehensions)
7. [Implicit и Implicit Resolution](#7-implicit-и-implicit-resolution)
8. [Implicit Conversions и Implicit Parameters](#8-implicit-conversions-и-implicit-parameters)
9. [Type Inference и Type Annotations](#9-type-inference-и-type-annotations)
10. [Функции apply и unapply](#10-функции-apply-и-unapply)
11. [Теория категорий для функционального программирования](#11-теория-категорий-для-функционального-программирования)

#### [День 3-4: Функциональное программирование](#день-3-4-функциональное-программирование)

#### [День 5-7: Type System](#день-5-7-type-system)

### [🚀 Неделя 2: Scala Collections + Concurrency](#-неделя-2-scala-collections--concurrency)

#### [День 1-3: Collections Deep Dive](#день-1-3-collections-deep-dive)

#### [День 4-7: Concurrency & Futures](#день-4-7-concurrency--futures)

### [💎 Неделя 3: Продвинутые темы](#-неделя-3-продвинутые-темы)

#### [День 1-3: Cats / Scalaz](#день-1-3-cats--scalaz)

#### [День 4-7: Akka / Akka Streams](#день-4-7-akka--akka-streams)

### [🏗️ Неделя 4: Архитектура и паттерны](#️-неделя-4-архитектура-и-паттерны)

#### [День 1-3: Design Patterns в Scala](#день-1-3-design-patterns-в-scala)

#### [День 4-7: Testing](#день-4-7-testing)

### [🗄️ Неделя 5: Базы данных и интеграции](#️-неделя-5-базы-данных-и-интеграции)

#### [День 1-4: Database access](#день-1-4-database-access)

#### [День 5-7: Message Queues & Integration](#день-5-7-message-queues--integration)

### [🏛️ Неделя 6: System Design + Interview Prep](#️-неделя-6-system-design--interview-prep)

#### [День 1-3: System Design](#день-1-3-system-design)

#### [День 4-7: Mock Interviews](#день-4-7-mock-interviews)

### [📚 Ресурсы для изучения](#-ресурсы-для-изучения)

### [🎤 Типичные вопросы на собеседовании](#-типичные-вопросы-на-собеседовании)

### [✅ Checklist перед собеседованием](#-checklist-перед-собеседованием)

### [💡 Советы](#-советы)

### [🎯 Финальный чек-лист навыков Senior Scala Developer](#-финальный-чек-лист-навыков-senior-scala-developer)

---

## 🎯 Неделя 1: Основы Scala

### День 1-2: Базовый синтаксис и концепции

#### 📖 Теоретические материалы

---

##### 1. Collections (List, Map, Set, Vector, Array)

**List - неизменяемый связный список:**
```scala
// Создание
val list1 = List(1, 2, 3)
val list2 = 1 :: 2 :: 3 :: Nil  // cons operator
val list3 = List.empty[Int]

// Основные операции
list1.head        // 1 - O(1)
list1.tail        // List(2, 3) - O(1)
list1.init        // List(1, 2) - O(n)
list1.last        // 3 - O(n)
0 :: list1        // List(0, 1, 2, 3) - O(1) prepend
list1 :+ 4        // List(1, 2, 3, 4) - O(n) append
```

**Характеристики:**
- Immutable (неизменяемый)
- Эффективен для prepend (добавление в начало) - O(1)
- Inefficient для append (добавление в конец) - O(n)
- Эффективен для итерации с головы
- Структура данных: односвязный список

**Vector - индексированная неизменяемая последовательность:**
```scala
val vector = Vector(1, 2, 3, 4, 5)

// Эффективный доступ по индексу
vector(2)         // 3 - effectively O(1)
vector.updated(2, 10)  // Vector(1, 2, 10, 4, 5) - effectively O(1)

// Эффективное добавление с обеих сторон
0 +: vector       // prepend - effectively O(1)
vector :+ 6       // append - effectively O(1)
```

**Характеристики:**
- Immutable
- Effectively constant time для большинства операций
- Структура данных: 32-way tree
- Хорош для произвольного доступа и апдейтов

**Array - изменяемый массив JVM:**
```scala
val arr = Array(1, 2, 3)
arr(0) = 10       // mutation - O(1)
arr(1)            // доступ - O(1)

// Interop с Java
val javaArray: Array[String] = Array("a", "b")
```

**Характеристики:**
- Mutable (изменяемый)
- Компилируется в Java array
- O(1) доступ и изменение по индексу
- Фиксированный размер

**Set - неупорядоченная коллекция уникальных элементов:**
```scala
val set1 = Set(1, 2, 3, 3)  // Set(1, 2, 3)
set1.contains(2)            // true - O(1) для HashSet
set1 + 4                    // Set(1, 2, 3, 4)
set1 - 2                    // Set(1, 3)

// Операции множеств
val set2 = Set(3, 4, 5)
set1 union set2             // Set(1, 2, 3, 4, 5)
set1 intersect set2         // Set(3)
set1 diff set2              // Set(1, 2)
```

**Map - коллекция пар ключ-значение:**
```scala
val map = Map("a" -> 1, "b" -> 2, "c" -> 3)
val map2 = Map(("a", 1), ("b", 2))  // альтернативный синтаксис

// Доступ
map("a")              // 1 - бросает NoSuchElementException если нет ключа
map.get("a")          // Some(1) - безопасный доступ
map.getOrElse("d", 0) // 0

// Операции
map + ("d" -> 4)      // добавление
map - "a"             // удаление
map.updated("a", 10)  // обновление
```

**Seq - абстрактная последовательность:**
```scala
// Seq - это trait, не конкретная реализация
val seq1: Seq[Int] = Seq(1, 2, 3)  // создается List по умолчанию
val seq2: Seq[Int] = List(1, 2, 3)
val seq3: Seq[Int] = Vector(1, 2, 3)

// Основные операции Seq
seq1(0)               // доступ по индексу
seq1.length           // размер
seq1.indexOf(2)       // поиск индекса элемента
seq1.contains(2)      // проверка наличия
seq1.reverse          // разворот
seq1 ++ seq2          // конкатенация
```

**Характеристики Seq:**
- Trait (абстракция) для упорядоченных коллекций
- Гарантирует порядок элементов
- Поддерживает доступ по индексу
- Имеет две основные ветки: IndexedSeq и LinearSeq

**Иерархия Seq:**
```
Seq
 ├── IndexedSeq (быстрый произвольный доступ)
 │   ├── Vector      (immutable, balanced)
 │   ├── Array       (mutable, Java array)
 │   ├── ArraySeq    (immutable wrapper over array)
 │   └── ArrayBuffer (mutable, resizable)
 │
 └── LinearSeq (быстрый последовательный доступ)
     ├── List        (immutable linked list)
     ├── LazyList    (lazy, infinite sequences)
     └── Queue       (FIFO)
```

**Seq vs List - ключевые отличия:**

| Аспект | Seq | List |
|--------|-----|------|
| **Тип** | Trait (абстракция) | Конкретная реализация |
| **По умолчанию** | Создает List | Всегда List |
| **Гибкость** | Можно заменить реализацию | Фиксированная структура |
| **Performance** | Зависит от реализации | Известные характеристики |
| **Использование** | В сигнатурах API | В реализации |

**Примеры различий:**
```scala
// 1. Seq - это trait, List - класс
val seq: Seq[Int] = Seq(1, 2, 3)      // фактически List(1, 2, 3)
val list: List[Int] = List(1, 2, 3)   // точно List

// 2. Seq можно переопределить на другую реализацию
val seq2: Seq[Int] = Vector(1, 2, 3)  // теперь это Vector
// val list2: List[Int] = Vector(1, 2, 3)  // ERROR! не компилируется

// 3. В API используйте Seq для гибкости
def processData(data: Seq[Int]): Int = data.sum  // принимает любую Seq
processData(List(1, 2, 3))    // работает
processData(Vector(1, 2, 3))  // работает
processData(Array(1, 2, 3))   // работает

// 4. В реализации используйте конкретный тип
class DataProcessor {
  private val cache: List[String] = List.empty  // точно List для предсказуемости
  
  def getData(): Seq[String] = cache  // возвращаем как Seq для гибкости
}

// 5. Pattern matching
val seq: Seq[Int] = Seq(1, 2, 3)
seq match {
  case List(1, 2, 3) => "List"        // работает, т.к. Seq(1,2,3) создает List
  case Vector(1, 2, 3) => "Vector"
  case _ => "Other"
}

val list: List[Int] = List(1, 2, 3)
list match {
  case head :: tail => s"Head: $head"  // работает с List
  case Nil => "Empty"
}
// Seq не поддерживает :: в pattern matching (это специфично для List)

// 6. Операции, специфичные для List
val list = List(1, 2, 3)
val newList = 0 :: list      // cons operator - только для List
// val seq: Seq[Int] = Seq(1, 2, 3)
// val newSeq = 0 :: seq     // ERROR! :: не определен для Seq

// Для Seq нужно использовать:
val seq: Seq[Int] = Seq(1, 2, 3)
val newSeq = 0 +: seq        // prepend - работает для всех Seq
```

**Когда использовать Seq:**
```scala
// ✅ В public API для гибкости
trait Repository {
  def findAll(): Seq[User]              // caller может получить List, Vector, и т.д.
  def saveAll(users: Seq[User]): Unit   // принимает любую Seq
}

// ✅ Когда не важна конкретная реализация
def average(numbers: Seq[Double]): Double = 
  if (numbers.isEmpty) 0.0 
  else numbers.sum / numbers.size

// ✅ Для совместимости с разными коллекциями
def merge(a: Seq[Int], b: Seq[Int]): Seq[Int] = a ++ b
```

**Когда использовать List:**
```scala
// ✅ Когда нужны специфичные операции List
def process(data: List[String]): List[String] = 
  "header" :: data  // :: operator

// ✅ В рекурсивных алгоритмах с pattern matching
def sum(list: List[Int]): Int = list match {
  case Nil => 0
  case head :: tail => head + sum(tail)
}

// ✅ Когда важна производительность prepend
def buildReversed(n: Int): List[Int] = {
  @scala.annotation.tailrec
  def loop(i: Int, acc: List[Int]): List[Int] =
    if (i == 0) acc else loop(i - 1, i :: acc)
  loop(n, Nil)
}

// ✅ В алгоритмах, требующих структурного разделения (head/tail)
def contains[A](list: List[A], elem: A): Boolean = list match {
  case Nil => false
  case head :: tail => head == elem || contains(tail, elem)
}
```

**Best Practices:**
```scala
// ✅ ХОРОШО: Seq в API, конкретный тип внутри
class UserService {
  private val cache: List[User] = List.empty
  
  def getUsers(): Seq[User] = cache
  def addUsers(users: Seq[User]): Unit = {
    // внутри можем конвертировать в нужный тип
    val userList = users.toList
    // ...
  }
}

// ❌ ПЛОХО: конкретный тип в API ограничивает caller'а
class UserService {
  def getUsers(): List[User] = ???  // заставляет всех работать с List
  def addUsers(users: List[User]): Unit = ???  // нельзя передать Vector
}

// ✅ ХОРОШО: используйте IndexedSeq для произвольного доступа
def findMiddle(data: IndexedSeq[Int]): Int = 
  data(data.length / 2)  // O(1) для Vector, Array

// ❌ ПЛОХО: List для частого доступа по индексу
def findMiddle(data: List[Int]): Int = 
  data(data.length / 2)  // O(n) для List!
```

**Performance сравнение:**
```scala
// List - лучше для:
val list = List(1, 2, 3)
0 :: list          // O(1) - prepend
list.head          // O(1) - доступ к голове
list.tail          // O(1) - хвост
list(1000)         // O(n) - доступ по индексу

// Vector (IndexedSeq) - лучше для:
val vector = Vector(1, 2, 3)
0 +: vector        // effectively O(1) - prepend
vector :+ 4        // effectively O(1) - append
vector(1000)       // effectively O(1) - доступ по индексу
vector.updated(1000, 42)  // effectively O(1) - обновление

// Общая рекомендация:
// - List: много prepend, итерация с головы, pattern matching
// - Vector: произвольный доступ, append/prepend с обеих сторон
// - Seq: в API когда не важна реализация
```

**Иерархия коллекций:**
```
Traversable
    ↓
  Iterable
    ↓
  ├── Seq (последовательности)
  │   ├── IndexedSeq (Vector, Array, ArrayBuffer)
  │   └── LinearSeq (List, LazyList, Queue)
  ├── Set
  └── Map
```

---

##### 2. Immutability vs Mutability

**Immutability (неизменяемость):**
```scala
val immutableList = List(1, 2, 3)
val newList = immutableList :+ 4  // создается новый список
// immutableList остается List(1, 2, 3)
// newList = List(1, 2, 3, 4)

val immutableMap = Map("a" -> 1)
val newMap = immutableMap + ("b" -> 2)  // новый Map
```

**Преимущества immutability:**
- **Thread-safety**: безопасность в многопоточной среде без синхронизации
- **Reasoning**: легче понять код - значения не меняются
- **Caching**: можно безопасно кешировать и делиться данными
- **Time-travel debugging**: можно хранить историю состояний
- **Referential transparency**: функции с одинаковыми входами дают одинаковый результат

**Mutability (изменяемость):**
```scala
import scala.collection.mutable

val mutableList = mutable.ListBuffer(1, 2, 3)
mutableList += 4          // изменение на месте
mutableList.append(5)     // изменение на месте

val mutableMap = mutable.Map("a" -> 1)
mutableMap("b") = 2       // изменение
mutableMap += ("c" -> 3)  // изменение
```

**Когда использовать mutable:**
- Performance-critical код с частыми изменениями
- Локальная оптимизация внутри функции
- Interop с Java библиотеками
- Алгоритмы, требующие in-place modifications

**Best practice:**
```scala
// Плохо - mutable видна наружу
class BadExample {
  val data = mutable.ListBuffer[Int]()
  def getData = data  // утечка mutability
}

// Хорошо - инкапсуляция mutability
class GoodExample {
  private val data = mutable.ListBuffer[Int]()
  def getData: List[Int] = data.toList  // возврат immutable копии
  def addData(x: Int): Unit = data += x
}

// Отлично - полностью immutable
class BestExample(private val data: List[Int]) {
  def addData(x: Int): BestExample = new BestExample(data :+ x)
  def getData: List[Int] = data
}
```

---

##### 3. Class, Object, Trait, Sealed Trait

**Class - обычный класс:**
```scala
// Базовый класс
class Person(val name: String, val age: Int) {
  def greet(): String = s"Hello, I'm $name"
}

val person = new Person("Alice", 30)  // нужен new
person.name    // "Alice"
person.greet() // "Hello, I'm Alice"

// Класс с конструктором и методами
class BankAccount(initialBalance: Double) {
  private var balance: Double = initialBalance  // private mutable state
  
  def deposit(amount: Double): Unit = {
    require(amount > 0, "Amount must be positive")
    balance += amount
  }
  
  def withdraw(amount: Double): Boolean = {
    if (amount <= balance) {
      balance -= amount
      true
    } else false
  }
  
  def getBalance: Double = balance
}

val account = new BankAccount(1000)
account.deposit(500)
account.getBalance  // 1500.0
```

**Характеристики Class:**
- Может иметь конструктор с параметрами
- Может содержать mutable и immutable состояние
- Поддерживает наследование
- Требует `new` для создания экземпляра
- Каждый вызов `new` создает новый объект в памяти

**Модификаторы параметров конструктора:**
```scala
// Без val/var - параметр недоступен как поле
class Person1(name: String) {
  // name доступен только в конструкторе
  def greet(): String = s"Hello, $name"
}
val p1 = new Person1("Alice")
// p1.name  // ERROR! нет такого поля

// val - immutable публичное поле
class Person2(val name: String)
val p2 = new Person2("Bob")
p2.name  // "Bob" - можно читать
// p2.name = "Charlie"  // ERROR! val нельзя изменить

// var - mutable публичное поле
class Person3(var name: String)
val p3 = new Person3("Charlie")
p3.name = "David"  // OK - можно изменить

// private val/var
class Person4(private val name: String) {
  def getName: String = name  // доступ через метод
}
```

**Множественные конструкторы:**
```scala
class Person(val name: String, val age: Int) {
  // Вспомогательный конструктор
  def this(name: String) = this(name, 0)
  
  // Еще один конструктор
  def this() = this("Unknown", 0)
}

val p1 = new Person("Alice", 30)
val p2 = new Person("Bob")
val p3 = new Person()
```

---

**Object - singleton объект:**
```scala
// Object - всегда только один экземпляр
object DatabaseConnection {
  private var connection: Option[Connection] = None
  
  def connect(url: String): Unit = {
    if (connection.isEmpty) {
      connection = Some(new Connection(url))
      println("Connected to database")
    }
  }
  
  def getConnection: Option[Connection] = connection
}

// Использование - не нужен new
DatabaseConnection.connect("jdbc:...")
val conn = DatabaseConnection.getConnection
```

**Характеристики Object:**
- Singleton - только один экземпляр в JVM
- Создается лениво при первом обращении
- Не требует `new`
- Не может иметь параметры конструктора
- Может наследовать классы и traits

**Companion Object - объект-компаньон:**
```scala
// Class и Object с одинаковым именем в одном файле
class Person(val name: String, val age: Int)

object Person {
  // Factory methods
  def apply(name: String, age: Int): Person = new Person(name, age)
  
  // Константы
  val ADULT_AGE = 18
  
  // Утилитные методы
  def isAdult(person: Person): Boolean = person.age >= ADULT_AGE
  
  // Приватный доступ к полям класса
  def getSecret(person: Person): String = person.secretField
}

// Class и Object имеют доступ к private членам друг друга
class Person(val name: String, val age: Int) {
  private val secretField = "secret"
  
  def isAdult: Boolean = Person.isAdult(this)
}

// Использование
val person = Person("Alice", 30)  // вызывает Person.apply
Person.isAdult(person)  // true
```

**Object для утилит и констант:**
```scala
object MathUtils {
  val PI: Double = 3.14159265359
  val E: Double = 2.71828182846
  
  def square(x: Double): Double = x * x
  def cube(x: Double): Double = x * x * x
}

MathUtils.square(5)  // 25.0

object StringUtils {
  def capitalize(s: String): String = 
    if (s.isEmpty) s else s.head.toUpper + s.tail
  
  def reverse(s: String): String = s.reverse
}

StringUtils.capitalize("hello")  // "Hello"
```

---

**Trait - интерфейс с реализацией:**
```scala
// Базовый trait
trait Drawable {
  def draw(): String  // абстрактный метод
  
  // Метод с реализацией
  def display(): Unit = {
    println(draw())
  }
}

// Trait может иметь абстрактные и конкретные члены
trait Shape {
  def area(): Double          // абстрактный
  def perimeter(): Double     // абстрактный
  
  def describe(): String = {  // конкретный
    s"Area: ${area()}, Perimeter: ${perimeter()}"
  }
}

// Класс реализует trait
class Circle(radius: Double) extends Shape {
  def area(): Double = math.Pi * radius * radius
  def perimeter(): Double = 2 * math.Pi * radius
}

val circle = new Circle(5)
circle.describe()  // "Area: 78.54..., Perimeter: 31.41..."
```

**Характеристики Trait:**
- Может содержать абстрактные и конкретные методы
- Может иметь поля (val/var)
- Не может иметь параметры конструктора (до Scala 3)
- Поддерживает множественное наследование (mixins)
- Может быть смешан (mixed in) с классом

**Множественное наследование с traits:**
```scala
trait Logging {
  def log(message: String): Unit = {
    println(s"[LOG] $message")
  }
}

trait Timestamped {
  def timestamp: Long = System.currentTimeMillis()
}

trait Authenticated {
  def authenticate(user: String): Boolean
}

// Класс с несколькими traits
class Service extends Logging with Timestamped with Authenticated {
  def authenticate(user: String): Boolean = {
    log(s"Authenticating user: $user at ${timestamp}")
    // логика аутентификации
    true
  }
  
  def process(): Unit = {
    log(s"Processing at ${timestamp}")
  }
}

val service = new Service()
service.process()
```

**Trait с состоянием:**
```scala
trait Counter {
  private var count: Int = 0
  
  def increment(): Unit = count += 1
  def getCount: Int = count
}

class MyClass extends Counter {
  def doSomething(): Unit = {
    increment()
    println(s"Count: $getCount")
  }
}

val obj = new MyClass()
obj.doSomething()  // Count: 1
obj.doSomething()  // Count: 2
```

**Self-types - требования к классу:**
```scala
trait DatabaseAccess {
  def query(sql: String): List[String]
}

// UserService требует DatabaseAccess
trait UserService {
  self: DatabaseAccess =>  // self-type annotation
  
  def getUsers(): List[String] = {
    query("SELECT * FROM users")  // можем использовать методы DatabaseAccess
  }
}

// Правильное использование
class ServiceImpl extends UserService with DatabaseAccess {
  def query(sql: String): List[String] = {
    // реализация
    List.empty
  }
}

// Неправильное - не компилируется
// class BadServiceImpl extends UserService {
//   // ERROR! Нужен DatabaseAccess
// }
```

---

**Sealed Trait - закрытая иерархия:**
```scala
// sealed означает, что все наследники должны быть в том же файле
sealed trait Shape
case class Circle(radius: Double) extends Shape
case class Rectangle(width: Double, height: Double) extends Shape
case class Triangle(base: Double, height: Double) extends Shape

// Попытка наследовать в другом файле - ошибка компиляции
// case class Square(side: Double) extends Shape  // ERROR если в другом файле

def area(shape: Shape): Double = shape match {
  case Circle(r) => math.Pi * r * r
  case Rectangle(w, h) => w * h
  case Triangle(b, h) => 0.5 * b * h
  // Компилятор знает все возможные варианты!
  // Если забыть case - compiler warning
}
```

**Характеристики Sealed Trait:**
- Все наследники должны быть объявлены в том же файле
- Компилятор знает полный список наследников
- Exhaustiveness checking в pattern matching
- Идеально для Algebraic Data Types (ADT)

**Algebraic Data Types (ADT) с sealed trait:**
```scala
// Sum type (Either/Or) - может быть одним из вариантов
sealed trait Result[+A]
case class Success[A](value: A) extends Result[A]
case class Failure(error: String) extends Result[Nothing]

def divide(a: Int, b: Int): Result[Int] = {
  if (b == 0) Failure("Division by zero")
  else Success(a / b)
}

divide(10, 2) match {
  case Success(value) => println(s"Result: $value")
  case Failure(error) => println(s"Error: $error")
}

// Product type (AND) - содержит все поля
case class User(id: Long, name: String, email: String)

// Комбинация
sealed trait Tree[+A]
case class Leaf[A](value: A) extends Tree[A]
case class Branch[A](left: Tree[A], right: Tree[A]) extends Tree[A]
case object Empty extends Tree[Nothing]

val tree: Tree[Int] = Branch(
  Branch(Leaf(1), Leaf(2)),
  Leaf(3)
)
```

**Sealed trait для состояний:**
```scala
sealed trait OrderStatus
case object Pending extends OrderStatus
case object Processing extends OrderStatus
case object Shipped extends OrderStatus
case object Delivered extends OrderStatus
case object Cancelled extends OrderStatus

class Order(var status: OrderStatus) {
  def nextStatus(): Unit = status match {
    case Pending => status = Processing
    case Processing => status = Shipped
    case Shipped => status = Delivered
    case Delivered => println("Already delivered")
    case Cancelled => println("Order cancelled")
  }
  
  def canCancel: Boolean = status match {
    case Pending | Processing => true
    case _ => false
  }
}
```

**Sealed trait vs Enum (Scala 3 preview):**
```scala
// В Scala 2 - sealed trait
sealed trait Color
case object Red extends Color
case object Green extends Color
case object Blue extends Color

// В Scala 3 - enum (более краткий синтаксис)
// enum Color {
//   case Red, Green, Blue
// }
```

---

**Сравнение: Class vs Object vs Trait vs Sealed Trait:**

| Аспект | Class | Object | Trait | Sealed Trait |
|--------|-------|--------|-------|--------------|
| **Инстанцирование** | Множественное (new) | Singleton | Не инстанцируется | Не инстанцируется |
| **Параметры конструктора** | Да | Нет | Нет (Scala 2) | Нет (Scala 2) |
| **Состояние** | Да | Да | Да | Обычно нет |
| **Наследование** | Одиночное | От класса/trait | Множественное | Множественное |
| **Mixins** | Нет | Может быть mixed in | Да | Да |
| **Pattern matching** | С unapply | С unapply | Нет | Да (для наследников) |
| **Exhaustiveness check** | Нет | Нет | Нет | Да |

**Когда использовать:**

**Class:**
```scala
// ✅ Когда нужны множественные экземпляры
class User(val id: Long, var name: String)

// ✅ Mutable состояние
class ShoppingCart {
  private val items = mutable.ListBuffer[Item]()
  def addItem(item: Item): Unit = items += item
}

// ✅ Encapsulation с private состоянием
class BankAccount(private var balance: Double) {
  def deposit(amount: Double): Unit = balance += amount
  def getBalance: Double = balance
}
```

**Object:**
```scala
// ✅ Singleton сервисы
object DatabaseConnection { /* ... */ }

// ✅ Утилиты и хелперы
object StringUtils {
  def reverse(s: String): String = s.reverse
}

// ✅ Константы и конфигурация
object Config {
  val API_URL = "https://api.example.com"
  val TIMEOUT = 30
}

// ✅ Companion object для factory methods
object User {
  def apply(name: String): User = new User(name)
}
```

**Trait:**
```scala
// ✅ Общая функциональность для разных классов
trait Logging {
  def log(msg: String): Unit = println(s"[LOG] $msg")
}

// ✅ Интерфейсы с частичной реализацией
trait Repository[T] {
  def findById(id: Long): Option[T]
  def save(entity: T): T
  
  // Общая логика для всех репозиториев
  def exists(id: Long): Boolean = findById(id).isDefined
}

// ✅ Mixins для добавления функциональности
trait Timestamped {
  val createdAt: Long = System.currentTimeMillis()
}

class User(name: String) extends Timestamped
```

**Sealed Trait:**
```scala
// ✅ ADT - закрытый набор вариантов
sealed trait PaymentMethod
case object Cash extends PaymentMethod
case object CreditCard extends PaymentMethod
case object BankTransfer extends PaymentMethod

// ✅ State machines
sealed trait ConnectionState
case object Disconnected extends ConnectionState
case object Connecting extends ConnectionState
case object Connected extends ConnectionState

// ✅ Error types
sealed trait AppError
case class ValidationError(msg: String) extends AppError
case class DatabaseError(msg: String) extends AppError
case class NetworkError(msg: String) extends AppError

// ✅ Recursive data structures
sealed trait Json
case object JNull extends Json
case class JBool(value: Boolean) extends Json
case class JNumber(value: Double) extends Json
case class JString(value: String) extends Json
case class JArray(values: List[Json]) extends Json
case class JObject(fields: Map[String, Json]) extends Json
```

**Best Practices:**
```scala
// ✅ ХОРОШО: sealed trait для закрытых иерархий
sealed trait Result[+A]
case class Success[A](value: A) extends Result[A]
case class Failure(error: String) extends Result[Nothing]

// ✅ ХОРОШО: companion object для factory methods
case class Email private (address: String)

object Email {
  private val emailRegex = """^[\w.-]+@[\w.-]+\.\w+$""".r
  
  def apply(address: String): Option[Email] = {
    if (emailRegex.matches(address)) Some(new Email(address))
    else None
  }
}

// ✅ ХОРОШО: trait для mixins
trait Auditable {
  val createdAt: Long = System.currentTimeMillis()
  val modifiedAt: Long = System.currentTimeMillis()
}

// ❌ ПЛОХО: object с mutable состоянием (не thread-safe)
object BadCounter {
  var count = 0  // ПЛОХО! Глобальное mutable состояние
  def increment(): Unit = count += 1
}

// ✅ ХОРОШО: immutable состояние или правильная синхронизация
object GoodCounter {
  private val count = new AtomicInteger(0)
  def increment(): Int = count.incrementAndGet()
}

// ❌ ПЛОХО: наследование sealed trait в другом файле
// sealed trait Shape  // в файле Shape.scala
// case class Square(side: Double) extends Shape  // в другом файле - ERROR!

// ✅ ХОРОШО: все наследники в одном файле
sealed trait Shape
case class Circle(r: Double) extends Shape
case class Rectangle(w: Double, h: Double) extends Shape
```

---

##### 4. Case Classes vs Classes

**Case Class - специальный класс для работы с данными:**
```scala
case class Person(name: String, age: Int)

val person1 = Person("Alice", 30)  // не нужен new
val person2 = Person("Alice", 30)

// Автоматически генерируется:
// 1. equals и hashCode
person1 == person2  // true (structural equality)

// 2. toString
person1.toString  // Person(Alice,30)

// 3. copy метод
val person3 = person1.copy(age = 31)  // Person(Alice,31)

// 4. apply и unapply (для pattern matching)
person1 match {
  case Person(name, age) => println(s"$name is $age")
}

// 5. Все параметры становятся val полями
person1.name  // "Alice"
```

**Regular Class:**
```scala
class RegularPerson(val name: String, val age: Int)

val p1 = new RegularPerson("Alice", 30)  // нужен new
val p2 = new RegularPerson("Alice", 30)

p1 == p2  // false (referential equality по умолчанию)
p1.toString  // RegularPerson@4f3f5b24

// Нет автоматического copy
// Нет pattern matching
```

**Когда использовать case class:**
- Immutable data transfer objects (DTO)
- Value objects
- Algebraic Data Types (ADT)
- Когда нужен pattern matching
- Когда важно structural equality

**Когда использовать regular class:**
- Mutable состояние
- Когда нужен referential equality
- Inheritance (case classes плохо подходят для наследования)
- Классы с side effects

---

##### 4.1. Structural Equality vs Referential Equality

**Referential Equality (равенство по ссылке):**
```scala
// Regular класс использует referential equality по умолчанию
class Person(val name: String, val age: Int)

val p1 = new Person("Alice", 30)
val p2 = new Person("Alice", 30)
val p3 = p1

p1 == p2        // false - разные объекты в памяти (разные ссылки)
p1 eq p2        // false - eq всегда проверяет ссылки
p1 == p3        // true - одна и та же ссылка
p1 eq p3        // true

// Сравниваются адреса в памяти
println(p1)     // Person@4f3f5b24
println(p2)     // Person@7c30a502 - другой адрес!
```

**Определение:**
- **Referential equality** (==, eq): два объекта равны, если это один и тот же объект в памяти
- Проверяется адрес в памяти, а не содержимое
- По умолчанию в Java/Scala для regular classes

**Structural Equality (равенство по структуре/значениям):**
```scala
// Case класс автоматически использует structural equality
case class Person(name: String, age: Int)

val p1 = Person("Alice", 30)
val p2 = Person("Alice", 30)
val p3 = Person("Bob", 25)

p1 == p2        // true - одинаковые значения полей
p1 eq p2        // false - разные объекты в памяти
p1 == p3        // false - разные значения

// toString показывает содержимое
println(p1)     // Person(Alice,30)
println(p2)     // Person(Alice,30) - то же содержимое!
```

**Определение:**
- **Structural equality** (==): два объекта равны, если все их поля равны
- Проверяется содержимое, а не адрес в памяти
- Автоматически для case classes
- Требует переопределения equals/hashCode для regular classes

**Детальное сравнение:**
```scala
// 1. Case class - structural equality из коробки
case class Point(x: Int, y: Int)

val point1 = Point(1, 2)
val point2 = Point(1, 2)
val point3 = Point(3, 4)

point1 == point2           // true - structural equality
point1.equals(point2)      // true - то же самое
point1 eq point2           // false - разные объекты
point1 == point3           // false - разные значения

// Работает правильно в коллекциях
val set = Set(point1, point2)
println(set.size)          // 1 - point1 и point2 считаются одинаковыми

val map = Map(point1 -> "A", point2 -> "B")
println(map.size)          // 1 - ключи одинаковые
println(map(point1))       // "B" - второе значение перезаписало первое

// 2. Regular class - referential equality по умолчанию
class PointClass(val x: Int, val y: Int)

val pc1 = new PointClass(1, 2)
val pc2 = new PointClass(1, 2)

pc1 == pc2                 // false - referential equality!
pc1.equals(pc2)            // false
pc1 eq pc2                 // false

// Проблемы в коллекциях
val set2 = Set(pc1, pc2)
println(set2.size)         // 2 - считаются разными!

val map2 = Map(pc1 -> "A", pc2 -> "B")
println(map2.size)         // 2 - разные ключи
// map2(new PointClass(1, 2)) // Не найдет! Каждый new = новый ключ
```

**Переопределение equals и hashCode для regular class:**
```scala
class Point(val x: Int, val y: Int) {
  // Переопределяем equals для structural equality
  override def equals(obj: Any): Boolean = obj match {
    case that: Point => this.x == that.x && this.y == that.y
    case _ => false
  }
  
  // ВАЖНО: всегда переопределяйте hashCode вместе с equals!
  override def hashCode(): Int = {
    val prime = 31
    var result = 1
    result = prime * result + x
    result = prime * result + y
    result
  }
  
  // Опционально: toString для читаемости
  override def toString: String = s"Point($x, $y)"
}

val p1 = new Point(1, 2)
val p2 = new Point(1, 2)

p1 == p2                   // true - теперь structural equality!
p1 eq p2                   // false - все еще разные объекты

// Теперь работает правильно в коллекциях
val set = Set(p1, p2)
println(set.size)          // 1 - правильно!

val map = Map(p1 -> "A", p2 -> "B")
println(map(new Point(1, 2)))  // "B" - работает!
```

**Контракт equals и hashCode:**
```scala
// ПРАВИЛО 1: Если a.equals(b) == true, то a.hashCode() == b.hashCode()
// ПРАВИЛО 2: Если a.hashCode() != b.hashCode(), то a.equals(b) == false

// Плохой пример - нарушение контракта
class BadPoint(val x: Int, val y: Int) {
  override def equals(obj: Any): Boolean = obj match {
    case that: BadPoint => this.x == that.x && this.y == that.y
    case _ => false
  }
  // hashCode не переопределен! Используется дефолтный из Object
  // Это нарушает контракт и ломает коллекции
}

val bp1 = new BadPoint(1, 2)
val bp2 = new BadPoint(1, 2)

bp1 == bp2                 // true
bp1.hashCode() == bp2.hashCode()  // false! ПРОБЛЕМА!

// В коллекциях будет хаос
val badSet = Set(bp1, bp2)
println(badSet.size)       // Может быть 1 или 2 - недетерминированно!
badSet.contains(bp1)       // Может вернуть false даже если элемент есть!

// Хороший пример - case class всё делает правильно
case class GoodPoint(x: Int, y: Int)

val gp1 = GoodPoint(1, 2)
val gp2 = GoodPoint(1, 2)

gp1 == gp2                 // true
gp1.hashCode() == gp2.hashCode()  // true - контракт соблюден!
```

**Практические примеры:**
```scala
// Пример 1: Value Object с structural equality
case class Money(amount: BigDecimal, currency: String) {
  def +(other: Money): Money = {
    require(currency == other.currency, "Currencies must match")
    Money(amount + other.amount, currency)
  }
}

val money1 = Money(100, "USD")
val money2 = Money(100, "USD")
money1 == money2           // true - важно для value objects!

// Пример 2: Entity с referential equality
class User(val id: Long, var name: String, var email: String) {
  // Entity сравнивается по ID, а не по всем полям
  override def equals(obj: Any): Boolean = obj match {
    case that: User => this.id == that.id
    case _ => false
  }
  
  override def hashCode(): Int = id.hashCode()
  
  override def toString: String = s"User($id, $name, $email)"
}

val user1 = new User(1, "Alice", "alice@example.com")
val user2 = new User(1, "Alice", "alice@example.com")
val user3 = new User(1, "Alice Updated", "newemail@example.com")

user1 == user2             // true - тот же ID
user1 == user3             // true - ID одинаковый, даже если поля изменились!
user1 eq user2             // false - разные объекты

// Пример 3: Кеширование - structural equality важно
case class CacheKey(userId: Long, resourceType: String)

val cache = scala.collection.mutable.Map[CacheKey, String]()
cache(CacheKey(1, "profile")) = "data1"

// Позже с новым объектом, но теми же значениями
val key = CacheKey(1, "profile")
cache.get(key)             // Some("data1") - работает благодаря structural equality!

// С regular class без правильного equals/hashCode это не работало бы
```

**Когда использовать каждый тип equality:**

**Structural Equality (case class):**
```scala
// ✅ Value Objects - объекты определяются их значениями
case class Email(address: String)
case class Temperature(degrees: Double, unit: String)
case class Coordinate(latitude: Double, longitude: Double)

// ✅ DTOs / Records - простые контейнеры данных
case class UserDto(name: String, email: String)
case class ApiResponse(status: Int, body: String)

// ✅ Immutable конфигурации
case class DatabaseConfig(host: String, port: Int, database: String)

// ✅ События
case class UserRegistered(userId: Long, timestamp: Long)
```

**Referential Equality (regular class):**
```scala
// ✅ Entities - объекты с идентичностью
class User(val id: Long, var name: String) {
  // Entity равны, если равны их ID, независимо от других полей
  override def equals(obj: Any): Boolean = obj match {
    case that: User => this.id == that.id
    case _ => false
  }
  override def hashCode(): Int = id.hashCode()
}

// ✅ Mutable объекты с состоянием
class Connection(val id: String) {
  private var isOpen: Boolean = true
  def close(): Unit = isOpen = false
  // Referential equality - каждое соединение уникально
}

// ✅ Resources / Handles
class FileHandle(val path: String) {
  // Каждый handle уникален, даже если path одинаковый
}

// ✅ Actors
class UserActor extends Actor {
  // Каждый актор - отдельная сущность с уникальным mailbox
}
```

**Best Practices:**
```scala
// ✅ ХОРОШО: case class для data
case class Person(name: String, age: Int)

// ❌ ПЛОХО: переопределение equals в case class
case class Person(name: String, age: Int) {
  override def equals(obj: Any): Boolean = ???  // НЕ ДЕЛАЙТЕ ТАК!
}

// ✅ ХОРОШО: переопределение equals И hashCode вместе
class Person(val name: String) {
  override def equals(obj: Any): Boolean = ???
  override def hashCode(): Int = ???  // ОБЯЗАТЕЛЬНО!
}

// ❌ ПЛОХО: equals без hashCode
class Person(val name: String) {
  override def equals(obj: Any): Boolean = ???
  // hashCode не переопределен - ОШИБКА!
}

// ✅ ХОРОШО: использование canEqual для type-safe equality
case class Point(x: Int, y: Int) {
  override def canEqual(other: Any): Boolean = other.isInstanceOf[Point]
}

// ✅ ХОРОШО: использование eq для проверки ссылок
val p1 = Person("Alice", 30)
val p2 = p1
if (p1 eq p2) {
  println("Same object in memory")
}
```

**Операторы сравнения в Scala:**
```scala
val p1 = Person("Alice", 30)
val p2 = Person("Alice", 30)
val p3 = p1

// == : structural equality (вызывает equals)
p1 == p2        // true для case class
p1 == p3        // true

// eq : referential equality (проверка ссылок)
p1 eq p2        // false - разные объекты
p1 eq p3        // true - одна ссылка

// ne : отрицание eq
p1 ne p2        // true - не одна ссылка
p1 ne p3        // false

// != : отрицание ==
p1 != p2        // false
```

---

**Case Class дополнительные фичи:**
```scala
// Можно делать var поля (но не рекомендуется)
case class MutablePerson(var name: String, age: Int)

// Можно добавлять методы
case class Point(x: Int, y: Int) {
  def +(other: Point): Point = Point(x + other.x, y + other.y)
  def distanceFrom(other: Point): Double = 
    math.sqrt(math.pow(x - other.x, 2) + math.pow(y - other.y, 2))
}

// Companion object автоматически создается
object Person {  // можно дополнить своими методами
  def adult(name: String): Person = Person(name, 18)
}
```

---

##### 5. Pattern Matching

**Базовое pattern matching:**
```scala
def describe(x: Any): String = x match {
  case 0 => "zero"
  case 1 => "one"
  case s: String => s"string: $s"
  case l: List[_] => s"list of ${l.size} elements"
  case _ => "something else"
}
```

**Exhaustiveness checking (sealed trait):**
```scala
sealed trait Shape
case class Circle(radius: Double) extends Shape
case class Rectangle(width: Double, height: Double) extends Shape
case class Triangle(base: Double, height: Double) extends Shape

// Компилятор предупредит, если забыли case
def area(shape: Shape): Double = shape match {
  case Circle(r) => math.Pi * r * r
  case Rectangle(w, h) => w * h
  case Triangle(b, h) => 0.5 * b * h
  // Если забыть case - compiler warning!
}
```

**Guards (условия в pattern matching):**
```scala
def classify(x: Int): String = x match {
  case n if n < 0 => "negative"
  case 0 => "zero"
  case n if n % 2 == 0 => "positive even"
  case n if n % 2 == 1 => "positive odd"
}

// Pattern matching с несколькими условиями
def complexMatch(x: Any): String = x match {
  case s: String if s.length > 5 => "long string"
  case s: String => "short string"
  case n: Int if n > 0 => "positive number"
  case _ => "other"
}
```

**Extractors (unapply):**
```scala
// Создание своего extractor
object Email {
  def unapply(str: String): Option[(String, String)] = {
    val parts = str.split("@")
    if (parts.length == 2) Some((parts(0), parts(1)))
    else None
  }
}

// Использование
"user@example.com" match {
  case Email(user, domain) => println(s"User: $user, Domain: $domain")
  case _ => println("Invalid email")
}

// Multiple extractors
object PositiveInt {
  def unapply(s: String): Option[Int] = {
    try {
      val n = s.toInt
      if (n > 0) Some(n) else None
    } catch {
      case _: NumberFormatException => None
    }
  }
}

"42" match {
  case PositiveInt(n) => println(s"Positive: $n")
  case _ => println("Not a positive int")
}
```

**Nested pattern matching:**
```scala
case class Address(city: String, country: String)
case class Person(name: String, address: Address)

val person = Person("Alice", Address("Copenhagen", "Denmark"))

person match {
  case Person(name, Address("Copenhagen", country)) =>
    println(s"$name lives in Copenhagen, $country")
  case Person(name, Address(city, "Denmark")) =>
    println(s"$name lives in $city, Denmark")
  case _ => println("Other")
}
```

**Pattern matching в других конструкциях:**
```scala
// В val/var
val (x, y) = (1, 2)
val List(first, second, _*) = List(1, 2, 3, 4, 5)
val head :: tail = List(1, 2, 3)

// В for-comprehension
val pairs = List(("a", 1), ("b", 2), ("c", 3))
for ((key, value) <- pairs) {
  println(s"$key -> $value")
}

// Partial functions
val pf: PartialFunction[Any, String] = {
  case s: String => s"String: $s"
  case i: Int => s"Int: $i"
}
```

---

##### 6. For-Comprehensions

**For-comprehension как syntactic sugar:**
```scala
// Это:
for {
  x <- List(1, 2, 3)
  y <- List(10, 20)
} yield x * y

// Компилируется в:
List(1, 2, 3).flatMap(x =>
  List(10, 20).map(y => x * y)
)
// Результат: List(10, 20, 20, 40, 30, 60)
```

**Правила трансформации:**
```scala
// Одна генератор-линия + yield → map
for (x <- xs) yield f(x)
// ≡ xs.map(x => f(x))

// Несколько генераторов + yield → flatMap + map
for {
  x <- xs
  y <- ys
} yield f(x, y)
// ≡ xs.flatMap(x => ys.map(y => f(x, y)))

// Guard (if условие) → withFilter
for {
  x <- xs
  if condition(x)
} yield f(x)
// ≡ xs.withFilter(x => condition(x)).map(x => f(x))

// Без yield → foreach
for (x <- xs) println(x)
// ≡ xs.foreach(x => println(x))
```

**Практические примеры:**
```scala
// Комбинации элементов
val suits = List("♠", "♥", "♦", "♣")
val ranks = List("A", "K", "Q", "J", "10")

val deck = for {
  suit <- suits
  rank <- ranks
} yield s"$rank$suit"

// С фильтрацией
val evenSquares = for {
  x <- 1 to 10
  if x % 2 == 0
} yield x * x
// List(4, 16, 36, 64, 100)

// С несколькими условиями
for {
  x <- 1 to 100
  if x % 3 == 0
  if x % 5 == 0
  if x < 50
} yield x
// List(15, 30, 45)

// Assignments внутри for
for {
  x <- 1 to 10
  double = x * 2
  if double > 10
} yield double
```

**For-comprehension с Option, Either, Future:**
```scala
// Option
val result: Option[Int] = for {
  a <- Some(10)
  b <- Some(20)
  c <- Some(30)
} yield a + b + c
// Some(60)

// Either
def divide(x: Int, y: Int): Either[String, Int] =
  if (y == 0) Left("Division by zero") else Right(x / y)

val computation: Either[String, Int] = for {
  a <- divide(10, 2)   // Right(5)
  b <- divide(20, 4)   // Right(5)
  c <- divide(a + b, 2) // Right(5)
} yield c
// Right(5)

// Future
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

val future: Future[String] = for {
  user <- getUserById(1)
  orders <- getOrdersByUser(user)
  total <- calculateTotal(orders)
} yield s"Total: $total"
```

---

##### 7. Implicit и Implicit Resolution

**Введение:**

Implicit - это механизм Scala, который позволяет компилятору автоматически находить и подставлять значения или выполнять преобразования. Это одна из самых мощных и одновременно сложных особенностей языка.

**Зачем нужны implicit?**
- Type classes (ad-hoc полиморфизм)
- Автоматическая передача контекста (ExecutionContext, Configuration)
- Extension methods (добавление методов к существующим типам)
- DSL (Domain Specific Languages)
- Уменьшение boilerplate кода

---

**7.1. Implicit Values**

**Базовый синтаксис:**
```scala
// Объявление implicit значения
implicit val defaultTimeout: Int = 30

// Функция с implicit параметром
def connect(host: String)(implicit timeout: Int): Unit = {
  println(s"Connecting to $host with timeout $timeout seconds")
}

// Вызов - компилятор автоматически находит implicit
connect("localhost")  // "Connecting to localhost with timeout 30 seconds"

// Или явно
connect("localhost")(60)
```

**Implicit значения разных типов:**
```scala
// Простые типы
implicit val defaultPort: Int = 8080
implicit val defaultHost: String = "localhost"
implicit val isDebugMode: Boolean = true

// Case classes
case class Config(host: String, port: Int)
implicit val defaultConfig: Config = Config("localhost", 8080)

// Traits и их реализации
trait Formatter[A] {
  def format(value: A): String
}

implicit val intFormatter: Formatter[Int] = new Formatter[Int] {
  def format(value: Int): String = s"Integer: $value"
}

implicit val stringFormatter: Formatter[String] = new Formatter[String] {
  def format(value: String): String = s"String: '$value'"
}

// Использование
def display[A](value: A)(implicit formatter: Formatter[A]): Unit = {
  println(formatter.format(value))
}

display(42)        // "Integer: 42"
display("hello")   // "String: 'hello'"
```

**Множественные implicit параметры:**
```scala
def process[A, B](value: A)(implicit 
  formatter: Formatter[A], 
  converter: Converter[A, B],
  logger: Logger
): B = {
  logger.log(s"Processing: ${formatter.format(value)}")
  converter.convert(value)
}
```

---

**7.2. Implicit Parameters**

**Context Bounds - синтаксический сахар:**
```scala
// Полная запись
def sort[A](list: List[A])(implicit ordering: Ordering[A]): List[A] = 
  list.sorted(ordering)

// То же самое с context bound
def sort[A: Ordering](list: List[A]): List[A] = 
  list.sorted

// Если нужен доступ к implicit внутри
def sort[A: Ordering](list: List[A]): List[A] = {
  val ord = implicitly[Ordering[A]]  // получить implicit явно
  list.sorted(ord)
}

// Множественные context bounds
def complex[A: Ordering: Numeric](values: List[A]): A = {
  val ord = implicitly[Ordering[A]]
  val num = implicitly[Numeric[A]]
  // используем оба type class
  values.max(ord)
}
```

**implicitly - получение implicit значения:**
```scala
// implicitly позволяет получить implicit из scope
def example[A: Ordering]: Unit = {
  val ordering = implicitly[Ordering[A]]
  println(s"Got ordering: $ordering")
}

// Полезно для debugging
implicit val x: Int = 42
val retrieved = implicitly[Int]
println(retrieved)  // 42

// Проверка наличия type class
def hasOrdering[A](implicit ev: Ordering[A] = null): Boolean = 
  ev != null

hasOrdering[Int]     // true
hasOrdering[String]  // true
// hasOrdering[MyClass]  // false (если нет Ordering[MyClass])
```

---

**7.3. Implicit Resolution - правила поиска**

**Порядок поиска implicit (приоритет от высокого к низкому):**

1. **Локальный scope** (current scope)
2. **Explicit imports** (явные импорты)
3. **Wildcard imports** (wildcard импорты)
4. **Companion objects** (объекты-компаньоны)

```scala
// 1. Локальный scope (highest priority)
def example1(): Unit = {
  implicit val localValue: Int = 10
  def useImplicit()(implicit x: Int): Int = x
  
  useImplicit()  // использует localValue = 10
}

// 2. Explicit imports
object Implicits {
  implicit val value: Int = 20
}

def example2(): Unit = {
  import Implicits.value  // explicit import
  def useImplicit()(implicit x: Int): Int = x
  
  useImplicit()  // использует Implicits.value = 20
}

// 3. Wildcard imports
object MoreImplicits {
  implicit val value: Int = 30
}

def example3(): Unit = {
  import MoreImplicits._  // wildcard import
  def useImplicit()(implicit x: Int): Int = x
  
  useImplicit()  // использует MoreImplicits.value = 30
}

// 4. Companion objects (lowest priority среди основных)
trait Show[A] {
  def show(a: A): String
}

object Show {
  // Компилятор автоматически смотрит сюда
  implicit val intShow: Show[Int] = (a: Int) => s"Int($a)"
}

case class User(name: String)

object User {
  // Компилятор автоматически смотрит в companion object User
  implicit val userShow: Show[User] = (u: User) => s"User(${u.name})"
}

def display[A](value: A)(implicit show: Show[A]): String = 
  show.show(value)

display(42)                // находит Show.intShow
display(User("Alice"))     // находит User.userShow
```

**Implicit Scope - где компилятор ищет:**

```scala
trait Codec[A] {
  def encode(a: A): String
  def decode(s: String): A
}

// 1. Companion object самого type class
object Codec {
  implicit val intCodec: Codec[Int] = new Codec[Int] {
    def encode(a: Int): String = a.toString
    def decode(s: String): Int = s.toInt
  }
}

// 2. Companion object типа-аргумента
case class Person(name: String, age: Int)

object Person {
  implicit val personCodec: Codec[Person] = new Codec[Person] {
    def encode(a: Person): String = s"${a.name},${a.age}"
    def decode(s: String): Person = {
      val parts = s.split(",")
      Person(parts(0), parts(1).toInt)
    }
  }
}

// 3. Companion objects базовых типов в type signature
trait Container[F[_], A] {
  def get: F[A]
}

object Container {
  // Компилятор смотрит в companion objects F и A
  implicit def listContainer[A: Codec]: Container[List, A] = ???
}

// Использование - компилятор найдет implicit автоматически
def encode[A](value: A)(implicit codec: Codec[A]): String = 
  codec.encode(value)

encode(42)                    // находит Codec.intCodec
encode(Person("Alice", 30))   // находит Person.personCodec
```

---

**7.4. Implicit Resolution - детальные правила**

**Правило 1: Специфичность (Specificity)**

Более специфичный implicit имеет приоритет над более общим.

```scala
trait Show[A] {
  def show(a: A): String
}

// Общий implicit для всех List
implicit def showList[A]: Show[List[A]] = 
  (list: List[A]) => s"List(${list.mkString(", ")})"

// Специфичный implicit для List[Int]
implicit val showListInt: Show[List[Int]] = 
  (list: List[Int]) => s"IntList[${list.mkString(",")}]"

def display[A](value: A)(implicit show: Show[A]): String = 
  show.show(value)

display(List(1, 2, 3))         // использует showListInt (более специфичный)
display(List("a", "b"))        // использует showList[String]
```

**Правило 2: Inheritance и type hierarchy**

```scala
trait Animal
class Dog extends Animal
class Cat extends Animal

implicit val animalValue: Animal = new Animal {}
implicit val dogValue: Dog = new Dog()

def test(implicit a: Animal): Animal = a

// Использует dogValue (более специфичный тип)
// но только если Dog запрашивается явно
```

**Правило 3: Ambiguous implicits - ошибка компиляции**

```scala
// ОШИБКА - ambiguous implicit values
implicit val value1: Int = 10
implicit val value2: Int = 20

def useImplicit()(implicit x: Int): Int = x

// useImplicit()  // ERROR: ambiguous implicit values
```

**Решение ambiguity:**

```scala
// Способ 1: Явное указание
def useImplicit()(implicit x: Int): Int = x
useImplicit()(value1)  // явно указываем какой использовать

// Способ 2: Локальный scope перекрывает
def example(): Unit = {
  implicit val value1: Int = 10
  implicit val value2: Int = 20  // ERROR в этом scope
}

// Способ 3: Разные типы
implicit val intValue: Int = 10
implicit val stringValue: String = "hello"
// Нет конфликта - разные типы

// Способ 4: Использовать более специфичный
trait LowPriorityImplicits {
  implicit val default: Int = 10
}

object Implicits extends LowPriorityImplicits {
  implicit val specific: Int = 20  // этот имеет приоритет
}
```

---

**7.5. Implicit Scope и Package Objects**

**Package objects для implicit значений:**

```scala
// В файле package.scala
package object myapp {
  implicit val defaultTimeout: Duration = 30.seconds
  implicit val defaultConfig: Config = Config.default
  
  // Type class instances
  implicit val userJsonFormat: JsonFormat[User] = ???
  implicit val orderJsonFormat: JsonFormat[Order] = ???
}

// В любом файле пакета myapp
package myapp

object Service {
  def connect()(implicit timeout: Duration): Unit = {
    // автоматически использует defaultTimeout из package object
  }
}
```

**Organizing implicits:**

```scala
// Хороший паттерн - группировка implicit в objects
object JsonFormats {
  implicit val userFormat: JsonFormat[User] = ???
  implicit val orderFormat: JsonFormat[Order] = ???
  implicit val productFormat: JsonFormat[Product] = ???
}

object DatabaseCodecs {
  implicit val userCodec: Codec[User] = ???
  implicit val orderCodec: Codec[Order] = ???
}

// Использование
import JsonFormats._  // импортируем все JSON форматы
import DatabaseCodecs.userCodec  // или только конкретный
```

---

**7.6. Implicit Priority (приоритеты наследованием)**

**Паттерн Low Priority Implicits:**

```scala
// Используется для разрешения ambiguity через приоритеты
trait LowPriorityImplicits {
  // Низкий приоритет - общий implicit
  implicit def defaultShow[A]: Show[A] = 
    (a: A) => a.toString
}

object Show extends LowPriorityImplicits {
  // Высокий приоритет - специфичные implicit
  implicit val intShow: Show[Int] = 
    (a: Int) => s"Int: $a"
  
  implicit val stringShow: Show[String] = 
    (a: String) => s"String: '$a'"
}

// При импорте Show._ компилятор предпочтет специфичные implicit
import Show._

def display[A: Show](value: A): String = 
  implicitly[Show[A]].show(value)

display(42)         // использует intShow (высокий приоритет)
display("hello")    // использует stringShow (высокий приоритет)
display(true)       // использует defaultShow (низкий приоритет)
```

**Многоуровневые приоритеты:**

```scala
trait LowestPriorityImplicits {
  implicit def fallback[A]: Converter[A] = ???
}

trait LowPriorityImplicits extends LowestPriorityImplicits {
  implicit def numeric[A: Numeric]: Converter[A] = ???
}

trait MediumPriorityImplicits extends LowPriorityImplicits {
  implicit def ordered[A: Ordering]: Converter[A] = ???
}

object Converter extends MediumPriorityImplicits {
  // Highest priority
  implicit val intConverter: Converter[Int] = ???
  implicit val stringConverter: Converter[String] = ???
}

// Приоритеты (от высокого к низкому):
// 1. Converter.intConverter / stringConverter
// 2. MediumPriorityImplicits.ordered
// 3. LowPriorityImplicits.numeric
// 4. LowestPriorityImplicits.fallback
```

---

**7.7. Debugging Implicit Resolution**

**Compiler options для отладки:**

```scala
// В build.sbt
scalacOptions ++= Seq(
  "-Xlog-implicits",           // Показывает какие implicit используются
  "-Xprint:typer",             // Показывает как компилятор разрешил типы
  "-explaintypes"              // Объясняет несовместимость типов
)
```

**Примеры ошибок и их решения:**

```scala
// Ошибка 1: Could not find implicit value
trait Show[A] {
  def show(a: A): String
}

case class User(name: String)

def display[A: Show](value: A): Unit = 
  println(implicitly[Show[A]].show(value))

// display(User("Alice"))  
// ERROR: could not find implicit value for evidence parameter of type Show[User]

// Решение: добавить implicit
implicit val userShow: Show[User] = (u: User) => s"User(${u.name})"
display(User("Alice"))  // OK


// Ошибка 2: Ambiguous implicit values
implicit val show1: Show[Int] = (i: Int) => s"Show1: $i"
implicit val show2: Show[Int] = (i: Int) => s"Show2: $i"

// display(42)  
// ERROR: ambiguous implicit values

// Решение: убрать один или использовать Low Priority Implicits


// Ошибка 3: Diverging implicit expansion
trait Recursive[A] {
  def process(a: A): String
}

implicit def recursiveList[A: Recursive]: Recursive[List[A]] = ???

// implicit val intRecursive: Recursive[Int] = ???
// val result = implicitly[Recursive[List[List[Int]]]]
// ERROR: diverging implicit expansion for type Recursive[List[List[Int]]]

// Решение: добавить базовый случай или ограничить рекурсию
```

**Ручная проверка implicit resolution:**

```scala
// Способ 1: implicitly
val ordering: Ordering[Int] = implicitly[Ordering[Int]]
println(s"Found: $ordering")

// Способ 2: shapeless.the (если используется shapeless)
// import shapeless._
// val evidence = the[Ordering[Int]]

// Способ 3: создать тестовую функцию
def checkImplicit[A](implicit ev: A = null): Boolean = ev != null

println(checkImplicit[Ordering[Int]])      // true
println(checkImplicit[Ordering[MyClass]])  // false если нет implicit

// Способ 4: использовать reify для просмотра expansion
import scala.reflect.runtime.universe._
def showImplicit[A: TypeTag](implicit a: A): Unit = {
  println(reify(a))
}
```

---

**7.8. Best Practices для Implicit**

**✅ Хорошие практики:**

```scala
// 1. Именование implicit values
implicit val defaultTimeout: Int = 30  // хорошо - понятное имя
// implicit val x: Int = 30            // плохо - непонятное имя

// 2. Один implicit на тип в scope
object Config {
  implicit val timeout: Duration = 30.seconds
  // implicit val anotherTimeout: Duration = 60.seconds  // ПЛОХО - ambiguous
}

// 3. Группировка implicit в отдельных objects
object Implicits {
  object json {
    implicit val userFormat: JsonFormat[User] = ???
    implicit val orderFormat: JsonFormat[Order] = ???
  }
  
  object database {
    implicit val userCodec: Codec[User] = ???
    implicit val orderCodec: Codec[Order] = ???
  }
}

// Импортируем только нужные
import Implicits.json._

// 4. Документирование implicit
/**
 * Implicit Ordering for User by name, then by age.
 * Used for sorting users in collections.
 */
implicit val userOrdering: Ordering[User] = 
  Ordering.by(u => (u.name, u.age))

// 5. Type class pattern в companion object
trait Serializer[A] {
  def serialize(a: A): String
}

object Serializer {
  def apply[A](implicit ser: Serializer[A]): Serializer[A] = ser
  
  // Базовые instances в companion object
  implicit val intSerializer: Serializer[Int] = _.toString
  implicit val stringSerializer: Serializer[String] = s => s""""$s""""
}

case class User(name: String)

object User {
  // Instance для User в его companion object
  implicit val userSerializer: Serializer[User] = 
    u => s"""{"name":"${u.name}"}"""
}
```

**❌ Плохие практики:**

```scala
// 1. Implicit для примитивных типов в широком scope
// implicit val defaultInt: Int = 42  // ПЛОХО - слишком общее

// 2. Implicit conversions без явной необходимости
// implicit def intToString(i: Int): String = i.toString  // ПЛОХО

// 3. Слишком много implicit в одном scope
object BadImplicits {
  implicit val a: Int = 1
  implicit val b: String = "hello"
  implicit val c: Boolean = true
  implicit val d: Double = 3.14
  // ... еще 20 implicit
  // ПЛОХО - сложно отследить
}

// 4. Неочевидные implicit
implicit val magicValue: Int = 42  // ПЛОХО - что это?

// 5. Implicit с побочными эффектами
implicit val config: Config = {
  println("Loading config...")  // ПЛОХО - side effect
  loadFromFile("config.json")
}
```

**Когда НЕ использовать implicit:**

```scala
// 1. Когда параметр критичен для понимания кода
// ПЛОХО:
def transfer(amount: Double)(implicit from: Account, to: Account): Unit = ???

// ХОРОШО:
def transfer(from: Account, to: Account, amount: Double): Unit = ???

// 2. Когда есть несколько разумных значений по умолчанию
// ПЛОХО:
implicit val defaultTimeout: Int = 30  // или 60? или 120?

// ХОРОШО:
val shortTimeout: Int = 30
val longTimeout: Int = 120
def connect(timeout: Int = 30): Unit = ???

// 3. Когда implicit делает код менее читаемым
// ПЛОХО:
def process()(implicit a: Int, b: String, c: Boolean, d: Config): Unit = ???

// ХОРОШО:
case class ProcessConfig(a: Int, b: String, c: Boolean, d: Config)
def process(config: ProcessConfig): Unit = ???
```

---

**7.9. Практические примеры**

**Пример 1: ExecutionContext для Future**

```scala
import scala.concurrent.{Future, ExecutionContext}

// Глобальный ExecutionContext
import scala.concurrent.ExecutionContext.Implicits.global

def fetchData()(implicit ec: ExecutionContext): Future[String] = 
  Future {
    // асинхронная операция
    "data"
  }

// Или создаем свой
implicit val customEc: ExecutionContext = 
  ExecutionContext.fromExecutor(java.util.concurrent.Executors.newFixedThreadPool(10))

fetchData()  // использует customEc
```

**Пример 2: Type class для JSON serialization**

```scala
trait JsonWriter[A] {
  def write(value: A): String
}

object JsonWriter {
  def apply[A](implicit writer: JsonWriter[A]): JsonWriter[A] = writer
  
  implicit val intWriter: JsonWriter[Int] = 
    (value: Int) => value.toString
  
  implicit val stringWriter: JsonWriter[String] = 
    (value: String) => s""""$value""""
  
  implicit val booleanWriter: JsonWriter[Boolean] = 
    (value: Boolean) => value.toString
  
  implicit def listWriter[A](implicit writer: JsonWriter[A]): JsonWriter[List[A]] = 
    (values: List[A]) => values.map(writer.write).mkString("[", ",", "]")
  
  implicit def optionWriter[A](implicit writer: JsonWriter[A]): JsonWriter[Option[A]] = {
    case Some(value) => writer.write(value)
    case None => "null"
  }
}

def toJson[A](value: A)(implicit writer: JsonWriter[A]): String = 
  writer.write(value)

// Использование
toJson(42)                           // "42"
toJson("hello")                      // "\"hello\""
toJson(List(1, 2, 3))               // "[1,2,3]"
toJson(Some("test"))                // "\"test\""
toJson(Option.empty[String])        // "null"
```

**Пример 3: Dependency Injection через implicit**

```scala
trait Database {
  def query(sql: String): List[String]
}

trait Cache {
  def get(key: String): Option[String]
  def put(key: String, value: String): Unit
}

class UserService(implicit db: Database, cache: Cache) {
  def getUser(id: Long): Option[String] = {
    val cacheKey = s"user:$id"
    cache.get(cacheKey).orElse {
      val result = db.query(s"SELECT * FROM users WHERE id = $id").headOption
      result.foreach(cache.put(cacheKey, _))
      result
    }
  }
}

// Setup
implicit val database: Database = new Database { /* ... */ }
implicit val cache: Cache = new Cache { /* ... */ }

val userService = new UserService()  // implicit параметры подставляются автоматически
```

---

**7.10. Implicit Parameters (детальное рассмотрение)**

**Определение и использование:**

```scala
// Объявление implicit value
implicit val defaultTimeout: Int = 5000
implicit val defaultRetries: Int = 3

// Функция с implicit параметром
def makeRequest(url: String)(implicit timeout: Int, retries: Int): Response = {
  println(s"Request to $url with timeout=$timeout, retries=$retries")
  // логика запроса
  ???
}

// Вызов без явной передачи параметров
makeRequest("http://api.example.com")
// Компилятор автоматически найдет и подставит defaultTimeout и defaultRetries

// Или явная передача, если нужно переопределить
makeRequest("http://api.example.com")(10000, 5)
```

**Implicit в companion objects:**

```scala
case class ExecutionConfig(threads: Int, timeout: Int)

object ExecutionConfig {
  // Implicit значение в companion object
  implicit val default: ExecutionConfig = ExecutionConfig(4, 5000)
}

def execute(task: Task)(implicit config: ExecutionConfig): Unit = {
  println(s"Executing with ${config.threads} threads")
  // выполнение задачи
}

// Компилятор найдет implicit в companion object
execute(myTask)
```

**Множественные implicit параметры:**

```scala
// Можно иметь несколько implicit parameter lists
def process[A](data: A)(implicit 
  serializer: Serializer[A], 
  validator: Validator[A],
  logger: Logger
): Result = {
  logger.log(s"Processing data")
  if (validator.isValid(data)) {
    serializer.serialize(data)
  } else {
    throw new ValidationException()
  }
}

// Все три implicit будут найдены автоматически
implicit val intSerializer: Serializer[Int] = ???
implicit val intValidator: Validator[Int] = ???
implicit val consoleLogger: Logger = ???

process(42)
```

---

**7.2. Implicit Resolution (Разрешение неявных значений)**

**Как работает поиск implicit:**

Когда компилятор встречает вызов функции с implicit параметром, он ищет подходящее implicit значение в определенном порядке:

```scala
def greet(name: String)(implicit greeting: String): String = 
  s"$greeting, $name"

// Компилятор ищет implicit val greeting: String
```

**Порядок поиска (приоритет от высшего к низшему):**

1. **Локальный scope или наследованный**
2. **Explicit imports**
3. **Wildcard imports**
4. **Companion objects** связанных типов

```scala
// 1. ЛОКАЛЬНЫЙ SCOPE - наивысший приоритет
def example1(): Unit = {
  implicit val localGreeting: String = "Hi"  // Локальный scope
  
  def greet(name: String)(implicit greeting: String) = s"$greeting, $name"
  
  println(greet("Alice"))  // "Hi, Alice"
}

// 2. НАСЛЕДОВАННЫЙ SCOPE
class Base {
  implicit val baseGreeting: String = "Hello"
}

class Derived extends Base {
  def greet(name: String)(implicit greeting: String) = s"$greeting, $name"
  
  def test(): Unit = {
    println(greet("Bob"))  // "Hello, Bob" - найден в родительском классе
  }
}

// 3. EXPLICIT IMPORTS
object Implicits {
  implicit val greetingFromObject: String = "Greetings"
}

def example2(): Unit = {
  import Implicits.greetingFromObject  // Явный import
  
  def greet(name: String)(implicit greeting: String) = s"$greeting, $name"
  println(greet("Charlie"))  // "Greetings, Charlie"
}

// 4. WILDCARD IMPORTS
object MoreImplicits {
  implicit val anotherGreeting: String = "Welcome"
}

def example3(): Unit = {
  import MoreImplicits._  // Wildcard import
  
  def greet(name: String)(implicit greeting: String) = s"$greeting, $name"
  println(greet("David"))  // "Welcome, David"
}

// 5. COMPANION OBJECT - самый низкий приоритет
case class Greeting(text: String)

object Greeting {
  implicit val default: Greeting = Greeting("Hey")
}

def greet(name: String)(implicit greeting: Greeting): String = 
  s"${greeting.text}, $name"

// Компилятор найдет implicit в companion object Greeting
println(greet("Eve"))  // "Hey, Eve"
```

**Детальные правила поиска:**

```scala
// Компилятор ищет implicit в companion objects:
// 1. Companion object типа параметра
// 2. Companion object типа результата
// 3. Companion objects в цепочке наследования

trait Serializer[A] {
  def serialize(a: A): String
}

case class User(name: String)

object User {
  // Companion object типа User
  implicit val userSerializer: Serializer[User] = new Serializer[User] {
    def serialize(u: User): String = s"User(${u.name})"
  }
}

object Serializer {
  // Companion object типа Serializer
  implicit val intSerializer: Serializer[Int] = new Serializer[Int] {
    def serialize(i: Int): String = i.toString
  }
}

def toJson[A](value: A)(implicit serializer: Serializer[A]): String = 
  serializer.serialize(value)

// Оба implicit будут найдены автоматически:
toJson(User("Alice"))  // найден в companion object User
toJson(42)             // найден в companion object Serializer
```

---

**7.3. Implicit Resolution для Generic Types**

**Поиск implicit для параметризованных типов:**

```scala
trait Show[A] {
  def show(a: A): String
}

object Show {
  // Implicit для базовых типов
  implicit val intShow: Show[Int] = (i: Int) => s"Int($i)"
  implicit val stringShow: Show[String] = (s: String) => s"String($s)"
  
  // Implicit для Option - рекурсивный поиск
  implicit def optionShow[A](implicit showA: Show[A]): Show[Option[A]] = 
    new Show[Option[A]] {
      def show(opt: Option[A]): String = opt match {
        case Some(a) => s"Some(${showA.show(a)})"
        case None => "None"
      }
    }
  
  // Implicit для List - рекурсивный поиск
  implicit def listShow[A](implicit showA: Show[A]): Show[List[A]] = 
    new Show[List[A]] {
      def show(list: List[A]): String = 
        list.map(showA.show).mkString("List(", ", ", ")")
    }
}

def print[A](value: A)(implicit show: Show[A]): Unit = 
  println(show.show(value))

// Компилятор рекурсивно находит implicit:
print(42)                        // Show[Int]
print("hello")                   // Show[String]
print(Some(42))                  // Show[Option[Int]] требует Show[Int]
print(List(1, 2, 3))            // Show[List[Int]] требует Show[Int]
print(Some(List(1, 2, 3)))      // Show[Option[List[Int]]] требует Show[List[Int]] и Show[Int]
```

**Implicit resolution для вложенных типов:**

```scala
trait Codec[A] {
  def encode(a: A): String
  def decode(s: String): A
}

object Codec {
  implicit val intCodec: Codec[Int] = new Codec[Int] {
    def encode(i: Int): String = i.toString
    def decode(s: String): Int = s.toInt
  }
  
  implicit def pairCodec[A, B](
    implicit ca: Codec[A], cb: Codec[B]
  ): Codec[(A, B)] = new Codec[(A, B)] {
    def encode(pair: (A, B)): String = 
      s"${ca.encode(pair._1)},${cb.encode(pair._2)}"
    def decode(s: String): (A, B) = {
      val parts = s.split(",")
      (ca.decode(parts(0)), cb.decode(parts(1)))
    }
  }
  
  implicit def mapCodec[K, V](
    implicit ck: Codec[K], cv: Codec[V]
  ): Codec[Map[K, V]] = new Codec[Map[K, V]] {
    def encode(m: Map[K, V]): String = 
      m.map { case (k, v) => s"${ck.encode(k)}:${cv.encode(v)}" }
       .mkString(";")
    def decode(s: String): Map[K, V] = 
      s.split(";").map { pair =>
        val Array(k, v) = pair.split(":")
        ck.decode(k) -> cv.decode(v)
      }.toMap
  }
}

def serialize[A](value: A)(implicit codec: Codec[A]): String = 
  codec.encode(value)

// Компилятор строит цепочку implicit:
serialize(42)                              // Codec[Int]
serialize((1, 2))                          // Codec[(Int, Int)] требует 2x Codec[Int]
serialize(Map(1 -> 2, 3 -> 4))            // Codec[Map[Int, Int]] требует 2x Codec[Int]
```

---

**7.4. Context Bounds (Контекстные границы)**

**Синтаксический сахар для implicit параметров:**

```scala
// Без context bound - явный implicit параметр
def printValue1[A](value: A)(implicit show: Show[A]): Unit = 
  println(show.show(value))

// С context bound - более короткий синтаксис
def printValue2[A: Show](value: A): Unit = {
  val show = implicitly[Show[A]]  // получаем implicit
  println(show.show(value))
}

// Эквивалентны:
printValue1(42)
printValue2(42)

// Context bound для нескольких type classes
def process[A: Show: Codec: Ordering](value: A): Unit = {
  val show = implicitly[Show[A]]
  val codec = implicitly[Codec[A]]
  val ord = implicitly[Ordering[A]]
  
  println(show.show(value))
  // использование codec и ord
}
```

**summoner pattern для context bounds:**

```scala
// Вместо implicitly можно создать apply метод в companion object
trait Show[A] {
  def show(a: A): String
}

object Show {
  // summoner/apply метод
  def apply[A](implicit show: Show[A]): Show[A] = show
  
  implicit val intShow: Show[Int] = _.toString
}

def printValue[A: Show](value: A): Unit = {
  // Более чистый синтаксис вместо implicitly
  println(Show[A].show(value))
}

// Или еще короче с extension method
implicit class ShowOps[A](value: A) {
  def show(implicit s: Show[A]): String = s.show(value)
}

def printValue2[A: Show](value: A): Unit = {
  println(value.show)  // очень чисто!
}
```

---

**7.5. Implicit Scope (Область видимости)**

**Где компилятор ищет implicit значения:**

```scala
// 1. ТЕКУЩИЙ SCOPE
def example1(): Unit = {
  implicit val x: Int = 42
  
  def needsInt(implicit i: Int): Int = i
  
  println(needsInt)  // найдет x в текущем scope
}

// 2. IMPORTED SCOPE
object MyImplicits {
  implicit val y: Int = 100
}

def example2(): Unit = {
  import MyImplicits.y
  
  def needsInt(implicit i: Int): Int = i
  
  println(needsInt)  // найдет y из import
}

// 3. COMPANION OBJECT самого типа
case class Config(timeout: Int)

object Config {
  implicit val default: Config = Config(5000)
}

def useConfig(implicit config: Config): Unit = 
  println(config.timeout)

useConfig  // найдет Config.default автоматически

// 4. COMPANION OBJECT type constructor'а
trait Parser[A] {
  def parse(s: String): A
}

object Parser {
  implicit val intParser: Parser[Int] = (s: String) => s.toInt
  implicit val stringParser: Parser[String] = (s: String) => s
}

def parse[A](s: String)(implicit parser: Parser[A]): A = 
  parser.parse(s)

parse[Int]("42")  // найдет Parser.intParser автоматически

// 5. COMPANION OBJECTS родительских типов
trait Animal
case class Dog(name: String) extends Animal

object Animal {
  implicit val animalShow: Show[Animal] = ???
}

object Dog {
  // наследует implicit из Animal companion object
}

// 6. OUTER SCOPE для вложенных определений
class Outer {
  implicit val outerValue: Int = 1
  
  class Inner {
    def useImplicit(implicit i: Int): Int = i
    
    def test(): Unit = {
      println(useImplicit)  // найдет outerValue из outer scope
    }
  }
}
```

---

**7.6. Приоритет Implicit Resolution**

**Конфликты и их разрешение:**

```scala
// Если несколько implicit подходят, компилятор выберет более специфичный

// Пример 1: Локальный vs Imported
object Implicits {
  implicit val importedValue: Int = 100
}

def example(): Unit = {
  import Implicits.importedValue
  implicit val localValue: Int = 42  // более высокий приоритет
  
  def needsInt(implicit i: Int): Int = i
  
  println(needsInt)  // 42 - локальный побеждает
}

// Пример 2: Более специфичный тип побеждает
trait Show[A] {
  def show(a: A): String
}

object Show {
  // Общий implicit для всех типов
  implicit def anyShow[A]: Show[A] = (a: A) => a.toString
  
  // Специфичный implicit для Int
  implicit val intShow: Show[Int] = (i: Int) => s"Int: $i"
}

def print[A](a: A)(implicit show: Show[A]): Unit = 
  println(show.show(a))

print(42)      // использует intShow (более специфичный)
print("hello") // использует anyShow

// Пример 3: Explicit import vs wildcard import
object Implicits1 {
  implicit val value1: Int = 1
}

object Implicits2 {
  implicit val value2: Int = 2
}

def example2(): Unit = {
  import Implicits1._              // wildcard import
  import Implicits2.value2         // explicit import - выше приоритет
  
  def needsInt(implicit i: Int): Int = i
  
  println(needsInt)  // 2 - explicit import побеждает
}
```

**Ambiguous Implicit Values:**

```scala
// ОШИБКА: ambiguous implicit values
object Implicits1 {
  implicit val timeout1: Int = 30
}

object Implicits2 {
  implicit val timeout2: Int = 60
}

def example(): Unit = {
  import Implicits1._
  import Implicits2._
  
  def needsTimeout(implicit timeout: Int): Int = timeout
  
  // println(needsTimeout)  // ERROR: ambiguous implicit values
  
  // Решение: явно указать какой использовать
  println(needsTimeout(Implicits1.timeout1))
}
```

---

**7.7. Debugging Implicit Resolution**

**Как понять, что нашел компилятор:**

```scala
// 1. Использование implicitly для проверки
def checkImplicit(): Unit = {
  implicit val x: Int = 42
  
  val found: Int = implicitly[Int]  // проверяем что implicit найден
  println(found)  // 42
}

// 2. Compiler flags для отладки
// -Xlog-implicits - показывает все implicit resolution
// -Xprint:typer - показывает код после type checking

// 3. Явное получение implicit для отладки
def debugImplicit[A: Show](value: A): Unit = {
  val show = implicitly[Show[A]]
  println(s"Found Show: ${show.getClass.getName}")
  println(show.show(value))
}

// 4. IntelliJ IDEA показывает implicit подсветкой
def example()(implicit timeout: Int): Unit = {
  println(timeout)  // IDE покажет откуда взят timeout
}

// 5. Ошибки компиляции подсказывают что искалось
def needsShow[A](value: A)(implicit show: Show[A]): String = 
  show.show(value)

// needsShow(42)  
// ERROR: could not find implicit value for parameter show: Show[Int]
```

---

**7.8. Best Practices для Implicit**

```scala
// ✅ ХОРОШО: Один implicit на тип в scope
implicit val defaultTimeout: Int = 5000

// ❌ ПЛОХО: Несколько implicit одного типа
// implicit val timeout1: Int = 5000
// implicit val timeout2: Int = 10000  // ambiguous!

// ✅ ХОРОШО: Использование newtype для разных значений
case class Timeout(value: Int)
case class MaxRetries(value: Int)

implicit val timeout: Timeout = Timeout(5000)
implicit val retries: MaxRetries = MaxRetries(3)

// ✅ ХОРОШО: Implicit в companion object
trait Serializer[A] {
  def serialize(a: A): String
}

object Serializer {
  implicit val intSerializer: Serializer[Int] = _.toString
}

// ✅ ХОРОШО: Явные названия implicit значений
implicit val defaultExecutionContext: ExecutionContext = ???
implicit val jsonSerializer: Serializer[Json] = ???

// ❌ ПЛОХО: Неявные имена
// implicit val x = ???
// implicit val impl = ???

// ✅ ХОРОШО: Context bounds для чистоты
def process[A: Ordering](list: List[A]): List[A] = list.sorted

// ❌ ПЛОХО: Слишком много explicit implicit параметров
// def process[A](list: List[A])(implicit o: Ordering[A], s: Show[A], c: Codec[A]): Unit

// ✅ ХОРОШО: Группировка через type classes
trait ProcessContext[A] {
  def ordering: Ordering[A]
  def show: Show[A]
  def codec: Codec[A]
}

def process[A](list: List[A])(implicit ctx: ProcessContext[A]): Unit = ???

// ✅ ХОРОШО: Package object для common implicits
package object myapp {
  implicit val defaultTimeout: Timeout = Timeout(5000)
  implicit val defaultLogger: Logger = ConsoleLogger
}

// ❌ ПЛОХО: Implicit conversions (избегайте когда возможно)
// implicit def intToString(x: Int): String = x.toString  // confusing!

// ✅ ХОРОШО: Explicit extension methods
implicit class RichInt(val x: Int) extends AnyVal {
  def times(f: => Unit): Unit = (1 to x).foreach(_ => f)
}
```

---

**7.9. Практические паттерны**

```scala
// 1. Type Class Pattern
trait JsonWriter[A] {
  def write(value: A): Json
}

object JsonWriter {
  def apply[A](implicit writer: JsonWriter[A]): JsonWriter[A] = writer
  
  implicit val stringWriter: JsonWriter[String] = 
    str => JsString(str)
  
  implicit val intWriter: JsonWriter[Int] = 
    num => JsNumber(num)
  
  implicit def listWriter[A](implicit writer: JsonWriter[A]): JsonWriter[List[A]] = 
    list => JsArray(list.map(writer.write))
}

// 2. Implicit Evidence Pattern (constraint)
def onlyNumbers[A](value: A)(implicit ev: A =:= Int): Int = value

onlyNumbers(42)       // OK
// onlyNumbers("hello")  // ERROR: cannot prove String =:= Int

// 3. Implicit Not Found annotation
@implicitNotFound("No JsonWriter found for type ${A}. Please provide an implicit JsonWriter[${A}]")
trait JsonWriter[A] {
  def write(value: A): Json
}

// Теперь ошибка будет более информативной

// 4. Conditional Implicit (Low Priority)
trait LowPriorityImplicits {
  implicit def defaultShow[A]: Show[A] = (a: A) => a.toString
}

object Show extends LowPriorityImplicits {
  implicit val intShow: Show[Int] = (i: Int) => s"Int($i)"
  // intShow имеет выше приоритет чем defaultShow
}

// 5. Materialization pattern
trait Default[A] {
  def value: A
}

object Default {
  def apply[A](implicit default: Default[A]): Default[A] = default
  
  // Macro для автоматической генерации Default instances
  implicit def materialize[A]: Default[A] = macro materializeImpl[A]
}
```

---

##### 8. Implicit Conversions и Implicit Parameters

**Implicit Conversions:**
```scala
// Implicit conversion function
implicit def intToString(x: Int): String = x.toString

val s: String = 42  // компилятор применяет intToString(42)

// Implicit class для extension methods
implicit class StringOps(s: String) {
  def exclaim: String = s + "!"
  def repeat(n: Int): String = s * n
}

"hello".exclaim  // "hello!"
"ab".repeat(3)   // "ababab"

// Реальный пример - RichInt
1.to(10)         // метод из RichInt
1 until 10       // метод из RichInt
```

**Implicit Parameters:**
```scala
// Определение функции с implicit параметром
def greet(name: String)(implicit greeting: String): String =
  s"$greeting, $name"

// Implicit value в scope
implicit val defaultGreeting: String = "Hello"

greet("Alice")  // "Hello, Alice" - компилятор находит implicit

// Context bounds (синтаксический сахар)
def printSorted[A: Ordering](list: List[A]): Unit = {
  println(list.sorted)  // Ordering используется неявно
}

// Это эквивалентно:
def printSorted[A](list: List[A])(implicit ord: Ordering[A]): Unit = {
  println(list.sorted(ord))
}
```

**Implicit resolution rules:**
```scala
// 1. Локальный scope или наследованный
implicit val x: Int = 42
def f()(implicit i: Int) = i
f()  // 42

// 2. Explicit imports
import MyImplicits.myImplicit

// 3. Companion object
trait Show[A] {
  def show(a: A): String
}

object Show {
  implicit val intShow: Show[Int] = (i: Int) => s"Int: $i"
}

def print[A](a: A)(implicit s: Show[A]): String = s.show(a)
print(42)  // компилятор находит implicit в companion object

// Приоритет:
// Локальные > Explicit imports > Companion object
```

**Type Class pattern с implicit:**
```scala
// Type class
trait JsonSerializer[A] {
  def toJson(value: A): String
}

// Implicit instances
object JsonSerializer {
  implicit val intSerializer: JsonSerializer[Int] =
    (value: Int) => value.toString
  
  implicit val stringSerializer: JsonSerializer[String] =
    (value: String) => s""""$value""""
  
  implicit def listSerializer[A](implicit aSerializer: JsonSerializer[A]): JsonSerializer[List[A]] =
    (values: List[A]) => values.map(aSerializer.toJson).mkString("[", ",", "]")
}

// Использование
def toJson[A](value: A)(implicit serializer: JsonSerializer[A]): String =
  serializer.toJson(value)

toJson(42)                    // "42"
toJson("hello")               // "\"hello\""
toJson(List(1, 2, 3))         // "[1,2,3]"
toJson(List("a", "b"))        // "[\"a\",\"b\"]"

// Или с context bound:
def toJson[A: JsonSerializer](value: A): String =
  implicitly[JsonSerializer[A]].toJson(value)
```

**Осторожно с implicit conversions:**
```scala
// Плохо - неявные конверсии могут сбивать с толку
implicit def stringToInt(s: String): Int = s.toInt

val x: Int = "42"  // работает, но неочевидно

// Хорошо - явная конверсия
val x: Int = "42".toInt

// Implicit conversions считаются bad practice в современном Scala
// Лучше использовать implicit classes для extension methods
```

---

##### 9. Type Inference и Type Annotations

**Type Inference - вывод типов:**
```scala
// Компилятор выводит типы автоматически
val x = 42              // x: Int
val s = "hello"         // s: String
val list = List(1, 2, 3) // list: List[Int]

// Функции - тип параметров нужен, возвращаемый выводится
def add(x: Int, y: Int) = x + y  // возвращает Int

// Generic типы
def identity[A](x: A) = x
identity(42)      // A = Int выводится из аргумента
identity("hello") // A = String

// Вывод из context
val list: List[String] = List("a", "b")  // элементы выводятся как String
```

**Когда нужны Type Annotations:**
```scala
// 1. Параметры функций (обязательно)
def process(data: String): Unit = ???

// 2. Public API (best practice)
class Service {
  def getData(): List[User] = ???  // лучше указать явно
}

// 3. Recursive functions (обязательно)
def factorial(n: Int): Int = {  // Int обязателен
  if (n <= 1) 1
  else n * factorial(n - 1)
}

// 4. Когда тип неочевиден или слишком generic
val data: List[String] = processFile()  // явно указываем тип

// 5. Overloaded методы
def show(x: Int): String = x.toString
def show(x: String): String = x
val f: Int => String = show  // нужно указать тип

// 6. Empty collections
val emptyList = List.empty[Int]  // иначе List[Nothing]
val emptyMap: Map[String, Int] = Map()
```

**Type Ascription (принудительное указание типа):**
```scala
// Использование : для указания типа
val x = 42: Int
val y = "hello": String

// Полезно для уточнения типа
val list: Seq[Int] = List(1, 2, 3)  // List приводится к Seq

// Для работы с generics
def process[A](x: A): A = x
process[String]("hello")  // явно указываем type parameter
```

**Best Practices:**
```scala
// Плохо - избыточные аннотации
val x: Int = 42
val s: String = "hello"
val list: List[Int] = List(1, 2, 3)

// Хорошо - только где нужно
val x = 42
val s = "hello"
val list = List(1, 2, 3)

// Хорошо - аннотации для clarity в public API
trait UserService {
  def getUser(id: Long): Future[Option[User]]
  def createUser(user: User): Future[Either[Error, User]]
}

// Хорошо - аннотации для сложных типов
val complexFunction: (Int => String) => List[String] = ???
```

---

##### 10. Функции apply и unapply

**Apply - вызов объектов как функций:**

```scala
// apply позволяет вызывать объект как функцию
class Greeter(greeting: String) {
  def apply(name: String): String = s"$greeting, $name!"
}

val greeter = new Greeter("Hello")
greeter.apply("Alice")    // "Hello, Alice!"
greeter("Alice")          // то же самое - синтаксический сахар

// Apply в companion object - factory pattern
class Person(val name: String, val age: Int)

object Person {
  // apply как фабричный метод
  def apply(name: String, age: Int): Person = new Person(name, age)
  
  // можно иметь несколько overloaded apply
  def apply(name: String): Person = new Person(name, 0)
}

// Использование без new благодаря apply
val person1 = Person("Alice", 30)    // вызывает Person.apply("Alice", 30)
val person2 = Person("Bob")          // вызывает Person.apply("Bob")
```

**Case class автоматически создает apply:**
```scala
case class User(id: Long, name: String)

// Компилятор автоматически генерирует apply в companion object
// object User {
//   def apply(id: Long, name: String): User = new User(id, name)
// }

val user = User(1, "Alice")  // не нужен new!
```

**Apply для коллекций:**
```scala
// Все коллекции используют apply для создания и доступа
val list = List(1, 2, 3)        // List.apply(1, 2, 3)
val element = list(0)            // list.apply(0) = 1

val map = Map("a" -> 1, "b" -> 2)  // Map.apply(...)
val value = map("a")             // map.apply("a") = 1

val array = Array(1, 2, 3)
array(0) = 10                    // array.update(0, 10)
val elem = array(0)              // array.apply(0)
```

**Практические применения apply:**
```scala
// 1. Builder pattern
class QueryBuilder {
  private var table: String = ""
  private var conditions: List[String] = List.empty
  
  def from(t: String): QueryBuilder = { table = t; this }
  def where(cond: String): QueryBuilder = { 
    conditions = conditions :+ cond
    this 
  }
  
  // apply для выполнения запроса
  def apply(): String = {
    val whereClause = if (conditions.isEmpty) "" 
                      else s" WHERE ${conditions.mkString(" AND ")}"
    s"SELECT * FROM $table$whereClause"
  }
}

val query = new QueryBuilder()
  .from("users")
  .where("age > 18")
  .where("active = true")

println(query())  // SELECT * FROM users WHERE age > 18 AND active = true

// 2. Function object
object Multiplier {
  def apply(x: Int, y: Int): Int = x * y
}

Multiplier(3, 4)  // 12

// 3. Partial application
class Adder(x: Int) {
  def apply(y: Int): Int = x + y
}

val add5 = new Adder(5)
add5(3)   // 8
add5(10)  // 15

// 4. Callable configuration
case class ServerConfig(host: String, port: Int) {
  def apply(): String = s"http://$host:$port"
}

val config = ServerConfig("localhost", 8080)
println(config())  // "http://localhost:8080"
```

---

**Unapply - экстракторы для pattern matching:**

```scala
// unapply - обратная операция apply, используется для pattern matching
class Email(val user: String, val domain: String)

object Email {
  // apply для создания
  def apply(user: String, domain: String): Email = 
    new Email(user, domain)
  
  // unapply для деконструкции (pattern matching)
  def unapply(email: Email): Option[(String, String)] = 
    Some((email.user, email.domain))
  
  // unapply также может парсить строку
  def unapply(emailStr: String): Option[(String, String)] = {
    val parts = emailStr.split("@")
    if (parts.length == 2) Some((parts(0), parts(1)))
    else None
  }
}

// Использование в pattern matching
val email = Email("user", "example.com")
email match {
  case Email(user, domain) => println(s"User: $user, Domain: $domain")
}

// Использование с String
"alice@example.com" match {
  case Email(user, domain) => println(s"User: $user, Domain: $domain")
  case _ => println("Invalid email")
}
```

**Case class автоматически создает unapply:**
```scala
case class Person(name: String, age: Int)

// Компилятор автоматически генерирует unapply
// object Person {
//   def unapply(p: Person): Option[(String, Int)] = 
//     Some((p.name, p.age))
// }

val person = Person("Alice", 30)
person match {
  case Person(name, age) => println(s"$name is $age years old")
}

// Можно игнорировать поля
person match {
  case Person(name, _) => println(s"Name: $name")
  case Person(_, age) => println(s"Age: $age")
}
```

**Типы unapply:**

```scala
// 1. unapply с Option - для опционального извлечения
object Even {
  def unapply(n: Int): Option[Int] = 
    if (n % 2 == 0) Some(n) else None
}

42 match {
  case Even(n) => println(s"$n is even")
  case _ => println("odd")
}

// 2. unapply с Boolean - для проверок без извлечения
object Positive {
  def unapply(n: Int): Boolean = n > 0
}

10 match {
  case Positive() => println("positive")  // обратите внимание на ()
  case _ => println("not positive")
}

// 3. unapplySeq - для последовательностей переменной длины
object Names {
  def unapplySeq(s: String): Option[Seq[String]] = 
    Some(s.split(" ").toSeq)
}

"Alice Bob Charlie" match {
  case Names(first, second, third) => 
    println(s"$first, $second, $third")
  case Names(first, second, _*) => 
    println(s"At least: $first, $second")
  case _ => println("No names")
}

// 4. unapply для сложных структур
object FullName {
  def unapply(name: String): Option[(String, String, Option[String])] = {
    val parts = name.split(" ")
    parts.length match {
      case 2 => Some((parts(0), parts(1), None))
      case 3 => Some((parts(0), parts(1), Some(parts(2))))
      case _ => None
    }
  }
}

"John Smith" match {
  case FullName(first, last, None) => 
    println(s"$first $last")
  case FullName(first, middle, Some(last)) => 
    println(s"$first $middle $last")
  case _ => println("Invalid name")
}
```

**Продвинутые паттерны с unapply:**

```scala
// 1. Вложенные extractors
case class Address(city: String, country: String)
case class Person(name: String, address: Address)

val person = Person("Alice", Address("Copenhagen", "Denmark"))

person match {
  case Person(name, Address("Copenhagen", country)) =>
    println(s"$name lives in Copenhagen, $country")
  case Person(name, Address(city, "Denmark")) =>
    println(s"$name lives in $city, Denmark")
  case _ => println("Other")
}

// 2. Custom extractors с валидацией
object PositiveInt {
  def unapply(s: String): Option[Int] = {
    try {
      val n = s.toInt
      if (n > 0) Some(n) else None
    } catch {
      case _: NumberFormatException => None
    }
  }
}

object NonEmptyString {
  def unapply(s: String): Option[String] = 
    if (s != null && s.nonEmpty) Some(s) else None
}

// Комбинирование extractors
def processInput(input: String): String = input match {
  case PositiveInt(n) => s"Positive number: $n"
  case NonEmptyString(s) => s"Non-empty string: $s"
  case _ => "Invalid input"
}

processInput("42")      // "Positive number: 42"
processInput("hello")   // "Non-empty string: hello"
processInput("")        // "Invalid input"

// 3. Infix notation с unapply
object :: {
  def unapply[A](list: List[A]): Option[(A, List[A])] = 
    if (list.isEmpty) None else Some((list.head, list.tail))
}

List(1, 2, 3, 4) match {
  case first :: second :: rest => 
    println(s"First: $first, Second: $second, Rest: $rest")
  case _ => println("Too short")
}

// 4. Name-based extractors (Scala 2.13+)
object Twice {
  def unapply(n: Int): Some[Int] = Some(n * 2)
}

21 match {
  case Twice(result) => println(s"Twice 21 is $result")  // 42
}

// 5. Regular expression extractor
object EmailRegex {
  private val pattern = "(.+)@(.+)\\.(.+)".r
  
  def unapply(email: String): Option[(String, String, String)] = 
    email match {
      case pattern(user, domain, tld) => Some((user, domain, tld))
      case _ => None
    }
}

"user@example.com" match {
  case EmailRegex(user, domain, tld) =>
    println(s"User: $user, Domain: $domain, TLD: $tld")
  case _ => println("Invalid email")
}
```

**Apply и unapply вместе - симметрия:**

```scala
// Классический паттерн: apply создает, unapply разбирает
class Fraction(val numerator: Int, val denominator: Int) {
  require(denominator != 0, "Denominator cannot be zero")
  
  override def toString: String = s"$numerator/$denominator"
}

object Fraction {
  // apply - конструктор
  def apply(numerator: Int, denominator: Int): Fraction = 
    new Fraction(numerator, denominator)
  
  // unapply - деконструктор
  def unapply(f: Fraction): Option[(Int, Int)] = 
    Some((f.numerator, f.denominator))
  
  // Дополнительный apply для упрощения
  def apply(whole: Int): Fraction = new Fraction(whole, 1)
  
  // Дополнительный unapply для строки
  def unapply(s: String): Option[(Int, Int)] = {
    val parts = s.split("/")
    if (parts.length == 2) {
      try {
        Some((parts(0).toInt, parts(1).toInt))
      } catch {
        case _: NumberFormatException => None
      }
    } else None
  }
}

// Использование apply
val f1 = Fraction(3, 4)      // apply(Int, Int)
val f2 = Fraction(5)         // apply(Int)

// Использование unapply с объектом
f1 match {
  case Fraction(n, d) => println(s"$n/$d")
}

// Использование unapply со строкой
"3/4" match {
  case Fraction(n, d) => println(s"Parsed: $n/$d")
  case _ => println("Invalid fraction")
}

// Симметрия: apply создает, unapply разбирает
val fraction = Fraction(3, 4)           // apply
val Fraction(num, den) = fraction       // unapply
println(s"$num / $den")                 // 3 / 4
```

**Best Practices:**

```scala
// ✅ ХОРОШО: apply для factory methods
object User {
  def apply(name: String): User = new User(name.capitalize)
  def apply(id: Long): User = loadFromDatabase(id)
}

// ✅ ХОРОШО: unapply возвращает Option для безопасности
object SafeInt {
  def unapply(s: String): Option[Int] = 
    try Some(s.toInt) catch { case _: Exception => None }
}

// ✅ ХОРОШО: используйте case class для простых случаев
case class Point(x: Int, y: Int)  // apply и unapply автоматически

// ❌ ПЛОХО: unapply который всегда возвращает Some
object BadExtractor {
  def unapply(x: Int): Some[Int] = Some(x)  // бесполезно
}

// ✅ ХОРОШО: meaningful extractors
object Even {
  def unapply(n: Int): Option[Int] = 
    if (n % 2 == 0) Some(n) else None
}

// ❌ ПЛОХО: сложная логика в apply
object User {
  def apply(data: String): User = {
    // 100 строк парсинга и валидации - плохо!
    // Лучше вынести в отдельный метод
  }
}

// ✅ ХОРОШО: чистый apply, логика вынесена
object User {
  def apply(data: String): Option[User] = parseUser(data)
  
  private def parseUser(data: String): Option[User] = {
    // сложная логика здесь
  }
}
```

**Практические примеры:**

```scala
// 1. URL Parser
object URL {
  def unapply(url: String): Option[(String, String, String)] = {
    val pattern = """(https?):\/\/([^\/]+)(\/.*)""".r
    url match {
      case pattern(protocol, host, path) => Some((protocol, host, path))
      case _ => None
    }
  }
}

"https://example.com/api/users" match {
  case URL(protocol, host, path) =>
    println(s"$protocol://$host$path")
}

// 2. Range checker
object InRange {
  def unapply(n: Int): Option[Int] = 
    if (n >= 0 && n <= 100) Some(n) else None
}

def processScore(score: Int): String = score match {
  case InRange(s) => s"Valid score: $s"
  case _ => "Score out of range"
}

// 3. Type validator
object ValidEmail {
  private val emailRegex = 
    """^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$""".r
  
  def unapply(email: String): Option[String] = 
    if (emailRegex.matches(email)) Some(email) else None
}

def subscribe(email: String): String = email match {
  case ValidEmail(e) => s"Subscribed: $e"
  case _ => "Invalid email"
}

// 4. Complex data extraction
case class Order(id: Long, items: List[String], total: BigDecimal)

object ExpensiveOrder {
  def unapply(order: Order): Option[(Long, BigDecimal)] = 
    if (order.total > 1000) Some((order.id, order.total))
    else None
}

val order = Order(1, List("laptop", "mouse"), BigDecimal(1500))

order match {
  case ExpensiveOrder(id, total) =>
    println(s"Expensive order #$id: $$${total}")
  case _ => println("Regular order")
}
```

---

##### 11. Теория категорий для функционального программирования

**Введение:**

Теория категорий - математическая дисциплина, изучающая абстрактные структуры и связи между ними. В функциональном программировании она предоставляет формальную основу для понимания композиции, абстракции и полиморфизма.

**Зачем теория категорий в Scala?**
- Понимание библиотек типа Cats и Scalaz
- Композиция и переиспользование кода
- Reasoning о программах на высоком уровне абстракции
- Гарантии корректности через типы
- Паттерны функционального программирования

---

**11.1. Категория (Category)**

**Определение категории:**

Категория состоит из:
1. **Объекты** (Objects) - в программировании это типы
2. **Морфизмы** (Morphisms/Arrows) - в программировании это функции
3. **Композиция морфизмов** - композиция функций
4. **Identity морфизм** для каждого объекта

**Законы категории:**
```scala
// 1. Ассоциативность композиции
// (f ∘ g) ∘ h = f ∘ (g ∘ h)

def f(x: Int): String = x.toString
def g(x: Double): Int = x.toInt
def h(x: Boolean): Double = if (x) 1.0 else 0.0

// Композиция ассоциативна
val compose1 = (f compose g) compose h
val compose2 = f compose (g compose h)
// compose1(true) == compose2(true)

// 2. Identity - нейтральный элемент композиции
// f ∘ id = f
// id ∘ f = f

def identity[A](a: A): A = a

val result1 = f compose identity[Int]
val result2 = identity[String] compose f
// result1(42) == f(42) == result2(42)
```

**Категория Scala:**
```scala
// В Scala:
// - Объекты категории = типы (Int, String, List[A], Option[A], etc.)
// - Морфизмы = функции (A => B)
// - Композиция = andThen или compose
// - Identity = identity функция

object ScalaCategory {
  // Identity морфизм
  def id[A]: A => A = a => a
  
  // Композиция морфизмов
  def compose[A, B, C](f: B => C, g: A => B): A => C = 
    a => f(g(a))
  
  // Или используя встроенные методы
  def example(): Unit = {
    val f: Int => String = _.toString
    val g: String => Int = _.length
    
    val h1 = f andThen g  // слева направо
    val h2 = g compose f  // справа налево
    
    println(h1(42))  // 2
    println(h2(42))  // 2
  }
}
```

**Примеры категорий в программировании:**
```scala
// 1. Категория типов Scala (Scal)
//    - Объекты: Int, String, Boolean, User, etc.
//    - Морфизмы: функции A => B

// 2. Категория для конкретного типа (например, List)
//    - Один объект: тип List
//    - Морфизмы: List[A] => List[B] через map

// 3. Kleisli категория (для монад)
//    - Морфизмы: A => F[B] где F - монада
```

---

**11.2. Функтор (Functor)**

**Определение:**

Функтор - это отображение между категориями, которое:
1. Отображает объекты в объекты
2. Отображает морфизмы в морфизмы
3. Сохраняет композицию
4. Сохраняет identity

**В программировании:**
Функтор - это type constructor `F[_]` с операцией `map`, которая применяет функцию к значению внутри контейнера.

**Trait Functor:**
```scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

// Законы функтора:
// 1. Identity: map(fa)(identity) == fa
// 2. Composition: map(map(fa)(f))(g) == map(fa)(f andThen g)
```

**Примеры функторов в Scala:**
```scala
// 1. Option - функтор
val optionFunctor = new Functor[Option] {
  def map[A, B](fa: Option[A])(f: A => B): Option[B] = fa match {
    case Some(a) => Some(f(a))
    case None => None
  }
}

val opt: Option[Int] = Some(42)
optionFunctor.map(opt)(_ * 2)  // Some(84)

// 2. List - функтор
val listFunctor = new Functor[List] {
  def map[A, B](fa: List[A])(f: A => B): List[B] = fa match {
    case Nil => Nil
    case head :: tail => f(head) :: map(tail)(f)
  }
}

val list = List(1, 2, 3)
listFunctor.map(list)(_ * 2)  // List(2, 4, 6)

// 3. Either[E, _] - функтор (по правому типу)
class EitherFunctor[E] extends Functor[Either[E, *]] {
  def map[A, B](fa: Either[E, A])(f: A => B): Either[E, B] = fa match {
    case Right(a) => Right(f(a))
    case Left(e) => Left(e)
  }
}

// 4. Future - функтор
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

val futureFunctor = new Functor[Future] {
  def map[A, B](fa: Future[A])(f: A => B): Future[B] = 
    fa.map(f)
}

// 5. Function1[R, *] - функтор (по результату)
class Function1Functor[R] extends Functor[Function1[R, *]] {
  def map[A, B](fa: R => A)(f: A => B): R => B = 
    r => f(fa(r))
}
```

**Проверка законов функтора:**
```scala
import org.scalacheck.Prop.forAll
import org.scalacheck.Properties

object FunctorLaws extends Properties("Functor") {
  // Закон Identity
  property("identity") = forAll { (list: List[Int]) =>
    val id = (x: Int) => x
    listFunctor.map(list)(id) == list
  }
  
  // Закон Composition
  property("composition") = forAll { (list: List[Int]) =>
    val f = (x: Int) => x * 2
    val g = (x: Int) => x + 1
    
    val left = listFunctor.map(listFunctor.map(list)(f))(g)
    val right = listFunctor.map(list)(f andThen g)
    
    left == right
  }
}
```

**Практическое применение функторов:**
```scala
// Функторы позволяют применять функции в контексте
case class Box[A](value: A)

implicit val boxFunctor: Functor[Box] = new Functor[Box] {
  def map[A, B](fa: Box[A])(f: A => B): Box[B] = Box(f(fa.value))
}

// Generic функция работает с любым функтором
def doubleInContext[F[_]](fa: F[Int])(implicit F: Functor[F]): F[Int] = 
  F.map(fa)(_ * 2)

doubleInContext(Some(21))      // Some(42)
doubleInContext(List(1, 2, 3)) // List(2, 4, 6)
doubleInContext(Box(21))       // Box(42)
```

---

**11.3. Аппликативный функтор (Applicative)**

**Определение:**

Аппликативный функтор - это функтор с двумя дополнительными операциями:
1. `pure` (или `point`) - помещает значение в контекст
2. `ap` (apply) - применяет функцию в контексте к значению в контексте

**Trait Applicative:**
```scala
trait Applicative[F[_]] extends Functor[F] {
  def pure[A](a: A): F[A]
  
  def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
  
  // map можно выразить через pure и ap
  override def map[A, B](fa: F[A])(f: A => B): F[B] = 
    ap(pure(f))(fa)
}

// Законы Applicative:
// 1. Identity: ap(pure(id))(fa) == fa
// 2. Composition: ap(ap(ap(pure(compose))(u))(v))(w) == ap(u)(ap(v)(w))
// 3. Homomorphism: ap(pure(f))(pure(x)) == pure(f(x))
// 4. Interchange: ap(u)(pure(y)) == ap(pure(f => f(y)))(u)
```

**Примеры Applicative:**
```scala
// Option Applicative
implicit val optionApplicative: Applicative[Option] = new Applicative[Option] {
  def pure[A](a: A): Option[A] = Some(a)
  
  def ap[A, B](ff: Option[A => B])(fa: Option[A]): Option[B] = 
    (ff, fa) match {
      case (Some(f), Some(a)) => Some(f(a))
      case _ => None
    }
}

// List Applicative
implicit val listApplicative: Applicative[List] = new Applicative[List] {
  def pure[A](a: A): List[A] = List(a)
  
  def ap[A, B](ff: List[A => B])(fa: List[A]): List[B] = 
    for {
      f <- ff
      a <- fa
    } yield f(a)
}

// Использование
val optF: Option[Int => String] = Some(_.toString)
val optA: Option[Int] = Some(42)
optionApplicative.ap(optF)(optA)  // Some("42")

val listF: List[Int => Int] = List(_ * 2, _ + 10)
val listA: List[Int] = List(1, 2, 3)
listApplicative.ap(listF)(listA)  // List(2, 4, 6, 11, 12, 13)
```

**Applicative для независимых вычислений:**
```scala
// Applicative позволяет комбинировать независимые вычисления
import cats.implicits._

case class User(name: String, age: Int, email: String)

def validateName(name: String): Option[String] = 
  if (name.nonEmpty) Some(name) else None

def validateAge(age: Int): Option[Int] = 
  if (age >= 0 && age <= 150) Some(age) else None

def validateEmail(email: String): Option[String] = 
  if (email.contains("@")) Some(email) else None

// С Applicative можем комбинировать валидации
def createUser(name: String, age: Int, email: String): Option[User] = 
  (validateName(name), validateAge(age), validateEmail(email)).mapN(User.apply)

createUser("Alice", 30, "alice@example.com")  // Some(User(...))
createUser("", 30, "alice@example.com")       // None
```

**Applicative vs Monad:**
```scala
// Applicative - независимые вычисления (можно параллелить)
def fetchUserApplicative(id1: Int, id2: Int): Option[(User, User)] = 
  (fetchUser(id1), fetchUser(id2)).tupled  // параллельно

// Monad - зависимые вычисления (последовательно)
def fetchUserMonad(id: Int): Option[User] = 
  for {
    user <- fetchUser(id)         // сначала получаем user
    friend <- fetchUser(user.friendId)  // затем используем его данные
  } yield friend
```

---

**11.4. Монада (Monad)**

**Определение:**

Монада - это аппликативный функтор с операцией `flatMap` (или `bind`), которая позволяет последовательно комбинировать вычисления в контексте.

**Trait Monad:**
```scala
trait Monad[F[_]] extends Applicative[F] {
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  
  // pure уже есть от Applicative
  // def pure[A](a: A): F[A]
  
  // map и ap можно выразить через flatMap и pure
  override def map[A, B](fa: F[A])(f: A => B): F[B] = 
    flatMap(fa)(a => pure(f(a)))
  
  override def ap[A, B](ff: F[A => B])(fa: F[A]): F[B] = 
    flatMap(ff)(f => map(fa)(f))
}

// Законы монады:
// 1. Left Identity: flatMap(pure(a))(f) == f(a)
// 2. Right Identity: flatMap(fa)(pure) == fa
// 3. Associativity: flatMap(flatMap(fa)(f))(g) == flatMap(fa)(a => flatMap(f(a))(g))
```

**Примеры монад:**
```scala
// Option Monad
implicit val optionMonad: Monad[Option] = new Monad[Option] {
  def pure[A](a: A): Option[A] = Some(a)
  
  def flatMap[A, B](fa: Option[A])(f: A => Option[B]): Option[B] = 
    fa match {
      case Some(a) => f(a)
      case None => None
    }
}

// List Monad
implicit val listMonad: Monad[List] = new Monad[List] {
  def pure[A](a: A): List[A] = List(a)
  
  def flatMap[A, B](fa: List[A])(f: A => List[B]): List[B] = 
    fa.flatMap(f)
}

// Either Monad
class EitherMonad[E] extends Monad[Either[E, *]] {
  def pure[A](a: A): Either[E, A] = Right(a)
  
  def flatMap[A, B](fa: Either[E, A])(f: A => Either[E, B]): Either[E, B] = 
    fa match {
      case Right(a) => f(a)
      case Left(e) => Left(e)
    }
}

// Future Monad
import scala.concurrent.{Future, ExecutionContext}

class FutureMonad(implicit ec: ExecutionContext) extends Monad[Future] {
  def pure[A](a: A): Future[A] = Future.successful(a)
  
  def flatMap[A, B](fa: Future[A])(f: A => Future[B]): Future[B] = 
    fa.flatMap(f)
}
```

**For-comprehension = Monadic composition:**
```scala
// For-comprehension это синтаксический сахар для flatMap
val result: Option[Int] = for {
  x <- Some(10)          // flatMap
  y <- Some(20)          // flatMap
  z <- Some(30)          // map (последний)
} yield x + y + z

// Компилируется в:
val result2: Option[Int] = 
  Some(10).flatMap(x =>
    Some(20).flatMap(y =>
      Some(30).map(z =>
        x + y + z
      )
    )
  )
```

**Практическое применение монад:**
```scala
// 1. Обработка опциональных значений
def getUser(id: Long): Option[User] = ???
def getAddress(user: User): Option[Address] = ???
def getCountry(address: Address): Option[Country] = ???

def getUserCountry(id: Long): Option[Country] = 
  for {
    user <- getUser(id)
    address <- getAddress(user)
    country <- getCountry(address)
  } yield country

// 2. Обработка ошибок
def divide(a: Int, b: Int): Either[String, Int] = 
  if (b == 0) Left("Division by zero") else Right(a / b)

val computation: Either[String, Int] = for {
  x <- divide(10, 2)   // Right(5)
  y <- divide(20, 4)   // Right(5)
  z <- divide(x + y, 2) // Right(5)
} yield z

// 3. Асинхронные вычисления
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def fetchUser(id: Long): Future[User] = ???
def fetchOrders(user: User): Future[List[Order]] = ???
def calculateTotal(orders: List[Order]): Future[Double] = ???

val totalFuture: Future[Double] = for {
  user <- fetchUser(1)
  orders <- fetchOrders(user)
  total <- calculateTotal(orders)
} yield total

// 4. IO операции
sealed trait IO[A] {
  def flatMap[B](f: A => IO[B]): IO[B] = FlatMap(this, f)
  def map[B](f: A => B): IO[B] = flatMap(a => Pure(f(a)))
}
case class Pure[A](value: A) extends IO[A]
case class Effect[A](effect: () => A) extends IO[A]
case class FlatMap[A, B](io: IO[A], f: A => IO[B]) extends IO[B]

object IO {
  def pure[A](a: A): IO[A] = Pure(a)
  def effect[A](a: => A): IO[A] = Effect(() => a)
}

val program: IO[Unit] = for {
  _ <- IO.effect(println("What's your name?"))
  name <- IO.effect(scala.io.StdIn.readLine())
  _ <- IO.effect(println(s"Hello, $name!"))
} yield ()
```

---

**11.5. Natural Transformation (Естественное преобразование)**

**Определение:**

Natural Transformation - это преобразование между функторами, которое сохраняет структуру.

```scala
// Natural Transformation от F к G
trait NaturalTransformation[F[_], G[_]] {
  def apply[A](fa: F[A]): G[A]
}

// Часто обозначается как F ~> G
type ~>[F[_], G[_]] = NaturalTransformation[F, G]

// Закон естественности:
// G.map(transform(fa))(f) == transform(F.map(fa)(f))
```

**Примеры:**
```scala
// Option ~> List
val optionToList: Option ~> List = new (Option ~> List) {
  def apply[A](fa: Option[A]): List[A] = fa.toList
}

optionToList(Some(42))  // List(42)
optionToList(None)      // List()

// List ~> Option (берем первый элемент)
val listToOption: List ~> Option = new (List ~> Option) {
  def apply[A](fa: List[A]): Option[A] = fa.headOption
}

listToOption(List(1, 2, 3))  // Some(1)
listToOption(List())         // None

// Either[String, *] ~> Option
val eitherToOption: Either[String, *] ~> Option = 
  new (Either[String, *] ~> Option) {
    def apply[A](fa: Either[String, A]): Option[A] = fa.toOption
  }

// Try ~> Either[Throwable, *]
import scala.util.Try

val tryToEither: Try ~> Either[Throwable, *] = 
  new (Try ~> Either[Throwable, *]) {
    def apply[A](fa: Try[A]): Either[Throwable, A] = fa.toEither
  }
```

---

**11.6. Monoid (Моноид)**

**Определение:**

Моноид - это алгебраическая структура с:
1. Бинарной ассоциативной операцией `combine`
2. Нейтральным элементом `empty`

```scala
trait Monoid[A] {
  def empty: A
  def combine(x: A, y: A): A
}

// Законы моноида:
// 1. Associativity: combine(combine(x, y), z) == combine(x, combine(y, z))
// 2. Left Identity: combine(empty, x) == x
// 3. Right Identity: combine(x, empty) == x
```

**Примеры моноидов:**
```scala
// Int с сложением
implicit val intAdditionMonoid: Monoid[Int] = new Monoid[Int] {
  def empty: Int = 0
  def combine(x: Int, y: Int): Int = x + y
}

// Int с умножением
val intMultiplicationMonoid: Monoid[Int] = new Monoid[Int] {
  def empty: Int = 1
  def combine(x: Int, y: Int): Int = x * y
}

// String
implicit val stringMonoid: Monoid[String] = new Monoid[String] {
  def empty: String = ""
  def combine(x: String, y: String): String = x + y
}

// List
implicit def listMonoid[A]: Monoid[List[A]] = new Monoid[List[A]] {
  def empty: List[A] = List.empty
  def combine(x: List[A], y: List[A]): List[A] = x ++ y
}

// Option (с внутренним моноидом)
implicit def optionMonoid[A](implicit A: Monoid[A]): Monoid[Option[A]] = 
  new Monoid[Option[A]] {
    def empty: Option[A] = None
    def combine(x: Option[A], y: Option[A]): Option[A] = 
      (x, y) match {
        case (Some(a), Some(b)) => Some(A.combine(a, b))
        case (Some(a), None) => Some(a)
        case (None, Some(b)) => Some(b)
        case (None, None) => None
      }
  }

// Map (слияние с моноидом для значений)
implicit def mapMonoid[K, V](implicit V: Monoid[V]): Monoid[Map[K, V]] = 
  new Monoid[Map[K, V]] {
    def empty: Map[K, V] = Map.empty
    def combine(x: Map[K, V], y: Map[K, V]): Map[K, V] = 
      y.foldLeft(x) { case (acc, (k, v)) =>
        acc.updated(k, V.combine(acc.getOrElse(k, V.empty), v))
      }
  }
```

**Практическое применение моноидов:**
```scala
// Generic функция для fold
def foldMap[A, B](list: List[A])(f: A => B)(implicit M: Monoid[B]): B = 
  list.map(f).foldLeft(M.empty)(M.combine)

// Примеры использования
foldMap(List(1, 2, 3, 4))(identity)  // 10 (с intAdditionMonoid)
foldMap(List("a", "b", "c"))(identity)  // "abc" (с stringMonoid)

// Параллельное вычисление с моноидами
def parallelSum[A](list: List[A])(implicit M: Monoid[A]): A = {
  if (list.length <= 1) {
    list.headOption.getOrElse(M.empty)
  } else {
    val (left, right) = list.splitAt(list.length / 2)
    M.combine(parallelSum(left), parallelSum(right))
  }
}

// Word count с моноидами
case class WordCount(words: Int, chars: Int)

implicit val wordCountMonoid: Monoid[WordCount] = new Monoid[WordCount] {
  def empty: WordCount = WordCount(0, 0)
  def combine(x: WordCount, y: WordCount): WordCount = 
    WordCount(x.words + y.words, x.chars + y.chars)
}

def countWords(text: String): WordCount = 
  WordCount(text.split("\\s+").length, text.length)

val texts = List("Hello world", "Scala is great", "Monoids are useful")
val totalCount = texts.map(countWords).foldLeft(wordCountMonoid.empty)(wordCountMonoid.combine)
// WordCount(7, 42)
```

---

**11.7. Semigroup (Полугруппа)**

**Определение:**

Semigroup - это моноид без нейтрального элемента (только бинарная ассоциативная операция).

```scala
trait Semigroup[A] {
  def combine(x: A, y: A): A
}

// Monoid extends Semigroup
trait Monoid[A] extends Semigroup[A] {
  def empty: A
}

// Закон Semigroup:
// Associativity: combine(combine(x, y), z) == combine(x, combine(y, z))
```

**Примеры:**
```scala
// NonEmptyList - semigroup но не monoid (нет empty)
case class NonEmptyList[A](head: A, tail: List[A])

implicit def nelSemigroup[A]: Semigroup[NonEmptyList[A]] = 
  new Semigroup[NonEmptyList[A]] {
    def combine(x: NonEmptyList[A], y: NonEmptyList[A]): NonEmptyList[A] = 
      NonEmptyList(x.head, x.tail ++ (y.head :: y.tail))
  }

// Max/Min - semigroup для чисел
def maxSemigroup[A: Ordering]: Semigroup[A] = new Semigroup[A] {
  def combine(x: A, y: A): A = 
    if (Ordering[A].compare(x, y) >= 0) x else y
}

def minSemigroup[A: Ordering]: Semigroup[A] = new Semigroup[A] {
  def combine(x: A, y: A): A = 
    if (Ordering[A].compare(x, y) <= 0) x else y
}
```

---

**11.8. Связь концепций - диаграмма иерархии**

```
Semigroup
    ↓
  Monoid
    
Functor
    ↓
Applicative
    ↓
  Monad
```

**Практический пример использующий всю иерархию:**
```scala
import cats._
import cats.implicits._

// Пример: валидация формы

case class FormData(name: String, email: String, age: Int)

// Используем Validated (Applicative, не Monad)
type ValidationResult[A] = ValidatedNel[String, A]

def validateName(name: String): ValidationResult[String] =
  if (name.nonEmpty) name.validNel
  else "Name cannot be empty".invalidNel

def validateEmail(email: String): ValidationResult[String] =
  if (email.contains("@")) email.validNel
  else "Invalid email".invalidNel

def validateAge(age: Int): ValidationResult[Int] =
  if (age >= 18) age.validNel
  else "Must be at least 18".invalidNel

// Applicative позволяет собрать все ошибки
def validateForm(name: String, email: String, age: Int): ValidationResult[FormData] =
  (validateName(name), validateEmail(email), validateAge(age)).mapN(FormData.apply)

// Примеры
validateForm("Alice", "alice@example.com", 30)
// Valid(FormData("Alice", "alice@example.com", 30))

validateForm("", "invalid", 15)
// Invalid(NonEmptyList("Name cannot be empty", "Invalid email", "Must be at least 18"))
```

---

**Практика:**

```scala
// Задача 1: Реализовать immutable Stack
trait Stack[+A] {
  def push[B >: A](elem: B): Stack[B]
  def pop: (A, Stack[A])
  def isEmpty: Boolean
}

// Задача 2: Pattern matching с extractors
// Написать extractor для Email валидации

// Задача 3: For-comprehension
// Реализовать flatten для Option[Option[A]]
```

**Вопросы для самопроверки:**

- Разница между val, var, def, lazy val?
- Что такое contravariance и covariance? Когда использовать +A и -A?
- Как работает implicit resolution?
- Разница между Seq, IndexedSeq, LinearSeq?

---

### День 3-4: Функциональное программирование

**Темы:**

- Higher-order functions (map, flatMap, fold, reduce)
- Function composition
- Currying и partial application
- Монады (Option, Either, Try, Future)
- For-comprehensions как syntactic sugar для flatMap
- Recursion vs tail recursion (@tailrec)
- Lazy evaluation (Stream/LazyList)

**Практика:**

```scala
// Задача 1: Реализовать свою монаду
trait Monad[F[_]] {
  def pure[A](a: A): F[A]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  def map[A, B](fa: F[A])(f: A => B): F[B] = flatMap(fa)(a => pure(f(a)))
}

// Задача 2: Tail recursive Fibonacci
def fibonacci(n: Int): BigInt = ???

// Задача 3: Implement traverse for List
def traverse[A, B](list: List[A])(f: A => Option[B]): Option[List[B]] = ???

// Задача 4: Kleisli composition
def compose[A, B, C](f: A => Option[B], g: B => Option[C]): A => Option[C] = ???
```

**Вопросы:**

- Что такое монада? Законы монад?
- Разница между map и flatMap?
- Что такое Applicative? Разница с Monad?
- Как работает @tailrec?

---

### День 5-7: Type System

**Темы:**

- Variance annotations (+, -, invariant)
- Type bounds (upper <:, lower >:)
- Type classes (ad-hoc polymorphism)
- Context bounds и view bounds
- Path-dependent types
- Existential types
- Phantom types
- Higher-kinded types (HKT)

**Практика:**

```scala
// Задача 1: Type class для сериализации
trait Serializer[A] {
  def serialize(a: A): String
}

object Serializer {
  def apply[A](implicit ser: Serializer[A]): Serializer[A] = ser
  
  implicit val intSerializer: Serializer[Int] = ???
  implicit def listSerializer[A: Serializer]: Serializer[List[A]] = ???
}

// Задача 2: Variance
class Box[+A]
// Почему этот код не компилируется?
// class Box[+A] { def set(a: A): Unit = ??? }

// Задача 3: Higher-kinded types
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

// Задача 4: Path-dependent types
class Graph {
  class Node
  class Edge(val from: Node, val to: Node)
}
```

**Вопросы:**

- Что такое type class? Преимущества над наследованием?
- Объясните variance. Когда использовать каждый вид?
- Что такое context bounds (A: TypeClass)?
- Разница между F[_] и F[A]?

---

## 🚀 Неделя 2: Scala Collections + Concurrency

### День 1-3: Collections Deep Dive

**Темы:**

- Collection hierarchy (Traversable, Iterable, Seq, Set, Map)
- Performance characteristics (O-notation для каждой операции)
- View (lazy collections)
- Parallel collections
- Custom collections
- Collection combinators (partition, groupBy, span, etc.)

**Практика:**

```scala
// Задача 1: Реализовать свой LinkedList
sealed trait MyList[+A] {
  def head: A
  def tail: MyList[A]
  def isEmpty: Boolean
  def ::[B >: A](elem: B): MyList[B]
}

// Задача 2: Performance analysis
// Сравните производительность:
// List vs Vector vs Array для разных операций

// Задача 3: groupBy + map
case class Transaction(userId: String, amount: Double, category: String)
// Найти общую сумму трат по категориям для каждого пользователя

// Задача 4: Sliding window
// Реализовать moving average используя sliding
```

**Вопросы:**

- Разница между List и Vector? Когда использовать каждый?
- Как работает Stream/LazyList?
- Performance: head/tail vs init/last?
- Что такое view? Когда использовать?

---

### День 4-7: Concurrency & Futures

**Темы:**

- Future и Promise
- ExecutionContext
- Future composition (map, flatMap, sequence, traverse)
- Error handling (recover, recoverWith, fallbackTo)
- Blocking vs non-blocking
- Async/Await patterns
- Actor model (Akka basics)
- STM (Software Transactional Memory)

**Практика:**

```scala
// Задача 1: Parallel HTTP requests
def fetchUrls(urls: List[String]): Future[List[Response]] = ???

// Задача 2: Timeout implementation
def withTimeout[A](future: Future[A], timeout: Duration): Future[A] = ???

// Задача 3: Retry logic
def retry[A](f: => Future[A], retries: Int): Future[A] = ???

// Задача 4: Circuit breaker pattern
class CircuitBreaker {
  def call[A](f: => Future[A]): Future[A] = ???
}

// Задача 5: Rate limiter
class RateLimiter(maxRequests: Int, per: Duration) {
  def execute[A](f: => Future[A]): Future[A] = ???
}
```

**Вопросы:**

- Разница между Future и Promise?
- Как работает ExecutionContext?
- Что такое callback hell? Как избежать?
- Blocking operations в Future - почему плохо?

---

## 💎 Неделя 3: Продвинутые темы

### День 1-3: Cats / Scalaz

**Темы:**

- Semigroup, Monoid
- Functor, Applicative, Monad
- Monad Transformers (OptionT, EitherT)
- Validated vs Either
- IO Monad
- Free Monad
- Tagless Final

**Практика:**

```scala
import cats._
import cats.implicits._

// Задача 1: Implement Monoid для своего типа
case class Stats(count: Int, sum: Double)
implicit val statsMonoid: Monoid[Stats] = ???

// Задача 2: Traverse
def validateAll[A](list: List[A])(f: A => Either[String, A]): Either[String, List[A]] = ???

// Задача 3: Monad Transformer
def getUser(id: String): Future[Option[User]] = ???
def getOrders(user: User): Future[Option[List[Order]]] = ???
// Compose используя OptionT

// Задача 4: Validated
case class ValidationError(errors: List[String])
// Validate form with multiple fields, collect all errors
```

**Вопросы:**

- Разница между Validated и Either?
- Что такое Monad Transformer? Зачем нужен?
- Объясните Tagless Final
- IO Monad vs Future?

---

### День 4-7: Akka / Akka Streams

**Темы:**

- Actor model
- Actor lifecycle
- Supervision strategies
- Message passing patterns
- Akka Streams (Source, Flow, Sink)
- Backpressure
- Graph DSL
- Akka HTTP basics

**Практика:**

```scala
// Задача 1: Simple Actor
class WorkerActor extends Actor {
  def receive: Receive = ???
}

// Задача 2: Supervision
// Реализовать supervisor с restart strategy

// Задача 3: Akka Streams
// CSV file processing pipeline
val source: Source[ByteString, Future[IOResult]] = FileIO.fromPath(path)
// Parse, transform, write to DB

// Задача 4: Backpressure
// Implement throttling stream

// Задача 5: Actor state machine
// Order processing: New -> Processing -> Completed/Failed
```

**Вопросы:**

- Actor model vs Thread-based concurrency?
- Что такое supervision?
- Как работает backpressure в Akka Streams?
- Actor selection vs Actor reference?

---

## 🏗️ Неделя 4: Архитектура и паттерны

### День 1-3: Design Patterns в Scala

**Темы:**

- Creational patterns (Factory, Builder, Singleton)
- Structural patterns (Adapter, Decorator, Proxy)
- Behavioral patterns (Strategy, Observer, Command)
- Functional patterns (Monads, Kleisli, Reader, Writer, State)
- Cake pattern (Dependency Injection)
- Type classes pattern
- Phantom types pattern

**Практика:**

```scala
// Задача 1: Builder pattern с типами
class QueryBuilder[T] private (
  private val table: String,
  private val where: Option[String],
  private val limit: Option[Int]
) {
  def where(condition: String): QueryBuilder[T] = ???
  def limit(n: Int): QueryBuilder[T] = ???
  def build(implicit ev: T =:= Complete): Query = ???
}

// Задача 2: Cake pattern
trait UserRepositoryComponent {
  val userRepository: UserRepository
  trait UserRepository {
    def find(id: String): Future[Option[User]]
  }
}

trait UserServiceComponent { this: UserRepositoryComponent =>
  val userService: UserService
  class UserService {
    def getUser(id: String): Future[User] = ???
  }
}

// Задача 3: Reader Monad
case class Config(dbUrl: String, apiKey: String)
type ConfigReader[A] = Reader[Config, A]

def getDbConnection: ConfigReader[Connection] = ???
def getApiClient: ConfigReader[ApiClient] = ???
```

**Вопросы:**

- Как реализовать Singleton в Scala?
- Преимущества Cake pattern?
- Reader Monad - когда использовать?
- Type-safe Builder - как это работает?

---

### День 4-7: Testing

**Темы:**

- ScalaTest / Specs2
- Property-based testing (ScalaCheck)
- Mocking (Mockito, ScalaMock)
- Akka TestKit
- Integration testing
- TDD approach

**Практика:**

```scala
// Задача 1: Unit tests
class UserServiceTest extends AnyFlatSpec with Matchers {
  "UserService" should "return user by id" in {
    ???
  }
  
  it should "handle missing users" in {
    ???
  }
}

// Задача 2: Property-based testing
property("reverse twice equals original") {
  forAll { (list: List[Int]) =>
    list.reverse.reverse shouldBe list
  }
}

// Задача 3: Async testing
"async operation" should "complete successfully" in {
  val future = service.asyncCall()
  future.map { result =>
    result shouldBe expected
  }
}

// Задача 4: Mock testing
"UserController" should "call repository" in {
  val mockRepo = mock[UserRepository]
  when(mockRepo.find("123")).thenReturn(Future.successful(Some(user)))
  // test
}
```

---

## 🗄️ Неделя 5: Базы данных и интеграции

### День 1-4: Database access

**Темы:**

- Slick (основы, queries, schema)
- Doobie (functional JDBC)
- Connection pooling (HikariCP)
- Transactions
- Quill (compile-time queries)
- Migration tools (Flyway, Liquibase)

**Практика:**

```scala
// Задача 1: Slick queries
class Users(tag: Tag) extends Table[User](tag, "users") {
  def id = column[Long]("id", O.PrimaryKey, O.AutoInc)
  def name = column[String]("name")
  def email = column[String]("email")
  def * = (id, name, email).mapTo[User]
}

// Complex join query
def getUsersWithOrders: DBIO[Seq[(User, Order)]] = ???

// Задача 2: Doobie
def findUser(id: Long): ConnectionIO[Option[User]] = 
  sql"SELECT id, name, email FROM users WHERE id = $id"
    .query[User]
    .option

// Задача 3: Transaction
def transferMoney(from: Long, to: Long, amount: Double): DBIO[Unit] = ???
```

**Вопросы:**

- Slick vs Doobie - когда что использовать?
- Как работают connection pools?
- N+1 query problem - как решить?
- Optimistic vs Pessimistic locking?

---

### День 5-7: Message Queues & Integration

**Темы:**

- Kafka (Producer, Consumer, Streams)
- RabbitMQ
- Redis integration
- HTTP clients (Akka HTTP, http4s)
- gRPC / Protobuf
- JSON (Circe, Play JSON)

**Практика:**

```scala
// Задача 1: Kafka Consumer
val settings = ConsumerSettings(system, new StringDeserializer, new StringDeserializer)
  .withBootstrapServers("localhost:9092")
  .withGroupId("group1")

Consumer
  .plainSource(settings, Subscriptions.topics("topic1"))
  .map(record => processRecord(record))
  .runWith(Sink.ignore)

// Задача 2: Circe JSON
case class User(id: Long, name: String, email: String)
implicit val userEncoder: Encoder[User] = ???
implicit val userDecoder: Decoder[User] = ???

// Задача 3: HTTP Client with retry
def callExternalApi(url: String): Future[Response] = ???
// Add retry with exponential backoff

// Задача 4: gRPC service
trait UserService {
  def getUser(request: GetUserRequest): Future[GetUserResponse]
}
```

---

## 🏛️ Неделя 6: System Design + Interview Prep

### День 1-3: System Design

**Темы для изучения:**

- Microservices architecture
- Event-driven architecture
- CQRS + Event Sourcing
- CAP theorem
- Distributed transactions (Saga pattern)
- Load balancing strategies
- Caching strategies
- API design (REST, GraphQL)

**Задачи для проектирования:**

#### 1. URL Shortener
- Requirements: 100M URLs/day, 10:1 read/write ratio
- Дизайн: Database schema, API, caching, sharding

#### 2. Rate Limiter
- Requirements: Per user, per API endpoint
- Algorithms: Token bucket, Sliding window
- Distributed implementation

#### 3. Real-time Chat System
- WebSockets, message delivery guarantees
- Presence service, read receipts
- Scaling to millions of users

#### 4. E-commerce Order System
- Inventory management
- Payment processing
- Order fulfillment
- Saga pattern для distributed transactions

#### 5. Metrics & Monitoring System
- Time-series data storage
- Aggregation, alerting
- Dashboard querying

---

### День 4-7: Mock Interviews

**Coding Practice (LeetCode/HackerRank):**

**Easy:**
- Two Sum
- Valid Parentheses
- Merge Two Sorted Lists
- Best Time to Buy and Sell Stock

**Medium:**
- Add Two Numbers (Linked List)
- Longest Substring Without Repeating Characters
- Group Anagrams
- Course Schedule (Graph)
- LRU Cache

**Hard:**
- Median of Two Sorted Arrays
- Trapping Rain Water
- Word Ladder
- Serialize and Deserialize Binary Tree

**Scala-specific tasks:**

```scala
// 1. Implement lazy infinite Stream of Fibonacci
def fibonacci: LazyList[BigInt] = ???

// 2. Functional LRU Cache
class LRUCache[K, V](capacity: Int) {
  def get(key: K): Option[V] = ???
  def put(key: K, value: V): LRUCache[K, V] = ???
}

// 3. Parse expression tree and evaluate
sealed trait Expr
case class Num(n: Int) extends Expr
case class Add(l: Expr, r: Expr) extends Expr
case class Mul(l: Expr, r: Expr) extends Expr

def eval(expr: Expr): Int = ???

// 4. Type-safe expression builder
// "2 + 3 * 4" должно компилироваться
// "2 + + 3" не должно компилироваться
```

---

## 📚 Ресурсы для изучения

### Книги (Must-read):

1. **"Functional Programming in Scala"** (Red Book) - Paul Chiusano, Rúnar Bjarnason
2. **"Scala with Cats"** - Noel Welsh, Dave Gurnell
3. **"Programming in Scala"** - Martin Odersky
4. **"Effective Scala"** - Joshua Suereth
5. **"Akka in Action"** - Raymond Roestenburg

### Online курсы:

- Coursera: Functional Programming Principles in Scala (Martin Odersky)
- Rock the JVM: Scala courses
- Udemy: Advanced Scala and Functional Programming

### Документация:

- Scala official docs
- Cats documentation
- Akka documentation
- Slick manual

### Practice:

- LeetCode (Scala solutions)
- HackerRank (Functional Programming track)
- Exercism (Scala track)
- Scala Exercises

---

## 🎤 Типичные вопросы на собеседовании

### Scala Basics

- Разница между val, var, def, lazy val?
- Что такое case class? Чем отличается от обычного class?
- Sealed trait - зачем нужен?
- Pattern matching - exhaustiveness checking?
- For-comprehension - как работает под капотом?

### Functional Programming

- Что такое монада? Приведите примеры
- Разница между map и flatMap?
- Чистые функции - что это? Преимущества?
- Immutability - зачем нужна?
- Tail recursion - как работает?

### Type System

- Variance - что это? +A, -A, invariant?
- Type classes - что это? Примеры использования?
- Higher-kinded types - объясните на примере
- Implicit resolution - как работает?
- Path-dependent types - когда использовать?

### Concurrency

- Future vs Promise - разница?
- ExecutionContext - что это?
- Blocking operations - почему опасны?
- Actor model - преимущества?
- Backpressure - что это?

### Architecture

- Microservices - когда использовать?
- Event-driven architecture - плюсы/минусы?
- CQRS - что это? Когда применять?
- Saga pattern - объясните
- CAP theorem - что выбрать?

### Performance

- Tail call optimization - как работает?
- View - когда использовать?
- Vector vs List - performance?
- Memory leaks в Scala - возможны ли?
- JVM tuning - основные параметры?

---

## ✅ Checklist перед собеседованием

### За неделю до:

- ✅ Повторить основы Scala
- ✅ Решить 10-15 задач на LeetCode
- ✅ Просмотреть свои проекты на GitHub
- ✅ Подготовить примеры из опыта (STAR method)

### За день до:

- ✅ Хороший сон (8 часов)
- ✅ Проверить оборудование (камера, микрофон)
- ✅ Подготовить вопросы интервьюеру
- ✅ Повторить основные концепции

### В день собеседования:

- ✅ Прийти за 10 минут до начала
- ✅ Иметь под рукой: бумагу, ручку, воду
- ✅ Спокойствие и уверенность
- ✅ Думать вслух во время решения задач

---

## 💡 Советы

### Во время coding interview:

1. **Уточните требования** - не спешите сразу писать код
2. **Обсудите подход** - объясните план решения
3. **Думайте вслух** - интервьюер должен понимать ваш ход мыслей
4. **Начните с простого** - потом оптимизируйте
5. **Тестируйте** - придумайте test cases
6. **Обсудите сложность** - O-notation для времени и памяти

### Во время system design:

1. **Clarify requirements** - functional and non-functional
2. **Начните с high-level** - потом углубляйтесь
3. **Обсуждайте trade-offs** - нет идеального решения
4. **Масштабируемость** - думайте о росте
5. **Рисуйте диаграммы** - визуализация помогает

### Behavioral questions (STAR method):

- **Situation**: Опишите контекст
- **Task**: Какая была задача?
- **Action**: Что вы сделали?
- **Result**: Каков результат?

**Примеры вопросов:**

- Расскажите о сложном проекте
- Конфликт в команде - как решали?
- Ошибка в production - ваши действия?
- Технический долг - как боролись?

---

## 🎯 Финальный чек-лист навыков Senior Scala Developer

### Must-have (обязательно):

- ✅ Scala syntax и идиомы
- ✅ Functional programming (монады, type classes)
- ✅ Concurrency (Future, Actor model)
- ✅ Collections (performance, использование)
- ✅ Type system (variance, HKT, type classes)
- ✅ Testing (unit, integration, property-based)
- ✅ Database access (Slick/Doobie)
- ✅ Build tools (SBT)

### Nice-to-have (желательно):

- ✅ Cats/Scalaz
- ✅ Akka Streams
- ✅ Kafka/RabbitMQ
- ✅ gRPC/Protobuf
- ✅ Docker/Kubernetes basics
- ✅ CI/CD pipeline
- ✅ Monitoring (Prometheus, Grafana)

### Senior-level:

- ✅ System design
- ✅ Architecture decisions
- ✅ Performance optimization
- ✅ Mentoring experience
- ✅ Code review skills
- ✅ Production debugging
- ✅ Technical leadership

---

