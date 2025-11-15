# Полное Руководство по Akka для Senior Scala Developer

## Оглавление

1. [План подготовки](#план-подготовки)
2. [Введение в Actor Model](#1-введение-в-actor-model)
3. [Akka Classic Actors](#2-akka-classic-actors)
4. [Akka Typed Actors](#3-akka-typed-actors)
5. [Supervision и Error Handling](#4-supervision-и-error-handling)
6. [Akka Streams](#5-akka-streams)
7. [Akka HTTP](#6-akka-http)
8. [Akka Persistence](#7-akka-persistence)
9. [Akka Cluster](#8-akka-cluster)
10. [Akka Sharding](#9-akka-sharding)
11. [Testing Akka Applications](#10-testing-akka-applications)
12. [Performance и Best Practices](#11-performance-и-best-practices)
13. [Migration to Pekko](#12-migration-to-pekko)
14. [Real-World Patterns](#13-real-world-patterns)

---

## План подготовки

### Неделя 1: Основы Actor Model
- День 1-2: Actor Model концепции, создание акторов
- День 3-4: Сообщения, mailbox, lifecycle
- День 5-6: Supervision strategies
- День 7: Практика - создание простой actor системы

### Неделя 2: Typed Actors и Advanced Patterns
- День 1-2: Akka Typed API
- День 3-4: Behaviors и state management
- День 5-6: Actor patterns (router, pool, dispatcher)
- День 7: Практика - рефакторинг на Typed

### Неделя 3: Akka Streams
- День 1-2: Sources, Flows, Sinks
- День 3-4: Graph DSL
- День 5-6: Backpressure, error handling
- День 7: Практика - streaming pipeline

### Неделя 4: Akka HTTP
- День 1-2: Routing DSL
- День 3-4: Directives, marshalling
- День 5-6: WebSockets, streaming responses
- День 7: Практика - REST API

### Неделя 5: Akka Persistence
- День 1-2: Event Sourcing концепции
- День 3-4: PersistentActor, EventSourcedBehavior
- День 5-6: Snapshots, recovery
- День 7: Практика - event-sourced система

### Неделя 6: Akka Cluster
- День 1-2: Cluster formation, membership
- День 3-4: Cluster Singleton, Distributed Data
- День 5-6: Cluster Sharding
- День 7: Практика - distributed application

### Неделя 7-8: Advanced Topics и Practice
- Тестирование
- Performance tuning
- Production patterns
- Real-world проекты

---

## 1. Введение в Actor Model

### 1.1 Теория Actor Model

**Actor Model** - математическая модель параллельных вычислений, предложенная Carl Hewitt в 1973 году.

#### Основные принципы:

```
┌─────────────────────────────────────────┐
│  Actor - основная единица вычислений    │
│                                         │
│  Может:                                 │
│  1. Получать сообщения                  │
│  2. Создавать новых акторов             │
│  3. Отправлять сообщения                │
│  4. Определять поведение для следующего │
│     сообщения                          │
└─────────────────────────────────────────┘
```

#### Модель коммуникации:

```
     Actor A                    Actor B
┌──────────────┐          ┌──────────────┐
│   State      │          │   State      │
│   Behavior   │          │   Behavior   │
│   Mailbox    │          │   Mailbox    │
└──────┬───────┘          └──────┬───────┘
       │                         │
       │   Message (async)       │
       └────────────────────────▶│
                                 │
                          ┌──────▼───────┐
                          │   Process    │
                          │   Message    │
                          └──────────────┘
```

#### Преимущества Actor Model:

1. **Изолированное состояние** - нет shared mutable state
2. **Асинхронная коммуникация** - non-blocking
3. **Location transparency** - актор может быть где угодно
4. **Fault tolerance** - supervision иерархия
5. **Масштабируемость** - легко распределяется

#### Сравнение с другими моделями:

```scala
// Традиционный multithreading (shared mutable state)
class Counter {
  private var count: Int = 0
  
  def increment(): Unit = synchronized {
    count += 1  // требует синхронизации!
  }
  
  def get: Int = synchronized { count }
}

// Actor Model (isolated state)
class CounterActor extends Actor {
  private var count: Int = 0  // никакой синхронизации не нужно!
  
  def receive: Receive = {
    case "increment" => count += 1
    case "get" => sender() ! count
  }
}
```

### 1.2 Akka Framework Overview

**Akka** - toolkit для построения concurrent, distributed, resilient message-driven приложений.

```
Akka Ecosystem:
┌────────────────────────────────────────────┐
│  Akka Actor           ← Core               │
│  Akka Typed           ← Type-safe actors   │
│  Akka Streams         ← Reactive streams   │
│  Akka HTTP            ← HTTP server/client │
│  Akka Persistence     ← Event sourcing     │
│  Akka Cluster         ← Distribution       │
│  Akka Sharding        ← Partitioning       │
└────────────────────────────────────────────┘
```

#### Основные модули:

1. **akka-actor** - базовая реализация акторов
2. **akka-actor-typed** - type-safe акторы
3. **akka-stream** - реактивные потоки
4. **akka-http** - HTTP сервер и клиент
5. **akka-persistence** - event sourcing
6. **akka-cluster** - clustering support
7. **akka-cluster-sharding** - distributed actors

---

## 2. Akka Classic Actors

### 2.1 Создание первого актора

```scala
import akka.actor.{Actor, ActorRef, ActorSystem, Props}

// 1. Определяем сообщения
case class Greet(name: String)
case class Greeting(message: String)

// 2. Определяем актор
class GreeterActor extends Actor {
  
  def receive: Receive = {
    case Greet(name) =>
      println(s"Hello, $name!")
      sender() ! Greeting(s"Hello, $name!")
    
    case msg =>
      println(s"Unknown message: $msg")
  }
}

// 3. Создаем ActorSystem и актор
object GreeterApp extends App {
  val system = ActorSystem("GreeterSystem")
  
  val greeter: ActorRef = system.actorOf(
    Props[GreeterActor],
    name = "greeter"
  )
  
  // 4. Отправляем сообщения
  greeter ! Greet("Alice")
  greeter ! Greet("Bob")
  
  // 5. Корректное завершение
  Thread.sleep(1000)
  system.terminate()
}
```

### 2.2 Props и Actor Creation

```scala
// Способы создания Props

// 1. Без параметров
class SimpleActor extends Actor {
  def receive: Receive = {
    case msg => println(msg)
  }
}

val props1 = Props[SimpleActor]
val props2 = Props(new SimpleActor)  // НЕ рекомендуется!

// 2. С параметрами конструктора
class ParameterizedActor(greeting: String, times: Int) extends Actor {
  def receive: Receive = {
    case name: String =>
      (1 to times).foreach(_ => println(s"$greeting, $name!"))
  }
}

// Правильно - через Props
val props3 = Props(classOf[ParameterizedActor], "Hello", 3)

// Неправильно - замыкание может привести к проблемам
// val props4 = Props(new ParameterizedActor("Hello", 3))

// 3. С companion object (рекомендуется)
object ParameterizedActor {
  def props(greeting: String, times: Int): Props =
    Props(classOf[ParameterizedActor], greeting, times)
}

val props5 = ParameterizedActor.props("Hi", 2)

// 4. Разные варианты создания акторов
class ParentActor extends Actor {
  
  // Дочерний актор
  val child: ActorRef = context.actorOf(
    Props[ChildActor],
    name = "child"
  )
  
  def receive: Receive = {
    case msg => child ! msg
  }
}

// В ActorSystem
val system = ActorSystem("MySystem")
val rootActor = system.actorOf(Props[ParentActor], "parent")

// Получение актора по пути
val selection = system.actorSelection("/user/parent/child")
```

### 2.3 Actor Lifecycle

```scala
class LifecycleActor extends Actor {
  
  println("Constructor called")
  
  // Вызывается при старте актора
  override def preStart(): Unit = {
    println("preStart: Actor is starting")
    // Инициализация ресурсов
  }
  
  // Вызывается перед остановкой
  override def postStop(): Unit = {
    println("postStop: Actor is stopping")
    // Освобождение ресурсов
  }
  
  // Вызывается при перезапуске (после ошибки)
  override def preRestart(reason: Throwable, message: Option[Any]): Unit = {
    println(s"preRestart: Actor restarting due to: ${reason.getMessage}")
    super.preRestart(reason, message)
    // По умолчанию вызывает postStop
  }
  
  // Вызывается после перезапуска
  override def postRestart(reason: Throwable): Unit = {
    println(s"postRestart: Actor restarted")
    super.postRestart(reason)
    // По умолчанию вызывает preStart
  }
  
  def receive: Receive = {
    case "stop" =>
      context.stop(self)
    
    case "crash" =>
      throw new RuntimeException("Intentional crash!")
    
    case msg =>
      println(s"Received: $msg")
  }
}

// Lifecycle diagram
/*
┌─────────────┐
│ Constructor │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  preStart   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Receiving  │◄────┐
└──────┬──────┘     │
       │            │
       ├── Error ───┤
       │            │
       ▼            │
┌─────────────┐    │
│ preRestart  │    │
└──────┬──────┘    │
       │            │
       ▼            │
┌─────────────┐    │
│ postRestart │────┘
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   postStop  │
└─────────────┘
*/
```

### 2.4 Message Patterns

#### Ask Pattern (Request-Response)

```scala
import akka.pattern.ask
import akka.util.Timeout
import scala.concurrent.duration._
import scala.concurrent.Future

case class GetUser(id: Long)
case class User(id: Long, name: String)

class UserActor extends Actor {
  private val users = Map(
    1L -> User(1, "Alice"),
    2L -> User(2, "Bob")
  )
  
  def receive: Receive = {
    case GetUser(id) =>
      sender() ! users.get(id)
  }
}

object AskExample extends App {
  val system = ActorSystem("AskSystem")
  val userActor = system.actorOf(Props[UserActor])
  
  implicit val timeout: Timeout = Timeout(5.seconds)
  import system.dispatcher
  
  // Ask возвращает Future
  val future: Future[Any] = userActor ? GetUser(1)
  
  val userFuture: Future[Option[User]] = future.mapTo[Option[User]]
  
  userFuture.foreach {
    case Some(user) => println(s"Found: $user")
    case None => println("User not found")
  }
  
  Thread.sleep(1000)
  system.terminate()
}
```

#### Fire and Forget (Tell)

```scala
class LoggerActor extends Actor {
  def receive: Receive = {
    case msg: String =>
      println(s"[${java.time.Instant.now}] $msg")
  }
}

val logger = system.actorOf(Props[LoggerActor])

// Tell - отправить и забыть (no response expected)
logger ! "Application started"
logger ! "Processing request"
logger ! "Request completed"
```

#### Forward Pattern

```scala
class ProxyActor(target: ActorRef) extends Actor {
  def receive: Receive = {
    case msg =>
      // Forward сохраняет оригинального sender
      target forward msg
  }
}

class WorkerActor extends Actor {
  def receive: Receive = {
    case msg: String =>
      // sender() будет оригинальным отправителем, не ProxyActor
      sender() ! msg.toUpperCase
  }
}
```

#### PipeTo Pattern

```scala
import akka.pattern.pipe

class DatabaseActor extends Actor {
  import context.dispatcher
  
  def receive: Receive = {
    case query: String =>
      // Выполняем асинхронную операцию
      val futureResult: Future[String] = Future {
        Thread.sleep(100)
        s"Result for: $query"
      }
      
      // Pipe результат обратно себе или другому актору
      futureResult pipeTo sender()
  }
}
```

### 2.5 Stateful Actors

```scala
// Счетчик с состоянием
class CounterActor extends Actor {
  private var count: Int = 0
  
  def receive: Receive = {
    case "increment" =>
      count += 1
      sender() ! count
    
    case "decrement" =>
      count -= 1
      sender() ! count
    
    case "get" =>
      sender() ! count
    
    case "reset" =>
      count = 0
  }
}

// FSM (Finite State Machine) с become/unbecome
class AuthenticationActor extends Actor {
  
  def receive: Receive = unauthenticated
  
  def unauthenticated: Receive = {
    case Login(username, password) =>
      if (authenticate(username, password)) {
        println(s"User $username authenticated")
        context.become(authenticated(username))
        sender() ! LoginSuccess
      } else {
        sender() ! LoginFailure
      }
  }
  
  def authenticated(username: String): Receive = {
    case GetProfile =>
      sender() ! UserProfile(username, "user@example.com")
    
    case Logout =>
      println(s"User $username logged out")
      context.unbecome()  // возврат к unauthenticated
      sender() ! LogoutSuccess
    
    case Login(_, _) =>
      sender() ! AlreadyAuthenticated
  }
  
  private def authenticate(username: String, password: String): Boolean = {
    // Simplified authentication
    username == "admin" && password == "secret"
  }
}

case class Login(username: String, password: String)
case object GetProfile
case object Logout
case object LoginSuccess
case object LoginFailure
case object LogoutSuccess
case object AlreadyAuthenticated
case class UserProfile(username: String, email: String)
```

### 2.6 Router Patterns

```scala
import akka.routing._

// 1. Round Robin Router
class WorkerActor extends Actor {
  def receive: Receive = {
    case work: String =>
      println(s"${self.path.name} processing: $work")
      Thread.sleep(1000)
      sender() ! s"Completed: $work"
  }
}

val roundRobinRouter: ActorRef = system.actorOf(
  RoundRobinPool(5).props(Props[WorkerActor]),
  "round-robin-router"
)

// Сообщения распределяются по круговому принципу
(1 to 10).foreach(i => roundRobinRouter ! s"Task-$i")

// 2. Smallest Mailbox Router
val smallestMailboxRouter: ActorRef = system.actorOf(
  SmallestMailboxPool(5).props(Props[WorkerActor]),
  "smallest-mailbox-router"
)

// Отправляет сообщения актору с наименьшим mailbox

// 3. Broadcast Router
val broadcastRouter: ActorRef = system.actorOf(
  BroadcastPool(5).props(Props[WorkerActor]),
  "broadcast-router"
)

// Отправляет сообщения ВСЕМ routees
broadcastRouter ! "Broadcast message"

// 4. Random Router
val randomRouter: ActorRef = system.actorOf(
  RandomPool(5).props(Props[WorkerActor]),
  "random-router"
)

// 5. Consistent Hashing Router
class HashableWorkActor extends Actor {
  def receive: Receive = {
    case work: HashableWork =>
      println(s"${self.path.name} processing: ${work.key}")
  }
}

case class HashableWork(key: String, payload: String) extends ConsistentHashable {
  override def consistentHashKey: Any = key
}

val hashingRouter: ActorRef = system.actorOf(
  ConsistentHashingPool(5).props(Props[HashableWorkActor]),
  "hashing-router"
)

// Одинаковые ключи всегда идут к одному и тому же routee
hashingRouter ! HashableWork("user-123", "data1")
hashingRouter ! HashableWork("user-123", "data2")  // к тому же воркеру
hashingRouter ! HashableWork("user-456", "data3")  // к другому воркеру

// 6. Custom Router Logic
class CustomRouter extends Actor {
  val workers: IndexedSeq[ActorRef] = (1 to 5).map { i =>
    context.actorOf(Props[WorkerActor], s"worker-$i")
  }
  
  private var currentIndex = 0
  
  def receive: Receive = {
    case PriorityWork(priority, work) =>
      // Кастомная логика распределения
      val workerIndex = if (priority > 5) 0 else currentIndex
      workers(workerIndex) ! work
      currentIndex = (currentIndex + 1) % workers.size
    
    case work =>
      workers(currentIndex) ! work
      currentIndex = (currentIndex + 1) % workers.size
  }
}

case class PriorityWork(priority: Int, work: String)
```

### 2.7 Dispatchers

```scala
// application.conf
/*
my-dispatcher {
  type = Dispatcher
  executor = "thread-pool-executor"
  thread-pool-executor {
    fixed-pool-size = 10
  }
  throughput = 100
}

my-pinned-dispatcher {
  type = PinnedDispatcher
  executor = "thread-pool-executor"
}

blocking-io-dispatcher {
  type = Dispatcher
  executor = "thread-pool-executor"
  thread-pool-executor {
    fixed-pool-size = 32
  }
  throughput = 1
}
*/

// Использование диспетчеров
class IOActor extends Actor {
  def receive: Receive = {
    case ReadFile(path) =>
      // Blocking I/O operation
      val content = scala.io.Source.fromFile(path).mkString
      sender() ! content
  }
}

// Создание актора с кастомным диспетчером
val ioActor = system.actorOf(
  Props[IOActor].withDispatcher("blocking-io-dispatcher"),
  "io-actor"
)

// Pinned dispatcher - один поток на актор
val importantActor = system.actorOf(
  Props[ImportantActor].withDispatcher("my-pinned-dispatcher"),
  "important-actor"
)

// Default dispatcher
val normalActor = system.actorOf(Props[NormalActor])
```

---

## 3. Akka Typed Actors

### 3.1 Основы Akka Typed

**Akka Typed** - type-safe API для акторов с проверкой типов на этапе компиляции.

```scala
import akka.actor.typed._
import akka.actor.typed.scaladsl._

// Протокол актора
sealed trait CounterCommand
case class Increment(replyTo: ActorRef[Int]) extends CounterCommand
case class Decrement(replyTo: ActorRef[Int]) extends CounterCommand
case class GetValue(replyTo: ActorRef[Int]) extends CounterCommand

// Typed actor behavior
object CounterActor {
  def apply(): Behavior[CounterCommand] = counter(0)
  
  private def counter(value: Int): Behavior[CounterCommand] =
    Behaviors.receive { (context, message) =>
      message match {
        case Increment(replyTo) =>
          val newValue = value + 1
          replyTo ! newValue
          counter(newValue)
        
        case Decrement(replyTo) =>
          val newValue = value - 1
          replyTo ! newValue
          counter(newValue)
        
        case GetValue(replyTo) =>
          replyTo ! value
          Behaviors.same
      }
    }
}

// Использование
object TypedExample extends App {
  val system: ActorSystem[CounterCommand] = 
    ActorSystem(CounterActor(), "CounterSystem")
  
  import akka.actor.typed.scaladsl.AskPattern._
  import akka.util.Timeout
  import scala.concurrent.duration._
  import scala.concurrent.ExecutionContext.Implicits.global
  
  implicit val timeout: Timeout = 3.seconds
  
  val future = system.ask(ref => Increment(ref))
  future.foreach(value => println(s"Value: $value"))
  
  Thread.sleep(1000)
  system.terminate()
}
```

### 3.2 Behavior Types

```scala
// 1. Receive - базовое поведение
def basicBehavior: Behavior[String] = 
  Behaviors.receive { (context, message) =>
    context.log.info(s"Received: $message")
    Behaviors.same
  }

// 2. ReceiveMessage - без доступа к context
def messageBehavior: Behavior[String] =
  Behaviors.receiveMessage { message =>
    println(s"Got: $message")
    Behaviors.same
  }

// 3. Setup - с инициализацией
def setupBehavior: Behavior[String] =
  Behaviors.setup { context =>
    val child = context.spawn(childBehavior, "child")
    context.log.info("Actor initialized")
    
    Behaviors.receiveMessage { message =>
      child ! message
      Behaviors.same
    }
  }

// 4. Stopped - остановленное состояние
def stoppableBehavior: Behavior[String] =
  Behaviors.receiveMessage {
    case "stop" =>
      Behaviors.stopped
    case msg =>
      println(msg)
      Behaviors.same
  }

// 5. Empty - игнорирует все сообщения
val emptyBehavior: Behavior[String] = Behaviors.empty

// 6. Ignore - игнорирует все сообщения
val ignoreBehavior: Behavior[String] = Behaviors.ignore
```

### 3.3 Stateful Behaviors

```scala
object ShoppingCart {
  sealed trait Command
  case class AddItem(item: String, replyTo: ActorRef[Response]) extends Command
  case class RemoveItem(item: String, replyTo: ActorRef[Response]) extends Command
  case class GetItems(replyTo: ActorRef[Items]) extends Command
  case class Checkout(replyTo: ActorRef[Response]) extends Command
  
  sealed trait Response
  case object ItemAdded extends Response
  case object ItemRemoved extends Response
  case object CheckedOut extends Response
  case class Items(items: List[String])
  
  def apply(): Behavior[Command] = empty
  
  private def empty: Behavior[Command] = 
    Behaviors.receive { (context, message) =>
      message match {
        case AddItem(item, replyTo) =>
          replyTo ! ItemAdded
          withItems(List(item))
        
        case RemoveItem(_, replyTo) =>
          replyTo ! ItemRemoved
          Behaviors.same
        
        case GetItems(replyTo) =>
          replyTo ! Items(Nil)
          Behaviors.same
        
        case Checkout(replyTo) =>
          replyTo ! CheckedOut
          Behaviors.same
      }
    }
  
  private def withItems(items: List[String]): Behavior[Command] =
    Behaviors.receive { (context, message) =>
      message match {
        case AddItem(item, replyTo) =>
          replyTo ! ItemAdded
          withItems(item :: items)
        
        case RemoveItem(item, replyTo) =>
          replyTo ! ItemRemoved
          withItems(items.filterNot(_ == item))
        
        case GetItems(replyTo) =>
          replyTo ! Items(items)
          Behaviors.same
        
        case Checkout(replyTo) =>
          context.log.info(s"Checking out: ${items.mkString(", ")}")
          replyTo ! CheckedOut
          empty
      }
    }
}
```

### 3.4 Actor Context Operations

```scala
object ParentActor {
  sealed trait Command
  case class SpawnChild(name: String, replyTo: ActorRef[ActorRef[String]]) extends Command
  case class StopChild(name: String) extends Command
  case class WatchChild(name: String) extends Command
  case object GetChildren extends Command
  
  def apply(): Behavior[Command] =
    Behaviors.receive { (context, message) =>
      message match {
        case SpawnChild(name, replyTo) =>
          // Создание дочернего актора
          val child = context.spawn(
            Behaviors.receiveMessage[String] { msg =>
              context.log.info(s"Child received: $msg")
              Behaviors.same
            },
            name
          )
          replyTo ! child
          Behaviors.same
        
        case StopChild(name) =>
          // Остановка дочернего актора
          context.child(name).foreach(context.stop)
          Behaviors.same
        
        case WatchChild(name) =>
          // Watch для мониторинга жизненного цикла
          context.child(name).foreach(context.watch)
          Behaviors.same
        
        case GetChildren =>
          // Получение всех дочерних акторов
          context.children.foreach { child =>
            context.log.info(s"Child: ${child.path}")
          }
          Behaviors.same
      }
    }
}

// Timers
object TimerExample {
  sealed trait Command
  case object Tick extends Command
  case class StartTimer(interval: FiniteDuration) extends Command
  case object StopTimer extends Command
  
  def apply(): Behavior[Command] =
    Behaviors.setup { context =>
      Behaviors.withTimers { timers =>
        Behaviors.receiveMessage {
          case StartTimer(interval) =>
            timers.startTimerWithFixedDelay("ticker", Tick, interval)
            Behaviors.same
          
          case Tick =>
            context.log.info("Tick!")
            Behaviors.same
          
          case StopTimer =>
            timers.cancel("ticker")
            Behaviors.same
        }
      }
    }
}

// Stash
object StashExample {
  sealed trait Command
  case class Process(data: String) extends Command
  case object Initialize extends Command
  case object Ready extends Command
  
  def apply(): Behavior[Command] =
    Behaviors.withStash(100) { buffer =>
      Behaviors.receive { (context, message) =>
        message match {
          case Initialize =>
            // Имитация инициализации
            context.self ! Ready
            Behaviors.same
          
          case Ready =>
            // Unstash все отложенные сообщения
            buffer.unstashAll(ready)
          
          case Process(data) =>
            // Откладываем сообщения до инициализации
            buffer.stash(message)
            Behaviors.same
        }
      }
    }
  
  private def ready: Behavior[Command] =
    Behaviors.receiveMessage {
      case Process(data) =>
        println(s"Processing: $data")
        Behaviors.same
      
      case _ =>
        Behaviors.unhandled
    }
}
```

### 3.5 Actor Discovery

```scala
// Receptionist - service discovery
object Worker {
  sealed trait Command
  case class DoWork(task: String) extends Command
  
  val WorkerServiceKey = ServiceKey[Command]("worker")
  
  def apply(): Behavior[Command] =
    Behaviors.setup { context =>
      // Регистрация в Receptionist
      context.system.receptionist ! Receptionist.Register(
        WorkerServiceKey,
        context.self
      )
      
      Behaviors.receiveMessage {
        case DoWork(task) =>
          context.log.info(s"Working on: $task")
          Behaviors.same
      }
    }
}

object Coordinator {
  sealed trait Command
  case class DistributeWork(tasks: List[String]) extends Command
  private case class WorkersUpdated(workers: Set[ActorRef[Worker.Command]]) extends Command
  
  def apply(): Behavior[Command] =
    Behaviors.setup { context =>
      // Подписка на обновления worker'ов
      val subscriptionAdapter = context.messageAdapter[Receptionist.Listing] {
        case Worker.WorkerServiceKey.Listing(workers) =>
          WorkersUpdated(workers)
      }
      
      context.system.receptionist ! Receptionist.Subscribe(
        Worker.WorkerServiceKey,
        subscriptionAdapter
      )
      
      waiting(Set.empty)
    }
  
  private def waiting(workers: Set[ActorRef[Worker.Command]]): Behavior[Command] =
    Behaviors.receiveMessage {
      case WorkersUpdated(newWorkers) =>
        waiting(newWorkers)
      
      case DistributeWork(tasks) if workers.nonEmpty =>
        val workersList = workers.toList
        tasks.zipWithIndex.foreach { case (task, index) =>
          val worker = workersList(index % workersList.size)
          worker ! Worker.DoWork(task)
        }
        Behaviors.same
      
      case DistributeWork(_) =>
        context.log.warn("No workers available")
        Behaviors.same
    }
}
```

---

## 4. Supervision и Error Handling

### 4.1 Supervision Strategies

```
Supervision Hierarchy:

        Guardian (/)
             |
       User Guardian (/user)
        /    |    \
       /     |     \
   Parent1 Parent2 Parent3
    /   \    |
   /     \   |
Child1  Child2  Child3
```

#### Classic Supervision

```scala
import akka.actor.SupervisorStrategy._
import akka.actor.OneForOneStrategy

class SupervisorActor extends Actor {
  
  override val supervisorStrategy: SupervisorStrategy =
    OneForOneStrategy(
      maxNrOfRetries = 3,
      withinTimeRange = 1.minute,
      loggingEnabled = true
    ) {
      case _: ArithmeticException =>
        println("Resume on ArithmeticException")
        Resume  // Продолжить с текущим состоянием
      
      case _: NullPointerException =>
        println("Restart on NullPointerException")
        Restart  // Перезапустить с начальным состоянием
      
      case _: IllegalArgumentException =>
        println("Stop on IllegalArgumentException")
        Stop  // Остановить актор
      
      case _: Exception =>
        println("Escalate on other Exceptions")
        Escalate  // Передать наверх по иерархии
    }
  
  def receive: Receive = {
    case props: Props =>
      sender() ! context.actorOf(props)
  }
}

// Child actor
class WorkerActor extends Actor {
  private var state: Int = 0
  
  override def preRestart(reason: Throwable, message: Option[Any]): Unit = {
    println(s"Worker restarting. Current state: $state")
    super.preRestart(reason, message)
  }
  
  override def postRestart(reason: Throwable): Unit = {
    println(s"Worker restarted. State reset to: $state")
    super.postRestart(reason)
  }
  
  def receive: Receive = {
    case "increment" =>
      state += 1
      sender() ! state
    
    case "divide" =>
      val result = 10 / 0  // ArithmeticException -> Resume
      sender() ! result
    
    case "null" =>
      val s: String = null
      s.length  // NullPointerException -> Restart
    
    case "illegal" =>
      throw new IllegalArgumentException()  // -> Stop
    
    case "other" =>
      throw new Exception("Unknown error")  // -> Escalate
  }
}
```

#### All-For-One Strategy

```scala
class AllForOneSupervisor extends Actor {
  
  // Все дочерние акторы получают одинаковую директиву
  override val supervisorStrategy: SupervisorStrategy =
    AllForOneStrategy(maxNrOfRetries = 3, withinTimeRange = 1.minute) {
      case _: Exception => Restart
    }
  
  val child1 = context.actorOf(Props[WorkerActor], "child1")
  val child2 = context.actorOf(Props[WorkerActor], "child2")
  val child3 = context.actorOf(Props[WorkerActor], "child3")
  
  def receive: Receive = {
    case msg => child1 ! msg  // Если child1 крашится, все перезапускаются
  }
}
```

### 4.2 Typed Supervision

```scala
import akka.actor.typed.SupervisorStrategy

object TypedWorker {
  sealed trait Command
  case class DoWork(task: String) extends Command
  
  def apply(): Behavior[Command] = {
    Behaviors.supervise {
      Behaviors.receiveMessage {
        case DoWork(task) =>
          if (task == "crash") {
            throw new RuntimeException("Simulated crash")
          }
          println(s"Processing: $task")
          Behaviors.same
      }
    }.onFailure[RuntimeException](
      SupervisorStrategy.restart
        .withLimit(maxNrOfRetries = 3, withinTimeRange = 1.minute)
    )
  }
}

// Различные стратегии
object SupervisionStrategies {
  
  // Restart with exponential backoff
  def withBackoff(): Behavior[Command] = {
    Behaviors.supervise {
      basicBehavior
    }.onFailure[Exception](
      SupervisorStrategy.restartWithBackoff(
        minBackoff = 1.second,
        maxBackoff = 30.seconds,
        randomFactor = 0.2
      ).withMaxRestarts(10)
    )
  }
  
  // Resume - продолжить без перезапуска
  def withResume(): Behavior[Command] = {
    Behaviors.supervise {
      basicBehavior
    }.onFailure[IllegalArgumentException](
      SupervisorStrategy.resume
    )
  }
  
  // Stop - остановить актор
  def withStop(): Behavior[Command] = {
    Behaviors.supervise {
      basicBehavior
    }.onFailure[FatalException](
      SupervisorStrategy.stop
    )
  }
  
  // Multiple strategies
  def withMultiple(): Behavior[Command] = {
    Behaviors.supervise {
      Behaviors.supervise {
        basicBehavior
      }.onFailure[IllegalArgumentException](
        SupervisorStrategy.resume
      )
    }.onFailure[Exception](
      SupervisorStrategy.restart
    )
  }
}
```

### 4.3 Death Watch

```scala
// Classic
class WatcherActor extends Actor {
  val child = context.actorOf(Props[ChildActor], "child")
  
  // Начать следить за актором
  context.watch(child)
  
  def receive: Receive = {
    case Terminated(actorRef) =>
      println(s"Actor ${actorRef.path} terminated")
      // Создать нового child
      val newChild = context.actorOf(Props[ChildActor], "child-2")
      context.watch(newChild)
  }
}

// Typed
object TypedWatcher {
  sealed trait Command
  private case class ChildTerminated(ref: ActorRef[Nothing]) extends Command
  
  def apply(): Behavior[Command] =
    Behaviors.setup { context =>
      val child = context.spawn(childBehavior, "child")
      
      // Watch с адаптером
      context.watchWith(child, ChildTerminated(child))
      
      Behaviors.receiveMessage {
        case ChildTerminated(ref) =>
          context.log.info(s"Child ${ref.path.name} terminated")
          Behaviors.same
      }
    }
}
```

### 4.4 BackoffSupervisor

```scala
import akka.pattern.BackoffSupervisor
import akka.pattern.BackoffOpts

// Classic backoff supervisor
val childProps = Props[DatabaseActor]

val backoffProps = BackoffSupervisor.props(
  BackoffOpts.onFailure(
    childProps = childProps,
    childName = "db-actor",
    minBackoff = 3.seconds,
    maxBackoff = 30.seconds,
    randomFactor = 0.2
  ).withAutoReset(10.seconds)
   .withSupervisorStrategy(
     OneForOneStrategy() {
       case _: SQLException => SupervisorStrategy.Restart
     }
   )
)

val backoffSupervisor = system.actorOf(backoffProps, "db-supervisor")

// Typed version (встроено в SupervisorStrategy.restartWithBackoff)
```

---

## 5. Akka Streams

### 5.1 Основы Streams

```scala
import akka.stream._
import akka.stream.scaladsl._
import akka.actor.ActorSystem

implicit val system: ActorSystem = ActorSystem("StreamSystem")
implicit val materializer: Materializer = Materializer(system)

// Source - источник данных
val source1: Source[Int, NotUsed] = Source(1 to 10)
val source2: Source[Int, NotUsed] = Source.single(42)
val source3: Source[Int, NotUsed] = Source.repeat(1)
val source4: Source[Int, NotUsed] = Source.tick(
  initialDelay = 0.seconds,
  interval = 1.second,
  tick = 1
)

// Flow - трансформация
val doubleFlow: Flow[Int, Int, NotUsed] = Flow[Int].map(_ * 2)
val filterFlow: Flow[Int, Int, NotUsed] = Flow[Int].filter(_ % 2 == 0)
val mapFlow: Flow[Int, String, NotUsed] = Flow[Int].map(_.toString)

// Sink - приемник данных
val printSink: Sink[Any, Future[Done]] = Sink.foreach(println)
val seqSink: Sink[Int, Future[Seq[Int]]] = Sink.seq[Int]
val headSink: Sink[Int, Future[Int]] = Sink.head[Int]
val ignoreSink: Sink[Any, Future[Done]] = Sink.ignore

// Соединение компонентов
val runnable1: RunnableGraph[NotUsed] = 
  source1.via(doubleFlow).to(printSink)

val runnable2: RunnableGraph[NotUsed] =
  source1.map(_ * 2).filter(_ % 2 == 0).to(printSink)

// Запуск
runnable1.run()

// Материализация значения
val future: Future[Seq[Int]] = source1
  .map(_ * 2)
  .filter(_ < 10)
  .runWith(seqSink)

import system.dispatcher
future.foreach(seq => println(s"Result: $seq"))
```

### 5.2 Advanced Stream Operations

```scala
// Группировка
val grouped: Source[Seq[Int], NotUsed] = 
  Source(1 to 100).grouped(10)

// Sliding window
val sliding: Source[Seq[Int], NotUsed] =
  Source(1 to 10).sliding(3, step = 1)
// Результат: Seq(1,2,3), Seq(2,3,4), Seq(3,4,5), ...

// Fold
val sum: Future[Int] = Source(1 to 100)
  .runFold(0)(_ + _)

// Scan (аккумулирующая map)
val accumulated: Source[Int, NotUsed] = Source(1 to 5)
  .scan(0)(_ + _)
// Результат: 0, 1, 3, 6, 10, 15

// Concat
val concat: Source[Int, NotUsed] = 
  Source(1 to 5).concat(Source(10 to 15))

// Merge
val merge: Source[Int, NotUsed] = Source.combine(
  Source(1 to 5),
  Source(10 to 15)
)(Merge(_))

// Zip
val zip: Source[(Int, String), NotUsed] = 
  Source(1 to 5).zip(Source(List("a", "b", "c", "d", "e")))
// Результат: (1,"a"), (2,"b"), ...

// Broadcast - раздача одного stream нескольким sink'ам
val graph = RunnableGraph.fromGraph(GraphDSL.create() { implicit builder =>
  import GraphDSL.Implicits._
  
  val source = Source(1 to 10)
  val broadcast = builder.add(Broadcast[Int](2))
  val sink1 = Sink.foreach[Int](x => println(s"Sink1: $x"))
  val sink2 = Sink.foreach[Int](x => println(s"Sink2: $x"))
  
  source ~> broadcast ~> sink1
            broadcast ~> sink2
  
  ClosedShape
})

graph.run()

// Buffer
val buffered: Source[Int, NotUsed] = Source(1 to 100)
  .buffer(10, OverflowStrategy.backpressure)

// Throttle
val throttled: Source[Int, NotUsed] = Source(1 to 100)
  .throttle(10, 1.second)

// MapAsync - параллельная асинхронная обработка
def fetchUser(id: Int): Future[User] = Future {
  Thread.sleep(100)
  User(id, s"User-$id")
}

val users: Source[User, NotUsed] = Source(1 to 100)
  .mapAsync(parallelism = 10)(fetchUser)

// MapAsyncUnordered - быстрее, но без гарантии порядка
val usersUnordered: Source[User, NotUsed] = Source(1 to 100)
  .mapAsyncUnordered(parallelism = 10)(fetchUser)
```

### 5.3 Backpressure

```scala
// Backpressure - автоматическая регуляция скорости

// Быстрый producer, медленный consumer
val fastProducer: Source[Int, NotUsed] = Source(1 to 1000000)

val slowConsumer: Sink[Int, Future[Done]] = 
  Flow[Int]
    .throttle(1, 100.millis)  // медленная обработка
    .to(Sink.foreach(println))

// Akka Streams автоматически применит backpressure
fastProducer.runWith(slowConsumer)

// Явные стратегии overflow
val withBuffer = Source(1 to 1000000)
  .buffer(100, OverflowStrategy.dropHead)  // удалить старые
  .buffer(100, OverflowStrategy.dropTail)  // удалить новые
  .buffer(100, OverflowStrategy.dropBuffer) // очистить весь буфер
  .buffer(100, OverflowStrategy.dropNew)   // удалить новое сообщение
  .buffer(100, OverflowStrategy.backpressure) // применить backpressure
  .buffer(100, OverflowStrategy.fail)      // завершить с ошибкой

// Пример: Stream с контролем скорости
def rateControlledStream[T](
  source: Source[T, NotUsed],
  maxRate: Int,
  per: FiniteDuration
): Source[T, NotUsed] = {
  source
    .buffer(maxRate * 2, OverflowStrategy.backpressure)
    .throttle(maxRate, per, maxRate, ThrottleMode.Shaping)
}
```

### 5.4 Graph DSL

```scala
import akka.stream.ClosedShape

// Комплексный граф обработки
val complexGraph = RunnableGraph.fromGraph(
  GraphDSL.create() { implicit builder =>
    import GraphDSL.Implicits._
    
    // Компоненты
    val source = Source(1 to 100)
    val broadcast = builder.add(Broadcast[Int](3))
    val merge = builder.add(Merge[Int](2))
    val balance = builder.add(Balance[Int](2))
    val zip = builder.add(Zip[Int, Int]())
    val sink = Sink.foreach[Any](println)
    
    // Граф связей
    source ~> broadcast
              broadcast ~> Flow[Int].map(_ * 2) ~> merge
              broadcast ~> Flow[Int].map(_ * 3) ~> merge
              broadcast ~> Flow[Int].filter(_ % 2 == 0) ~> balance
                                                           balance ~> zip.in0
              merge ~> balance
                      balance ~> zip.in1
              
    zip.out ~> sink
    
    ClosedShape
  }
)

complexGraph.run()

// Partial Graph - переиспользуемый компонент
val partialGraph = GraphDSL.create() { implicit builder =>
  import GraphDSL.Implicits._
  
  val broadcast = builder.add(Broadcast[Int](2))
  val merge = builder.add(Merge[Int](2))
  
  broadcast ~> Flow[Int].map(_ * 2) ~> merge
  broadcast ~> Flow[Int].map(_ * 3) ~> merge
  
  FlowShape(broadcast.in, merge.out)
}

// Использование partial graph
Source(1 to 10)
  .via(partialGraph)
  .runWith(Sink.foreach(println))

// Custom Graph Stage
class DuplicateStage[T] extends GraphStage[FlowShape[T, T]] {
  val in: Inlet[T] = Inlet("Duplicate.in")
  val out: Outlet[T] = Outlet("Duplicate.out")
  
  override val shape: FlowShape[T, T] = FlowShape(in, out)
  
  override def createLogic(inheritedAttributes: Attributes): GraphStageLogic =
    new GraphStageLogic(shape) {
      setHandler(in, new InHandler {
        override def onPush(): Unit = {
          val element = grab(in)
          emit(out, element)
          emit(out, element)  // дублируем элемент
        }
      })
      
      setHandler(out, new OutHandler {
        override def onPull(): Unit = {
          pull(in)
        }
      })
    }
}

// Использование
Source(1 to 5)
  .via(new DuplicateStage[Int])
  .runWith(Sink.foreach(println))
// Результат: 1, 1, 2, 2, 3, 3, 4, 4, 5, 5
```

### 5.5 Error Handling в Streams

```scala
// 1. Recover
val recovered: Source[Int, NotUsed] = Source(1 to 10)
  .map { n =>
    if (n == 5) throw new Exception("Error at 5")
    n
  }
  .recover {
    case ex: Exception => -1
  }

// 2. RecoverWithRetries
val recoveredWithRetries: Source[Int, NotUsed] = Source(1 to 10)
  .map { n =>
    if (n == 5) throw new Exception("Error at 5")
    n
  }
  .recoverWithRetries(
    attempts = 3,
    {
      case ex: Exception => Source.single(0)
    }
  )

// 3. Supervision Strategy
val decider: Supervision.Decider = {
  case _: IllegalArgumentException => Supervision.Resume
  case _: NullPointerException => Supervision.Restart
  case _ => Supervision.Stop
}

val supervised: Source[Int, NotUsed] = Source(1 to 10)
  .map { n =>
    if (n == 5) throw new IllegalArgumentException()
    n
  }
  .withAttributes(ActorAttributes.supervisionStrategy(decider))

// 4. RestartSource - автоматический перезапуск
val restartSource = RestartSource.withBackoff(
  minBackoff = 3.seconds,
  maxBackoff = 30.seconds,
  randomFactor = 0.2,
  maxRestarts = 10
) { () =>
  Source(1 to 10).map { n =>
    if (n == 5) throw new Exception("Temporary failure")
    n
  }
}
```

### 5.6 Integration with Actors

```scala
// Source from Actor
val actorSource: Source[String, ActorRef] = 
  Source.actorRef[String](
    completionMatcher = {
      case "complete" => CompletionStrategy.immediately
    },
    failureMatcher = PartialFunction.empty,
    bufferSize = 100,
    overflowStrategy = OverflowStrategy.dropHead
  )

val (actorRef, source) = actorSource.preMaterialize()

source.runWith(Sink.foreach(println))

actorRef ! "Hello"
actorRef ! "World"
actorRef ! "complete"

// Sink to Actor
case class StreamElement(data: String)
case object StreamCompleted
case class StreamFailure(ex: Throwable)

class ReceiverActor extends Actor {
  def receive: Receive = {
    case StreamElement(data) =>
      println(s"Received: $data")
    
    case StreamCompleted =>
      println("Stream completed")
    
    case StreamFailure(ex) =>
      println(s"Stream failed: ${ex.getMessage}")
  }
}

val receiver = system.actorOf(Props[ReceiverActor])

val actorSink: Sink[String, NotUsed] = Sink.actorRef[String](
  ref = receiver,
  onCompleteMessage = StreamCompleted,
  onFailureMessage = (ex: Throwable) => StreamFailure(ex)
)

Source(List("a", "b", "c"))
  .map(StreamElement)
  .runWith(actorSink)

// Ask pattern в Stream
val askFlow: Flow[Int, String, NotUsed] = Flow[Int].mapAsync(4) { n =>
  import akka.pattern.ask
  implicit val timeout: Timeout = 3.seconds
  
  (someActor ? n).mapTo[String]
}
```

---

## 6. Akka HTTP

### 6.1 Routing DSL

```scala
import akka.http.scaladsl.Http
import akka.http.scaladsl.server.Directives._
import akka.http.scaladsl.server.Route
import akka.http.scaladsl.model.StatusCodes

case class User(id: Long, name: String, email: String)

object UserRoutes {
  
  // Простые маршруты
  val simpleRoutes: Route =
    path("hello") {
      get {
        complete("Hello, World!")
      }
    }
  
  // Путь с параметрами
  val paramRoutes: Route =
    path("users" / LongNumber) { userId =>
      get {
        // GET /users/123
        complete(s"Getting user: $userId")
      }
    }
  
  // Query параметры
  val queryRoutes: Route =
    path("search") {
      get {
        parameters("query", "limit".as[Int] ? 10) { (query, limit) =>
          // GET /search?query=test&limit=20
          complete(s"Searching for: $query, limit: $limit")
        }
      }
    }
  
  // Комбинирование маршрутов
  val routes: Route =
    pathPrefix("api" / "v1") {
      concat(
        // GET /api/v1/users
        path("users") {
          get {
            complete("List of users")
          } ~
          post {
            complete("Create user")
          }
        },
        // GET /api/v1/users/:id
        path("users" / LongNumber) { userId =>
          get {
            complete(s"Get user $userId")
          } ~
          put {
            complete(s"Update user $userId")
          } ~
          delete {
            complete(s"Delete user $userId")
          }
        },
        // GET /api/v1/users/:id/orders
        path("users" / LongNumber / "orders") { userId =>
          get {
            complete(s"Orders for user $userId")
          }
        }
      )
    }
  
  // Запуск сервера
  def startServer()(implicit system: ActorSystem): Future[Http.ServerBinding] = {
    implicit val executionContext = system.dispatcher
    
    Http().newServerAt("localhost", 8080).bind(routes)
  }
}
```

### 6.2 JSON Support

```scala
import spray.json._
import akka.http.scaladsl.marshallers.sprayjson.SprayJsonSupport._

// JSON Protocol
trait JsonProtocol extends DefaultJsonProtocol {
  implicit val userFormat: RootJsonFormat[User] = jsonFormat3(User)
  implicit val createUserRequestFormat: RootJsonFormat[CreateUserRequest] = 
    jsonFormat2(CreateUserRequest)
}

case class CreateUserRequest(name: String, email: String)

object JsonRoutes extends JsonProtocol {
  
  val routes: Route =
    pathPrefix("api") {
      concat(
        // POST /api/users with JSON body
        path("users") {
          post {
            entity(as[CreateUserRequest]) { request =>
              val user = User(
                id = Random.nextLong(),
                name = request.name,
                email = request.email
              )
              complete(StatusCodes.Created, user)
            }
          }
        },
        // GET /api/users/:id returns JSON
        path("users" / LongNumber) { userId =>
          get {
            val user = User(userId, "John", "john@example.com")
            complete(user)
          }
        }
      )
    }
}

// Circe support (альтернатива Spray JSON)
import io.circe.generic.auto._
import de.heikoseeberger.akkahttpcirce.FailFastCirceSupport._

object CirceRoutes {
  
  val routes: Route =
    path("users") {
      post {
        entity(as[CreateUserRequest]) { request =>
          val user = User(Random.nextLong(), request.name, request.email)
          complete(StatusCodes.Created, user)
        }
      }
    }
}
```

### 6.3 Directives

```scala
// Custom directive
def requireAuthentication: Directive1[String] =
  headerValueByName("Authorization").flatMap { token =>
    if (validateToken(token)) {
      provide(extractUserId(token))
    } else {
      complete(StatusCodes.Unauthorized, "Invalid token")
    }
  }

def validateToken(token: String): Boolean = {
  // Validation logic
  token.startsWith("Bearer ")
}

def extractUserId(token: String): String = {
  token.replace("Bearer ", "")
}

// Использование
val protectedRoutes: Route =
  requireAuthentication { userId =>
    path("profile") {
      get {
        complete(s"Profile for user: $userId")
      }
    }
  }

// Conditional directives
val conditionalRoutes: Route =
  path("resource") {
    get {
      conditional(eTag => eTag == "abc123") {
        complete("Resource content")
      }
    }
  }

// Exception handling
val exceptionHandler = ExceptionHandler {
  case ex: IllegalArgumentException =>
    complete(StatusCodes.BadRequest, ex.getMessage)
  
  case ex: NoSuchElementException =>
    complete(StatusCodes.NotFound, "Resource not found")
  
  case ex: Exception =>
    complete(StatusCodes.InternalServerError, "Internal error")
}

val routesWithExceptionHandling: Route =
  handleExceptions(exceptionHandler) {
    path("test") {
      get {
        throw new IllegalArgumentException("Bad input")
      }
    }
  }

// Rejection handling
val rejectionHandler = RejectionHandler.newBuilder()
  .handle {
    case MissingQueryParamRejection(paramName) =>
      complete(StatusCodes.BadRequest, s"Missing parameter: $paramName")
  }
  .handle {
    case AuthorizationFailedRejection =>
      complete(StatusCodes.Forbidden, "Not authorized")
  }
  .handleNotFound {
    complete(StatusCodes.NotFound, "Route not found")
  }
  .result()

val routesWithRejectionHandling: Route =
  handleRejections(rejectionHandler) {
    path("test") {
      get {
        parameter("required") { value =>
          complete(s"Value: $value")
        }
      }
    }
  }
```

### 6.4 File Upload/Download

```scala
// File upload
val uploadRoutes: Route =
  path("upload") {
    post {
      fileUpload("file") {
        case (metadata, byteSource) =>
          val fileName = metadata.fileName
          val destination = Paths.get(s"/tmp/$fileName")
          
          val uploadedFuture: Future[IOResult] = 
            byteSource.runWith(FileIO.toPath(destination))
          
          onSuccess(uploadedFuture) { result =>
            complete(s"Uploaded ${result.count} bytes to $fileName")
          }
      }
    }
  }

// File download
val downloadRoutes: Route =
  path("download" / Segment) { fileName =>
    get {
      val file = Paths.get(s"/tmp/$fileName")
      
      if (Files.exists(file)) {
        val source = FileIO.fromPath(file)
        complete(
          HttpEntity(
            ContentTypes.`application/octet-stream`,
            source
          )
        )
      } else {
        complete(StatusCodes.NotFound)
      }
    }
  }

// Multipart form data
val multipartRoutes: Route =
  path("form") {
    post {
      entity(as[Multipart.FormData]) { formData =>
        val parts: Source[Multipart.FormData.BodyPart, Any] = formData.parts
        
        val processed: Future[Map[String, String]] = parts
          .mapAsync(1) { part =>
            part.entity.dataBytes
              .runFold(ByteString.empty)(_ ++ _)
              .map(bs => part.name -> bs.utf8String)
          }
          .runFold(Map.empty[String, String])(_ + _)
        
        onSuccess(processed) { fields =>
          complete(s"Received fields: $fields")
        }
      }
    }
  }
```

### 6.5 WebSockets

```scala
import akka.http.scaladsl.model.ws._

// Simple echo WebSocket
val echoWebSocket: Route =
  path("ws" / "echo") {
    get {
      handleWebSocketMessages(
        Flow[Message].mapConcat {
          case TextMessage.Strict(text) =>
            TextMessage(s"Echo: $text") :: Nil
          case _ =>
            Nil
        }
      )
    }
  }

// Chat room WebSocket
object ChatRoom {
  case class Join(name: String)
  case class Message(author: String, text: String)
  case class Leave(name: String)
  
  def chatFlow(name: String): Flow[Message, Message, NotUsed] = {
    // Simplified - in production use actor-based implementation
    Flow[Message].map { msg =>
      TextMessage(s"[$name]: $msg")
    }
  }
}

val chatWebSocket: Route =
  path("ws" / "chat") {
    parameter("name") { name =>
      handleWebSocketMessages(
        Flow[Message]
          .collect {
            case TextMessage.Strict(text) => text
          }
          .via(ChatRoom.chatFlow(name))
      )
    }
  }

// Actor-based WebSocket
class WebSocketActor extends Actor {
  def receive: Receive = {
    case TextMessage.Strict(text) =>
      sender() ! TextMessage(s"Received: $text")
  }
}

val actorBasedWebSocket: Route =
  path("ws" / "actor") {
    get {
      extractActorSystem { implicit system =>
        val handler = system.actorOf(Props[WebSocketActor])
        
        handleWebSocketMessages(
          Flow[Message]
            .collect { case tm: TextMessage => tm }
            .mapAsync(1) { msg =>
              import akka.pattern.ask
              implicit val timeout: Timeout = 3.seconds
              (handler ? msg).mapTo[TextMessage]
            }
        )
      }
    }
  }
```

### 6.6 HTTP Client

```scala
import akka.http.scaladsl.Http
import akka.http.scaladsl.model._

object HttpClientExample {
  
  // Simple GET request
  def simpleGet()(implicit system: ActorSystem): Future[String] = {
    implicit val ec = system.dispatcher
    
    val request = HttpRequest(uri = "https://api.github.com/users/octocat")
    
    Http().singleRequest(request).flatMap { response =>
      response.entity.dataBytes
        .runFold(ByteString.empty)(_ ++ _)
        .map(_.utf8String)
    }
  }
  
  // POST with JSON
  def postJson(user: User)(implicit system: ActorSystem): Future[HttpResponse] = {
    import spray.json._
    
    val json = user.toJson.compactPrint
    
    val request = HttpRequest(
      method = HttpMethods.POST,
      uri = "https://api.example.com/users",
      entity = HttpEntity(ContentTypes.`application/json`, json)
    )
    
    Http().singleRequest(request)
  }
  
  // Connection pool
  val connectionPool: Flow[
    (HttpRequest, Int),
    (Try[HttpResponse], Int),
    Http.HostConnectionPool
  ] = Http().cachedHostConnectionPool[Int]("api.example.com")
  
  def batchRequests(ids: List[Int])(implicit system: ActorSystem): Future[List[String]] = {
    implicit val ec = system.dispatcher
    
    Source(ids)
      .map(id => (HttpRequest(uri = s"/users/$id"), id))
      .via(connectionPool)
      .mapAsync(4) {
        case (Success(response), id) =>
          response.entity.dataBytes
            .runFold(ByteString.empty)(_ ++ _)
            .map(_.utf8String)
        case (Failure(ex), id) =>
          Future.successful(s"Request failed: ${ex.getMessage}")
      }
      .runWith(Sink.seq)
      .map(_.toList)
  }
}
```

---

## 7. Akka Persistence

### 7.1 Event Sourcing Basics

```scala
import akka.persistence._

// Events
sealed trait AccountEvent
case class AccountCreated(accountId: String, initialBalance: BigDecimal) extends AccountEvent
case class MoneyDeposited(amount: BigDecimal) extends AccountEvent
case class MoneyWithdrawn(amount: BigDecimal) extends AccountEvent

// Commands
sealed trait AccountCommand
case class CreateAccount(accountId: String, initialBalance: BigDecimal) extends AccountCommand
case class Deposit(amount: BigDecimal) extends AccountCommand
case class Withdraw(amount: BigDecimal) extends AccountCommand
case object GetBalance extends AccountCommand

// State
case class AccountState(accountId: String, balance: BigDecimal) {
  def applyEvent(event: AccountEvent): AccountState = event match {
    case AccountCreated(id, initial) => copy(accountId = id, balance = initial)
    case MoneyDeposited(amount) => copy(balance = balance + amount)
    case MoneyWithdrawn(amount) => copy(balance = balance - amount)
  }
}

// PersistentActor
class BankAccountActor(accountId: String) extends PersistentActor {
  
  override def persistenceId: String = s"account-$accountId"
  
  private var state = AccountState(accountId, BigDecimal(0))
  
  override def receiveCommand: Receive = {
    case CreateAccount(id, initial) if initial >= 0 =>
      persist(AccountCreated(id, initial)) { event =>
        state = state.applyEvent(event)
        sender() ! s"Account created with balance: ${state.balance}"
      }
    
    case Deposit(amount) if amount > 0 =>
      persist(MoneyDeposited(amount)) { event =>
        state = state.applyEvent(event)
        sender() ! s"Deposited $amount. New balance: ${state.balance}"
      }
    
    case Withdraw(amount) if amount > 0 && amount <= state.balance =>
      persist(MoneyWithdrawn(amount)) { event =>
        state = state.applyEvent(event)
        sender() ! s"Withdrawn $amount. New balance: ${state.balance}"
      }
    
    case Withdraw(amount) =>
      sender() ! s"Insufficient funds. Balance: ${state.balance}"
    
    case GetBalance =>
      sender() ! state.balance
  }
  
  override def receiveRecover: Receive = {
    case event: AccountEvent =>
      state = state.applyEvent(event)
    
    case SnapshotOffer(_, snapshot: AccountState) =>
      state = snapshot
    
    case RecoveryCompleted =>
      println(s"Recovery completed. Current balance: ${state.balance}")
  }
}
```

### 7.2 Snapshots

```scala
class SnapshotBankAccount(accountId: String) extends PersistentActor {
  
  override def persistenceId: String = s"account-$accountId"
  
  private var state = AccountState(accountId, BigDecimal(0))
  private var eventsSinceLastSnapshot = 0
  private val snapshotInterval = 10
  
  override def receiveCommand: Receive = {
    case cmd: Deposit =>
      persist(MoneyDeposited(cmd.amount)) { event =>
        updateState(event)
        maybeSnapshot()
        sender() ! s"Deposited. Balance: ${state.balance}"
      }
    
    case cmd: Withdraw if cmd.amount <= state.balance =>
      persist(MoneyWithdrawn(cmd.amount)) { event =>
        updateState(event)
        maybeSnapshot()
        sender() ! s"Withdrawn. Balance: ${state.balance}"
      }
    
    case GetBalance =>
      sender() ! state.balance
    
    case SaveSnapshotSuccess(metadata) =>
      println(s"Snapshot saved: ${metadata.sequenceNr}")
      // Удаляем старые snapshot'ы
      deleteSnapshots(SnapshotSelectionCriteria(
        maxSequenceNr = metadata.sequenceNr - 2
      ))
    
    case SaveSnapshotFailure(metadata, reason) =>
      println(s"Snapshot failed: ${reason.getMessage}")
  }
  
  override def receiveRecover: Receive = {
    case event: AccountEvent =>
      updateState(event)
    
    case SnapshotOffer(metadata, snapshot: AccountState) =>
      println(s"Recovering from snapshot: ${metadata.sequenceNr}")
      state = snapshot
  }
  
  private def updateState(event: AccountEvent): Unit = {
    state = state.applyEvent(event)
    eventsSinceLastSnapshot += 1
  }
  
  private def maybeSnapshot(): Unit = {
    if (eventsSinceLastSnapshot >= snapshotInterval) {
      saveSnapshot(state)
      eventsSinceLastSnapshot = 0
    }
  }
}
```

### 7.3 Typed Persistence (EventSourcedBehavior)

```scala
import akka.persistence.typed.PersistenceId
import akka.persistence.typed.scaladsl.{Effect, EventSourcedBehavior}

object TypedBankAccount {
  
  // Commands
  sealed trait Command
  final case class CreateAccount(initialBalance: BigDecimal, replyTo: ActorRef[Response]) extends Command
  final case class Deposit(amount: BigDecimal, replyTo: ActorRef[Response]) extends Command
  final case class Withdraw(amount: BigDecimal, replyTo: ActorRef[Response]) extends Command
  final case class GetBalance(replyTo: ActorRef[BigDecimal]) extends Command
  
  // Events
  sealed trait Event
  final case class AccountCreated(initialBalance: BigDecimal) extends Event
  final case class Deposited(amount: BigDecimal) extends Event
  final case class Withdrawn(amount: BigDecimal) extends Event
  
  // Responses
  sealed trait Response
  final case class AccountCreatedResponse(balance: BigDecimal) extends Response
  final case class BalanceUpdated(balance: BigDecimal) extends Response
  final case class InsufficientFunds(requested: BigDecimal, available: BigDecimal) extends Response
  
  // State
  final case class State(balance: BigDecimal) {
    def applyEvent(event: Event): State = event match {
      case AccountCreated(initial) => copy(balance = initial)
      case Deposited(amount) => copy(balance = balance + amount)
      case Withdrawn(amount) => copy(balance = balance - amount)
    }
  }
  
  object State {
    val empty: State = State(BigDecimal(0))
  }
  
  def apply(accountId: String): Behavior[Command] = {
    EventSourcedBehavior[Command, Event, State](
      persistenceId = PersistenceId.ofUniqueId(accountId),
      emptyState = State.empty,
      commandHandler = commandHandler,
      eventHandler = eventHandler
    )
  }
  
  private def commandHandler: (State, Command) => Effect[Event, State] = {
    (state, command) => command match {
      case CreateAccount(initial, replyTo) if initial >= 0 =>
        Effect.persist(AccountCreated(initial))
          .thenReply(replyTo)(s => AccountCreatedResponse(s.balance))
      
      case Deposit(amount, replyTo) if amount > 0 =>
        Effect.persist(Deposited(amount))
          .thenReply(replyTo)(s => BalanceUpdated(s.balance))
      
      case Withdraw(amount, replyTo) if amount > 0 =>
        if (amount <= state.balance) {
          Effect.persist(Withdrawn(amount))
            .thenReply(replyTo)(s => BalanceUpdated(s.balance))
        } else {
          Effect.reply(replyTo)(InsufficientFunds(amount, state.balance))
        }
      
      case GetBalance(replyTo) =>
        Effect.reply(replyTo)(state.balance)
      
      case _ =>
        Effect.unhandled
    }
  }
  
  private def eventHandler: (State, Event) => State = {
    (state, event) => state.applyEvent(event)
  }
}
```

### 7.4 Event Adapters

```scala
import akka.persistence.journal.{EventAdapter, EventSeq}

// V1 Event
case class UserCreatedV1(username: String)

// V2 Event (новая версия с email)
case class UserCreatedV2(username: String, email: String)

// Event Adapter
class UserEventAdapter extends EventAdapter {
  
  override def manifest(event: Any): String = event.getClass.getName
  
  override def toJournal(event: Any): Any = event match {
    case evt: UserCreatedV2 => evt
    case _ => event
  }
  
  override def fromJournal(event: Any, manifest: String): EventSeq = event match {
    // Миграция старых событий
    case UserCreatedV1(username) =>
      EventSeq.single(UserCreatedV2(username, s"$username@example.com"))
    
    case evt: UserCreatedV2 =>
      EventSeq.single(evt)
    
    case other =>
      EventSeq.single(other)
  }
}

// Configuration in application.conf
/*
akka.persistence.journal {
  plugin = "akka.persistence.journal.leveldb"
  leveldb.dir = "target/journal"
  
  leveldb.event-adapters {
    user-adapter = "com.example.UserEventAdapter"
  }
  
  leveldb.event-adapter-bindings {
    "com.example.UserCreatedV1" = user-adapter
    "com.example.UserCreatedV2" = user-adapter
  }
}
*/
```

### 7.5 Persistence Query

```scala
import akka.persistence.query._
import akka.persistence.query.journal.leveldb.scaladsl.LeveldbReadJournal
import akka.stream.scaladsl.Source

object PersistenceQueryExample {
  
  def queryEvents()(implicit system: ActorSystem): Unit = {
    implicit val mat = Materializer(system)
    
    val queries = PersistenceQuery(system)
      .readJournalFor[LeveldbReadJournal](
        LeveldbReadJournal.Identifier
      )
    
    // All persistence IDs
    val allPersistenceIds: Source[String, NotUsed] = 
      queries.currentPersistenceIds()
    
    allPersistenceIds.runForeach { persistenceId =>
      println(s"Found persistence ID: $persistenceId")
    }
    
    // Events for specific persistence ID
    val events: Source[EventEnvelope, NotUsed] =
      queries.currentEventsByPersistenceId("account-123", 0L, Long.MaxValue)
    
    events.runForeach { envelope =>
      println(s"Event: ${envelope.event} at sequence ${envelope.sequenceNr}")
    }
    
    // Live stream of events
    val liveEvents: Source[EventEnvelope, NotUsed] =
      queries.eventsByPersistenceId("account-123", 0L, Long.MaxValue)
    
    // Events by tag
    val taggedEvents: Source[EventEnvelope, NotUsed] =
      queries.eventsByTag("banking", Offset.noOffset)
    
    taggedEvents.runForeach { envelope =>
      println(s"Tagged event: ${envelope.event}")
    }
  }
}
```

---

## 8. Akka Cluster

### 8.1 Cluster Basics

```scala
// application.conf
/*
akka {
  actor {
    provider = cluster
  }
  
  remote.artery {
    canonical {
      hostname = "127.0.0.1"
      port = 2551
    }
  }
  
  cluster {
    seed-nodes = [
      "akka://ClusterSystem@127.0.0.1:2551",
      "akka://ClusterSystem@127.0.0.1:2552"
    ]
    
    downing-provider-class = "akka.cluster.sbr.SplitBrainResolverProvider"
  }
}
*/

import akka.cluster.Cluster
import akka.cluster.ClusterEvent._

class ClusterListener extends Actor with ActorLogging {
  
  val cluster = Cluster(context.system)
  
  override def preStart(): Unit = {
    cluster.subscribe(
      self,
      initialStateMode = InitialStateAsEvents,
      classOf[MemberEvent],
      classOf[UnreachableMember]
    )
  }
  
  override def postStop(): Unit = {
    cluster.unsubscribe(self)
  }
  
  def receive: Receive = {
    case MemberUp(member) =>
      log.info(s"Member is Up: ${member.address}")
    
    case UnreachableMember(member) =>
      log.info(s"Member detected as unreachable: ${member.address}")
    
    case MemberRemoved(member, previousStatus) =>
      log.info(s"Member is Removed: ${member.address} after $previousStatus")
    
    case _: MemberEvent =>
      // ignore
  }
}

// Typed Cluster
object TypedClusterListener {
  sealed trait Event
  private case class ReachabilityChange(reachabilityEvent: ReachabilityEvent) extends Event
  private case class MemberChange(event: MemberEvent) extends Event
  
  def apply(): Behavior[Event] =
    Behaviors.setup { context =>
      val memberEventAdapter = context.messageAdapter[MemberEvent](MemberChange)
      val reachabilityAdapter = context.messageAdapter[ReachabilityEvent](ReachabilityChange)
      
      Cluster(context.system).subscriptions ! Subscribe(memberEventAdapter, classOf[MemberEvent])
      Cluster(context.system).subscriptions ! Subscribe(reachabilityAdapter, classOf[ReachabilityEvent])
      
      Behaviors.receiveMessage {
        case MemberChange(event) =>
          context.log.info(s"Member event: $event")
          Behaviors.same
        
        case ReachabilityChange(event) =>
          context.log.info(s"Reachability event: $event")
          Behaviors.same
      }
    }
}
```

### 8.2 Cluster Singleton

```scala
import akka.cluster.singleton._

// Создание Cluster Singleton
class SingletonActor extends Actor {
  def receive: Receive = {
    case msg => println(s"Singleton received: $msg")
  }
}

object ClusterSingletonExample {
  
  def startSingleton()(implicit system: ActorSystem): ActorRef = {
    ClusterSingleton(system).init(
      SingletonActor(
        Props[SingletonActor],
        "singleton",
        PoisonPill
      ).withStopMessage(PoisonPill)
    )
  }
}

// Typed Cluster Singleton
object TypedSingletonActor {
  sealed trait Command
  case class DoWork(task: String) extends Command
  
  def apply(): Behavior[Command] =
    Behaviors.receive { (context, message) =>
      message match {
        case DoWork(task) =>
          context.log.info(s"Singleton working on: $task")
          Behaviors.same
      }
    }
}

object TypedClusterSingletonExample {
  
  def startSingleton()(implicit system: ActorSystem[_]): ActorRef[TypedSingletonActor.Command] = {
    ClusterSingleton(system).init(
      SingletonActor(TypedSingletonActor(), "typed-singleton")
    )
  }
}
```

### 8.3 Distributed Data

```scala
import akka.cluster.ddata._
import akka.cluster.ddata.Replicator._

class DistributedDataActor extends Actor {
  
  val replicator = DistributedData(context.system).replicator
  implicit val node = Cluster(context.system)
  
  // Counter CRDT
  val counterKey = PNCounterKey("my-counter")
  
  def receive: Receive = {
    case "increment" =>
      replicator ! Update(counterKey, PNCounter.empty, WriteLocal)(_ :+ 1)
    
    case "decrement" =>
      replicator ! Update(counterKey, PNCounter.empty, WriteLocal)(_ :- 1)
    
    case "get" =>
      replicator ! Get(counterKey, ReadLocal)
    
    case g @ GetSuccess(key, _) if key == counterKey =>
      val counter = g.get(counterKey)
      println(s"Counter value: ${counter.value}")
    
    case NotFound(key, _) =>
      println("Counter not found")
    
    // ORSet (Observed-Remove Set)
    case "add" / value: String =>
      val setKey = ORSetKey[String]("my-set")
      replicator ! Update(setKey, ORSet.empty[String], WriteLocal)(_ :+ value)
    
    case "remove" / value: String =>
      val setKey = ORSetKey[String]("my-set")
      replicator ! Update(setKey, ORSet.empty[String], WriteLocal)(_ :- value)
  }
}

// LWWMap (Last-Writer-Wins Map)
object DistributedMapExample {
  
  case class PutValue(key: String, value: String)
  case class GetValue(key: String)
  case class RemoveValue(key: String)
  
  class MapActor extends Actor {
    val replicator = DistributedData(context.system).replicator
    implicit val node = Cluster(context.system)
    
    val mapKey = LWWMapKey[String, String]("my-map")
    
    def receive: Receive = {
      case PutValue(key, value) =>
        replicator ! Update(mapKey, LWWMap.empty[String, String], WriteLocal) { map =>
          map :+ (key -> value)
        }
      
      case GetValue(key) =>
        replicator ! Get(mapKey, ReadLocal, Some(sender()))
      
      case g @ GetSuccess(`mapKey`, Some(replyTo: ActorRef)) =>
        val map = g.get(mapKey)
        replyTo ! map.entries.get(key)
      
      case RemoveValue(key) =>
        replicator ! Update(mapKey, LWWMap.empty[String, String], WriteLocal) { map =>
          map - key
        }
    }
  }
}
```

---

## 9. Akka Sharding

### 9.1 Cluster Sharding Basics

```scala
import akka.cluster.sharding._

// Entity Actor
class ShardedEntity extends Actor {
  override def receive: Receive = {
    case msg => println(s"Entity ${self.path.name} received: $msg")
  }
}

object ShardingExample {
  
  // Message Extractor
  val extractEntityId: ShardRegion.ExtractEntityId = {
    case msg @ EntityMessage(entityId, payload) =>
      (entityId.toString, payload)
  }
  
  val extractShardId: ShardRegion.ExtractShardId = {
    case EntityMessage(entityId, _) =>
      (Math.abs(entityId.hashCode) % 100).toString
  }
  
  case class EntityMessage(entityId: Long, payload: Any)
  
  def startSharding()(implicit system: ActorSystem): ActorRef = {
    ClusterSharding(system).start(
      typeName = "ShardedEntity",
      entityProps = Props[ShardedEntity],
      settings = ClusterShardingSettings(system),
      extractEntityId = extractEntityId,
      extractShardId = extractShardId
    )
  }
  
  // Использование
  def useSharding(shardRegion: ActorRef): Unit = {
    shardRegion ! EntityMessage(123, "Hello")
    shardRegion ! EntityMessage(456, "World")
    shardRegion ! EntityMessage(123, "Again")  // идет к тому же entity
  }
}
```

### 9.2 Typed Sharding

```scala
import akka.cluster.sharding.typed.scaladsl.{ClusterSharding, Entity, EntityTypeKey}

object TypedShardedEntity {
  
  sealed trait Command
  final case class ProcessMessage(message: String, replyTo: ActorRef[Response]) extends Command
  final case class GetState(replyTo: ActorRef[State]) extends Command
  
  final case class Response(message: String)
  final case class State(messages: List[String])
  
  val TypeKey: EntityTypeKey[Command] = EntityTypeKey[Command]("ShardedEntity")
  
  def apply(entityId: String): Behavior[Command] = {
    Behaviors.setup { context =>
      context.log.info(s"Entity $entityId started")
      
      active(State(Nil))
    }
  }
  
  private def active(state: State): Behavior[Command] = {
    Behaviors.receive { (context, message) =>
      message match {
        case ProcessMessage(msg, replyTo) =>
          context.log.info(s"Processing: $msg")
          val newState = state.copy(messages = msg :: state.messages)
          replyTo ! Response(s"Processed: $msg")
          active(newState)
        
        case GetState(replyTo) =>
          replyTo ! state
          Behaviors.same
      }
    }
  }
}

object TypedShardingExample {
  
  def init()(implicit system: ActorSystem[_]): ActorRef[ShardingEnvelope[TypedShardedEntity.Command]] = {
    ClusterSharding(system).init(
      Entity(TypedShardedEntity.TypeKey) { entityContext =>
        TypedShardedEntity(entityContext.entityId)
      }
    )
  }
  
  // Использование
  def useSharding()(implicit system: ActorSystem[_]): Unit = {
    val sharding = ClusterSharding(system)
    
    import TypedShardedEntity._
    
    implicit val timeout: Timeout = 3.seconds
    implicit val scheduler = system.scheduler
    implicit val ec = system.executionContext
    
    // Отправка сообщения
    val entityRef = sharding.entityRefFor(TypeKey, "entity-123")
    
    val future: Future[Response] = entityRef.ask(ref => ProcessMessage("Hello", ref))
    
    future.foreach(response => println(s"Got response: $response"))
  }
}
```

### 9.3 Passivation

```scala
// Автоматическая пассивация неактивных entities
object PassivationExample {
  
  object ShardedCounter {
    sealed trait Command
    case class Increment(replyTo: ActorRef[Int]) extends Command
    case class GetCount(replyTo: ActorRef[Int]) extends Command
    private case object Passivate extends Command
    
    val TypeKey = EntityTypeKey[Command]("Counter")
    
    def apply(entityId: String, shard: ActorRef[ClusterSharding.ShardCommand]): Behavior[Command] = {
      Behaviors.setup { context =>
        // Таймер для пассивации
        Behaviors.withTimers { timers =>
          timers.startSingleTimer(Passivate, 30.seconds)
          
          active(shard, 0)
        }
      }
    }
    
    private def active(
      shard: ActorRef[ClusterSharding.ShardCommand],
      count: Int
    ): Behavior[Command] = {
      Behaviors.receive { (context, message) =>
        message match {
          case Increment(replyTo) =>
            val newCount = count + 1
            replyTo ! newCount
            active(shard, newCount)
          
          case GetCount(replyTo) =>
            replyTo ! count
            Behaviors.same
          
          case Passivate =>
            // Сообщаем шарду, что хотим остановиться
            shard ! ClusterSharding.Passivate(context.self)
            Behaviors.same
        }
      }.receiveSignal {
        case (context, PostStop) =>
          context.log.info(s"Counter passivated with count: $count")
          Behaviors.same
      }
    }
  }
  
  def init()(implicit system: ActorSystem[_]): Unit = {
    ClusterSharding(system).init(
      Entity(ShardedCounter.TypeKey) { entityContext =>
        ShardedCounter(entityContext.entityId, entityContext.shard)
      }.withStopMessage(ShardedCounter.Passivate)
    )
  }
}
```

### 9.4 Remember Entities

```scala
// Remember Entities - автоматический перезапуск entities после сбоя

object RememberEntitiesExample {
  
  def init()(implicit system: ActorSystem[_]): Unit = {
    val settings = ClusterShardingSettings(system)
      .withRememberEntities(true)  // включаем remember entities
    
    ClusterSharding(system).init(
      Entity(TypedShardedEntity.TypeKey) { entityContext =>
        TypedShardedEntity(entityContext.entityId)
      }.withSettings(settings)
    )
  }
}

// Ручное управление remember entities
object ManualRememberExample {
  
  sealed trait Command
  case class StartEntity(entityId: String) extends Command
  case class StopEntity(entityId: String) extends Command
  
  def controller(): Behavior[Command] = {
    Behaviors.setup { context =>
      val sharding = ClusterSharding(context.system)
      
      Behaviors.receiveMessage {
        case StartEntity(entityId) =>
          val entityRef = sharding.entityRefFor(TypedShardedEntity.TypeKey, entityId)
          // Entity будет создан при первом обращении
          Behaviors.same
        
        case StopEntity(entityId) =>
          val entityRef = sharding.entityRefFor(TypedShardedEntity.TypeKey, entityId)
          context.stop(entityRef)
          Behaviors.same
      }
    }
  }
}
```

---

## 10. Testing Akka Applications

### 10.1 Testing Classic Actors

```scala
import akka.testkit._
import org.scalatest.BeforeAndAfterAll
import org.scalatest.wordspec.AnyWordSpecLike
import org.scalatest.matchers.should.Matchers

class EchoActorSpec 
  extends TestKit(ActorSystem("TestSystem"))
  with ImplicitSender
  with AnyWordSpecLike
  with Matchers
  with BeforeAndAfterAll {
  
  override def afterAll(): Unit = {
    TestKit.shutdownActorSystem(system)
  }
  
  "An EchoActor" should {
    "reply with same message" in {
      val echo = system.actorOf(Props[EchoActor])
      echo ! "hello"
      expectMsg("hello")
    }
    
    "reply within 1 second" in {
      val echo = system.actorOf(Props[EchoActor])
      within(1.second) {
        echo ! "test"
        expectMsg("test")
      }
    }
  }
  
  "A CounterActor" should {
    "increment counter" in {
      val counter = system.actorOf(Props[CounterActor])
      counter ! "increment"
      counter ! "get"
      expectMsg(1)
    }
    
    "handle multiple operations" in {
      val counter = system.actorOf(Props[CounterActor])
      counter ! "increment"
      counter ! "increment"
      counter ! "decrement"
      counter ! "get"
      expectMsg(1)
    }
  }
}

// TestProbe
class InteractionSpec extends TestKit(ActorSystem("InteractionSystem"))
  with ImplicitSender
  with AnyWordSpecLike
  with Matchers
  with BeforeAndAfterAll {
  
  "Parent and Child actors" should {
    "communicate correctly" in {
      val childProbe = TestProbe()
      val parent = system.actorOf(Props(new ParentActor {
        override def createChild(): ActorRef = childProbe.ref
      }))
      
      parent ! "work"
      childProbe.expectMsg("do work")
    }
  }
  
  "Actor with multiple collaborators" should {
    "delegate correctly" in {
      val probe1 = TestProbe()
      val probe2 = TestProbe()
      
      val coordinator = system.actorOf(Props(new CoordinatorActor(probe1.ref, probe2.ref)))
      
      coordinator ! "coordinate"
      
      probe1.expectMsg("task1")
      probe2.expectMsg("task2")
    }
  }
}
```

### 10.2 Testing Typed Actors

```scala
import akka.actor.testkit.typed.scaladsl.{ActorTestKit, BehaviorTestKit, TestProbe}
import org.scalatest.BeforeAndAfterAll
import org.scalatest.wordspec.AnyWordSpec

class TypedActorSpec extends AnyWordSpec with BeforeAndAfterAll {
  
  val testKit = ActorTestKit()
  
  override def afterAll(): Unit = testKit.shutdownTestKit()
  
  "A typed counter" should {
    "increment correctly" in {
      val probe = testKit.createTestProbe[Int]()
      val counter = testKit.spawn(CounterActor())
      
      counter ! Increment(probe.ref)
      probe.expectMessage(1)
      
      counter ! Increment(probe.ref)
      probe.expectMessage(2)
    }
    
    "handle get value" in {
      val probe = testKit.createTestProbe[Int]()
      val counter = testKit.spawn(CounterActor())
      
      counter ! GetValue(probe.ref)
      probe.expectMessage(0)
    }
  }
  
  "A shopping cart" should {
    "add items correctly" in {
      val responseProbe = testKit.createTestProbe[ShoppingCart.Response]()
      val itemsProbe = testKit.createTestProbe[ShoppingCart.Items]()
      
      val cart = testKit.spawn(ShoppingCart())
      
      cart ! ShoppingCart.AddItem("apple", responseProbe.ref)
      responseProbe.expectMessage(ShoppingCart.ItemAdded)
      
      cart ! ShoppingCart.GetItems(itemsProbe.ref)
      itemsProbe.expectMessage(ShoppingCart.Items(List("apple")))
    }
  }
}

// BehaviorTestKit - synchronous testing
class SyncTypedActorSpec extends AnyWordSpec {
  
  "Synchronous counter" should {
    "increment" in {
      val testKit = BehaviorTestKit(CounterActor())
      val probe = TestProbe[Int]()
      
      testKit.run(Increment(probe.ref))
      testKit.expectEffect(Effects.messageAdapter[Int])
      
      val result = testKit.returnedBehavior
      // assertions on behavior
    }
  }
}
```

### 10.3 Testing Streams

```scala
import akka.stream.testkit.scaladsl.{TestSink, TestSource}

class StreamSpec extends AnyWordSpec with Matchers {
  
  implicit val system: ActorSystem = ActorSystem("StreamTestSystem")
  implicit val materializer: Materializer = Materializer(system)
  
  "A simple stream" should {
    "transform elements correctly" in {
      val source = Source(1 to 5)
      val result = source.map(_ * 2).runWith(Sink.seq)
      
      import system.dispatcher
      Await.result(result, 3.seconds) shouldBe Seq(2, 4, 6, 8, 10)
    }
  }
  
  "TestSink" should {
    "allow element-by-element assertion" in {
      Source(1 to 5)
        .map(_ * 2)
        .runWith(TestSink.probe[Int])
        .request(5)
        .expectNext(2, 4, 6, 8, 10)
        .expectComplete()
    }
    
    "handle backpressure" in {
      Source(1 to 100)
        .runWith(TestSink.probe[Int])
        .request(2)
        .expectNext(1, 2)
        .request(1)
        .expectNext(3)
        .cancel()
    }
  }
  
  "TestSource" should {
    "allow manual element emission" in {
      val (source, sink) = TestSource.probe[Int]
        .map(_ * 2)
        .toMat(TestSink.probe[Int])(Keep.both)
        .run()
      
      sink.request(2)
      source.sendNext(1)
      sink.expectNext(2)
      
      source.sendNext(2)
      sink.expectNext(4)
      
      source.sendComplete()
      sink.expectComplete()
    }
  }
  
  "Flow under test" should {
    "work correctly" in {
      val flow = Flow[Int].map(_ * 2).filter(_ > 5)
      
      val (source, sink) = TestSource.probe[Int]
        .via(flow)
        .toMat(TestSink.probe[Int])(Keep.both)
        .run()
      
      sink.request(3)
      source.sendNext(1, 2, 3, 4, 5)
      sink.expectNext(6, 8, 10)
    }
  }
}
```

### 10.4 Testing HTTP Routes

```scala
import akka.http.scaladsl.testkit.ScalatestRouteTest

class RouteSpec extends AnyWordSpec with Matchers with ScalatestRouteTest {
  
  val routes = UserRoutes.routes
  
  "User routes" should {
    "return list of users for GET /api/v1/users" in {
      Get("/api/v1/users") ~> routes ~> check {
        status shouldBe StatusCodes.OK
        responseAs[String] should include("users")
      }
    }
    
    "create user for POST /api/v1/users" in {
      val userJson = """{"name":"Alice","email":"alice@example.com"}"""
      
      Post("/api/v1/users", HttpEntity(ContentTypes.`application/json`, userJson)) ~> 
        routes ~> 
        check {
          status shouldBe StatusCodes.Created
          val response = responseAs[User]
          response.name shouldBe "Alice"
        }
    }
    
    "return 404 for non-existent user" in {
      Get("/api/v1/users/999") ~> routes ~> check {
        status shouldBe StatusCodes.NotFound
      }
    }
    
    "handle query parameters" in {
      Get("/api/v1/search?query=test&limit=20") ~> routes ~> check {
        status shouldBe StatusCodes.OK
      }
    }
  }
}
```

### 10.5 Testing Persistence

```scala
import akka.persistence.testkit.scaladsl.EventSourcedBehaviorTestKit

class PersistenceSpec extends AnyWordSpec with BeforeAndAfterAll {
  
  val testKit = ActorTestKit()
  
  override def afterAll(): Unit = testKit.shutdownTestKit()
  
  "BankAccount" should {
    "persist deposit events" in {
      val accountId = "test-account-1"
      val eventSourcedTestKit = EventSourcedBehaviorTestKit[
        TypedBankAccount.Command,
        TypedBankAccount.Event,
        TypedBankAccount.State
      ](testKit.system, TypedBankAccount(accountId))
      
      val result1 = eventSourcedTestKit.runCommand[TypedBankAccount.Response](
        TypedBankAccount.Deposit(100, _)
      )
      
      result1.event shouldBe TypedBankAccount.Deposited(100)
      result1.state.balance shouldBe 100
      
      val result2 = eventSourcedTestKit.runCommand[TypedBankAccount.Response](
        TypedBankAccount.Withdraw(50, _)
      )
      
      result2.event shouldBe TypedBankAccount.Withdrawn(50)
      result2.state.balance shouldBe 50
    }
    
    "recover from events" in {
      val accountId = "test-account-2"
      val eventSourcedTestKit = EventSourcedBehaviorTestKit[
        TypedBankAccount.Command,
        TypedBankAccount.Event,
        TypedBankAccount.State
      ](testKit.system, TypedBankAccount(accountId))
      
      eventSourcedTestKit.runCommand[TypedBankAccount.Response](
        TypedBankAccount.Deposit(100, _)
      )
      
      eventSourcedTestKit.restart()
      
      val result = eventSourcedTestKit.runCommand[BigDecimal](
        TypedBankAccount.GetBalance
      )
      
      result.state.balance shouldBe 100
    }
  }
}
```

---

## 11. Performance и Best Practices

### 11.1 Performance Tuning

```scala
// Dispatcher configuration
/*
my-dispatcher {
  type = Dispatcher
  executor = "fork-join-executor"
  
  fork-join-executor {
    parallelism-min = 8
    parallelism-factor = 3.0
    parallelism-max = 64
  }
  
  throughput = 100
}

blocking-io-dispatcher {
  type = Dispatcher
  executor = "thread-pool-executor"
  
  thread-pool-executor {
    fixed-pool-size = 32
  }
  
  throughput = 1
}
*/

// Mailbox configuration
/*
bounded-mailbox {
  mailbox-type = "akka.dispatch.BoundedMailbox"
  mailbox-capacity = 1000
  mailbox-push-timeout-time = 10s
}

priority-mailbox {
  mailbox-type = "com.example.PriorityMailbox"
}
*/

// Custom Priority Mailbox
class PriorityMailbox(settings: ActorSystem.Settings, config: Config)
  extends UnboundedPriorityMailbox(
    PriorityGenerator {
      case HighPriorityMessage => 0
      case NormalMessage => 1
      case LowPriorityMessage => 2
      case _ => 3
    }
  )

// Router для балансировки нагрузки
val optimalRouter = system.actorOf(
  SmallestMailboxPool(
    nrOfInstances = Runtime.getRuntime.availableProcessors() * 2
  ).props(Props[WorkerActor]),
  "optimal-router"
)

// Batching для эффективности
class BatchingActor extends Actor {
  import context.dispatcher
  
  private var batch: Vector[Message] = Vector.empty
  private val batchSize = 100
  private val batchTimeout = 100.millis
  
  context.system.scheduler.scheduleWithFixedDelay(
    initialDelay = batchTimeout,
    delay = batchTimeout,
    receiver = self,
    message = FlushBatch
  )
  
  def receive: Receive = {
    case msg: Message =>
      batch = batch :+ msg
      if (batch.size >= batchSize) {
        processBatch(batch)
        batch = Vector.empty
      }
    
    case FlushBatch if batch.nonEmpty =>
      processBatch(batch)
      batch = Vector.empty
  }
  
  def processBatch(messages: Vector[Message]): Unit = {
    // Обработка батча сообщений
  }
}
```

### 11.2 Memory Management

```scala
// Ограничение размера mailbox
val boundedActor = system.actorOf(
  Props[MyActor].withMailbox("bounded-mailbox")
)

// Backpressure в акторах
class BackpressureActor extends Actor {
  var pendingWork: Int = 0
  val maxPending = 100
  
  def receive: Receive = {
    case work: Work if pendingWork < maxPending =>
      pendingWork += 1
      processWork(work).onComplete { _ =>
        self ! WorkCompleted
      }
    
    case work: Work =>
      sender() ! Busy  // сообщаем отправителю о перегрузке
    
    case WorkCompleted =>
      pendingWork -= 1
  }
  
  def processWork(work: Work): Future[Unit] = ???
}

// Passivation для освобождения памяти
class PassivatingActor extends Actor with Timers {
  
  val PassivateTimeout = 30.seconds
  
  override def preStart(): Unit = {
    resetPassivationTimer()
  }
  
  def receive: Receive = {
    case msg =>
      handleMessage(msg)
      resetPassivationTimer()
    
    case Passivate =>
      context.stop(self)
  }
  
  def resetPassivationTimer(): Unit = {
    timers.startSingleTimer("passivate", Passivate, PassivateTimeout)
  }
  
  def handleMessage(msg: Any): Unit = ???
}
```

### 11.3 Monitoring и Metrics

```scala
// Kamon integration
/*
libraryDependencies ++= Seq(
  "io.kamon" %% "kamon-bundle" % "2.5.0",
  "io.kamon" %% "kamon-prometheus" % "2.5.0"
)
*/

import kamon.Kamon

object MonitoringExample {
  
  def startMonitoring(): Unit = {
    Kamon.init()
  }
  
  class MonitoredActor extends Actor {
    val messageCounter = Kamon.counter("actor.messages.processed")
    val processingTime = Kamon.timer("actor.processing.time")
    
    def receive: Receive = {
      case msg =>
        val started = System.nanoTime()
        
        processMessage(msg)
        
        messageCounter.increment()
        processingTime.record(System.nanoTime() - started, TimeUnit.NANOSECONDS)
    }
    
    def processMessage(msg: Any): Unit = ???
  }
}

// Custom Metrics Actor
class MetricsActor extends Actor with ActorLogging {
  
  private var messageCount: Long = 0
  private var errorCount: Long = 0
  private val startTime = System.currentTimeMillis()
  
  import context.dispatcher
  context.system.scheduler.scheduleWithFixedDelay(
    10.seconds,
    10.seconds,
    self,
    PrintMetrics
  )
  
  def receive: Receive = {
    case msg: WorkMessage =>
      messageCount += 1
      try {
        processWork(msg)
      } catch {
        case ex: Exception =>
          errorCount += 1
          log.error(ex, "Error processing message")
      }
    
    case PrintMetrics =>
      val uptime = (System.currentTimeMillis() - startTime) / 1000
      val rate = messageCount.toDouble / uptime
      log.info(s"Messages: $messageCount, Errors: $errorCount, Rate: ${rate}/s")
  }
  
  def processWork(msg: WorkMessage): Unit = ???
}
```

### 11.4 Best Practices

```scala
// 1. Никогда не блокируйте в акторе
class BadActor extends Actor {
  def receive: Receive = {
    case msg =>
      // ПЛОХО: блокирующая операция
      Thread.sleep(1000)
      
      // ПЛОХО: синхронный I/O
      val result = scala.io.Source.fromFile("file.txt").mkString
      
      sender() ! result
  }
}

class GoodActor extends Actor {
  import context.dispatcher
  
  // Используем отдельный диспетчер для блокирующих операций
  implicit val blockingEc: ExecutionContext = 
    context.system.dispatchers.lookup("blocking-io-dispatcher")
  
  def receive: Receive = {
    case msg =>
      val originalSender = sender()
      
      Future {
        // Блокирующая операция в отдельном thread pool
        scala.io.Source.fromFile("file.txt").mkString
      }.pipeTo(originalSender)
  }
}

// 2. Избегайте замыканий над mutable state
class BadStateActor extends Actor {
  var counter = 0
  
  def receive: Receive = {
    case "increment" =>
      Future {
        // ОПАСНО: замыкание над mutable state
        counter += 1
      }
  }
}

class GoodStateActor extends Actor {
  var counter = 0
  
  def receive: Receive = {
    case "increment" =>
      val current = counter  // захватываем immutable значение
      Future {
        // Безопасно
        val result = current + 1
        self ! UpdateCounter(result)
      }
    
    case UpdateCounter(value) =>
      counter = value
  }
}

// 3. Правильно используйте sender()
class BadSenderActor extends Actor {
  import context.dispatcher
  
  def receive: Receive = {
    case msg =>
      Future {
        Thread.sleep(1000)
        // ОПАСНО: sender() может измениться
        sender() ! "response"
      }
  }
}

class GoodSenderActor extends Actor {
  import context.dispatcher
  
  def receive: Receive = {
    case msg =>
      val replyTo = sender()  // захватываем sender
      Future {
        Thread.sleep(1000)
        replyTo ! "response"  // безопасно
      }
  }
}

// 4. Используйте become для FSM
class StatefulActor extends Actor {
  
  def receive: Receive = disconnected
  
  def disconnected: Receive = {
    case Connect =>
      // переход к состоянию connected
      context.become(connected)
      sender() ! Connected
  }
  
  def connected: Receive = {
    case Disconnect =>
      // возврат к disconnected
      context.become(disconnected)
      sender() ! Disconnected
    
    case msg: DataMessage =>
      // обработка данных только в connected состоянии
      processData(msg)
  }
  
  def processData(msg: DataMessage): Unit = ???
}

// 5. Правильно завершайте акторы
class GracefulStopActor extends Actor {
  
  override def postStop(): Unit = {
    // Освобождаем ресурсы
    closeConnections()
    flushBuffers()
  }
  
  def receive: Receive = {
    case Shutdown =>
      context.stop(self)
  }
  
  def closeConnections(): Unit = ???
  def flushBuffers(): Unit = ???
}
```

---

## 12. Migration to Pekko

### 12.1 Зачем нужен Pekko?

**Apache Pekko** - форк Akka после изменения лицензии Akka на BSL (Business Source License).

```
Akka 2.6 (Apache 2.0) → Pekko 1.0 (Apache 2.0)
Akka 2.7+ (BSL) → Commercial license required
```

### 12.2 Migration Guide

```scala
// Build.sbt changes

// OLD (Akka)
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor-typed" % "2.6.20",
  "com.typesafe.akka" %% "akka-stream" % "2.6.20",
  "com.typesafe.akka" %% "akka-http" % "10.2.10"
)

// NEW (Pekko)
libraryDependencies ++= Seq(
  "org.apache.pekko" %% "pekko-actor-typed" % "1.0.2",
  "org.apache.pekko" %% "pekko-stream" % "1.0.2",
  "org.apache.pekko" %% "pekko-http" % "1.0.1"
)

// Import changes

// OLD
import akka.actor.typed._
import akka.stream._
import akka.http.scaladsl._

// NEW
import org.apache.pekko.actor.typed._
import org.apache.pekko.stream._
import org.apache.pekko.http.scaladsl._

// Configuration changes (application.conf)

// OLD
akka {
  actor {
    provider = cluster
  }
}

// NEW
pekko {
  actor {
    provider = cluster
  }
}
```

### 12.3 Compatibility

```scala
// Pekko совместим с Akka 2.6 API
// Большинство кода работает без изменений

// Было (Akka 2.6)
val system = ActorSystem("MySystem")
val actor = system.actorOf(Props[MyActor])

// Стало (Pekko)
val system = ActorSystem("MySystem")  // тот же API
val actor = system.actorOf(Props[MyActor])  // тот же API

// Serialization - добавлен jackson
libraryDependencies += "org.apache.pekko" %% "pekko-serialization-jackson" % "1.0.2"
```

---

## 13. Real-World Patterns

### 13.1 Saga Pattern

```scala
// Distributed Transaction Saga
object SagaExample {
  
  sealed trait SagaStep {
    def execute(): Future[StepResult]
    def compensate(): Future[Unit]
  }
  
  sealed trait StepResult
  case object StepSuccess extends StepResult
  case class StepFailure(reason: String) extends StepResult
  
  class OrderSaga(
    orderService: OrderService,
    paymentService: PaymentService,
    inventoryService: InventoryService
  ) {
    
    def executeOrder(userId: String, items: List[Item]): Future[OrderResult] = {
      val steps = List(
        CreateOrderStep(orderService, userId, items),
        ReserveInventoryStep(inventoryService, items),
        ProcessPaymentStep(paymentService, userId, items)
      )
      
      executeSaga(steps)
    }
    
    private def executeSaga(steps: List[SagaStep]): Future[OrderResult] = {
      def loop(
        remaining: List[SagaStep],
        completed: List[SagaStep]
      ): Future[OrderResult] = {
        remaining match {
          case Nil =>
            Future.successful(OrderSuccess)
          
          case step :: tail =>
            step.execute().flatMap {
              case StepSuccess =>
                loop(tail, step :: completed)
              
              case StepFailure(reason) =>
                // Compensate все выполненные шаги
                compensateAll(completed).map(_ => OrderFailed(reason))
            }
        }
      }
      
      loop(steps, Nil)
    }
    
    private def compensateAll(steps: List[SagaStep]): Future[Unit] = {
      Future.sequence(steps.reverse.map(_.compensate())).map(_ => ())
    }
  }
  
  case class CreateOrderStep(
    service: OrderService,
    userId: String,
    items: List[Item]
  ) extends SagaStep {
    
    private var orderId: Option[String] = None
    
    def execute(): Future[StepResult] = {
      service.createOrder(userId, items).map { id =>
        orderId = Some(id)
        StepSuccess
      }.recover {
        case ex => StepFailure(ex.getMessage)
      }
    }
    
    def compensate(): Future[Unit] = {
      orderId match {
        case Some(id) => service.cancelOrder(id)
        case None => Future.successful(())
      }
    }
  }
}
```

### 13.2 CQRS с Akka

```scala
// Command side
object WriteModel {
  
  sealed trait Command
  case class CreateUser(userId: String, name: String) extends Command
  case class UpdateUserName(userId: String, newName: String) extends Command
  
  sealed trait Event
  case class UserCreated(userId: String, name: String, timestamp: Long) extends Event
  case class UserNameUpdated(userId: String, newName: String, timestamp: Long) extends Event
  
  class UserWriteModel(userId: String) extends EventSourcedBehavior[Command, Event, UserState] {
    
    override def persistenceId: PersistenceId = PersistenceId.ofUniqueId(userId)
    override def emptyState: UserState = UserState.empty
    
    override def commandHandler: CommandHandler[Command, Event, UserState] = {
      (state, command) => command match {
        case CreateUser(id, name) if state.isEmpty =>
          Effect.persist(UserCreated(id, name, System.currentTimeMillis()))
        
        case UpdateUserName(id, newName) if state.isDefined =>
          Effect.persist(UserNameUpdated(id, newName, System.currentTimeMillis()))
        
        case _ =>
          Effect.none
      }
    }
    
    override def eventHandler: EventHandler[UserState, Event] = {
      (state, event) => event match {
        case UserCreated(id, name, ts) =>
          UserState(Some(id), name, ts)
        
        case UserNameUpdated(_, newName, ts) =>
          state.copy(name = newName, lastModified = ts)
      }
    }
  }
  
  case class UserState(
    userId: Option[String],
    name: String,
    lastModified: Long
  ) {
    def isEmpty: Boolean = userId.isEmpty
    def isDefined: Boolean = userId.isDefined
  }
  
  object UserState {
    val empty: UserState = UserState(None, "", 0L)
  }
}

// Query side (Read Model)
object ReadModel {
  
  case class UserReadModel(
    userId: String,
    name: String,
    email: String,
    createdAt: Long,
    lastModified: Long,
    totalOrders: Int
  )
  
  class UserReadModelUpdater extends Actor {
    import WriteModel._
    
    // Подписка на события
    override def preStart(): Unit = {
      context.system.eventStream.subscribe(self, classOf[Event])
    }
    
    def receive: Receive = {
      case UserCreated(userId, name, timestamp) =>
        saveToReadDatabase(UserReadModel(
          userId, name, "", timestamp, timestamp, 0
        ))
      
      case UserNameUpdated(userId, newName, timestamp) =>
        updateReadDatabase(userId, newName, timestamp)
    }
    
    def saveToReadDatabase(model: UserReadModel): Unit = {
      // Save to optimized read database (e.g., PostgreSQL, Elasticsearch)
    }
    
    def updateReadDatabase(userId: String, newName: String, timestamp: Long): Unit = {
      // Update read database
    }
  }
  
  // Query Service
  class UserQueryService {
    def getUserById(userId: String): Future[Option[UserReadModel]] = {
      // Query from read database
      ???
    }
    
    def searchUsers(query: String): Future[List[UserReadModel]] = {
      // Full-text search
      ???
    }
    
    def getUsersByDateRange(from: Long, to: Long): Future[List[UserReadModel]] = {
      // Range query
      ???
    }
  }
}
```

### 13.3 Circuit Breaker

```scala
import akka.pattern.CircuitBreaker
import scala.concurrent.duration._

class ResilientServiceClient(implicit system: ActorSystem) {
  import system.dispatcher
  
  private val breaker = new CircuitBreaker(
    system.scheduler,
    maxFailures = 5,
    callTimeout = 10.seconds,
    resetTimeout = 1.minute
  ).onOpen {
    println("Circuit breaker opened")
  }.onHalfOpen {
    println("Circuit breaker half-open, trying to reset")
  }.onClose {
    println("Circuit breaker closed")
  }
  
  def callExternalService(request: Request): Future[Response] = {
    breaker.withCircuitBreaker {
      // Actual external service call
      makeHttpCall(request)
    }.recoverWith {
      case _: CircuitBreakerOpenException =>
        // Circuit is open, return cached response or default
        Future.successful(CachedResponse)
    }
  }
  
  private def makeHttpCall(request: Request): Future[Response] = {
    // HTTP call implementation
    ???
  }
}

// Typed Circuit Breaker
object TypedCircuitBreakerExample {
  
  sealed trait Command
  case class CallService(request: String, replyTo: ActorRef[Response]) extends Command
  private case class ServiceResponse(response: String) extends Command
  private case class ServiceFailure(ex: Throwable) extends Command
  
  sealed trait Response
  case class Success(data: String) extends Response
  case class Failure(reason: String) extends Response
  
  def apply()(implicit system: ActorSystem[_]): Behavior[Command] = {
    import system.executionContext
    
    val breaker = new CircuitBreaker(
      system.scheduler,
      maxFailures = 3,
      callTimeout = 5.seconds,
      resetTimeout = 30.seconds
    )
    
    Behaviors.receive { (context, message) =>
      message match {
        case CallService(request, replyTo) =>
          breaker.withCircuitBreaker {
            externalServiceCall(request)
          }.onComplete {
            case scala.util.Success(response) =>
              replyTo ! Success(response)
            case scala.util.Failure(ex) =>
              replyTo ! Failure(ex.getMessage)
          }
          
          Behaviors.same
      }
    }
  }
  
  private def externalServiceCall(request: String): Future[String] = {
    // External service call
    ???
  }
}
```

### 13.4 Work Pulling Pattern

```scala
// Work pulling - воркеры сами запрашивают работу
object WorkPullingPattern {
  
  // Master
  object Master {
    sealed trait Command
    case class WorkAvailable(work: Work) extends Command
    case class WorkerRequest(worker: ActorRef[Worker.Command]) extends Command
    
    def apply(): Behavior[Command] = {
      idle(Queue.empty, Queue.empty)
    }
    
    private def idle(
      pendingWork: Queue[Work],
      idleWorkers: Queue[ActorRef[Worker.Command]]
    ): Behavior[Command] = {
      Behaviors.receive { (context, message) =>
        message match {
          case WorkAvailable(work) =>
            idleWorkers.dequeueOption match {
              case Some((worker, remaining)) =>
                worker ! Worker.DoWork(work)
                idle(pendingWork, remaining)
              case None =>
                idle(pendingWork.enqueue(work), idleWorkers)
            }
          
          case WorkerRequest(worker) =>
            pendingWork.dequeueOption match {
              case Some((work, remaining)) =>
                worker ! Worker.DoWork(work)
                idle(remaining, idleWorkers)
              case None =>
                idle(pendingWork, idleWorkers.enqueue(worker))
            }
        }
      }
    }
  }
  
  // Worker
  object Worker {
    sealed trait Command
    case class DoWork(work: Work) extends Command
    private case object WorkDone extends Command
    
    def apply(master: ActorRef[Master.Command]): Behavior[Command] = {
      Behaviors.setup { context =>
        // Запрашиваем работу при старте
        master ! Master.WorkerRequest(context.self)
        
        ready(master)
      }
    }
    
    private def ready(master: ActorRef[Master.Command]): Behavior[Command] = {
      Behaviors.receive { (context, message) =>
        message match {
          case DoWork(work) =>
            // Асинхронная обработка
            processWork(work).onComplete { _ =>
              context.self ! WorkDone
            }(context.executionContext)
            
            working(master)
        }
      }
    }
    
    private def working(master: ActorRef[Master.Command]): Behavior[Command] = {
      Behaviors.receive { (context, message) =>
        message match {
          case WorkDone =>
            // Запрашиваем следующую работу
            master ! Master.WorkerRequest(context.self)
            ready(master)
          
          case _ =>
            Behaviors.unhandled
        }
      }
    }
    
    private def processWork(work: Work): Future[Unit] = {
      Future {
        Thread.sleep(1000)
        println(s"Processed: $work")
      }
    }
  }
  
  case class Work(id: String, payload: String)
}
```

---

## Заключение

### Ключевые выводы:

1. **Actor Model** - мощная модель для concurrent и distributed систем
2. **Akka Typed** - предпочтительный API для новых проектов
3. **Akka Streams** - elegant reactive streams implementation
4. **Akka Persistence** - надежный event sourcing
5. **Akka Cluster** - распределенные системы из коробки
6. **Testing** - comprehensive testing toolkit
7. **Pekko** - open-source альтернатива Akka 2.7+

### Дополнительные ресурсы:

**Документация:**
- [Akka Documentation](https://doc.akka.io/)
- [Lightbend Academy](https://academy.lightbend.com/)
- [Pekko Documentation](https://pekko.apache.org/)

**Книги:**
- "Akka in Action" - Raymond Roestenburg
- "Reactive Design Patterns" - Roland Kuhn
- "Akka Cookbook" - Héctor Veiga Ortiz

**Курсы:**
- Rock the JVM - Akka courses
- Udemy - Akka Essentials
- Coursera - Reactive Programming

**Практика:**
- Создайте chat application с WebSockets
- Реализуйте distributed counter с Cluster
- Постройте event-sourced систему
- Напишите streaming pipeline

### Подготовка к интервью:

✓ Понимание Actor Model концепций
✓ Знание Akka Typed API  
✓ Опыт с Akka Streams
✓ Понимание Supervision strategies
✓ Знание Akka HTTP
✓ Event Sourcing с Akka Persistence
✓ Cluster и Sharding patterns
✓ Testing strategies
✓ Performance tuning
✓ Production best practices

**Успехов в изучении Akka!** 🚀
