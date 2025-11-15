# Полный план подготовки по ZIO

## Оглавление
1. [Введение в ZIO](#введение-в-zio)
2. [Основы ZIO](#основы-zio)
3. [Типы эффектов](#типы-эффектов)
4. [Обработка ошибок](#обработка-ошибок)
5. [Конкурентность и параллелизм](#конкурентность-и-параллелизм)
6. [Управление ресурсами](#управление-ресурсов)
7. [ZIO Слои (ZLayer)](#zio-слои-zlayer)
8. [ZIO Streams](#zio-streams)
9. [Тестирование](#тестирование)
10. [Продвинутые темы](#продвинутые-темы)
11. [Практические проекты](#практические-проекты)

---

## Введение в ZIO

### Что такое ZIO?

ZIO — это функциональная библиотека эффектов для Scala, которая решает следующие задачи:
- Управление побочными эффектами (side effects)
- Асинхронное программирование
- Конкурентность и параллелизм
- Обработка ошибок
- Управление ресурсами
- Dependency injection

### Почему ZIO?

**Преимущества:**
- Типобезопасность
- Композиционность
- Тестируемость
- Производительность
- Богатая экосистема

**Сравнение с другими подходами:**
```scala
// Императивный стиль
def getUserName(id: Int): String = {
  val user = database.getUser(id) // может бросить исключение
  user.name
}

// Future (проблемы: eager execution, ограниченная обработка ошибок)
def getUserName(id: Int): Future[String] = {
  database.getUser(id).map(_.name)
}

// ZIO (lazy, типобезопасная обработка ошибок)
def getUserName(id: Int): ZIO[Database, DatabaseError, String] = {
  ZIO.serviceWithZIO[Database](_.getUser(id)).map(_.name)
}
```

### Установка и настройка

**build.sbt:**
```scala
libraryDependencies ++= Seq(
  "dev.zio" %% "zio" % "2.0.19",
  "dev.zio" %% "zio-streams" % "2.0.19",
  "dev.zio" %% "zio-test" % "2.0.19" % Test,
  "dev.zio" %% "zio-test-sbt" % "2.0.19" % Test
)

testFrameworks += new TestFramework("zio.test.sbt.ZTestFramework")
```

---

## Основы ZIO

### Типы ZIO

**Основной тип:**
```scala
ZIO[-R, +E, +A]
```

- **R** — тип окружения (Environment), требуемый для выполнения
- **E** — тип ошибки (Error)
- **A** — тип успешного результата

**Алиасы:**
```scala
type IO[+E, +A] = ZIO[Any, E, A]        // не требует окружения
type Task[+A] = ZIO[Any, Throwable, A]  // может бросить Throwable
type RIO[-R, +A] = ZIO[R, Throwable, A] // требует окружение, может бросить Throwable
type UIO[+A] = ZIO[Any, Nothing, A]     // не может упасть
type URIO[-R, +A] = ZIO[R, Nothing, A]  // требует окружение, не может упасть
```

### Создание ZIO эффектов

**Успешные значения:**
```scala
import zio._

// Чистое значение
val number: UIO[Int] = ZIO.succeed(42)

// Ленивое вычисление
val computation: Task[String] = ZIO.attempt {
  println("Выполняется вычисление")
  "результат"
}

// Из опции
val maybeValue: IO[Option[Nothing], Int] = ZIO.fromOption(Some(42))

// Из Either
val eitherValue: IO[String, Int] = ZIO.fromEither(Right(42))

// Из Future
import scala.concurrent.Future
val futureValue: Task[Int] = ZIO.fromFuture { implicit ec =>
  Future.successful(42)
}
```

**Неудачные значения:**
```scala
// Ошибка
val error: IO[String, Nothing] = ZIO.fail("Произошла ошибка")

// Исключение
val exception: Task[Nothing] = ZIO.die(new RuntimeException("Критическая ошибка"))

// Условный успех/неудача
def divide(a: Int, b: Int): IO[String, Int] = {
  if (b == 0) ZIO.fail("Деление на ноль")
  else ZIO.succeed(a / b)
}
```

### Комбинирование эффектов

**Последовательное выполнение:**
```scala
val program: Task[Unit] = for {
  _ <- Console.printLine("Введите имя:")
  name <- Console.readLine
  _ <- Console.printLine(s"Привет, $name!")
} yield ()
```

**Параллельное выполнение:**
```scala
val result: Task[(Int, String, Boolean)] = for {
  tuple <- ZIO.collectAllPar(
    ZIO.succeed(42),
    ZIO.succeed("hello"),
    ZIO.succeed(true)
  )
} yield tuple

// Или используя zipPar
val parallel: Task[(Int, String)] = 
  ZIO.succeed(42).zipPar(ZIO.succeed("hello"))
```

**Трансформация результатов:**
```scala
val original: UIO[Int] = ZIO.succeed(42)

// map - трансформация значения
val doubled: UIO[Int] = original.map(_ * 2)

// flatMap - цепочка эффектов
val computed: Task[String] = original.flatMap { n =>
  ZIO.attempt(n.toString)
}

// as - замена результата
val replaced: UIO[String] = original.as("новое значение")
```

---

## Типы эффектов

### Console

```scala
import zio._

object ConsoleExample extends ZIOAppDefault {
  val program: Task[Unit] = for {
    _ <- Console.printLine("Как вас зовут?")
    name <- Console.readLine
    _ <- Console.printLine(s"Здравствуйте, $name!")
    _ <- Console.printLineError("Это сообщение об ошибке")
  } yield ()

  def run = program
}
```

### Random

```scala
import zio._

val randomProgram: UIO[Unit] = for {
  number <- Random.nextInt
  _ <- Console.printLine(s"Случайное число: $number").orDie
  
  // Число в диапазоне
  dice <- Random.nextIntBounded(6).map(_ + 1)
  _ <- Console.printLine(s"Бросок кубика: $dice").orDie
  
  // Случайный boolean
  coin <- Random.nextBoolean
  _ <- Console.printLine(s"Монетка: ${if (coin) "Орел" else "Решка"}").orDie
} yield ()
```

### Clock

```scala
import zio._
import java.time.Instant

val clockProgram: UIO[Unit] = for {
  now <- Clock.instant
  _ <- Console.printLine(s"Текущее время: $now").orDie
  
  // Измерение времени выполнения
  duration <- ZIO.succeed(Thread.sleep(1000)).timed.map(_._1)
  _ <- Console.printLine(s"Заняло: ${duration.toMillis}мс").orDie
} yield ()
```

---

## Обработка ошибок

### Типы ошибок

```scala
// Доменные ошибки (восстанавливаемые)
sealed trait UserError
case class UserNotFound(id: Int) extends UserError
case class InvalidEmail(email: String) extends UserError

// Дефекты (невосстанавливаемые)
// Используются для программных ошибок, которые не должны происходить
```

### Обработка ошибок

**catchAll — обработать все ошибки:**
```scala
def getUser(id: Int): IO[UserError, User] = ???

val handled: UIO[User] = getUser(42).catchAll {
  case UserNotFound(id) => 
    ZIO.succeed(User.default)
  case InvalidEmail(email) => 
    ZIO.succeed(User.default)
}
```

**catchSome — обработать некоторые ошибки:**
```scala
val partiallyHandled: IO[UserError, User] = getUser(42).catchSome {
  case UserNotFound(id) => ZIO.succeed(User.default)
  // InvalidEmail остается необработанным
}
```

**orElse — альтернативный эффект:**
```scala
val withFallback: IO[UserError, User] = 
  getUser(42).orElse(getUser(1))
```

**fold — обработать и успех, и ошибку:**
```scala
val folded: UIO[String] = getUser(42).fold(
  error => s"Ошибка: $error",
  user => s"Пользователь: ${user.name}"
)
```

**either — конвертация в Either:**
```scala
val asEither: UIO[Either[UserError, User]] = getUser(42).either
```

**option — конвертация в Option:**
```scala
val asOption: UIO[Option[User]] = getUser(42).option
```

### Повторные попытки

```scala
import zio._

// Простой retry
val retried: IO[UserError, User] = 
  getUser(42).retry(Schedule.recurs(3))

// Retry с экспоненциальной задержкой
val exponentialRetry: IO[UserError, User] = 
  getUser(42).retry(
    Schedule.exponential(1.second) && Schedule.recurs(5)
  )

// Retry с условием
val conditionalRetry: IO[UserError, User] = 
  getUser(42).retryWhile {
    case UserNotFound(_) => true
    case _ => false
  }
```

### Timeout и Racing

```scala
// Timeout
val withTimeout: IO[UserError, Option[User]] = 
  getUser(42).timeout(5.seconds)

// Прервать при timeout
val timeoutFail: IO[UserError, User] = 
  getUser(42).timeoutFail(new TimeoutException)(5.seconds)

// Racing двух эффектов
val raced: Task[Either[String, Int]] = 
  ZIO.succeed("left").delay(2.seconds).either
    .race(ZIO.succeed(42).delay(1.second).either)
```

---

## Конкурентность и параллелизм

### Fiber (файберы)

Fiber — это легковесный поток выполнения в ZIO.

```scala
import zio._

// Запуск в фоне
val fiber1: UIO[Fiber[Nothing, Int]] = ZIO.succeed(42).fork

// Ожидание результата
val program: UIO[Int] = for {
  fiber <- ZIO.succeed(42).delay(1.second).fork
  result <- fiber.join
} yield result

// Прерывание fiber
val interrupted: UIO[Exit[Nothing, Int]] = for {
  fiber <- ZIO.succeed(42).delay(10.seconds).fork
  _ <- fiber.interrupt
  exit <- fiber.await
} yield exit
```

### Параллельное выполнение

```scala
// foreachPar - параллельная обработка коллекции
val numbers = List(1, 2, 3, 4, 5)
val parallelProcessing: Task[List[Int]] = 
  ZIO.foreachPar(numbers) { n =>
    ZIO.succeed(n * 2).delay(1.second)
  }

// collectAllPar - запуск нескольких эффектов параллельно
val parallel: Task[List[Int]] = ZIO.collectAllPar(List(
  ZIO.succeed(1).delay(1.second),
  ZIO.succeed(2).delay(1.second),
  ZIO.succeed(3).delay(1.second)
))

// Ограничение параллелизма
val withLimit: Task[List[Int]] = 
  ZIO.foreachPar(numbers) { n =>
    ZIO.succeed(n * 2)
  }.withParallelism(2) // максимум 2 одновременно
```

### Ref — изменяемое состояние

```scala
import zio._

// Создание Ref
val program: UIO[Int] = for {
  ref <- Ref.make(0)
  _ <- ref.update(_ + 1)
  _ <- ref.update(_ + 1)
  value <- ref.get
} yield value // 2

// Атомарные операции
val atomicOps: UIO[Int] = for {
  ref <- Ref.make(0)
  _ <- ZIO.foreachPar(1 to 100) { _ =>
    ref.update(_ + 1)
  }
  result <- ref.get
} yield result // гарантированно 100

// modify - изменение и возврат значения
val modified: UIO[(String, Int)] = for {
  ref <- Ref.make(0)
  result <- ref.modify { current =>
    (s"Было: $current", current + 1)
  }
} yield result // ("Было: 0", новое значение 1)
```

### Promise

```scala
import zio._

// Promise - однократное присвоение значения
val promiseExample: UIO[String] = for {
  promise <- Promise.make[Nothing, String]
  
  // Fiber который заполнит promise
  _ <- (ZIO.sleep(1.second) *> promise.succeed("готово")).fork
  
  // Ожидание результата
  result <- promise.await
} yield result

// Использование для синхронизации
def waitForSignal: UIO[Unit] = for {
  latch <- Promise.make[Nothing, Unit]
  
  worker = for {
    _ <- ZIO.sleep(2.seconds)
    _ <- Console.printLine("Работа выполнена").orDie
    _ <- latch.succeed(())
  } yield ()
  
  _ <- worker.fork
  _ <- Console.printLine("Ожидание завершения работы...").orDie
  _ <- latch.await
  _ <- Console.printLine("Работа завершена!").orDie
} yield ()
```

### Queue

```scala
import zio._

// Создание очереди
val queueExample: UIO[List[Int]] = for {
  queue <- Queue.bounded[Int](10)
  
  // Producer
  producer = ZIO.foreach(1 to 5) { n =>
    queue.offer(n) *> ZIO.sleep(100.millis)
  }
  
  // Consumer
  consumer = ZIO.foreach(1 to 5) { _ =>
    queue.take
  }
  
  _ <- producer.fork
  results <- consumer
} yield results

// Различные типы очередей
val unboundedQueue: UIO[Queue[Int]] = Queue.unbounded[Int]
val boundedQueue: UIO[Queue[Int]] = Queue.bounded[Int](100)
val slidingQueue: UIO[Queue[Int]] = Queue.sliding[Int](100) // отбрасывает старые
val droppingQueue: UIO[Queue[Int]] = Queue.dropping[Int](100) // отбрасывает новые
```

### Semaphore

```scala
import zio._

// Ограничение конкурентного доступа
def limitedAccess: UIO[List[String]] = for {
  semaphore <- Semaphore.make(permits = 3) // максимум 3 одновременно
  
  results <- ZIO.foreachPar(1 to 10) { n =>
    semaphore.withPermit {
      Console.printLine(s"Обработка $n").orDie *>
      ZIO.sleep(1.second) *>
      ZIO.succeed(s"Результат $n")
    }
  }
} yield results
```

---

## Управление ресурсами

### ZIO.acquireRelease

```scala
import zio._
import java.io.{File, FileInputStream}

// Безопасная работа с файлами
def readFile(path: String): Task[String] = {
  ZIO.acquireReleaseWith(
    acquire = ZIO.attempt(new FileInputStream(path))
  )(
    release = stream => ZIO.succeed(stream.close())
  )(
    use = stream => ZIO.attempt {
      val bytes = new Array[Byte](stream.available())
      stream.read(bytes)
      new String(bytes)
    }
  )
}
```

### Scope

```scala
import zio._

// Создание scoped ресурса
def makeManagedFile(path: String): ZIO[Scope, Throwable, FileInputStream] = {
  ZIO.acquireRelease(
    ZIO.attempt(new FileInputStream(path))
  )(stream => 
    ZIO.succeed(stream.close())
  )
}

// Использование
val program: Task[String] = ZIO.scoped {
  for {
    stream <- makeManagedFile("file.txt")
    content <- ZIO.attempt {
      val bytes = new Array[Byte](stream.available())
      stream.read(bytes)
      new String(bytes)
    }
  } yield content
}
```

### Composing Resources

```scala
import zio._

case class Database(connection: String) {
  def close(): Unit = println(s"Закрытие $connection")
}

case class HttpServer(port: Int) {
  def shutdown(): Unit = println(s"Остановка сервера на порту $port")
}

def makeDatabase: ZIO[Scope, Nothing, Database] = {
  ZIO.acquireRelease(
    ZIO.succeed {
      println("Открытие БД")
      Database("db-connection")
    }
  )(db => 
    ZIO.succeed(db.close())
  )
}

def makeServer: ZIO[Scope, Nothing, HttpServer] = {
  ZIO.acquireRelease(
    ZIO.succeed {
      println("Запуск сервера")
      HttpServer(8080)
    }
  )(server => 
    ZIO.succeed(server.shutdown())
  )
}

// Композиция ресурсов
val application: Task[Unit] = ZIO.scoped {
  for {
    db <- makeDatabase
    server <- makeServer
    _ <- Console.printLine("Приложение запущено")
    _ <- ZIO.never // работает вечно
  } yield ()
}
```

---

## ZIO Слои (ZLayer)

### Что такое ZLayer?

ZLayer предоставляет dependency injection в ZIO.

```scala
import zio._

// Определение сервисов
trait Database {
  def getUser(id: Int): Task[User]
}

trait EmailService {
  def sendEmail(to: String, subject: String, body: String): Task[Unit]
}

trait UserService {
  def notifyUser(id: Int): Task[Unit]
}
```

### Создание слоев

```scala
import zio._

// Реализация Database
case class DatabaseLive() extends Database {
  def getUser(id: Int): Task[User] = 
    ZIO.succeed(User(id, s"user$id@example.com", s"User $id"))
}

object DatabaseLive {
  val layer: ULayer[Database] = ZLayer.succeed(DatabaseLive())
}

// Реализация EmailService
case class EmailServiceLive() extends EmailService {
  def sendEmail(to: String, subject: String, body: String): Task[Unit] = 
    Console.printLine(s"Отправка email: $to - $subject")
}

object EmailServiceLive {
  val layer: ULayer[EmailService] = ZLayer.succeed(EmailServiceLive())
}

// Реализация UserService с зависимостями
case class UserServiceLive(db: Database, email: EmailService) extends UserService {
  def notifyUser(id: Int): Task[Unit] = for {
    user <- db.getUser(id)
    _ <- email.sendEmail(user.email, "Уведомление", s"Привет, ${user.name}!")
  } yield ()
}

object UserServiceLive {
  val layer: ZLayer[Database & EmailService, Nothing, UserService] = 
    ZLayer.fromFunction(UserServiceLive.apply _)
}
```

### Использование слоев

```scala
import zio._

// Программа, требующая UserService
val program: ZIO[UserService, Throwable, Unit] = for {
  userService <- ZIO.service[UserService]
  _ <- userService.notifyUser(42)
} yield ()

// Или используя ZIO.serviceWithZIO
val program2: ZIO[UserService, Throwable, Unit] = 
  ZIO.serviceWithZIO[UserService](_.notifyUser(42))

// Предоставление зависимостей
object MainApp extends ZIOAppDefault {
  // Композиция слоев
  val appLayer: ULayer[UserService] = 
    DatabaseLive.layer ++
    EmailServiceLive.layer >>>
    UserServiceLive.layer
  
  def run = program.provide(appLayer)
}
```

### Комбинирование слоев

```scala
import zio._

// Горизонтальная композиция (++)
val horizontal: ULayer[Database & EmailService] = 
  DatabaseLive.layer ++ EmailServiceLive.layer

// Вертикальная композиция (>>>)
val vertical: ULayer[UserService] = 
  (DatabaseLive.layer ++ EmailServiceLive.layer) >>> UserServiceLive.layer

// Автоматическая композиция
val automatic: ULayer[UserService] = 
  ZLayer.make[UserService](
    DatabaseLive.layer,
    EmailServiceLive.layer,
    UserServiceLive.layer
  )
```

### Слои с конфигурацией

```scala
import zio._
import zio.config._
import zio.config.magnolia._
import zio.config.typesafe._

// Конфигурация
case class DbConfig(url: String, user: String, password: String)
case class AppConfig(db: DbConfig, port: Int)

// Загрузка конфигурации
val configLayer: Layer[ReadError[String], AppConfig] = 
  TypesafeConfigProvider.fromResourcePath().load(descriptor[AppConfig])

// Database с конфигурацией
case class DatabaseLive(config: DbConfig) extends Database {
  def getUser(id: Int): Task[User] = 
    ZIO.succeed(User(id, s"user$id@${config.url}", s"User $id"))
}

object DatabaseLive {
  val layer: ZLayer[DbConfig, Nothing, Database] = 
    ZLayer.fromFunction(DatabaseLive.apply _)
}

// Полная композиция
val fullAppLayer: Layer[ReadError[String], UserService] = 
  ZLayer.make[UserService](
    configLayer,
    DatabaseLive.layer.map(_.db),
    EmailServiceLive.layer,
    UserServiceLive.layer
  )
```

---

## ZIO Streams

### Основы ZStream

```scala
import zio._
import zio.stream._

// Создание стримов
val numbers: Stream[Nothing, Int] = ZStream.fromIterable(1 to 10)
val infinite: Stream[Nothing, Int] = ZStream.iterate(0)(_ + 1)
val fromEffect: Stream[Throwable, String] = ZStream.fromZIO(Console.readLine)

// Трансформация стримов
val doubled: Stream[Nothing, Int] = numbers.map(_ * 2)
val filtered: Stream[Nothing, Int] = numbers.filter(_ % 2 == 0)
val flattened: Stream[Nothing, Int] = numbers.flatMap(n => ZStream(n, n * 2))
```

### Операции со стримами

```scala
import zio._
import zio.stream._

// take - взять первые N элементов
val firstFive: Stream[Nothing, Int] = ZStream.iterate(0)(_ + 1).take(5)

// drop - пропустить первые N элементов
val skipFive: Stream[Nothing, Int] = ZStream.fromIterable(1 to 10).drop(5)

// takeWhile - брать пока условие истинно
val lessThan10: Stream[Nothing, Int] = 
  ZStream.iterate(0)(_ + 1).takeWhile(_ < 10)

// dropWhile - пропускать пока условие истинно
val from5: Stream[Nothing, Int] = 
  ZStream.fromIterable(1 to 10).dropWhile(_ < 5)

// collect - частичное применение функции
val evenDoubled: Stream[Nothing, Int] = 
  ZStream.fromIterable(1 to 10).collect {
    case n if n % 2 == 0 => n * 2
  }
```

### Запуск стримов

```scala
import zio._
import zio.stream._

// runCollect - собрать все элементы
val collected: Task[Chunk[Int]] = 
  ZStream.fromIterable(1 to 5).runCollect

// runFold - свертка
val sum: Task[Int] = 
  ZStream.fromIterable(1 to 10).runFold(0)(_ + _)

// runForEach - выполнить эффект для каждого элемента
val printed: Task[Unit] = 
  ZStream.fromIterable(1 to 5).runForEach { n =>
    Console.printLine(s"Число: $n")
  }

// runDrain - просто выполнить стрим
val drained: Task[Unit] = 
  ZStream.fromIterable(1 to 5).tap(n => Console.printLine(n.toString)).runDrain
```

### Комбинирование стримов

```scala
import zio._
import zio.stream._

// concat - объединение последовательно
val concatenated: Stream[Nothing, Int] = 
  ZStream(1, 2, 3) ++ ZStream(4, 5, 6)

// merge - объединение конкурентно
val merged: Stream[Nothing, Int] = 
  ZStream(1, 2, 3).merge(ZStream(4, 5, 6))

// zip - объединение попарно
val zipped: Stream[Nothing, (Int, String)] = 
  ZStream(1, 2, 3).zip(ZStream("a", "b", "c"))

// zipWith - объединение с функцией
val combined: Stream[Nothing, String] = 
  ZStream(1, 2, 3).zipWith(ZStream("a", "b", "c"))((n, s) => s"$n-$s")
```

### Группировка и батчинг

```scala
import zio._
import zio.stream._

// grouped - группировка по размеру
val grouped: Stream[Nothing, Chunk[Int]] = 
  ZStream.fromIterable(1 to 10).grouped(3)

// groupedWithin - группировка по размеру или времени
val groupedTimed: Stream[Nothing, Chunk[Int]] = 
  ZStream.fromIterable(1 to 100)
    .schedule(Schedule.spaced(100.millis))
    .groupedWithin(10, 1.second)

// mapChunks - работа с чанками
val processedChunks: Stream[Nothing, Int] = 
  ZStream.fromIterable(1 to 10).mapChunks { chunk =>
    chunk.map(_ * 2)
  }
```

### Практический пример: обработка файлов

```scala
import zio._
import zio.stream._
import java.nio.file.Paths

// Чтение файла построчно
def processFile(path: String): Task[Unit] = {
  ZStream
    .fromPath(Paths.get(path))
    .via(ZPipeline.utf8Decode >>> ZPipeline.splitLines)
    .filter(_.nonEmpty)
    .map(_.toUpperCase)
    .runForeach(line => Console.printLine(line))
}

// Запись в файл
def writeToFile(path: String, data: List[String]): Task[Unit] = {
  ZStream
    .fromIterable(data)
    .intersperse("\n")
    .via(ZPipeline.utf8Encode)
    .run(ZSink.fromPath(Paths.get(path)))
}

// HTTP stream обработка
def streamApi: Task[Unit] = {
  val numbers = ZStream.fromIterable(1 to 1000)
  
  numbers
    .mapZIO { n =>
      // Имитация HTTP запроса
      ZIO.attempt(n * 2).delay(100.millis)
    }
    .buffer(10) // буферизация
    .throttleShape(10, 1.second)(_ => 1) // rate limiting
    .runForeach(result => Console.printLine(s"Результат: $result"))
}
```

---

## Тестирование

### ZIO Test

```scala
import zio._
import zio.test._
import zio.test.Assertion._

// Простой тест
object SimpleSpec extends ZIOSpecDefault {
  def spec = suite("SimpleSpec")(
    test("2 + 2 equals 4") {
      assertTrue(2 + 2 == 4)
    },
    
    test("String concatenation") {
      assertTrue("hello" + " " + "world" == "hello world")
    }
  )
}
```

### Assertions

```scala
import zio.test._
import zio.test.Assertion._

object AssertionSpec extends ZIOSpecDefault {
  def spec = suite("AssertionSpec")(
    test("equalTo") {
      assertTrue(42 == 42)
    },
    
    test("contains") {
      assert(List(1, 2, 3))(contains(2))
    },
    
    test("hasSize") {
      assert(List(1, 2, 3))(hasSize(equalTo(3)))
    },
    
    test("forall") {
      assert(List(2, 4, 6))(forall(isEven))
    },
    
    test("exists") {
      assert(List(1, 2, 3))(exists(equalTo(2)))
    }
  )
  
  def isEven = Assertion.assertion[Int]("isEven")(_ % 2 == 0)
}
```

### Тестирование эффектов

```scala
import zio._
import zio.test._

object EffectSpec extends ZIOSpecDefault {
  def divide(a: Int, b: Int): IO[String, Int] = {
    if (b == 0) ZIO.fail("Division by zero")
    else ZIO.succeed(a / b)
  }
  
  def spec = suite("EffectSpec")(
    test("successful division") {
      for {
        result <- divide(10, 2)
      } yield assertTrue(result == 5)
    },
    
    test("division by zero fails") {
      for {
        exit <- divide(10, 0).exit
      } yield assertTrue(exit.isFailure)
    },
    
    test("error message is correct") {
      assertZIO(divide(10, 0).flip)(Assertion.equalTo("Division by zero"))
    }
  )
}
```

### Генераторы и Property-based тестирование

```scala
import zio._
import zio.test._

object PropertySpec extends ZIOSpecDefault {
  def spec = suite("PropertySpec")(
    test("list reverse twice is identity") {
      check(Gen.listOf(Gen.int)) { list =>
        assertTrue(list.reverse.reverse == list)
      }
    },
    
    test("addition is commutative") {
      check(Gen.int, Gen.int) { (a, b) =>
        assertTrue(a + b == b + a)
      }
    },
    
    test("string concatenation length") {
      check(Gen.alphaNumericString, Gen.alphaNumericString) { (s1, s2) =>
        assertTrue((s1 + s2).length == s1.length + s2.length)
      }
    }
  )
}
```

### Mock сервисов

```scala
import zio._
import zio.test._

trait UserRepository {
  def getUser(id: Int): Task[User]
  def saveUser(user: User): Task[Unit]
}

case class User(id: Int, name: String)

// Тестовая реализация
case class TestUserRepository(users: Ref[Map[Int, User]]) extends UserRepository {
  def getUser(id: Int): Task[User] = 
    users.get.flatMap { map =>
      ZIO.fromOption(map.get(id))
        .orElseFail(new Exception(s"User $id not found"))
    }
  
  def saveUser(user: User): Task[Unit] = 
    users.update(_ + (user.id -> user))
}

object TestUserRepository {
  val layer: ULayer[UserRepository] = ZLayer {
    for {
      users <- Ref.make(Map.empty[Int, User])
    } yield TestUserRepository(users)
  }
}

// Использование в тестах
object UserServiceSpec extends ZIOSpecDefault {
  def spec = suite("UserServiceSpec")(
    test("save and retrieve user") {
      for {
        repo <- ZIO.service[UserRepository]
        user = User(1, "Alice")
        _ <- repo.saveUser(user)
        retrieved <- repo.getUser(1)
      } yield assertTrue(retrieved == user)
    }
  ).provide(TestUserRepository.layer)
}
```

### Test aspects

```scala
import zio._
import zio.test._
import zio.test.TestAspect._

object AspectSpec extends ZIOSpecDefault {
  def spec = suite("AspectSpec")(
    test("flaky test with retries") {
      for {
        random <- Random.nextBoolean
        _ <- ZIO.when(random)(ZIO.fail("Random failure"))
      } yield assertCompletes
    } @@ flaky @@ retries(3),
    
    test("timeout test") {
      ZIO.sleep(10.seconds)
    } @@ timeout(1.second) @@ failing,
    
    test("ignored test") {
      assertTrue(1 == 2)
    } @@ ignore,
    
    test("timed test") {
      ZIO.sleep(100.millis)
    } @@ timed
  )
}
```

---

## Продвинутые темы

### ZIO Hub

Hub — это асинхронная структура pub-sub для многих издателей и подписчиков.

```scala
import zio._

val hubExample: UIO[Unit] = for {
  hub <- Hub.bounded[String](10)
  
  // Publisher
  publisher = ZIO.foreach(1 to 5) { n =>
    hub.publish(s"Сообщение $n") *> ZIO.sleep(100.millis)
  }
  
  // Subscriber 1
  subscriber1 = hub.subscribe.flatMap { queue =>
    ZIO.foreach(1 to 5) { _ =>
      queue.take.flatMap(msg => Console.printLine(s"Sub1: $msg").orDie)
    }
  }
  
  // Subscriber 2
  subscriber2 = hub.subscribe.flatMap { queue =>
    ZIO.foreach(1 to 5) { _ =>
      queue.take.flatMap(msg => Console.printLine(s"Sub2: $msg").orDie)
    }
  }
  
  _ <- publisher.fork
  _ <- subscriber1.zipPar(subscriber2)
} yield ()
```

### Interruption и Finalizers

```scala
import zio._

// Безопасное прерывание
val interruptible: UIO[Unit] = {
  val longRunning = (
    Console.printLine("Начало работы").orDie *>
    ZIO.sleep(10.seconds) *>
    Console.printLine("Работа завершена").orDie
  ).ensuring(
    Console.printLine("Cleanup выполнен").orDie
  )
  
  for {
    fiber <- longRunning.fork
    _ <- ZIO.sleep(1.second)
    _ <- fiber.interrupt
  } yield ()
}

// Неприрываемый регион
val uninterruptible: UIO[Unit] = {
  val critical = (
    Console.printLine("Критическая секция начата").orDie *>
    ZIO.sleep(2.seconds) *>
    Console.printLine("Критическая секция завершена").orDie
  ).uninterruptible
  
  for {
    fiber <- critical.fork
    _ <- fiber.interrupt // не прервет критическую секцию
    _ <- fiber.await
  } yield ()
}
```

### Schedule

```scala
import zio._

// Различные расписания
val schedules = {
  // Фиксированный интервал
  val fixed = Schedule.fixed(1.second)
  
  // Экспоненциальная задержка
  val exponential = Schedule.exponential(100.millis, 2.0)
  
  // Линейная задержка
  val linear = Schedule.linear(100.millis)
  
  // Fibonacci
  val fibonacci = Schedule.fibonacci(100.millis)
  
  // Комбинирование
  val combined = Schedule.exponential(100.millis) && Schedule.recurs(5)
  
  // Условное расписание
  val conditional = Schedule.recurWhile[String](_ != "stop")
}

// Использование Schedule
def retryWithSchedule[R, E, A](
  effect: ZIO[R, E, A],
  schedule: Schedule[R, E, Any]
): ZIO[R, E, A] = {
  effect.retry(schedule)
}

// Пример: экспоненциальный retry с ограничением
val task: Task[String] = ZIO.attempt {
  if (scala.util.Random.nextBoolean()) "success"
  else throw new Exception("failure")
}

val retriedTask: Task[String] = task.retry(
  Schedule.exponential(100.millis) && Schedule.recurs(5)
)
```

### Custom ZIO операторы

```scala
import zio._

// Расширение ZIO с custom операторами
implicit class ZIOOps[R, E, A](zio: ZIO[R, E, A]) {
  // Логирование результата
  def logResult(prefix: String): ZIO[R, E, A] = 
    zio.tap(value => Console.printLine(s"$prefix: $value").orDie)
  
  // Измерение времени выполнения
  def timed: ZIO[R, E, (Duration, A)] = 
    zio.timedWith(Clock.nanoTime)((start, end, result) => 
      (Duration.fromNanos(end - start), result)
    )
  
  // Кеширование результата
  def cached(ttl: Duration): ZIO[R, E, A] = 
    zio.cached(ttl)
}

// Использование
val program: Task[Int] = for {
  (duration, result) <- ZIO.attempt(42).delay(1.second).logResult("Результат").timed
  _ <- Console.printLine(s"Заняло: ${duration.toMillis}мс")
} yield result
```

### STM (Software Transactional Memory)

```scala
import zio._
import zio.stm._

// Атомарные транзакции
case class Account(balance: TRef[Int])

def transfer(from: Account, to: Account, amount: Int): UIO[Unit] = {
  STM.atomically {
    for {
      _ <- from.balance.update(_ - amount)
      _ <- to.balance.update(_ + amount)
    } yield ()
  }
}

// Пример использования
val bankExample: UIO[Unit] = for {
  account1 <- TRef.make(1000).commit
  account2 <- TRef.make(500).commit
  
  _ <- transfer(Account(account1), Account(account2), 200)
  
  balance1 <- account1.get.commit
  balance2 <- account2.get.commit
  
  _ <- Console.printLine(s"Счет 1: $balance1").orDie
  _ <- Console.printLine(s"Счет 2: $balance2").orDie
} yield ()
```

---

## Практические проекты

### Проект 1: REST API сервер

```scala
import zio._
import zio.http._
import zio.json._

// Модели данных
case class User(id: Int, name: String, email: String)

object User {
  implicit val encoder: JsonEncoder[User] = DeriveJsonEncoder.gen[User]
  implicit val decoder: JsonDecoder[User] = DeriveJsonDecoder.gen[User]
}

// Репозиторий
trait UserRepository {
  def getUser(id: Int): Task[Option[User]]
  def getAllUsers: Task[List[User]]
  def createUser(user: User): Task[User]
  def updateUser(id: Int, user: User): Task[Option[User]]
  def deleteUser(id: Int): Task[Boolean]
}

case class InMemoryUserRepository(users: Ref[Map[Int, User]]) extends UserRepository {
  def getUser(id: Int): Task[Option[User]] = 
    users.get.map(_.get(id))
  
  def getAllUsers: Task[List[User]] = 
    users.get.map(_.values.toList)
  
  def createUser(user: User): Task[User] = 
    users.update(_ + (user.id -> user)).as(user)
  
  def updateUser(id: Int, user: User): Task[Option[User]] = 
    users.modify { map =>
      if (map.contains(id)) (Some(user), map + (id -> user))
      else (None, map)
    }
  
  def deleteUser(id: Int): Task[Boolean] = 
    users.modify { map =>
      if (map.contains(id)) (true, map - id)
      else (false, map)
    }
}

object InMemoryUserRepository {
  val layer: ULayer[UserRepository] = ZLayer {
    for {
      users <- Ref.make(Map.empty[Int, User])
    } yield InMemoryUserRepository(users)
  }
}

// HTTP маршруты
object UserRoutes {
  def apply(): Http[UserRepository, Throwable, Request, Response] = {
    Http.collectZIO[Request] {
      // GET /users
      case Method.GET -> Root / "users" =>
        for {
          repo <- ZIO.service[UserRepository]
          users <- repo.getAllUsers
        } yield Response.json(users.toJson)
      
      // GET /users/:id
      case Method.GET -> Root / "users" / id =>
        for {
          repo <- ZIO.service[UserRepository]
          user <- repo.getUser(id.toInt)
          response <- user match {
            case Some(u) => ZIO.succeed(Response.json(u.toJson))
            case None => ZIO.succeed(Response.status(Status.NotFound))
          }
        } yield response
      
      // POST /users
      case req @ Method.POST -> Root / "users" =>
        for {
          body <- req.body.asString
          user <- ZIO.fromEither(body.fromJson[User])
            .mapError(e => new Exception(e))
          repo <- ZIO.service[UserRepository]
          created <- repo.createUser(user)
        } yield Response.json(created.toJson)
    }
  }
}

// Главное приложение
object HttpServerApp extends ZIOAppDefault {
  val app = UserRoutes()
  
  val program = for {
    _ <- Console.printLine("Сервер запущен на http://localhost:8080")
    _ <- Server.serve(app).provide(
      Server.defaultWithPort(8080),
      InMemoryUserRepository.layer
    )
  } yield ()
  
  def run = program
}
```

### Проект 2: Консольное TODO приложение

```scala
import zio._
import zio.Console._

// Модель данных
case class Todo(id: Int, title: String, completed: Boolean)

// Сервис управления задачами
trait TodoService {
  def addTodo(title: String): UIO[Todo]
  def listTodos: UIO[List[Todo]]
  def completeTodo(id: Int): UIO[Boolean]
  def deleteTodo(id: Int): UIO[Boolean]
}

case class TodoServiceLive(
  todos: Ref[Map[Int, Todo]],
  nextId: Ref[Int]
) extends TodoService {
  
  def addTodo(title: String): UIO[Todo] = for {
    id <- nextId.getAndUpdate(_ + 1)
    todo = Todo(id, title, completed = false)
    _ <- todos.update(_ + (id -> todo))
  } yield todo
  
  def listTodos: UIO[List[Todo]] = 
    todos.get.map(_.values.toList.sortBy(_.id))
  
  def completeTodo(id: Int): UIO[Boolean] = 
    todos.modify { map =>
      map.get(id) match {
        case Some(todo) => 
          (true, map + (id -> todo.copy(completed = true)))
        case None => 
          (false, map)
      }
    }
  
  def deleteTodo(id: Int): UIO[Boolean] = 
    todos.modify { map =>
      if (map.contains(id)) (true, map - id)
      else (false, map)
    }
}

object TodoServiceLive {
  val layer: ULayer[TodoService] = ZLayer {
    for {
      todos <- Ref.make(Map.empty[Int, Todo])
      nextId <- Ref.make(1)
    } yield TodoServiceLive(todos, nextId)
  }
}

// CLI интерфейс
object TodoApp extends ZIOAppDefault {
  
  def displayMenu: UIO[Unit] = printLine(
    """
      |=== TODO Приложение ===
      |1. Добавить задачу
      |2. Показать все задачи
      |3. Завершить задачу
      |4. Удалить задачу
      |5. Выход
      |
      |Выберите действие:
    """.stripMargin
  ).orDie
  
  def displayTodos(todos: List[Todo]): UIO[Unit] = {
    if (todos.isEmpty) {
      printLine("Задач нет").orDie
    } else {
      ZIO.foreach(todos) { todo =>
        val status = if (todo.completed) "[x]" else "[ ]"
        printLine(s"${todo.id}. $status ${todo.title}").orDie
      }.unit
    }
  }
  
  def handleAddTodo: ZIO[TodoService, Nothing, Unit] = for {
    _ <- printLine("Введите название задачи:").orDie
    title <- readLine.orDie
    service <- ZIO.service[TodoService]
    todo <- service.addTodo(title)
    _ <- printLine(s"Задача добавлена: ${todo.title}").orDie
  } yield ()
  
  def handleListTodos: ZIO[TodoService, Nothing, Unit] = for {
    service <- ZIO.service[TodoService]
    todos <- service.listTodos
    _ <- displayTodos(todos)
  } yield ()
  
  def handleCompleteTodo: ZIO[TodoService, Nothing, Unit] = for {
    _ <- printLine("Введите ID задачи:").orDie
    id <- readLine.orDie.map(_.toInt)
    service <- ZIO.service[TodoService]
    success <- service.completeTodo(id)
    _ <- if (success) printLine("Задача завершена").orDie
         else printLine("Задача не найдена").orDie
  } yield ()
  
  def handleDeleteTodo: ZIO[TodoService, Nothing, Unit] = for {
    _ <- printLine("Введите ID задачи:").orDie
    id <- readLine.orDie.map(_.toInt)
    service <- ZIO.service[TodoService]
    success <- service.deleteTodo(id)
    _ <- if (success) printLine("Задача удалена").orDie
         else printLine("Задача не найдена").orDie
  } yield ()
  
  def mainLoop: ZIO[TodoService, Nothing, Unit] = {
    val iteration = for {
      _ <- displayMenu
      choice <- readLine.orDie
      _ <- choice match {
        case "1" => handleAddTodo
        case "2" => handleListTodos
        case "3" => handleCompleteTodo
        case "4" => handleDeleteTodo
        case "5" => printLine("До свидания!").orDie *> ZIO.succeed(false)
        case _ => printLine("Неверный выбор").orDie *> ZIO.succeed(true)
      }
    } yield choice != "5"
    
    iteration.flatMap {
      case true => mainLoop
      case false => ZIO.unit
    }
  }
  
  def run = mainLoop.provide(TodoServiceLive.layer)
}
```

### Проект 3: Параллельный веб-скрапер

```scala
import zio._
import zio.stream._

case class PageContent(url: String, content: String, links: List[String])

trait WebScraper {
  def fetchPage(url: String): Task[PageContent]
  def scrapeWebsite(startUrl: String, maxDepth: Int): Task[List[PageContent]]
}

case class WebScraperLive() extends WebScraper {
  
  def fetchPage(url: String): Task[PageContent] = {
    // Упрощенная реализация
    ZIO.attempt {
      val content = s"Содержимое страницы $url"
      val links = List(s"$url/link1", s"$url/link2")
      PageContent(url, content, links)
    }.delay(500.millis) // имитация сетевой задержки
  }
  
  def scrapeWebsite(startUrl: String, maxDepth: Int): Task[List[PageContent]] = {
    def scrapeLevel(
      urls: Set[String],
      visited: Set[String],
      depth: Int
    ): Task[List[PageContent]] = {
      if (depth > maxDepth || urls.isEmpty) {
        ZIO.succeed(List.empty)
      } else {
        val newUrls = urls -- visited
        
        for {
          pages <- ZIO.foreachPar(newUrls.toList) { url =>
            fetchPage(url).tap(page =>
              Console.printLine(s"Загружена страница: $url").orDie
            )
          }.withParallelism(5) // максимум 5 одновременных запросов
          
          allLinks = pages.flatMap(_.links).toSet
          nextLevel <- scrapeLevel(allLinks, visited ++ newUrls, depth + 1)
        } yield pages ++ nextLevel
      }
    }
    
    scrapeLevel(Set(startUrl), Set.empty, 0)
  }
}

object WebScraperLive {
  val layer: ULayer[WebScraper] = ZLayer.succeed(WebScraperLive())
}

// Использование
object ScraperApp extends ZIOAppDefault {
  val program: ZIO[WebScraper, Throwable, Unit] = for {
    scraper <- ZIO.service[WebScraper]
    pages <- scraper.scrapeWebsite("https://example.com", maxDepth = 2)
    _ <- Console.printLine(s"Загружено ${pages.length} страниц")
  } yield ()
  
  def run = program.provide(WebScraperLive.layer)
}
```

---

## Полезные ресурсы

### Официальная документация
- [ZIO Documentation](https://zio.dev/documentation/)
- [ZIO GitHub](https://github.com/zio/zio)
- [Scaladoc](https://javadoc.io/doc/dev.zio/zio_2.13/latest/zio/index.html)

### Книги
- "ZIO by Example" (официальная книга)
- "Functional Programming in Scala" (фундаментальные основы)

### Видео курсы
- Rock the JVM - ZIO курс
- ZIO World конференции

### Сообщество
- ZIO Discord сервер
- ZIO Forum
- Stack Overflow (тег zio)

### Дополнительные библиотеки экосистемы
- **zio-http** - HTTP клиент/сервер
- **zio-json** - JSON сериализация
- **zio-kafka** - интеграция с Kafka
- **zio-sql** - типобезопасный SQL
- **zio-config** - управление конфигурацией
- **zio-logging** - логирование
- **zio-metrics** - метрики и мониторинг
- **zio-cache** - кеширование

---

## План обучения (рекомендуемый порядок)

### Неделя 1-2: Основы
- Понимание ZIO эффектов
- Создание и комбинирование эффектов
- Базовая обработка ошибок
- Практика: простые CLI приложения

### Неделя 3-4: Конкурентность
- Fibers и параллелизм
- Ref, Promise, Queue
- Обработка ошибок и повторы
- Практика: многопоточные приложения

### Неделя 5-6: Продвинутые концепции
- ZLayer и dependency injection
- ZIO Streams
- Управление ресурсами
- Практика: REST API

### Неделя 7-8: Мастерство
- Тестирование с ZIO Test
- STM
- Продвинутые паттерны
- Практика: полноценное приложение

### Постоянная практика
- Решение задач на GitHub
- Участие в open-source проектах
- Чтение исходного кода ZIO
- Написание собственных библиотек

---

## Заключение

ZIO — мощный инструмент для построения надежных, типобезопасных и высокопроизводительных приложений на Scala. Освоение ZIO откроет новые горизонты в функциональном программировании и поможет писать более качественный код.

Ключевые моменты для запоминания:
1. ZIO эффекты ленивы и композиционны
2. Типобезопасность помогает избежать ошибок на этапе компиляции
3. ZIO предоставляет отличные инструменты для конкурентности
4. Тестируемость достигается через dependency injection с ZLayer
5. Экосистема ZIO богата и постоянно развивается

Удачи в изучении ZIO! 🚀
