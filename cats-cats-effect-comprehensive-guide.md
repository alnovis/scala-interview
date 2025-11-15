# Полное Руководство по Cats и Cats Effect для Senior Scala Developer

## Оглавление

1. [План подготовки](#план-подготовки)
2. [Введение в Category Theory](#1-введение-в-category-theory)
3. [Cats Core - Type Classes](#2-cats-core---type-classes)
4. [Functors и Applicatives](#3-functors-и-applicatives)
5. [Monads](#4-monads)
6. [Monad Transformers](#5-monad-transformers)
7. [Validated и Parallel](#6-validated-и-parallel)
8. [Cats Effect - IO Monad](#7-cats-effect---io-monad)
9. [Concurrency в Cats Effect](#8-concurrency-в-cats-effect)
10. [Resource Management](#9-resource-management)
11. [Streaming с FS2](#10-streaming-с-fs2)
12. [Error Handling](#11-error-handling)
13. [Testing](#12-testing)
14. [Advanced Patterns](#13-advanced-patterns)
15. [Performance Optimization](#14-performance-optimization)

---

## План подготовки

### Неделя 1: Основы Category Theory и Type Classes
- День 1-2: Functor, Contravariant, Invariant
- День 3-4: Apply, Applicative
- День 5-6: Monad, FlatMap
- День 7: Практика - реализация собственных type classes

### Неделя 2: Продвинутые Type Classes
- День 1-2: Semigroup, Monoid
- День 3-4: Foldable, Traverse
- День 5-6: MonadError, ApplicativeError
- День 7: Практика - композиция type classes

### Неделя 3: Data Types
- День 1-2: Option, Either, Validated
- День 3-4: Writer, Reader, State
- День 5-6: Monad Transformers
- День 7: Практика - complex data pipelines

### Неделя 4: Cats Effect Basics
- День 1-2: IO Monad, defer, pure, delay
- День 3-4: Error handling, resource safety
- День 5-6: Fiber, concurrent operations
- День 7: Практика - async application

### Неделя 5: Concurrency
- День 1-2: Ref, Deferred, Semaphore
- День 3-4: Queue, Topic
- День 5-6: Concurrent patterns
- День 7: Практика - concurrent system

### Неделя 6: Resource Management
- День 1-2: Resource, bracket patterns
- День 3-4: Lifecycle management
- День 5-6: Database integration
- День 7: Практика - full-stack app

### Неделя 7: FS2 Streams
- День 1-2: Stream basics, transformations
- День 3-4: Concurrency, parallelism
- День 5-6: Resource management
- День 7: Практика - streaming pipeline

### Неделя 8: Advanced Topics
- День 1-3: Performance optimization
- День 4-5: Testing strategies
- День 6-7: Production patterns

---

## 1. Введение в Category Theory

### 1.1 Основы Category Theory

**Category Theory** - математическая теория, изучающая абстрактные структуры и отношения между ними.

```
Category состоит из:
┌─────────────────────────────────┐
│  1. Objects (типы)              │
│  2. Morphisms (функции)         │
│  3. Composition (композиция)    │
│  4. Identity (тождество)        │
└─────────────────────────────────┘
```

#### Законы Category

```scala
// 1. Associativity (ассоциативность)
// (f >>> g) >>> h == f >>> (g >>> h)

// 2. Identity (тождество)
// f >>> id == f
// id >>> f == f

// Пример в Scala
val f: Int => String = _.toString
val g: String => List[Char] = _.toList
val h: List[Char] => Int = _.length

// Associativity
val composed1 = (f andThen g) andThen h
val composed2 = f andThen (g andThen h)

composed1(42) == composed2(42) // true

// Identity
val identity: String => String = x => x
(f andThen identity)(42) == f(42) // true
(identity andThen g)("test") == g("test") // true
```

### 1.2 Functors

**Functor** - отображение между категориями, сохраняющее структуру.

```
┌──────────┐         map          ┌──────────┐
│  F[A]    │─────────────────────▶│  F[B]    │
└──────────┘                       └──────────┘
     │                                  │
     │ F                                │ F
     ▼                                  ▼
┌──────────┐         f            ┌──────────┐
│    A     │─────────────────────▶│    B     │
└──────────┘                       └──────────┘
```

```scala
// Functor laws
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
  
  // Laws:
  // 1. Identity: fa.map(identity) == fa
  // 2. Composition: fa.map(f).map(g) == fa.map(f andThen g)
}

// List Functor
implicit val listFunctor: Functor[List] = new Functor[List] {
  def map[A, B](fa: List[A])(f: A => B): List[B] = fa.map(f)
}

// Option Functor
implicit val optionFunctor: Functor[Option] = new Functor[Option] {
  def map[A, B](fa: Option[A])(f: A => B): Option[B] = fa.map(f)
}
```

### 1.3 Natural Transformations

**Natural Transformation** - трансформация между функторами.

```scala
// Natural Transformation: F[_] ~> G[_]
trait ~>[F[_], G[_]] {
  def apply[A](fa: F[A]): G[A]
}

// Пример: Option ~> List
val optionToList: Option ~> List = new (Option ~> List) {
  def apply[A](fa: Option[A]): List[A] = fa.toList
}

// Использование
optionToList(Some(42)) // List(42)
optionToList(None) // List()

// Пример: List ~> Option
val listToOption: List ~> Option = new (List ~> Option) {
  def apply[A](fa: List[A]): Option[A] = fa.headOption
}

listToOption(List(1, 2, 3)) // Some(1)
listToOption(Nil) // None
```

---

## 2. Cats Core - Type Classes

### 2.1 Semigroup

**Semigroup** - тип с ассоциативной бинарной операцией.

```scala
import cats._
import cats.implicits._

// Semigroup type class
trait Semigroup[A] {
  def combine(x: A, y: A): A
  
  // Law: Associativity
  // combine(combine(x, y), z) == combine(x, combine(y, z))
}

// Примеры встроенных Semigroup
val intAddition = Semigroup[Int] // сложение
intAddition.combine(1, 2) // 3

val stringConcat = Semigroup[String] // конкатенация
stringConcat.combine("Hello, ", "World!") // "Hello, World!"

val listConcat = Semigroup[List[Int]]
listConcat.combine(List(1, 2), List(3, 4)) // List(1, 2, 3, 4)

// Синтаксис
1 |+| 2 |+| 3 // 6
"Hello" |+| " " |+| "World" // "Hello World"
List(1, 2) |+| List(3, 4) // List(1, 2, 3, 4)

// Custom Semigroup
case class Money(amount: BigDecimal)

implicit val moneySemigroup: Semigroup[Money] = new Semigroup[Money] {
  def combine(x: Money, y: Money): Money = Money(x.amount + y.amount)
}

Money(100) |+| Money(50) // Money(150)

// Semigroup для Option
implicit def optionSemigroup[A: Semigroup]: Semigroup[Option[A]] = 
  new Semigroup[Option[A]] {
    def combine(x: Option[A], y: Option[A]): Option[A] = (x, y) match {
      case (Some(a), Some(b)) => Some(Semigroup[A].combine(a, b))
      case (Some(a), None) => Some(a)
      case (None, Some(b)) => Some(b)
      case (None, None) => None
    }
  }

Some(1) |+| Some(2) // Some(3)
Some(1) |+| None // Some(1)
```

### 2.2 Monoid

**Monoid** - Semigroup с единичным элементом (identity).

```scala
trait Monoid[A] extends Semigroup[A] {
  def empty: A
  
  // Laws:
  // 1. Associativity: combine(combine(x, y), z) == combine(x, combine(y, z))
  // 2. Left identity: combine(empty, x) == x
  // 3. Right identity: combine(x, empty) == x
}

// Примеры Monoid
Monoid[Int].empty // 0
Monoid[String].empty // ""
Monoid[List[Int]].empty // List()

// CombineAll
List(1, 2, 3, 4, 5).combineAll // 15
List("a", "b", "c").combineAll // "abc"

// FoldMap
case class Order(totalAmount: BigDecimal)

implicit val orderMonoid: Monoid[Order] = new Monoid[Order] {
  def empty: Order = Order(BigDecimal(0))
  def combine(x: Order, y: Order): Order = 
    Order(x.totalAmount + y.totalAmount)
}

val orders = List(Order(100), Order(250), Order(75))
orders.combineAll // Order(425)

// Map Monoid
implicit def mapMonoid[K, V: Semigroup]: Monoid[Map[K, V]] = 
  new Monoid[Map[K, V]] {
    def empty: Map[K, V] = Map.empty
    
    def combine(x: Map[K, V], y: Map[K, V]): Map[K, V] = {
      y.foldLeft(x) { case (acc, (k, v)) =>
        acc.updated(k, acc.get(k).fold(v)(Semigroup[V].combine(_, v)))
      }
    }
  }

Map("a" -> 1, "b" -> 2) |+| Map("b" -> 3, "c" -> 4)
// Map("a" -> 1, "b" -> 5, "c" -> 4)
```

### 2.3 Foldable

**Foldable** - структуры данных, которые можно свернуть.

```scala
import cats.Foldable

trait Foldable[F[_]] {
  def foldLeft[A, B](fa: F[A], b: B)(f: (B, A) => B): B
  def foldRight[A, B](fa: F[A], lb: Eval[B])(f: (A, Eval[B]) => Eval[B]): Eval[B]
}

// Примеры использования
val list = List(1, 2, 3, 4, 5)

// foldLeft
Foldable[List].foldLeft(list, 0)(_ + _) // 15

// foldRight (stack-safe через Eval)
Foldable[List].foldRight(list, Eval.now(0))((a, lb) => 
  lb.map(_ + a)
).value // 15

// combineAll
Foldable[List].combineAll(list) // 15

// foldMap
Foldable[List].foldMap(list)(_.toString) // "12345"

case class Person(name: String, age: Int)

val people = List(
  Person("Alice", 30),
  Person("Bob", 25),
  Person("Charlie", 35)
)

// find
Foldable[List].find(people)(_.age > 30) // Some(Person("Charlie", 35))

// exists
Foldable[List].exists(people)(_.age > 40) // false

// forall
Foldable[List].forall(people)(_.age > 20) // true

// size
Foldable[List].size(people) // 3

// isEmpty
Foldable[List].isEmpty(List.empty[Person]) // true

// nonEmpty
Foldable[List].nonEmpty(people) // true

// Custom Foldable
case class Tree[A](value: A, left: Option[Tree[A]], right: Option[Tree[A]])

implicit val treeFoldable: Foldable[Tree] = new Foldable[Tree] {
  def foldLeft[A, B](fa: Tree[A], b: B)(f: (B, A) => B): B = {
    val leftResult = fa.left.fold(b)(foldLeft(_, b)(f))
    val valueResult = f(leftResult, fa.value)
    fa.right.fold(valueResult)(foldLeft(_, valueResult)(f))
  }
  
  def foldRight[A, B](fa: Tree[A], lb: Eval[B])(f: (A, Eval[B]) => Eval[B]): Eval[B] = {
    val rightResult = fa.right.fold(lb)(foldRight(_, lb)(f))
    val valueResult = f(fa.value, rightResult)
    fa.left.fold(valueResult)(foldRight(_, valueResult)(f))
  }
}
```

### 2.4 Traverse

**Traverse** - комбинация map и sequence.

```scala
import cats.Traverse
import cats.syntax.traverse._

trait Traverse[F[_]] extends Functor[F] with Foldable[F] {
  def traverse[G[_]: Applicative, A, B](fa: F[A])(f: A => G[B]): G[F[B]]
  
  def sequence[G[_]: Applicative, A](fga: F[G[A]]): G[F[A]] =
    traverse(fga)(identity)
}

// Примеры
def validateAge(age: Int): Option[Int] =
  if (age >= 0 && age <= 120) Some(age) else None

// traverse
val ages = List(25, 30, 35)
ages.traverse(validateAge) // Some(List(25, 30, 35))

val invalidAges = List(25, -5, 35)
invalidAges.traverse(validateAge) // None

// sequence
val options: List[Option[Int]] = List(Some(1), Some(2), Some(3))
options.sequence // Some(List(1, 2, 3))

val optionsWithNone: List[Option[Int]] = List(Some(1), None, Some(3))
optionsWithNone.sequence // None

// Практический пример
case class User(id: Long, name: String, email: String)

def fetchUser(id: Long): Option[User] = {
  // Database lookup
  if (id > 0) Some(User(id, s"User-$id", s"user$id@example.com"))
  else None
}

val userIds = List(1L, 2L, 3L)
val users: Option[List[User]] = userIds.traverse(fetchUser)
// Some(List(User(1,...), User(2,...), User(3,...)))

// С Either
def parseAge(s: String): Either[String, Int] =
  try {
    Right(s.toInt)
  } catch {
    case _: NumberFormatException => Left(s"Invalid age: $s")
  }

val ageStrings = List("25", "30", "35")
ageStrings.traverse(parseAge) // Right(List(25, 30, 35))

val invalidAgeStrings = List("25", "invalid", "35")
invalidAgeStrings.traverse(parseAge) // Left("Invalid age: invalid")

// Parallel traverse
import cats.effect.IO
import cats.syntax.parallel._

def fetchUserAsync(id: Long): IO[User] = IO {
  Thread.sleep(100)
  User(id, s"User-$id", s"user$id@example.com")
}

// Sequential
userIds.traverse(fetchUserAsync) // выполняется последовательно

// Parallel
userIds.parTraverse(fetchUserAsync) // выполняется параллельно
```

---

## 3. Functors и Applicatives

### 3.1 Functor Deep Dive

```scala
import cats.Functor
import cats.syntax.functor._

// Covariant Functor
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
  
  // Derived methods
  def lift[A, B](f: A => B): F[A] => F[B] =
    fa => map(fa)(f)
  
  def as[A, B](fa: F[A], b: B): F[B] =
    map(fa)(_ => b)
  
  def void[A](fa: F[A]): F[Unit] =
    as(fa, ())
  
  def fproduct[A, B](fa: F[A])(f: A => B): F[(A, B)] =
    map(fa)(a => (a, f(a)))
  
  def tupleLeft[A, B](fa: F[A], b: B): F[(B, A)] =
    map(fa)(a => (b, a))
  
  def tupleRight[A, B](fa: F[A], b: B): F[(A, B)] =
    map(fa)(a => (a, b))
}

// Примеры использования derived methods
val list = List(1, 2, 3)

list.as(42) // List(42, 42, 42)
list.void // List((), (), ())
list.fproduct(_ * 2) // List((1,2), (2,4), (3,6))
list.tupleLeft("x") // List(("x",1), ("x",2), ("x",3))
list.tupleRight("x") // List((1,"x"), (2,"x"), (3,"x"))

// lift
val addOne: Int => Int = _ + 1
val liftedAddOne: List[Int] => List[Int] = Functor[List].lift(addOne)
liftedAddOne(List(1, 2, 3)) // List(2, 3, 4)

// Compose Functors
val listOption: List[Option[Int]] = List(Some(1), None, Some(3))

Functor[List].compose[Option].map(listOption)(_ * 2)
// List(Some(2), None, Some(6))

// Custom Functor
case class Box[A](value: A)

implicit val boxFunctor: Functor[Box] = new Functor[Box] {
  def map[A, B](fa: Box[A])(f: A => B): Box[B] = Box(f(fa.value))
}

Box(42).map(_ * 2) // Box(84)
```

### 3.2 Contravariant Functor

```scala
import cats.Contravariant

// Contravariant для "consumers"
trait Contravariant[F[_]] {
  def contramap[A, B](fa: F[A])(f: B => A): F[B]
}

// Пример: Encoder
trait Encoder[A] {
  def encode(value: A): String
}

object Encoder {
  def apply[A](implicit enc: Encoder[A]): Encoder[A] = enc
  
  implicit val contravariant: Contravariant[Encoder] = 
    new Contravariant[Encoder] {
      def contramap[A, B](fa: Encoder[A])(f: B => A): Encoder[B] =
        new Encoder[B] {
          def encode(value: B): String = fa.encode(f(value))
        }
    }
}

// Base encoder
implicit val intEncoder: Encoder[Int] = new Encoder[Int] {
  def encode(value: Int): String = value.toString
}

// Derived encoder через contramap
case class Money(amount: Int)

implicit val moneyEncoder: Encoder[Money] = 
  Encoder[Int].contramap[Money](_.amount)

moneyEncoder.encode(Money(100)) // "100"

// Пример: Show
import cats.Show
import cats.syntax.show._

case class Person(name: String, age: Int)

implicit val personShow: Show[Person] = Show.show { person =>
  s"${person.name} (${person.age} years old)"
}

Person("Alice", 30).show // "Alice (30 years old)"

// Contramap для Show
case class Employee(person: Person, department: String)

implicit val employeeShow: Show[Employee] = 
  Show[Person].contramap[Employee](_.person)

Employee(Person("Bob", 25), "IT").show // "Bob (25 years old)"
```

### 3.3 Invariant Functor

```scala
import cats.Invariant

// Invariant для "bidirectional" transformations
trait Invariant[F[_]] {
  def imap[A, B](fa: F[A])(f: A => B)(g: B => A): F[B]
}

// Пример: Codec (может кодировать и декодировать)
trait Codec[A] {
  def encode(value: A): String
  def decode(value: String): Option[A]
}

object Codec {
  implicit val invariant: Invariant[Codec] = new Invariant[Codec] {
    def imap[A, B](fa: Codec[A])(f: A => B)(g: B => A): Codec[B] =
      new Codec[B] {
        def encode(value: B): String = fa.encode(g(value))
        def decode(value: String): Option[B] = fa.decode(value).map(f)
      }
  }
}

// Int Codec
implicit val intCodec: Codec[Int] = new Codec[Int] {
  def encode(value: Int): String = value.toString
  def decode(value: String): Option[Int] = 
    try Some(value.toInt) catch { case _: NumberFormatException => None }
}

// Money Codec через imap
implicit val moneyCodec: Codec[Money] = 
  Codec.invariant.imap(intCodec)(Money(_))(_.amount)

moneyCodec.encode(Money(100)) // "100"
moneyCodec.decode("200") // Some(Money(200))
```

### 3.4 Apply и Applicative

```scala
import cats.Apply
import cats.Applicative
import cats.syntax.apply._

// Apply - Apply transformations in sequence
trait Apply[F[_]] extends Functor[F] {
  def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
  
  // Derived methods
  def map2[A, B, Z](fa: F[A], fb: F[B])(f: (A, B) => Z): F[Z]
  def product[A, B](fa: F[A], fb: F[B]): F[(A, B)]
}

// Applicative - Apply с pure
trait Applicative[F[_]] extends Apply[F] {
  def pure[A](a: A): F[A]
}

// Примеры
import cats.instances.option._

val option1 = Some(3)
val option2 = Some(5)
val option3 = Some(7)

// map2
(option1, option2).mapN(_ + _) // Some(8)

// map3
(option1, option2, option3).mapN(_ + _ + _) // Some(15)

// tupled
(option1, option2, option3).tupled // Some((3, 5, 7))

// Практический пример - валидация формы
case class UserForm(name: String, age: String, email: String)
case class ValidatedUser(name: String, age: Int, email: String)

def validateName(name: String): Option[String] =
  if (name.nonEmpty) Some(name) else None

def validateAge(age: String): Option[Int] =
  try {
    val parsed = age.toInt
    if (parsed > 0 && parsed < 120) Some(parsed) else None
  } catch {
    case _: NumberFormatException => None
  }

def validateEmail(email: String): Option[String] =
  if (email.contains("@")) Some(email) else None

def validateUser(form: UserForm): Option[ValidatedUser] =
  (
    validateName(form.name),
    validateAge(form.age),
    validateEmail(form.email)
  ).mapN(ValidatedUser)

validateUser(UserForm("Alice", "30", "alice@example.com"))
// Some(ValidatedUser("Alice", 30, "alice@example.com"))

validateUser(UserForm("", "30", "alice@example.com"))
// None

// Applicative Builder Pattern
import cats.syntax.applicative._

42.pure[Option] // Some(42)
42.pure[List] // List(42)
42.pure[Either[String, *]] // Right(42)

// ap - применение функции в контексте
val funcOption: Option[Int => String] = Some(_.toString)
val valueOption: Option[Int] = Some(42)

funcOption.ap(valueOption) // Some("42")

// Последовательное vs Независимое выполнение
def fetchUser(id: Long): Option[User] = {
  println(s"Fetching user $id")
  Some(User(id, s"User-$id", s"user$id@example.com"))
}

// Монадическое (последовательное)
for {
  user1 <- fetchUser(1)
  user2 <- fetchUser(2)
} yield (user1, user2)
// Выводит:
// Fetching user 1
// Fetching user 2

// Аппликативное (может быть параллельным)
(fetchUser(1), fetchUser(2)).tupled
```

---

## 4. Monads

### 4.1 Monad Basics

```scala
import cats.Monad
import cats.syntax.flatMap._
import cats.syntax.functor._

trait Monad[F[_]] extends Applicative[F] {
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  
  // Derived from flatMap and pure
  def flatten[A](ffa: F[F[A]]): F[A] = flatMap(ffa)(identity)
  
  // Laws:
  // 1. Left identity: pure(a).flatMap(f) == f(a)
  // 2. Right identity: fa.flatMap(pure) == fa
  // 3. Associativity: fa.flatMap(f).flatMap(g) == fa.flatMap(a => f(a).flatMap(g))
}

// List Monad
val result = for {
  x <- List(1, 2, 3)
  y <- List(10, 20)
} yield x * y
// List(10, 20, 20, 40, 30, 60)

// Option Monad
def divide(a: Int, b: Int): Option[Int] =
  if (b != 0) Some(a / b) else None

val computation = for {
  x <- Some(10)
  y <- Some(2)
  result <- divide(x, y)
} yield result
// Some(5)

// Either Monad
type Result[A] = Either[String, A]

def validateAge(age: Int): Result[Int] =
  if (age >= 0 && age <= 120) Right(age)
  else Left("Invalid age")

def validateName(name: String): Result[String] =
  if (name.nonEmpty) Right(name)
  else Left("Name cannot be empty")

val validation = for {
  age <- validateAge(25)
  name <- validateName("Alice")
} yield (name, age)
// Right(("Alice", 25))

// Custom Monad
case class State[S, A](run: S => (S, A))

implicit def stateMonad[S]: Monad[State[S, *]] = new Monad[State[S, *]] {
  def pure[A](a: A): State[S, A] = State(s => (s, a))
  
  def flatMap[A, B](fa: State[S, A])(f: A => State[S, B]): State[S, B] =
    State { s =>
      val (s1, a) = fa.run(s)
      f(a).run(s1)
    }
  
  def tailRecM[A, B](a: A)(f: A => State[S, Either[A, B]]): State[S, B] =
    State { s =>
      @annotation.tailrec
      def loop(state: S, current: A): (S, B) = {
        val (nextState, result) = f(current).run(state)
        result match {
          case Left(nextA) => loop(nextState, nextA)
          case Right(b) => (nextState, b)
        }
      }
      loop(s, a)
    }
}
```

### 4.2 MonadError

```scala
import cats.MonadError

trait MonadError[F[_], E] extends Monad[F] {
  def raiseError[A](e: E): F[A]
  def handleErrorWith[A](fa: F[A])(f: E => F[A]): F[A]
  
  // Derived methods
  def handleError[A](fa: F[A])(f: E => A): F[A] =
    handleErrorWith(fa)(e => pure(f(e)))
  
  def attempt[A](fa: F[A]): F[Either[E, A]]
  def ensure[A](fa: F[A])(e: E)(p: A => Boolean): F[A]
}

// Either MonadError
type ErrorOr[A] = Either[String, A]

val me = MonadError[ErrorOr, String]

// raiseError
me.raiseError[Int]("Error!") // Left("Error!")

// handleError
val result: ErrorOr[Int] = Left("error")
me.handleError(result)(_ => 0) // Right(0)

// ensure
me.ensure(Right(5): ErrorOr[Int])("Value must be > 10")(_ > 10)
// Left("Value must be > 10")

// attempt
me.attempt(Right(42): ErrorOr[Int]) // Right(Right(42))
me.attempt(Left("error"): ErrorOr[Int]) // Right(Left("error"))

// Практический пример
case class User(id: Long, name: String, age: Int)

sealed trait ValidationError
case class InvalidAge(age: Int) extends ValidationError
case class InvalidName(name: String) extends ValidationError
case class DatabaseError(message: String) extends ValidationError

type Validated[A] = Either[ValidationError, A]

def validateUser(id: Long, name: String, age: Int): Validated[User] = {
  val me = MonadError[Validated, ValidationError]
  
  for {
    validAge <- me.ensure(
      me.pure(age)
    )(InvalidAge(age))(a => a > 0 && a < 120)
    
    validName <- me.ensure(
      me.pure(name)
    )(InvalidName(name))(_.nonEmpty)
    
    user <- me.pure(User(id, validName, validAge))
  } yield user
}

validateUser(1, "Alice", 30) // Right(User(1, "Alice", 30))
validateUser(1, "", 30) // Left(InvalidName(""))
validateUser(1, "Bob", -5) // Left(InvalidAge(-5))

// Recovery
def getUserWithFallback(id: Long): Validated[User] = {
  val me = MonadError[Validated, ValidationError]
  
  me.handleErrorWith(getUser(id)) {
    case DatabaseError(_) => me.pure(User(0, "Guest", 0))
    case other => me.raiseError(other)
  }
}
```

### 4.3 Monad Combinators

```scala
import cats.Monad
import cats.syntax.flatMap._
import cats.syntax.functor._

// ifM - conditional в монадическом контексте
def ifM[F[_]: Monad, A](
  condition: F[Boolean]
)(ifTrue: F[A], ifFalse: F[A]): F[A] = {
  condition.flatMap { cond =>
    if (cond) ifTrue else ifFalse
  }
}

// Пример
def isEven(n: Int): Option[Boolean] = Some(n % 2 == 0)

ifM(isEven(4))(
  ifTrue = Some("Even"),
  ifFalse = Some("Odd")
) // Some("Even")

// whileM - цикл while в монадическом контексте
def whileM[F[_]: Monad](
  condition: F[Boolean]
)(body: F[Unit]): F[Unit] = {
  condition.flatMap { cond =>
    if (cond) body >> whileM(condition)(body)
    else Monad[F].unit
  }
}

// untilM - цикл do-while
def untilM[F[_]: Monad](
  body: F[Unit]
)(condition: F[Boolean]): F[Unit] = {
  body >> condition.flatMap { cond =>
    if (cond) Monad[F].unit
    else untilM(body)(condition)
  }
}

// iterateWhile - итерация с условием
def processUntilValid(input: String): Option[String] = {
  Some(input).iterateWhile(s => s.length < 10)(s => s + "x")
}

processUntilValid("test") // Some("testxxxxxx")

// iterateUntil - итерация до условия
Some(1).iterateUntil(_ > 100)(_ * 2) // Some(128)

// foreverM - бесконечный цикл
import cats.effect.IO

def printForever: IO[Unit] = 
  IO.println("Hello").foreverM

// replicateA - повторить N раз
List(1, 2, 3).replicateA(2) // List(List(1,1), List(1,2), ...)
```

---

## 5. Monad Transformers

### 5.1 OptionT

```scala
import cats.data.OptionT
import cats.effect.IO
import cats.syntax.applicative._

// OptionT[F[_], A] is equivalent to F[Option[A]]
type AsyncOption[A] = OptionT[IO, A]

def findUser(id: Long): IO[Option[User]] = IO {
  if (id > 0) Some(User(id, s"User-$id", s"user$id@example.com"))
  else None
}

def findOrders(userId: Long): IO[Option[List[Order]]] = IO {
  if (userId > 0) Some(List(Order(1, userId, BigDecimal(100))))
  else None
}

// Без OptionT (неудобно)
def getUserOrders1(userId: Long): IO[Option[List[Order]]] = {
  findUser(userId).flatMap {
    case Some(user) => findOrders(user.id)
    case None => IO.pure(None)
  }
}

// С OptionT (элегантно)
def getUserOrders2(userId: Long): OptionT[IO, List[Order]] = {
  for {
    user <- OptionT(findUser(userId))
    orders <- OptionT(findOrders(user.id))
  } yield orders
}

// Использование
val result: IO[Option[List[Order]]] = getUserOrders2(1).value

// OptionT constructors
OptionT.pure[IO](42) // OptionT(IO(Some(42)))
OptionT.none[IO, Int] // OptionT(IO(None))
OptionT.fromOption[IO](Some(42)) // OptionT(IO(Some(42)))
OptionT.liftF(IO(42)) // OptionT(IO(Some(42)))

// OptionT operations
val opt1: OptionT[IO, Int] = OptionT.pure[IO](10)
val opt2: OptionT[IO, Int] = OptionT.pure[IO](20)

// map
opt1.map(_ * 2) // OptionT(IO(Some(20)))

// flatMap
opt1.flatMap(x => OptionT.pure[IO](x + 5)) // OptionT(IO(Some(15)))

// getOrElse
opt1.getOrElse(0) // IO(10)
OptionT.none[IO, Int].getOrElse(0) // IO(0)

// getOrElseF
OptionT.none[IO, Int].getOrElseF(IO(42)) // IO(42)

// orElse
OptionT.none[IO, Int].orElse(OptionT.pure[IO](42)) // OptionT(IO(Some(42)))

// fold
opt1.fold(0)(x => x * 2) // IO(20)

// subflatMap
opt1.subflatMap(x => if (x > 5) Some(x) else None) // OptionT(IO(Some(10)))
```

### 5.2 EitherT

```scala
import cats.data.EitherT

// EitherT[F[_], E, A] is equivalent to F[Either[E, A]]
type AsyncEither[A] = EitherT[IO, String, A]

def validateAge(age: Int): IO[Either[String, Int]] = IO {
  if (age >= 0 && age <= 120) Right(age)
  else Left("Invalid age")
}

def validateName(name: String): IO[Either[String, String]] = IO {
  if (name.nonEmpty) Right(name)
  else Left("Name cannot be empty")
}

// С EitherT
def validateUser(name: String, age: Int): EitherT[IO, String, (String, Int)] = {
  for {
    validName <- EitherT(validateName(name))
    validAge <- EitherT(validateAge(age))
  } yield (validName, validAge)
}

// Использование
val result: IO[Either[String, (String, Int)]] = 
  validateUser("Alice", 30).value

// EitherT constructors
EitherT.pure[IO, String](42) // EitherT(IO(Right(42)))
EitherT.leftT[IO, Int]("error") // EitherT(IO(Left("error")))
EitherT.rightT[IO, String](42) // EitherT(IO(Right(42)))
EitherT.fromEither[IO](Right(42): Either[String, Int]) // EitherT(IO(Right(42)))
EitherT.liftF(IO(42)) // EitherT(IO(Right(42)))

// EitherT operations
val either1: EitherT[IO, String, Int] = EitherT.pure[IO, String](10)
val either2: EitherT[IO, String, Int] = EitherT.pure[IO, String](20)

// map
either1.map(_ * 2) // EitherT(IO(Right(20)))

// leftMap
either1.leftMap(_.toUpperCase) // transform error

// bimap
either1.bimap(_.toUpperCase, _ * 2) // transform both sides

// flatMap
either1.flatMap(x => EitherT.pure[IO, String](x + 5))

// getOrElse
either1.getOrElse(0) // IO(10)

// getOrElseF
EitherT.leftT[IO, Int]("error").getOrElseF(IO(42)) // IO(42)

// orElse
EitherT.leftT[IO, Int]("error").orElse(EitherT.pure[IO, String](42))

// fold
either1.fold(err => 0, x => x * 2) // IO(20)

// swap
EitherT.leftT[IO, Int]("error").swap // EitherT(IO(Right("error")))

// Практический пример - композиция сервисов
case class DatabaseError(message: String)
case class ValidationError(message: String)

type ServiceResult[A] = EitherT[IO, String, A]

def findUserInDb(id: Long): IO[Either[String, User]] = IO {
  if (id > 0) Right(User(id, s"User-$id", s"user$id@example.com"))
  else Left("User not found")
}

def checkPermissions(user: User): IO[Either[String, Unit]] = IO {
  if (user.id % 2 == 0) Right(())
  else Left("Access denied")
}

def performAction(user: User): IO[Either[String, String]] = IO {
  Right(s"Action performed for ${user.name}")
}

def secureAction(userId: Long): ServiceResult[String] = {
  for {
    user <- EitherT(findUserInDb(userId))
    _ <- EitherT(checkPermissions(user))
    result <- EitherT(performAction(user))
  } yield result
}
```

### 5.3 ReaderT

```scala
import cats.data.ReaderT

// ReaderT[F[_], E, A] is equivalent to E => F[A]
case class Config(dbUrl: String, apiKey: String)

type ConfigReader[A] = ReaderT[IO, Config, A]

def getDbConnection: ConfigReader[String] = ReaderT { config =>
  IO(s"Connected to ${config.dbUrl}")
}

def callApi: ConfigReader[String] = ReaderT { config =>
  IO(s"API call with key: ${config.apiKey}")
}

def program: ConfigReader[String] = {
  for {
    db <- getDbConnection
    api <- callApi
  } yield s"$db, $api"
}

// Использование
val config = Config("localhost:5432", "secret-key")
program.run(config) // IO("Connected to localhost:5432, API call with key: secret-key")

// ReaderT constructors
ReaderT.pure[IO, Config](42) // ReaderT(_ => IO(42))
ReaderT.liftF(IO(42)) // ReaderT(_ => IO(42))
ReaderT.ask[IO, Config] // ReaderT(config => IO(config))

// Operations
val reader: ConfigReader[Int] = ReaderT.pure[IO, Config](10)

// map
reader.map(_ * 2)

// flatMap
reader.flatMap(x => ReaderT.pure[IO, Config](x + 5))

// local - transform environment
reader.local[Config](config => 
  config.copy(dbUrl = "modified-url")
)
```

### 5.4 WriterT

```scala
import cats.data.WriterT

// WriterT[F[_], L, A] is equivalent to F[(L, A)]
type Logged[A] = WriterT[IO, List[String], A]

def logOperation(name: String, value: Int): Logged[Int] = {
  WriterT.put(IO(value))(List(s"Operation $name: $value"))
}

def program: Logged[Int] = {
  for {
    a <- logOperation("step1", 10)
    b <- logOperation("step2", 20)
    c <- logOperation("step3", a + b)
  } yield c
}

// Использование
program.run // IO((List("Operation step1: 10", "Operation step2: 20", "Operation step3: 30"), 30))

// WriterT constructors
WriterT.value[IO, List[String]](42) // WriterT(IO((List(), 42)))
WriterT.tell[IO](List("log entry")) // WriterT(IO((List("log entry"), ())))
WriterT.put(IO(42))(List("computed 42")) // WriterT(IO((List("computed 42"), 42)))

// Operations
val writer: Logged[Int] = WriterT.put(IO(10))(List("initial"))

// map
writer.map(_ * 2)

// flatMap
writer.flatMap(x => WriterT.put(IO(x + 5))(List("added 5")))

// mapWritten - transform log
writer.mapWritten(_.map(_.toUpperCase))

// mapBoth - transform both
writer.mapBoth { (log, value) =>
  (log :+ "processed", value * 2)
}

// reset - clear log
writer.reset // WriterT(IO((List(), 10)))
```

### 5.5 StateT

```scala
import cats.data.StateT

// StateT[F[_], S, A] is equivalent to S => F[(S, A)]
type StatefulIO[A] = StateT[IO, Int, A]

def increment: StatefulIO[Unit] = StateT.modify[IO, Int](_ + 1)
def get: StatefulIO[Int] = StateT.get[IO, Int]
def set(value: Int): StatefulIO[Unit] = StateT.set[IO, Int](value)

def program: StatefulIO[String] = {
  for {
    _ <- increment
    _ <- increment
    value <- get
    _ <- set(100)
    finalValue <- get
  } yield s"Value was $value, now is $finalValue"
}

// Использование
program.run(0) // IO((100, "Value was 2, now is 100"))

// StateT constructors
StateT.pure[IO, Int, String]("hello") // StateT(s => IO((s, "hello")))
StateT.get[IO, Int] // StateT(s => IO((s, s)))
StateT.set[IO, Int](42) // StateT(_ => IO((42, ())))
StateT.modify[IO, Int](_ + 1) // StateT(s => IO((s + 1, ())))
StateT.liftF(IO("hello")) // StateT(s => IO("hello").map((s, _)))

// Практический пример - генератор ID
case class IdGenerator(nextId: Long)

type IdGenIO[A] = StateT[IO, IdGenerator, A]

def generateId: IdGenIO[Long] = StateT { gen =>
  IO.pure((IdGenerator(gen.nextId + 1), gen.nextId))
}

def createUsers(names: List[String]): IdGenIO[List[User]] = {
  names.traverse { name =>
    for {
      id <- generateId
    } yield User(id, name, s"$name@example.com")
  }
}

// Использование
val names = List("Alice", "Bob", "Charlie")
createUsers(names).run(IdGenerator(1))
// IO((IdGenerator(4), List(User(1,"Alice",...), User(2,"Bob",...), User(3,"Charlie",...))))
```

### 5.6 Nested Transformers

```scala
// Комбинация нескольких трансформеров
type Stack[A] = EitherT[OptionT[IO, *], String, A]

def stackExample: Stack[Int] = {
  for {
    a <- EitherT.pure[OptionT[IO, *], String](10)
    b <- EitherT.pure[OptionT[IO, *], String](20)
  } yield a + b
}

// Использование
val result: OptionT[IO, Either[String, Int]] = stackExample.value
val finalResult: IO[Option[Either[String, Int]]] = result.value

// Практический пример - real application stack
type AppStack[A] = ReaderT[EitherT[IO, AppError, *], AppContext, A]

case class AppContext(config: Config, db: Database)
sealed trait AppError
case class ValidationError(msg: String) extends AppError
case class DbError(msg: String) extends AppError

def getUser(id: Long): AppStack[User] = ReaderT { ctx =>
  EitherT {
    ctx.db.findUser(id).map {
      case Some(user) => Right(user)
      case None => Left(DbError(s"User $id not found"))
    }
  }
}

def validateAge(age: Int): AppStack[Int] = ReaderT { _ =>
  EitherT.fromEither[IO] {
    if (age >= 0 && age <= 120) Right(age)
    else Left(ValidationError("Invalid age"))
  }
}

def application(userId: Long, age: Int): AppStack[String] = {
  for {
    user <- getUser(userId)
    validAge <- validateAge(age)
  } yield s"User: ${user.name}, Age: $validAge"
}

// Запуск
val ctx = AppContext(config, database)
val result: EitherT[IO, AppError, String] = application(1, 30).run(ctx)
val finalResult: IO[Either[AppError, String]] = result.value
```

---

## 6. Validated и Parallel

### 6.1 Validated

```scala
import cats.data.Validated
import cats.data.Validated.{Valid, Invalid}
import cats.syntax.validated._
import cats.syntax.apply._

// Validated accumulates errors (в отличие от Either)
type ValidationResult[A] = Validated[List[String], A]

def validateName(name: String): ValidationResult[String] =
  if (name.nonEmpty) name.valid
  else List("Name cannot be empty").invalid

def validateAge(age: Int): ValidationResult[Int] =
  if (age >= 0 && age <= 120) age.valid
  else List("Age must be between 0 and 120").invalid

def validateEmail(email: String): ValidationResult[String] =
  if (email.contains("@")) email.valid
  else List("Email must contain @").invalid

// С Either (останавливается на первой ошибке)
def validateWithEither(name: String, age: Int, email: String): Either[String, User] = {
  for {
    n <- if (name.nonEmpty) Right(name) else Left("Name error")
    a <- if (age >= 0) Right(age) else Left("Age error")
    e <- if (email.contains("@")) Right(email) else Left("Email error")
  } yield User(0, n, e)
}

validateWithEither("", -5, "invalid")
// Left("Name error") - только первая ошибка!

// С Validated (собирает все ошибки)
def validateUser(name: String, age: Int, email: String): ValidationResult[User] = {
  (
    validateName(name),
    validateAge(age),
    validateEmail(email)
  ).mapN((n, a, e) => User(0, n, e))
}

validateUser("", -5, "invalid")
// Invalid(List("Name cannot be empty", "Age must be between 0 and 120", "Email must contain @"))

// ValidatedNec - Validated with NonEmptyChain
import cats.data.ValidatedNec

type ValidationNec[A] = ValidatedNec[String, A]

def validateNameNec(name: String): ValidationNec[String] =
  if (name.nonEmpty) name.validNec
  else "Name cannot be empty".invalidNec

def validateAgeNec(age: Int): ValidationNec[Int] =
  if (age >= 0 && age <= 120) age.validNec
  else "Age must be between 0 and 120".invalidNec

def validateEmailNec(email: String): ValidationNec[String] =
  if (email.contains("@")) email.validNec
  else "Email must contain @".invalidNec

def validateUserNec(name: String, age: Int, email: String): ValidationNec[User] = {
  (
    validateNameNec(name),
    validateAgeNec(age),
    validateEmailNec(email)
  ).mapN((n, a, e) => User(0, n, e))
}

// Operations
val valid: Validated[String, Int] = 42.valid
val invalid: Validated[String, Int] = "error".invalid

// map
valid.map(_ * 2) // Valid(84)

// leftMap
invalid.leftMap(_.toUpperCase) // Invalid("ERROR")

// bimap
valid.bimap(_.toUpperCase, _ * 2) // Valid(84)

// fold
valid.fold(err => 0, x => x * 2) // 84

// toEither
valid.toEither // Right(42)
invalid.toEither // Left("error")

// toOption
valid.toOption // Some(42)
invalid.toOption // None

// andThen - преобразование в другой Validated (fails fast)
valid.andThen(x => 
  if (x > 40) (x * 2).valid 
  else "too small".invalid
) // Valid(84)

// orElse
invalid.orElse(100.valid) // Valid(100)

// ensure
valid.ensure(List("must be even"))(_ % 2 == 0) // Valid(42)

// Conversion between Either and Validated
val either: Either[String, Int] = Right(42)
Validated.fromEither(either) // Valid(42)

val validated: Validated[String, Int] = 42.valid
validated.toEither // Right(42)
```

### 6.2 Parallel

```scala
import cats.Parallel
import cats.syntax.parallel._

// Parallel execution with Applicative
val io1 = IO.sleep(1.second) >> IO.pure(10)
val io2 = IO.sleep(1.second) >> IO.pure(20)
val io3 = IO.sleep(1.second) >> IO.pure(30)

// Sequential (takes 3 seconds)
val sequential = for {
  a <- io1
  b <- io2
  c <- io3
} yield a + b + c

// Parallel (takes 1 second)
val parallel = (io1, io2, io3).parMapN(_ + _ + _)

// parTraverse
val ids = List(1L, 2L, 3L, 4L, 5L)

def fetchUser(id: Long): IO[User] = IO.sleep(100.millis) >> IO.pure(
  User(id, s"User-$id", s"user$id@example.com")
)

// Sequential traverse
ids.traverse(fetchUser) // takes 500ms

// Parallel traverse
ids.parTraverse(fetchUser) // takes ~100ms

// parSequence
val ios: List[IO[Int]] = List(
  IO.sleep(100.millis) >> IO.pure(1),
  IO.sleep(100.millis) >> IO.pure(2),
  IO.sleep(100.millis) >> IO.pure(3)
)

ios.sequence // sequential, takes 300ms
ios.parSequence // parallel, takes ~100ms

// Validated with Parallel
import cats.data.NonEmptyList

def validateUserPar(name: String, age: Int, email: String): IO[ValidatedNec[String, User]] = {
  val validations = List(
    IO(validateNameNec(name)),
    IO(validateAgeNec(age)),
    IO(validateEmailNec(email))
  )
  
  validations.parSequence.map { results =>
    (results(0), results(1), results(2)).mapN((n, a, e) => User(0, n, e))
  }
}

// parFlatTraverse
val nested: List[IO[List[Int]]] = List(
  IO.pure(List(1, 2, 3)),
  IO.pure(List(4, 5, 6))
)

nested.parFlatTraverse(identity) // IO(List(1, 2, 3, 4, 5, 6))

// Racing computations
val fast = IO.sleep(100.millis) >> IO.pure("fast")
val slow = IO.sleep(1.second) >> IO.pure("slow")

IO.race(fast, slow) // IO(Left("fast"))
```

---

## 7. Cats Effect - IO Monad

### 7.1 IO Basics

```scala
import cats.effect.IO
import cats.effect.unsafe.implicits.global

// Creating IO
val pure = IO.pure(42) // уже вычисленное значение
val delay = IO.delay { // вычисляется при запуске
  println("Computing...")
  42
}
val apply = IO(42) // alias для delay
val async = IO.async_[Int] { cb => // асинхронное вычисление
  scala.concurrent.ExecutionContext.global.execute(() => {
    Thread.sleep(1000)
    cb(Right(42))
  })
}

// Запуск IO
pure.unsafeRunSync() // блокирует поток
delay.unsafeRunAsync(result => println(result)) // неблокирующий

// IO operations
val io = IO(42)

// map
io.map(_ * 2) // IO(84)

// flatMap
io.flatMap(x => IO(x * 2)) // IO(84)

// as
io.as("result") // IO("result")

// void
io.void // IO(())

// attempt - error handling
IO.raiseError(new Exception("error")).attempt // IO(Left(Exception))

// handleError
IO.raiseError[Int](new Exception("error")).handleError(_ => 0) // IO(0)

// redeem - комбинация map и handleError
io.redeem(
  err => 0,
  value => value * 2
) // IO(84)

// redeemWith - монадический redeem
io.redeemWith(
  err => IO.pure(0),
  value => IO.pure(value * 2)
) // IO(84)

// Комбинирование IO
val io1 = IO(println("First"))
val io2 = IO(println("Second"))

// >> - sequential execution, ignore first result
io1 >> io2 // prints "First" then "Second"

// *> - alias for >>
io1 *> io2

// <* - execute both, return first result
io1 <* io2

// Практический пример
def fetchFromApi(url: String): IO[String] = IO {
  println(s"Fetching from $url")
  Thread.sleep(1000)
  s"Data from $url"
}

def saveToDb(data: String): IO[Unit] = IO {
  println(s"Saving to DB: $data")
}

val program = for {
  data <- fetchFromApi("https://api.example.com")
  _ <- saveToDb(data)
  _ <- IO.println("Done!")
} yield ()

program.unsafeRunSync()
```

### 7.2 Error Handling

```scala
import cats.effect.IO

// raiseError
val error: IO[Int] = IO.raiseError(new Exception("Error!"))

// raiseWhen / raiseUnless
IO.raiseWhen(true)(new Exception("Condition met")) // IO(())
IO.raiseUnless(false)(new Exception("Condition not met")) // IO(())

// attempt
error.attempt // IO(Left(Exception("Error!")))

// handleError
error.handleError(ex => 0) // IO(0)

// handleErrorWith
error.handleErrorWith(ex => IO.pure(0)) // IO(0)

// recover (partial function)
error.recover {
  case _: IllegalArgumentException => 0
  case _: NullPointerException => 1
} // may still fail if doesn't match

// recoverWith
error.recoverWith {
  case ex: Exception => IO.pure(0)
}

// onError - выполнить side effect при ошибке
error.onError {
  case ex => IO.println(s"Error occurred: ${ex.getMessage}")
}

// Retry logic
import scala.concurrent.duration._

def unreliableOperation: IO[String] = IO {
  if (scala.util.Random.nextBoolean()) "success"
  else throw new Exception("Failed")
}

// Simple retry
def retry[A](io: IO[A], times: Int): IO[A] = {
  io.handleErrorWith { err =>
    if (times > 0) retry(io, times - 1)
    else IO.raiseError(err)
  }
}

retry(unreliableOperation, 3)

// Exponential backoff retry
def retryWithBackoff[A](
  io: IO[A],
  initialDelay: FiniteDuration,
  maxRetries: Int
): IO[A] = {
  def loop(remaining: Int, delay: FiniteDuration): IO[A] = {
    io.handleErrorWith { err =>
      if (remaining > 0) {
        IO.sleep(delay) >> loop(remaining - 1, delay * 2)
      } else {
        IO.raiseError(err)
      }
    }
  }
  loop(maxRetries, initialDelay)
}

retryWithBackoff(unreliableOperation, 100.millis, 5)

// Timeout
val slowOperation = IO.sleep(10.seconds) >> IO.pure("done")

slowOperation.timeout(1.second) // fails with TimeoutException

// timeoutTo - with fallback
slowOperation.timeoutTo(1.second, IO.pure("fallback"))

// Guarantee - всегда выполняется (даже при ошибке)
val withGuarantee = IO.println("Main operation")
  .guarantee(IO.println("Cleanup"))

// guaranteeCase - с информацией о завершении
import cats.effect.kernel.Outcome

IO.println("Main").guaranteeCase {
  case Outcome.Succeeded(_) => IO.println("Success")
  case Outcome.Errored(e) => IO.println(s"Error: $e")
  case Outcome.Canceled() => IO.println("Canceled")
}

// bracketCase - для ресурсов
IO(openConnection).bracketCase { conn =>
  useConnection(conn)
} {
  case (conn, Outcome.Succeeded(_)) => IO(conn.close())
  case (conn, Outcome.Errored(_)) => IO(conn.forceClose())
  case (conn, Outcome.Canceled()) => IO(conn.forceClose())
}
```

### 7.3 Cancelation

```scala
import cats.effect.IO
import scala.concurrent.duration._

// Cancelable IO
val cancelable = for {
  _ <- IO.println("Starting...")
  _ <- IO.sleep(5.seconds)
  _ <- IO.println("Done!")
} yield ()

// Cancel after 1 second
val withCancel = cancelable.timeout(1.second)
  .handleError(_ => println("Canceled!"))

// uncancelable - защита от отмены
val uncancelable = IO.uncancelable { poll =>
  for {
    _ <- IO.println("This cannot be canceled")
    _ <- poll(IO.sleep(1.second)) // но это может быть отменено
    _ <- IO.println("This also cannot be canceled")
  } yield ()
}

// onCancel - выполнить при отмене
val withOnCancel = IO.sleep(5.seconds)
  .onCancel(IO.println("Canceled!"))

// Практический пример - cancelable background task
def backgroundTask: IO[Unit] = {
  def loop(n: Int): IO[Unit] = {
    IO.println(s"Working... $n") >> 
    IO.sleep(1.second) >> 
    loop(n + 1)
  }
  loop(0).onCancel(IO.println("Background task canceled"))
}

val program = for {
  fiber <- backgroundTask.start
  _ <- IO.sleep(5.seconds)
  _ <- fiber.cancel
  _ <- IO.println("Main program done")
} yield ()
```

### 7.4 IOApp

```scala
import cats.effect.{IO, IOApp, ExitCode}

// Simple IOApp
object MyApp extends IOApp {
  def run(args: List[String]): IO[ExitCode] = {
    val program = for {
      _ <- IO.println("Hello, World!")
      _ <- IO.println(s"Args: ${args.mkString(", ")}")
    } yield ()
    
    program.as(ExitCode.Success)
  }
}

// IOApp.Simple - без ExitCode
object SimpleApp extends IOApp.Simple {
  def run: IO[Unit] = {
    IO.println("Hello, World!")
  }
}

// С обработкой ошибок
object RobustApp extends IOApp {
  def run(args: List[String]): IO[ExitCode] = {
    program.as(ExitCode.Success).handleErrorWith { err =>
      IO.println(s"Error: ${err.getMessage}").as(ExitCode.Error)
    }
  }
  
  def program: IO[Unit] = {
    for {
      _ <- IO.println("Starting...")
      _ <- doWork
      _ <- IO.println("Done!")
    } yield ()
  }
  
  def doWork: IO[Unit] = ???
}
```

---

## 8. Concurrency в Cats Effect

### 8.1 Fiber

```scala
import cats.effect.{IO, Fiber}
import scala.concurrent.duration._

// Создание Fiber
val fiber: IO[Fiber[IO, Throwable, Int]] = IO(42).start

// join - ожидание результата
val result: IO[Int] = for {
  fib <- IO(Thread.sleep(1000)).as(42).start
  value <- fib.join
} yield value

// joinWithNever - вечное ожидание (безопасно для фонового потока)
val background = for {
  fib <- IO.never.start
  _ <- fib.joinWithNever
} yield ()

// cancel - отмена Fiber
val cancelExample = for {
  fib <- IO.sleep(10.seconds).start
  _ <- IO.sleep(1.second)
  _ <- fib.cancel
  _ <- IO.println("Fiber canceled")
} yield ()

// Parallel execution
val parallel = for {
  fib1 <- IO.sleep(1.second).as(10).start
  fib2 <- IO.sleep(1.second).as(20).start
  result1 <- fib1.join
  result2 <- fib2.join
} yield result1 + result2 // takes ~1 second, not 2

// Race - первый завершившийся
val fast = IO.sleep(100.millis).as("fast")
val slow = IO.sleep(1.second).as("slow")

IO.race(fast, slow) // IO(Left("fast"))

// racePair - получить оба Fiber
IO.racePair(fast, slow).flatMap {
  case Left((fastResult, slowFiber)) =>
    slowFiber.cancel >> IO.pure(fastResult)
  case Right((fastFiber, slowResult)) =>
    fastFiber.cancel >> IO.pure(slowResult)
}

// both - ждать оба
IO.both(fast, slow) // IO(("fast", "slow"))

// Практический пример - timeout с fallback
def timeoutWithFallback[A](
  io: IO[A],
  timeout: FiniteDuration,
  fallback: IO[A]
): IO[A] = {
  IO.race(io, IO.sleep(timeout)).flatMap {
    case Left(result) => IO.pure(result)
    case Right(_) => fallback
  }
}

// Background task pattern
def withBackgroundTask[A](task: IO[Unit])(main: IO[A]): IO[A] = {
  for {
    fiber <- task.start
    result <- main.onError(_ => fiber.cancel.void)
    _ <- fiber.cancel
  } yield result
}

// Heartbeat pattern
def heartbeat(interval: FiniteDuration): IO[Fiber[IO, Throwable, Unit]] = {
  (IO.sleep(interval) >> IO.println("Heartbeat")).foreverM.start
}

val programWithHeartbeat = for {
  hb <- heartbeat(1.second)
  _ <- doWork
  _ <- hb.cancel
} yield ()
```

### 8.2 Ref

```scala
import cats.effect.{IO, Ref}

// Ref - потокобезопасная изменяемая ссылка
val program = for {
  ref <- Ref.of[IO, Int](0)
  _ <- ref.set(10)
  value <- ref.get
  _ <- IO.println(s"Value: $value")
} yield ()

// update - атомарное изменение
val updateExample = for {
  ref <- Ref.of[IO, Int](0)
  _ <- ref.update(_ + 1)
  _ <- ref.update(_ + 1)
  value <- ref.get
  _ <- IO.println(s"Value: $value") // 2
} yield ()

// modify - атомарное изменение с возвратом значения
val modifyExample = for {
  ref <- Ref.of[IO, Int](0)
  prev <- ref.modify(n => (n + 1, n))
  _ <- IO.println(s"Previous: $prev") // 0
  current <- ref.get
  _ <- IO.println(s"Current: $current") // 1
} yield ()

// getAndSet
val getAndSetExample = for {
  ref <- Ref.of[IO, Int](42)
  old <- ref.getAndSet(100)
  _ <- IO.println(s"Old: $old, New: ${ref.get}")
} yield ()

// updateAndGet
val updateAndGetExample = for {
  ref <- Ref.of[IO, Int](0)
  newValue <- ref.updateAndGet(_ + 10)
  _ <- IO.println(s"New value: $newValue") // 10
} yield ()

// Concurrent counter
def concurrentCounter: IO[Int] = {
  for {
    ref <- Ref.of[IO, Int](0)
    fibers <- List.fill(100)(
      ref.update(_ + 1)
    ).traverse(_.start)
    _ <- fibers.traverse_(_.join)
    result <- ref.get
  } yield result
}

concurrentCounter.unsafeRunSync() // всегда 100

// Практический пример - rate limiter
class RateLimiter(maxRequests: Int, window: FiniteDuration) {
  case class State(count: Int, resetTime: Long)
  
  private val stateRef: IO[Ref[IO, State]] = 
    Ref.of[IO, State](State(0, System.currentTimeMillis() + window.toMillis))
  
  def acquire: IO[Boolean] = stateRef.flatMap { ref =>
    ref.modify { state =>
      val now = System.currentTimeMillis()
      if (now > state.resetTime) {
        // Window reset
        (State(1, now + window.toMillis), true)
      } else if (state.count < maxRequests) {
        // Within limit
        (state.copy(count = state.count + 1), true)
      } else {
        // Limit exceeded
        (state, false)
      }
    }
  }
}

// Cache implementation
class Cache[K, V](ttl: FiniteDuration) {
  case class CacheEntry(value: V, expiresAt: Long)
  
  private val cacheRef: IO[Ref[IO, Map[K, CacheEntry]]] = 
    Ref.of[IO, Map[K, CacheEntry]](Map.empty)
  
  def get(key: K): IO[Option[V]] = cacheRef.flatMap { ref =>
    ref.get.map { cache =>
      cache.get(key).flatMap { entry =>
        if (System.currentTimeMillis() < entry.expiresAt) {
          Some(entry.value)
        } else {
          None
        }
      }
    }
  }
  
  def put(key: K, value: V): IO[Unit] = cacheRef.flatMap { ref =>
    val expiresAt = System.currentTimeMillis() + ttl.toMillis
    ref.update(_ + (key -> CacheEntry(value, expiresAt)))
  }
}
```

### 8.3 Deferred

```scala
import cats.effect.{IO, Deferred}

// Deferred - one-time promise
val deferredExample = for {
  deferred <- Deferred[IO, Int]
  fiber <- deferred.get.start // блокируется до complete
  _ <- IO.sleep(1.second)
  _ <- deferred.complete(42)
  result <- fiber.join
  _ <- IO.println(s"Result: $result")
} yield ()

// Producer-Consumer pattern
def producer(deferred: Deferred[IO, String]): IO[Unit] = {
  for {
    _ <- IO.sleep(2.seconds)
    _ <- IO.println("Producer: Computing value...")
    _ <- deferred.complete("Hello from producer")
  } yield ()
}

def consumer(deferred: Deferred[IO, String]): IO[Unit] = {
  for {
    _ <- IO.println("Consumer: Waiting for value...")
    value <- deferred.get
    _ <- IO.println(s"Consumer: Received $value")
  } yield ()
}

val producerConsumer = for {
  deferred <- Deferred[IO, String]
  consumerFiber <- consumer(deferred).start
  producerFiber <- producer(deferred).start
  _ <- consumerFiber.join
  _ <- producerFiber.join
} yield ()

// Timeout with Deferred
def withTimeout[A](io: IO[A], timeout: FiniteDuration): IO[Option[A]] = {
  for {
    deferred <- Deferred[IO, Option[A]]
    _ <- io.map(Some(_)).flatMap(deferred.complete).start
    _ <- (IO.sleep(timeout) >> deferred.complete(None)).start
    result <- deferred.get
  } yield result
}

// Coordination between multiple fibers
def coordinatedWork: IO[Unit] = {
  for {
    startSignal <- Deferred[IO, Unit]
    
    workers = List.fill(5) {
      for {
        _ <- IO.println("Worker ready")
        _ <- startSignal.get // wait for signal
        _ <- IO.println("Worker started")
      } yield ()
    }
    
    fibers <- workers.traverse(_.start)
    _ <- IO.sleep(1.second)
    _ <- IO.println("Starting all workers")
    _ <- startSignal.complete(())
    _ <- fibers.traverse_(_.join)
  } yield ()
}
```

### 8.4 Semaphore

```scala
import cats.effect.std.Semaphore
import cats.effect.IO

// Semaphore - ограничение параллелизма
val semaphoreExample = for {
  sem <- Semaphore[IO](2) // max 2 concurrent
  
  task = (id: Int) => for {
    _ <- IO.println(s"Task $id waiting...")
    _ <- sem.permit.use { _ =>
      for {
        _ <- IO.println(s"Task $id running")
        _ <- IO.sleep(1.second)
        _ <- IO.println(s"Task $id done")
      } yield ()
    }
  } yield ()
  
  fibers <- (1 to 5).toList.traverse(id => task(id).start)
  _ <- fibers.traverse_(_.join)
} yield ()

// acquire/release manually
val manualExample = for {
  sem <- Semaphore[IO](1)
  _ <- sem.acquire
  _ <- IO.println("Critical section")
  _ <- sem.release
} yield ()

// tryAcquire - non-blocking
val tryAcquireExample = for {
  sem <- Semaphore[IO](1)
  _ <- sem.acquire
  acquired <- sem.tryAcquire
  _ <- IO.println(s"Acquired: $acquired") // false
  _ <- sem.release
} yield ()

// Rate limiting
class RateLimiter(maxPerSecond: Int) {
  private val sem = Semaphore[IO](maxPerSecond)
  
  def execute[A](io: IO[A]): IO[A] = {
    for {
      s <- sem
      result <- s.permit.use(_ => io)
      _ <- IO.sleep(1.second / maxPerSecond) // replenish
    } yield result
  }
}

// Connection pool
class ConnectionPool(maxConnections: Int) {
  case class Connection(id: Int)
  
  private val semaphore = Semaphore[IO](maxConnections)
  
  def withConnection[A](f: Connection => IO[A]): IO[A] = {
    semaphore.flatMap { sem =>
      sem.permit.use { _ =>
        for {
          conn <- acquireConnection
          result <- f(conn)
          _ <- releaseConnection(conn)
        } yield result
      }
    }
  }
  
  private def acquireConnection: IO[Connection] = 
    IO(Connection(scala.util.Random.nextInt()))
  
  private def releaseConnection(conn: Connection): IO[Unit] = 
    IO.println(s"Released connection ${conn.id}")
}
```

### 8.5 Queue

```scala
import cats.effect.std.Queue
import cats.effect.IO

// Queue - thread-safe queue
val queueExample = for {
  queue <- Queue.bounded[IO, Int](10)
  
  producer = (1 to 5).toList.traverse_ { i =>
    queue.offer(i) >> IO.println(s"Produced $i")
  }
  
  consumer = (1 to 5).toList.traverse_ { _ =>
    queue.take.flatMap(i => IO.println(s"Consumed $i"))
  }
  
  _ <- (producer, consumer).parTupled
} yield ()

// tryOffer - non-blocking offer
val tryOfferExample = for {
  queue <- Queue.bounded[IO, Int](2)
  _ <- queue.offer(1)
  _ <- queue.offer(2)
  offered <- queue.tryOffer(3) // false - queue full
  _ <- IO.println(s"Offered: $offered")
} yield ()

// tryTake - non-blocking take
val tryTakeExample = for {
  queue <- Queue.bounded[IO, Int](10)
  taken <- queue.tryTake // None - queue empty
  _ <- IO.println(s"Taken: $taken")
} yield ()

// size
val sizeExample = for {
  queue <- Queue.bounded[IO, Int](10)
  _ <- queue.offer(1)
  _ <- queue.offer(2)
  size <- queue.size
  _ <- IO.println(s"Size: $size") // 2
} yield ()

// Worker pool pattern
def workerPool(numWorkers: Int): IO[Unit] = {
  for {
    queue <- Queue.bounded[IO, Int](100)
    
    producer = (1 to 20).toList.traverse_ { i =>
      queue.offer(i) >> IO.println(s"Queued task $i")
    }
    
    worker = (id: Int) => {
      def loop: IO[Unit] = {
        queue.take.flatMap { task =>
          IO.println(s"Worker $id processing task $task") >>
          IO.sleep(1.second) >>
          loop
        }
      }
      loop
    }
    
    workers = (1 to numWorkers).toList.map(worker)
    
    _ <- (producer :: workers).parTraverse_(identity)
  } yield ()
}

// Bounded buffer
class BoundedBuffer[A](capacity: Int) {
  private val queue = Queue.bounded[IO, A](capacity)
  
  def put(item: A): IO[Unit] = queue.flatMap(_.offer(item))
  
  def take: IO[A] = queue.flatMap(_.take)
  
  def tryPut(item: A): IO[Boolean] = queue.flatMap(_.tryOffer(item))
  
  def tryTake: IO[Option[A]] = queue.flatMap(_.tryTake)
}
```

---

## 9. Resource Management

### 9.1 Resource

```scala
import cats.effect.{IO, Resource}

// Resource - safe resource management
def makeResource: Resource[IO, Connection] = {
  Resource.make(
    acquire = IO.println("Acquiring connection") >> IO(new Connection)
  )(
    release = conn => IO.println("Releasing connection") >> IO(conn.close())
  )
}

// Использование
val program = makeResource.use { conn =>
  IO.println("Using connection") >> IO(conn.query("SELECT * FROM users"))
}

// Resource комбинирование
val combined = for {
  conn <- makeResource
  stmt <- Resource.make(IO(conn.prepareStatement))(s => IO(s.close()))
} yield (conn, stmt)

combined.use { case (conn, stmt) =>
  IO(stmt.execute())
}

// Resource.eval - создание ресурса из IO
val evalResource: Resource[IO, String] = 
  Resource.eval(IO("computed value"))

// Resource.pure - чистое значение
val pureResource: Resource[IO, Int] = 
  Resource.pure(42)

// onFinalize - добавить cleanup action
val withFinalize = makeResource.onFinalize(
  IO.println("Additional cleanup")
)

// allocated - получить acquire и release отдельно
val allocated: IO[(Connection, IO[Unit])] = makeResource.allocated

allocated.flatMap { case (conn, release) =>
  for {
    _ <- IO(conn.query("SELECT * FROM users"))
    _ <- release
  } yield ()
}

// Практический пример - database connection pool
class ConnectionPool(size: Int) {
  def acquire: Resource[IO, Connection] = {
    Resource.make(
      IO.println("Getting connection from pool") >> IO(new Connection)
    )(
      conn => IO.println("Returning connection to pool") >> IO(conn.close())
    )
  }
}

// HTTP client
def httpClient: Resource[IO, HttpClient] = {
  Resource.make(
    IO.println("Starting HTTP client") >> IO(new HttpClient)
  )(
    client => IO.println("Stopping HTTP client") >> IO(client.close())
  )
}

// Application with multiple resources
val application = for {
  pool <- Resource.eval(IO(new ConnectionPool(10)))
  conn <- pool.acquire
  client <- httpClient
} yield (conn, client)

application.use { case (conn, client) =>
  for {
    data <- IO(client.get("https://api.example.com"))
    _ <- IO(conn.insert(data))
  } yield ()
}

// Resource leak detection
val leaky = Resource.make(IO(new Connection))(
  _ => IO.raiseError(new Exception("Release failed!"))
)

// Безопасная обработка
val safe = leaky.use(conn => IO(conn.query("SELECT 1")))
  .handleErrorWith(err => IO.println(s"Error: ${err.getMessage}"))
```

### 9.2 Bracket Pattern

```scala
import cats.effect.IO

// bracket - simple resource pattern
def bracket[A, B](
  acquire: IO[A]
)(use: A => IO[B])(release: A => IO[Unit]): IO[B] = {
  acquire.bracket(use)(release)
}

// Пример
val fileOperation = IO(new FileReader("file.txt")).bracket { reader =>
  IO(reader.read())
} { reader =>
  IO(reader.close())
}

// bracketCase - с информацией о завершении
import cats.effect.kernel.Outcome

val bracketCaseExample = IO(new Connection).bracketCase { conn =>
  IO(conn.query("SELECT * FROM users"))
} {
  case (conn, Outcome.Succeeded(_)) => 
    IO.println("Success") >> IO(conn.close())
  case (conn, Outcome.Errored(e)) => 
    IO.println(s"Error: $e") >> IO(conn.rollback()) >> IO(conn.close())
  case (conn, Outcome.Canceled()) => 
    IO.println("Canceled") >> IO(conn.forceClose())
}

// guarantee - always execute (даже при ошибке или отмене)
val withGuarantee = IO.println("Main operation")
  .guarantee(IO.println("Cleanup always runs"))

// guaranteeCase
val withGuaranteeCase = IO.println("Main operation").guaranteeCase {
  case Outcome.Succeeded(_) => IO.println("Success cleanup")
  case Outcome.Errored(_) => IO.println("Error cleanup")
  case Outcome.Canceled() => IO.println("Cancel cleanup")
}

// onCancel - выполнить при отмене
val withOnCancel = IO.sleep(10.seconds)
  .onCancel(IO.println("Operation was canceled"))

// Transaction pattern
def transaction[A](f: Connection => IO[A]): IO[A] = {
  IO(new Connection).bracket { conn =>
    for {
      _ <- IO(conn.begin())
      result <- f(conn)
      _ <- IO(conn.commit())
    } yield result
  } { conn =>
    IO(conn.rollback()).handleError(_ => ()) >> IO(conn.close())
  }
}

// Использование
transaction { conn =>
  for {
    _ <- IO(conn.execute("INSERT INTO users VALUES (...)"))
    _ <- IO(conn.execute("UPDATE orders SET ..."))
  } yield ()
}
```

---

## 10. Streaming с FS2

### 10.1 Stream Basics

```scala
import fs2.Stream
import cats.effect.IO

// Creating streams
val stream1: Stream[IO, Int] = Stream(1, 2, 3, 4, 5)
val stream2: Stream[IO, Int] = Stream.range(1, 100)
val stream3: Stream[IO, Int] = Stream.iterate(0)(_ + 1)
val stream4: Stream[IO, Int] = Stream.unfold(0)(n => Some((n, n + 1)))

// eval - effect в stream
val effectStream: Stream[IO, String] = Stream.eval(IO("hello"))

// evals - multiple effects
val evalsStream: Stream[IO, Int] = 
  Stream.evals(IO(List(1, 2, 3)))

// repeatEval - repeat effect
val repeatStream: Stream[IO, String] = 
  Stream.repeatEval(IO(scala.util.Random.nextInt().toString))

// Компиляция и запуск
stream1.compile.toList // IO(List(1, 2, 3, 4, 5))
stream1.compile.toVector // IO(Vector(1, 2, 3, 4, 5))
stream1.compile.drain // IO(()) - ignore results
stream1.compile.fold(0)(_ + _) // IO(15)
stream1.compile.lastOrError // IO(5)
stream1.compile.count // IO(5)

// Transformations
stream1.map(_ * 2) // Stream(2, 4, 6, 8, 10)
stream1.filter(_ % 2 == 0) // Stream(2, 4)
stream1.flatMap(n => Stream.range(0, n)) // Stream(0, 0,1, 0,1,2, ...)

// take / drop
stream1.take(3) // Stream(1, 2, 3)
stream1.drop(2) // Stream(3, 4, 5)
stream1.takeWhile(_ < 4) // Stream(1, 2, 3)
stream1.dropWhile(_ < 3) // Stream(3, 4, 5)

// Chunking
stream1.chunkN(2) // Stream(Chunk(1,2), Chunk(3,4), Chunk(5))

// fold
stream1.fold(0)(_ + _) // Stream(15)

// scan (накопительный fold)
stream1.scan(0)(_ + _) // Stream(0, 1, 3, 6, 10, 15)

// Практический пример
def processNumbers: IO[Int] = {
  Stream.range(1, 101)
    .filter(_ % 2 == 0)
    .map(_ * 2)
    .take(10)
    .compile
    .fold(0)(_ + _)
}
```

### 10.2 Stream Combinators

```scala
import fs2.Stream
import cats.effect.IO

// ++ (concat)
val s1 = Stream(1, 2, 3)
val s2 = Stream(4, 5, 6)
s1 ++ s2 // Stream(1, 2, 3, 4, 5, 6)

// merge - interleave элементы
s1.merge(s2) // произвольный порядок

// concurrently - run two streams in parallel
val background = Stream.repeatEval(IO.println("background"))
val main = Stream.range(1, 5).evalMap(n => IO.println(s"main: $n"))

main.concurrently(background)

// parJoin - слияние нескольких streams параллельно
val streams = List(
  Stream.eval(IO.sleep(1.second) >> IO(1)),
  Stream.eval(IO.sleep(2.seconds) >> IO(2)),
  Stream.eval(IO.sleep(1.second) >> IO(3))
)

Stream.emits(streams).parJoin(3) // все параллельно

// zip - объединение поэлементно
s1.zip(s2) // Stream((1,4), (2,5), (3,6))

// zipWith
s1.zipWith(s2)(_ + _) // Stream(5, 7, 9)

// interleave - чередование
s1.interleave(s2) // Stream(1, 4, 2, 5, 3, 6)

// either - tag with Left/Right
s1.map(Left(_): Either[Int, String])
  .merge(Stream("a", "b").map(Right(_)))

// Практический пример - параллельная обработка
def processInParallel(ids: List[Long]): IO[List[User]] = {
  Stream.emits(ids)
    .parEvalMap(10)(id => fetchUser(id)) // max 10 parallel
    .compile
    .toList
}

def fetchUser(id: Long): IO[User] = IO.sleep(100.millis) >> IO(
  User(id, s"User-$id", s"user$id@example.com")
)
```

### 10.3 Resource Management in Streams

```scala
import fs2.Stream
import cats.effect.{IO, Resource}

// bracket
def fileStream(path: String): Stream[IO, String] = {
  Stream.bracket(IO(new FileReader(path)))(reader => IO(reader.close()))
    .flatMap { reader =>
      Stream.repeatEval(IO(reader.readLine()))
        .takeWhile(_ != null)
    }
}

// resource
def resourceStream(path: String): Stream[IO, String] = {
  Stream.resource(makeFileResource(path))
    .flatMap { reader =>
      Stream.repeatEval(IO(reader.readLine()))
        .takeWhile(_ != null)
    }
}

def makeFileResource(path: String): Resource[IO, FileReader] = {
  Resource.make(IO(new FileReader(path)))(r => IO(r.close()))
}

// onFinalize - cleanup при завершении stream
val withCleanup: Stream[IO, Int] = 
  Stream.range(1, 10)
    .onFinalize(IO.println("Stream finished"))

// Практический пример - обработка файла построчно
def processFile(inputPath: String, outputPath: String): IO[Unit] = {
  val input = Stream.resource(makeFileResource(inputPath))
    .flatMap { reader =>
      Stream.repeatEval(IO(reader.readLine()))
        .takeWhile(_ != null)
    }
  
  val output = Stream.resource(makeWriterResource(outputPath))
  
  input
    .filter(_.nonEmpty)
    .map(_.toUpperCase)
    .through(output.flatMap { writer =>
      _.evalMap(line => IO(writer.writeLine(line)))
    })
    .compile
    .drain
}

// Connection pool с Resource
def withConnectionPool[A](f: Connection => Stream[IO, A]): Stream[IO, A] = {
  Stream.resource(connectionPoolResource)
    .flatMap { pool =>
      Stream.resource(pool.acquire).flatMap(f)
    }
}
```

### 10.4 Error Handling in Streams

```scala
import fs2.Stream
import cats.effect.IO

// handle - обработка ошибок
val errorStream: Stream[IO, Int] = Stream(1, 2) ++ 
  Stream.raiseError[IO](new Exception("error")) ++
  Stream(3, 4)

errorStream.handle {
  case ex: Exception => -1
} // Stream(1, 2, -1)

// handleErrorWith
errorStream.handleErrorWith { ex =>
  Stream.eval(IO.println(s"Error: $ex")) >> Stream(0)
}

// attempt - Either
errorStream.attempt // Stream(Right(1), Right(2), Left(Exception))

// onError - side effect при ошибке
errorStream.onError { ex =>
  Stream.eval(IO.println(s"Error occurred: $ex"))
}

// retry
def unreliableStream: Stream[IO, Int] = {
  Stream.eval(IO {
    if (scala.util.Random.nextBoolean()) 42
    else throw new Exception("Failed")
  })
}

unreliableStream.attempts(3) // retry до 3 раз

// Backpressure и bounded queues
Stream.range(1, 1000000)
  .evalMap(n => IO.sleep(1.millis) >> IO.pure(n))
  .through(slowProcessor)

def slowProcessor: fs2.Pipe[IO, Int, Int] = 
  _.evalMap(n => IO.sleep(10.millis) >> IO.pure(n * 2))
```

---

## 11. Error Handling

### 11.1 ApplicativeError

```scala
import cats.ApplicativeError
import cats.syntax.applicativeError._

def validateAge[F[_]](age: Int)(implicit F: ApplicativeError[F, String]): F[Int] = {
  if (age >= 0 && age <= 120) F.pure(age)
  else F.raiseError("Invalid age")
}

// С Either
type Result[A] = Either[String, A]
validateAge[Result](25) // Right(25)
validateAge[Result](-5) // Left("Invalid age")

// С IO
validateAge[IO](25) // IO(25)
validateAge[IO](-5) // IO.raiseError(...)

// attempt
def riskyOperation[F[_]: ApplicativeError[*[_], Throwable]]: F[Int] = {
  ApplicativeError[F, Throwable].raiseError(new Exception("Error"))
}

riskyOperation[IO].attempt // IO(Left(Exception))

// handleError
riskyOperation[IO].handleError(_ => 0) // IO(0)

// recover
riskyOperation[IO].recover {
  case _: IllegalArgumentException => 1
  case _: NullPointerException => 2
}
```

### 11.2 MonadError Patterns

```scala
import cats.MonadError
import cats.syntax.monadError._

// Custom error ADT
sealed trait AppError
case class ValidationError(msg: String) extends AppError
case class DatabaseError(msg: String) extends AppError
case class NetworkError(msg: String) extends AppError

type AppResult[A] = Either[AppError, A]

def validateInput(input: String): AppResult[String] = {
  if (input.nonEmpty) Right(input)
  else Left(ValidationError("Input cannot be empty"))
}

def saveToDb(data: String): AppResult[Long] = {
  if (data.length < 100) Right(scala.util.Random.nextLong())
  else Left(DatabaseError("Data too large"))
}

def sendNotification(id: Long): AppResult[Unit] = {
  if (id > 0) Right(())
  else Left(NetworkError("Invalid ID"))
}

// Композиция
def process(input: String): AppResult[Unit] = {
  for {
    validated <- validateInput(input)
    id <- saveToDb(validated)
    _ <- sendNotification(id)
  } yield ()
}

// Error recovery
def processWithRecovery(input: String): AppResult[Unit] = {
  process(input).handleErrorWith {
    case ValidationError(msg) => 
      Left(ValidationError(s"Fixed: $msg"))
    case DatabaseError(_) =>
      // Retry with backup database
      Right(())
    case err => 
      Left(err)
  }
}

// EitherT for IO
import cats.data.EitherT

type AsyncResult[A] = EitherT[IO, AppError, A]

def validateInputAsync(input: String): AsyncResult[String] = {
  EitherT.fromEither[IO](validateInput(input))
}

def saveToDbAsync(data: String): AsyncResult[Long] = {
  EitherT(IO {
    Thread.sleep(100)
    saveToDb(data)
  })
}

def processAsync(input: String): AsyncResult[Unit] = {
  for {
    validated <- validateInputAsync(input)
    id <- saveToDbAsync(validated)
  } yield ()
}

// Запуск
processAsync("test").value // IO(Either[AppError, Unit])
```

### 11.3 Error Accumulation

```scala
import cats.data.{Validated, ValidatedNec}
import cats.syntax.validated._
import cats.syntax.apply._

// Accumulating errors with Validated
case class UserRegistration(
  name: String,
  age: Int,
  email: String,
  password: String
)

type ValidationResult[A] = ValidatedNec[String, A]

def validateName(name: String): ValidationResult[String] =
  if (name.length >= 3) name.validNec
  else "Name must be at least 3 characters".invalidNec

def validateAge(age: Int): ValidationResult[Int] =
  if (age >= 18 && age <= 120) age.validNec
  else "Age must be between 18 and 120".invalidNec

def validateEmail(email: String): ValidationResult[String] =
  if (email.contains("@")) email.validNec
  else "Email must contain @".invalidNec

def validatePassword(password: String): ValidationResult[String] = {
  val errors = Vector.newBuilder[String]
  
  if (password.length < 8) 
    errors += "Password must be at least 8 characters"
  if (!password.exists(_.isUpper)) 
    errors += "Password must contain uppercase letter"
  if (!password.exists(_.isDigit)) 
    errors += "Password must contain digit"
  
  val errs = errors.result()
  if (errs.isEmpty) password.validNec
  else Validated.invalid(cats.data.NonEmptyChain.fromSeq(errs).get)
}

def validateRegistration(
  name: String,
  age: Int,
  email: String,
  password: String
): ValidationResult[UserRegistration] = {
  (
    validateName(name),
    validateAge(age),
    validateEmail(email),
    validatePassword(password)
  ).mapN(UserRegistration)
}

// Использование
validateRegistration("Al", 15, "invalid", "weak")
// Invalid(NonEmptyChain(
//   "Name must be at least 3 characters",
//   "Age must be between 18 and 120",
//   "Email must contain @",
//   "Password must be at least 8 characters",
//   "Password must contain uppercase letter",
//   "Password must contain digit"
// ))

// Parallel validation с IO
import cats.syntax.parallel._

def validateAsync(input: String): IO[ValidationResult[String]] = IO {
  if (input.nonEmpty) input.validNec
  else "Cannot be empty".invalidNec
}

def parallelValidation(
  name: String,
  email: String,
  password: String
): IO[ValidationResult[(String, String, String)]] = {
  (
    validateAsync(name),
    validateAsync(email),
    validateAsync(password)
  ).parMapN((n, e, p) => (n, e, p).tupled)
}
```

---

## 12. Testing

### 12.1 Testing Pure Functions

```scala
import cats.effect.IO
import cats.effect.testing.scalatest.AsyncIOSpec
import org.scalatest.matchers.should.Matchers
import org.scalatest.freespec.AsyncFreeSpec

class PureFunctionSpec extends AsyncFreeSpec with AsyncIOSpec with Matchers {
  
  "Pure functions" - {
    "should be testable with Cats" in {
      import cats.syntax.functor._
      import cats.instances.option._
      
      val result = Option(42).map(_ * 2)
      result shouldBe Some(84)
    }
    
    "should compose correctly" in {
      import cats.syntax.flatMap._
      import cats.instances.option._
      
      val result = for {
        x <- Option(10)
        y <- Option(20)
      } yield x + y
      
      result shouldBe Some(30)
    }
  }
}
```

### 12.2 Testing IO

```scala
import cats.effect.IO
import cats.effect.testing.scalatest.AsyncIOSpec
import org.scalatest.matchers.should.Matchers
import org.scalatest.freespec.AsyncFreeSpec

class IOSpec extends AsyncFreeSpec with AsyncIOSpec with Matchers {
  
  "IO operations" - {
    "should execute successfully" in {
      val io = IO.pure(42)
      
      io.asserting(_ shouldBe 42)
    }
    
    "should handle errors" in {
      val io = IO.raiseError[Int](new Exception("Error"))
      
      io.attempt.asserting {
        case Left(ex) => ex.getMessage shouldBe "Error"
        case Right(_) => fail("Should have failed")
      }
    }
    
    "should compose with for-comprehension" in {
      val program = for {
        a <- IO.pure(10)
        b <- IO.pure(20)
        c <- IO.pure(a + b)
      } yield c
      
      program.asserting(_ shouldBe 30)
    }
    
    "should handle concurrent operations" in {
      val io1 = IO.sleep(100.millis) >> IO.pure(10)
      val io2 = IO.sleep(100.millis) >> IO.pure(20)
      
      (io1, io2).parTupled.asserting { case (a, b) =>
        a shouldBe 10
        b shouldBe 20
      }
    }
  }
}
```

### 12.3 Testing with TestControl

```scala
import cats.effect.IO
import cats.effect.testkit.TestControl
import scala.concurrent.duration._

class TimeSpec extends AsyncFreeSpec with AsyncIOSpec with Matchers {
  
  "Time-dependent operations" - {
    "should be testable" in {
      val program = for {
        _ <- IO.sleep(1.hour)
        result <- IO.pure(42)
      } yield result
      
      TestControl.executeEmbed(program).asserting(_ shouldBe 42)
    }
    
    "should respect timing" in {
      val program = for {
        start <- IO.realTime
        _ <- IO.sleep(5.seconds)
        end <- IO.realTime
      } yield end - start
      
      TestControl.executeEmbed(program).asserting { duration =>
        duration shouldBe 5.seconds
      }
    }
  }
}
```

### 12.4 Property-Based Testing

```scala
import cats.effect.IO
import org.scalacheck.Prop.forAll
import org.scalatest.propspec.AnyPropSpec
import org.scalatestplus.scalacheck.ScalaCheckPropertyChecks

class PropertySpec extends AnyPropSpec with ScalaCheckPropertyChecks {
  
  property("Functor identity law") {
    forAll { (x: Int) =>
      import cats.syntax.functor._
      import cats.instances.option._
      
      val result = Option(x).map(identity)
      result == Option(x)
    }
  }
  
  property("Functor composition law") {
    forAll { (x: Int, f: Int => Int, g: Int => Int) =>
      import cats.syntax.functor._
      import cats.instances.option._
      
      val result1 = Option(x).map(f).map(g)
      val result2 = Option(x).map(f andThen g)
      result1 == result2
    }
  }
  
  property("Monad left identity") {
    forAll { (x: Int, f: Int => Option[Int]) =>
      import cats.syntax.flatMap._
      import cats.instances.option._
      
      val result1 = Option(x).flatMap(f)
      val result2 = f(x)
      result1 == result2
    }
  }
}
```

### 12.5 Mocking and Test Doubles

```scala
import cats.effect.IO

// Interface for database
trait UserRepository {
  def findUser(id: Long): IO[Option[User]]
  def saveUser(user: User): IO[Unit]
}

// Test double
class TestUserRepository extends UserRepository {
  private var users: Map[Long, User] = Map.empty
  
  def findUser(id: Long): IO[Option[User]] = 
    IO.pure(users.get(id))
  
  def saveUser(user: User): IO[Unit] = 
    IO.delay { users = users + (user.id -> user) }
}

// Service under test
class UserService(repo: UserRepository) {
  def registerUser(name: String, email: String): IO[User] = {
    val user = User(scala.util.Random.nextLong(), name, email)
    repo.saveUser(user).as(user)
  }
  
  def getUser(id: Long): IO[Option[User]] = 
    repo.findUser(id)
}

// Test
class UserServiceSpec extends AsyncFreeSpec with AsyncIOSpec with Matchers {
  
  "UserService" - {
    "should save and retrieve user" in {
      val repo = new TestUserRepository
      val service = new UserService(repo)
      
      val program = for {
        user <- service.registerUser("Alice", "alice@example.com")
        retrieved <- service.getUser(user.id)
      } yield retrieved
      
      program.asserting(_ shouldBe defined)
    }
  }
}
```

---

## 13. Advanced Patterns

### 13.1 Tagless Final

```scala
import cats.effect.IO
import cats.Monad
import cats.syntax.functor._
import cats.syntax.flatMap._

// Tagless Final pattern
trait UserAlgebra[F[_]] {
  def findUser(id: Long): F[Option[User]]
  def saveUser(user: User): F[Unit]
}

trait OrderAlgebra[F[_]] {
  def findOrders(userId: Long): F[List[Order]]
  def createOrder(order: Order): F[Long]
}

// Implementation for IO
class IOUserAlgebra extends UserAlgebra[IO] {
  private var users: Map[Long, User] = Map.empty
  
  def findUser(id: Long): IO[Option[User]] = 
    IO.pure(users.get(id))
  
  def saveUser(user: User): IO[Unit] = 
    IO.delay { users = users + (user.id -> user) }
}

// Service using algebras
class UserOrderService[F[_]: Monad](
  users: UserAlgebra[F],
  orders: OrderAlgebra[F]
) {
  def getUserWithOrders(userId: Long): F[Option[(User, List[Order])]] = {
    for {
      userOpt <- users.findUser(userId)
      result <- userOpt match {
        case Some(user) =>
          orders.findOrders(userId).map(ords => Some((user, ords)))
        case None =>
          Monad[F].pure(None)
      }
    } yield result
  }
}

// Использование
val userAlg = new IOUserAlgebra
val orderAlg = new IOOrderAlgebra
val service = new UserOrderService[IO](userAlg, orderAlg)

service.getUserWithOrders(1L)
```

### 13.2 Free Monads

```scala
import cats.free.Free
import cats.{Id, ~>}

// DSL for CRUD operations
sealed trait CrudOp[A]
case class Create(item: String) extends CrudOp[Long]
case class Read(id: Long) extends CrudOp[Option[String]]
case class Update(id: Long, item: String) extends CrudOp[Boolean]
case class Delete(id: Long) extends CrudOp[Boolean]

type CrudIO[A] = Free[CrudOp, A]

// Smart constructors
def create(item: String): CrudIO[Long] = 
  Free.liftF(Create(item))

def read(id: Long): CrudIO[Option[String]] = 
  Free.liftF(Read(id))

def update(id: Long, item: String): CrudIO[Boolean] = 
  Free.liftF(Update(id, item))

def delete(id: Long): CrudIO[Boolean] = 
  Free.liftF(Delete(id))

// Program
val program: CrudIO[Option[String]] = for {
  id <- create("Hello")
  item <- read(id)
  _ <- update(id, "Hello, World!")
  updated <- read(id)
} yield updated

// Interpreter for testing
val testInterpreter: CrudOp ~> Id = new (CrudOp ~> Id) {
  private var storage: Map[Long, String] = Map.empty
  private var nextId: Long = 1L
  
  def apply[A](fa: CrudOp[A]): Id[A] = fa match {
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

// Interpreter for IO
val ioInterpreter: CrudOp ~> IO = new (CrudOp ~> IO) {
  // Database implementation
  ???
}

// Запуск
program.foldMap(testInterpreter) // Id(Some("Hello, World!"))
program.foldMap(ioInterpreter) // IO(Some("Hello, World!"))
```

### 13.3 MTL Style

```scala
import cats.Monad
import cats.data.{EitherT, ReaderT, StateT}
import cats.mtl._

// MTL type classes для эффектов
class UserService[F[_]](implicit 
  F: Monad[F],
  ask: Ask[F, Config],
  handle: Handle[F, AppError],
  state: Stateful[F, AppState]
) {
  def getUser(id: Long): F[User] = {
    for {
      config <- ask.ask
      appState <- state.get
      user <- findUserInDb(id, config.dbUrl)
      _ <- state.modify(s => s.copy(requestCount = s.requestCount + 1))
    } yield user
  }
  
  private def findUserInDb(id: Long, dbUrl: String): F[User] = 
    F.pure(User(id, s"User-$id", s"user$id@example.com"))
}

case class Config(dbUrl: String)
case class AppState(requestCount: Int)

sealed trait AppError
case class NotFoundError(msg: String) extends AppError

// Type alias для application stack
type AppM[A] = ReaderT[EitherT[StateT[IO, AppState, *], AppError, *], Config, A]

// Instances
implicit val askInstance: Ask[AppM, Config] = 
  Ask.askForReaderT[EitherT[StateT[IO, AppState, *], AppError, *], Config]

implicit val handleInstance: Handle[AppM, AppError] = 
  Handle.handleForEitherT[StateT[IO, AppState, *], AppError]

implicit val stateInstance: Stateful[AppM, AppState] = 
  Stateful.statefulForStateT[EitherT[IO, AppError, *], AppState]
```

### 13.4 Kleisli Arrows

```scala
import cats.data.Kleisli
import cats.effect.IO

// Kleisli[F[_], A, B] is essentially A => F[B]
type ConfigReader[A] = Kleisli[IO, Config, A]

def getDbConnection: ConfigReader[Connection] = Kleisli { config =>
  IO.delay(new Connection(config.dbUrl))
}

def getApiKey: ConfigReader[String] = Kleisli { config =>
  IO.pure(config.apiKey)
}

// Composition
val program: ConfigReader[String] = for {
  conn <- getDbConnection
  key <- getApiKey
  result <- Kleisli.liftF(IO.pure(s"Connected with key: $key"))
} yield result

// Запуск
val config = Config("localhost:5432", "secret")
program.run(config) // IO(String)

// local - transform environment
val withModifiedConfig: ConfigReader[String] = 
  program.local[Config](c => c.copy(apiKey = "modified"))

// Практический пример - dependency injection
trait Database {
  def query(sql: String): IO[List[String]]
}

trait EmailService {
  def send(to: String, body: String): IO[Unit]
}

case class Dependencies(db: Database, email: EmailService)

type App[A] = Kleisli[IO, Dependencies, A]

def findUser(id: Long): App[Option[User]] = Kleisli { deps =>
  deps.db.query(s"SELECT * FROM users WHERE id = $id")
    .map(_.headOption.map(row => parseUser(row)))
}

def sendWelcome(user: User): App[Unit] = Kleisli { deps =>
  deps.email.send(user.email, s"Welcome, ${user.name}!")
}

def registerUser(name: String, email: String): App[User] = {
  for {
    user <- Kleisli.liftF(IO.pure(User(Random.nextLong(), name, email)))
    _ <- sendWelcome(user)
  } yield user
}
```

---

## 14. Performance Optimization

### 14.1 Stack Safety

```scala
import cats.Eval

// Stack-unsafe recursion
def sumListUnsafe(list: List[Int]): Int = list match {
  case Nil => 0
  case h :: t => h + sumListUnsafe(t) // StackOverflowError для больших списков
}

// Stack-safe с Eval
def sumListSafe(list: List[Int]): Eval[Int] = list match {
  case Nil => Eval.now(0)
  case h :: t => Eval.defer(sumListSafe(t)).map(_ + h)
}

sumListSafe((1 to 100000).toList).value // OK

// tailRecM для stack safety
import cats.Monad

def factorial[F[_]: Monad](n: Int): F[BigInt] = {
  Monad[F].tailRecM((n, BigInt(1))) {
    case (0, acc) => Monad[F].pure(Right(acc))
    case (n, acc) => Monad[F].pure(Left((n - 1, acc * n)))
  }
}

// IO уже stack-safe
def ioLoop(n: Int): IO[Int] = {
  if (n <= 0) IO.pure(0)
  else IO.defer(ioLoop(n - 1)).map(_ + 1)
}

ioLoop(100000).unsafeRunSync() // OK
```

### 14.2 Batching and Chunking

```scala
import cats.effect.IO
import fs2.Stream

// Batching database operations
def batchInsert(users: List[User]): IO[Unit] = {
  users.grouped(100).toList.traverse_ { batch =>
    IO.println(s"Inserting batch of ${batch.size}") >>
    IO(database.insertBatch(batch))
  }
}

// FS2 chunking
def processLargeFile(path: String): IO[Unit] = {
  Stream.resource(makeFileResource(path))
    .flatMap { reader =>
      Stream.repeatEval(IO(reader.read(1024))) // read in chunks
        .takeWhile(_.nonEmpty)
    }
    .chunkN(100) // process 100 chunks at a time
    .evalMap(chunk => processChunk(chunk))
    .compile
    .drain
}

// Parallel batching
def parallelBatchProcess(items: List[Item]): IO[Unit] = {
  Stream.emits(items)
    .chunkN(50) // batches of 50
    .parEvalMap(4)(chunk => processBatch(chunk.toList)) // 4 parallel batches
    .compile
    .drain
}
```

### 14.3 Caching

```scala
import cats.effect.{IO, Ref}
import scala.concurrent.duration._

// Simple cache
class Cache[K, V](ttl: FiniteDuration) {
  case class Entry(value: V, expiresAt: Long)
  
  private val cache: IO[Ref[IO, Map[K, Entry]]] = 
    Ref.of[IO, Map[K, Entry]](Map.empty)
  
  def get(key: K, fetch: => IO[V]): IO[V] = cache.flatMap { ref =>
    ref.get.flatMap { map =>
      map.get(key) match {
        case Some(entry) if System.currentTimeMillis() < entry.expiresAt =>
          IO.pure(entry.value)
        case _ =>
          for {
            value <- fetch
            expiresAt = System.currentTimeMillis() + ttl.toMillis
            _ <- ref.update(_ + (key -> Entry(value, expiresAt)))
          } yield value
      }
    }
  }
}

// Memoization
def memoize[A, B](f: A => IO[B]): IO[A => IO[B]] = {
  Ref.of[IO, Map[A, B]](Map.empty).map { ref => (a: A) =>
    ref.get.flatMap { map =>
      map.get(a) match {
        case Some(b) => IO.pure(b)
        case None =>
          for {
            b <- f(a)
            _ <- ref.update(_ + (a -> b))
          } yield b
      }
    }
  }
}

// Использование
val expensiveComputation: Int => IO[Int] = n => 
  IO.sleep(1.second) >> IO.pure(n * n)

val cached = memoize(expensiveComputation)

cached.flatMap { f =>
  for {
    _ <- f(5) // takes 1 second
    _ <- f(5) // instant
  } yield ()
}
```

### 14.4 Resource Pooling

```scala
import cats.effect.{IO, Resource, Ref}
import cats.effect.std.Queue

// Connection pool
class ConnectionPool(size: Int) {
  private def createPool: IO[Queue[IO, Connection]] = {
    for {
      queue <- Queue.bounded[IO, Connection](size)
      connections <- List.fill(size)(createConnection).sequence
      _ <- connections.traverse_(queue.offer)
    } yield queue
  }
  
  def withConnection[A](f: Connection => IO[A]): IO[A] = {
    Resource.make(
      createPool.flatMap(_.take)
    )(conn => 
      createPool.flatMap(_.offer(conn))
    ).use(f)
  }
  
  private def createConnection: IO[Connection] = 
    IO(new Connection)
}

// Worker pool
class WorkerPool[A, B](workers: Int)(process: A => IO[B]) {
  private val queue: IO[Queue[IO, A]] = Queue.unbounded[IO, A]
  
  private def worker(queue: Queue[IO, A]): IO[Unit] = {
    queue.take.flatMap(process).foreverM
  }
  
  def submit(item: A): IO[Unit] = 
    queue.flatMap(_.offer(item))
  
  def start: IO[Unit] = {
    for {
      q <- queue
      _ <- List.fill(workers)(worker(q).start).sequence
    } yield ()
  }
}
```

### 14.5 Benchmarking

```scala
import cats.effect.IO
import scala.concurrent.duration._

// Simple benchmark
def benchmark[A](name: String)(io: IO[A]): IO[A] = {
  for {
    start <- IO.realTime
    result <- io
    end <- IO.realTime
    duration = end - start
    _ <- IO.println(s"$name took ${duration.toMillis}ms")
  } yield result
}

// Использование
benchmark("Database query") {
  fetchUsersFromDb
}

// Warmup и multiple iterations
def benchmarkWithWarmup[A](
  name: String,
  warmups: Int = 3,
  iterations: Int = 10
)(io: IO[A]): IO[Unit] = {
  for {
    _ <- IO.println(s"Warming up $name...")
    _ <- List.fill(warmups)(io).sequence
    
    _ <- IO.println(s"Running $iterations iterations...")
    times <- List.fill(iterations) {
      for {
        start <- IO.realTime
        _ <- io
        end <- IO.realTime
      } yield (end - start).toMicros
    }.sequence
    
    avg = times.sum / times.length
    min = times.min
    max = times.max
    
    _ <- IO.println(s"$name results:")
    _ <- IO.println(s"  Average: ${avg}μs")
    _ <- IO.println(s"  Min: ${min}μs")
    _ <- IO.println(s"  Max: ${max}μs")
  } yield ()
}
```

---

## Заключение

### Ключевые выводы:

1. **Cats Core** - мощные абстракции для FP (Functor, Monad, Traverse)
2. **Type Classes** - композируемые, расширяемые интерфейсы
3. **Cats Effect** - powerful IO monad для side effects
4. **Concurrency** - безопасные примитивы (Ref, Deferred, Semaphore)
5. **Resource Management** - bracket и Resource для безопасности
6. **Error Handling** - MonadError, Validated, Either
7. **Streaming** - FS2 для эффективной обработки данных
8. **Testing** - comprehensive testing toolkit

### Дополнительные ресурсы:

**Документация:**
- [Cats Documentation](https://typelevel.org/cats/)
- [Cats Effect Documentation](https://typelevel.org/cats-effect/)
- [FS2 Documentation](https://fs2.io/)

**Книги:**
- "Scala with Cats" - Noel Welsh, Dave Gurnell
- "Functional Programming in Scala" - Paul Chiusano, Rúnar Bjarnason
- "Practical FP in Scala" - Gabriel Volpe

**Курсы:**
- Rock the JVM - Cats Effect course
- Scala Exercises - Cats exercises
- Typelevel workshops

**Практика:**
- Реализуйте собственные type classes
- Создайте HTTP service с http4s
- Постройте streaming pipeline с FS2
- Напишите concurrent application

### Подготовка к интервью:

✓ Понимание Category Theory основ
✓ Знание Cats type classes
✓ Опыт с IO Monad и Cats Effect
✓ Понимание concurrency primitives
✓ Знание Resource management patterns
✓ Error handling strategies
✓ FS2 streaming
✓ Testing с Cats Effect
✓ Performance optimization
✓ Production patterns (Tagless Final, MTL)

**Успехов в изучении Cats и Cats Effect!** 🐱🚀
