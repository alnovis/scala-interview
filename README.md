# План подготовки к собеседованию Senior Scala Developer

## 📋 Общая структура (4-6 недель)

**Неделя 1-2**: Основы Scala + Функциональное программирование  
**Неделя 3-4**: Продвинутые темы + Экосистема  
**Неделя 5-6**: Системный дизайн + Mock интервью

---

## 🎯 Неделя 1: Основы Scala

### День 1-2: Базовый синтаксис и концепции

**Темы для повторения:**

- Collections (List, Map, Set, Vector, Array)
- Immutability vs Mutability
- Case classes vs Classes
- Pattern matching (exhaustiveness, guards, extractors)
- For-comprehensions
- Implicit conversions и implicit parameters
- Type inference и type annotations

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

