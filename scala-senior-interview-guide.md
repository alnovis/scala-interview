# Полное Руководство по Подготовке к Интервью Senior Scala Developer

## Оглавление

1. [План подготовки](#план-подготовки)
2. [Основы Scala (Advanced Level)](#1-основы-scala-advanced-level)
3. [Функциональное программирование](#2-функциональное-программирование)
4. [Type System и Type Classes](#3-type-system-и-type-classes)
5. [Concurrency и Параллелизм](#4-concurrency-и-параллелизм)
6. [Scala 3 (Dotty) - Новые возможности](#5-scala-3-dotty---новые-возможности)
7. [Effect Systems (Cats Effect, ZIO)](#6-effect-systems-cats-effect-zio)
8. [Streaming и Reactive Programming](#7-streaming-и-reactive-programming)
9. [Архитектура и Design Patterns](#8-архитектура-и-design-patterns)
10. [Testing Strategies](#9-testing-strategies)
11. [Performance и Optimization](#10-performance-и-optimization)
12. [System Design](#11-system-design)
13. [Behavioral Questions](#12-behavioral-questions)

---

## План подготовки

### Недели 1-2: Фундаментальные концепции
- Глубокое изучение type system
- Функциональное программирование (монады, функторы, аппликативы)
- Implicits и Context Parameters (Scala 3)
- Pattern matching и ADT

### Недели 3-4: Продвинутые темы
- Effect systems (Cats Effect 3.x / ZIO 2.x)
- Concurrency models
- Streaming (fs2, Akka Streams)
- Type classes и их имплементация

### Недели 5-6: Фреймворки и практика
- Akka/Pekko
- HTTP4s / ZIO HTTP
- Tapir для API
- Doobie / Quill для БД

### Недели 7-8: System Design и практика
- Распределенные системы
- Event Sourcing / CQRS
- Микросервисная архитектура
- Code review и рефакторинг

---

## 1. Основы Scala (Advanced Level)

### 1.1 Implicits и Context Parameters

#### Теория
**Implicits** (Scala 2) и **Given/Using** (Scala 3) - механизм для передачи параметров неявно.

**Типы implicits:**
1. **Implicit parameters** - неявные параметры
2. **Implicit conversions** - неявные преобразования
3. **Implicit classes** - расширение функциональности (extension methods)

#### Примеры Scala 2

```scala
// Implicit parameters
trait Ordering[T] {
  def compare(x: T, y: T): Int
}

implicit val intOrdering: Ordering[Int] = new Ordering[Int] {
  def compare(x: Int, y: Int): Int = x - y
}

def max[T](a: T, b: T)(implicit ord: Ordering[T]): T =
  if (ord.compare(a, b) > 0) a else b

max(5, 10) // intOrdering передается неявно
```

```scala
// Implicit conversions (использовать осторожно!)
implicit class StringOps(s: String) {
  def toSnakeCase: String = 
    s.replaceAll("([A-Z])", "_$1").toLowerCase
}

"HelloWorld".toSnakeCase // "_hello_world"
```

```scala
// Type classes pattern
trait JsonEncoder[A] {
  def encode(value: A): String
}

implicit val intEncoder: JsonEncoder[Int] = 
  (value: Int) => value.toString

implicit val stringEncoder: JsonEncoder[String] = 
  (value: String) => s""""$value""""

def toJson[A](value: A)(implicit encoder: JsonEncoder[A]): String =
  encoder.encode(value)

// Автоматическая деривация для case classes
implicit def optionEncoder[A](implicit enc: JsonEncoder[A]): JsonEncoder[Option[A]] = {
  case Some(value) => enc.encode(value)
  case None => "null"
}
```

#### Примеры Scala 3

```scala
// Given/Using синтаксис
given Ordering[Int] with
  def compare(x: Int, y: Int): Int = x - y

def max[T](a: T, b: T)(using ord: Ordering[T]): T =
  if ord.compare(a, b) > 0 then a else b

// Extension methods
extension (s: String)
  def toSnakeCase: String = 
    s.replaceAll("([A-Z])", "_$1").toLowerCase

// Context bounds
def sort[T: Ordering](list: List[T]): List[T] = 
  list.sorted
```

### 1.2 Variance (Ковариантность и Контравариантность)

#### Теория

```
Invariance:     C[T]         - не может заменяться
Covariance:     C[+T]        - если B <: A, то C[B] <: C[A]
Contravariance: C[-T]        - если B <: A, то C[A] <: C[B]
```

#### Диаграмма

```
         Animal
         /    \
       Dog    Cat
       
Covariant List[+A]:
  List[Dog] <: List[Animal]  ✓
  
Contravariant Function[-A, +B]:
  Animal => Dog   <:  Dog => Animal  ✓
```

#### Примеры

```scala
// Covariance - producers
class Box[+A](val item: A) {
  // def set(newItem: A): Unit = ??? // ОШИБКА! A в contravariant position
  
  def get: A = item // OK
}

abstract class Animal
case class Dog() extends Animal
case class Cat() extends Animal

val dogBox: Box[Dog] = new Box(Dog())
val animalBox: Box[Animal] = dogBox // OK благодаря ковариантности

// Contravariance - consumers
trait Serializer[-A] {
  def serialize(a: A): String
}

val animalSerializer: Serializer[Animal] = 
  (a: Animal) => s"Animal"

val dogSerializer: Serializer[Dog] = animalSerializer // OK

// Function variance
// Function1[-T, +R] - контравариантна по входу, ковариантна по выходу
val f: Animal => Dog = (a: Animal) => Dog()
val g: Dog => Animal = f // OK
```

### 1.3 Path-Dependent Types

#### Теория
Path-dependent types позволяют типам зависеть от конкретных экземпляров объектов.

```scala
class Outer {
  class Inner
  def process(inner: Inner): Unit = println("Processing")
}

val outer1 = new Outer
val outer2 = new Outer

val inner1: outer1.Inner = new outer1.Inner
val inner2: outer2.Inner = new outer2.Inner

// outer1.process(inner2) // ОШИБКА! Разные типы
outer1.process(inner1) // OK
```

#### Практическое применение

```scala
// Type-safe Builder pattern
trait Repository {
  type Entity
  def save(entity: Entity): Unit
  def findAll: List[Entity]
}

class UserRepository extends Repository {
  case class User(id: Long, name: String)
  type Entity = User
  
  private var users: List[User] = Nil
  
  def save(entity: User): Unit = users = entity :: users
  def findAll: List[User] = users
}

def processRepo(repo: Repository)(entities: List[repo.Entity]): Unit = {
  entities.foreach(repo.save)
}
```

### 1.4 Self Types и Cake Pattern

#### Теория
Self types определяют зависимости между traits без использования наследования.

```scala
// Self type
trait UserRepository {
  def findUser(id: Long): Option[User]
}

trait EmailService {
  self: UserRepository => // требует UserRepository
  
  def sendEmail(userId: Long, message: String): Unit = {
    findUser(userId).foreach { user =>
      println(s"Sending to ${user.email}: $message")
    }
  }
}

case class User(id: Long, email: String)

// Использование
class UserApp extends UserRepository with EmailService {
  private val users = Map(1L -> User(1, "user@example.com"))
  
  def findUser(id: Long): Option[User] = users.get(id)
}
```

#### Cake Pattern для DI

```scala
// Components
trait DatabaseComponent {
  val database: Database
  
  trait Database {
    def query(sql: String): List[String]
  }
}

trait UserServiceComponent {
  self: DatabaseComponent =>
  
  val userService: UserService
  
  class UserService {
    def getUser(id: Long): Option[String] = {
      database.query(s"SELECT * FROM users WHERE id = $id").headOption
    }
  }
}

// Production implementation
trait ProductionDatabaseComponent extends DatabaseComponent {
  val database = new Database {
    def query(sql: String): List[String] = {
      println(s"Executing: $sql")
      List("user1", "user2")
    }
  }
}

// Assembly
object ProductionApp 
  extends ProductionDatabaseComponent 
  with UserServiceComponent {
  
  val userService = new UserService
}
```

### 1.5 Higher-Kinded Types (HKT)

#### Теория
HKT - типы, которые принимают другие типы с параметрами.

```
Type:           Int, String
Type Constructor: List[_], Option[_]
Higher-Kinded:  F[_], G[_[_]]
```

#### Примеры

```scala
// Простой пример
trait Container[F[_]] {
  def wrap[A](value: A): F[A]
}

implicit val listContainer: Container[List] = new Container[List] {
  def wrap[A](value: A): List[A] = List(value)
}

implicit val optionContainer: Container[Option] = new Container[Option] {
  def wrap[A](value: A): Option[A] = Some(value)
}

def wrapValue[F[_], A](value: A)(implicit container: Container[F]): F[A] =
  container.wrap(value)

wrapValue[List, Int](42)    // List(42)
wrapValue[Option, String]("hello") // Some("hello")
```

```scala
// Functor - классический пример HKT
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

implicit val listFunctor: Functor[List] = new Functor[List] {
  def map[A, B](fa: List[A])(f: A => B): List[B] = fa.map(f)
}

implicit val optionFunctor: Functor[Option] = new Functor[Option] {
  def map[A, B](fa: Option[A])(f: A => B): Option[B] = fa.map(f)
}

def increment[F[_]: Functor](fa: F[Int]): F[Int] = {
  val F = implicitly[Functor[F]]
  F.map(fa)(_ + 1)
}

increment(List(1, 2, 3))      // List(2, 3, 4)
increment(Some(5))            // Some(6)
```

---

## 2. Функциональное программирование

### 2.1 Монады (Monads)

#### Теория
Монада - это паттерн для последовательного выполнения операций с контекстом.

**Законы монад:**
1. **Left identity**: `pure(a).flatMap(f) == f(a)`
2. **Right identity**: `m.flatMap(pure) == m`
3. **Associativity**: `m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g))`

```
        pure: A => F[A]
        flatMap: F[A] => (A => F[B]) => F[B]
```

#### Диаграмма

```
Regular function composition:
  A --f--> B --g--> C
  
Monadic composition:
  F[A] --flatMap(f)--> F[B] --flatMap(g)--> F[C]
  
where f: A => F[B]
      g: B => F[C]
```

#### Примеры

```scala
// Определение Monad
trait Monad[F[_]] extends Functor[F] {
  def pure[A](a: A): F[A]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  
  // Derived from flatMap
  def map[A, B](fa: F[A])(f: A => B): F[B] =
    flatMap(fa)(a => pure(f(a)))
}

// Option Monad
implicit val optionMonad: Monad[Option] = new Monad[Option] {
  def pure[A](a: A): Option[A] = Some(a)
  def flatMap[A, B](fa: Option[A])(f: A => Option[B]): Option[B] = 
    fa.flatMap(f)
}

// List Monad
implicit val listMonad: Monad[List] = new Monad[List] {
  def pure[A](a: A): List[A] = List(a)
  def flatMap[A, B](fa: List[A])(f: A => List[B]): List[B] = 
    fa.flatMap(f)
}

// Either Monad
implicit def eitherMonad[E]: Monad[Either[E, *]] = 
  new Monad[Either[E, *]] {
    def pure[A](a: A): Either[E, A] = Right(a)
    def flatMap[A, B](fa: Either[E, A])(f: A => Either[E, B]): Either[E, B] =
      fa.flatMap(f)
  }
```

#### Практический пример - композиция операций

```scala
case class User(id: Long, name: String, email: String)
case class Order(id: Long, userId: Long, amount: Double)
case class Payment(orderId: Long, status: String)

def findUser(id: Long): Option[User] = 
  if (id > 0) Some(User(id, "John", "john@example.com")) else None

def findOrders(userId: Long): Option[List[Order]] =
  Some(List(Order(1, userId, 100.0), Order(2, userId, 200.0)))

def processPayment(order: Order): Option[Payment] =
  if (order.amount > 0) Some(Payment(order.id, "completed")) else None

// Monadic composition
def processUserOrders(userId: Long): Option[List[Payment]] = {
  for {
    user <- findUser(userId)
    orders <- findOrders(user.id)
    payments <- orders.traverse(processPayment) // traverse from Cats
  } yield payments
}

// Без монад (императивный стиль)
def processUserOrdersImperative(userId: Long): Option[List[Payment]] = {
  val userOpt = findUser(userId)
  if (userOpt.isEmpty) return None
  
  val ordersOpt = findOrders(userOpt.get.id)
  if (ordersOpt.isEmpty) return None
  
  val payments = ordersOpt.get.map { order =>
    val payment = processPayment(order)
    if (payment.isEmpty) return None
    payment.get
  }
  
  Some(payments)
}
```

### 2.2 Applicative Functors

#### Теория
Applicative позволяет комбинировать независимые эффекты.

```scala
trait Applicative[F[_]] extends Functor[F] {
  def pure[A](a: A): F[A]
  def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
  
  // Derived
  def map2[A, B, C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C] =
    ap(map(fa)(a => (b: B) => f(a, b)))(fb)
}
```

#### Отличие от Monad

```
Monad:       зависимые вычисления
             fa.flatMap(a => fb(a))
             
Applicative: независимые вычисления  
             (fa, fb).mapN((a, b) => f(a, b))
```

#### Примеры

```scala
import cats.syntax.apply._
import cats.instances.option._

case class Person(name: String, age: Int, email: String)

val nameOpt: Option[String] = Some("Alice")
val ageOpt: Option[Int] = Some(30)
val emailOpt: Option[String] = Some("alice@example.com")

// С Applicative (независимые)
val personOpt: Option[Person] = 
  (nameOpt, ageOpt, emailOpt).mapN(Person.apply)

// С Monad (зависимые - более мощно, но менее эффективно)
val personMonadic: Option[Person] = for {
  name <- nameOpt
  age <- ageOpt
  email <- emailOpt
} yield Person(name, age, email)
```

#### Validation с помощью Applicative

```scala
import cats.data.ValidatedNec
import cats.syntax.validated._
import cats.syntax.apply._

type ValidationResult[A] = ValidatedNec[String, A]

def validateName(name: String): ValidationResult[String] =
  if (name.nonEmpty) name.validNec 
  else "Name cannot be empty".invalidNec

def validateAge(age: Int): ValidationResult[Int] =
  if (age >= 18) age.validNec 
  else "Age must be >= 18".invalidNec

def validateEmail(email: String): ValidationResult[String] =
  if (email.contains("@")) email.validNec 
  else "Invalid email".invalidNec

// Собираем все ошибки сразу (не останавливаемся на первой)
def validatePerson(name: String, age: Int, email: String): ValidationResult[Person] =
  (validateName(name), validateAge(age), validateEmail(email))
    .mapN(Person.apply)

// Использование
validatePerson("", 15, "invalid")
// Invalid(NonEmptyChain("Name cannot be empty", "Age must be >= 18", "Invalid email"))

validatePerson("Alice", 30, "alice@example.com")
// Valid(Person("Alice", 30, "alice@example.com"))
```

### 2.3 Monoid и Semigroup

#### Теория

**Semigroup** - бинарная ассоциативная операция:
- `combine(a, combine(b, c)) == combine(combine(a, b), c)`

**Monoid** - Semigroup с единичным элементом:
- `combine(a, empty) == a`
- `combine(empty, a) == a`

```scala
trait Semigroup[A] {
  def combine(x: A, y: A): A
}

trait Monoid[A] extends Semigroup[A] {
  def empty: A
}
```

#### Примеры

```scala
// Int Monoid
implicit val intAdditionMonoid: Monoid[Int] = new Monoid[Int] {
  def empty: Int = 0
  def combine(x: Int, y: Int): Int = x + y
}

implicit val intMultiplicationMonoid: Monoid[Int] = new Monoid[Int] {
  def empty: Int = 1
  def combine(x: Int, y: Int): Int = x * y
}

// String Monoid
implicit val stringMonoid: Monoid[String] = new Monoid[String] {
  def empty: String = ""
  def combine(x: String, y: String): String = x + y
}

// List Monoid
implicit def listMonoid[A]: Monoid[List[A]] = new Monoid[List[A]] {
  def empty: List[A] = Nil
  def combine(x: List[A], y: List[A]): List[A] = x ++ y
}

// Option Monoid (если A имеет Semigroup)
implicit def optionMonoid[A: Semigroup]: Monoid[Option[A]] = 
  new Monoid[Option[A]] {
    def empty: Option[A] = None
    def combine(x: Option[A], y: Option[A]): Option[A] = (x, y) match {
      case (Some(a), Some(b)) => Some(Semigroup[A].combine(a, b))
      case (Some(a), None) => Some(a)
      case (None, Some(b)) => Some(b)
      case (None, None) => None
    }
  }
```

#### Практическое применение

```scala
// Fold с Monoid
def combineAll[A: Monoid](list: List[A]): A = {
  val M = implicitly[Monoid[A]]
  list.foldLeft(M.empty)(M.combine)
}

combineAll(List(1, 2, 3, 4, 5))           // 15
combineAll(List("Hello", " ", "World"))    // "Hello World"
combineAll(List(List(1, 2), List(3, 4)))  // List(1, 2, 3, 4)

// MapReduce pattern
def parallelFold[A: Monoid](list: List[A], parallelism: Int): A = {
  val M = implicitly[Monoid[A]]
  list
    .grouped(list.size / parallelism + 1)
    .toList
    .par
    .map(chunk => chunk.foldLeft(M.empty)(M.combine))
    .foldLeft(M.empty)(M.combine)
}

// Агрегация сложных структур
case class Stats(count: Int, sum: Double, min: Double, max: Double)

implicit val statsMonoid: Monoid[Stats] = new Monoid[Stats] {
  def empty: Stats = Stats(0, 0.0, Double.MaxValue, Double.MinValue)
  
  def combine(x: Stats, y: Stats): Stats = Stats(
    count = x.count + y.count,
    sum = x.sum + y.sum,
    min = math.min(x.min, y.min),
    max = math.max(x.max, y.max)
  )
}

def analyze(values: List[Double]): Stats = {
  combineAll(values.map(v => Stats(1, v, v, v)))
}
```

### 2.4 Functor и его виды

#### Covariant Functor

```scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

// Законы:
// 1. Identity: fa.map(identity) == fa
// 2. Composition: fa.map(f).map(g) == fa.map(f andThen g)
```

#### Contravariant Functor

```scala
trait Contravariant[F[_]] {
  def contramap[A, B](fa: F[A])(f: B => A): F[B]
}

// Пример: Encoder
trait Encoder[A] {
  def encode(value: A): String
}

implicit val intEncoder: Encoder[Int] = (value: Int) => value.toString

implicit val contravariantEncoder: Contravariant[Encoder] = 
  new Contravariant[Encoder] {
    def contramap[A, B](fa: Encoder[A])(f: B => A): Encoder[B] =
      (value: B) => fa.encode(f(value))
  }

case class Person(name: String, age: Int)

implicit val personEncoder: Encoder[Person] = 
  contravariantEncoder.contramap[Int, Person](intEncoder)(_.age)
```

#### Invariant Functor

```scala
trait Invariant[F[_]] {
  def imap[A, B](fa: F[A])(f: A => B)(g: B => A): F[B]
}

// Пример: Codec (может и кодировать и декодировать)
trait Codec[A] {
  def encode(value: A): String
  def decode(value: String): A
}
```

### 2.5 Free Monads

#### Теория
Free Monad позволяет разделить описание программы от её интерпретации.

```scala
// Простая Free Monad
sealed trait Free[F[_], A]
case class Pure[F[_], A](a: A) extends Free[F, A]
case class FlatMap[F[_], A, B](fa: Free[F, A], f: A => Free[F, B]) 
  extends Free[F, B]
case class Suspend[F[_], A](fa: F[A]) extends Free[F, A]

// DSL для CRUD операций
sealed trait CrudOp[A]
case class Create(item: String) extends CrudOp[Long]
case class Read(id: Long) extends CrudOp[Option[String]]
case class Update(id: Long, item: String) extends CrudOp[Boolean]
case class Delete(id: Long) extends CrudOp[Boolean]

type CrudIO[A] = Free[CrudOp, A]

// Smart constructors
def create(item: String): CrudIO[Long] = 
  Suspend(Create(item))

def read(id: Long): CrudIO[Option[String]] = 
  Suspend(Read(id))

// Программа
def program: CrudIO[Option[String]] = for {
  id <- create("item")
  item <- read(id)
} yield item

// Интерпретатор для тестирования
def testInterpreter: CrudOp ~> Id = new (CrudOp ~> Id) {
  private var storage: Map[Long, String] = Map.empty
  private var nextId: Long = 1L
  
  def apply[A](fa: CrudOp[A]): A = fa match {
    case Create(item) =>
      val id = nextId
      nextId += 1
      storage = storage + (id -> item)
      id
    case Read(id) =>
      storage.get(id)
    case Update(id, item) =>
      if (storage.contains(id)) {
        storage = storage + (id -> item)
        true
      } else false
    case Delete(id) =>
      if (storage.contains(id)) {
        storage = storage - id
        true
      } else false
  }
}
```

---

## 3. Type System и Type Classes

### 3.1 Type Classes Deep Dive

#### Теория
Type class - это паттерн для ad-hoc полиморфизма.

**Компоненты:**
1. Type class trait
2. Type class instances
3. Interface methods/syntax

```
┌─────────────────────────┐
│   Type Class Trait      │
│   trait Show[A]         │
└───────────┬─────────────┘
            │
            │ implements
            │
┌───────────┴─────────────┐
│  Type Class Instance     │
│  implicit val intShow    │
└───────────┬─────────────┘
            │
            │ uses
            │
┌───────────┴─────────────┐
│  Interface/Syntax        │
│  def show[A: Show](a: A) │
└─────────────────────────┘
```

#### Полный пример

```scala
// 1. Type class definition
trait Show[A] {
  def show(a: A): String
}

object Show {
  // 2. Summoner method
  def apply[A](implicit instance: Show[A]): Show[A] = instance
  
  // 3. Constructor method
  def instance[A](f: A => String): Show[A] = new Show[A] {
    def show(a: A): String = f(a)
  }
  
  // 4. Interface syntax
  implicit class ShowOps[A](val a: A) extends AnyVal {
    def show(implicit s: Show[A]): String = s.show(a)
  }
}

// 5. Type class instances
implicit val intShow: Show[Int] = Show.instance(_.toString)
implicit val stringShow: Show[String] = Show.instance(identity)
implicit val booleanShow: Show[Boolean] = Show.instance(_.toString)

// 6. Derived instances
implicit def listShow[A](implicit s: Show[A]): Show[List[A]] = 
  Show.instance(list => list.map(s.show).mkString("[", ", ", "]"))

implicit def optionShow[A](implicit s: Show[A]): Show[Option[A]] = 
  Show.instance {
    case Some(a) => s"Some(${s.show(a)})"
    case None => "None"
  }

// Использование
import Show._

42.show                           // "42"
List(1, 2, 3).show               // "[1, 2, 3]"
Some("hello").show               // "Some(hello)"
```

### 3.2 Type Class Derivation

#### Теория
Автоматическая генерация type class instances для ADT.

#### Shapeless (Scala 2)

```scala
import shapeless._

// Generic derivation
trait Encoder[A] {
  def encode(a: A): Map[String, String]
}

object Encoder {
  def apply[A](implicit enc: Encoder[A]): Encoder[A] = enc
  
  def instance[A](f: A => Map[String, String]): Encoder[A] =
    (a: A) => f(a)
  
  // Base instances
  implicit val stringEncoder: Encoder[String] = 
    instance(s => Map("value" -> s))
  
  implicit val intEncoder: Encoder[Int] = 
    instance(i => Map("value" -> i.toString))
  
  // HList encoder
  implicit val hnilEncoder: Encoder[HNil] = 
    instance(_ => Map.empty)
  
  implicit def hconsEncoder[H, T <: HList](
    implicit
    hEncoder: Lazy[Encoder[H]],
    tEncoder: Encoder[T]
  ): Encoder[H :: T] = instance {
    case h :: t => 
      hEncoder.value.encode(h) ++ tEncoder.encode(t)
  }
  
  // Generic derivation
  implicit def genericEncoder[A, R](
    implicit
    gen: Generic.Aux[A, R],
    enc: Lazy[Encoder[R]]
  ): Encoder[A] = instance { a =>
    enc.value.encode(gen.to(a))
  }
}

case class Person(name: String, age: Int)

// Автоматически выводится!
val encoder = Encoder[Person]
encoder.encode(Person("Alice", 30))
// Map("value" -> "Alice", "value" -> "30")
```

#### Scala 3 - Derives Clause

```scala
// Scala 3 - встроенная деривация
import scala.deriving.*

trait Encoder[T] {
  def encode(t: T): String
}

object Encoder {
  given Encoder[Int] = _.toString
  given Encoder[String] = s => s""""$s""""
  
  inline def derived[T](using m: Mirror.Of[T]): Encoder[T] = {
    val elemInstances = summonAll[m.MirroredElemTypes]
    inline m match {
      case s: Mirror.SumOf[T] => // enum/sealed trait
        sumEncoder(s, elemInstances)
      case p: Mirror.ProductOf[T] => // case class
        productEncoder(p, elemInstances)
    }
  }
}

// Автоматическая деривация
case class Person(name: String, age: Int) derives Encoder

// Использование
val encoder = summon[Encoder[Person]]
```

### 3.3 Phantom Types

#### Теория
Phantom types используются только на уровне компиляции для type-safety.

```scala
sealed trait ConnectionState
trait Connected extends ConnectionState
trait Disconnected extends ConnectionState

class Connection[State <: ConnectionState] private (url: String) {
  def connect()(implicit ev: State =:= Disconnected): Connection[Connected] =
    new Connection[Connected](url)
  
  def disconnect()(implicit ev: State =:= Connected): Connection[Disconnected] =
    new Connection[Disconnected](url)
  
  def query(sql: String)(implicit ev: State =:= Connected): List[String] = {
    println(s"Executing: $sql")
    List("result")
  }
}

object Connection {
  def create(url: String): Connection[Disconnected] =
    new Connection[Disconnected](url)
}

// Использование
val conn1 = Connection.create("jdbc://...")
// conn1.query("SELECT ...") // ОШИБКА КОМПИЛЯЦИИ!

val conn2 = conn1.connect()
conn2.query("SELECT ...") // OK

val conn3 = conn2.disconnect()
// conn3.query("SELECT ...") // ОШИБКА КОМПИЛЯЦИИ!
```

#### Type-safe Builder

```scala
sealed trait BuilderState
trait Empty extends BuilderState
trait WithName extends BuilderState
trait WithAge extends BuilderState
trait Complete extends BuilderState

class PersonBuilder[State <: BuilderState] private (
  name: Option[String] = None,
  age: Option[Int] = None
) {
  def withName(n: String)(implicit ev: State =:= Empty): PersonBuilder[WithName] =
    new PersonBuilder[WithName](Some(n), age)
  
  def withAge(a: Int)(implicit ev: State =:= WithName): PersonBuilder[Complete] =
    new PersonBuilder[Complete](name, Some(a))
  
  def build()(implicit ev: State =:= Complete): Person =
    Person(name.get, age.get)
}

object PersonBuilder {
  def apply(): PersonBuilder[Empty] = new PersonBuilder[Empty]()
}

// Использование
val person = PersonBuilder()
  .withName("Alice")
  .withAge(30)
  .build()

// PersonBuilder().withAge(30) // ОШИБКА КОМПИЛЯЦИИ!
// PersonBuilder().withName("Bob").build() // ОШИБКА КОМПИЛЯЦИИ!
```

### 3.4 Type-Level Programming

#### Теория
Вычисления на уровне типов для compile-time проверок.

```scala
// Type-level натуральные числа
sealed trait Nat
sealed trait Zero extends Nat
sealed trait Succ[N <: Nat] extends Nat

type _0 = Zero
type _1 = Succ[_0]
type _2 = Succ[_1]
type _3 = Succ[_2]

// Type-level сложение
trait Sum[A <: Nat, B <: Nat] {
  type Result <: Nat
}

object Sum {
  type Aux[A <: Nat, B <: Nat, R <: Nat] = Sum[A, B] { type Result = R }
  
  implicit def zeroSum[B <: Nat]: Aux[Zero, B, B] = 
    new Sum[Zero, B] { type Result = B }
  
  implicit def succSum[A <: Nat, B <: Nat, R <: Nat](
    implicit sum: Aux[A, B, R]
  ): Aux[Succ[A], B, Succ[R]] = 
    new Sum[Succ[A], B] { type Result = Succ[R] }
}

// Type-level Vector с размером
case class Vec[N <: Nat, A](elements: List[A])

object Vec {
  def apply[A](elements: A*): Vec[Succ[Zero], A] = 
    Vec[Succ[Zero], A](elements.toList)
  
  def concat[N1 <: Nat, N2 <: Nat, A, R <: Nat](
    v1: Vec[N1, A],
    v2: Vec[N2, A]
  )(implicit sum: Sum.Aux[N1, N2, R]): Vec[R, A] =
    Vec[R, A](v1.elements ++ v2.elements)
}
```

#### Heterogeneous Lists (HList)

```scala
// Simplified HList
sealed trait HList
case object HNil extends HList
case class ::[+H, +T <: HList](head: H, tail: T) extends HList

// Type-safe operations
object HList {
  // Prepend
  def ::[A](a: A): A :: HNil = ::(a, HNil)
  
  // Get by type
  trait Selector[L <: HList, A] {
    def apply(l: L): A
  }
  
  implicit def headSelector[H, T <: HList]: Selector[H :: T, H] =
    (l: H :: T) => l.head
  
  implicit def tailSelector[H, T <: HList, A](
    implicit st: Selector[T, A]
  ): Selector[H :: T, A] =
    (l: H :: T) => st(l.tail)
}

// Использование
val hlist = "hello" :: 42 :: true :: HNil
val str: String = hlist.select[String]   // "hello"
val int: Int = hlist.select[Int]         // 42
val bool: Boolean = hlist.select[Boolean] // true
```

---

## 4. Concurrency и Параллелизм

### 4.1 Основы JVM Concurrency

#### Thread и Runnable

```scala
// Создание потоков
val thread1 = new Thread(new Runnable {
  def run(): Unit = println("Thread 1")
})

val thread2 = new Thread(() => println("Thread 2"))

thread1.start()
thread2.start()

thread1.join() // ждем завершения
thread2.join()
```

#### Synchronized и Locks

```scala
class Counter {
  private var count: Int = 0
  
  // Synchronized method
  def increment(): Unit = synchronized {
    count += 1
  }
  
  def get: Int = synchronized {
    count
  }
}

// ReentrantLock
import java.util.concurrent.locks.ReentrantLock

class CounterWithLock {
  private var count: Int = 0
  private val lock = new ReentrantLock()
  
  def increment(): Unit = {
    lock.lock()
    try {
      count += 1
    } finally {
      lock.unlock()
    }
  }
}
```

#### Проблемы concurrency

```scala
// Race condition
class BrokenCounter {
  private var count: Int = 0
  
  def increment(): Unit = {
    val temp = count  // read
    // другой поток может изменить count здесь!
    count = temp + 1  // write
  }
}

// Deadlock
class DeadlockExample {
  val lock1 = new Object
  val lock2 = new Object
  
  def method1(): Unit = {
    lock1.synchronized {
      Thread.sleep(100)
      lock2.synchronized {
        println("Method 1")
      }
    }
  }
  
  def method2(): Unit = {
    lock2.synchronized {
      Thread.sleep(100)
      lock1.synchronized { // Deadlock!
        println("Method 2")
      }
    }
  }
}
```

### 4.2 Futures и Promises

#### Future

```scala
import scala.concurrent._
import scala.concurrent.duration._
import ExecutionContext.Implicits.global

// Создание Future
val future1: Future[Int] = Future {
  Thread.sleep(1000)
  42
}

// Callbacks
future1.onComplete {
  case Success(value) => println(s"Result: $value")
  case Failure(ex) => println(s"Error: ${ex.getMessage}")
}

// Комбинирование
val future2: Future[Int] = future1.map(_ * 2)
val future3: Future[String] = future2.map(_.toString)

// For-comprehension
val combinedFuture: Future[Int] = for {
  a <- Future(10)
  b <- Future(20)
  c <- Future(30)
} yield a + b + c

// Параллельное выполнение
def fetchUser(id: Long): Future[User] = ???
def fetchOrders(userId: Long): Future[List[Order]] = ???
def fetchPayments(orderId: Long): Future[Payment] = ???

// Последовательно (медленно)
val sequential: Future[(User, List[Order])] = for {
  user <- fetchUser(1L)
  orders <- fetchOrders(user.id)
} yield (user, orders)

// Параллельно (быстро)
val parallel: Future[(User, List[Order])] = {
  val userFuture = fetchUser(1L)
  val ordersFuture = fetchOrders(1L)
  
  for {
    user <- userFuture
    orders <- ordersFuture
  } yield (user, orders)
}

// Future.sequence - ждем все Future
val futures: List[Future[Int]] = List(
  Future(1),
  Future(2),
  Future(3)
)

val allResults: Future[List[Int]] = Future.sequence(futures)
// Future(List(1, 2, 3))

// Future.traverse
val ids = List(1L, 2L, 3L)
val users: Future[List[User]] = Future.traverse(ids)(fetchUser)
```

#### Promise

```scala
import scala.concurrent.Promise

// Promise - ручное управление Future
val promise = Promise[Int]()
val future: Future[Int] = promise.future

// В другом потоке завершаем Promise
Future {
  Thread.sleep(1000)
  promise.success(42)
  // или promise.failure(new Exception("Error"))
}

future.onComplete {
  case Success(v) => println(s"Got: $v")
  case Failure(e) => println(s"Failed: $e")
}
```

#### Продвинутые паттерны

```scala
// Retry logic
def retry[T](times: Int)(f: => Future[T]): Future[T] = {
  f.recoverWith {
    case _ if times > 0 => retry(times - 1)(f)
  }
}

// Timeout
def withTimeout[T](future: Future[T], duration: FiniteDuration): Future[T] = {
  val timeoutPromise = Promise[T]()
  
  val timer = new java.util.Timer()
  timer.schedule(
    new java.util.TimerTask {
      def run(): Unit = {
        timeoutPromise.failure(new TimeoutException())
        timer.cancel()
      }
    },
    duration.toMillis
  )
  
  future.onComplete { result =>
    timer.cancel()
    timeoutPromise.tryComplete(result)
  }
  
  timeoutPromise.future
}

// Fallback
def withFallback[T](primary: Future[T], fallback: => Future[T]): Future[T] = {
  primary.recoverWith {
    case _ => fallback
  }
}

// Parallel execution with limited concurrency
def parallelLimit[A, B](
  items: List[A],
  parallelism: Int
)(f: A => Future[B]): Future[List[B]] = {
  items
    .grouped(parallelism)
    .foldLeft(Future.successful(List.empty[B])) { (acc, batch) =>
      for {
        prevResults <- acc
        batchResults <- Future.sequence(batch.map(f))
      } yield prevResults ++ batchResults
    }
}
```

### 4.3 Akka Actors

#### Теория

```
Actor Model:
┌────────────────┐
│  Actor A       │
│  - State       │◄──── Message
│  - Behavior    │
│  - Mailbox     │
└───────┬────────┘
        │ sends
        ▼
┌────────────────┐
│  Actor B       │
└────────────────┘

Принципы:
1. Изолированное состояние
2. Асинхронная коммуникация
3. Location transparency
4. Supervision
```

#### Классический API

```scala
import akka.actor._

// Сообщения
sealed trait UserMessage
case class CreateUser(name: String) extends UserMessage
case class GetUser(id: Long) extends UserMessage
case class UpdateUser(id: Long, name: String) extends UserMessage
case class DeleteUser(id: Long) extends UserMessage

case class User(id: Long, name: String)

// Actor
class UserActor extends Actor with ActorLogging {
  private var users: Map[Long, User] = Map.empty
  private var nextId: Long = 1L
  
  def receive: Receive = {
    case CreateUser(name) =>
      val user = User(nextId, name)
      users = users + (nextId -> user)
      nextId += 1
      log.info(s"Created user: $user")
      sender() ! user
    
    case GetUser(id) =>
      val user = users.get(id)
      log.info(s"Get user $id: $user")
      sender() ! user
    
    case UpdateUser(id, name) =>
      users.get(id) match {
        case Some(user) =>
          val updated = user.copy(name = name)
          users = users + (id -> updated)
          sender() ! Some(updated)
        case None =>
          sender() ! None
      }
    
    case DeleteUser(id) =>
      users = users - id
      sender() ! ()
  }
}

// Использование
implicit val system: ActorSystem = ActorSystem("UserSystem")
implicit val timeout: Timeout = Timeout(5.seconds)
import akka.pattern.ask

val userActor = system.actorOf(Props[UserActor], "userActor")

val future: Future[Any] = userActor ? CreateUser("Alice")
future.mapTo[User].foreach(user => println(s"Created: $user"))
```

#### Typed Actors (Akka Typed)

```scala
import akka.actor.typed._
import akka.actor.typed.scaladsl._

object UserRegistry {
  sealed trait Command
  final case class CreateUser(name: String, replyTo: ActorRef[User]) extends Command
  final case class GetUser(id: Long, replyTo: ActorRef[Option[User]]) extends Command
  
  def apply(): Behavior[Command] = registry(Map.empty, 1L)
  
  private def registry(users: Map[Long, User], nextId: Long): Behavior[Command] =
    Behaviors.receiveMessage {
      case CreateUser(name, replyTo) =>
        val user = User(nextId, name)
        replyTo ! user
        registry(users + (nextId -> user), nextId + 1)
      
      case GetUser(id, replyTo) =>
        replyTo ! users.get(id)
        Behaviors.same
    }
}

// Использование
import akka.actor.typed.scaladsl.AskPattern._

implicit val system: ActorSystem[UserRegistry.Command] = 
  ActorSystem(UserRegistry(), "UserSystem")

implicit val timeout: Timeout = 3.seconds

val futureUser: Future[User] = 
  system.ask(replyTo => UserRegistry.CreateUser("Alice", replyTo))
```

#### Supervision Strategy

```scala
import akka.actor.SupervisorStrategy._

class Supervisor extends Actor {
  override val supervisorStrategy: SupervisorStrategy =
    OneForOneStrategy(maxNrOfRetries = 3, withinTimeRange = 1.minute) {
      case _: ArithmeticException => Resume      // продолжить с текущим состоянием
      case _: NullPointerException => Restart    // перезапустить с начальным состоянием
      case _: IllegalArgumentException => Stop   // остановить актор
      case _: Exception => Escalate              // передать наверх
    }
  
  def receive: Receive = {
    case props: Props => sender() ! context.actorOf(props)
  }
}

// Worker actor
class Worker extends Actor {
  var state: Int = 0
  
  def receive: Receive = {
    case "increment" => state += 1
    case "crash" => throw new Exception("Crashed!")
    case "get" => sender() ! state
  }
}
```

### 4.4 Atomic References и CAS

```scala
import java.util.concurrent.atomic._

class LockFreeCounter {
  private val count = new AtomicInteger(0)
  
  def increment(): Int = count.incrementAndGet()
  def get: Int = count.get()
}

// AtomicReference
class LockFreeStack[A] {
  private val head = new AtomicReference[List[A]](Nil)
  
  @annotation.tailrec
  final def push(element: A): Unit = {
    val currentHead = head.get()
    val newHead = element :: currentHead
    if (!head.compareAndSet(currentHead, newHead)) {
      push(element) // retry
    }
  }
  
  @annotation.tailrec
  final def pop(): Option[A] = {
    val currentHead = head.get()
    currentHead match {
      case Nil => None
      case h :: tail =>
        if (head.compareAndSet(currentHead, tail)) Some(h)
        else pop() // retry
    }
  }
}
```

### 4.5 Thread Pools и Execution Contexts

```scala
import scala.concurrent.ExecutionContext
import java.util.concurrent._

// Fixed thread pool
val fixedPool = ExecutionContext.fromExecutor(
  Executors.newFixedThreadPool(4)
)

// Cached thread pool (создает новые потоки при необходимости)
val cachedPool = ExecutionContext.fromExecutor(
  Executors.newCachedThreadPool()
)

// Work-stealing pool (для CPU-bound задач)
val workStealingPool = ExecutionContext.fromExecutor(
  Executors.newWorkStealingPool()
)

// Кастомный thread pool
val customPool = new ThreadPoolExecutor(
  4,                          // core pool size
  10,                         // maximum pool size
  60L,                        // keep alive time
  TimeUnit.SECONDS,
  new LinkedBlockingQueue[Runnable](100),
  new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
)

val customEC = ExecutionContext.fromExecutor(customPool)

// Использование
Future {
  // CPU-intensive work
}(workStealingPool)

Future {
  // I/O work
}(cachedPool)

// Мониторинг
println(s"Active threads: ${customPool.getActiveCount}")
println(s"Queue size: ${customPool.getQueue.size}")
```

---

## 5. Scala 3 (Dotty) - Новые возможности

### 5.1 New Control Syntax

```scala
// Scala 3 - significant indentation
def findMax(numbers: List[Int]): Option[Int] =
  if numbers.isEmpty then
    None
  else
    Some(numbers.max)

// Pattern matching без фигурных скобок
def describe(x: Any): String = x match
  case 0 => "zero"
  case _: Int => "integer"
  case _: String => "string"
  case _ => "something else"

// For comprehensions
def example =
  for
    x <- List(1, 2, 3)
    y <- List(4, 5, 6)
    if x < y
  yield x * y
```

### 5.2 Enums

```scala
// Простой enum
enum Color:
  case Red, Green, Blue

// Enum с параметрами
enum Planet(mass: Double, radius: Double):
  case Mercury extends Planet(3.303e+23, 2.4397e6)
  case Venus extends Planet(4.869e+24, 6.0518e6)
  case Earth extends Planet(5.976e+24, 6.37814e6)
  
  def surfaceGravity: Double =
    val G = 6.67300e-11
    G * mass / (radius * radius)

// ADT как enum
enum Option[+A]:
  case Some(value: A)
  case None
  
  def map[B](f: A => B): Option[B] = this match
    case Some(v) => Some(f(v))
    case None => None

// Enum с методами
enum Direction(val degrees: Int):
  case North extends Direction(0)
  case East extends Direction(90)
  case South extends Direction(180)
  case West extends Direction(270)
  
  def opposite: Direction = this match
    case North => South
    case South => North
    case East => West
    case West => East
```

### 5.3 Union и Intersection Types

```scala
// Union types
type IntOrString = Int | String

def process(value: IntOrString): String = value match
  case i: Int => s"Int: $i"
  case s: String => s"String: $s"

// Intersection types
trait Resettable:
  def reset(): Unit

trait Closeable:
  def close(): Unit

def cleanup(resource: Resettable & Closeable): Unit =
  resource.reset()
  resource.close()

// Практическое применение
type Json = String | Int | Double | Boolean | Null | 
            List[Json] | Map[String, Json]

def parseJson(input: String): Json = ???

// Error handling без Either
def divide(a: Int, b: Int): Int | String =
  if b == 0 then "Division by zero"
  else a / b

divide(10, 2) match
  case i: Int => println(s"Result: $i")
  case error: String => println(s"Error: $error")
```

### 5.4 Opaque Types

```scala
// Проблема: type alias не добавляет type safety
type UserId = Long
type OrderId = Long

def getUser(id: UserId): User = ???
def getOrder(id: OrderId): Order = ???

// Можно случайно перепутать:
val userId: UserId = 123L
getOrder(userId) // компилируется, но неправильно!

// Решение: opaque types
object UserModule:
  opaque type UserId = Long
  
  object UserId:
    def apply(value: Long): UserId = value
  
  extension (id: UserId)
    def value: Long = id

object OrderModule:
  opaque type OrderId = Long
  
  object OrderId:
    def apply(value: Long): OrderId = value
  
  extension (id: OrderId)
    def value: Long = id

import UserModule._
import OrderModule._

val userId = UserId(123L)
val orderId = OrderId(456L)

// getOrder(userId) // ОШИБКА КОМПИЛЯЦИИ! ✓
getOrder(orderId) // OK

// Внутри модуля UserId = Long, снаружи - новый тип
```

#### Практические примеры

```scala
// Type-safe measurements
object Measurements:
  opaque type Meters = Double
  opaque type Seconds = Double
  opaque type MetersPerSecond = Double
  
  object Meters:
    def apply(value: Double): Meters = value
    extension (m: Meters)
      def +(other: Meters): Meters = m + other
      def /(s: Seconds): MetersPerSecond = m / s
      def value: Double = m
  
  object Seconds:
    def apply(value: Double): Seconds = value
    extension (s: Seconds)
      def value: Double = s
  
  object MetersPerSecond:
    def apply(value: Double): MetersPerSecond = value
    extension (mps: MetersPerSecond)
      def value: Double = mps

import Measurements._

val distance = Meters(100.0)
val time = Seconds(10.0)
val speed = distance / time // MetersPerSecond(10.0)

// val wrong = distance + time // ОШИБКА КОМПИЛЯЦИИ!
```

### 5.5 Extension Methods

```scala
// Scala 3 syntax
extension (s: String)
  def isValidEmail: Boolean = s.contains("@")
  def toSnakeCase: String = 
    s.replaceAll("([A-Z])", "_$1").toLowerCase

"test@example.com".isValidEmail // true
"HelloWorld".toSnakeCase // "_hello_world"

// Generic extensions
extension [A](list: List[A])
  def second: Option[A] = list.drop(1).headOption
  def secondLast: Option[A] = list.reverse.drop(1).headOption

List(1, 2, 3).second // Some(2)

// Extension methods с type constraints
extension [A: Ordering](list: List[A])
  def sortedDesc: List[A] = list.sorted.reverse

List(3, 1, 4, 1, 5).sortedDesc // List(5, 4, 3, 1, 1)

// Collective extensions
extension [A](list: List[A])
  def second: Option[A] = list.drop(1).headOption
  def third: Option[A] = list.drop(2).headOption
  def fourth: Option[A] = list.drop(3).headOption
```

### 5.6 Given Imports

```scala
// Scala 2 - import всех implicits
import cats.instances.all._
import cats.syntax.all._

// Scala 3 - более точный контроль
import cats.given // только given instances
import cats._ // все кроме given
import cats.{given, _} // все включая given

// Selective import
import scala.concurrent.ExecutionContext.given

// Импорт по типу
import Ordering.given Ordering[Int]
```

### 5.7 Context Functions

```scala
// Context function type
type Executable[T] = ExecutionContext ?=> T

def runAsync[T](body: Executable[T]): Future[T] =
  given ec: ExecutionContext = ???
  Future(body)

// Использование
val result: Future[Int] = runAsync {
  // ExecutionContext доступен неявно
  println("Running async")
  42
}

// Builder pattern с context functions
trait Transaction
type Transactional[T] = Transaction ?=> T

def inTransaction[T](body: Transactional[T]): T =
  given tx: Transaction = new Transaction {}
  body

def query(sql: String): Transactional[List[String]] =
  println(s"Executing: $sql")
  List("result")

def update(sql: String): Transactional[Int] =
  println(s"Executing: $sql")
  1

// Использование
val result = inTransaction {
  val users = query("SELECT * FROM users")
  val count = update("UPDATE users SET active = true")
  (users, count)
}
```

### 5.8 Match Types

```scala
// Pattern matching на уровне типов
type Elem[X] = X match
  case String => Char
  case Array[t] => t
  case Iterable[t] => t
  case _ => X

val charElem: Elem[String] = 'a' // Char
val intElem: Elem[Array[Int]] = 42 // Int
val stringElem: Elem[List[String]] = "hello" // String

// Практическое применение - type-safe JSON
type JsonValue[T] = T match
  case String => String
  case Int => Int
  case Boolean => Boolean
  case List[t] => List[JsonValue[t]]
  case None.type => Null

def encodeJson[T](value: T): JsonValue[T] = value match
  case s: String => s
  case i: Int => i
  case b: Boolean => b
  case list: List[t] => list.map(encodeJson)
  case None => null
```

---

## 6. Effect Systems (Cats Effect, ZIO)

### 6.1 Cats Effect 3.x

#### IO Monad

```scala
import cats.effect._
import cats.effect.unsafe.implicits.global

// Создание IO
val io1: IO[Int] = IO.pure(42)
val io2: IO[Int] = IO(println("Side effect!")).as(42)
val io3: IO[Int] = IO.delay {
  println("Delayed effect")
  42
}

// Комбинирование
val combined: IO[Int] = for {
  a <- IO(10)
  b <- IO(20)
  _ <- IO.println(s"Sum: ${a + b}")
} yield a + b

// Параллельное выполнение
val parallel: IO[(Int, String)] = (
  IO.sleep(1.second) *> IO.pure(42),
  IO.sleep(1.second) *> IO.pure("hello")
).parTupled

// Error handling
val withError: IO[Int] = IO.raiseError(new Exception("Error!"))

val recovered: IO[Int] = withError.handleErrorWith { error =>
  IO.println(s"Error: ${error.getMessage}") *> IO.pure(0)
}

// Resource management
def makeResource: Resource[IO, Connection] =
  Resource.make(
    IO.println("Acquiring connection") *> 
      IO.pure(new Connection("jdbc:..."))
  )(conn =>
    IO.println("Releasing connection") *>
      IO(conn.close())
  )

val program: IO[List[String]] = makeResource.use { conn =>
  IO(conn.query("SELECT * FROM users"))
}
```

#### Fiber - легковесные потоки

```scala
// Создание Fiber
val fiber1: IO[Fiber[IO, Throwable, Int]] = IO(42).start

// Join - ожидание результата
val result: IO[Int] = for {
  fiber <- IO.sleep(1.second).as(42).start
  value <- fiber.join
} yield value

// Cancel - отмена Fiber
val cancelable: IO[Unit] = for {
  fiber <- IO.sleep(10.seconds).start
  _ <- IO.sleep(1.second)
  _ <- fiber.cancel // отменяет выполнение
} yield ()

// Параллельное выполнение с Fibers
def parallelTasks: IO[List[Int]] = {
  val tasks = List(
    IO.sleep(1.second).as(1),
    IO.sleep(2.seconds).as(2),
    IO.sleep(3.seconds).as(3)
  )
  
  tasks.parSequence // выполнится за 3 секунды, не за 6
}

// Race - первый завершившийся
val race: IO[Either[Int, String]] = IO.race(
  IO.sleep(1.second).as(42),
  IO.sleep(2.seconds).as("hello")
) // Left(42)
```

#### Ref - потокобезопасное изменяемое состояние

```scala
import cats.effect.Ref

// Создание Ref
val program: IO[Int] = for {
  ref <- Ref.of[IO, Int](0)
  _ <- ref.update(_ + 1)
  _ <- ref.update(_ + 1)
  value <- ref.get
} yield value // 2

// Atomic operations
def incrementCounter(ref: Ref[IO, Int], times: Int): IO[Unit] =
  ref.update(_ + 1).replicateA(times).void

val concurrentProgram: IO[Int] = for {
  ref <- Ref.of[IO, Int](0)
  fibers <- (1 to 100).toList.traverse(_ => 
    incrementCounter(ref, 100).start
  )
  _ <- fibers.traverse(_.join)
  result <- ref.get
} yield result // всегда 10000
```

#### Deferred - одноразовое обещание

```scala
import cats.effect.Deferred

def producer(deferred: Deferred[IO, Int]): IO[Unit] =
  IO.sleep(2.seconds) *> deferred.complete(42).void

def consumer(deferred: Deferred[IO, Int]): IO[Int] =
  IO.println("Waiting...") *> deferred.get <* IO.println("Got value!")

val program: IO[Int] = for {
  deferred <- Deferred[IO, Int]
  _ <- producer(deferred).start
  result <- consumer(deferred)
} yield result
```

#### Practical Example - HTTP Client

```scala
import cats.effect._
import org.http4s._
import org.http4s.client._
import org.http4s.client.blaze._

case class User(id: Long, name: String)

class UserService[F[_]: Async](client: Client[F]) {
  
  def getUser(id: Long): F[Option[User]] =
    client.expect[String](s"https://api.example.com/users/$id")
      .map(json => Some(User(id, json))) // упрощенный парсинг
      .handleErrorWith { error =>
        Async[F].println(s"Error: $error").as(None)
      }
  
  def getUsers(ids: List[Long]): F[List[User]] =
    ids.parTraverse(getUser).map(_.flatten)
  
  def getUserWithTimeout(id: Long, duration: FiniteDuration): F[Option[User]] =
    getUser(id).timeout(duration).handleErrorWith {
      case _: TimeoutException =>
        Async[F].println("Request timed out").as(None)
      case error =>
        Async[F].raiseError(error)
    }
}

// Использование
object Main extends IOApp {
  def run(args: List[String]): IO[ExitCode] = {
    BlazeClientBuilder[IO].resource.use { client =>
      val service = new UserService[IO](client)
      
      for {
        user <- service.getUser(1L)
        _ <- IO.println(s"User: $user")
        users <- service.getUsers(List(1L, 2L, 3L))
        _ <- IO.println(s"Users: $users")
      } yield ExitCode.Success
    }
  }
}
```

### 6.2 ZIO 2.x

#### ZIO Type

```scala
import zio._

// ZIO[R, E, A]
// R - Environment (dependencies)
// E - Error type
// A - Success type

// Простые примеры
val pure: UIO[Int] = ZIO.succeed(42)
val effect: Task[Int] = ZIO.attempt(Random.nextInt())
val failed: IO[String, Nothing] = ZIO.fail("Error")

// Комбинирование
val combined: Task[Int] = for {
  a <- ZIO.succeed(10)
  b <- ZIO.succeed(20)
  _ <- Console.printLine(s"Sum: ${a + b}")
} yield a + b

// Параллельное выполнение
val parallel: UIO[(Int, String)] = (
  ZIO.succeed(42),
  ZIO.succeed("hello")
).zipPar

// Error handling
val withRecovery: UIO[Int] = ZIO.fail("Error")
  .catchAll(error => Console.printLine(error) *> ZIO.succeed(0))

val withOrElse: UIO[Int] = ZIO.fail("Error").orElse(ZIO.succeed(42))
```

#### ZLayer - Dependency Injection

```scala
// Определение сервисов
trait UserRepository {
  def getUser(id: Long): Task[Option[User]]
  def saveUser(user: User): Task[Unit]
}

object UserRepository {
  def getUser(id: Long): RIO[UserRepository, Option[User]] =
    ZIO.serviceWithZIO[UserRepository](_.getUser(id))
  
  def saveUser(user: User): RIO[UserRepository, Unit] =
    ZIO.serviceWithZIO[UserRepository](_.saveUser(user))
}

trait EmailService {
  def sendEmail(to: String, body: String): Task[Unit]
}

object EmailService {
  def sendEmail(to: String, body: String): RIO[EmailService, Unit] =
    ZIO.serviceWithZIO[EmailService](_.sendEmail(to, body))
}

// Имплементации
case class UserRepositoryLive() extends UserRepository {
  private val users = scala.collection.mutable.Map.empty[Long, User]
  
  def getUser(id: Long): Task[Option[User]] =
    ZIO.succeed(users.get(id))
  
  def saveUser(user: User): Task[Unit] =
    ZIO.succeed(users.put(user.id, user)).unit
}

case class EmailServiceLive() extends EmailService {
  def sendEmail(to: String, body: String): Task[Unit] =
    Console.printLine(s"Sending email to $to: $body")
}

// Layers
val userRepositoryLayer: ULayer[UserRepository] =
  ZLayer.succeed(UserRepositoryLive())

val emailServiceLayer: ULayer[EmailService] =
  ZLayer.succeed(EmailServiceLive())

// Сервис, зависящий от других сервисов
trait UserService {
  def registerUser(name: String, email: String): Task[User]
}

case class UserServiceLive(
  userRepo: UserRepository,
  emailService: EmailService
) extends UserService {
  
  def registerUser(name: String, email: String): Task[User] = for {
    user <- ZIO.succeed(User(Random.nextLong(), name, email))
    _ <- userRepo.saveUser(user)
    _ <- emailService.sendEmail(email, s"Welcome, $name!")
  } yield user
}

object UserServiceLive {
  val layer: RLayer[UserRepository with EmailService, UserService] =
    ZLayer {
      for {
        userRepo <- ZIO.service[UserRepository]
        emailService <- ZIO.service[EmailService]
      } yield UserServiceLive(userRepo, emailService)
    }
}

// Использование
object MainApp extends ZIOAppDefault {
  val program: RIO[UserService, User] = for {
    service <- ZIO.service[UserService]
    user <- service.registerUser("Alice", "alice@example.com")
  } yield user
  
  def run = program.provide(
    userRepositoryLayer,
    emailServiceLayer,
    UserServiceLive.layer
  )
}
```

#### STM - Software Transactional Memory

```scala
import zio.stm._

// TRef - transactional reference
val transfer: UIO[Unit] = for {
  aliceAccount <- TRef.make(100).commit
  bobAccount <- TRef.make(50).commit
  
  _ <- STM.atomically {
    for {
      aliceBalance <- aliceAccount.get
      _ <- if (aliceBalance >= 30) {
        for {
          _ <- aliceAccount.update(_ - 30)
          _ <- bobAccount.update(_ + 30)
        } yield ()
      } else STM.fail(new Exception("Insufficient funds"))
    } yield ()
  }
  
  alice <- aliceAccount.get.commit
  bob <- bobAccount.get.commit
  _ <- Console.printLine(s"Alice: $alice, Bob: $bob")
} yield ()

// TMap - transactional map
case class BankSystem(accounts: TMap[String, Int]) {
  
  def transfer(from: String, to: String, amount: Int): UIO[Either[String, Unit]] =
    (for {
      fromBalance <- accounts.getOrElse(from, 0)
      _ <- if (fromBalance >= amount) {
        for {
          _ <- accounts.update(from, _ - amount)
          _ <- accounts.update(to, _ + amount)
        } yield ()
      } else STM.fail("Insufficient funds")
    } yield ()).commit.either
  
  def getBalance(account: String): UIO[Int] =
    accounts.getOrElse(account, 0).commit
}
```

#### Streaming с ZIO Streams

```scala
import zio.stream._

// Создание Stream
val stream1: ZStream[Any, Nothing, Int] = ZStream(1, 2, 3, 4, 5)
val stream2: ZStream[Any, Nothing, Int] = ZStream.fromIterable(1 to 100)
val stream3: ZStream[Any, Nothing, Int] = ZStream.iterate(0)(_ + 1)

// Трансформации
val doubled: ZStream[Any, Nothing, Int] = stream1.map(_ * 2)
val filtered: ZStream[Any, Nothing, Int] = stream1.filter(_ % 2 == 0)

// Chunking для эффективности
val chunked: ZStream[Any, Nothing, Chunk[Int]] = 
  stream2.grouped(10)

// Side effects
val withEffects: ZStream[Any, IOException, Int] = stream1.tap { n =>
  Console.printLine(s"Processing: $n")
}

// Parallel processing
val parallel: ZStream[Any, Nothing, Int] = stream2
  .mapZIOPar(4)(n => ZIO.succeed(n * 2))

// Практический пример - обработка файла
def processLargeFile(path: String): ZIO[Any, Throwable, Unit] = {
  ZStream
    .fromFileName(path)
    .via(ZPipeline.utf8Decode)
    .via(ZPipeline.splitLines)
    .filter(_.nonEmpty)
    .map(_.split(","))
    .filter(_.length == 3)
    .map { parts =>
      User(parts(0).toLong, parts(1), parts(2))
    }
    .foreach { user =>
      Console.printLine(s"Processed user: $user")
    }
}
```

---

## 7. Streaming и Reactive Programming

### 7.1 FS2 (Functional Streams for Scala)

#### Основы

```scala
import fs2._
import cats.effect._

// Создание Stream
val stream1: Stream[Pure, Int] = Stream(1, 2, 3, 4, 5)
val stream2: Stream[Pure, Int] = Stream.range(1, 100)
val stream3: Stream[Pure, Int] = Stream.iterate(0)(_ + 1)

// Stream с эффектами
val effectfulStream: Stream[IO, Int] = Stream.eval(IO(Random.nextInt()))

// Трансформации
val doubled = stream1.map(_ * 2)
val filtered = stream1.filter(_ % 2 == 0)
val flatMapped = stream1.flatMap(n => Stream.range(0, n))

// Chunking
val chunked = stream2.chunkN(10)

// Практический пример
def readFile(path: String): Stream[IO, String] =
  Stream
    .resource(Resource.fromAutoCloseable(IO(Source.fromFile(path))))
    .flatMap(source => Stream.fromIterator[IO](source.getLines(), 1024))

def processFile(path: String): IO[Unit] =
  readFile(path)
    .filter(_.nonEmpty)
    .map(_.split(","))
    .filter(_.length == 3)
    .map { parts => User(parts(0).toLong, parts(1), parts(2)) }
    .evalMap(user => IO.println(s"Processing: $user"))
    .compile
    .drain
```

#### Concurrent Streams

```scala
// Merge - объединение потоков
val merged = stream1.merge(stream2)

// Parallel processing
val parallel = stream2
  .parEvalMap(4)(n => IO.sleep(100.millis) *> IO.pure(n * 2))

// Broadcasting
val broadcastProgram = Stream(1, 2, 3, 4, 5)
  .broadcastThrough(
    _.map(_ * 2).evalMap(n => IO.println(s"Doubled: $n")),
    _.filter(_ % 2 == 0).evalMap(n => IO.println(s"Even: $n"))
  )
  .compile
  .drain

// Queue-based streams
def queueExample: IO[Unit] = {
  Stream.eval(Queue.bounded[IO, Int](10)).flatMap { queue =>
    val producer = Stream
      .iterate(0)(_ + 1)
      .evalMap(i => queue.offer(i) *> IO.sleep(100.millis))
    
    val consumer = Stream
      .fromQueueUnterminated(queue)
      .evalMap(i => IO.println(s"Consumed: $i"))
    
    producer.merge(consumer)
  }.compile.drain
}
```

#### Resource Management

```scala
def withResource[A](acquire: IO[A])(release: A => IO[Unit]): Stream[IO, A] =
  Stream.bracket(acquire)(release)

// Database connection pool
case class ConnectionPool(size: Int) {
  def getConnection: IO[Connection] = ???
  def releaseConnection(conn: Connection): IO[Unit] = ???
}

def streamQuery(pool: ConnectionPool): Stream[IO, User] =
  Stream
    .bracket(pool.getConnection)(pool.releaseConnection)
    .flatMap { conn =>
      Stream.eval(IO(conn.query("SELECT * FROM users")))
        .flatMap(Stream.emits)
    }
```

### 7.2 Akka Streams

#### Основы

```scala
import akka.actor.ActorSystem
import akka.stream._
import akka.stream.scaladsl._

implicit val system: ActorSystem = ActorSystem("StreamSystem")

// Source
val source1: Source[Int, NotUsed] = Source(1 to 10)
val source2: Source[Int, NotUsed] = Source.repeat(42)
val source3: Source[Int, NotUsed] = Source.tick(
  initialDelay = 0.seconds,
  interval = 1.second,
  tick = 1
)

// Flow
val doubleFlow: Flow[Int, Int, NotUsed] = Flow[Int].map(_ * 2)
val filterFlow: Flow[Int, Int, NotUsed] = Flow[Int].filter(_ % 2 == 0)

// Sink
val printSink: Sink[Int, Future[Done]] = Sink.foreach(println)
val sumSink: Sink[Int, Future[Int]] = Sink.fold(0)(_ + _)

// Runnable Graph
val result: Future[Done] = source1
  .via(doubleFlow)
  .via(filterFlow)
  .runWith(printSink)

// Короткая форма
source1.map(_ * 2).filter(_ % 2 == 0).runForeach(println)
```

#### Backpressure

```scala
// Медленный consumer
val slowConsumer = Flow[Int].throttle(
  elements = 1,
  per = 1.second,
  maximumBurst = 10,
  mode = ThrottleMode.Shaping
)

source1.via(slowConsumer).runForeach(println)

// Buffer
val buffered = Flow[Int].buffer(
  size = 100,
  overflowStrategy = OverflowStrategy.backpressure
)

// Conflict strategies
val dropHead = Flow[Int].buffer(10, OverflowStrategy.dropHead)
val dropTail = Flow[Int].buffer(10, OverflowStrategy.dropTail)
val dropBuffer = Flow[Int].buffer(10, OverflowStrategy.dropBuffer)
```

#### Graph DSL

```scala
import akka.stream.scaladsl.GraphDSL
import akka.stream.ClosedShape

val graph = GraphDSL.create() { implicit builder =>
  import GraphDSL.Implicits._
  
  val source = Source(1 to 10)
  val broadcast = builder.add(Broadcast[Int](2))
  val merge = builder.add(Merge[Int](2))
  val sink = Sink.foreach(println)
  
  source ~> broadcast ~> Flow[Int].map(_ * 2) ~> merge ~> sink
            broadcast ~> Flow[Int].map(_ * 3) ~> merge
  
  ClosedShape
}

RunnableGraph.fromGraph(graph).run()

// Practical example - Fan-out/Fan-in
val processingGraph = GraphDSL.create() { implicit builder =>
  import GraphDSL.Implicits._
  
  val input = Source(1 to 100)
  val broadcast = builder.add(Broadcast[Int](3))
  val merge = builder.add(Merge[String](3))
  val output = Sink.foreach[String](println)
  
  input ~> broadcast ~> Flow[Int].map(n => s"Process 1: ${n * 2}") ~> merge ~> output
           broadcast ~> Flow[Int].map(n => s"Process 2: ${n * 3}") ~> merge
           broadcast ~> Flow[Int].map(n => s"Process 3: ${n * 4}") ~> merge
  
  ClosedShape
}
```

#### Integration with Futures

```scala
def fetchUser(id: Long): Future[User] = ???

// MapAsync - preserves order
val orderedResults: Source[User, NotUsed] = Source(1L to 100L)
  .mapAsync(parallelism = 10)(fetchUser)

// MapAsyncUnordered - faster but no order guarantee
val unorderedResults: Source[User, NotUsed] = Source(1L to 100L)
  .mapAsyncUnordered(parallelism = 10)(fetchUser)
```

### 7.3 Reactive Streams

#### Publisher/Subscriber

```scala
import org.reactivestreams._

class NumberPublisher(max: Int) extends Publisher[Int] {
  def subscribe(subscriber: Subscriber[_ >: Int]): Unit = {
    subscriber.onSubscribe(new Subscription {
      private var current = 0
      private var cancelled = false
      
      def request(n: Long): Unit = {
        (0L until n).foreach { _ =>
          if (current < max && !cancelled) {
            subscriber.onNext(current)
            current += 1
          } else if (current >= max && !cancelled) {
            subscriber.onComplete()
            cancelled = true
          }
        }
      }
      
      def cancel(): Unit = cancelled = true
    })
  }
}

// Custom Subscriber
class PrintSubscriber extends Subscriber[Int] {
  private var subscription: Subscription = _
  
  def onSubscribe(s: Subscription): Unit = {
    subscription = s
    subscription.request(1) // request one element
  }
  
  def onNext(t: Int): Unit = {
    println(s"Received: $t")
    subscription.request(1) // request next element
  }
  
  def onError(t: Throwable): Unit = {
    println(s"Error: ${t.getMessage}")
  }
  
  def onComplete(): Unit = {
    println("Stream completed")
  }
}
```

---

## 8. Архитектура и Design Patterns

### 8.1 Clean Architecture

```
┌────────────────────────────────┐
│   Presentation Layer           │
│   (HTTP, gRPC, CLI)            │
└───────────┬────────────────────┘
            │
┌───────────┴────────────────────┐
│   Application Layer            │
│   (Use Cases, Services)        │
└───────────┬────────────────────┘
            │
┌───────────┴────────────────────┐
│   Domain Layer                 │
│   (Entities, Value Objects)    │
└───────────┬────────────────────┘
            │
┌───────────┴────────────────────┐
│   Infrastructure Layer         │
│   (DB, External APIs, Config)  │
└────────────────────────────────┘
```

#### Implementation

```scala
// Domain Layer
package domain

case class UserId(value: Long) extends AnyVal
case class Email(value: String) extends AnyVal

case class User(
  id: UserId,
  name: String,
  email: Email,
  createdAt: Instant
)

trait UserRepository {
  def findById(id: UserId): IO[Option[User]]
  def save(user: User): IO[Unit]
  def findByEmail(email: Email): IO[Option[User]]
}

// Application Layer
package application

class UserService(repo: UserRepository, emailService: EmailService) {
  
  def registerUser(name: String, email: String): IO[Either[String, User]] = {
    for {
      validEmail <- validateEmail(email)
      existing <- repo.findByEmail(Email(email))
      result <- existing match {
        case Some(_) => IO.pure(Left("Email already registered"))
        case None =>
          val user = User(
            UserId(Random.nextLong()),
            name,
            Email(email),
            Instant.now()
          )
          for {
            _ <- repo.save(user)
            _ <- emailService.sendWelcome(email, name)
          } yield Right(user)
      }
    } yield result
  }
  
  private def validateEmail(email: String): IO[Either[String, Email]] =
    if (email.contains("@")) IO.pure(Right(Email(email)))
    else IO.pure(Left("Invalid email"))
}

// Infrastructure Layer
package infrastructure

class PostgresUserRepository(xa: Transactor[IO]) extends UserRepository {
  import doobie._
  import doobie.implicits._
  
  def findById(id: UserId): IO[Option[User]] =
    sql"""
      SELECT id, name, email, created_at 
      FROM users 
      WHERE id = ${id.value}
    """
      .query[User]
      .option
      .transact(xa)
  
  def save(user: User): IO[Unit] =
    sql"""
      INSERT INTO users (id, name, email, created_at)
      VALUES (${user.id.value}, ${user.name}, ${user.email.value}, ${user.createdAt})
    """
      .update
      .run
      .transact(xa)
      .void
  
  def findByEmail(email: Email): IO[Option[User]] =
    sql"""
      SELECT id, name, email, created_at 
      FROM users 
      WHERE email = ${email.value}
    """
      .query[User]
      .option
      .transact(xa)
}

// Presentation Layer
package presentation

class UserRoutes(service: UserService) {
  import org.http4s._
  import org.http4s.dsl.io._
  import org.http4s.circe.CirceEntityCodec._
  
  val routes: HttpRoutes[IO] = HttpRoutes.of[IO] {
    case req @ POST -> Root / "users" =>
      for {
        input <- req.as[CreateUserRequest]
        result <- service.registerUser(input.name, input.email)
        response <- result match {
          case Right(user) => Created(user)
          case Left(error) => BadRequest(error)
        }
      } yield response
    
    case GET -> Root / "users" / LongVar(id) =>
      service.repo.findById(UserId(id)).flatMap {
        case Some(user) => Ok(user)
        case None => NotFound()
      }
  }
}

case class CreateUserRequest(name: String, email: String)
```

### 8.2 Hexagonal Architecture (Ports & Adapters)

```scala
// Domain - Core business logic
package domain

trait NotificationPort {
  def send(userId: UserId, message: String): IO[Unit]
}

trait PaymentPort {
  def processPayment(userId: UserId, amount: Money): IO[PaymentResult]
}

class OrderService(
  orderRepo: OrderRepository,
  notification: NotificationPort,
  payment: PaymentPort
) {
  def placeOrder(userId: UserId, items: List[OrderItem]): IO[Order] = {
    for {
      order <- createOrder(userId, items)
      _ <- orderRepo.save(order)
      paymentResult <- payment.processPayment(userId, order.total)
      _ <- if (paymentResult.success) {
        notification.send(userId, s"Order ${order.id} confirmed!")
      } else {
        IO.raiseError(new Exception("Payment failed"))
      }
    } yield order
  }
}

// Adapters - Implementations
package adapters

class EmailNotificationAdapter(client: EmailClient) extends NotificationPort {
  def send(userId: UserId, message: String): IO[Unit] = {
    for {
      user <- getUserEmail(userId)
      _ <- client.sendEmail(user, "Order Update", message)
    } yield ()
  }
}

class StripePaymentAdapter(apiKey: String) extends PaymentPort {
  def processPayment(userId: UserId, amount: Money): IO[PaymentResult] = {
    // Integration with Stripe API
    ???
  }
}

class SmsNotificationAdapter(twilioClient: TwilioClient) extends NotificationPort {
  def send(userId: UserId, message: String): IO[Unit] = {
    // Integration with Twilio
    ???
  }
}
```

### 8.3 Repository Pattern

```scala
trait Repository[F[_], K, V] {
  def findById(id: K): F[Option[V]]
  def findAll: F[List[V]]
  def save(entity: V): F[Unit]
  def update(entity: V): F[Unit]
  def delete(id: K): F[Unit]
}

// Generic implementation with Doobie
abstract class DoobieRepository[K, V](xa: Transactor[IO])
  extends Repository[IO, K, V] {
  
  protected def tableName: String
  protected def toRow(entity: V): Map[String, Any]
  protected def fromRow(row: Map[String, Any]): V
  protected def idColumn: String
  protected def getId(entity: V): K
  
  def findById(id: K): IO[Option[V]] =
    Fragment
      .const(s"SELECT * FROM $tableName WHERE $idColumn = ")
      .append(fr"$id")
      .query[V]
      .option
      .transact(xa)
  
  def findAll: IO[List[V]] =
    Fragment
      .const(s"SELECT * FROM $tableName")
      .query[V]
      .to[List]
      .transact(xa)
  
  // ... other methods
}

// Specific repository
class UserRepositoryImpl(xa: Transactor[IO]) 
  extends DoobieRepository[UserId, User](xa) {
  
  protected def tableName: String = "users"
  protected def idColumn: String = "id"
  protected def getId(user: User): UserId = user.id
  // ... implementations
  
  // Additional domain-specific methods
  def findByEmail(email: Email): IO[Option[User]] = ???
  def findActive: IO[List[User]] = ???
}
```

### 8.4 CQRS (Command Query Responsibility Segregation)

```scala
// Commands - изменяют состояние
sealed trait UserCommand
case class CreateUser(name: String, email: String) extends UserCommand
case class UpdateUser(id: UserId, name: String) extends UserCommand
case class DeleteUser(id: UserId) extends UserCommand

// Queries - только читают
sealed trait UserQuery
case class GetUser(id: UserId) extends UserQuery
case class GetUserByEmail(email: Email) extends UserQuery
case class GetAllUsers() extends UserQuery

// Command Handler
class UserCommandHandler(
  writeRepo: UserWriteRepository,
  eventBus: EventBus
) {
  def handle(command: UserCommand): IO[Unit] = command match {
    case CreateUser(name, email) =>
      for {
        user <- IO.pure(User(UserId(Random.nextLong()), name, Email(email), Instant.now()))
        _ <- writeRepo.save(user)
        _ <- eventBus.publish(UserCreated(user.id, name, email))
      } yield ()
    
    case UpdateUser(id, name) =>
      for {
        _ <- writeRepo.update(id, name)
        _ <- eventBus.publish(UserUpdated(id, name))
      } yield ()
    
    case DeleteUser(id) =>
      for {
        _ <- writeRepo.delete(id)
        _ <- eventBus.publish(UserDeleted(id))
      } yield ()
  }
}

// Query Handler
class UserQueryHandler(readRepo: UserReadRepository) {
  def handle(query: UserQuery): IO[QueryResult] = query match {
    case GetUser(id) =>
      readRepo.findById(id).map(QueryResult.UserResult)
    
    case GetUserByEmail(email) =>
      readRepo.findByEmail(email).map(QueryResult.UserResult)
    
    case GetAllUsers() =>
      readRepo.findAll.map(QueryResult.UsersResult)
  }
}

// Read Model - denormalized, optimized for queries
case class UserReadModel(
  id: UserId,
  name: String,
  email: String,
  orderCount: Int,
  totalSpent: Money,
  lastOrderDate: Option[Instant]
)

// Event Handler - updates read model
class UserEventHandler(readModelRepo: UserReadModelRepository) {
  def handle(event: DomainEvent): IO[Unit] = event match {
    case UserCreated(id, name, email) =>
      readModelRepo.create(UserReadModel(
        UserId(id),
        name,
        email,
        0,
        Money.zero,
        None
      ))
    
    case OrderPlaced(userId, amount, timestamp) =>
      readModelRepo.incrementOrderCount(userId, amount, timestamp)
    
    case _ => IO.unit
  }
}
```

### 8.5 Event Sourcing

```scala
// Events
sealed trait DomainEvent {
  def aggregateId: UserId
  def timestamp: Instant
}

case class UserCreated(
  aggregateId: UserId,
  name: String,
  email: String,
  timestamp: Instant
) extends DomainEvent

case class UserEmailChanged(
  aggregateId: UserId,
  newEmail: String,
  timestamp: Instant
) extends DomainEvent

case class UserDeleted(
  aggregateId: UserId,
  timestamp: Instant
) extends DomainEvent

// Aggregate
case class UserAggregate(
  id: UserId,
  name: String,
  email: String,
  version: Long,
  deleted: Boolean = false
) {
  def apply(event: DomainEvent): UserAggregate = event match {
    case UserCreated(_, name, email, _) =>
      copy(name = name, email = email, version = version + 1)
    
    case UserEmailChanged(_, newEmail, _) =>
      copy(email = newEmail, version = version + 1)
    
    case UserDeleted(_, _) =>
      copy(deleted = true, version = version + 1)
    
    case _ => this
  }
}

// Event Store
trait EventStore[F[_]] {
  def saveEvents(aggregateId: UserId, events: List[DomainEvent], expectedVersion: Long): F[Unit]
  def getEvents(aggregateId: UserId): F[List[DomainEvent]]
}

// Repository with Event Sourcing
class EventSourcedUserRepository(eventStore: EventStore[IO]) {
  
  def save(aggregate: UserAggregate, events: List[DomainEvent]): IO[Unit] =
    eventStore.saveEvents(aggregate.id, events, aggregate.version)
  
  def load(id: UserId): IO[Option[UserAggregate]] = {
    eventStore.getEvents(id).map { events =>
      if (events.isEmpty) None
      else {
        val initial = UserAggregate(id, "", "", 0)
        Some(events.foldLeft(initial)(_ apply _))
      }
    }
  }
}

// Command handling with events
class UserCommandService(
  repository: EventSourcedUserRepository,
  eventBus: EventBus
) {
  def createUser(name: String, email: String): IO[UserId] = {
    val id = UserId(Random.nextLong())
    val event = UserCreated(id, name, email, Instant.now())
    val aggregate = UserAggregate(id, "", "", 0).apply(event)
    
    for {
      _ <- repository.save(aggregate, List(event))
      _ <- eventBus.publish(event)
    } yield id
  }
  
  def changeEmail(id: UserId, newEmail: String): IO[Unit] = {
    for {
      aggregateOpt <- repository.load(id)
      aggregate <- IO.fromOption(aggregateOpt)(new Exception("User not found"))
      event = UserEmailChanged(id, newEmail, Instant.now())
      updated = aggregate.apply(event)
      _ <- repository.save(updated, List(event))
      _ <- eventBus.publish(event)
    } yield ()
  }
}
```

---

## 9. Testing Strategies

### 9.1 ScalaTest

```scala
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers

class CalculatorSpec extends AnyFlatSpec with Matchers {
  
  "Calculator" should "add two numbers" in {
    val result = Calculator.add(2, 3)
    result should be(5)
  }
  
  it should "subtract two numbers" in {
    Calculator.subtract(5, 3) shouldEqual 2
  }
  
  it should "throw exception for division by zero" in {
    assertThrows[ArithmeticException] {
      Calculator.divide(10, 0)
    }
  }
  
  "A List" should "have size 3 when 3 elements added" in {
    val list = List(1, 2, 3)
    list should have size 3
  }
  
  it should "contain element 2" in {
    List(1, 2, 3) should contain(2)
  }
}

// Property-based testing
import org.scalatest.propspec.AnyPropSpec
import org.scalatestplus.scalacheck.ScalaCheckPropertyChecks

class StringPropertySpec extends AnyPropSpec with ScalaCheckPropertyChecks {
  
  property("string concatenation length") {
    forAll { (s1: String, s2: String) =>
      (s1 + s2).length should be(s1.length + s2.length)
    }
  }
  
  property("reverse twice gives original") {
    forAll { (list: List[Int]) =>
      list.reverse.reverse should be(list)
    }
  }
}
```

### 9.2 Mocking с Mockito

```scala
import org.scalatestplus.mockito.MockitoSugar
import org.mockito.Mockito._
import org.mockito.ArgumentMatchers._

class UserServiceSpec extends AnyFlatSpec with Matchers with MockitoSugar {
  
  "UserService" should "create user and send welcome email" in {
    // Arrange
    val mockRepo = mock[UserRepository]
    val mockEmailService = mock[EmailService]
    val service = new UserService(mockRepo, mockEmailService)
    
    val user = User(UserId(1), "Alice", Email("alice@example.com"), Instant.now())
    
    when(mockRepo.findByEmail(any[Email])).thenReturn(IO.pure(None))
    when(mockRepo.save(any[User])).thenReturn(IO.unit)
    when(mockEmailService.sendWelcome(anyString(), anyString())).thenReturn(IO.unit)
    
    // Act
    val result = service.registerUser("Alice", "alice@example.com").unsafeRunSync()
    
    // Assert
    result shouldBe a[Right[_, _]]
    verify(mockRepo, times(1)).save(any[User])
    verify(mockEmailService, times(1)).sendWelcome("alice@example.com", "Alice")
  }
}
```

### 9.3 Cats Effect Testing

```scala
import cats.effect.testing.scalatest.AsyncIOSpec
import org.scalatest.matchers.should.Matchers
import org.scalatest.freespec.AsyncFreeSpec

class AsyncServiceSpec extends AsyncFreeSpec with AsyncIOSpec with Matchers {
  
  "AsyncService" - {
    "should process data asynchronously" in {
      val service = new AsyncService
      
      service.processData("test").asserting { result =>
        result should be("PROCESSED: test")
      }
    }
    
    "should handle errors" in {
      val service = new AsyncService
      
      service.processData("error").assertThrows[Exception]
    }
    
    "should run multiple operations in parallel" in {
      val service = new AsyncService
      
      val program = for {
        result1 <- service.processData("a")
        result2 <- service.processData("b")
      } yield (result1, result2)
      
      program.asserting { case (r1, r2) =>
        r1 should be("PROCESSED: a")
        r2 should be("PROCESSED: b")
      }
    }
  }
}
```

### 9.4 Integration Testing

```scala
import cats.effect._
import doobie._
import doobie.implicits._
import org.scalatest.BeforeAndAfterAll

class UserRepositoryIntegrationSpec 
  extends AnyFlatSpec 
  with Matchers 
  with BeforeAndAfterAll {
  
  var xa: Transactor[IO] = _
  var repo: PostgresUserRepository = _
  
  override def beforeAll(): Unit = {
    // Setup test database
    xa = Transactor.fromDriverManager[IO](
      "org.postgresql.Driver",
      "jdbc:postgresql:testdb",
      "test",
      "test"
    )
    
    // Run migrations
    sql"""
      CREATE TABLE IF NOT EXISTS users (
        id BIGINT PRIMARY KEY,
        name VARCHAR(255),
        email VARCHAR(255),
        created_at TIMESTAMP
      )
    """.update.run.transact(xa).unsafeRunSync()
    
    repo = new PostgresUserRepository(xa)
  }
  
  override def afterAll(): Unit = {
    // Cleanup
    sql"DROP TABLE users".update.run.transact(xa).unsafeRunSync()
  }
  
  "UserRepository" should "save and retrieve user" in {
    val user = User(
      UserId(1),
      "Alice",
      Email("alice@example.com"),
      Instant.now()
    )
    
    val program = for {
      _ <- repo.save(user)
      retrieved <- repo.findById(UserId(1))
    } yield retrieved
    
    val result = program.unsafeRunSync()
    result shouldBe Some(user)
  }
  
  it should "find user by email" in {
    val user = User(
      UserId(2),
      "Bob",
      Email("bob@example.com"),
      Instant.now()
    )
    
    val program = for {
      _ <- repo.save(user)
      retrieved <- repo.findByEmail(Email("bob@example.com"))
    } yield retrieved
    
    val result = program.unsafeRunSync()
    result shouldBe Some(user)
  }
}
```

### 9.5 Contract Testing (Pact)

```scala
import au.com.dius.pact.consumer._
import au.com.dius.pact.consumer.dsl._
import au.com.dius.pact.model._

class UserApiConsumerSpec extends AnyFlatSpec with Matchers {
  
  "User API client" should "fetch user by ID" in {
    val builder = ConsumerPactBuilder
      .consumer("UserService")
      .hasPactWith("UserAPI")
    
    val pact = builder
      .given("user 1 exists")
      .uponReceiving("a request for user 1")
      .path("/users/1")
      .method("GET")
      .willRespondWith()
      .status(200)
      .headers(Map("Content-Type" -> "application/json"))
      .body("""{"id":1,"name":"Alice","email":"alice@example.com"}""")
      .toPact()
    
    MockProviderConfig.createDefault().runTest { mockServer =>
      val client = new UserApiClient(mockServer.getUrl)
      val user = client.getUser(1).unsafeRunSync()
      
      user.id should be(UserId(1))
      user.name should be("Alice")
    }
  }
}
```

---

## 10. Performance и Optimization

### 10.1 Profiling

```scala
// JMH Benchmark
import org.openjdk.jmh.annotations._
import java.util.concurrent.TimeUnit

@State(Scope.Thread)
@BenchmarkMode(Array(Mode.AverageTime))
@OutputTimeUnit(TimeUnit.MICROSECONDS)
class CollectionBenchmark {
  
  val size = 10000
  val list = (1 to size).toList
  val vector = (1 to size).toVector
  val array = (1 to size).toArray
  
  @Benchmark
  def listAppend(): List[Int] = {
    var result = List.empty[Int]
    (1 to 100).foreach(i => result = result :+ i)
    result
  }
  
  @Benchmark
  def listPrepend(): List[Int] = {
    var result = List.empty[Int]
    (1 to 100).foreach(i => result = i :: result)
    result
  }
  
  @Benchmark
  def vectorAppend(): Vector[Int] = {
    var result = Vector.empty[Int]
    (1 to 100).foreach(i => result = result :+ i)
    result
  }
  
  @Benchmark
  def listRandomAccess(): Int = {
    list(size / 2)
  }
  
  @Benchmark
  def vectorRandomAccess(): Int = {
    vector(size / 2)
  }
  
  @Benchmark
  def arrayRandomAccess(): Int = {
    array(size / 2)
  }
}
```

### 10.2 Memory Optimization

```scala
// Value Classes - zero overhead
case class UserId(value: Long) extends AnyVal
case class Email(value: String) extends AnyVal

// Without AnyVal: создается объект (16 bytes header + 8 bytes field)
// With AnyVal: используется примитив напрямую (8 bytes)

// Specialized Collections для примитивов
class PrimitiveArray {
  // НЕ оптимизировано - boxing
  val numbers: Array[Integer] = Array(1, 2, 3)
  
  // Оптимизировано - no boxing
  val optimized: Array[Int] = Array(1, 2, 3)
}

// Lazy evaluation
class DataProcessor {
  // Вычисляется при создании объекта
  val eagerData = expensiveOperation()
  
  // Вычисляется при первом обращении
  lazy val lazyData = expensiveOperation()
  
  // Вычисляется при каждом обращении
  def defData = expensiveOperation()
  
  private def expensiveOperation(): String = {
    Thread.sleep(1000)
    "data"
  }
}
```

### 10.3 Tail Recursion

```scala
// НЕ tail recursive - может привести к StackOverflowError
def factorial(n: Int): Int = {
  if (n <= 1) 1
  else n * factorial(n - 1) // рекурсивный вызов не в хвостовой позиции
}

// Tail recursive - оптимизируется компилятором
@tailrec
def factorialTailRec(n: Int, acc: Int = 1): Int = {
  if (n <= 1) acc
  else factorialTailRec(n - 1, n * acc) // хвостовой вызов
}

// Пример: sum списка
@tailrec
def sum(list: List[Int], acc: Int = 0): Int = list match {
  case Nil => acc
  case h :: tail => sum(tail, acc + h)
}

// foldLeft - tail recursive
// foldRight - NOT tail recursive
val numbers = (1 to 10000).toList

val sumLeft = numbers.foldLeft(0)(_ + _) // OK
// val sumRight = numbers.foldRight(0)(_ + _) // может упасть с StackOverflowError
```

### 10.4 Caching Strategies

```scala
import com.github.benmanes.caffeine.cache._
import scala.jdk.CollectionConverters._

class CachedUserService(repo: UserRepository) {
  
  // Caffeine cache
  private val cache: Cache[UserId, User] = Caffeine.newBuilder()
    .maximumSize(10000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .recordStats()
    .build[UserId, User]()
  
  def getUser(id: UserId): IO[Option[User]] = {
    Option(cache.getIfPresent(id)) match {
      case Some(user) => IO.pure(Some(user))
      case None =>
        repo.findById(id).flatMap {
          case Some(user) =>
            IO(cache.put(id, user)).as(Some(user))
          case None =>
            IO.pure(None)
        }
    }
  }
  
  def stats: CacheStats = cache.stats()
}

// Memoization
class Fibonacci {
  private val memo = scala.collection.mutable.Map.empty[Int, BigInt]
  
  def fib(n: Int): BigInt = {
    memo.getOrElseUpdate(n, {
      if (n <= 1) BigInt(n)
      else fib(n - 1) + fib(n - 2)
    })
  }
}

// Cats Caching
import cats.effect.kernel.Ref

class CatsCache[F[_]: Async, K, V](
  ttl: FiniteDuration,
  fetch: K => F[V]
) {
  case class CacheEntry(value: V, expiresAt: Instant)
  
  private val cache: Ref[F, Map[K, CacheEntry]] = 
    Ref.unsafe(Map.empty)
  
  def get(key: K): F[V] = {
    for {
      now <- Async[F].realTimeInstant
      entries <- cache.get
      result <- entries.get(key) match {
        case Some(entry) if entry.expiresAt.isAfter(now) =>
          Async[F].pure(entry.value)
        case _ =>
          for {
            value <- fetch(key)
            expiresAt = now.plusMillis(ttl.toMillis)
            _ <- cache.update(_ + (key -> CacheEntry(value, expiresAt)))
          } yield value
      }
    } yield result
  }
}
```

### 10.5 Batching и Bulk Operations

```scala
// Batching requests
class BatchingService[F[_]: Async](repo: UserRepository) {
  
  def getUsersBatch(ids: List[UserId]): F[List[User]] = {
    ids.grouped(100).toList.flatTraverse { batch =>
      repo.findByIds(batch)
    }
  }
}

// Bulk insert optimization
class BulkUserRepository(xa: Transactor[IO]) {
  
  def saveAll(users: List[User]): IO[Unit] = {
    val sql = "INSERT INTO users (id, name, email, created_at) VALUES (?, ?, ?, ?)"
    
    Update[User](sql)
      .updateMany(users)
      .transact(xa)
      .void
  }
  
  // Более эффективно для больших объемов
  def saveBulk(users: Stream[IO, User]): IO[Unit] = {
    val sql = "INSERT INTO users (id, name, email, created_at) VALUES (?, ?, ?, ?)"
    
    users
      .chunkN(1000)
      .evalMap { chunk =>
        Update[User](sql)
          .updateMany(chunk.toList)
          .transact(xa)
      }
      .compile
      .drain
  }
}
```

---

## 11. System Design

### 11.1 Микросервисная Архитектура

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   API        │     │   API        │     │   API        │
│   Gateway    │────▶│   Service A  │────▶│   Service B  │
└──────────────┘     └──────────────┘     └──────────────┘
       │                    │                     │
       │                    │                     │
       ▼                    ▼                     ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Service    │     │   Service    │     │   Service    │
│   Registry   │     │   Discovery  │     │   Config     │
└──────────────┘     └──────────────┘     └──────────────┘
       │
       ▼
┌──────────────┐
│   Message    │
│   Bus        │
│   (Kafka)    │
└──────────────┘
```

#### Service Communication

```scala
// HTTP4s Service
import org.http4s._
import org.http4s.dsl.io._
import org.http4s.client._
import org.http4s.circe.CirceEntityCodec._

class UserService(client: Client[IO]) {
  
  def getUser(id: UserId): IO[Option[User]] = {
    val uri = Uri.unsafeFromString(s"http://user-service/users/${id.value}")
    
    client.get(uri) {
      case Successful(response) => response.as[User].map(Some(_))
      case NotFound(_) => IO.pure(None)
      case response => 
        response.as[String].flatMap { body =>
          IO.raiseError(new Exception(s"Unexpected response: $body"))
        }
    }
  }
}

// gRPC Service
import io.grpc._
import scalapb.grpc.Grpc

class UserGrpcService extends UserServiceGrpc.UserService {
  
  def getUser(request: GetUserRequest): Future[GetUserResponse] = {
    // Implementation
    ???
  }
}

// Service with Circuit Breaker
import cats.effect.std.CircuitBreaker
import scala.concurrent.duration._

class ResilientUserService(
  client: Client[IO],
  circuitBreaker: CircuitBreaker[IO]
) {
  
  def getUser(id: UserId): IO[Option[User]] = {
    circuitBreaker.protect {
      client.get(Uri.unsafeFromString(s"http://user-service/users/${id.value}")) {
        case Successful(response) => response.as[User].map(Some(_))
        case NotFound(_) => IO.pure(None)
        case _ => IO.raiseError(new Exception("Service unavailable"))
      }
    }.handleErrorWith { error =>
      IO.println(s"Circuit breaker opened: ${error.getMessage}").as(None)
    }
  }
}
```

### 11.2 Event-Driven Architecture

```scala
// Event Bus
trait EventBus[F[_]] {
  def publish[E <: DomainEvent](event: E): F[Unit]
  def subscribe[E <: DomainEvent](handler: E => F[Unit]): F[Unit]
}

// Kafka Implementation
class KafkaEventBus(producer: KafkaProducer[IO]) extends EventBus[IO] {
  
  def publish[E <: DomainEvent](event: E): IO[Unit] = {
    val record = ProducerRecord(
      topic = event.getClass.getSimpleName,
      key = event.aggregateId.toString,
      value = event.toJson
    )
    producer.send(record).void
  }
  
  def subscribe[E <: DomainEvent](handler: E => IO[Unit]): IO[Unit] = {
    // Kafka consumer implementation
    ???
  }
}

// Saga Pattern for distributed transactions
sealed trait SagaStep[F[_]] {
  def execute: F[Unit]
  def compensate: F[Unit]
}

class OrderSaga[F[_]: Async](
  orderService: OrderService[F],
  paymentService: PaymentService[F],
  inventoryService: InventoryService[F]
) {
  
  def createOrder(userId: UserId, items: List[OrderItem]): F[Either[String, OrderId]] = {
    val steps = List(
      createOrderStep(userId, items),
      reserveInventoryStep(items),
      processPaymentStep(userId, items)
    )
    
    executeSaga(steps)
  }
  
  private def executeSaga(steps: List[SagaStep[F]]): F[Either[String, OrderId]] = {
    steps.foldLeft(Async[F].pure(List.empty[SagaStep[F]])) { (executed, step) =>
      for {
        completedSteps <- executed
        result <- step.execute.attempt
        _ <- result match {
          case Left(error) =>
            // Compensate all completed steps
            completedSteps.reverse.traverse_(_.compensate)
          case Right(_) =>
            Async[F].unit
        }
      } yield result match {
        case Right(_) => step :: completedSteps
        case Left(_) => completedSteps
      }
    }.flatMap { _ =>
      // Return result
      ???
    }
  }
}
```

### 11.3 Rate Limiting

```scala
import cats.effect.std.Semaphore

class RateLimiter[F[_]: Async](
  maxRequests: Int,
  window: FiniteDuration
) {
  private val semaphore: F[Semaphore[F]] = 
    Semaphore[F](maxRequests)
  
  def throttle[A](fa: F[A]): F[A] = {
    semaphore.flatMap { sem =>
      sem.permit.use { _ =>
        fa <* Async[F].sleep(window / maxRequests)
      }
    }
  }
}

// Token Bucket Implementation
class TokenBucket[F[_]: Async](
  capacity: Int,
  refillRate: Int,
  refillInterval: FiniteDuration
) {
  private val tokens: Ref[F, Int] = Ref.unsafe(capacity)
  
  private val refiller: F[Unit] = {
    (Async[F].sleep(refillInterval) *> tokens.update(t => math.min(capacity, t + refillRate)))
      .foreverM
      .start
      .void
  }
  
  def acquire: F[Boolean] = {
    tokens.modify { current =>
      if (current > 0) (current - 1, true)
      else (current, false)
    }
  }
  
  def throttle[A](fa: F[A]): F[A] = {
    acquire.flatMap {
      case true => fa
      case false => Async[F].raiseError(new Exception("Rate limit exceeded"))
    }
  }
}
```

---

## 12. Behavioral Questions

### 12.1 STAR Method

**S**ituation - контекст  
**T**ask - задача  
**A**ction - действие  
**R**esult - результат

### 12.2 Типичные вопросы и подготовка

**Technical Leadership:**
- Опишите ситуацию, когда вы должны были принять важное архитектурное решение
- Расскажите о случае, когда вы оптимизировали production систему
- Как вы помогаете junior разработчикам расти?

**Problem Solving:**
- Расскажите о сложной технической проблеме, которую вы решили
- Опишите случай, когда ваш первоначальный подход не сработал

**Collaboration:**
- Расскажите о конфликте в команде и как вы его разрешили
- Как вы проводите code review?

**Best Practices:**
- Как вы обеспечиваете качество кода?
- Опишите ваш подход к тестированию
- Как вы управляете техническим долгом?

### 12.3 Вопросы к интервьюеру

- Какие технологии используются в команде?
- Как организован процесс разработки?
- Какие вызовы стоят перед командой?
- Как выглядит типичный спринт?
- Возможности для роста и обучения?
- Процесс code review?
- Как измеряется success в роли?

---

## Финальные Рекомендации

### Подготовка к интервью:

1. **Code Practice** - решайте задачи на LeetCode/HackerRank на Scala
2. **System Design** - проектируйте системы, рисуйте диаграммы
3. **Live Coding** - практикуйтесь писать код на доске/экране
4. **Code Review** - читайте чужой код, давайте feedback
5. **Pet Projects** - создавайте проекты с современным стеком

### Ресурсы:

- **Книги:**
  - "Functional Programming in Scala" (Red Book)
  - "Scala with Cats"
  - "Practical FP in Scala"
  
- **Онлайн:**
  - Typelevel ecosystem documentation
  - Rock the JVM courses
  - Scala exercises

- **Community:**
  - Scala Discord/Gitter
  - ScalaDays talks
  - Local Scala meetups

### Ключевые моменты:

✓ Знание основ FP (монады, функторы, type classes)  
✓ Опыт с production effect systems  
✓ Понимание concurrency и streaming  
✓ Знание Scala 3 новшеств  
✓ System design и архитектурные паттерны  
✓ Testing best practices  
✓ Performance optimization  

**Удачи на интервью!** 🚀
