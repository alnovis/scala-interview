# План подготовки к собеседованию Senior Scala Developer (2025)

> **Обновлено**: ноябрь 2025  
> **Продолжительность**: 6-8 недель  
> **Уровень**: Senior

---

## 🎓 Теоретические основы (Обязательное чтение перед началом)

### Теория категорий для функционального программирования

**Зачем учить теорию категорий?**
Теория категорий - это математическая основа функционального программирования. Понимание базовых концептов поможет:
- Глубоко понять монады, функторы, аппликативы
- Правильно использовать библиотеки (Cats, ZIO)
- Проектировать композируемые API
- Рассуждать о типах на более высоком уровне

#### 1. Категория (Category)

**Определение**: Категория C состоит из:
- Коллекции **объектов** (objects)
- Коллекции **морфизмов** (morphisms/arrows) между объектами
- Операции **композиции** морфизмов
- **Identity** морфизм для каждого объекта

**Законы категории:**
```
1. Ассоциативность: f ∘ (g ∘ h) = (f ∘ g) ∘ h
2. Идентичность: id ∘ f = f = f ∘ id
```

**В Scala:**
```scala
// Категория типов в Scala
// Объекты = типы (Int, String, User, etc.)
// Морфизмы = функции между типами

// Identity morphism
def identity[A](a: A): A = a

// Композиция
def compose[A, B, C](f: A => B, g: B => C): A => C = 
  a => g(f(a))

// Проверка законов
val f: Int => String = _.toString
val g: String => Boolean = _.nonEmpty
val h: Boolean => Int = if (_) 1 else 0

// Ассоциативность
compose(f, compose(g, h)) == compose(compose(f, g), h)

// Идентичность  
compose(identity, f) == f
compose(f, identity) == f
```

#### 2. Функтор (Functor)

**Определение**: Функтор F между категориями C и D - это отображение, которое:
- Каждому объекту X в C ставит в соответствие объект F(X) в D
- Каждому морфизму f: X → Y ставит в соответствие F(f): F(X) → F(Y)

**Законы функтора:**
```
1. Сохранение идентичности: F(id_X) = id_F(X)
2. Сохранение композиции: F(g ∘ f) = F(g) ∘ F(f)
```

**В Scala:**
```scala
trait Functor[F[_]]:
  def map[A, B](fa: F[A])(f: A => B): F[B]

// Законы (property-based tests)
import org.scalacheck.Prop._

def functorLaws[F[_]: Functor, A, B, C](
  fa: F[A],
  f: A => B,
  g: B => C
): Prop =
  // Закон идентичности
  val identityLaw = fa.map(identity) == fa
  
  // Закон композиции
  val compositionLaw = 
    fa.map(f).map(g) == fa.map(f andThen g)
    
  identityLaw && compositionLaw

// Примеры функторов
given Functor[Option] with
  def map[A, B](fa: Option[A])(f: A => B): Option[B] = 
    fa match
      case Some(a) => Some(f(a))
      case None => None

given Functor[List] with
  def map[A, B](fa: List[A])(f: A => B): List[B] = 
    fa match
      case Nil => Nil
      case head :: tail => f(head) :: map(tail)(f)
```

**Диаграмма функтора:**
```
Category C              Category D
    X ----f----> Y          F(X) ---F(f)---> F(Y)
    |            |            |                |
   id           id          id              id
    |            |            |                |
    X            Y          F(X)             F(Y)
```

#### 3. Натуральное преобразование (Natural Transformation)

**Определение**: Натуральное преобразование η между функторами F и G - это семейство морфизмов:
```
η_X: F(X) → G(X) для каждого объекта X
```

**Коммутативная диаграмма:**
```
F(X) ---η_X---> G(X)
 |               |
F(f)           G(f)
 |               |
 v               v
F(Y) ---η_Y---> G(Y)
```

**В Scala:**
```scala
// Natural transformation as ~> (type lambda)
trait ~>[F[_], G[_]]:
  def apply[A](fa: F[A]): G[A]

// Пример: Option ~> List
val optionToList = new (Option ~> List):
  def apply[A](fa: Option[A]): List[A] = 
    fa.toList

// Использование
optionToList(Some(42))  // List(42)
optionToList(None)      // List()
```

#### 4. Монада (Monad)

**Математическое определение**: Монада в категории C - это тройка (T, η, μ) где:
- T: C → C - эндофунктор
- η: Id → T - натуральное преобразование (unit/pure)
- μ: T∘T → T - натуральное преобразование (join/flatten)

**Диаграмма монады:**
```
       T(T(T(X)))
         /    \
    T(μ_X)    μ_T(X)
       /        \
   T(T(X))     T(T(X))
        \      /
         μ_X
          |
         T(X)
```

**Законы монады:**
```scala
// 1. Левая идентичность: pure(a).flatMap(f) == f(a)
pure(42).flatMap(x => Some(x + 1)) == Some(43)

// 2. Правая идентичность: m.flatMap(pure) == m
Some(42).flatMap(x => pure(x)) == Some(42)

// 3. Ассоциативность: m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g))
Some(42).flatMap(f).flatMap(g) == Some(42).flatMap(x => f(x).flatMap(g))
```

**Реализация:**
```scala
trait Monad[F[_]] extends Functor[F]:
  // Базовые операции
  def pure[A](a: A): F[A]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  
  // Производные операции
  def map[A, B](fa: F[A])(f: A => B): F[B] = 
    flatMap(fa)(a => pure(f(a)))
    
  def flatten[A](ffa: F[F[A]]): F[A] = 
    flatMap(ffa)(identity)

// Option Monad
given Monad[Option] with
  def pure[A](a: A): Option[A] = Some(a)
  
  def flatMap[A, B](fa: Option[A])(f: A => Option[B]): Option[B] =
    fa match
      case Some(a) => f(a)
      case None => None

// Использование
val result = for
  x <- Some(2)
  y <- Some(3)
  z <- Some(4)
yield x + y + z  // Some(9)

// Desugaring to flatMap
Some(2).flatMap(x =>
  Some(3).flatMap(y =>
    Some(4).map(z => x + y + z)))
```

#### 5. Аппликативный функтор (Applicative)

**Определение**: Аппликатив - это функтор с дополнительными операциями:
```scala
trait Applicative[F[_]] extends Functor[F]:
  def pure[A](a: A): F[A]
  def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
  
  // Альтернативная формулировка
  def map2[A, B, C](fa: F[A], fb: F[B])(f: (A, B) => C): F[C]
```

**Законы аппликатива:**
```scala
// 1. Identity
ap(pure(identity))(v) == v

// 2. Composition  
ap(ap(ap(pure(compose))(u))(v))(w) == ap(u)(ap(v)(w))

// 3. Homomorphism
ap(pure(f))(pure(x)) == pure(f(x))

// 4. Interchange
ap(u)(pure(y)) == ap(pure(f => f(y)))(u)
```

**Разница Applicative vs Monad:**
```scala
// Applicative - независимые вычисления
// Можно распараллелить!
val app: Option[Int] = 
  (Some(2), Some(3), Some(4)).mapN(_ + _ + _)

// Monad - зависимые вычисления
// Нельзя распараллелить (y зависит от x)
val mon: Option[Int] = for
  x <- readUser(id)
  y <- readOrders(x.id)  // зависит от x!
yield y.total
```

#### 6. Моноид (Monoid)

**Определение**: Моноид - это алгебраическая структура (M, •, e) где:
- M - множество
- • - бинарная ассоциативная операция: M × M → M
- e - нейтральный элемент

**Законы моноида:**
```scala
trait Monoid[A]:
  def empty: A                        // нейтральный элемент
  def combine(x: A, y: A): A          // бинарная операция
  
  // Законы:
  // 1. Ассоциативность: combine(x, combine(y, z)) == combine(combine(x, y), z)
  // 2. Идентичность: combine(x, empty) == x == combine(empty, x)

// Примеры моноидов
given Monoid[Int] with
  def empty: Int = 0
  def combine(x: Int, y: Int): Int = x + y

given Monoid[String] with
  def empty: String = ""
  def combine(x: String, y: String): String = x + y

given [A]: Monoid[List[A]] with
  def empty: List[A] = Nil
  def combine(x: List[A], y: List[A]): List[A] = x ++ y

// Использование
def combineAll[A](list: List[A])(using m: Monoid[A]): A =
  list.foldLeft(m.empty)(m.combine)

combineAll(List(1, 2, 3, 4))  // 10
combineAll(List("a", "b", "c"))  // "abc"
```

**Связь с категориями**: Моноид можно рассматривать как категорию с одним объектом!

#### 7. Kleisli Category

**Определение**: Kleisli категория для монады M имеет:
- Объекты: типы A, B, C...
- Морфизмы: функции типа A => M[B]
- Композиция: Kleisli композиция

**В Scala:**
```scala
case class Kleisli[F[_], A, B](run: A => F[B]):
  def compose[C](k: Kleisli[F, C, A])(using M: Monad[F]): Kleisli[F, C, B] =
    Kleisli(c => M.flatMap(k.run(c))(run))
    
  def andThen[C](k: Kleisli[F, B, C])(using M: Monad[F]): Kleisli[F, A, C] =
    k.compose(this)

// Пример: валидация с Option
val checkPositive: Kleisli[Option, Int, Int] = 
  Kleisli(n => if n > 0 then Some(n) else None)
  
val checkEven: Kleisli[Option, Int, Int] =
  Kleisli(n => if n % 2 == 0 then Some(n) else None)
  
val checkBoth = checkPositive andThen checkEven
checkBoth.run(4)   // Some(4)
checkBoth.run(-4)  // None
checkBoth.run(3)   // None
```

---

### Type System теория

#### 1. Система типов Хиндли-Милнера

**Базовые концепции:**
```scala
// Type inference
val x = 42              // x: Int (inferred)
val y = "hello"         // y: String (inferred)
val z = List(1, 2, 3)   // z: List[Int] (inferred)

// Обобщённые типы (parametric polymorphism)
def identity[A](a: A): A = a

// Type constraints
def sorted[A: Ordering](list: List[A]): List[A] = 
  list.sorted
```

#### 2. Variance (Вариантность)

**Ковариантность (+A):**
```scala
// Если A <: B, то F[A] <: F[B]
trait Producer[+A]:
  def produce(): A

// Можно использовать более специфичный тип
val animalProducer: Producer[Animal] = new Producer[Dog]:
  def produce(): Dog = new Dog()
```

**Контравариантность (-A):**
```scala
// Если A <: B, то F[B] <: F[A] (обратно!)
trait Consumer[-A]:
  def consume(a: A): Unit

// Можно использовать более общий тип
val dogConsumer: Consumer[Dog] = new Consumer[Animal]:
  def consume(a: Animal): Unit = println(s"Consuming $a")
```

**Инвариантность (A):**
```scala
// Нет никакого отношения между F[A] и F[B]
trait Box[A]:
  def get: A
  def set(a: A): Unit
```

**RULES:**
- Ковариантность: только в return positions
- Контравариантность: только в parameter positions
- Инвариантность: в обоих случаях

#### 3. Higher-Kinded Types (HKT)

**Определение**: Типы, которые принимают другие типы как параметры

```scala
// Kind: * (простой тип)
type A = Int
type B = String

// Kind: * -> * (принимает 1 тип, возвращает тип)
type F[X] = List[X]
type G[X] = Option[X]

// Kind: * -> * -> * (принимает 2 типа)
type H[X, Y] = Either[X, Y]

// Kind: (* -> *) -> * (принимает type constructor)
trait Functor[F[_]]:
  def map[A, B](fa: F[A])(f: A => B): F[B]
```

**Type Lambda (в Scala 3):**
```scala
// Scala 2 style (сложно)
type EitherString[A] = Either[String, A]

// Scala 3 style (просто)
type EitherString = [A] =>> Either[String, A]

// Использование
given Functor[EitherString] with
  def map[A, B](fa: Either[String, A])(f: A => B): Either[String, B] =
    fa.map(f)
```

---

## 📋 Общая структура

**Неделя 1-2**: Scala 3 + Основы функционального программирования  
**Неделя 3-4**: Effect Systems (ZIO/Cats Effect) + Экосистема  
**Неделя 5-6**: Системный дизайн + Архитектура  
**Неделя 7-8**: AI/ML интеграция + Mock интервью

---

## 🎯 Неделя 1: Scala 3 - Новые возможности

### День 1-2: Миграция на Scala 3

**🔥 КРИТИЧЕСКИ ВАЖНО**: Scala 3 становится стандартом индустрии в 2025. Scala 3.9 LTS выйдет в Q2 2026 с требованием JDK 17+.

#### 📖 Теоретическая база: Почему Scala 3?

**История и мотивация:**
Scala 3 (изначально называвшийся Dotty) - это complete rewrite компилятора с фокусом на:
- **Упрощение языка**: удаление излишней сложности
- **Улучшенная типовая система**: более принципиальный подход
- **Лучшая производительность**: оптимизированный компилятор
- **Developer experience**: понятные сообщения об ошибках

**Философия дизайна:**
1. **Intent over mechanism** - явное выражение намерения вместо универсальных механизмов
2. **Principled type system** - математически обоснованная типизация
3. **Simplicity through features** - больше специализированных feature вместо одного мощного

**Ключевые улучшения:**
- **Compilation speed**: 2-3x быстрее компиляция больших проектов
- **Error messages**: контекстуальные, понятные сообщения
- **IDE support**: лучшая интеграция (IntelliJ, VS Code)
- **Binary compatibility**: улучшенная совместимость

**Темы для изучения:**

#### Новый синтаксис
- ✅ **Optional braces** (отступы вместо фигурных скобок)
- ✅ **New control syntax** (`if`/`while`/`for` без скобок)
- ✅ **Top-level definitions** (без `object`)
- ✅ **Extension methods** (замена implicit classes)
- ✅ **Export clauses** (делегирование методов)

#### Given/Using система (замена implicits)

**📖 Теория: Контекстная абстракция**

**Проблема с implicit:**
В Scala 2 `implicit` использовался для множества несвязанных концепций:
1. Implicit conversions (неявные преобразования)
2. Implicit parameters (контекстные параметры)
3. Extension methods (методы расширения)
4. Type classes (классы типов)

Это создавало:
- **Путаницу**: одно ключевое слово для разных целей
- **Сложность**: непонятно, что именно делает implicit
- **Ошибки**: неожиданное поведение при resolution

**Решение в Scala 3: Разделение по намерению**

1. **Given/Using** - для контекстных параметров:
```scala
// Явное определение контекста
given intOrdering: Ordering[Int] with
  def compare(x: Int, y: Int): Int = x.compare(y)

// Использование контекста
def sort[A](list: List[A])(using ord: Ordering[A]): List[A] = 
  list.sorted(ord)
```

2. **Extension methods** - для добавления методов:
```scala
extension (s: String)
  def isPalindrome: Boolean = s == s.reverse
  
"hello".isPalindrome  // false
```

3. **Given imports** - выборочный import:
```scala
import scala.math.Ordering.given  // импорт всех given
import scala.math.Ordering.{given Ordering[Int]}  // selective
```

**Механизм resolution:**

Given resolution следует строгим правилам:
1. **Локальный scope**: ищет в текущем scope
2. **Companion objects**: проверяет companion объекты типов
3. **Package object**: глобальные given
4. **Imports**: явно импортированные given

**Приоритет resolution:**
- Более специфичный тип > более общий
- Локальный given > импортированный
- Явный parameter > неявный

**Типичные паттерны:**

```scala
// Type class pattern
trait Show[A]:
  def show(a: A): String

given Show[Int] with
  def show(a: Int): String = a.toString

given Show[String] with  
  def show(a: String): String = s"\"$a\""

// Derivation
given [A](using sa: Show[A]): Show[List[A]] with
  def show(list: List[A]): String = 
    list.map(sa.show).mkString("[", ", ", "]")

// Context bounds синтаксис
def print[A: Show](a: A): String = summon[Show[A]].show(a)
// эквивалентно
def print[A](a: A)(using Show[A]): String = summon[Show[A]].show(a)
```

**Anti-patterns и best practices:**

❌ **Плохо:**
```scala
// Ambiguous givens
given Ordering[Int] = Ordering.Int
given Ordering[Int] = (a, b) => b.compare(a)  // ОШИБКА!
```

✅ **Хорошо:**
```scala
// Named givens для clarity
given ascendingInt: Ordering[Int] = Ordering.Int
given descendingInt: Ordering[Int] = (a, b) => b.compare(a)

// Явное использование
sort(list)(using descendingInt)
```

```scala
// Scala 2 style (deprecated)
implicit val ordering: Ordering[Person] = ???

// Scala 3 style
given Ordering[Person] with
  def compare(x: Person, y: Person): Int = ???

// Context parameters
def sort[A](list: List[A])(using ord: Ordering[A]): List[A] = ???
```

#### Enumerations (first-class)
```scala
enum Color:
  case Red, Green, Blue

enum Tree[+T]:
  case Leaf(elem: T)
  case Branch(left: Tree[T], right: Tree[T])
```

#### Union and Intersection Types

**📖 Теория: Алгебраические типы**

**Мотивация:**
В Scala 2 для выражения "или" использовались sealed trait hierarchies:
```scala
sealed trait Result
case class Success(value: Int) extends Result
case class Failure(error: String) extends Result
```

Проблемы:
- Избыточность для простых случаев
- Необходимость создания дополнительных типов
- Pattern matching обязателен даже для простых операций

**Union Types (A | B) - "Или"**

Union type представляет значение, которое может быть типа A ИЛИ типа B:

```scala
type StringOrInt = String | Int

val x: StringOrInt = "hello"  // OK
val y: StringOrInt = 42       // OK
val z: StringOrInt = true     // Ошибка компиляции!

// Pattern matching с union types
def process(value: String | Int): String = value match
  case s: String => s"Got string: $s"
  case i: Int => s"Got int: $i"
```

**Математическая основа:**
Union types основаны на теории множеств:
- A | B = {x | x ∈ A ∨ x ∈ B}
- Коммутативность: A | B = B | A
- Ассоциативность: (A | B) | C = A | (B | C)
- Идемпотентность: A | A = A

**Расширенные примеры:**

```scala
// Nullable типы
type Nullable[A] = A | Null
val maybeString: Nullable[String] = null  // OK

// Ошибки с union types
type Result[A] = A | Error
sealed trait Error
case class ValidationError(msg: String) extends Error
case class NetworkError(cause: Throwable) extends Error

def fetchUser(id: String): Result[User] = ???

// Обработка
fetchUser("123") match
  case user: User => println(s"Got user: ${user.name}")
  case err: ValidationError => println(s"Validation failed: ${err.msg}")
  case err: NetworkError => println(s"Network error: ${err.cause}")

// Множественные типы
type JsonValue = String | Int | Double | Boolean | Null | JsonArray | JsonObject
```

**Intersection Types (A & B) - "И"**

Intersection type представляет значение, которое одновременно имеет типы A И B:

```scala
trait Serializable:
  def toBytes: Array[Byte]

trait Comparable[A]:
  def compareTo(other: A): Int

// Тип, который И Serializable И Comparable
type SerializableComparable = Serializable & Comparable[SerializableComparable]

// Использование
def persist[A <: Serializable & Comparable[A]](item: A): Unit =
  val bytes = item.toBytes
  if item.compareTo(lastItem) > 0 then
    save(bytes)
```

**Математическая основа:**
- A & B = {x | x ∈ A ∧ x ∈ B}
- Коммутативность: A & B = B & A
- Ассоциативность: (A & B) & C = A & (B & C)

**Практические применения:**

1. **Trait composition**
```scala
trait Logged:
  def log(msg: String): Unit

trait Cached:
  def cache[A](key: String, value: A): Unit

// Service с обоими capabilities
def createService[S <: Logged & Cached](service: S): Unit =
  service.log("Starting")
  service.cache("key", "value")
```

2. **Self types improvement**
```scala
// Вместо self types
trait Component:
  self: Database & Cache =>
  
// Можно использовать
trait Component[Deps <: Database & Cache]:
  def dependencies: Deps
```

**Сравнение с Sealed Traits:**

| Аспект | Sealed Traits | Union Types |
|--------|---------------|-------------|
| Определение | Требует иерархию | Inline определение |
| Extensibility | Закрыта | Открыта |
| Pattern match | Exhaustiveness check | Type-based |
| Performance | Allocation | No allocation |
| Use case | Domain modeling | Ad-hoc unions |

**Best Practices:**

✅ **Union types для:**
- Simple sum types
- Error handling
- API boundaries
- JSON values

✅ **Sealed traits для:**
- Domain modeling
- Complex hierarchies
- Добавление методов
- Pattern matching exhaustiveness

```scala
type StringOrInt = String | Int
type Readable = java.io.Serializable & java.lang.Readable

def process(value: String | Int): String = value match
  case s: String => s"Got string: $s"
  case i: Int => s"Got int: $i"
```

#### Opaque Types

**📖 Теория: Zero-cost abstractions**

**Проблема: Type safety vs Performance**

В Scala 2 для type-safe wrappers использовались value classes:
```scala
// Scala 2
case class UserId(value: Long) extends AnyVal
case class OrderId(value: Long) extends AnyVal

// Проблема: все еще есть allocation в некоторых случаях
def process(ids: List[UserId]): Unit = ???  // boxing!
```

**Value classes ограничения:**
- Boxing при использовании в коллекциях
- Boxing при pattern matching
- Не работают с примитивными типами всегда
- Runtime представление может быть неожиданным

**Opaque Types - решение**

Opaque types предоставляют полную type safety БЕЗ runtime overhead:

```scala
object Prices:
  opaque type Price = Double
  
  object Price:
    def apply(value: Double): Price = value
    def safe(value: Double): Option[Price] = 
      if value >= 0 then Some(value) else None
    
  extension (p: Price)
    def +(other: Price): Price = p + other
    def *(multiplier: Double): Price = p * multiplier
    def amount: Double = p  // accessor
```

**Как это работает:**

1. **Внутри модуля** - Price это Double:
```scala
// В Prices object
val p: Price = 10.0  // OK, прямое присваивание
val sum = p + 5.0    // OK, это Double
```

2. **Снаружи модуля** - Price это отдельный тип:
```scala
// Вне Prices object
val p: Price = 10.0           // ОШИБКА!
val p: Price = Price(10.0)    // OK
val d: Double = p             // ОШИБКА!
val d: Double = p.amount      // OK
```

**Compile-time vs Runtime:**

Compile time:
```scala
val price: Price = Price(100.0)
val total = price + Price(50.0)
```

Runtime (после erasure):
```scala
val price: Double = 100.0
val total = price + 50.0
```

**Нет дополнительных allocation, boxing, wrapper классов!**

**Продвинутые паттерны:**

1. **Smart constructors**
```scala
object EmailAddress:
  opaque type Email = String
  
  object Email:
    private val emailRegex = "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$".r
    
    def apply(value: String): Either[String, Email] =
      if emailRegex.matches(value) then Right(value)
      else Left(s"Invalid email: $value")
    
    // Unsafe для внутреннего использования
    private[EmailAddress] def unsafe(value: String): Email = value
  
  extension (e: Email)
    def domain: String = e.split("@")(1)
    def localPart: String = e.split("@")(0)
```

2. **Type-safe IDs**
```scala
object Ids:
  opaque type UserId = Long
  opaque type OrderId = Long
  opaque type ProductId = Long
  
  object UserId:
    def apply(value: Long): UserId = value
  object OrderId:
    def apply(value: Long): OrderId = value
  object ProductId:
    def apply(value: Long): ProductId = value
  
  // Нельзя перепутать!
  def getUser(id: UserId): User = ???
  def getOrder(id: OrderId): Order = ???
  
  val userId = UserId(123)
  val orderId = OrderId(123)
  
  getUser(orderId)  // ОШИБКА компиляции!
```

3. **Refined types**
```scala
object Refined:
  opaque type Positive = Double
  opaque type NonEmpty[A] = List[A]
  
  object Positive:
    def apply(value: Double): Option[Positive] =
      if value > 0 then Some(value) else None
  
  object NonEmpty:
    def apply[A](head: A, tail: A*): NonEmpty[A] = 
      head :: tail.toList
    def fromList[A](list: List[A]): Option[NonEmpty[A]] =
      list match
        case h :: t => Some(h :: t)
        case Nil => None
```

**Сравнение подходов:**

| Feature | Case Class | Value Class | Opaque Type |
|---------|------------|-------------|-------------|
| Type safety | ✅ | ✅ | ✅ |
| Zero overhead | ❌ | ⚠️ (частично) | ✅ |
| Pattern matching | ✅ | ✅ | ❌ |
| Extension methods | ✅ | ✅ | ✅ |
| Inheritance | ✅ | ❌ | ❌ |
| Collections | Allocates | May box | No overhead |

**Performance benchmark:**

```scala
// Opaque type
opaque type Meter = Double
def distance(m: Meter): Meter = m * 2  
// Compiled to: (m: Double) => m * 2

// Value class  
case class Meter(value: Double) extends AnyVal
def distance(m: Meter): Meter = Meter(m.value * 2)
// May allocate wrapper

// Case class
case class Meter(value: Double)
def distance(m: Meter): Meter = Meter(m.value * 2)
// Always allocates
```

**Best practices:**

✅ **Используйте opaque types для:**
- Primitive wrappers (IDs, measurements)
- Units of measure
- Refined types (positive, non-empty)
- Performance-critical code

❌ **Не используйте для:**
- Complex domain objects
- Когда нужен pattern matching
- Когда нужна inheritance

```scala
object Prices:
  opaque type Price = Double
  
  object Price:
    def apply(value: Double): Price = value
    
  extension (p: Price)
    def +(other: Price): Price = p + other
    def amount: Double = p
```

**Практика:**
```scala
// Задача 1: Мигрировать Scala 2 код на Scala 3
// Используйте given/using, extension methods, enums

// Задача 2: Реализовать type-safe DSL с union types
sealed trait Result
case class Success(value: Int) extends Result
case class Failure(error: String) extends Result

// Переписать используя union types и opaque types

// Задача 3: Named tuples (Scala 3.6+)
type Person = (name: String, age: Int, city: String)
val p: Person = (name = "John", age = 30, city = "NYC")
println(p.name) // type-safe access!
```

**Вопросы для самопроверки:**
- Чем `given` отличается от `implicit`?
- Когда использовать opaque types вместо value classes?
- Как работают union types vs sealed trait hierarchies?
- Преимущества extension methods над implicit classes?

### День 3-4: Better-Fors и улучшения компилятора

**🆕 Scala 3.7+**: SIP-62 "Better Fors" в preview режиме

**Темы:**
- ✅ Упрощённый desugaring for-comprehensions
- ✅ Pattern bindings в for
- ✅ Улучшенная производительность компиляции
- ✅ Match types для type-level программирования
- ✅ Polymorphic function types

#### Match Types
```scala
type Elem[X] = X match
  case String => Char
  case Array[t] => t
  case Iterable[t] => t
  case AnyVal => X
```

#### Dependent Function Types
```scala
trait Entry:
  type Key
  val key: Key

type ExtractKey = (e: Entry) => e.Key // зависимый тип!
```

**Практика:**
```scala
// Задача 1: Type-level computation с match types
type Head[X <: Tuple] = X match
  case h *: t => h
  case EmptyTuple => Nothing

// Задача 2: Better-fors
// Написать for-comprehension с новым синтаксисом
for
  x <- Option(1)
  y <- Option(2) if x > 0  // улучшенная поддержка guards
  z = x + y                 // улучшенные bindings
yield z

// Задача 3: Polymorphic function
val id: [A] => A => A = [A] => (a: A) => a
```

**Вопросы:**
- Как better-fors улучшает производительность?
- Когда использовать match types vs type classes?
- Разница между dependent и polymorphic function types?

---

---

### 🧬 Теория Effect Systems

#### Что такое Effect?

**Определение**: Effect (эффект) - это описание вычисления, которое может:
- Завершиться успешно со значением
- Завершиться с ошибкой
- Никогда не завершиться
- Иметь побочные эффекты (I/O, исключения, изменение состояния)

**Функциональный эффект = Чистое описание нечистого вычисления**

```scala
// Нечистая функция (side effect!)
def println(s: String): Unit = 
  System.out.println(s)  // побочный эффект

// Чистое описание эффекта
def putStrLn(s: String): IO[Unit] = 
  IO { System.out.println(s) }  // описание, не выполнение!

// Выполнение происходит явно
val program: IO[Unit] = putStrLn("Hello")
// Эффект ещё НЕ выполнен
runtime.unsafeRun(program)  // ЗДЕСЬ выполнение
```

#### Referential Transparency (Ссылочная прозрачность)

**Определение**: Выражение является ссылочно-прозрачным, если его можно заменить на его значение без изменения поведения программы.

```scala
// НЕ ссылочно-прозрачно
val x = println("Hello")
val y = println("Hello")
// Печатает "Hello" дважды

val z = x
val w = x
// Ничего не печатает! x уже Unit

// Ссылочно-прозрачно  
val x = IO.println("Hello")
val y = IO.println("Hello")
// Ничего не печатает, только описание

val z = x
val w = x
// z и w - одинаковые описания

// Выполнение
runtime.unsafeRun(z)  // Печатает "Hello"
runtime.unsafeRun(w)  // Печатает "Hello" снова
```

#### ZIO Effect Type Theory

**Сигнатура**: `ZIO[R, E, A]`
- R (Requirement) - зависимости окружения
- E (Error) - тип ошибки
- A (value) - тип успешного значения

**Type Aliases:**
```scala
type IO[E, A] = ZIO[Any, E, A]        // нет зависимостей
type Task[A] = ZIO[Any, Throwable, A] // может упасть с Throwable
type RIO[R, A] = ZIO[R, Throwable, A] // требует окружение
type UIO[A] = ZIO[Any, Nothing, A]    // не может упасть
type URIO[R, A] = ZIO[R, Nothing, A]  // требует R, не падает
```

**Как это работает:**

```scala
// ZIO как описание computation graph
sealed trait ZIO[-R, +E, +A]:
  // Комбинаторы создают новые узлы в графе
  def map[B](f: A => B): ZIO[R, E, B]
  def flatMap[R1 <: R, E1 >: E, B](f: A => ZIO[R1, E1, B]): ZIO[R1, E1, B]
  def zip[R1 <: R, E1 >: E, B](that: ZIO[R1, E1, B]): ZIO[R1, E1, (A, B)]

// Исполнение графа
trait Runtime[R]:
  def unsafeRun[E, A](zio: ZIO[R, E, A]): A
```

**Fiber Model (Green Threads):**

```
OS Thread 1: [Fiber1, Fiber2, Fiber3, ...]
OS Thread 2: [Fiber4, Fiber5, ...]
OS Thread N: [FiberX, ...]

Fiber = lightweight thread
- Управляется runtime, не OS
- Очень дёшево создать (~100 байт)
- Cooperative multitasking (yielding)
- Structured concurrency
```

#### Cats Effect Type Theory

**IO Monad**: `IO[A]`
- Только один type parameter (упрощение)
- Ошибки всегда Throwable
- Нет явных dependencies (используют Tagless Final)

**Типы параллелизма:**

```scala
// Sequential (монадическая композиция)
for
  a <- getUser(id)
  b <- getOrders(a.id)  // зависит от a
yield (a, b)

// Parallel (аппликативная композиция)
(getUser(id), getOrders(id)).parMapN { (a, b) =>
  (a, b)  // независимые вычисления
}
```

**Resource Management:**

```scala
// RAII pattern (Resource Acquisition Is Initialization)
val resource: Resource[IO, File] = 
  Resource.make(
    acquire = IO { openFile("data.txt") }
  )(
    release = file => IO { file.close() }
  )

// Гарантированное освобождение даже при ошибках
resource.use { file =>
  processFile(file)
}
```

#### Concurrency Primitives

**1. Ref (Атомарная ячейка):**
```scala
// ZIO
val program = for
  ref <- Ref.make(0)
  _ <- ref.update(_ + 1).repeatN(1000).fork.replicateZIO(10)
  value <- ref.get
yield value  // 10000 (атомарно!)

// Cats Effect
val program = for
  ref <- Ref.of[IO](0)
  _ <- ref.update(_ + 1).replicateA(10000)
  value <- ref.get
yield value
```

**2. Deferred (Promise/Future):**
```scala
// Координация между fibers
val program = for
  deferred <- Deferred[IO, Int]
  fiber <- (IO.sleep(1.second) >> deferred.complete(42)).start
  value <- deferred.get  // блокируется до complete
yield value
```

**3. Queue:**
```scala
// Producer-Consumer pattern
val program = for
  queue <- Queue.bounded[IO, Int](10)
  producer = Stream.range(0, 100).through(queue.enqueue)
  consumer = queue.dequeue.take(100).compile.toList
  result <- (producer.compile.drain, consumer).parTupled
yield result._2
```

#### Error Channel Theory

**ZIO bifunctor error channel:**
```scala
// Два канала: Success (A) и Failure (E)
trait ZIO[R, E, A]:
  def fold[B](
    failure: E => B,
    success: A => B
  ): ZIO[R, Nothing, B]
  
  def catchAll[R1 <: R, E2, A1 >: A](
    h: E => ZIO[R1, E2, A1]
  ): ZIO[R1, E2, A1]

// Typed errors!
case class ValidationError(msg: String)
case class DbError(cause: Throwable)

val program: ZIO[Any, ValidationError | DbError, User] = ???

// Pattern matching на errors
program.catchAll {
  case ValidationError(msg) => ZIO.succeed(defaultUser)
  case DbError(cause) => ZIO.fail(cause)
}
```

**Cats Effect монадичная обработка:**
```scala
// Один канал: IO[A] или MonadError
def program: IO[User] = 
  getUser(id).handleErrorWith { err =>
    IO.raiseError(new CustomError(err))
  }

// Typed errors через EitherT
type Result[A] = EitherT[IO, AppError, A]

val program: Result[User] = for
  user <- EitherT.liftF(getUser(id))
  _ <- EitherT.fromEither[IO](validate(user))
yield user
```

#### Interruption Theory

**Structured Concurrency:**
```scala
// Scope определяет lifetime fibers
ZIO.scoped {
  for
    fiber1 <- task1.fork
    fiber2 <- task2.fork
    result <- fiber1.join <*> fiber2.join
  yield result
  // При выходе из scope - все fibers прерываются!
}
```

**Uninterruptible regions:**
```scala
// Критическая секция без прерывания
val atomicUpdate = 
  (readState *> compute *> writeState).uninterruptible

// Interruptible waiting
val waitForSignal = 
  signal.await.interruptible
```

---

## 🚀 Неделя 2: Effect Systems - ZIO vs Cats Effect

### День 1-3: Выбор Effect System

**🎯 ТРЕНД 2025**: Обе системы широко используются. Знание обеих - конкурентное преимущество.

#### 📖 Фундаментальная теория: Что такое Effect System?

**Определение:**
Effect System - это способ описания вычислений с побочными эффектами (side effects) в функциональном стиле, превращая императивные операции в композируемые значения.

**Проблема с императивным кодом:**

```scala
// Императивный код
def getUser(id: String): User = {
  val connection = openDatabase()  // Effect!
  try {
    val user = connection.query(id) // Effect!
    log(s"Found user: ${user.name}") // Effect!
    user
  } finally {
    connection.close()              // Effect!
  }
}
```

Проблемы:
- ❌ Не композируется
- ❌ Тестировать сложно
- ❌ Error handling разбросан
- ❌ Resource leaks возможны
- ❌ Нет control над execution

**Решение - Effect Types:**

```scala
// Functional effect
def getUser(id: String): IO[User] = 
  for {
    connection <- openDatabase()
    user       <- connection.query(id)
    _          <- log(s"Found user: ${user.name}")
  } yield user
```

Преимущества:
- ✅ Композируется
- ✅ Легко тестировать
- ✅ Централизованная обработка ошибок
- ✅ Автоматический resource management
- ✅ Control над execution

**Ключевые концепции:**

1. **Referential Transparency (Ссылочная прозрачность)**

```scala
// Не referentially transparent
val x = println("Hello")
val y = x  // не то же самое, что println("Hello")

// Referentially transparent
val x: IO[Unit] = IO.println("Hello")
val y = x  // точно такое же значение
```

2. **Separation of Description and Execution**

```scala
// Описание (только data structure)
val program: IO[Unit] = for {
  name <- IO.readLine
  _    <- IO.println(s"Hello, $name")
} yield ()

// Execution (происходит отдельно)
program.unsafeRun()  // здесь выполняются effects!
```

3. **Composition**

```scala
// Маленькие effects
val readConfig: IO[Config] = ???
val connectDB: IO[Database] = ???
val migrateSchema: IO[Unit] = ???

// Композируем в большой effect
val initialize: IO[Database] = for {
  config <- readConfig
  db     <- connectDB
  _      <- migrateSchema
} yield db
```

**Effect Type Anatomy:**

```scala
// Обобщённая структура
Effect[R, E, A]
//     ^  ^  ^
//     |  |  |
//     |  |  +-- Success type
//     |  +---- Error type
//     +-------- Requirements (dependencies)
```

**Типы эффектов по semantics:**

1. **IO/Task** - может упасть с Throwable
```scala
IO[A]        ≡ Effect[Any, Throwable, A]
```

2. **UIO** - не может упасть
```scala
UIO[A]       ≡ Effect[Any, Nothing, A]
```

3. **RIO** - требует environment
```scala
RIO[R, A]    ≡ Effect[R, Throwable, A]
```

**Fiber-based Concurrency:**

Традиционные threads vs Fibers:

```
Threads:
- OS-managed
- Heavy (~1MB stack)
- Ограниченное количество
- Context switching дорогой

Fibers:
- User-space
- Light (~KB)
- Миллионы fibers
- Cheap context switching
```

Пример:
```scala
// Spawning 1 million tasks
(1 to 1_000_000).foreach { i =>
  // С threads - OutOfMemoryError!
  new Thread(() => doWork(i)).start()
  
  // С fibers - OK!
  doWork(i).fork
}
```

**Green Threads модель:**

```
┌─────────────────────────────┐
│      JVM Thread Pool        │
│   (8 threads на 8 cores)    │
└─────────────┬───────────────┘
              │
        ┌─────┴─────┐
        │ Scheduler │
        └─────┬─────┘
              │
    ┌─────────┼─────────┐
    │         │         │
┌───▼───┐ ┌──▼───┐ ┌──▼───┐
│Fiber 1│ │Fiber2│...│FiberN│
└───────┘ └──────┘ └──────┘
  (1M fibers)
```

**Structured Concurrency:**

Принцип: child fibers не outlive parent fiber

```scala
// Parent fiber
val parent = for {
  fiber1 <- task1.fork  // child
  fiber2 <- task2.fork  // child
  _      <- fiber1.join
  _      <- fiber2.join
} yield ()

// Если parent cancelled/fails, все children автоматически cancelled
```

**Resource Safety:**

Проблема с try/finally:
```scala
// Может leak при interruption!
try {
  val resource = acquire()
  use(resource)
} finally {
  release(resource)
}
```

Effect system решение:
```scala
// Гарантированный cleanup
Resource.make(acquire)(release).use { resource =>
  use(resource)
}
```

#### ZIO 2.x Ecosystem
**Философия**: Opinionated, batteries-included framework

**📖 Теория: ZIO Architecture**

**Тип ZIO[R, E, A]:**

```scala
trait ZIO[-R, +E, +A]
//        ^  ^  ^
//        |  |  |
//        |  |  +-- Success value type (covariant)
//        |  +---- Error type (covariant)
//        +-------- Environment type (contravariant)
```

**Variance объяснение:**

- **R contravariant (-R)**: можно заменить на supertype
  ```scala
  val specific: ZIO[Database, E, A] = ???
  val general: ZIO[Any, E, A] = specific  // OK
  // Database <: Any, поэтому contravariance работает
  ```

- **E covariant (+E)**: можно заменить на subtype
  ```scala
  val specific: ZIO[R, IOException, A] = ???
  val general: ZIO[R, Exception, A] = specific  // OK
  // IOException <: Exception
  ```

- **A covariant (+A)**: standard return type variance
  ```scala
  val specific: ZIO[R, E, String] = ???
  val general: ZIO[R, E, Any] = specific  // OK
  ```

**Type aliases для convenience:**

```scala
type IO[+E, +A]      = ZIO[Any, E, A]         // No requirements
type Task[+A]        = ZIO[Any, Throwable, A] // Standard task
type RIO[-R, +A]     = ZIO[R, Throwable, A]   // Requires environment
type UIO[+A]         = ZIO[Any, Nothing, A]   // Cannot fail
type URIO[-R, +A]    = ZIO[R, Nothing, A]     // Needs env, cannot fail
```

**ZIO Runtime Architecture:**

```
Application Code
       │
       ▼
┌──────────────────┐
│   ZIO Effect     │ ◄─── Description (data structure)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  ZIO Runtime     │
├──────────────────┤
│ • Fiber Scheduler│
│ • Error Handler  │
│ • Executor       │
└────────┬─────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌───────┐
│ CPU   │ │Blocking│
│ Pool  │ │  Pool  │
└───────┘ └───────┘
```

**Typed Errors - ключевое преимущество:**

```scala
// Ошибки в типовой системе!
sealed trait UserError
case class NotFound(id: String) extends UserError
case class InvalidInput(field: String) extends UserError
case class Unauthorized() extends UserError

def getUser(id: String): ZIO[Database, UserError, User] = ???

// Компилятор заставляет обработать все ошибки
val program = getUser("123").flatMap {
  case user => processUser(user)
}.catchAll {
  case NotFound(id) => ZIO.succeed(defaultUser)
  case InvalidInput(field) => ZIO.fail(new Exception(s"Invalid: $field"))
  case Unauthorized() => ZIO.fail(new Exception("Access denied"))
}
```

**Error Channel vs Defect:**

```scala
// Error Channel - expected errors (типизированы)
val expected: IO[String, Int] = ZIO.fail("error")

// Defect - unexpected errors (не в error channel)
val unexpected: UIO[Int] = ZIO.die(new RuntimeException("bug"))

// Conversion
val caught: Task[Int] = unexpected.catchAllDefect {
  case e: RuntimeException => ZIO.succeed(0)
}
```

**ZLayer - Dependency Injection:**

```scala
// Service definition
trait Database:
  def query(sql: String): Task[ResultSet]

object Database:
  // Layer definition
  val live: ZLayer[Config, Throwable, Database] = 
    ZLayer.fromFunction { (config: Config) =>
      new Database {
        def query(sql: String) = ???
      }
    }

// Horizontal composition (providing multiple services)
type AppEnv = Database & Cache & Logger
val appLayer: ZLayer[Any, Throwable, AppEnv] = 
  Database.live ++ Cache.live ++ Logger.live

// Vertical composition (dependencies)
val userServiceLayer: ZLayer[Database, Nothing, UserService] = ???
val fullStack: ZLayer[Any, Throwable, UserService] = 
  Database.live >>> userServiceLayer
```

**Преимущества ZLayer:**

1. **Compile-time dependency checking**
```scala
def program: ZIO[Database & Cache, Throwable, Unit] = ???

// Забыли Cache - ошибка компиляции!
program.provide(Database.live)  // ERROR!

// Правильно
program.provide(Database.live, Cache.live)  // OK
```

2. **Automatic resource management**
```scala
val dbLayer = ZLayer.scoped {
  ZIO.acquireRelease(openConnection)(_.close())
}
// close() вызовется автоматически
```

3. **Testability**
```scala
// Production
val prodDB: ZLayer[Any, Nothing, Database] = Database.live

// Test
val testDB: ZLayer[Any, Nothing, Database] = ZLayer.succeed {
  new Database {
    def query(sql: String) = ZIO.succeed(mockResult)
  }
}

// Same code, different layer
myTest.provide(testDB)
```

**Преимущества:**
- ✅ Typed errors: `ZIO[R, E, A]`
- ✅ Built-in dependency injection (ZLayer)
- ✅ Rich error handling
- ✅ Structured concurrency
- ✅ Software Transactional Memory (STM)

**Основы ZIO:**
```scala
import zio._

// Создание эффектов
val succeed: UIO[Int] = ZIO.succeed(42)
val fail: IO[String, Nothing] = ZIO.fail("error")
val attempt: Task[String] = ZIO.attempt(readFile())

// Композиция
val program: Task[Unit] = for
  user <- getUser(id)
  orders <- getOrders(user)
  _ <- Console.printLine(s"Orders: $orders")
yield ()

// ZLayer для DI
trait UserRepo:
  def find(id: String): Task[User]

object UserRepo:
  val live: ZLayer[Database, Nothing, UserRepo] = ???

// Использование
val app = (for
  user <- ZIO.serviceWithZIO[UserRepo](_.find("123"))
yield user).provide(UserRepo.live)
```

**ZIO Streams:**
```scala
import zio.stream._

val stream = ZStream.fromIterable(1 to 1000)
  .mapZIOPar(8)(processItem)  // параллелизм
  .throttleShape(1, 1.second)(_.length)  // backpressure
  .run(ZSink.foldLeft(0)(_ + _))
```

#### Cats Effect 3.x Ecosystem
**Философия**: Lightweight, tagless final approach

**📖 Теория: Cats Effect Architecture**

**Typeclass-based дизайн:**

В отличие от конкретного типа ZIO, Cats Effect базируется на иерархии typeclasses:

```scala
// Иерархия typeclasses
trait Functor[F[_]]                    // map
    ↓
trait Applicative[F[_]]               // pure, ap
    ↓
trait Monad[F[_]]                     // flatMap
    ↓
trait MonadCancel[F[_], E]           // error handling, resource safety
    ↓
trait MonadError[F[_], E]            // raiseError, handleError
    ↓
trait Spawn[F[_]]                     // fiber operations (start, join)
    ↓
trait Concurrent[F[_]]                // Ref, Deferred
    ↓
trait Temporal[F[_]]                  // sleep, timeout
    ↓
trait Sync[F[_]]                      // suspend side effects
    ↓
trait Async[F[_]]                     // async boundaries
    ↓
trait Effect[F[_]]                    // run effects
```

**Tagless Final паттерн:**

```scala
// Generic программирование с F[_]
trait UserRepo[F[_]]:
  def findById(id: String): F[Option[User]]
  def save(user: User): F[Unit]

// Реализация для любого F с Monad
class UserRepoImpl[F[_]: Monad] extends UserRepo[F]:
  def findById(id: String): F[Option[User]] = 
    // implementation не знает, что такое F
    // может быть IO, Task, Future, или даже Id для тестов
    ???

// Использование с IO
val ioRepo: UserRepo[IO] = new UserRepoImpl[IO]

// Использование с другим effect
val taskRepo: UserRepo[Task] = new UserRepoImpl[Task]

// Тестирование с Id (синхронно, без effects)
type Id[A] = A
val testRepo: UserRepo[Id] = new UserRepoImpl[Id]
```

**Преимущества Tagless Final:**

1. **Полиморфизм по effect type**
```scala
def program[F[_]: Monad: UserRepo]: F[User] = 
  for {
    user <- UserRepo[F].findById("123")
    // один код работает с разными F
  } yield user
```

2. **Легкое тестирование**
```scala
// Test без effects
type TestF[A] = Either[String, A]
val testProgram = program[TestF]  // pure, deterministic
```

3. **Extensibility**
```scala
// Можно добавить новые effect types
case class MyEffect[A](run: () => A)
given Monad[MyEffect] = ???  // implement typeclass
// program[MyEffect] теперь работает!
```

**IO Monad - reference implementation:**

```scala
// IO - конкретный effect type
sealed trait IO[+A] {
  def map[B](f: A => B): IO[B]
  def flatMap[B](f: A => IO[B]): IO[B]
}

// Оптимизированные конструкторы
object IO {
  // Pure value
  def pure[A](a: A): IO[A] = Pure(a)
  
  // Suspend computation
  def delay[A](thunk: => A): IO[A] = Delay(() => thunk)
  
  // Async operation
  def async[A](k: (Either[Throwable, A] => Unit) => Unit): IO[A] = 
    Async(k)
  
  // Fork to new fiber
  def start[A](fa: IO[A]): IO[Fiber[IO, Throwable, A]] = 
    Start(fa)
}
```

**Trampolining для stack safety:**

```scala
// Без trampolining - stack overflow
def countdown(n: Int): IO[Unit] = 
  if (n <= 0) IO.unit
  else countdown(n - 1).flatMap(_ => IO.println(n))

countdown(100000)  // Stack overflow на обычной JVM

// Cats Effect использует trampolining:
// 1. Превращает рекурсию в loop
// 2. Использует heap вместо stack
// 3. Периодически yields для cooperative multitasking

// Result: stack safe!
```

**Fiber Model:**

```scala
trait Fiber[F[_], E, A]:
  // Ожидать результата
  def join: F[Outcome[F, E, A]]
  
  // Отменить fiber
  def cancel: F[Unit]

sealed trait Outcome[F[_], E, A]
case class Succeeded[F[_], E, A](fa: F[A]) extends Outcome[F, E, A]
case class Errored[F[_], E, A](e: E) extends Outcome[F, E, A]
case class Canceled[F[_], E, A]() extends Outcome[F, E, A]
```

**Resource Management:**

```scala
// Resource typeclass
sealed trait Resource[F[_], A]:
  def use[B](f: A => F[B]): F[B]

object Resource:
  // Create from acquire/release
  def make[F[_]: MonadCancel[*, E], A](
    acquire: F[A]
  )(
    release: A => F[Unit]
  ): Resource[F, A]
  
  // Composition
  def both[F[_]: Concurrent, A, B](
    ra: Resource[F, A],
    rb: Resource[F, B]
  ): Resource[F, (A, B)]
```

**Семантика cancellation:**

```scala
// Uncancelable region
IO.uncancelable { poll =>
  for {
    _ <- acquireResource  // не может быть cancelled
    a <- poll(useResource)  // может быть cancelled здесь
    _ <- releaseResource   // не может быть cancelled
  } yield a
}

// Poll позволяет вставить cancelable points внутри uncancelable
```

**Comparison: ZIO vs Cats Effect**

| Аспект | ZIO | Cats Effect |
|--------|-----|-------------|
| **Type** | Concrete (ZIO[R,E,A]) | Polymorphic (F[_]) |
| **Errors** | Typed (E) | Throwable only |
| **DI** | Built-in (ZLayer) | External (e.g. Distage) |
| **API** | Rich, batteries included | Minimal, typeclass-based |
| **Learning curve** | Steeper initially | Gradual |
| **Ecosystem** | ZIO-specific | Works with CE libs |
| **Performance** | Optimized for ZIO | Generic, still fast |

**Когда использовать каждый:**

**ZIO:**
- ✅ Greenfield projects
- ✅ Нужен typed errors
- ✅ Хотите DI из коробки
- ✅ Team новая в FP
- ✅ Нужна STM

**Cats Effect:**
- ✅ Existing typelevel stack (http4s, doobie, fs2)
- ✅ Нужен tagless final
- ✅ Интероп с Java libs
- ✅ Минималистичный подход
- ✅ Library development

**Преимущества:**
- ✅ Tagless Final pattern
- ✅ Широкая экосистема (http4s, doobie, fs2)
- ✅ Минимальная overhead
- ✅ Простой error handling (Throwable)

**Основы Cats Effect:**
```scala
import cats.effect._
import cats.effect.syntax.all._

// IO Monad
val io: IO[Unit] = IO.println("Hello")

// Resource management
val program: IO[Unit] = 
  Resource.make(IO(openFile()))(f => IO(f.close()))
    .use(file => processFile(file))

// Fiber-based concurrency
val parallel: IO[String] = for
  fiber1 <- task1.start
  fiber2 <- task2.start
  result1 <- fiber1.joinWithNever
  result2 <- fiber2.joinWithNever
yield s"$result1 $result2"
```

**FS2 Streams:**
```scala
import fs2._
import cats.effect._

Stream.eval(IO(1))
  .repeat
  .take(100)
  .parEvalMap(8)(processItem)  // параллелизм
  .metered(1.second)           // backpressure
  .compile
  .toList
```

**Практика:**

**ZIO задачи:**
```scala
// Задача 1: Retry с экспоненциальным backoff
def retry[R, E, A](
  zio: ZIO[R, E, A], 
  max: Int
): ZIO[R, E, A] = ???

// Задача 2: Circuit Breaker
class CircuitBreaker[R]:
  def call[E, A](effect: ZIO[R, E, A]): ZIO[R, E, A] = ???

// Задача 3: ZLayer композиция
trait Database
trait Cache  
trait UserService

object UserService:
  val live: ZLayer[Database & Cache, Nothing, UserService] = ???

// Задача 4: STM для concurrent counter
import zio.stm._

def incrementCounter(ref: TRef[Int]): USTM[Unit] = ???
```

**Cats Effect задачи:**
```scala
// Задача 1: Deferred для coordination
def waitForSignal[F[_]: Concurrent]: F[Unit] = for
  signal <- Deferred[F, Unit]
  _ <- (IO.sleep(1.second) >> signal.complete(())).start
  _ <- signal.get
yield ()

// Задача 2: Concurrent Ref для shared state
import cats.effect.std.Queue

def producer[F[_]: Async](queue: Queue[F, Int]): F[Unit] = ???
def consumer[F[_]: Async](queue: Queue[F, Int]): F[Unit] = ???

// Задача 3: Resource композиция
def resources[F[_]: Async]: Resource[F, (Database, Cache)] = for
  db <- Resource.make(openDB)(_.close)
  cache <- Resource.make(openCache)(_.close)
yield (db, cache)

// Задача 4: Tagless Final service
trait UserRepo[F[_]]:
  def find(id: String): F[Option[User]]
  def save(user: User): F[Unit]

class UserRepoImpl[F[_]: Monad] extends UserRepo[F]:
  def find(id: String): F[Option[User]] = ???
  def save(user: User): F[Unit] = ???
```

**Вопросы:**
- ZIO vs Cats Effect: когда выбрать каждый?
- Что такое fiber? Разница от Thread?
- Как работает Tagless Final?
- Typed errors (ZIO) vs Throwable (CE) - trade-offs?
- Что такое STM и когда его использовать?

### День 4-7: Продвинутые паттерны

#### Monad Transformers (для CE)
```scala
import cats.data._
import cats.effect._

type Result[A] = EitherT[IO, String, A]

def getUser(id: String): Result[User] = ???
def getOrders(user: User): Result[List[Order]] = ???

val program: Result[List[Order]] = for
  user <- getUser("123")
  orders <- getOrders(user)
yield orders

// Запуск
program.value: IO[Either[String, List[Order]]]
```

#### ZIO Environment Pattern
```scala
// Сервисный слой
trait UserService:
  def getUser(id: String): Task[User]

object UserService:
  def getUser(id: String): ZIO[UserService, Throwable, User] =
    ZIO.serviceWithZIO[UserService](_.getUser(id))

// Использование
val program = for
  user <- UserService.getUser("123")
  _ <- Console.printLine(user.name)
yield ()

// Provide dependencies
program.provide(UserService.live)
```

**Практика:**
```scala
// Задача 1: Многослойная архитектура с ZIO
trait Database
trait Logger
trait Config

trait UserRepo:
  def find(id: String): Task[User]

object UserRepo:
  val live: ZLayer[Database & Logger, Nothing, UserRepo] = ???

trait UserService:
  def getUser(id: String): Task[User]

object UserService:
  val live: ZLayer[UserRepo & Config, Nothing, UserService] = ???

// Compose все layers
val appLayer: ZLayer[Any, Throwable, UserService] = ???

// Задача 2: Error handling strategies
sealed trait AppError
case class ValidationError(msg: String) extends AppError
case class DatabaseError(cause: Throwable) extends AppError
case class NotFoundError(id: String) extends AppError

def handleErrors[A](
  zio: ZIO[Any, AppError, A]
): ZIO[Any, Nothing, Either[AppError, A]] = ???

// Задача 3: Testing с ZIO Test
import zio.test._

test("user service") {
  for
    user <- UserService.getUser("123")
  yield assertTrue(user.name == "John")
}.provide(
  UserService.test,  // test implementation
  UserRepo.test
)
```

---

---

### 🎭 Теория Actor Model и Reactive Systems

#### Actor Model (Модель акторов)

**История**: Разработана Carl Hewitt в 1973 году как математическая модель конкурентных вычислений.

**Основные принципы:**

**1. Actor - фундаментальная единица вычислений**
```
Actor = Behavior + State + Mailbox

Behavior: функция обработки сообщений
State: приватное состояние
Mailbox: очередь входящих сообщений
```

**2. Три основных операции актора:**
```scala
// 1. Send message (асинхронно)
actorRef ! Message

// 2. Create new actors
context.spawn(behavior, "name")

// 3. Change behavior
Behaviors.receive { (context, msg) =>
  msg match
    case Increment => 
      counter(state + 1)  // новое поведение
    case GetValue(replyTo) =>
      replyTo ! state
      Behaviors.same  // то же поведение
}
```

**3. Асинхронная передача сообщений (Message Passing)**
```
Actor A                    Actor B
   |                          |
   |-------- Message -------->|
   |                          | (обработка в mailbox)
   |<------- Response --------|
   |                          |
```

**Математическая модель:**
```
Actor = (State, Behavior)
Behavior: Message -> (State, Behavior, Effects)

где Effects = {
  SendMessage(target, msg),
  CreateActor(behavior),
  StopActor(ref)
}
```

#### Location Transparency

**Принцип**: ActorRef не знает, где физически находится актор:
```scala
// Локальный актор
val local: ActorRef[Message] = system.spawn(behavior, "local")

// Удалённый актор (на другой машине)
val remote: ActorRef[Message] = 
  system.receptionist.find(ServiceKey)

// Одинаковый интерфейс!
local ! Message
remote ! Message
```

#### Supervision (Надзор)

**Иерархия акторов:**
```
        Guardian
         /    \
    SupervisorA  SupervisorB
      /    \         |
  WorkerA1 WorkerA2 WorkerB1
```

**Стратегии восстановления:**

```scala
sealed trait SupervisorStrategy
case object Restart   // перезапустить с начальным состоянием
case object Resume    // продолжить с текущим состоянием
case object Stop      // остановить актор
case object Escalate  // поднять ошибку выше

// Пример
Behaviors.supervise(workerBehavior)
  .onFailure[IllegalArgumentException](
    SupervisorStrategy.restart
      .withLimit(maxNrOfRetries = 3, withinTimeRange = 1.minute)
  )
```

**Fault Tolerance Pattern:**
```
Let it crash!

- Не пытайтесь обработать все ошибки
- Supervisor знает, как восстанавливаться
- Изоляция: падение одного актора не влияет на других
```

#### Reactive Manifesto

**4 принципа реактивных систем:**

**1. Responsive (Отзывчивость)**
```
- Быстрый и предсказуемый ответ
- Проблемы обнаруживаются быстро
- Эффективная обработка ошибок
```

**2. Resilient (Устойчивость)**
```
- Система остаётся отзывчивой при сбоях
- Репликация
- Изоляция
- Делегирование
```

**3. Elastic (Эластичность)**
```
- Система масштабируется под нагрузкой
- Нет единой точки отказа
- Sharding
- Распределение нагрузки
```

**4. Message Driven (Управление сообщениями)**
```
- Асинхронная передача сообщений
- Backpressure
- Non-blocking
- Loose coupling
```

#### Backpressure Theory

**Проблема**: Producer быстрее, чем Consumer
```
Producer ----[1000 msg/s]----> Consumer [100 msg/s]
                                   |
                              Overflow!
```

**Решения:**

**1. Bounded Buffer (Ограниченный буфер)**
```scala
// Стратегии при переполнении:
enum OverflowStrategy:
  case DropHead      // удалить старые
  case DropTail      // удалить новые
  case DropBuffer    // очистить буфер
  case DropNew       // отклонить новое
  case Fail          // упасть с ошибкой
  case Backpressure  // блокировать producer
```

**2. Reactive Streams Protocol**
```scala
trait Publisher[T]:
  def subscribe(s: Subscriber[T]): Unit

trait Subscriber[T]:
  def onSubscribe(s: Subscription): Unit
  def onNext(t: T): Unit
  def onError(t: Throwable): Unit
  def onComplete(): Unit

trait Subscription:
  def request(n: Long): Unit  // Consumer запрашивает N элементов
  def cancel(): Unit

// Flow control
subscriber.onSubscribe(subscription)
subscription.request(10)  // "Готов принять 10 элементов"
// Publisher отправляет максимум 10
```

**3. Windowing & Buffering**
```scala
// Pekko Streams
source
  .buffer(100, OverflowStrategy.backpressure)
  .throttle(10, 1.second)
  .async  // boundary для backpressure
  .map(process)
  .to(sink)
```

#### Streaming Theory

**Pull-based Streams:**
```
Consumer <--pull-- Producer
Consumer --ack---> Producer

+ Backpressure из коробки
+ Эффективное использование памяти
- Latency (нужен roundtrip)
```

**Push-based Streams:**
```
Producer --push--> Consumer

+ Низкая latency
- Нужна явная реализация backpressure
- Risk of overflow
```

**Hybrid (Akka/Pekko Streams):**
```scala
// Graph DSL
val graph = RunnableGraph.fromGraph(
  GraphDSL.create() { implicit builder =>
    import GraphDSL.Implicits._
    
    val source = Source(1 to 100)
    val broadcast = builder.add(Broadcast[Int](2))
    val merge = builder.add(Merge[Int](2))
    val sink = Sink.foreach(println)
    
    source ~> broadcast ~> Flow[Int].map(_ * 2) ~> merge ~> sink
              broadcast ~> Flow[Int].map(_ * 3) ~> merge
              
    ClosedShape
  }
)
```

#### Event-Driven Architecture Theory

**Event Sourcing Pattern:**
```
Traditional:
  State = Current State in DB

Event Sourcing:
  State = fold(Events, InitialState)
  
Events = immutable log of facts
```

**Пример:**
```scala
sealed trait OrderEvent
case class OrderCreated(id: String, items: List[Item]) extends OrderEvent
case class ItemAdded(id: String, item: Item) extends OrderEvent
case class OrderPaid(id: String, amount: Money) extends OrderEvent

case class OrderState(
  id: String,
  items: List[Item],
  isPaid: Boolean
)

def aggregate(state: OrderState, event: OrderEvent): OrderState = 
  event match
    case OrderCreated(id, items) => 
      OrderState(id, items, false)
    case ItemAdded(id, item) => 
      state.copy(items = state.items :+ item)
    case OrderPaid(id, amount) => 
      state.copy(isPaid = true)

// Воспроизведение истории
val currentState = events.foldLeft(OrderState.empty)(aggregate)
```

**Преимущества Event Sourcing:**
```
+ Полная история изменений (audit log)
+ Time travel (состояние на любой момент)
+ Event replay для отладки
+ Projections (multiple read models)
- Сложность
- Eventual consistency
```

#### CQRS Pattern (Command Query Responsibility Segregation)

```
             Commands                Queries
                |                       |
          +-----------+           +-----------+
          |  Write    |           |   Read    |
          |  Model    |           |   Model   |
          +-----------+           +-----------+
                |                       |
          Write DB              Read DB (denormalized)
                |                       |
          Event Bus ------------------>
```

**Разделение:**
```scala
// Command side (write)
trait OrderCommandHandler:
  def createOrder(cmd: CreateOrder): ZIO[Any, AppError, OrderId]
  def addItem(cmd: AddItem): ZIO[Any, AppError, Unit]
  
// Query side (read)
trait OrderQueryHandler:
  def findById(id: OrderId): ZIO[Any, AppError, OrderView]
  def findByStatus(status: Status): ZIO[Any, AppError, List[OrderView]]
  
// Event propagation
trait EventBus:
  def publish(event: DomainEvent): UIO[Unit]
  
// Read model updater
val projector = EventBus.subscribe[OrderEvent].foreach { event =>
  updateReadModel(event)  // денормализация для queries
}
```

---

## 💎 Neделя 3: Apache Pekko и Reactive Systems

### День 1-3: Apache Pekko (Akka замена)

**🔥 КРИТИЧНО 2025**: Akka изменил лицензию на BSL. Apache Pekko - открытая замена.

#### 📖 Теория: Actor Model и Reactive Programming

**История:**
Actor Model предложен Carl Hewitt (1973) как математическая модель concurrent computation. В Scala реализован через Akka (теперь Pekko).

**Фундаментальные принципы:**

1. **"Everything is an Actor"**
```
Actor - это computational entity, который:
- Имеет mailbox для сообщений
- Обрабатывает сообщения последовательно
- Может создавать новых actors
- Может отправлять сообщения
- Может изменить поведение для следующего сообщения
```

2. **Location Transparency**
```
┌──────────┐     message     ┌──────────┐
│ Actor A  │ ─────────────► │ Actor B  │
└──────────┘                 └──────────┘
  (local)                      (может быть на другой машине!)
```

3. **Message-driven Architecture**
```
Преимущества:
✅ Loose coupling
✅ Async non-blocking
✅ Backpressure естественным образом
✅ Location transparency
```

**Actor Lifecycle:**
```
   ┌──────────┐
   │ Starting │
   └────┬─────┘
        │
   ┌────▼──────┐
   │  Running  │◄────┐
   └────┬──────┘     │
        │            │
   ┌────▼──────┐    │restart
   │ Stopping  │    │
   └────┬──────┘    │
        │      ┌────┴─────┐
   ┌────▼──────┤Restarting│
   │ Stopped   │          │
   └───────────┴──────────┘
```

**Supervision Tree:**
```
           Supervisor
                │
        ┌───────┼───────┐
     Child1  Child2  Child3

Если Child2 fails:
- Supervisor получает failure signal
- Применяет strategy (Restart/Resume/Stop/Escalate)
- Child1 и Child3 не затронуты
- Локализация failure!
```

**Supervision Strategies:**

```scala
// One-For-One: только failed child
OneForOneStrategy() {
  case _: ArithmeticException => Restart
  case _: IOException => Resume
  case _: IllegalArgumentException => Stop
  case _: Exception => Escalate
}

// All-For-One: все children
AllForOneStrategy(maxNrOfRetries = 3) {
  case _: Exception => Restart
}
```

**Message Delivery Semantics:**

| Guarantee | Description | Use Case |
|-----------|-------------|----------|
| At-Most-Once | Fire and forget | Metrics, logs |
| At-Least-Once | Retry до success | Orders, payments |
| Exactly-Once | Impossible!* | Идеально, но нереально |

*Можно эмулировать через idempotency

**Typed vs Classic Actors:**

```scala
// Classic (deprecated)
class OldActor extends Actor {
  def receive = {
    case msg: Any => // небезопасно!
  }
}

// Typed (современный подход)
object NewActor:
  sealed trait Command  // type-safe!
  case class DoWork(data: String) extends Command
  
  def apply(): Behavior[Command] = ???
```

**Reactive Manifesto принципы:**

```
Responsive: быстрый ответ
    ↑
    │
Resilient ←→ Elastic
    ↓
Message Driven
```

1. **Responsive**: быстрые response times
2. **Resilient**: остаётся responsive при failures
3. **Elastic**: responsive под разной нагрузкой
4. **Message Driven**: async message passing

**Backpressure:**

```
Fast Producer → Slow Consumer (проблема!)

Решения:
1. Buffering (ограничен размер)
2. Dropping (теряем данные)
3. Backpressure (сигнал producer замедлиться)
```

Pekko Streams реализует backpressure через Reactive Streams:
```scala
Source → Flow → Sink
  ↑       ↓       ↓
  └───demand─────┘
```

**Миграция Akka → Pekko:**
```scala
// Было (Akka)
import akka.actor._
import akka.stream._

// Стало (Pekko)
import org.apache.pekko.actor._
import org.apache.pekko.stream._

// Dependency
"org.apache.pekko" %% "pekko-actor-typed" % "1.0.x"
"org.apache.pekko" %% "pekko-stream" % "1.0.x"
```

#### Pekko Actors (Typed)
```scala
import org.apache.pekko.actor.typed._
import org.apache.pekko.actor.typed.scaladsl._

object Counter:
  sealed trait Command
  case class Increment(amount: Int) extends Command
  case class GetValue(replyTo: ActorRef[Int]) extends Command
  
  def apply(): Behavior[Command] = counter(0)
  
  private def counter(value: Int): Behavior[Command] = 
    Behaviors.receive { (context, message) =>
      message match
        case Increment(amount) => 
          counter(value + amount)
        case GetValue(replyTo) =>
          replyTo ! value
          Behaviors.same
    }
```

#### Pekko Streams
```scala
import org.apache.pekko.stream._
import org.apache.pekko.stream.scaladsl._

val source = Source(1 to 100)
val flow = Flow[Int].map(_ * 2)
val sink = Sink.foreach[Int](println)

val graph = source.via(flow).to(sink)

// Backpressure handling
val throttled = source
  .throttle(10, 1.second)
  .buffer(100, OverflowStrategy.backpressure)
  .via(flow)
  .runWith(sink)
```

#### Pekko HTTP
```scala
import org.apache.pekko.http.scaladsl._
import org.apache.pekko.http.scaladsl.server.Directives._

val route = 
  path("users" / IntNumber) { id =>
    get {
      complete(s"User $id")
    }
  } ~
  path("users") {
    post {
      entity(as[User]) { user =>
        complete(StatusCodes.Created, user)
      }
    }
  }

Http().newServerAt("localhost", 8080).bind(route)
```

**Практика:**
```scala
// Задача 1: Actor state machine
object OrderActor:
  sealed trait Command
  sealed trait State
  case object New extends State
  case object Processing extends State
  case object Completed extends State
  
  // Реализовать FSM для обработки заказов

// Задача 2: Supervision strategy
// Реализовать supervisor с restart на failure

// Задача 3: Stream processing pipeline
// CSV файл → parse → validate → write to DB
val pipeline = FileIO.fromPath(path)
  .via(CsvParsing.lineScanner())
  .map(parseRecord)
  .filter(validate)
  .grouped(100)  // batch writes
  .mapAsync(4)(writeToDB)
  .runWith(Sink.ignore)

// Задача 4: Actor clustering
// Распределённый cache с Pekko Cluster
```

**Вопросы:**
- Разница между Typed и Classic Actors?
- Как работает backpressure в Pekko Streams?
- Supervision strategies: restart vs resume vs stop?
- Кластеризация: split-brain problem и решения?

### День 4-7: Альтернативы Actor Model

**🎯 ТРЕНД 2025**: Переход от Actors к Effect Systems

#### ZIO Actors (легковесная альтернатива)
```scala
import zio.actors._

sealed trait CounterMessage
case class Increment(amount: Int) extends CounterMessage
case object GetValue extends CounterMessage

val counterHandler: Stateful[Any, Int, CounterMessage] =
  new Stateful[Any, Int, CounterMessage] {
    override def receive[A](
      state: Int,
      msg: CounterMessage[A],
      context: Context
    ): UIO[(Int, A)] = msg match
      case Increment(amount) => 
        UIO.succeed((state + amount, ()))
      case GetValue => 
        UIO.succeed((state, state))
  }
```

#### Реактивное программирование без Actors
```scala
// FS2 для streaming
import fs2._
import cats.effect._

Stream.awakeEvery[IO](1.second)
  .evalMap(_ => processEvent())
  .handleErrorWith(err => Stream.eval(logError(err)))
  .compile
  .drain
```

**Практика:**
```scala
// Задача 1: Сравнить реализации
// Реализовать rate limiter тремя способами:
// 1. Pekko Actor
// 2. ZIO + STM
// 3. Cats Effect + Ref

// Задача 2: Migrate от Akka к ZIO
// Взять Akka actor и мигрировать на ZIO

// Задача 3: Streaming comparison
// Реализовать Kafka consumer:
// - Pekko Streams
// - FS2
// - ZIO Streams
```

---

---

### 💾 Теория баз данных и транзакций

#### ACID Properties

**Atomicity (Атомарность)**
```
Транзакция либо выполняется полностью, либо не выполняется вообще

Пример:
BEGIN TRANSACTION
  UPDATE accounts SET balance = balance - 100 WHERE id = 1
  UPDATE accounts SET balance = balance + 100 WHERE id = 2
COMMIT

Если упадёт на второй операции - откатится первая (rollback)
```

**Consistency (Согласованность)**
```
База всегда переходит из одного консистентного состояния в другое

Constraints:
- Foreign keys
- Unique constraints
- Check constraints
- Application-level invariants
```

**Isolation (Изоляция)**
```
Конкурентные транзакции не видят промежуточные состояния друг друга

Уровни изоляции (от слабого к сильному):

1. Read Uncommitted
   - Грязное чтение (dirty reads)
   - Phantom reads
   - Non-repeatable reads

2. Read Committed (PostgreSQL default)
   - Нет dirty reads
   - Есть phantom reads
   - Есть non-repeatable reads

3. Repeatable Read
   - Нет dirty reads
   - Нет non-repeatable reads
   - Есть phantom reads

4. Serializable
   - Полная изоляция
   - Как будто транзакции выполнялись последовательно
```

**Durability (Долговечность)**
```
После COMMIT данные гарантированно сохранены

Механизмы:
- Write-Ahead Logging (WAL)
- Fsync
- Replication
```

#### Transaction Isolation Anomalies

**1. Dirty Read (Грязное чтение)**
```sql
-- Transaction 1
BEGIN;
UPDATE accounts SET balance = 1000 WHERE id = 1;
-- не закоммичено!

-- Transaction 2 (Read Uncommitted)
SELECT balance FROM accounts WHERE id = 1;
-- Видит 1000, хотя может откатиться!
```

**2. Non-Repeatable Read**
```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 100
-- между чтениями...

-- Transaction 2
UPDATE accounts SET balance = 200 WHERE id = 1;
COMMIT;

-- Transaction 1 (продолжение)
SELECT balance FROM accounts WHERE id = 1;  -- 200 (изменилось!)
```

**3. Phantom Read**
```sql
-- Transaction 1
BEGIN;
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- 10

-- Transaction 2
INSERT INTO orders (status) VALUES ('pending');
COMMIT;

-- Transaction 1
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- 11 (фантом!)
```

#### Locking Strategies

**Pessimistic Locking (Пессимистичная блокировка)**
```scala
// SQL
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Блокирует строку до конца транзакции

// Doobie
sql"SELECT * FROM accounts WHERE id = $id FOR UPDATE"
  .query[Account]
  .unique
  .transact(xa)
```

**Optimistic Locking (Оптимистичная блокировка)**
```scala
case class Account(
  id: Long,
  balance: BigDecimal,
  version: Long  // version field!
)

def updateWithOptimisticLock(account: Account): ConnectionIO[Int] =
  sql"""
    UPDATE accounts 
    SET balance = ${account.balance}, 
        version = ${account.version + 1}
    WHERE id = ${account.id} 
      AND version = ${account.version}
  """.update.run

// Если version изменился - UPDATE вернёт 0 строк
// Нужно retry с новой версией
```

**Сравнение:**
```
Pessimistic:
+ Гарантированно без конфликтов
- Блокировки (плохая производительность)
- Deadlocks

Optimistic:
+ Нет блокировок
+ Высокая производительность
- Нужно handle conflicts
- Retry logic
```

#### Connection Pooling Theory

**Проблема без pooling:**
```
Request 1 -> Open Connection -> Query -> Close
Request 2 -> Open Connection -> Query -> Close
Request 3 -> Open Connection -> Query -> Close

Opening connection ~100ms overhead!
```

**С pooling:**
```
Pool: [Conn1, Conn2, ..., ConnN]

Request 1 -> Borrow Conn1 -> Query -> Return Conn1
Request 2 -> Borrow Conn2 -> Query -> Return Conn2
Request 3 -> Borrow Conn1 -> Query -> Return Conn1

Connection reuse!
```

**HikariCP конфигурация:**
```scala
val config = HikariConfig()
config.setMaximumPoolSize(20)  // max connections
config.setMinimumIdle(5)       // min idle connections
config.setConnectionTimeout(30000)  // 30s timeout
config.setIdleTimeout(600000)  // 10min idle
config.setMaxLifetime(1800000) // 30min lifetime

// Formula для maximumPoolSize:
// connections = ((core_count * 2) + effective_spindle_count)
// Для SSD: connections ≈ core_count * 2
```

#### CAP Theorem

```
Consistency         Availability        Partition Tolerance
     |                    |                      |
     +--------------------+----------------------+
              Выбери любые ДВА
```

**Consistency (Согласованность)**
```
Все узлы видят одни и те же данные одновременно
```

**Availability (Доступность)**
```
Система отвечает на каждый запрос (успех или ошибка)
```

**Partition Tolerance (Устойчивость к разделению)**
```
Система продолжает работать при разрыве сети между узлами
```

**Варианты:**

```
CP (Consistency + Partition Tolerance):
- MongoDB, HBase, Redis (with replication)
- Жертвуем доступностью
- Блокируем запросы при split-brain

AP (Availability + Partition Tolerance):
- Cassandra, DynamoDB, CouchDB
- Жертвуем согласованностью
- Eventual consistency

CA (Consistency + Availability):
- RDBMS в single-node setup
- В распределённой системе невозможно!
```

#### Query Optimization

**N+1 Query Problem:**
```scala
// ПЛОХО - N+1 запросов
def getUsersWithOrders: List[(User, List[Order])] =
  val users = sql"SELECT * FROM users".query[User].to[List]
  users.map { user =>
    val orders = sql"SELECT * FROM orders WHERE user_id = ${user.id}"
      .query[Order].to[List]
    (user, orders)
  }
// 1 запрос для users + N запросов для orders!

// ХОРОШО - 1 запрос с JOIN
def getUsersWithOrders: List[(User, List[Order])] =
  sql"""
    SELECT u.*, o.*
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
  """
    .query[(User, Option[Order])]
    .to[List]
    .groupMap(_._1)(_._2.toList.flatten)
```

**Index Theory:**
```sql
-- Без индекса: O(n) - full table scan
SELECT * FROM users WHERE email = 'test@example.com';

-- С индексом: O(log n) - binary search on B-tree
CREATE INDEX idx_users_email ON users(email);

-- Composite index для сложных запросов
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Используется для:
WHERE user_id = 123 AND status = 'pending'
WHERE user_id = 123  -- prefix works!

-- НЕ используется для:
WHERE status = 'pending'  -- нет prefix
```

**Query Planner:**
```sql
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.name;

Output:
  -> Hash Join (cost=1000..5000 rows=100)
     -> Seq Scan on users (cost=0..100 rows=1000)
     -> Hash (cost=500..1000 rows=5000)
        -> Index Scan on orders (cost=0..500 rows=5000)
```

#### Event Sourcing & CQRS для баз данных

**Event Store Design:**
```scala
case class Event(
  aggregateId: String,
  eventType: String,
  data: Json,
  version: Long,
  timestamp: Instant
)

// Append-only table
CREATE TABLE events (
  aggregate_id VARCHAR NOT NULL,
  version BIGINT NOT NULL,
  event_type VARCHAR NOT NULL,
  data JSONB NOT NULL,
  timestamp TIMESTAMP NOT NULL,
  PRIMARY KEY (aggregate_id, version)
);

// Загрузка aggregate
def loadAggregate(id: String): ConnectionIO[OrderAggregate] =
  sql"SELECT * FROM events WHERE aggregate_id = $id ORDER BY version"
    .query[Event]
    .to[List]
    .map(events => events.foldLeft(OrderAggregate.empty)(_.apply(_)))
```

**Snapshot pattern:**
```scala
// Для производительности: snapshot каждые N событий
CREATE TABLE snapshots (
  aggregate_id VARCHAR PRIMARY KEY,
  version BIGINT NOT NULL,
  state JSONB NOT NULL
);

def loadWithSnapshot(id: String): ConnectionIO[OrderAggregate] =
  for
    snapshot <- sql"SELECT * FROM snapshots WHERE aggregate_id = $id"
      .query[Snapshot].option
    events <- sql"SELECT * FROM events WHERE aggregate_id = $id AND version > ${snapshot.map(_.version).getOrElse(0L)}"
      .query[Event].to[List]
  yield events.foldLeft(snapshot.map(_.state).getOrElse(OrderAggregate.empty))(_.apply(_))
```

---

## 🏗️ Неделя 4: Базы данных и интеграции

### День 1-3: Функциональные ORM

#### 📖 Теория: Functional Database Access

**Проблемы традиционного JDBC:**

```scala
// Императивный JDBC
val conn = DriverManager.getConnection(url)
try {
  val stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?")
  try {
    stmt.setLong(1, userId)
    val rs = stmt.executeQuery()
    try {
      if (rs.next()) Some(User(rs.getLong(1), rs.getString(2)))
      else None
    } finally {
      rs.close()
    }
  } finally {
    stmt.close()
  }
} finally {
  conn.close()
}
```

Проблемы:
- ❌ Ручной resource management
- ❌ Не композируется
- ❌ Error handling разбросан
- ❌ Нет type safety для SQL
- ❌ Тяжело тестировать

**Functional подход: 3 уровня абстракции**

```
Level 1: ConnectionIO[A] - описание DB операции
Level 2: Transactor       - executor с connection pool
Level 3: Effect type (IO) - интеграция с effect system
```

**ConnectionIO Monad:**

```scala
// ConnectionIO[A] - это просто description!
val query: ConnectionIO[User] = 
  sql"SELECT * FROM users WHERE id = 1".query[User].unique

// Ничего не выполнилось!
// Это просто data structure, которая описывает операцию

// Execution происходит через transactor
query.transact(xa): IO[User]
```

**Преимущества:**

1. **Композируемость**
```scala
// Маленькие queries
def findUser(id: Long): ConnectionIO[Option[User]] = ???
def updateEmail(id: Long, email: String): ConnectionIO[Int] = ???
def logAccess(userId: Long): ConnectionIO[Unit] = ???

// Композируем в transaction
val program = for {
  user  <- findUser(123).flatMap {
    case Some(u) => ConnectionIO.pure(u)
    case None    => ConnectionIO.raiseError(new Exception("Not found"))
  }
  _     <- updateEmail(user.id, "new@email.com")
  _     <- logAccess(user.id)
} yield user

// Все выполнится в одной транзакции!
```

2. **Автоматический resource management**
```scala
// Transactor автоматически:
// - открывает connection
// - создаёт statement
// - закрывает все resources
// - rollback при ошибке

program.transact(xa)  // всё автоматически!
```

3. **Testability**
```scala
// Mock transactor для тестов
val testTransactor = Transactor.fromConnection[IO](mockConnection)
program.transact(testTransactor)  // чистое тестирование
```

**Connection Pooling теория:**

```
┌─────────────────────────────┐
│   Connection Pool (10)      │
│                              │
│  [C1][C2][C3]...[C10]       │
│    ↑   ↑   ↑                │
└────┼───┼───┼────────────────┘
     │   │   │
  ┌──┴┐ ┌┴─┐ ┌┴──┐
  │T1│ │T2│ │T3 │  Threads
  └───┘ └──┘ └───┘

Без pooling:
- Каждый request создаёт connection (дорого!)
- Limited connections к DB

С pooling:
- Reuse connections
- Настраиваемый размер pool
- Timeout configuration
```

**HikariCP settings:**

```scala
HikariTransactor.newHikariTransactor[IO](
  driverClassName = "org.postgresql.Driver",
  url = "jdbc:postgresql://localhost/mydb",
  user = "user",
  pass = "pass",
  
  // Pool settings
  connectEC = ???,           // ExecutionContext для JDBC
  
  // Эти параметры критичны для performance!
  config = { hikariConfig =>
    hikariConfig.setMaximumPoolSize(20)     // макс connections
    hikariConfig.setMinimumIdle(5)          // мин idle connections
    hikariConfig.setConnectionTimeout(30000) // 30s timeout
    hikariConfig.setIdleTimeout(600000)     // 10m idle timeout
    hikariConfig.setMaxLifetime(1800000)    // 30m max lifetime
  }
)
```

**Transaction Isolation Levels:**

```
Read Uncommitted:  Dirty reads possible
     ↓
Read Committed:    No dirty reads (PostgreSQL default)
     ↓
Repeatable Read:   Same data in transaction
     ↓
Serializable:      Полная изоляция (самый медленный)

Выбор зависит от:
- Consistency requirements
- Performance needs
- Database capabilities
```

**ACID Properties:**

```
Atomicity:    All or nothing
Consistency:  Valid state → Valid state
Isolation:    Concurrent transactions не мешают
Durability:   Committed data persists
```

Пример нарушения без transactions:
```scala
// БЕЗ transaction
def transfer(from: Long, to: Long, amount: Double) = {
  withdraw(from, amount).transact(xa)   // Success
  // CRASH здесь!
  deposit(to, amount).transact(xa)      // Не выполнится
}
// Money исчезли! ❌

// С transaction
def transfer(from: Long, to: Long, amount: Double) = {
  (for {
    _ <- withdraw(from, amount)
    _ <- deposit(to, amount)
  } yield ()).transact(xa)
}
// Atomicity гарантирована! ✅
```

**Comparison: Doobie vs Slick vs Quill**

| Feature | Doobie | Slick | Quill |
|---------|--------|-------|-------|
| **Approach** | Plain SQL | Type-safe DSL | Quotation/Macros |
| **Compile-time check** | ❌ | ✅ | ✅ |
| **Learning curve** | Low | Medium | High |
| **Flexibility** | High (SQL) | Medium (DSL) | Medium (Quotation) |
| **Performance** | Good | Good | Excellent |
| **Effect type** | ConnectionIO | DBIO/Future | F[_] polymorphic |

**Когда использовать каждый:**

**Doobie:**
- ✅ Complex SQL queries
- ✅ Legacy databases
- ✅ SQL expertise в команде
- ✅ Cats Effect stack

**Slick:**
- ✅ Type safety paramount
- ✅ Schema evolution
- ✅ Java interop
- ✅ Future-based code

**Quill:**
- ✅ Maximum performance
- ✅ Compile-time queries
- ✅ Polyglot (multiple effects)
- ✅ Smaller codebase

#### Doobie (FP-friendly JDBC)
```scala
import doobie._
import doobie.implicits._
import cats.effect._

// Type-safe queries
def findUser(id: Long): ConnectionIO[Option[User]] =
  sql"SELECT id, name, email FROM users WHERE id = $id"
    .query[User]
    .option

// Transactional composition
val program: ConnectionIO[Unit] = for
  user <- findUser(1)
  _ <- user match
    case Some(u) => updateUser(u.copy(email = "new@email.com"))
    case None => FC.raiseError(new Exception("Not found"))
yield ()

// Connection pooling
val transactor: Resource[IO, HikariTransactor[IO]] = 
  HikariTransactor.newHikariTransactor[IO](
    "org.postgresql.Driver",
    "jdbc:postgresql:mydb",
    "user",
    "pass"
  )

// Execute
program.transact(xa)
```

#### Slick (FRM - Functional Relational Mapping)
```scala
import slick.jdbc.PostgresProfile.api._

class Users(tag: Tag) extends Table[User](tag, "users") {
  def id = column[Long]("id", O.PrimaryKey, O.AutoInc)
  def name = column[String]("name")
  def email = column[String]("email")
  def * = (id, name, email).mapTo[User]
}

val users = TableQuery[Users]

// Type-safe queries
val query = users
  .filter(_.name like "%John%")
  .sortBy(_.id)
  .take(10)

// Compile-time SQL generation
val action: DBIO[Seq[User]] = query.result

// Execute
db.run(action): Future[Seq[User]]
```

#### Quill (Compile-time queries)
```scala
import io.getquill._

val ctx = new PostgresZioJdbcContext(SnakeCase)
import ctx._

case class User(id: Long, name: String, email: String)

// Quoted queries - checked at compile time!
def findUser(id: Long) = quote {
  query[User].filter(_.id == lift(id))
}

// Compose queries
def activeUsers = quote {
  query[User].filter(_.isActive == true)
}

// Execute
ctx.run(findUser(123)): ZIO[Any, Throwable, List[User]]
```

**Практика:**
```scala
// Задача 1: Complex join с Doobie
def getUserWithOrders(userId: Long): ConnectionIO[UserWithOrders] = ???

// Задача 2: Migration с Flyway
// Настроить миграции для multi-tenant приложения

// Задача 3: Quill compile-time safety
// Написать сложный query с joins, проверить ошибки компиляции

// Задача 4: Connection pooling optimization
// Настроить HikariCP для high-throughput приложения

// Задача 5: Transaction isolation
// Реализовать optimistic locking с version column
```

### День 4-7: Message Brokers и Event Streaming

#### Kafka с функциональными обёртками

**fs2-kafka (Cats Effect):**
```scala
import fs2.kafka._
import cats.effect._

val consumerSettings = 
  ConsumerSettings[IO, String, String]
    .withBootstrapServers("localhost:9092")
    .withGroupId("group")
    .withAutoOffsetReset(AutoOffsetReset.Earliest)

val stream = KafkaConsumer
  .stream(consumerSettings)
  .subscribeTo("topic")
  .records
  .mapAsync(25) { committable =>
    processRecord(committable.record).as(committable.offset)
  }
  .through(commitBatchWithin(500, 15.seconds))

stream.compile.drain
```

**zio-kafka (ZIO):**
```scala
import zio._
import zio.kafka.consumer._
import zio.kafka.serde._

val consumer = Consumer.subscribeAnd(Subscription.topics("topic"))
  .plainStream(Serde.string, Serde.string)
  .mapZIOPar(25) { record =>
    processRecord(record.value).as(record.offset)
  }
  .aggregateAsync(Consumer.offsetBatches)
  .mapZIO(_.commit)
  .runDrain
```

#### Event Sourcing паттерн
```scala
// Event Store с Kafka
sealed trait OrderEvent
case class OrderCreated(orderId: String, items: List[Item]) extends OrderEvent
case class OrderPaid(orderId: String, amount: Double) extends OrderEvent
case class OrderShipped(orderId: String) extends OrderEvent

case class OrderState(
  id: String,
  items: List[Item],
  isPaid: Boolean,
  isShipped: Boolean
)

def aggregate(state: OrderState, event: OrderEvent): OrderState = event match
  case OrderCreated(id, items) => 
    OrderState(id, items, false, false)
  case OrderPaid(id, amount) => 
    state.copy(isPaid = true)
  case OrderShipped(id) => 
    state.copy(isShipped = true)

// Read side projection
def projectOrders(events: Stream[OrderEvent]): Stream[OrderState] =
  events.scan(OrderState.empty)(aggregate)
```

**Практика:**
```scala
// Задача 1: Exactly-once semantics
// Реализовать idempotent consumer с deduplication

// Задача 2: Dead letter queue
// Настроить DLQ для failed messages

// Задача 3: CQRS with Event Sourcing
// Реализовать command/query separation для Order system

// Задача 4: Kafka Streams DSL
// Реализовать real-time aggregation с windowing

// Задача 5: Schema Registry
// Настроить Avro/Protobuf с Schema Registry
```

---

---

### 🌐 Теория распределённых систем

#### Распределённые системы: базовые концепции

**Определение**: Распределённая система - это совокупность независимых компьютеров, которые представляются пользователям как единая согласованная система.

**Вызовы распределённых систем:**
```
1. Частичные сбои (Partial Failures)
2. Ненадёжная сеть (Unreliable Network)
3. Ненадёжные часы (Unreliable Clocks)
4. Задержки (Latency)
```

#### Fallacies of Distributed Computing (8 заблуждений)

```
1. Сеть надёжна                    ❌
2. Latency = 0                     ❌  
3. Пропускная способность infinite  ❌
4. Сеть безопасна                  ❌
5. Топология не меняется           ❌
6. Один администратор              ❌
7. Транспорт бесплатный            ❌
8. Сеть однородная                 ❌
```

#### CAP Theorem (подробно)

**Формальное определение:**
```
В распределённой системе невозможно одновременно гарантировать:
- Consistency (линеаризуемость)
- Availability (каждый узел отвечает)
- Partition tolerance (работа при разрыве сети)
```

**Практические компромиссы:**

**CP системы (Consistency + Partition Tolerance):**
```scala
// Пример: Raft consensus
// При разделении сети: minority partition недоступна

trait RaftNode:
  def write(key: String, value: String): Either[NotLeader, Unit]
  
val cluster = List(node1, node2, node3)

// Network partition: [node1, node2] | [node3]
// Majority (node1, node2) - работает
// Minority (node3) - возвращает NotLeader

// Жертва: availability (node3 недоступен)
// Выигрыш: consistency (все видят одни данные)
```

**AP системы (Availability + Partition Tolerance):**
```scala
// Пример: Cassandra с eventual consistency

// При разделении сети: оба partition работают
// [node1, node2] <-X-> [node3]

node1.write("key", "value1")  // OK
node3.write("key", "value2")  // OK (конфликт!)

// После восстановления сети: conflict resolution
// Last-Write-Wins, Vector Clocks, CRDTs

// Жертва: consistency (временно разные данные)
// Выигрыш: availability (все узлы доступны)
```

#### Consistency Models

**Strong Consistency (Линеаризуемость):**
```
Операции выглядят так, как будто выполнялись атомарно 
в некотором глобальном порядке

Client 1: write(x, 1) ----OK----> read(x) ----1--->
                                     ^
                                     |
Client 2:                    read(x) ----1--->

Все клиенты видят изменения мгновенно
```

**Eventual Consistency:**
```
Если не будет новых updates, все replicas сойдутся к одному значению

Client 1: write(x, 1) ----OK----> 
                                   
Client 2:                    read(x) ----0----> (stale!)
                             wait...
                             read(x) ----1----> (caught up!)

Возможны stale reads, но система сходится
```

**Causal Consistency:**
```
Причинно-связанные операции видны в правильном порядке

Client 1: write(x, 1) --> write(y, 2)  // y зависит от x
                                        
Client 2: read(x) -> 0, read(y) -> 0   ✅
          read(x) -> 1, read(y) -> 0   ✅
          read(x) -> 1, read(y) -> 2   ✅
          read(x) -> 0, read(y) -> 2   ❌ (нарушение causality!)
```

#### Distributed Transactions

**Two-Phase Commit (2PC):**
```
Coordinator              Participant 1       Participant 2
    |                          |                  |
    |------PREPARE--------->   |                  |
    |------PREPARE---------------------->         |
    |                          |                  |
    |<------VOTE-YES-------    |                  |
    |<------VOTE-YES-----------------------       |
    |                          |                  |
    |------COMMIT----------->  |                  |
    |------COMMIT----------------------->         |
    |                          |                  |
    
Проблема: Coordinator - single point of failure
Если упал на фазе commit - участники заблокированы!
```

**Saga Pattern (альтернатива 2PC):**
```scala
// Декомпозиция на локальные транзакции + компенсации

sealed trait SagaStep[A]
case class Transaction[A](
  execute: Task[A],
  compensate: A => Task[Unit]  // откат!
) extends SagaStep[A]

// Пример: Order processing
val orderSaga = for
  reservation <- reserveInventory(items)
    .compensate(cancelReservation)
  payment <- processPayment(total)
    .compensate(refundPayment)
  shipping <- scheduleShipping(address)
    .compensate(cancelShipping)
yield Order(reservation, payment, shipping)

// Если shipping упал - откат payment и reservation
orderSaga.run.catchAll { error =>
  // Автоматический rollback через compensations
  orderSaga.compensate
}
```

**Сравнение:**
```
2PC:
+ Атомарность
+ ACID гарантии
- Blocking (locks)
- Single point of failure
- Плохая производительность

Saga:
+ Non-blocking
+ Лучше масштабируется
- Eventual consistency
- Сложнее reasoning
- Нужны idempotent compensations
```

#### Distributed Consensus (Консенсус)

**Проблема**: Как N узлов договариваются о значении при сбоях и сетевых разделениях?

**Raft Algorithm:**
```
Состояния узла:
1. Follower - следует за leader
2. Candidate - кандидат в leaders
3. Leader - координирует операции

Election:
1. Follower timeout -> становится Candidate
2. Candidate голосует за себя, запрашивает голоса
3. Majority голосов -> становится Leader
4. Leader отправляет heartbeats

Log Replication:
1. Client -> Leader: append entry
2. Leader -> Followers: replicate entry
3. Followers ack
4. Leader: commit (majority acks)
5. Leader -> Followers: notify commit
```

**В Scala:**
```scala
trait RaftNode[Command]:
  def submitCommand(cmd: Command): Task[Unit]
  def getState: Task[NodeState]
  
sealed trait NodeState
case class Leader(term: Long) extends NodeState
case class Follower(term: Long, leader: NodeId) extends NodeState
case class Candidate(term: Long) extends NodeState

// Log entry
case class LogEntry[C](
  term: Long,
  index: Long,
  command: C
)

// Consensus invariant
def consensusProperty(nodes: List[RaftNode]): Boolean =
  // Все committed entries одинаковы на всех узлах
  val committedLogs = nodes.map(_.getCommittedLog)
  committedLogs.forall(_ == committedLogs.head)
```

#### Time and Ordering

**Lamport Clock (Логические часы):**
```scala
class LamportClock:
  private var clock: Long = 0
  
  def tick(): Long =
    clock += 1
    clock
    
  def update(received: Long): Long =
    clock = math.max(clock, received) + 1
    clock

// Happened-before отношение
// e1 -> e2 если timestamp(e1) < timestamp(e2)

case class Event(
  node: NodeId,
  timestamp: Long,  // Lamport timestamp
  data: String
)

// Частичный порядок событий
def happenedBefore(e1: Event, e2: Event): Boolean =
  e1.timestamp < e2.timestamp
```

**Vector Clock (для causal consistency):**
```scala
case class VectorClock(clocks: Map[NodeId, Long]):
  def increment(nodeId: NodeId): VectorClock =
    VectorClock(clocks.updated(nodeId, clocks.getOrElse(nodeId, 0L) + 1))
    
  def merge(other: VectorClock): VectorClock =
    val keys = clocks.keySet ++ other.clocks.keySet
    VectorClock(keys.map { k =>
      k -> math.max(clocks.getOrElse(k, 0L), other.clocks.getOrElse(k, 0L))
    }.toMap)
    
  def happenedBefore(other: VectorClock): Boolean =
    clocks.forall { case (k, v) =>
      v <= other.clocks.getOrElse(k, 0L)
    } && this != other

// Пример
val v1 = VectorClock(Map("A" -> 1, "B" -> 0))
val v2 = VectorClock(Map("A" -> 1, "B" -> 1))
val v3 = VectorClock(Map("A" -> 2, "B" -> 0))

v1.happenedBefore(v2)  // true (B увеличился)
v1.happenedBefore(v3)  // false (concurrent!)
```

#### Conflict-Free Replicated Data Types (CRDTs)

**Определение**: Структуры данных, которые могут безопасно реплицироваться без координации.

**G-Counter (Grow-only Counter):**
```scala
case class GCounter(counts: Map[NodeId, Long]):
  def increment(nodeId: NodeId): GCounter =
    GCounter(counts.updated(nodeId, counts.getOrElse(nodeId, 0L) + 1))
    
  def value: Long = counts.values.sum
  
  def merge(other: GCounter): GCounter =
    val keys = counts.keySet ++ other.counts.keySet
    GCounter(keys.map { k =>
      k -> math.max(counts.getOrElse(k, 0L), other.counts.getOrElse(k, 0L))
    }.toMap)

// Replica 1: GCounter(Map("A" -> 5, "B" -> 3))
// Replica 2: GCounter(Map("A" -> 4, "B" -> 7))
// Merge: GCounter(Map("A" -> 5, "B" -> 7)) = 12

// Свойство: merge коммутативен и идемпотентен
g1.merge(g2) == g2.merge(g1)
g1.merge(g1) == g1
```

**LWW-Register (Last-Write-Wins):**
```scala
case class LWWRegister[A](
  value: A,
  timestamp: Long,
  nodeId: NodeId  // для tie-breaking
):
  def write(newValue: A, ts: Long, node: NodeId): LWWRegister[A] =
    if ts > timestamp || (ts == timestamp && node > nodeId) then
      LWWRegister(newValue, ts, node)
    else
      this
      
  def merge(other: LWWRegister[A]): LWWRegister[A] =
    if other.timestamp > timestamp then other
    else if other.timestamp < timestamp then this
    else if other.nodeId > nodeId then other
    else this
```

**OR-Set (Observed-Remove Set):**
```scala
case class ORSet[A](
  elements: Map[A, Set[UUID]],  // element -> unique tags
  tombstones: Set[UUID]          // removed tags
):
  def add(elem: A): ORSet[A] =
    val tag = UUID.randomUUID()
    ORSet(
      elements.updated(elem, elements.getOrElse(elem, Set.empty) + tag),
      tombstones
    )
    
  def remove(elem: A): ORSet[A] =
    val tags = elements.getOrElse(elem, Set.empty)
    ORSet(elements - elem, tombstones ++ tags)
    
  def contains(elem: A): Boolean =
    elements.get(elem).exists(tags => (tags -- tombstones).nonEmpty)
    
  def merge(other: ORSet[A]): ORSet[A] =
    val mergedElements = (elements.keySet ++ other.elements.keySet).map { elem =>
      elem -> (elements.getOrElse(elem, Set.empty) ++ other.elements.getOrElse(elem, Set.empty))
    }.toMap
    ORSet(mergedElements, tombstones ++ other.tombstones)

// Concurrent add/remove решаются правильно!
```

#### Service Mesh Patterns

**Circuit Breaker Theory:**
```
States: Closed -> Open -> Half-Open -> Closed

Closed: нормальная работа
  failures > threshold -> Open

Open: все запросы fail fast
  timeout elapsed -> Half-Open

Half-Open: пробный запрос
  success -> Closed
  failure -> Open

Математика:
failure_rate = failures / total_requests
if failure_rate > threshold then OPEN
```

**Bulkhead Pattern:**
```
Изоляция ресурсов для fault isolation

ThreadPool A: [10 threads] -> Service A
ThreadPool B: [10 threads] -> Service B

Если Service A падает:
- ThreadPool A исчерпан
- ThreadPool B продолжает работать
- Service B доступен!

Without Bulkhead:
SharedPool [20 threads]
- Service A падает
- Все threads заняты
- Service B тоже недоступен!
```

**Rate Limiting Algorithms:**

**1. Token Bucket:**
```scala
class TokenBucket(
  capacity: Int,
  refillRate: Int  // tokens per second
):
  private var tokens: Int = capacity
  private var lastRefill: Instant = Instant.now()
  
  def tryAcquire(): Boolean = synchronized {
    refill()
    if tokens > 0 then
      tokens -= 1
      true
    else
      false
  }
  
  private def refill(): Unit =
    val now = Instant.now()
    val elapsed = Duration.between(lastRefill, now).getSeconds
    val newTokens = (elapsed * refillRate).toInt
    tokens = math.min(capacity, tokens + newTokens)
    lastRefill = now
```

**2. Sliding Window:**
```scala
class SlidingWindow(
  maxRequests: Int,
  windowSize: Duration
):
  private val requests = mutable.Queue[Instant]()
  
  def tryAcquire(): Boolean = synchronized {
    val now = Instant.now()
    val cutoff = now.minus(windowSize)
    
    // Remove old requests
    while requests.nonEmpty && requests.head.isBefore(cutoff) do
      requests.dequeue()
    
    if requests.size < maxRequests then
      requests.enqueue(now)
      true
    else
      false
  }
```

---

## 🧠 Неделя 5: Системный дизайн для Senior уровня

#### 📖 Фундаментальная теория: Distributed Systems

**Определение:**
Distributed system - система, компоненты которой находятся на разных networked computers и координируются через message passing.

**Основные challenges:**

1. **Network is unreliable**
```
Request  →  [Network] → Response
            ↑
            ├─ Packet loss
            ├─ High latency  
            ├─ Partial failures
            └─ Network partitions
```

2. **Clocks are not synchronized**
```
Server A time: 10:00:01.000
Server B time: 10:00:00.998
Server C time: 10:00:01.003

Невозможно точное ordering events!
```

3. **Partial failures**
```
┌────────┐     ┌────────┐     ┌────────┐
│Service │     │Service │     │Service │
│   A    │ OK  │   B    │ DOWN│   C    │ OK
└────────┘     └────────┘     └────────┘

Система работает частично - сложно!
```

**CAP Theorem (фундаментальный):**

```
        Consistency
             ▲
            ╱ ╲
           ╱   ╲
          ╱     ╲
         ╱  CAP  ╲
        ╱ Theorem ╲
       ╱           ╲
Availability ─────── Partition Tolerance

Можно выбрать только 2 из 3!
```

- **Consistency**: все nodes видят одинаковые данные
- **Availability**: каждый request получает response
- **Partition Tolerance**: система работает при network partition

**Примеры:**

```scala
// CP системы (Consistency + Partition Tolerance)
// MongoDB, HBase, Redis (с потерей availability)
// Trade-off: во время partition могут быть unavailable

// AP системы (Availability + Partition Tolerance)  
// Cassandra, DynamoDB, CouchDB
// Trade-off: eventual consistency, могут быть stale reads

// CA системы (Consistency + Availability)
// Traditional RDBMS (MySQL, PostgreSQL)
// Trade-off: не работают при partition
// В distributed мире практически не существуют!
```

**BASE vs ACID:**

```
ACID (Traditional):
- Atomicity
- Consistency
- Isolation
- Durability

BASE (Distributed):
- Basically Available
- Soft state
- Eventually consistent
```

**Consistency Models:**

```
Strong Consistency:
  Write(x=1) → Read(x) = 1  (всегда!)
  Медленно, сложно в distributed

Eventual Consistency:
  Write(x=1) → Read(x) = 0  (сейчас)
           │
           └→ Read(x) = 1   (позже)
  Быстро, но требует conflict resolution

Causal Consistency:
  Если A → B, то все видят в этом порядке
  Середина между strong и eventual
```

**Distributed Transactions паттерны:**

1. **2-Phase Commit (2PC)**
```
Coordinator
    │
    ├──► Participant 1: Prepare?
    ├──► Participant 2: Prepare?
    ├──► Participant 3: Prepare?
    │
    └──► All OK?
         ├─ YES → Commit all
         └─ NO  → Abort all

Проблемы:
❌ Blocking protocol
❌ Single point of failure (coordinator)
❌ Slow (2 round trips)
```

2. **Saga Pattern (лучше для микросервисов)**
```scala
// Sequence of local transactions
val saga = for {
  orderId    <- createOrder()
  _          <- reserveInventory(orderId)
  _          <- processPayment(orderId)
  _          <- shipOrder(orderId)
} yield orderId

// С compensating transactions при failure
def compensate = {
  cancelShipping
    .flatMap(_ => refundPayment)
    .flatMap(_ => releaseInventory)
    .flatMap(_ => cancelOrder)
}

// Choreography: каждый service знает что делать
// Orchestration: центральный coordinator
```

**Sharding/Partitioning:**

```
Зачем: один server не справляется с нагрузкой

Strategies:
1. Hash-based:
   userId.hashCode % numberOfShards
   
2. Range-based:
   Users 1-1000    → Shard 1
   Users 1001-2000 → Shard 2
   
3. Geography-based:
   EU users → EU shard
   US users → US shard

Trade-offs:
✅ Horizontal scalability
❌ Сложность queries
❌ Rebalancing при добавлении shards
```

**Replication:**

```
Primary-Replica:
┌─────────┐
│ Primary │ ←── Writes
└────┬────┘
     │
  ┌──┴──┐
  │     │
┌─▼─┐ ┌─▼─┐
│R1 │ │R2 │ ←── Reads
└───┘ └───┘

Multi-Primary:
┌──────┐   ┌──────┐
│Primary│←→ │Primary│
│  DC1  │   │  DC2  │
└──────┘   └──────┘
Conflict resolution needed!
```

**Consensus Algorithms:**

```
Raft / Paxos:
- Выбор leader
- Log replication
- Гарантия consistency

Используется в:
- etcd (Kubernetes)
- Consul
- ZooKeeper
```

### День 1-3: Microservices Architecture

#### 📖 Теория: Microservices Design

**Monolith vs Microservices:**

```
Monolith:
┌──────────────────────────┐
│  All code in one app     │
│  ┌────┐ ┌────┐ ┌────┐   │
│  │Auth│ │Order│ │Pay│   │
│  └────┘ └────┘ └────┘   │
└──────────────────────────┘
       Single DB

Microservices:
┌──────┐  ┌───────┐  ┌────────┐
│ Auth │  │ Order │  │Payment │
│Service│  │Service│  │Service │
└───┬──┘  └───┬───┘  └───┬────┘
    │         │          │
  ┌─▼─┐     ┌─▼─┐      ┌─▼─┐
  │DB1│     │DB2│      │DB3│
  └───┘     └───┘      └───┘
```

**Когда microservices?**

✅ **Да:**
- Большая команда (>20 devs)
- Независимый scaling нужен
- Разные tech stacks
- Continuous deployment
- Domain complexity

❌ **Нет:**
- Малая команда (<5 devs)
- Simple domain
- Тесная связанность
- Не нужен independent deployment

**Bounded Contexts (DDD):**

```
┌────────────────┐  ┌────────────────┐
│  Order Context │  │ Payment Context│
│                │  │                │
│  Customer      │  │  Customer      │
│  - orderId     │  │  - paymentId   │
│  - items       │  │  - creditCard  │
└────────────────┘  └────────────────┘
     Same entity, different meaning!
```

**Inter-service Communication:**

```
Synchronous (REST/gRPC):
Service A ──request──► Service B
         ◄─response──

+ Simple
- Coupling
- Cascading failures

Asynchronous (Events):
Service A ──event──► Message Broker
                          │
                     ┌────┴────┐
                     │         │
                Service B  Service C

+ Loose coupling
+ Resilient
- Complexity
- Eventual consistency
```

**Service Discovery & Communication:**

```scala
// gRPC сервис (Scala 3)
import scalapb.zio_grpc.*
import io.grpc.ServerBuilder

trait UserService:
  def getUser(request: GetUserRequest): ZIO[Any, Status, User]
  def createUser(request: CreateUserRequest): ZIO[Any, Status, User]

object UserServiceImpl extends UserService:
  def getUser(request: GetUserRequest) = 
    ZIO.attempt(findUser(request.id))
      .mapError(e => Status.INTERNAL.withDescription(e.getMessage))

// HTTP4s service (Cats Effect)
import org.http4s._
import org.http4s.dsl.io._

val userRoutes = HttpRoutes.of[IO] {
  case GET -> Root / "users" / IntVar(id) =>
    getUserById(id).flatMap {
      case Some(user) => Ok(user.asJson)
      case None => NotFound()
    }
  case req @ POST -> Root / "users" =>
    req.as[User].flatMap(createUser).flatMap(u => Created(u.asJson))
}
```

#### Circuit Breaker Pattern
```scala
import cats.effect._
import scala.concurrent.duration._

class CircuitBreaker[F[_]: Temporal](
  maxFailures: Int,
  resetTimeout: FiniteDuration
):
  enum State:
    case Closed
    case Open(openedAt: Instant)
    case HalfOpen
  
  def protect[A](fa: F[A]): F[A] = ???
  
// Использование
val cb = CircuitBreaker[IO](maxFailures = 5, resetTimeout = 30.seconds)
cb.protect(callExternalAPI())
```

#### Service Mesh паттерны
```scala
// Retry с exponential backoff
import retry._
import cats.effect.IO

def callWithRetry[A](fa: IO[A]): IO[A] =
  retryingOnAllErrors(
    policy = RetryPolicies.exponentialBackoff(1.second),
    onError = (err, details) => 
      IO.println(s"Retry attempt ${details.retriesSoFar}: ${err.getMessage}")
  )(fa)

// Load balancing
class LoadBalancer[F[_]: Concurrent](endpoints: List[Uri]):
  private val counter = Ref.unsafe[F, Int](0)
  
  def nextEndpoint: F[Uri] = 
    counter.modify(n => (n + 1, endpoints(n % endpoints.length)))
```

**Практика:**
```scala
// Задача 1: Distributed Tracing
// Интегрировать OpenTelemetry для trace propagation

// Задача 2: Service mesh
// Дизайн для 10+ микросервисов с Istio patterns

// Задача 3: API Gateway
// Реализовать gateway с rate limiting и authentication

// Задача 4: Saga Pattern
// Distributed transaction для order processing
```

### День 4-7: Event-Driven Architecture

#### Event Bus Design
```scala
// Typed Event Bus с ZIO
trait EventBus:
  def publish[E: Tag](event: E): UIO[Unit]
  def subscribe[E: Tag]: ZStream[Any, Nothing, E]

object EventBus:
  def make: UIO[EventBus] = for
    hub <- Hub.unbounded[Any]
  yield new EventBus:
    def publish[E: Tag](event: E) = hub.publish(event)
    def subscribe[E: Tag] = 
      ZStream.fromHub(hub).collect { case e: E => e }

// Использование
val bus <- EventBus.make

// Publisher
bus.publish(OrderCreated(orderId, items))

// Subscriber
bus.subscribe[OrderCreated]
  .foreach(event => handleOrderCreated(event))
```

#### CQRS Implementation
```scala
// Command side
sealed trait OrderCommand
case class CreateOrder(items: List[Item]) extends OrderCommand
case class PayOrder(orderId: String) extends OrderCommand

trait OrderCommandHandler:
  def handle(cmd: OrderCommand): ZIO[Any, AppError, Unit]

// Query side
trait OrderQueryHandler:
  def findById(id: String): ZIO[Any, AppError, Option[OrderView]]
  def findByUser(userId: String): ZIO[Any, AppError, List[OrderView]]

// Event store
trait EventStore:
  def saveEvents(streamId: String, events: List[DomainEvent]): Task[Unit]
  def loadEvents(streamId: String): Stream[Throwable, DomainEvent]
```

**Проектирование систем:**

**Задача 1: URL Shortener**
- 100M URLs/day
- 10:1 read/write ratio
- Global distribution
- Design: sharding, caching, database schema

**Задача 2: Real-time Chat**
- 1M concurrent users
- Message delivery guarantees
- WebSocket vs long polling
- Scaling strategy

**Задача 3: E-commerce Platform**
- Inventory management
- Order processing (Saga)
- Payment integration
- Event sourcing for audit log

**Задача 4: Metrics System**
- Time-series data
- Aggregation windows
- Retention policies
- Query optimization

---

---

### 🤖 Теория машинного обучения и AI для инженеров

#### Основы Machine Learning

**Типы обучения:**

**1. Supervised Learning (Обучение с учителем)**
```
Input: (X, Y) pairs - features and labels
Goal: Learn function f: X -> Y

Examples:
- Classification: spam detection, image recognition
- Regression: price prediction, demand forecasting

Training: Minimize loss function
  loss = (1/N) Σ L(f(x_i), y_i)
```

**2. Unsupervised Learning (Без учителя)**
```
Input: X - только features
Goal: Find patterns/structure

Examples:
- Clustering: customer segmentation
- Dimensionality reduction: PCA, t-SNE
- Anomaly detection: fraud detection
```

**3. Reinforcement Learning (Обучение с подкреплением)**
```
Agent interacts with Environment
Receives rewards/penalties
Goal: Learn policy π to maximize cumulative reward

Examples:
- Game playing (AlphaGo)
- Robotics
- Recommendation systems
```

#### Model Evaluation Theory

**Train/Validation/Test Split:**
```
Total Data: 100%
├── Train: 70% (для обучения)
├── Validation: 15% (для hyperparameter tuning)
└── Test: 15% (для финальной оценки)

NEVER train on test set!
NEVER tune hyperparameters on test set!
```

**Metrics для классификации:**
```
Confusion Matrix:
                Predicted
              Pos    Neg
Actual  Pos   TP     FN
        Neg   FP     TN

Precision = TP / (TP + FP)  // из предсказанных положительных
Recall = TP / (TP + FN)     // из реальных положительных
F1 = 2 * (Precision * Recall) / (Precision + Recall)

Accuracy = (TP + TN) / (TP + TN + FP + FN)
```

**В Scala:**
```scala
case class ConfusionMatrix(tp: Int, fp: Int, fn: Int, tn: Int):
  def precision: Double = tp.toDouble / (tp + fp)
  def recall: Double = tp.toDouble / (tp + fn)
  def f1Score: Double = 2 * (precision * recall) / (precision + recall)
  def accuracy: Double = (tp + tn).toDouble / (tp + tn + fp + fn)

// Пример: бинарная классификация
def evaluate(predictions: List[(Boolean, Boolean)]): ConfusionMatrix =
  predictions.foldLeft(ConfusionMatrix(0, 0, 0, 0)) {
    case (cm, (predicted, actual)) =>
      (predicted, actual) match
        case (true, true) => cm.copy(tp = cm.tp + 1)
        case (true, false) => cm.copy(fp = cm.fp + 1)
        case (false, true) => cm.copy(fn = cm.fn + 1)
        case (false, false) => cm.copy(tn = cm.tn + 1)
  }
```

**Overfitting vs Underfitting:**
```
Underfitting (High Bias):
- Модель слишком простая
- Плохо на train и test
- Solution: более сложная модель, больше features

Overfitting (High Variance):
- Модель слишком сложная
- Отлично на train, плохо на test
- Solution: regularization, больше данных, early stopping

Sweet Spot:
- Оптимальная сложность
- Хорошо на train и test
```

**Regularization:**
```scala
// L1 Regularization (Lasso)
def l1Loss(weights: Vector, lambda: Double): Double =
  weights.map(math.abs).sum * lambda

// L2 Regularization (Ridge)
def l2Loss(weights: Vector, lambda: Double): Double =
  weights.map(w => w * w).sum * lambda

// Total loss with regularization
def totalLoss(predictions: Vector, actuals: Vector, weights: Vector): Double =
  val dataLoss = meanSquaredError(predictions, actuals)
  val regLoss = l2Loss(weights, lambda = 0.01)
  dataLoss + regLoss
```

#### Large Language Models (LLMs)

**Transformer Architecture:**
```
Input -> Embedding -> Position Encoding
  |
  v
Multi-Head Self-Attention
  |
  v
Feed-Forward Network
  |
  v
Repeat N times (layers)
  |
  v
Output Probabilities

Key Innovation: Self-Attention
- Каждый token "смотрит" на все другие tokens
- Учится контекстуальным зависимостям
- Параллелизуется хорошо (в отличие от RNN)
```

**Attention Mechanism:**
```
Query (Q), Key (K), Value (V)

Attention(Q, K, V) = softmax(Q * K^T / sqrt(d_k)) * V

Интуиция:
1. Q задаёт вопрос: "что мне нужно?"
2. K отвечает: "у меня есть это"
3. Similarity(Q, K) определяет вес
4. V содержит информацию
5. Weighted sum of V = output
```

**В Scala (упрощённо):**
```scala
case class Attention(
  queryWeights: Matrix,
  keyWeights: Matrix,
  valueWeights: Matrix
):
  def apply(input: Matrix): Matrix =
    val queries = input * queryWeights
    val keys = input * keyWeights
    val values = input * valueWeights
    
    // Scaled dot-product attention
    val scores = (queries * keys.transpose) / math.sqrt(keys.cols)
    val weights = softmax(scores)
    weights * values
    
  def softmax(x: Matrix): Matrix =
    x.mapRows { row =>
      val expRow = row.map(math.exp)
      val sum = expRow.sum
      expRow.map(_ / sum)
    }
```

**Tokenization:**
```scala
// BPE (Byte Pair Encoding)
trait Tokenizer:
  def encode(text: String): List[Int]
  def decode(tokens: List[Int]): String

// Пример
val tokenizer = GPT4Tokenizer()
val tokens = tokenizer.encode("Hello, world!")
// [15496, 11, 1917, 0]

val text = tokenizer.decode(tokens)
// "Hello, world!"

// Special tokens
object SpecialTokens:
  val BOS = 1  // Beginning of Sequence
  val EOS = 2  // End of Sequence
  val PAD = 0  // Padding
  val UNK = 3  // Unknown
```

**Prompt Engineering Theory:**
```
Few-Shot Prompting:
  Task: Classify sentiment
  
  Example 1: "Great product!" -> Positive
  Example 2: "Terrible service" -> Negative
  Example 3: "It's okay" -> Neutral
  
  Input: "Amazing experience!"
  Output: Positive

Chain-of-Thought:
  Task: Solve math problem
  
  Q: John has 5 apples. He gives 2 to Mary. How many left?
  A: Let's think step by step:
     1. John starts with 5 apples
     2. He gives away 2 apples
     3. 5 - 2 = 3
     Answer: 3 apples
```

#### RAG (Retrieval-Augmented Generation)

**Architecture:**
```
User Query
   |
   v
Embedding Model
   |
   v
Vector Search (retrieve relevant docs)
   |
   v
Context Construction
   |
   v
LLM (with augmented context)
   |
   v
Response
```

**Vector Similarity:**
```scala
case class Embedding(values: Vector[Float]):
  def cosineSimilarity(other: Embedding): Float =
    val dot = (values zip other.values).map { case (a, b) => a * b }.sum
    val normA = math.sqrt(values.map(x => x * x).sum)
    val normB = math.sqrt(other.values.map(x => x * x).sum)
    (dot / (normA * normB)).toFloat

// Vector database
trait VectorDB:
  def insert(id: String, embedding: Embedding, metadata: Json): Task[Unit]
  def search(query: Embedding, topK: Int): Task[List[SearchResult]]

case class SearchResult(
  id: String,
  similarity: Float,
  metadata: Json
)

// RAG Pipeline
def ragQuery(question: String)(using
  embedModel: EmbeddingModel,
  vectorDB: VectorDB,
  llm: LLM
): Task[String] = for
  // 1. Embed query
  queryEmbedding <- embedModel.embed(question)
  
  // 2. Vector search
  docs <- vectorDB.search(queryEmbedding, topK = 5)
  
  // 3. Construct context
  context = docs.map(_.metadata.as[Document].content).mkString("\n\n")
  
  // 4. Augmented prompt
  prompt = s"""
    Context:
    $context
    
    Question: $question
    
    Answer based only on the context above:
  """
  
  // 5. LLM completion
  answer <- llm.complete(prompt)
yield answer
```

**Chunking Strategies:**
```scala
// 1. Fixed-size chunks
def fixedSizeChunking(text: String, chunkSize: Int): List[String] =
  text.sliding(chunkSize, chunkSize).toList

// 2. Sentence-based chunks
def sentenceChunking(text: String, maxSentences: Int): List[String] =
  val sentences = text.split("[.!?]").toList
  sentences.grouped(maxSentences).map(_.mkString(". ")).toList

// 3. Semantic chunks (with overlap)
def semanticChunking(
  text: String,
  chunkSize: Int,
  overlap: Int
): List[String] =
  text.sliding(chunkSize, chunkSize - overlap).toList
```

#### Model Serving Architecture

**Inference Patterns:**

**1. Synchronous (Request-Response):**
```scala
trait ModelServer:
  def predict(input: Features): IO[Prediction]

// HTTP endpoint
val routes = HttpRoutes.of[IO] {
  case req @ POST -> Root / "predict" =>
    for
      features <- req.as[Features]
      prediction <- modelServer.predict(features)
      response <- Ok(prediction.asJson)
    yield response
}
```

**2. Asynchronous (Queue-based):**
```scala
// Producer
def submitRequest(req: PredictionRequest): Task[RequestId] = for
  id <- ZIO.succeed(UUID.randomUUID())
  _ <- queue.offer((id, req))
yield id

// Consumer (batch processing)
val batchProcessor = queue.takeBetween(1, 32).flatMap { batch =>
  for
    inputs <- ZIO.succeed(batch.map(_._2.features))
    predictions <- model.predictBatch(inputs)  // GPU-optimized
    _ <- ZIO.foreachDiscard(batch.zip(predictions)) {
      case ((id, _), pred) => resultStore.put(id, pred)
    }
  yield ()
}.forever

// Result retrieval
def getResult(id: RequestId): Task[Option[Prediction]] =
  resultStore.get(id)
```

**Model Versioning:**
```scala
case class ModelMetadata(
  version: String,
  trainingDate: Instant,
  accuracy: Double,
  features: List[FeatureSpec]
)

trait ModelRegistry:
  def register(model: Model, metadata: ModelMetadata): Task[Unit]
  def getModel(version: String): Task[Option[Model]]
  def getLatest: Task[Model]
  def listVersions: Task[List[String]]

// A/B Testing
class ABTestingRouter(
  modelA: Model,
  modelB: Model,
  splitRatio: Double  // 0.0 to 1.0
):
  def predict(input: Features): Task[Prediction] =
    Random.nextDouble.flatMap { rand =>
      if rand < splitRatio then
        modelA.predict(input).tap(logResult("A", _))
      else
        modelB.predict(input).tap(logResult("B", _))
    }
```

**Monitoring & Observability:**
```scala
case class PredictionMetrics(
  latency: Duration,
  inputSize: Int,
  confidence: Double,
  modelVersion: String
)

trait ModelMonitoring:
  def recordPrediction(metrics: PredictionMetrics): UIO[Unit]
  def detectDrift(baseline: Distribution, current: Distribution): Task[Boolean]

// Drift detection (Kolmogorov-Smirnov test)
def ksTest(
  baseline: List[Double],
  current: List[Double],
  alpha: Double = 0.05
): Boolean =
  // Compute empirical CDFs
  val baselineCDF = computeCDF(baseline)
  val currentCDF = computeCDF(current)
  
  // KS statistic
  val ks = (baselineCDF zip currentCDF)
    .map { case (b, c) => math.abs(b - c) }
    .max
  
  // Critical value
  val n = baseline.size
  val m = current.size
  val criticalValue = math.sqrt(-0.5 * math.log(alpha / 2) * (n + m) / (n * m))
  
  ks > criticalValue  // Drift detected
```

---

## 🤖 Неделя 6: AI/ML Integration (Новое в 2025!)

### День 1-3: AI-Ready Scala

**🔥 HOT TREND 2025**: Scala активно интегрируется с AI/ML workflows

#### LLM Integration
```scala
// OpenAI API с http4s
import org.http4s.client._
import org.http4s.circe._
import io.circe.generic.auto._

case class ChatRequest(
  model: String,
  messages: List[Message],
  temperature: Double = 0.7
)

case class Message(role: String, content: String)

def callGPT(prompt: String): IO[String] = 
  Client[IO].use { client =>
    val request = ChatRequest(
      model = "gpt-4",
      messages = List(Message("user", prompt))
    )
    
    client
      .expect[ChatResponse](
        Request[IO](Method.POST, uri"https://api.openai.com/v1/chat/completions")
          .withEntity(request.asJson)
          .putHeaders(Header.Raw(ci"Authorization", s"Bearer $apiKey"))
      )
      .map(_.choices.head.message.content)
  }
```

#### Vector Search с Scala
```scala
// Pinecone integration для RAG
import sttp.client3._

case class Vector(
  id: String,
  values: List[Float],
  metadata: Map[String, String]
)

def upsertVectors(vectors: List[Vector]): Task[Unit] = ???

def queryVectors(
  vector: List[Float], 
  topK: Int = 10
): Task[List[SearchResult]] = ???

// RAG pipeline
def ragQuery(question: String): IO[String] = for
  // 1. Generate embedding для вопроса
  embedding <- generateEmbedding(question)
  
  // 2. Vector search для relevant docs
  docs <- queryVectors(embedding, topK = 5)
  
  // 3. Compose prompt с context
  context = docs.map(_.content).mkString("\n\n")
  prompt = s"Context:\n$context\n\nQuestion: $question"
  
  // 4. Call LLM
  answer <- callGPT(prompt)
yield answer
```

#### Spark ML с Scala
```scala
import org.apache.spark.ml.feature._
import org.apache.spark.ml.classification._

// ML Pipeline
val tokenizer = new Tokenizer()
  .setInputCol("text")
  .setOutputCol("words")

val hashingTF = new HashingTF()
  .setInputCol("words")
  .setOutputCol("features")

val lr = new LogisticRegression()
  .setMaxIter(10)

val pipeline = new Pipeline()
  .setStages(Array(tokenizer, hashingTF, lr))

val model = pipeline.fit(trainingData)
```

**Практика:**
```scala
// Задача 1: Chatbot с memory
// Реализовать conversational AI с context window

// Задача 2: Document QA system
// RAG pipeline: PDF parsing → chunking → embeddings → search

// Задача 3: Streaming ML predictions
// Real-time inference с Kafka + Spark Streaming

// Задача 4: A/B testing framework
// Statistical significance testing для ML models
```

### День 4-7: Production ML Systems

#### Model Serving
```scala
// HTTP endpoint для model inference
import org.http4s.dsl.io._

case class PredictionRequest(features: Map[String, Double])
case class PredictionResponse(prediction: Double, confidence: Double)

val mlRoutes = HttpRoutes.of[IO] {
  case req @ POST -> Root / "predict" =>
    for
      request <- req.as[PredictionRequest]
      prediction <- model.predict(request.features)
      response = PredictionResponse(prediction, confidence = 0.95)
    yield Ok(response.asJson)
}

// Batch processing
def batchPredict(
  input: Stream[IO, Features]
): Stream[IO, Prediction] =
  input
    .chunkN(100)  // batch для efficiency
    .evalMap(chunk => model.predictBatch(chunk))
    .flatMap(Stream.emits)
```

#### Monitoring ML Models
```scala
// Prometheus metrics для ML
import io.prometheus.client._

val predictionLatency = Histogram.build()
  .name("ml_prediction_latency_seconds")
  .help("ML prediction latency")
  .register()

val predictionCounter = Counter.build()
  .name("ml_predictions_total")
  .labelNames("model_version", "outcome")
  .help("Total predictions")
  .register()

// Drift detection
def detectDrift(
  baseline: Distribution,
  current: Distribution
): IO[Boolean] = ???
```

**Проекты:**
```scala
// Проект 1: Recommendation Engine
// Collaborative filtering с Spark ALS

// Проект 2: Fraud Detection
// Real-time scoring с streaming features

// Проект 3: LLM-powered search
// Semantic search с embeddings + reranking

// Проект 4: AutoML pipeline
// Hyperparameter tuning с Bayesian optimization
```

---

## 🎯 Неделя 7-8: Mock Interviews и Coding Practice

### День 1-3: Coding Challenges

**LeetCode Style (Functional Scala):**

**Easy:**
```scala
// 1. Two Sum - functional solution
def twoSum(nums: Array[Int], target: Int): Option[(Int, Int)] =
  nums.zipWithIndex
    .combinations(2)
    .collectFirst {
      case Array((a, i), (b, j)) if a + b == target => (i, j)
    }

// 2. Valid Parentheses
def isValid(s: String): Boolean =
  s.foldLeft(List.empty[Char]) {
    case (stack, c) if "([{".contains(c) => c :: stack
    case (h :: t, ')') if h == '(' => t
    case (h :: t, ']') if h == '[' => t
    case (h :: t, '}') if h == '{' => t
    case _ => return false
  }.isEmpty
```

**Medium:**
```scala
// 1. LRU Cache - immutable
class LRUCache[K, V](capacity: Int):
  private type Cache = (Map[K, V], List[K])
  
  def get(key: K, cache: Cache): (Option[V], Cache) = ???
  def put(key: K, value: V, cache: Cache): Cache = ???

// 2. Course Schedule (топологическая сортировка)
def canFinish(
  numCourses: Int,
  prerequisites: Array[(Int, Int)]
): Boolean =
  def hasCycle(graph: Map[Int, List[Int]]): Boolean = ???
  !hasCycle(buildGraph(prerequisites))

// 3. Word Ladder с BFS
def ladderLength(
  beginWord: String,
  endWord: String,
  wordList: List[String]
): Int =
  @tailrec
  def bfs(
    queue: Queue[(String, Int)],
    visited: Set[String]
  ): Int = ???
  
  bfs(Queue((beginWord, 1)), Set(beginWord))
```

**Hard:**
```scala
// 1. Трудная задача на ДП - Longest Valid Parentheses
def longestValidParentheses(s: String): Int =
  s.indices.foldLeft((0, List(-1))) {
    case ((maxLen, stack), i) =>
      s(i) match
        case '(' => (maxLen, i :: stack)
        case ')' =>
          stack match
            case _ :: Nil => (maxLen, i :: Nil)
            case _ :: tail =>
              val newMax = maxLen.max(i - tail.head)
              (newMax, tail)
  }._1

// 2. Sliding Window Maximum
def maxSlidingWindow(nums: Array[Int], k: Int): Array[Int] =
  nums.sliding(k)
    .map(_.max)
    .toArray
```

**Scala-Specific:**
```scala
// 1. Type-safe Expression DSL
sealed trait Expr[A]
case class Lit[A](value: A) extends Expr[A]
case class Add(l: Expr[Int], r: Expr[Int]) extends Expr[Int]
case class Concat(l: Expr[String], r: Expr[String]) extends Expr[String]
case class If[A](
  cond: Expr[Boolean],
  then: Expr[A],
  else: Expr[A]
) extends Expr[A]

def eval[A](expr: Expr[A]): A = expr match
  case Lit(v) => v
  case Add(l, r) => eval(l) + eval(r)
  case Concat(l, r) => eval(l) + eval(r)
  case If(c, t, e) => if eval(c) then eval(t) else eval(e)

// 2. LazyList для infinite sequences
val fibonacci: LazyList[BigInt] =
  BigInt(0) #:: BigInt(1) #:: fibonacci.zip(fibonacci.tail)
    .map { case (a, b) => a + b }

// 3. Parser Combinators
import scala.util.parsing.combinator._

class JsonParser extends JavaTokenParsers:
  def value: Parser[Any] = obj | arr | string | number
  def obj: Parser[Map[String, Any]] = ???
  def arr: Parser[List[Any]] = ???
```

### День 4-7: System Design Practice

**Примеры вопросов:**

**1. Design Twitter**
- Feed generation (pull vs push)
- Sharding strategies
- Cache invalidation
- Real-time updates

**2. Design Netflix**
- Video encoding pipeline
- CDN strategy
- Recommendation engine
- Adaptive streaming

**3. Design Uber**
- Geo-spatial indexing
- Matching algorithm
- Payment processing
- Real-time location updates

**4. Design WhatsApp**
- Message delivery guarantees
- End-to-end encryption
- Media storage
- Group chat scaling

**Framework для ответа:**
```
1. Requirements Clarification
   - Functional requirements
   - Non-functional requirements
   - Scale estimates

2. High-Level Design
   - Main components
   - Data flow
   - API design

3. Deep Dive
   - Database schema
   - Caching strategy
   - Load balancing
   - Monitoring

4. Trade-offs
   - CAP theorem
   - Consistency vs availability
   - Cost vs performance
```

### Behavioral Questions (STAR method)

**Подготовьте примеры:**

1. **Technical Challenge**
   - Situation: Migration 100+ microservices to Scala 3
   - Task: Zero downtime, maintain performance
   - Action: Phased rollout, feature flags, A/B testing
   - Result: Successfully migrated, 20% performance improvement

2. **Leadership**
   - Situation: Team struggling with ZIO adoption
   - Task: Improve team productivity
   - Action: Tech talks, pair programming, documentation
   - Result: 80% test coverage, reduced bugs by 40%

3. **Conflict Resolution**
   - Situation: Disagreement about effect system choice
   - Task: Make data-driven decision
   - Action: PoC with both, benchmark, team vote
   - Result: Consensus reached, smooth adoption

4. **Production Incident**
   - Situation: Memory leak in production
   - Task: Identify and fix quickly
   - Action: Heap dump analysis, profiling, hotfix
   - Result: Fixed in 2 hours, postmortem, monitoring improvements

---

## 📚 Обновлённые ресурсы 2025

### Книги (Must-read):
1. **"Scala 3 Programming"** - Martin Odersky et al. (2024)
2. **"Functional Programming in Scala"** (Red Book) - обновлённое издание
3. **"ZIO in Action"** - John De Goes & Adam Fraser (2024)
4. **"Effect Systems in Practice"** (2025)
5. **"Scala with Cats"** - Noel Welsh (обновлено для Scala 3)

### Online Courses:
- **Scala Center** - Scala 3 Migration Guide
- **Rock the JVM** - ZIO 2.x and Cats Effect 3.x courses
- **Udemy** - "Advanced Scala 3" by Daniel Ciocîrlan
- **YouTube** - ScalaDays 2025 talks

### Practice Platforms:
- **Exercism** - Scala 3 track
- **LeetCode** - Scala solutions
- **Codewars** - Functional programming katas
- **Scala Exercises** - обновлено для Scala 3

### Communities:
- **Discord**: Scala, ZIO, Typelevel
- **Reddit**: r/scala
- **Slack**: Apache Pekko, Scala community
- **Twitter/X**: #ScalaLang, #FunctionalProgramming

---

## ✅ Финальный Senior-Level Checklist

### Must-Have (Обязательно):
- ✅ Scala 3 syntax и новые features (given/using, enums, union types)
- ✅ Effect Systems - хотя бы один из (ZIO или Cats Effect) на продакшн уровне
- ✅ Apache Pekko или альтернативы (ZIO Actors, FS2)
- ✅ Functional database access (Doobie/Slick/Quill)
- ✅ Kafka integration (fs2-kafka или zio-kafka)
- ✅ Testing (property-based, unit, integration)
- ✅ Production debugging и profiling

### Nice-to-Have (Желательно):
- ✅ Оба effect systems (ZIO И Cats Effect)
- ✅ gRPC с ScalaPB
- ✅ Kubernetes/Docker deployment
- ✅ Observability (Prometheus, Grafana, OpenTelemetry)
- ✅ CI/CD для Scala проектов
- ✅ Performance tuning (JVM, GC)

### Senior-Level (Экспертиза):
- ✅ System design для distributed systems
- ✅ Архитектурные решения и trade-offs
- ✅ Migration strategies (Scala 2 → 3, Akka → Pekko)
- ✅ Code review и mentoring
- ✅ Production incident response
- ✅ Technical leadership
- ✅ **AI/ML integration** (новое в 2025!)

### 2025-Specific Skills:
- ✅ Scala 3 metaprogramming (макросы, inline, match types)
- ✅ JDK 17+ features knowledge
- ✅ AI/LLM integration patterns
- ✅ Modern observability (OpenTelemetry)
- ✅ Cloud-native patterns (service mesh, serverless)

---

## 🎤 Актуальные вопросы для 2025

### Scala 3
- Чем given/using лучше implicit? Migration path?
- Opaque types vs value classes - когда что?
- Better-fors (SIP-62) - что изменилось?
- Match types для type-level computation?

### Effect Systems
- ZIO vs Cats Effect - архитектурные отличия?
- Как работают fibers? Отличие от threads?
- STM в production - use cases?
- Tagless Final - преимущества и недостатки?

### Pekko/Reactive
- Почему Akka → Pekko? Licensing implications?
- Typed Actors vs Classic - migration strategy?
- Backpressure в streams - как реализовано?
- Alternatives to Actor model?

### Architecture
- Microservices vs modular monolith - trade-offs?
- Event Sourcing - когда применять?
- CQRS implementation challenges?
- Service mesh - когда необходим?

### AI/ML (Новое!)
- Scala в ML workflows - преимущества?
- RAG pipeline architecture?
- Model serving best practices?
- LLM integration patterns?

---

## 💡 Советы для Senior-level интервью

### Technical Interview:
1. **Объясняйте trade-offs** - нет идеального решения
2. **Производственный опыт** - ссылайтесь на реальные кейсы
3. **Масштабируемость** - всегда думайте о scale
4. **Мониторинг** - как вы будете отслеживать проблемы?
5. **Тестирование** - стратегия тестирования с самого начала

### System Design:
1. **Clarify first** - уточните требования
2. **High-level → Details** - сверху вниз
3. **Numbers matter** - back-of-envelope calculations
4. **Bottlenecks** - identify и address
5. **Evolution** - как система будет развиваться?

### Behavioral:
1. **STAR method** - структурированные ответы
2. **Leadership** - примеры влияния на команду
3. **Failures** - что вы узнали из ошибок?
4. **Mentoring** - как вы растите других?
5. **Initiative** - примеры проактивности

---

## 🎯 Финальная подготовка (последняя неделя)

### За неделю:
- ✅ Повторить ключевые концепции Scala 3
- ✅ Решить 20+ задач на LeetCode (разная сложность)
- ✅ Пройти 3-5 mock system design интервью
- ✅ Подготовить STAR stories (5-7 примеров)
- ✅ Обновить GitHub profile

### За день:
- ✅ 8+ часов сна
- ✅ Проверить оборудование (камера, микрофон, интернет)
- ✅ Подготовить среду (тихое место, вода, бумага)
- ✅ Просмотреть core concepts (без deep dive)
- ✅ Подготовить вопросы интервьюеру (5-10 вопросов)

### В день интервью:
- ✅ Прийти за 10 минут
- ✅ Думать вслух - показывайте процесс мышления
- ✅ Уточняйте требования - не спешите
- ✅ Тестируйте решение - edge cases
- ✅ Будьте честны - "не знаю" лучше чем неправда

---

## 🚀 Ключевые изменения 2025 vs 2024

### Что НОВОЕ:
1. **Scala 3** стал mainstream (65%+ adoption)
2. **Apache Pekko** заменил Akka в большинстве проектов
3. **AI/ML integration** стал must-have skill
4. **ZIO 2.x** vs **Cats Effect 3.x** - оба популярны
5. **JDK 17+** минимальное требование для Scala 3.8+
6. **Better-fors** (SIP-62) в preview
7. **Named tuples** (Scala 3.6+)
8. **Effect systems** вытесняют actor model

### Что DEPRECATED:
1. **Akka** (из-за BSL лицензии)
2. **Scala 2** для новых проектов (70% мигрировали)
3. **Implicit conversions** (используйте given/using)
4. **Classic Actors** (переход на Typed)
5. **Symbolic operators** (читаемость важнее)

### Растущие тренды:
1. **Serverless Scala** (AWS Lambda, GraalVM Native Image)
2. **Scala Native** для systems programming
3. **Scala.js** для full-stack applications
4. **Cloud-native patterns**
5. **Observability-first design**

---

## 📞 Финальные советы

1. **Практика > Теория** - больше кодите, меньше читайте
2. **Build projects** - GitHub portfolio важен
3. **Community** - участвуйте в Discord/Slack
4. **Mock interviews** - тренируйтесь с другими
5. **Stay current** - читайте Scala blog, ScalaTimes
6. **Open source** - contribute для видимости
7. **Blog/Speak** - share your knowledge

**Удачи на собеседовании! 🚀**

---

---

## 📖 Академические и теоретические ресурсы

### Теория категорий

**Книги:**
1. **"Category Theory for Programmers"** - Bartosz Milewski
   - Бесплатно онлайн
   - Примеры на Haskell и C++
   - Адаптируйте для Scala

2. **"Category Theory in Context"** - Emily Riehl
   - Математическая строгость
   - Для глубокого понимания

3. **"Conceptual Mathematics"** - Lawvere & Schanuel
   - Для начинающих
   - Интуитивный подход

**Статьи:**
- "Notions of Computation and Monads" - Moggi (основополагающая работа)
- "Applicative Programming with Effects" - McBride & Paterson

**Видео:**
- Bartosz Milewski Category Theory series (YouTube)
- "Category Theory for Programmers" курс

### Функциональное программирование

**Книги:**
1. **"Functional Programming in Scala"** (Red Book)
   - Глава 10-12: монады, аппликативы
   - Упражнения с решениями

2. **"Functional and Reactive Domain Modeling"** - Debasish Ghosh
   - DDD + FP
   - Практические паттерны

3. **"Purely Functional Data Structures"** - Chris Okasaki
   - Функциональные структуры данных
   - Анализ производительности

**Papers:**
- "Why Functional Programming Matters" - John Hughes
- "Theorems for Free!" - Philip Wadler
- "Parametricity" - Reynolds

### Type Theory

**Книги:**
1. **"Types and Programming Languages"** - Pierce
   - Система типов Хиндли-Милнера
   - Lambda calculus
   - Type inference

2. **"Advanced Topics in Types and Programming Languages"** - Pierce
   - Dependent types
   - Linear types
   - Effect systems

**Online:**
- Software Foundations (Coq)
- Type Theory курс (Oregon Programming Languages Summer School)

### Распределённые системы

**Книги:**
1. **"Designing Data-Intensive Applications"** - Martin Kleppmann
   - MUST READ для Senior!
   - Репликация, партиционирование, транзакции
   - Consistency models

2. **"Distributed Systems"** - Maarten van Steen & Andrew Tanenbaum
   - Академический учебник
   - Формальные доказательства

3. **"Database Internals"** - Alex Petrov
   - LSM-trees, B-trees
   - MVCC, WAL
   - Consensus algorithms

**Papers (классика):**
- "Time, Clocks, and the Ordering of Events" - Lamport
- "The Byzantine Generals Problem" - Lamport
- "Paxos Made Simple" - Lamport
- "In Search of an Understandable Consensus Algorithm" (Raft) - Ongaro & Ousterhout
- "Dynamo: Amazon's Highly Available Key-value Store"
- "Bigtable: A Distributed Storage System"
- "MapReduce: Simplified Data Processing"

### Concurrency Theory

**Книги:**
1. **"Java Concurrency in Practice"** - Goetz
   - Хотя про Java, концепции universal
   - Memory model
   - Thread safety

2. **"The Art of Multiprocessor Programming"** - Herlihy & Shavit
   - Lock-free алгоритмы
   - Consensus numbers
   - Linearizability

**Papers:**
- "Communicating Sequential Processes" - Hoare (основа Actor Model)
- "Software Transactional Memory" - Shavit & Touitou

### Machine Learning

**Книги:**
1. **"Hands-On Machine Learning with Scikit-Learn & TensorFlow"** - Géron
   - Практический подход
   - Примеры кода

2. **"Deep Learning"** - Goodfellow, Bengio, Courville
   - Теоретические основы
   - Математика behind deep learning

3. **"Attention Is All You Need"** (Paper)
   - Transformer architecture
   - Основа современных LLMs

**Online:**
- fast.ai курсы (практический подход)
- DeepLearning.AI (Andrew Ng)
- Stanford CS224N (NLP with Deep Learning)

---

## 🧪 Упражнения для закрепления теории

### Теория категорий

**Упражнение 1**: Докажите законы функтора для List
```scala
// Реализуйте и проверьте property-based tests
def functorLaws[A, B, C]: Prop = ???
```

**Упражнение 2**: Реализуйте Applicative для Option и докажите законы
```scala
given Applicative[Option] = ???
// Проверьте все 4 закона аппликатива
```

**Упражнение 3**: Kleisli композиция
```scala
// Реализуйте композицию для Option
def composeKleisli[A, B, C](
  f: A => Option[B],
  g: B => Option[C]
): A => Option[C] = ???

// Докажите ассоциативность
```

### Effect Systems

**Упражнение 1**: Реализуйте простой IO Monad
```scala
sealed trait IO[A]:
  def flatMap[B](f: A => IO[B]): IO[B]
  def map[B](f: A => B): IO[B]
  
object IO:
  def pure[A](a: A): IO[A] = ???
  def suspend[A](thunk: => A): IO[A] = ???
  
  // Добавьте trampolining для stack safety
```

**Упражнение 2**: Fiber scheduling
```scala
// Реализуйте простой cooperative scheduler
class FiberScheduler:
  def fork[A](io: IO[A]): IO[Fiber[A]] = ???
  def yield_: IO[Unit] = ???
```

### Распределённые системы

**Упражнение 1**: Реализуйте Vector Clock
```scala
// С property-based tests для свойств happens-before
```

**Упражнение 2**: G-Counter CRDT
```scala
// Докажите, что merge коммутативен и идемпотентен
```

**Упражнение 3**: Raft consensus (simplified)
```scala
// Реализуйте leader election
// Проверьте safety property: at most one leader per term
```

### Concurrency

**Упражнение 1**: Lock-free stack
```scala
// Используя AtomicReference
class LockFreeStack[A]:
  def push(a: A): Unit = ???
  def pop(): Option[A] = ???
```

**Упражнение 2**: Software Transactional Memory
```scala
// Простая STM с retry
trait STM[A]:
  def atomically[A](transaction: STM[A]): A = ???
```

---

## 📊 Математические основы (для углублённого изучения)

### Lambda Calculus

```
Синтаксис:
  e ::= x              (переменная)
      | λx.e           (абстракция)
      | e₁ e₂          (аппликация)

Beta-reduction:
  (λx.e₁) e₂ → e₁[x := e₂]

Пример:
  (λx.x + 1) 5 → 5 + 1 → 6
  
Church Numerals:
  0 := λf.λx.x
  1 := λf.λx.f x
  2 := λf.λx.f (f x)
  
  succ := λn.λf.λx.f (n f x)
  plus := λm.λn.λf.λx.m f (n f x)
```

### Type Theory Notation

```
Typing Judgments:
  Γ ⊢ e : τ
  (в контексте Γ, выражение e имеет тип τ)

Rules:
  T-Var:    x : τ ∈ Γ
           ———————————
            Γ ⊢ x : τ

  T-Abs:    Γ, x : τ₁ ⊢ e : τ₂
           ————————————————————
           Γ ⊢ λx.e : τ₁ → τ₂

  T-App:    Γ ⊢ e₁ : τ₁ → τ₂    Γ ⊢ e₂ : τ₁
           ————————————————————————————————
                 Γ ⊢ e₁ e₂ : τ₂
```

### Set Theory для типов

```
Типы как множества:
  Int = {..., -1, 0, 1, 2, ...}
  Bool = {true, false}
  
Product types:
  (A, B) = A × B = {(a, b) | a ∈ A, b ∈ B}
  
Sum types:
  Either[A, B] = A + B = {Left(a) | a ∈ A} ∪ {Right(b) | b ∈ B}
  
Function types:
  A => B = B^A = {f | f: A → B}
  
Cardinality:
  |A × B| = |A| × |B|
  |A + B| = |A| + |B|
  |A => B| = |B|^|A|
```

### Information Theory для ML

```
Entropy:
  H(X) = -Σ p(x) log₂ p(x)
  (мера неопределённости)

Cross-Entropy Loss:
  L = -Σ y_true * log(y_pred)
  (используется в классификации)

KL Divergence:
  D_KL(P || Q) = Σ P(x) log(P(x) / Q(x))
  (мера различия между распределениями)
```

---

## 🔬 Research Papers для Senior+ уровня

### Must-Read Papers

**Functional Programming:**
1. "Monads for Functional Programming" - Wadler
2. "Linear Types Can Change the World" - Wadler
3. "Propositions as Types" - Wadler

**Effect Systems:**
1. "Algebraic Effects and Handlers" - Plotkin & Pretnar
2. "Freer Monads, More Extensible Effects" - Kiselyov & Ishii
3. "ZIO: A ZIO-Based Effect System" - De Goes

**Distributed Systems:**
1. "Harvest, Yield, and Scalable Tolerant Systems" - Fox & Brewer
2. "Eventually Consistent" - Vogels (Amazon)
3. "Conflict-free Replicated Data Types" - Shapiro et al.

**ML Systems:**
1. "Hidden Technical Debt in Machine Learning Systems"
2. "TensorFlow: A System for Large-Scale Machine Learning"
3. "Ray: A Distributed Framework for Emerging AI Applications"

### Где искать papers

- **ArXiv.org** - препринты (cs.PL, cs.DC)
- **POPL** - Programming Languages
- **ICFP** - Functional Programming
- **OSDI/SOSP** - Operating Systems & Distributed Systems
- **VLDB/SIGMOD** - Databases
- **NeurIPS/ICML** - Machine Learning

---

*План подготовлен с учётом актуальных трендов Scala ecosystem на ноябрь 2025 года*
*Теоретические материалы основаны на академических источниках и production best practices*
