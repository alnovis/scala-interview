# Полное Руководство по Akka для Senior Scala Developer
## С Всеобъемлющими Теоретическими Материалами

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

#### История и философия

**Actor Model** была предложена Carl Hewitt в 1973 году в MIT как математическая модель для описания параллельных вычислений. Модель возникла из потребности в лучшем способе рассуждения о concurrent системах.

**Ключевые философские принципы:**

1. **"Everything is an actor"** - подобно OOP где "everything is an object"
2. **Асинхронная коммуникация как основа** - нет прямых вызовов методов
3. **Изоляция состояния** - каждый актор - независимая единица
4. **Location Transparency** - местоположение актора не имеет значения

```
Эволюция моделей параллельных вычислений:

1960s: Sequential Programming
       └─> Процедурное программирование

1970s: Shared Memory Concurrency
       └─> Threads + Locks
       └─> Сложности: deadlocks, race conditions

1973: Actor Model (Carl Hewitt)
       └─> Message Passing
       └─> Isolated State
       └─> Математическая основа

1980s-90s: Erlang OTP
       └─> Практическая реализация Actor Model
       └─> Fault tolerance через supervision

2000s: Akka (2009)
       └─> Actor Model для JVM
       └─> Enterprise-grade toolkit
```

#### Математическая модель

Актор - это примитив параллельных вычислений, который инкапсулирует:
- **Состояние (State)**: приватные данные
- **Поведение (Behavior)**: логика обработки сообщений
- **Mailbox**: очередь входящих сообщений

**Аксиомы Actor Model:**

```
Когда актор получает сообщение, он может:

1. CREATE: Создать конечное число новых акторов
   Actor + Message → Set[Actor]

2. SEND: Отправить конечное число сообщений другим акторам
   Actor + Message → Set[Message → ActorRef]

3. DESIGNATE: Определить поведение для следующего сообщения
   Actor + Message → Behavior'
   
Важно: Эти действия не имеют гарантированного порядка!
```

#### Модель коммуникации

```
                    Actor System
┌─────────────────────────────────────────────────┐
│                                                 │
│     Actor A                    Actor B          │
│  ┌──────────────┐          ┌──────────────┐    │
│  │   State: S1  │          │   State: S2  │    │
│  │   Behavior   │          │   Behavior   │    │
│  │              │          │              │    │
│  │   Mailbox:   │          │   Mailbox:   │    │
│  │   [M1, M2]   │          │   [M3]       │    │
│  └──────┬───────┘          └──────┬───────┘    │
│         │                         │            │
│         │   Message (async)       │            │
│         └────────────────────────►│            │
│                                   │            │
│                            ┌──────▼───────┐    │
│                            │   Process    │    │
│                            │   Message    │    │
│                            │   M3         │    │
│                            └──────────────┘    │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Свойства коммуникации:**

1. **Асинхронность**: отправитель не ждет ответа
2. **At-most-once delivery**: сообщение может быть доставлено 0 или 1 раз
3. **Не гарантированный порядок**: сообщения могут прийти в любом порядке
4. **Location transparency**: отправитель не знает, где находится получатель

#### Преимущества Actor Model

**1. Отсутствие Shared Mutable State**

```scala
// Проблема традиционного подхода:
class BankAccount(private var balance: BigDecimal) {
  def withdraw(amount: BigDecimal): Unit = synchronized {
    if (balance >= amount) {
      // Race condition даже с synchronized!
      Thread.sleep(10) // Симуляция обработки
      balance -= amount
    }
  }
}

// Actor Model решение:
class BankAccountActor extends Actor {
  private var balance: BigDecimal = 0
  
  def receive: Receive = {
    case Withdraw(amount) =>
      if (balance >= amount) {
        Thread.sleep(10) // Нет race condition!
        balance -= amount
        sender() ! Success(balance)
      } else {
        sender() ! InsufficientFunds
      }
  }
}
// Состояние изолировано, доступ только через сообщения
```

**2. Natural Parallelism**

```
Традиционный Thread Pool:
┌────────────────────────────────────┐
│  Shared Thread Pool (4 threads)   │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐             │
│  │T1│ │T2│ │T3│ │T4│             │
│  └──┘ └──┘ └──┘ └──┘             │
└────────────────────────────────────┘
         ↑
         │ Нужна синхронизация
         │
┌────────────────────────────────────┐
│    Shared Mutable State            │
│    (requires locks)                │
└────────────────────────────────────┘

Actor Model:
┌────────────────────────────────────┐
│  Dispatcher (thread pool)          │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐             │
│  │T1│ │T2│ │T3│ │T4│             │
│  └┬─┘ └┬─┘ └┬─┘ └┬─┘             │
└───┼────┼────┼────┼────────────────┘
    │    │    │    │
    ▼    ▼    ▼    ▼
  Actor Actor Actor Actor
  [S1]  [S2]  [S3]  [S4]
  
Каждый актор - независимая единица
Нет shared state, нет locks!
```

**3. Fault Tolerance через Supervision**

```
Supervision Tree:
              
       /guardian
           │
     ┌─────┴─────┐
     │           │
   /user      /system
     │
     ├─ Supervisor A
     │    │
     │    ├─ Worker 1
     │    ├─ Worker 2
     │    └─ Worker 3
     │
     └─ Supervisor B
          │
          └─ Worker 4

При ошибке в Worker 1:
1. Worker 1 падает
2. Supervisor A получает уведомление
3. Supervisor A принимает решение:
   - Restart worker 1
   - Stop worker 1
   - Escalate наверх
4. Другие workers не затронуты
```

**4. Location Transparency**

```scala
// Один и тот же код работает:

// 1. Локально в одной JVM
val localRef = system.actorOf(Props[MyActor], "myactor")

// 2. Удаленно через network
val remoteRef = system.actorSelection(
  "akka://RemoteSystem@host:port/user/myactor"
)

// Использование идентично:
localRef ! Message("hello")
remoteRef ! Message("hello")

// Программист не заботится о location!
```

#### Сравнение с другими моделями

**Actor Model vs Threads + Locks:**

```scala
// ❌ Threads + Locks (сложно, опасно)
class Counter {
  private val lock = new Object
  private var count: Int = 0
  
  def increment(): Unit = lock.synchronized {
    val temp = count
    Thread.sleep(1) // Симуляция работы
    count = temp + 1
  }
  
  // Проблемы:
  // - Deadlocks
  // - Race conditions
  // - Сложность композиции
  // - Плохая масштабируемость
}

// ✅ Actor Model (просто, безопасно)
class CounterActor extends Actor {
  private var count: Int = 0
  
  def receive: Receive = {
    case "increment" =>
      Thread.sleep(1)
      count += 1
      sender() ! count
  }
  
  // Преимущества:
  // - Нет deadlocks по дизайну
  // - Нет race conditions
  // - Легко композируется
  // - Масштабируется естественно
}
```

**Actor Model vs CSP (Communicating Sequential Processes):**

```
Actor Model (Akka, Erlang):
- Actors имеют identity (ActorRef)
- Асинхронные mailboxes
- Unbounded message buffers
- Location transparent
- Dynamic actor creation

CSP (Go channels):
- Channels - first-class citizens
- Синхронные каналы (по умолчанию)
- Bounded buffers
- Local focus
- Goroutines - анонимные
```

**Actor Model vs Reactive Streams:**

```
Actor Model:
┌─────────┐  Message  ┌─────────┐
│ Actor A │─────────▶ │ Actor B │
└─────────┘           └─────────┘
- Push-based
- Mailbox buffer
- No flow control

Reactive Streams:
┌──────────┐  Demand  ┌──────────┐
│Publisher │◀─────────│Subscriber│
│          │  Data    │          │
│          │─────────▶│          │
└──────────┘          └──────────┘
- Pull-based backpressure
- Flow control
- Standardized

Akka Streams = Actor Model + Reactive Streams
```

#### Теоретические гарантии

**Гарантии Actor Model:**

1. **Message Ordering** (между двумя акторами):
   ```
   Если A отправляет M1, затем M2 актору B,
   то B получит M1 перед M2 (если оба доставлены)
   ```

2. **Happens-Before Ordering**:
   ```scala
   actor ! M1  // happens-before
   actor ! M2  // обработка M1 happens-before обработка M2
   ```

3. **At-Most-Once Delivery**:
   ```
   Без дополнительных механизмов (ACK, retry):
   - Сообщение доставлено 0 или 1 раз
   - НЕТ гарантии доставки
   ```

4. **Single-Threaded Illusion**:
   ```
   Один актор обрабатывает только одно сообщение за раз
   → Нет concurrent доступа к состоянию актора
   → Не нужны locks внутри актора
   ```

**Что Actor Model НЕ гарантирует:**

```
❌ Гарантированную доставку сообщений
   → Нужно использовать At-Least-Once или Exactly-Once семантику

❌ Порядок сообщений от разных отправителей
   → A→B: M1, C→B: M2
   → B может получить в любом порядке

❌ Справедливость (fairness) в обработке
   → Один актор может "голодать"

❌ Bounded response time
   → Mailbox может расти безгранично
```

### 1.2 Akka Framework Overview

#### Архитектура Akka

**Общая архитектура:**

```
┌─────────────────────────────────────────────────────┐
│                  Application Layer                  │
│  ┌──────────────────────────────────────────────┐  │
│  │        Business Logic (Your Actors)           │  │
│  └──────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│               Akka Extension Layer                  │
│  ┌───────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │   Akka    │ │  Akka    │ │      Akka        │  │
│  │  Streams  │ │   HTTP   │ │   Persistence    │  │
│  └───────────┘ └──────────┘ └──────────────────┘  │
│  ┌───────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │   Akka    │ │  Akka    │ │      Akka        │  │
│  │  Cluster  │ │ Sharding │ │ Distributed Data │  │
│  └───────────┘ └──────────┘ └──────────────────┘  │
├─────────────────────────────────────────────────────┤
│                Akka Actor Core                      │
│  ┌──────────────────────────────────────────────┐  │
│  │  ActorSystem │ Dispatcher │ Mailbox │ Router │  │
│  │  Supervision │ Lifecycle  │ Scheduler        │  │
│  └──────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│           Infrastructure Layer                      │
│  ┌──────────────────────────────────────────────┐  │
│  │  Threading (ForkJoinPool, ThreadPoolExecutor)│  │
│  │  Serialization (Protocol Buffers, JSON)      │  │
│  │  Network (TCP, UDP, Artery)                  │  │
│  │  Configuration (HOCON)                       │  │
│  └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

#### Основные компоненты

**1. ActorSystem**

```
ActorSystem - root container для акторов

Отвечает за:
├─ Actor lifecycle management
├─ Message dispatching
├─ Resource management (threads, memory)
├─ Configuration loading
├─ Extension loading
├─ Serialization setup
└─ Termination coordination

┌────────────────────────────┐
│       ActorSystem          │
│  ┌──────────────────────┐  │
│  │   Guardian Actors    │  │
│  │  ┌────────────────┐  │  │
│  │  │  /user         │  │  │  ← User actors
│  │  └────────────────┘  │  │
│  │  ┌────────────────┐  │  │
│  │  │  /system       │  │  │  ← System actors
│  │  └────────────────┘  │  │
│  └──────────────────────┘  │
│                            │
│  ┌──────────────────────┐  │
│  │   Dispatchers        │  │
│  └──────────────────────┘  │
│  ┌──────────────────────┐  │
│  │   Scheduler          │  │
│  └──────────────────────┘  │
└────────────────────────────┘
```

**2. Dispatchers (Execution Context)**

```
Dispatcher = ExecutionContext + Mailbox Execution

Типы dispatchers:

1. Default Dispatcher (fork-join-executor)
   ├─ Для CPU-bound задач
   ├─ Работает на Fork-Join Pool
   └─ Default для всех акторов

2. Pinned Dispatcher (thread-pool-executor)
   ├─ Один thread на актор
   ├─ Для blocking операций
   └─ Дорого по ресурсам

3. Blocking Dispatcher
   ├─ Настроенный ThreadPool
   ├─ Для I/O операций
   └─ Изолирует blocking код

4. Balancing Dispatcher
   ├─ Разделяемый mailbox
   └─ Для балансировки нагрузки

Конфигурация:
┌────────────────────────────────────┐
│  my-dispatcher {                   │
│    type = Dispatcher               │
│    executor = "fork-join-executor" │
│    fork-join-executor {            │
│      parallelism-min = 2           │
│      parallelism-max = 8           │
│    }                               │
│    throughput = 5                  │
│  }                                 │
└────────────────────────────────────┘
```

**3. Mailbox**

```
Mailbox - очередь сообщений актора

Типы mailboxes:

1. Unbounded Mailbox (default)
   Queue: LinkedList
   Pro: Быстрая вставка
   Con: Может расти безгранично

2. Bounded Mailbox
   Queue: ArrayBlockingQueue
   Pro: Ограниченный размер
   Con: Блокировка при заполнении

3. Priority Mailbox
   Queue: PriorityQueue
   Pro: Приоритезация сообщений
   Con: Больше overhead

4. Control-Aware Mailbox
   Два queue: приоритетный и обычный
   Системные сообщения обрабатываются первыми

Структура:
┌────────────────────────────┐
│        Mailbox             │
│  ┌──────────────────────┐  │
│  │   System Messages    │  │
│  │   (высокий приоритет)│  │
│  └──────────────────────┘  │
│  ┌──────────────────────┐  │
│  │   User Messages      │  │
│  │   (FIFO order)       │  │
│  │   [M1, M2, M3, ...]  │  │
│  └──────────────────────┘  │
└────────────────────────────┘
```

#### Actor Model в Akka: Реализация

**Lifecycle Actor в Akka:**

```
               Constructor
                    │
                    ▼
           ┌────────────────┐
           │   preStart()   │
           └───────┬────────┘
                   │
                   ▼
           ┌────────────────┐
           │   Receiving    │◄────┐
           │   Messages     │     │
           └───────┬────────┘     │
                   │              │
                   │ Error        │ Restart
                   ▼              │
           ┌────────────────┐     │
           │ preRestart()   │     │
           └───────┬────────┘     │
                   │              │
                   ▼              │
           ┌────────────────┐     │
           │  postStop()    │     │
           └───────┬────────┘     │
                   │              │
                   ▼              │
           ┌────────────────┐     │
           │  Constructor   │     │
           └───────┬────────┘     │
                   │              │
                   ▼              │
           ┌────────────────┐     │
           │ postRestart()  │─────┘
           └────────────────┘
                   │
                   │ Stop
                   ▼
           ┌────────────────┐
           │  postStop()    │
           └────────────────┘
```

**Message Processing Pipeline:**

```
1. Message Send
   ┌─────────────┐
   │sender ! msg │
   └──────┬──────┘
          │
          ▼
2. Enqueue в Mailbox
   ┌────────────────┐
   │  actor.mailbox │
   │   += message   │
   └────────┬───────┘
            │
            ▼
3. Schedule на Dispatcher
   ┌───────────────────┐
   │ dispatcher.execute│
   │   (mailbox.run)   │
   └────────┬──────────┘
             │
             ▼
4. Dequeue и Process
   ┌─────────────────┐
   │ actor.receive   │
   │   .apply(msg)   │
   └────────┬────────┘
            │
            ▼
5. Throughput Check
   ┌──────────────────────┐
   │ if (moreMessages &&  │
   │  throughput < limit) │
   │    goto step 4       │
   │ else                 │
   │    reschedule        │
   └──────────────────────┘
```

#### Akka Ecosystem

**Модули и их назначение:**

```
1. akka-actor (Core)
   └─ Базовая имплементация Actor Model
   └─ Props, ActorRef, ActorSystem
   └─ Supervision, Dispatchers, Routers
   Use case: Concurrent приложения

2. akka-actor-typed
   └─ Type-safe actors с behavior
   └─ Нет sender(), явные протоколы
   └─ Функциональный стиль
   Use case: Новые проекты, type safety

3. akka-stream
   └─ Reactive Streams implementation
   └─ Source, Flow, Sink, Graph DSL
   └─ Backpressure handling
   Use case: Data processing pipelines

4. akka-http
   └─ HTTP server и client
   └─ Routing DSL, WebSocket support
   └─ Integration с Akka Streams
   Use case: REST APIs, web services

5. akka-persistence
   └─ Event Sourcing implementation
   └─ PersistentActor, EventSourcedBehavior
   └─ Snapshots, Recovery
   Use case: Stateful, durable systems

6. akka-cluster
   └─ Distributed computing
   └─ Gossip protocol для membership
   └─ Location transparency
   Use case: Multi-node приложения

7. akka-cluster-sharding
   └─ Distributed entities
   └─ Automatic shard rebalancing
   └─ Location transparent actors
   Use case: Millions of entities

8. akka-distributed-data
   └─ CRDTs implementation
   └─ Eventually consistent data
   └─ Gossip-based replication
   Use case: Distributed state

9. akka-cluster-tools
   └─ Cluster Singleton
   └─ Distributed Pub-Sub
   └─ Cluster Client
   Use case: Cluster patterns

10. akka-remote
    └─ Remote actor communication
    └─ Artery (Aeron-UDP based)
    └─ Serialization support
    Use case: Multi-JVM communication
```

#### Configuration Model

**HOCON Configuration:**

```hocon
# application.conf - Akka использует Typesafe Config

akka {
  # Общие настройки
  loglevel = "INFO"
  loggers = ["akka.event.slf4j.Slf4jLogger"]
  
  # Actor settings
  actor {
    # Default dispatcher
    default-dispatcher {
      type = Dispatcher
      executor = "fork-join-executor"
      
      fork-join-executor {
        # Min/Max threads
        parallelism-min = 8
        parallelism-factor = 3.0
        parallelism-max = 64
      }
      
      # Throughput
      throughput = 5
    }
    
    # Serialization
    serializers {
      java = "akka.serialization.JavaSerializer"
      proto = "akka.remote.serialization.ProtobufSerializer"
    }
    
    serialization-bindings {
      "java.io.Serializable" = java
      "com.google.protobuf.Message" = proto
    }
  }
  
  # Remote settings
  remote {
    artery {
      canonical.hostname = "127.0.0.1"
      canonical.port = 2551
    }
  }
  
  # Cluster settings
  cluster {
    seed-nodes = [
      "akka://MySystem@127.0.0.1:2551",
      "akka://MySystem@127.0.0.1:2552"
    ]
    
    roles = ["frontend", "backend"]
    
    min-nr-of-members = 2
  }
}
```

**Конфигурационные паттерны:**

```scala
// 1. Загрузка конфигурации
val config = ConfigFactory.load()
val system = ActorSystem("MySystem", config)

// 2. Программная конфигурация
val customConfig = ConfigFactory.parseString("""
  my-dispatcher {
    type = Dispatcher
    executor = "fork-join-executor"
    fork-join-executor {
      parallelism-min = 2
      parallelism-max = 4
    }
  }
""")

val system2 = ActorSystem("CustomSystem", 
  customConfig.withFallback(ConfigFactory.load()))

// 3. Использование в акторе
class MyActor extends Actor {
  val config = context.system.settings.config
  val myValue = config.getString("my.custom.value")
  
  def receive = ???
}
```

#### Akka Extensions

```
Extensions - механизм для добавления функциональности

Примеры встроенных extensions:
┌──────────────────────────────┐
│ ClusterExtension             │ → Cluster management
│ ShardingExtension            │ → Entity sharding
│ PersistenceExtension         │ → Event sourcing
│ DistributedData              │ → CRDTs
│ PubSubExtension              │ → Distributed pub-sub
└──────────────────────────────┘

Создание custom extension:
```

```scala
// 1. Extension interface
trait MyExtension extends Extension {
  def doSomething(): Unit
}

// 2. Extension implementation
class MyExtensionImpl(system: ExtendedActorSystem) 
  extends MyExtension {
  
  def doSomething(): Unit = {
    println(s"Doing something in ${system.name}")
  }
}

// 3. Extension ID
object MyExtension extends ExtensionId[MyExtension] 
  with ExtensionIdProvider {
  
  override def lookup = MyExtension
  
  override def createExtension(system: ExtendedActorSystem) =
    new MyExtensionImpl(system)
}

// 4. Использование
val extension = MyExtension(system)
extension.doSomething()
```

#### Threading Model

**Akka Threading Architecture:**

```
┌──────────────────────────────────────────────────────┐
│              ActorSystem Thread Pools                │
│                                                      │
│  Default Dispatcher (ForkJoinPool)                  │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐                       │
│  │ T1 │ │ T2 │ │ T3 │ │ T4 │                       │
│  └─┬──┘ └─┬──┘ └─┬──┘ └─┬──┘                       │
└────┼──────┼──────┼──────┼──────────────────────────┘
     │      │      │      │
     │      │      │      │
     ▼      ▼      ▼      ▼
   Actor  Actor  Actor  Actor
   [MB1]  [MB2]  [MB3]  [MB4]

Key points:
1. Акторы НЕ привязаны к threads
2. Thread выполняет mailbox.run()
3. После обработки N сообщений (throughput),
   thread освобождается
4. Другой thread может подхватить mailbox

Throughput механизм:
┌─────────────────────────────────┐
│  Thread processing Mailbox:     │
│  1. Dequeue message            │
│  2. Process message            │
│  3. Increment counter          │
│  4. If counter < throughput:   │
│     goto 1                     │
│  5. Else: release thread       │
│     and reschedule mailbox     │
└─────────────────────────────────┘
```

**Work-Stealing алгоритм (ForkJoinPool):**

```
Thread 1 Queue:  [T1, T2, T3, T4]
Thread 2 Queue:  [T5, T6]
Thread 3 Queue:  []  ← idle

Thread 3 "steals" from Thread 1:
Thread 1 Queue:  [T1, T2]
Thread 2 Queue:  [T5, T6]
Thread 3 Queue:  [T3, T4]  ← stolen

Преимущества:
✓ Automatic load balancing
✓ Эффективное использование CPU
✓ Хорошо для recursive задач
```

---

## 2. Akka Classic Actors

### 2.1 Создание первого актора

#### Анатомия актора

**Структура актора:**

```
Actor = State + Behavior + Mailbox

┌───────────────────────────────────────┐
│           Actor Instance              │
│                                       │
│  ┌─────────────────────────────────┐  │
│  │  Private State (var/val)        │  │
│  │  - Mutable variables            │  │
│  │  - Immutable data               │  │
│  │  - Resources (connections, etc) │  │
│  └─────────────────────────────────┘  │
│                                       │
│  ┌─────────────────────────────────┐  │
│  │  Behavior (receive method)      │  │
│  │  - Pattern matching             │  │
│  │  - Message handling logic       │  │
│  │  - State transitions            │  │
│  └─────────────────────────────────┘  │
│                                       │
│  ┌─────────────────────────────────┐  │
│  │  Context                        │  │
│  │  - ActorRef (self)              │  │
│  │  - Sender ref                   │  │
│  │  - Children management          │  │
│  │  - Supervision                  │  │
│  └─────────────────────────────────┘  │
│                                       │
│  ┌─────────────────────────────────┐  │
│  │  Mailbox (не в actor instance)  │  │
│  │  - Message queue                │  │
│  │  - Managed by dispatcher        │  │
│  └─────────────────────────────────┘  │
└───────────────────────────────────────┘
```

**Базовый актор с аннотациями:**

```scala
import akka.actor.{Actor, ActorRef, ActorSystem, Props}

// 1. Определение протокола сообщений
sealed trait GreeterProtocol
case class Greet(name: String) extends GreeterProtocol
case class Greeting(message: String) extends GreeterProtocol
case object GetGreetingCount extends GreeterProtocol

// 2. Реализация актора
class GreeterActor extends Actor {
  
  // Private state - доступен только этому актору
  private var greetingCount: Int = 0
  
  // receive - основной метод, определяющий behavior
  def receive: Receive = {
    // Pattern matching на входящих сообщениях
    case Greet(name) =>
      greetingCount += 1
      val message = s"Hello, $name! (Greeting #$greetingCount)"
      println(message)
      
      // sender() - ActorRef отправителя
      // ! (tell) - асинхронная отправка сообщения
      sender() ! Greeting(message)
    
    case GetGreetingCount =>
      sender() ! greetingCount
    
    // Default case для неизвестных сообщений
    case unknown =>
      println(s"Unknown message: $unknown")
      // Или можно использовать unhandled()
      unhandled(unknown)
  }
}

// 3. Создание и использование
object GreeterApp extends App {
  // ActorSystem - контейнер для акторов
  val system = ActorSystem("GreeterSystem")
  
  try {
    // Создание актора через Props
    val greeter: ActorRef = system.actorOf(
      Props[GreeterActor],
      name = "greeter"  // опциональное имя
    )
    
    // ActorRef - handle к актору (не сам актор!)
    // Можно безопасно передавать между threads
    
    // Fire-and-forget отправка
    greeter ! Greet("Alice")
    greeter ! Greet("Bob")
    greeter ! GetGreetingCount
    
    // Даем время на обработку
    Thread.sleep(1000)
    
  } finally {
    // Graceful shutdown
    system.terminate()
  }
}
```

#### ActorRef vs Actor Instance

**Важнейшая концепция:**

```
┌──────────────────────────────────────────────┐
│  ActorRef - единственный способ общения!     │
│                                              │
│  ✓ Thread-safe                               │
│  ✓ Serializable                              │
│  ✓ Location transparent                      │
│  ✓ Can be sent in messages                   │
│                                              │
│  Actor Instance - приватный, недоступный     │
│                                              │
│  ✗ НЕ thread-safe                            │
│  ✗ НЕ serializable                           │
│  ✗ Только один thread за раз                 │
│  ✗ НИКОГДА не передавайте прямую ссылку      │
└──────────────────────────────────────────────┘

Правильно:
  val ref: ActorRef = context.actorOf(Props[MyActor])
  ref ! Message  ✓

Неправильно:
  val actor = new MyActor  // ✗ НЕ ДЕЛАЙТЕ ТАК!
  actor.receive(Message)    // ✗ НИКОГДА!
```

**Получение ActorRef:**

```scala
class MyActor extends Actor {
  
  // 1. self - ссылка на себя
  val selfRef: ActorRef = self
  
  // 2. sender() - отправитель текущего сообщения
  def receive: Receive = {
    case msg =>
      val senderRef: ActorRef = sender()
      senderRef ! "Reply"
  }
  
  // 3. Creating children
  val child: ActorRef = context.actorOf(Props[ChildActor], "child")
  
  // 4. Actor selection по пути
  val selection: ActorSelection = 
    context.actorSelection("/user/someActor")
  
  // 5. Из ActorSystem
  val rootActor: ActorRef = 
    context.system.actorOf(Props[RootActor], "root")
}
```

### 2.2 Props и Actor Creation

#### Props - Factory для акторов

**Зачем нужны Props?**

```
Props инкапсулирует:
1. Класс актора
2. Конструктор и его параметры
3. Deploy configuration (dispatcher, mailbox, router)

Props позволяет:
✓ Type-safe creation
✓ Serializable actor deployment
✓ Remote actor creation
✓ Configuration separation
```

**Способы создания Props:**

```scala
// Класс актора
class WorkerActor(id: Int, name: String) extends Actor {
  def receive: Receive = {
    case Work(task) => 
      println(s"Worker $id ($name) processing: $task")
  }
}

// ❌ АНТИПАТТЕРН: Прямое создание
val props1 = Props(new WorkerActor(1, "Alice"))
// Проблемы:
// - Замыкание над внешними переменными
// - Не работает для remote deployment
// - Может привести к serialization issues

// ✓ ПРАВИЛЬНО: Через classOf
val props2 = Props(classOf[WorkerActor], 1, "Alice")
// Преимущества:
// - Акка контролирует создание instance
// - Работает с remote actors
// - Type-safe (compile-time check)

// ✓ РЕКОМЕНДУЕТСЯ: Companion object factory
object WorkerActor {
  def props(id: Int, name: String): Props =
    Props(classOf[WorkerActor], id, name)
}

val props3 = WorkerActor.props(1, "Alice")
// Лучшая практика:
// - Инкапсуляция деталей создания
// - Удобный API
// - Легко менять реализацию
```

**Props с deploy configuration:**

```scala
import akka.routing.RoundRobinPool

// Custom dispatcher
val propsWithDispatcher = Props[MyActor]
  .withDispatcher("my-custom-dispatcher")

// Custom mailbox
val propsWithMailbox = Props[MyActor]
  .withMailbox("priority-mailbox")

// Router configuration
val propsWithRouter = Props[MyActor]
  .withRouter(RoundRobinPool(5))

// Combine configurations
val props = Props(classOf[MyActor], arg1, arg2)
  .withDispatcher("blocking-dispatcher")
  .withMailbox("bounded-mailbox")
```

#### Actor Creation Patterns

**1. Top-level actors (root):**

```scala
val system = ActorSystem("MySystem")

// Создаются под /user guardian
val rootActor = system.actorOf(Props[RootActor], "root")
// Path: akka://MySystem/user/root
```

**2. Child actors (hierarchy):**

```scala
class ParentActor extends Actor {
  
  // Создание child при старте
  override def preStart(): Unit = {
    context.actorOf(Props[ChildActor], "child1")
  }
  
  // Создание child динамически
  def receive: Receive = {
    case CreateChild(name) =>
      val child = context.actorOf(Props[ChildActor], name)
      sender() ! ChildCreated(child)
    
    case msg =>
      // Пересылка всем children
      context.children.foreach(_ ! msg)
  }
}

// Hierarchy:
// /user/parent
//   ├─ /child1
//   ├─ /child2
//   └─ /child3
```

**3. Actor Selection:**

```scala
// Absolute path
val selection1 = system.actorSelection("/user/parent/child1")

// Relative path
class MyActor extends Actor {
  val sibling = context.actorSelection("../sibling")
  val child = context.actorSelection("child/*")  // wildcard
  val allChildren = context.actorSelection("**")  // recursive
  
  def receive: Receive = {
    case msg =>
      // ActorSelection может отправлять сообщения
      selection1 ! msg
      
      // Resolve to ActorRef
      import akka.pattern.ask
      import scala.concurrent.duration._
      implicit val timeout = Timeout(5.seconds)
      
      (selection1 ? Identify(None)).mapTo[ActorIdentity].foreach {
        case ActorIdentity(_, Some(ref)) =>
          // Got ActorRef
          println(s"Found actor: $ref")
        case ActorIdentity(_, None) =>
          println("Actor not found")
      }
  }
}
```

#### Naming и Actor Paths

**Actor Path Structure:**

```
akka://SystemName@Host:Port/user/parent/child

Разбор:
┌─────────┬──────────────┬──────────────────────┐
│ Protocol│ ActorSystem  │    Actor Path         │
│         │  Address     │                       │
│ akka:// │MySystem      │  /user/parent/child   │
│         │@host:port    │                       │
└─────────┴──────────────┴──────────────────────┘

Для remote:
akka://MySystem@192.168.1.10:2552/user/worker

Для local:
akka://MySystem/user/worker
```

**Guardian Actors:**

```
Root Guardian (/)
  │
  ├─ /system  (System Guardian)
  │    ├─ /system/deadLetters
  │    ├─ /system/log
  │    ├─ /system/IO-TCP
  │    └─ ... other system actors
  │
  └─ /user  (User Guardian)
       ├─ /user/myActor1
       ├─ /user/myActor2
       └─ /user/parent
            ├─ /user/parent/child1
            └─ /user/parent/child2
```

**Naming Rules:**

```scala
// ✓ Valid names
context.actorOf(Props[MyActor], "my-actor")
context.actorOf(Props[MyActor], "actor123")
context.actorOf(Props[MyActor], "my_actor")

// ✗ Invalid names
context.actorOf(Props[MyActor], "my actor")  // spaces
context.actorOf(Props[MyActor], "my/actor")  // slash
context.actorOf(Props[MyActor], "$actor")    // reserved chars

// Duplicate names
context.actorOf(Props[MyActor], "child")  // OK
context.actorOf(Props[MyActor], "child")  // ✗ InvalidActorNameException

// Anonymous actors (auto-generated name)
context.actorOf(Props[MyActor])  // name: $a, $b, $c, ...
```

### 2.3 Actor Lifecycle

#### Lifecycle Hooks

**Полный цикл жизни актора:**

```
         NEW
          │
    Constructor
          │
          ▼
     ┌─────────┐
     │preStart │  ← Initialization
     └────┬────┘
          │
          ▼
  ┌───────────────┐
  │   RUNNING     │  ← Normal operation
  │  (receiving)  │
  └───────┬───────┘
          │
          ├─── Error ────┐
          │              │
          │              ▼
          │       ┌──────────────┐
          │       │ preRestart   │
          │       └──────┬───────┘
          │              │
          │              ▼
          │       ┌──────────────┐
          │       │  postStop    │
          │       └──────┬───────┘
          │              │
          │              ▼
          │       ┌──────────────┐
          │       │ Constructor  │
          │       └──────┬───────┘
          │              │
          │              ▼
          │       ┌──────────────┐
          │       │ postRestart  │
          │       └──────┬───────┘
          │              │
          │              └────────┐
          │                       │
          │                       ▼
          │                   RUNNING
          │
          ├─── Stop ─────┐
          │              │
          ▼              ▼
    ┌──────────┐   ┌──────────┐
    │postStop  │   │postStop  │
    └────┬─────┘   └────┬─────┘
         │              │
         ▼              ▼
     TERMINATED     TERMINATED
```

**Детальное описание hooks:**

```scala
class LifecycleActor extends Actor with ActorLogging {
  
  // 1. Constructor - синхронное создание
  log.info("Constructor: Actor instance created")
  private var resource: Resource = null
  
  // 2. preStart() - вызывается ПЕРЕД обработкой первого сообщения
  override def preStart(): Unit = {
    log.info("preStart: Actor is starting")
    super.preStart()
    
    // Используйте для:
    // - Инициализация ресурсов
    // - Подписка на события
    // - Отправка начальных сообщений
    resource = Resource.create()
    context.system.eventStream.subscribe(self, classOf[SomeEvent])
  }
  
  // 3. receive - обработка сообщений
  def receive: Receive = {
    case msg =>
      log.info(s"Processing: $msg")
      // Бизнес-логика здесь
  }
  
  // 4. postStop() - вызывается ПОСЛЕ остановки актора
  override def postStop(): Unit = {
    log.info("postStop: Actor is stopping")
    
    // Используйте для:
    // - Освобождение ресурсов
    // - Отписка от событий
    // - Cleanup
    if (resource != null) {
      resource.close()
    }
    context.system.eventStream.unsubscribe(self)
    
    super.postStop()
  }
  
  // 5. preRestart() - вызывается перед перезапуском
  override def preRestart(reason: Throwable, message: Option[Any]): Unit = {
    log.error(reason, s"preRestart: Restarting due to failure. Message: $message")
    
    // По умолчанию:
    // 1. Останавливает всех children
    // 2. Вызывает postStop()
    super.preRestart(reason, message)
    
    // Можно переопределить, чтобы НЕ останавливать children:
    // Не вызываем super.preRestart()
    // Только postStop() без остановки детей
  }
  
  // 6. postRestart() - вызывается после перезапуска
  override def postRestart(reason: Throwable): Unit = {
    log.info(s"postRestart: Actor restarted after: ${reason.getMessage}")
    
    // По умолчанию вызывает preStart()
    super.postRestart(reason)
    
    // Можно переопределить для особой логики восстановления
  }
}
```

**Lifecycle в контексте supervision:**

```
Parent Actor
     │
     ├─ creates ─┐
     │           ▼
     │      Child Actor
     │           │
     │           ├─ preStart()
     │           │
     │           ▼
     │      Processing messages
     │           │
     │           │ Exception!
     │           ▼
     │      SupervisorStrategy
     │           │
     │           ├─ Restart Decision
     │           │
     │           ▼
     │      preRestart(exception, message)
     │           │
     │           ▼
     │      postStop()
     │           │
     │           ▼
     │      Constructor
     │           │
     │           ▼
     │      postRestart(exception)
     │           │
     │           ▼
     │      preStart()
     │           │
     │           ▼
     │      Processing messages
```

### 2.4 Message Passing

#### Tell vs Ask

**Tell (fire-and-forget):**

```
Tell (!) - асинхронная отправка без ожидания ответа

┌─────────┐           ┌─────────┐
│ Sender  │─── ! ───▶ │Receiver │
└─────────┘           └─────────┘
     │                     │
     │                     ├─ Enqueue message
     │                     ├─ Process message
     │                     └─ Maybe send response
     │
     └─ Continue immediately
        (no blocking)

Характеристики:
✓ Non-blocking
✓ Fast
✓ No Future overhead
✓ "Let it crash" philosophy
✗ No direct response handling
```

```scala
// Базовый tell
actorRef ! Message("hello")

// Tell с явным sender
actorRef.tell(Message("hello"), self)

// В акторе
class MyActor extends Actor {
  def receive: Receive = {
    case Request(data) =>
      // Process data
      sender() ! Response(result)  // Reply using sender()
  }
}
```

**Ask (request-response):**

```
Ask (?) - отправка с ожиданием ответа через Future

┌─────────┐    ?      ┌─────────┐
│ Sender  │─────────▶ │Receiver │
└────┬────┘           └────┬────┘
     │                     │
     │ Future[Response]    ├─ Enqueue
     │                     ├─ Process
     │                     └─ Send response
     │                          │
     │◀─────── Complete ────────┘
     │
     └─ onComplete / map / flatMap

Характеристики:
✓ Type-safe response
✓ Composable Futures
✓ Timeout handling
✗ Creates temporary actor
✗ Performance overhead
✗ Can cause memory leaks if not handled
```

```scala
import akka.pattern.ask
import akka.util.Timeout
import scala.concurrent.duration._
import scala.concurrent.Future

implicit val timeout: Timeout = Timeout(5.seconds)
import context.dispatcher  // ExecutionContext

// Ask returns Future
val future: Future[Any] = actorRef ? GetData

// Type-safe with mapTo
val typedFuture: Future[Response] = 
  (actorRef ? GetData).mapTo[Response]

// With onComplete
typedFuture.onComplete {
  case Success(response) => println(s"Got: $response")
  case Failure(ex) => println(s"Failed: $ex")
}

// In for-comprehension
val result = for {
  response1 <- (actor1 ? Query1).mapTo[Response1]
  response2 <- (actor2 ? Query2(response1)).mapTo[Response2]
} yield response2

// Обработка timeout
typedFuture.recover {
  case _: AskTimeoutException =>
    println("Request timed out")
    DefaultResponse
}
```

**Когда использовать tell vs ask:**

```
✓ Используйте TELL (!):
  - Default choice
  - Fire-and-forget semantics
  - Internal actor communication
  - High throughput needed
  - Reactive, event-driven flow

✓ Используйте ASK (?):
  - Request-response pattern
  - Integration с non-actor code
  - HTTP request handling
  - Database queries
  - Need explicit timeout handling

Example:
class ApiActor extends Actor {
  def receive: Receive = {
    case HttpRequest =>
      // Ask для database
      val future = (dbActor ? Query).mapTo[Result]
      
      // Pipe result back
      import akka.pattern.pipe
      future.pipeTo(sender())
  }
}

class BusinessActor extends Actor {
  def receive: Receive = {
    case BusinessEvent =>
      // Tell для internal logic
      workerActor ! ProcessTask
      
      // Tell себе для state change
      self ! UpdateState
  }
}
```

#### Message Delivery Semantics

**Гарантии доставки:**

```
1. At-Most-Once Delivery (default в Akka)
   ┌────┐      M      ┌────┐
   │ S  │ ─────────▶  │ R  │
   └────┘             └────┘
   
   Возможные исходы:
   - Сообщение доставлено и обработано (1 раз)
   - Сообщение потеряно (0 раз)
   
   ✓ Fast, low overhead
   ✗ May lose messages

2. At-Least-Once Delivery (Akka Persistence)
   ┌────┐   M,ACK    ┌────┐
   │ S  │◀──────────▶│ R  │
   └────┘  retry     └────┘
   
   Возможные исходы:
   - Сообщение доставлено и обработано (1+ раз)
   - Никогда: 0 раз
   
   ✓ Guaranteed delivery
   ✗ May duplicate
   ✗ Overhead: ACK, retry, persistence

3. Exactly-Once Delivery (manual implementation)
   ┌────┐  M,ID,ACK  ┌────┐
   │ S  │◀──────────▶│ R  │
   └────┘dedup,store └────┘
   
   Реализация:
   - Message ID
   - Deduplication на receiver
   - Persistent state на обеих сторонах
   
   ✓ No duplicates
   ✗ Complex
   ✗ High overhead
   ✗ Distributed transactions challenge
```

**Ordering гарантии:**

```
Scenario 1: Один отправитель, один получатель
┌─────┐  M1  ┌─────┐
│  A  │─────▶│  B  │
│     │  M2  │     │
│     │─────▶│     │
└─────┘      └─────┘

Гарантия: M1 обработается before M2
(если оба доставлены)

Scenario 2: Множественные отправители
┌─────┐  M1  ┌─────┐
│  A  │─────▶│     │
└─────┘      │  C  │
┌─────┐  M2  │     │
│  B  │─────▶│     │
└─────┘      └─────┘

НЕТ гарантии порядка между M1 и M2!
C может получить в любом порядке

Scenario 3: Forwarding
┌─────┐  M1  ┌─────┐  M1'  ┌─────┐
│  A  │─────▶│  B  │──────▶│  C  │
│     │  M2  │     │  M2'  │     │
│     │─────▶│     │──────▶│     │
└─────┘      └─────┘       └─────┘

Если B forwards в порядке,
C получит M1' перед M2'
```

### 2.5 Context и ActorContext

#### Context API

**ActorContext - окружение актора:**

```scala
class MyActor extends Actor {
  
  // ═══════════════════════════════════
  //  Lifecycle Management
  // ═══════════════════════════════════
  
  context.become(newBehavior)     // Смена behavior
  context.unbecome()              // Возврат к предыдущему
  context.stop(self)              // Остановка актора
  context.watch(otherActor)       // Наблюдение за актором
  context.unwatch(otherActor)     // Прекращение наблюдения
  
  // ═══════════════════════════════════
  //  Actor Creation
  // ═══════════════════════════════════
  
  val child = context.actorOf(Props[ChildActor], "child")
  context.actorSelection("/user/other")
  
  // ═══════════════════════════════════
  //  Identity
  // ═══════════════════════════════════
  
  context.self         // ActorRef этого актора
  context.sender()     // Отправитель текущего сообщения
  context.parent       // ActorRef родителя
  
  // ═══════════════════════════════════
  //  Hierarchy
  // ═══════════════════════════════════
  
  context.children     // Iterable[ActorRef] детей
  context.child("name") // Option[ActorRef] по имени
  
  // ═══════════════════════════════════
  //  System Access
  // ═══════════════════════════════════
  
  context.system       // ActorSystem
  context.dispatcher   // ExecutionContext
  
  // ═══════════════════════════════════
  //  Configuration
  // ═══════════════════════════════════
  
  context.setReceiveTimeout(10.seconds)
  
  def receive = ???
}
```

#### Become/Unbecome - Изменение поведения

**Finite State Machine с become:**

```
┌─────────────────────────────────────────────┐
│  Become/Unbecome - механизм смены поведения │
│                                             │
│  Stack of behaviors:                        │
│  ┌─────────────────┐                        │
│  │  Current (top)  │ ← active                │
│  ├─────────────────┤                        │
│  │  Previous       │                         │
│  ├─────────────────┤                        │
│  │  Initial        │                         │
│  └─────────────────┘                        │
│                                             │
│  become(behavior, discardOld=true)          │
│  → replace top                              │
│                                             │
│  become(behavior, discardOld=false)         │
│  → push to stack                            │
│                                             │
│  unbecome()                                 │
│  → pop from stack                           │
└─────────────────────────────────────────────┘
```

**Пример - Door Actor:**

```scala
class DoorActor extends Actor {
  
  // Initial behavior - закрытая дверь
  def receive: Receive = closed
  
  // State: Closed
  def closed: Receive = {
    case Open =>
      println("Door opening")
      context.become(opened)  // Смена на opened behavior
      sender() ! DoorOpened
    
    case Close =>
      sender() ! AlreadyClosed
    
    case Knock =>
      println("Knock knock")
  }
  
  // State: Opened
  def opened: Receive = {
    case Close =>
      println("Door closing")
      context.become(closed)  // Смена на closed behavior
      sender() ! DoorClosed
    
    case Open =>
      sender() ! AlreadyOpened
    
    case Knock =>
      println("Door is open, come in!")
  }
}

// State diagram:
//    Closed ◄──── Close ──── Opened
//      │                        │
//      └──────── Open ─────────┘
```

**Пример - Authentication:**

```scala
class AuthActor extends Actor with ActorLogging {
  
  // Initial: Unauthenticated
  def receive: Receive = unauthenticated
  
  def unauthenticated: Receive = {
    case Login(username, password) =>
      if (authenticate(username, password)) {
        log.info(s"User $username logged in")
        context.become(authenticated(username))
        sender() ! LoginSuccess(username)
      } else {
        log.warning(s"Failed login attempt for $username")
        sender() ! LoginFailed
      }
    
    case GetProfile =>
      sender() ! Unauthorized
    
    case Logout =>
      sender() ! NotLoggedIn
  }
  
  def authenticated(username: String): Receive = {
    case GetProfile =>
      sender() ! UserProfile(username, loadProfile(username))
    
    case Logout =>
      log.info(s"User $username logged out")
      context.unbecome()  // Возврат к unauthenticated
      sender() ! LogoutSuccess
    
    case Login(_, _) =>
      sender() ! AlreadyLoggedIn(username)
  }
  
  private def authenticate(user: String, pass: String): Boolean = ???
  private def loadProfile(user: String): Profile = ???
}
```

**Stacked Behaviors:**

```scala
class CalculatorActor extends Actor {
  
  def receive: Receive = calculator(0)
  
  def calculator(current: Int): Receive = {
    case Add(value) =>
      val result = current + value
      sender() ! Result(result)
      context.become(calculator(result))
    
    case Subtract(value) =>
      val result = current - value
      sender() ! Result(result)
      context.become(calculator(result))
    
    case Multiply(value) =>
      val result = current * value
      sender() ! Result(result)
      context.become(calculator(result))
    
    case Clear =>
      sender() ! Result(0)
      context.become(calculator(0))
    
    case Save =>
      // Push current state to stack
      context.become(calculator(current), discardOld = false)
      sender() ! Saved(current)
    
    case Restore =>
      // Pop from stack
      context.unbecome()
      sender() ! Restored
    
    case GetCurrent =>
      sender() ! Result(current)
  }
}

// Usage:
calculator ! Add(5)      // State: 5
calculator ! Multiply(3) // State: 15
calculator ! Save        // Stack: [15]
calculator ! Add(10)     // State: 25
calculator ! Restore     // State: 15 (from stack)
```

#### Watch/Unwatch - Death Watch

**Мониторинг жизненного цикла:**

```
Death Watch Pattern:

Watcher Actor          Watched Actor
     │                      │
     │──── watch(ref) ─────▶│
     │                      │
     │                      │ (working...)
     │                      │
     │                      ✗ (terminates)
     │                      │
     │◄─── Terminated(ref) ─┘
     │
     └─ Handle termination

Key points:
✓ Watcher получает Terminated message
✓ Гарантированная доставка
✓ Не зависит от причины остановки
✓ Автоматически unwatch при terminated
```

```scala
class SupervisorActor extends Actor with ActorLogging {
  
  override def preStart(): Unit = {
    // Создаем и начинаем наблюдать за worker
    val worker = context.actorOf(Props[WorkerActor], "worker")
    context.watch(worker)
  }
  
  def receive: Receive = {
    case Terminated(actorRef) =>
      log.warning(s"Actor terminated: ${actorRef.path}")
      
      // Можем принять решение:
      // 1. Перезапустить
      val newWorker = context.actorOf(Props[WorkerActor], "worker-new")
      context.watch(newWorker)
      
      // 2. Или остановить себя
      // context.stop(self)
      
      // 3. Или игнорировать
    
    case msg =>
      // Forward to worker
      context.child("worker").foreach(_ ! msg)
  }
  
  override def postStop(): Unit = {
    // unwatch автоматически вызывается при stop
  }
}
```

**Parent-Child автоматический watch:**

```
Parent-Child relationship:

Parent
  │
  ├─ creates ──▶ Child
  │
  │ (автоматический watch)
  │
  │              Child terminates
  │◄─────────── (no Terminated message!)
  │
  └─ But: supervisor notified via SupervisorStrategy

Отличие watch от parent-child:
┌─────────────────────────────────────────┐
│ Parent-Child:                           │
│ - Automatic supervision                 │
│ - SupervisorStrategy применяется        │
│ - Restart, Stop, Resume, Escalate       │
│                                         │
│ Watch:                                  │
│ - Explicit monitoring                   │
│ - Terminated message                    │
│ - No automatic restart                  │
│ - Can watch any actor                   │
└─────────────────────────────────────────┘
```

**Resource Cleanup Pattern:**

```scala
class ResourceActor extends Actor with ActorLogging {
  
  var resources: Set[Resource] = Set.empty
  var watchedActors: Set[ActorRef] = Set.empty
  
  def receive: Receive = {
    case RegisterResource(resource, owner) =>
      resources += resource
      
      // Watch owner для cleanup
      if (!watchedActors.contains(owner)) {
        context.watch(owner)
        watchedActors += owner
      }
      
      sender() ! ResourceRegistered(resource.id)
    
    case Terminated(owner) =>
      log.info(s"Owner $owner terminated, cleaning up resources")
      
      // Cleanup resources принадлежащие owner
      val ownerResources = resources.filter(_.owner == owner)
      ownerResources.foreach(_.close())
      
      resources --= ownerResources
      watchedActors -= owner
    
    case ReleaseResource(resourceId) =>
      resources.find(_.id == resourceId).foreach { resource =>
        resource.close()
        resources -= resource
      }
  }
  
  override def postStop(): Unit = {
    // Cleanup всех resources при остановке
    resources.foreach(_.close())
    resources = Set.empty
  }
}
```

---

## 3. Akka Typed Actors

### 3.1 Введение в Akka Typed

#### Мотивация и преимущества

**Проблемы Classic API:**

```scala
// ❌ Classic API - Runtime типы
class ClassicActor extends Actor {
  def receive: Receive = {
    case msg: String => println(msg)
    case msg: Int => println(msg)
    case _ => // Что делать с неизвестными типами?
  }
}

// Проблемы:
// 1. Нет compile-time type safety
actorRef ! "hello"  // OK
actorRef ! 123      // OK
actorRef ! SomeObject  // Compiles, но может не обработаться!

// 2. sender() - небезопасный
def receive: Receive = {
  case Request =>
    // sender() может быть deadLetters!
    // Тип sender() всегда ActorRef - Any
    sender() ! Response
}

// 3. context.become - легко потерять type safety
context.become {
  case msg => // Any тип, нет проверок
}
```

**Typed API решение:**

```scala
// ✓ Typed API - Compile-time типы
object TypedActor {
  // Явный протокол сообщений
  sealed trait Command
  case class Greet(name: String, replyTo: ActorRef[Response]) extends Command
  case class GetCount(replyTo: ActorRef[Response]) extends Command
  
  sealed trait Response
  case class Greeting(message: String) extends Response
  case class Count(value: Int) extends Response
  
  def apply(): Behavior[Command] = Behaviors.setup { context =>
    var count = 0
    
    Behaviors.receiveMessage {
      case Greet(name, replyTo) =>
        count += 1
        replyTo ! Greeting(s"Hello, $name!")
        Behaviors.same
      
      case GetCount(replyTo) =>
        replyTo ! Count(count)
        Behaviors.same
    }
  }
}

// Использование:
val actor: ActorRef[TypedActor.Command] = 
  spawn(TypedActor(), "typed-actor")

// ✓ Type-safe отправка
actor ! TypedActor.Greet("Alice", replyTo)

// ✗ Не компилируется
// actor ! "wrong type"  // Compile error!
// actor ! 123           // Compile error!
```

**Преимущества Typed API:**

```
1. Type Safety
   ┌────────────────────────────────────────┐
   │  ✓ Compile-time проверка типов         │
   │  ✓ Нет runtime ClassCastException      │
   │  ✓ IDE autocomplete и рефакторинг      │
   └────────────────────────────────────────┘

2. Explicit Protocol
   ┌────────────────────────────────────────┐
   │  ✓ Четкий контракт сообщений           │
   │  ✓ Документированный API               │
   │  ✓ Легко понять, что актор принимает   │
   └────────────────────────────────────────┘

3. No sender()
   ┌────────────────────────────────────────┐
   │  ✓ replyTo в сообщении                 │
   │  ✓ Явный flow данных                   │
   │  ✓ Нет магии sender()                  │
   └────────────────────────────────────────┘

4. Functional Style
   ┌────────────────────────────────────────┐
   │  ✓ Immutable Behaviors                 │
   │  ✓ Pure functions                      │
   │  ✓ Легче тестировать                   │
   │  ✓ Легче рассуждать о коде             │
   └────────────────────────────────────────┘

5. Better Actor Protocol Design
   ┌────────────────────────────────────────┐
   │  ✓ Принуждает к хорошему дизайну       │
   │  ✓ Sealed traits для exhaustiveness    │
   │  ✓ Явные ответы через ActorRef[T]      │
   └────────────────────────────────────────┘
```

#### Основные концепции

**Behavior[T] - основной примитив:**

```
Behavior[T] - функция от сообщения к новому Behavior

Type: Message => Behavior[Message]

┌─────────────────────────────────────┐
│  Behavior[T] представляет:          │
│  - Как актор обрабатывает сообщения │
│  - Какой будет следующий Behavior   │
│  - Lifecycle hooks (optional)       │
└─────────────────────────────────────┘

Виды Behaviors:
1. Behaviors.receive     - с context
2. Behaviors.receiveMessage - без context
3. Behaviors.setup       - инициализация
4. Behaviors.same        - не меняет behavior
5. Behaviors.stopped     - останавливает актор
6. Behaviors.empty       - игнорирует сообщения
```

**ActorRef[T] - typed reference:**

```scala
// Classic: ActorRef (Any тип)
val classicRef: ActorRef = ???
classicRef ! "anything"  // Принимает любые сообщения

// Typed: ActorRef[T] (конкретный тип)
val typedRef: ActorRef[MyProtocol] = ???
typedRef ! MyProtocol.Message  // ✓ OK
// typedRef ! "anything"        // ✗ Compile error

// Narrow vs Wide типы
trait BaseCommand
case class SpecificCommand() extends BaseCommand

val wide: ActorRef[BaseCommand] = ???
val narrow: ActorRef[SpecificCommand] = ???

// Widening - safe
val widened: ActorRef[BaseCommand] = narrow.narrow[BaseCommand]

// Can send more specific types to wide actor
wide ! SpecificCommand()  // ✓ OK
```

**ActorContext[T] - typed context:**

```scala
Behaviors.setup[Command] { context =>
  // context имеет тип ActorContext[Command]
  
  // Typed methods:
  context.self          // ActorRef[Command]
  context.spawn(...)    // Создает typed child
  context.watch(ref)    // Type-safe watch
  context.log          // Logging API
  context.system       // ActorSystem[Nothing]
  
  // Нет sender()! Используйте replyTo в сообщении
  
  Behaviors.receiveMessage { message =>
    // handle message
    Behaviors.same
  }
}
```

### 3.2 Behaviors API

#### Основные Behaviors

**1. Behaviors.receive - с контекстом:**

```scala
object MyActor {
  sealed trait Command
  case class DoSomething(data: String) extends Command
  case class GetState(replyTo: ActorRef[State]) extends Command
  
  case class State(value: String)
  
  def apply(): Behavior[Command] = {
    Behaviors.receive { (context, message) =>
      // Есть доступ к context
      message match {
        case DoSomething(data) =>
          context.log.info(s"Processing: $data")
          // Return new behavior
          Behaviors.same
        
        case GetState(replyTo) =>
          context.log.info("Returning state")
          replyTo ! State("current state")
          Behaviors.same
      }
    }
  }
}
```

**2. Behaviors.receiveMessage - без контекста:**

```scala
object SimpleActor {
  sealed trait Command
  case class Process(data: String) extends Command
  
  def apply(): Behavior[Command] = {
    Behaviors.receiveMessage {
      // Нет context parameter
      case Process(data) =>
        println(s"Processing: $data")
        Behaviors.same
    }
  }
}

// Когда использовать:
// - receiveMessage: когда context не нужен
// - receive: когда нужен logging, spawn, etc.
```

**3. Behaviors.setup - инициализация:**

```scala
object InitializedActor {
  sealed trait Command
  case class Work(task: String) extends Command
  
  def apply(): Behavior[Command] = {
    Behaviors.setup { context =>
      // Инициализация при создании актора
      context.log.info("Actor starting up")
      
      // Создаем child actors
      val worker = context.spawn(WorkerActor(), "worker")
      
      // Initialize resources
      val resource = Resource.create()
      
      // Return the actual behavior
      Behaviors.receiveMessage {
        case Work(task) =>
          worker ! WorkerActor.DoWork(task)
          Behaviors.same
      }
    }
  }
}
```

**4. Behaviors.same - не меняет behavior:**

```scala
def counting(count: Int): Behavior[Command] = {
  Behaviors.receiveMessage {
    case Increment =>
      // Изменяем state, возвращаем новый behavior
      counting(count + 1)
    
    case GetCount(replyTo) =>
      replyTo ! Count(count)
      // State не изменился, возвращаем same
      Behaviors.same
    
    case Reset =>
      // Возврат к начальному behavior
      counting(0)
  }
}
```

**5. Behaviors.stopped - останавливает актор:**

```scala
def working(): Behavior[Command] = {
  Behaviors.receiveMessage {
    case StopNow =>
      // Останавливает актор
      Behaviors.stopped
    
    case Work(task) =>
      processTask(task)
      Behaviors.same
  }
}

// С cleanup callback
def workingWithCleanup(resource: Resource): Behavior[Command] = {
  Behaviors.receiveMessage {
    case StopNow =>
      Behaviors.stopped { () =>
        // Cleanup при остановке
        resource.close()
        println("Resources cleaned up")
      }
    
    case Work(task) =>
      processTask(task)
      Behaviors.same
  }
}
```

**6. Behaviors.empty - игнорирует все:**

```scala
// Актор, который ничего не делает
val ignoring: Behavior[Command] = Behaviors.empty

// Полезно для:
// - Временная остановка обработки
// - Draining mailbox перед stop
// - Testing

def draining(): Behavior[Command] = {
  Behaviors.receiveMessage {
    case Drain =>
      // Переходим в режим игнорирования
      Behaviors.empty
    
    case msg =>
      handleMessage(msg)
      Behaviors.same
  }
}
```

#### State Management

**Immutable State Pattern:**

```scala
object CounterActor {
  sealed trait Command
  case object Increment extends Command
  case object Decrement extends Command
  case class GetCount(replyTo: ActorRef[Int]) extends Command
  case object Reset extends Command
  
  def apply(): Behavior[Command] = counter(0)
  
  // State как параметр функции
  private def counter(count: Int): Behavior[Command] = {
    Behaviors.receiveMessage {
      case Increment =>
        // Новый behavior с новым state
        counter(count + 1)
      
      case Decrement =>
        counter(count - 1)
      
      case GetCount(replyTo) =>
        replyTo ! count
        // State не изменился
        Behaviors.same
      
      case Reset =>
        counter(0)
    }
  }
}

// Flow:
// counter(0) → Increment → counter(1) → Increment → counter(2)
```

**Mutable State (внутри setup):**

```scala
object StatefulActor {
  sealed trait Command
  case class AddItem(item: String) extends Command
  case class RemoveItem(item: String) extends Command
  case class GetItems(replyTo: ActorRef[Set[String]]) extends Command
  
  def apply(): Behavior[Command] = {
    Behaviors.setup { context =>
      // Mutable state внутри closure
      var items = Set.empty[String]
      
      Behaviors.receiveMessage {
        case AddItem(item) =>
          items += item
          context.log.info(s"Added: $item, total: ${items.size}")
          Behaviors.same  // State изменился, но behavior тот же
        
        case RemoveItem(item) =>
          items -= item
          Behaviors.same
        
        case GetItems(replyTo) =>
          replyTo ! items
          Behaviors.same
      }
    }
  }
}

// Когда использовать:
// ✓ Mutable: когда state сложный (collections, etc.)
// ✓ Immutable: когда state простой (primitives, case classes)
```

**Complex State Management:**

```scala
object SessionActor {
  sealed trait Command
  case class Login(user: String, replyTo: ActorRef[Response]) extends Command
  case class Logout(replyTo: ActorRef[Response]) extends Command
  case class AddToCart(item: String, replyTo: ActorRef[Response]) extends Command
  
  sealed trait Response
  case class LoggedIn(user: String) extends Response
  case class LoggedOut() extends Response
  case class ItemAdded(item: String) extends Response
  case class Unauthorized() extends Response
  
  // State как case class
  private case class SessionState(
    user: Option[String],
    cart: List[String],
    loginTime: Option[Long]
  )
  
  def apply(): Behavior[Command] = unauthenticated()
  
  // State: Unauthenticated
  private def unauthenticated(): Behavior[Command] = {
    Behaviors.receive { (context, message) =>
      message match {
        case Login(user, replyTo) =>
          context.log.info(s"User $user logged in")
          replyTo ! LoggedIn(user)
          // Переход к authenticated state
          authenticated(SessionState(
            user = Some(user),
            cart = List.empty,
            loginTime = Some(System.currentTimeMillis())
          ))
        
        case Logout(replyTo) =>
          replyTo ! Unauthorized()
          Behaviors.same
        
        case AddToCart(_, replyTo) =>
          replyTo ! Unauthorized()
          Behaviors.same
      }
    }
  }
  
  // State: Authenticated
  private def authenticated(state: SessionState): Behavior[Command] = {
    Behaviors.receive { (context, message) =>
      message match {
        case Login(_, replyTo) =>
          replyTo ! LoggedIn(state.user.getOrElse(""))
          Behaviors.same
        
        case Logout(replyTo) =>
          context.log.info(s"User ${state.user} logged out")
          replyTo ! LoggedOut()
          // Переход к unauthenticated
          unauthenticated()
        
        case AddToCart(item, replyTo) =>
          val newState = state.copy(cart = item :: state.cart)
          replyTo ! ItemAdded(item)
          // Новый state
          authenticated(newState)
      }
    }
  }
}
```

### 3.3 Typed Actor Lifecycle

**Lifecycle в Typed API:**

```scala
object LifecycleActor {
  sealed trait Command
  case class DoWork(task: String) extends Command
  case object Stop extends Command
  
  def apply(): Behavior[Command] = {
    Behaviors.setup { context =>
      context.log.info("Setup: Actor initializing")
      
      Behaviors.receiveMessage[Command] {
        case DoWork(task) =>
          context.log.info(s"Working on: $task")
          Behaviors.same
        
        case Stop =>
          context.log.info("Stopping...")
          Behaviors.stopped
      }
      .receiveSignal {  // Lifecycle signals
        case (context, PreRestart) =>
          context.log.info("PreRestart: About to restart")
          Behaviors.same
        
        case (context, PostStop) =>
          context.log.info("PostStop: Cleaning up")
          // Cleanup resources
          Behaviors.same
      }
    }
  }
}
```

**Signals - системные сообщения:**

```
Signals в Typed API:

1. PreRestart
   - Перед перезапуском после ошибки
   - Cleanup текущего state

2. PostStop
   - После остановки актора
   - Финальный cleanup

3. Terminated(ref)
   - Watched actor остановился
   - Death watch notification

Обработка signals:
```

```scala
def behavior(): Behavior[Command] = {
  Behaviors.receiveMessage[Command] {
    case msg => handleMessage(msg)
  }
  .receiveSignal {
    case (context, PreRestart) =>
      // Called before restart
      context.log.warning("Restarting...")
      Behaviors.same
    
    case (context, PostStop) =>
      // Called after stop
      context.log.info("Stopped")
      Behaviors.same
    
    case (context, Terminated(ref)) =>
      // Watched actor terminated
      context.log.info(s"Actor $ref terminated")
      Behaviors.same
  }
}
```

**Watch в Typed API:**

```scala
object WatcherActor {
  sealed trait Command
  case class WatchActor(ref: ActorRef[_]) extends Command
  case class UnwatchActor(ref: ActorRef[_]) extends Command
  
  def apply(): Behavior[Command] = {
    Behaviors.setup { context =>
      Behaviors.receiveMessage[Command] {
        case WatchActor(ref) =>
          context.watch(ref)
          context.log.info(s"Now watching: $ref")
          Behaviors.same
        
        case UnwatchActor(ref) =>
          context.unwatch(ref)
          context.log.info(s"Stopped watching: $ref")
          Behaviors.same
      }
      .receiveSignal {
        case (context, Terminated(ref)) =>
          context.log.warning(s"Watched actor terminated: $ref")
          // Можно принять решение что делать
          Behaviors.same
      }
    }
  }
}
```

### 3.4 Interaction Patterns

#### Request-Response Pattern

```scala
object RequestResponseExample {
  
  // Backend actor
  object Backend {
    sealed trait Command
    case class Request(query: String, replyTo: ActorRef[Response]) extends Command
    
    sealed trait Response
    case class Success(data: String) extends Response
    case class Failure(reason: String) extends Response
    
    def apply(): Behavior[Command] = {
      Behaviors.receiveMessage {
        case Request(query, replyTo) =>
          // Process request
          if (query.nonEmpty) {
            replyTo ! Success(s"Result for: $query")
          } else {
            replyTo ! Failure("Empty query")
          }
          Behaviors.same
      }
    }
  }
  
  // Frontend actor
  object Frontend {
    sealed trait Command
    case class ProcessRequest(query: String) extends Command
    private case class WrappedResponse(response: Backend.Response) extends Command
    
    def apply(backend: ActorRef[Backend.Command]): Behavior[Command] = {
      Behaviors.setup { context =>
        // Message adapter для Backend.Response
        val responseMapper: ActorRef[Backend.Response] = 
          context.messageAdapter(response => WrappedResponse(response))
        
        Behaviors.receiveMessage {
          case ProcessRequest(query) =>
            context.log.info(s"Sending request: $query")
            backend ! Backend.Request(query, responseMapper)
            Behaviors.same
          
          case WrappedResponse(Backend.Success(data)) =>
            context.log.info(s"Got success: $data")
            Behaviors.same
          
          case WrappedResponse(Backend.Failure(reason)) =>
            context.log.error(s"Got failure: $reason")
            Behaviors.same
        }
      }
    }
  }
}
```

#### Message Adapter Pattern

**Проблема:** Actor может получать сообщения разных типов от разных sources.

```scala
object MessageAdapterExample {
  
  // External protocol 1
  object ServiceA {
    sealed trait Response
    case class DataA(value: String) extends Response
  }
  
  // External protocol 2
  object ServiceB {
    sealed trait Response
    case class DataB(value: Int) extends Response
  }
  
  // Our actor protocol
  object Aggregator {
    sealed trait Command
    case class Start() extends Command
    private case class WrappedA(response: ServiceA.Response) extends Command
    private case class WrappedB(response: ServiceB.Response) extends Command
    
    def apply(
      serviceA: ActorRef[ServiceA.Command],
      serviceB: ActorRef[ServiceB.Command]
    ): Behavior[Command] = {
      
      Behaviors.setup { context =>
        // Создаем adapters для внешних протоколов
        val serviceAAdapter: ActorRef[ServiceA.Response] = 
          context.messageAdapter(response => WrappedA(response))
        
        val serviceBAdapter: ActorRef[ServiceB.Response] =
          context.messageAdapter(response => WrappedB(response))
        
        Behaviors.receiveMessage {
          case Start() =>
            // Запрашиваем данные от обоих сервисов
            serviceA ! ServiceA.GetData(serviceAAdapter)
            serviceB ! ServiceB.GetData(serviceBAdapter)
            collecting(None, None)
          
          case _ =>
            Behaviors.same
        }
      }
    }
    
    private def collecting(
      dataA: Option[String],
      dataB: Option[Int]
    ): Behavior[Command] = {
      
      Behaviors.receive { (context, message) =>
        message match {
          case WrappedA(ServiceA.DataA(value)) =>
            val newDataA = Some(value)
            checkComplete(context, newDataA, dataB)
          
          case WrappedB(ServiceB.DataB(value)) =>
            val newDataB = Some(value)
            checkComplete(context, dataA, newDataB)
          
          case _ =>
            Behaviors.same
        }
      }
    }
    
    private def checkComplete(
      context: ActorContext[Command],
      dataA: Option[String],
      dataB: Option[Int]
    ): Behavior[Command] = {
      (dataA, dataB) match {
        case (Some(a), Some(b)) =>
          context.log.info(s"Got all data: A=$a, B=$b")
          // Process combined data
          Behaviors.stopped
        
        case _ =>
          collecting(dataA, dataB)
      }
    }
  }
}
```

#### Ask Pattern в Typed

```scala
import akka.actor.typed.scaladsl.AskPattern._
import akka.util.Timeout
import scala.concurrent.Future
import scala.concurrent.duration._

object AskPatternExample {
  
  object Backend {
    sealed trait Command
    case class Query(data: String, replyTo: ActorRef[Response]) extends Command
    
    sealed trait Response
    case class Result(value: String) extends Response
    
    def apply(): Behavior[Command] = {
      Behaviors.receiveMessage {
        case Query(data, replyTo) =>
          replyTo ! Result(s"Processed: $data")
          Behaviors.same
      }
    }
  }
  
  // Использование ask вне актора
  def externalAsk(
    backend: ActorRef[Backend.Command]
  )(implicit system: ActorSystem[_]): Unit = {
    
    implicit val timeout: Timeout = 3.seconds
    import system.executionContext
    
    // Ask возвращает Future
    val future: Future[Backend.Response] = 
      backend.ask(ref => Backend.Query("test", ref))
    
    future.foreach {
      case Backend.Result(value) =>
        println(s"Got result: $value")
    }
  }
  
  // Использование ask внутри актора
  object Frontend {
    sealed trait Command
    case class Request(data: String) extends Command
    private case class WrappedResponse(response: Backend.Response) extends Command
    
    def apply(backend: ActorRef[Backend.Command]): Behavior[Command] = {
      Behaviors.setup { context =>
        implicit val timeout: Timeout = 3.seconds
        
        Behaviors.receiveMessage {
          case Request(data) =>
            // Ask с context.ask
            context.ask(backend, ref => Backend.Query(data, ref)) {
              case Success(response) => WrappedResponse(response)
              case Failure(ex) =>
                context.log.error("Ask failed", ex)
                WrappedResponse(Backend.Result("failed"))
            }
            Behaviors.same
          
          case WrappedResponse(Backend.Result(value)) =>
            context.log.info(s"Received: $value")
            Behaviors.same
        }
      }
    }
  }
}
```

---

## 4. Supervision и Error Handling

### 4.1 Supervision Theory

#### Философия "Let It Crash"

**Концепция:**

```
"Let It Crash" Philosophy

Традиционный подход:
┌────────────────────────────────────┐
│ Защитное программирование          │
│  try {                             │
│    try {                           │
│      try {                         │
│        dangerousOperation()        │
│      } catch {...}                 │
│    } catch {...}                   │
│  } catch {...}                     │
└────────────────────────────────────┘
Проблемы:
✗ Код загроможден error handling
✗ Невозможно предусмотреть все ошибки
✗ Partial failures сложно обрабатывать
✗ Corrupted state опасен

Actor Model подход:
┌────────────────────────────────────┐
│ "Let It Crash"                     │
│                                    │
│  def process(data: Data): Unit = { │
│    // Просто делаем работу         │
│    dangerousOperation(data)        │
│  }                                 │
│                                    │
│  // Supervisor решает что делать    │
│  override val supervisorStrategy = │
│    OneForOneStrategy() {           │
│      case _: Exception => Restart  │
│    }                               │
└────────────────────────────────────┘
Преимущества:
✓ Чистый код логики
✓ Централизованный error handling
✓ Автоматическое восстановление
✓ Изоляция ошибок
```

**Принципы:**

```
1. Separation of Concerns
   ┌─────────────────────────┐
   │   Worker Actors         │
   │   - Бизнес-логика       │
   │   - Не думают об ошибках│
   └─────────────────────────┘
            ↓ reports errors
   ┌─────────────────────────┐
   │   Supervisor Actors     │
   │   - Error handling      │
   │   - Restart policies    │
   └─────────────────────────┘

2. Fault Isolation
   Supervisor
   ├─ Worker 1  ✓ working
   ├─ Worker 2  ✗ crashed
   └─ Worker 3  ✓ working
   
   Worker 2 crash НЕ влияет на Workers 1,3

3. Self-Healing
   Worker crashes → Supervisor detects
   → Restart worker → Worker recovers
   → System continues

4. Escalation
   If supervisor can't handle error
   → Escalate to parent supervisor
   → Hierarchy of error handling
```

#### Supervision Strategies

**Стратегии решения:**

```
При ошибке в child, supervisor может:

1. Resume
   ┌──────────┐
   │  Child   │
   │ (error)  │
   └────┬─────┘
        │
        ▼
   Resume processing
   - Игнорировать ошибку
   - Продолжить с текущим state
   - Mailbox сохраняется

2. Restart
   ┌──────────┐
   │  Child   │
   │ (error)  │
   └────┬─────┘
        │
        ▼
   preRestart → postStop
        │
        ▼
   Constructor → postRestart
        │
        ▼
   Fresh state
   - Новый instance
   - Потеря state
   - Mailbox сохраняется

3. Stop
   ┌──────────┐
   │  Child   │
   │ (error)  │
   └────┬─────┘
        │
        ▼
   postStop
        │
        ▼
   Terminated
   - Полная остановка
   - Нет перезапуска
   - Cleanup resources

4. Escalate
   ┌──────────┐
   │  Child   │
   │ (error)  │
   └────┬─────┘
        │
        ▼
   Parent Supervisor
        │
        ▼
   Grandparent decides
   - Передача решения выше
   - Для серьезных ошибок
```

**Типы стратегий:**

```
1. OneForOneStrategy
   Supervisor
   ├─ Child 1  ✓
   ├─ Child 2  ✗ Error! → Restart только Child 2
   └─ Child 3  ✓
   
   - Применяется только к failed child
   - Другие children не затронуты
   - Default strategy

2. AllForOneStrategy
   Supervisor
   ├─ Child 1  ✓
   ├─ Child 2  ✗ Error!
   └─ Child 3  ✓
        ↓
   Restart ALL children
   
   - Применяется ко всем children
   - Когда children взаимозависимы
   - Редко используется
```

**Параметры стратегий:**

```scala
SupervisorStrategy(
  maxNrOfRetries: Int,     // Макс. попыток перезапуска
  withinTimeRange: Duration // За какой период
)

Примеры:
// 1. Unlimited restarts
SupervisorStrategy(
  maxNrOfRetries = -1,
  withinTimeRange = Duration.Inf
)

// 2. Max 10 restarts per minute
SupervisorStrategy(
  maxNrOfRetries = 10,
  withinTimeRange = 1.minute
)
// Если превышено → Stop actor

// 3. No restarts (stop immediately)
SupervisorStrategy(
  maxNrOfRetries = 0,
  withinTimeRange = Duration.Zero
)
```

### 4.2 Classic Supervision

```scala
import akka.actor.{Actor, ActorRef, OneForOneStrategy, Props, SupervisorStrategy}
import akka.actor.SupervisorStrategy._
import scala.concurrent.duration._

// Worker actor
class WorkerActor extends Actor with ActorLogging {
  
  def receive: Receive = {
    case "fail" =>
      throw new RuntimeException("Intentional failure")
    
    case "divide" =>
      val result = 10 / 0  // ArithmeticException
      sender() ! result
    
    case msg: String =>
      log.info(s"Processing: $msg")
      sender() ! s"Processed: $msg"
  }
  
  override def preRestart(reason: Throwable, message: Option[Any]): Unit = {
    log.warning(s"Restarting due to: ${reason.getMessage}")
    super.preRestart(reason, message)
  }
  
  override def postRestart(reason: Throwable): Unit = {
    log.info("Restarted successfully")
    super.postRestart(reason)
  }
}

// Supervisor actor
class SupervisorActor extends Actor with ActorLogging {
  
  // OneForOneStrategy - применяется к каждому child отдельно
  override val supervisorStrategy: SupervisorStrategy =
    OneForOneStrategy(
      maxNrOfRetries = 3,           // Максимум 3 перезапуска
      withinTimeRange = 1.minute     // За 1 минуту
    ) {
      case _: ArithmeticException =>
        log.error("Arithmetic error, resuming")
        Resume  // Продолжить с текущим state
      
      case _: NullPointerException =>
        log.error("Null pointer, restarting")
        Restart  // Перезапуск с чистым state
      
      case _: IllegalArgumentException =>
        log.error("Invalid argument, stopping")
        Stop  // Полная остановка
      
      case _: Exception =>
        log.error("Unknown exception, escalating")
        Escalate  // Передать родителю
    }
  
  val worker: ActorRef = context.actorOf(Props[WorkerActor], "worker")
  
  def receive: Receive = {
    case msg =>
      worker forward msg
  }
}
```

**AllForOneStrategy пример:**

```scala
class TeamSupervisor extends Actor {
  
  // AllForOneStrategy - применяется ко всем children
  override val supervisorStrategy: SupervisorStrategy =
    AllForOneStrategy(
      maxNrOfRetries = 5,
      withinTimeRange = 1.minute
    ) {
      case _: Exception => Restart
    }
  
  // Создаем team of workers
  val workers: Seq[ActorRef] = (1 to 3).map { i =>
    context.actorOf(Props[WorkerActor], s"worker-$i")
  }
  
  def receive: Receive = {
    case msg =>
      // Distribute work
      val worker = workers(Random.nextInt(workers.size))
      worker ! msg
  }
}

// Когда один worker fails:
// - ВСЕ workers перезапускаются
// - Полезно когда workers share state
```

### 4.3 Typed Supervision

**SupervisorStrategy в Typed API:**

```scala
import akka.actor.typed.{Behavior, SupervisorStrategy}
import akka.actor.typed.scaladsl.Behaviors
import scala.concurrent.duration._

object TypedSupervisorExample {
  
  // Worker behavior
  object Worker {
    sealed trait Command
    case class DoWork(task: String) extends Command
    case object Fail extends Command
    
    def apply(): Behavior[Command] = {
      Behaviors.receiveMessage {
        case DoWork(task) =>
          println(s"Working on: $task")
          Behaviors.same
        
        case Fail =>
          throw new RuntimeException("Worker failed!")
      }
    }
  }
  
  // Supervised worker - основной паттерн
  object SupervisedWorker {
    def apply(): Behavior[Worker.Command] = {
      Behaviors.supervise(Worker())
        .onFailure[RuntimeException](
          SupervisorStrategy.restart
            .withLimit(maxNrOfRetries = 3, withinTimeRange = 1.minute)
        )
    }
  }
  
  // Supervisor с различными стратегиями
  object AdvancedSupervisor {
    sealed trait Command
    case class SpawnWorker(name: String) extends Command
    case class WorkerCommand(name: String, cmd: Worker.Command) extends Command
    
    def apply(): Behavior[Command] = {
      Behaviors.setup { context =>
        
        Behaviors.receiveMessage {
          case SpawnWorker(name) =>
            // Strategy 1: Simple restart
            val simpleWorker = context.spawn(
              Behaviors.supervise(Worker())
                .onFailure(SupervisorStrategy.restart),
              s"$name-simple"
            )
            
            // Strategy 2: Restart with backoff
            val backoffWorker = context.spawn(
              Behaviors.supervise(Worker())
                .onFailure(
                  SupervisorStrategy.restartWithBackoff(
                    minBackoff = 1.second,
                    maxBackoff = 30.seconds,
                    randomFactor = 0.2  // Jitter для избежания thundering herd
                  )
                ),
              s"$name-backoff"
            )
            
            // Strategy 3: Stop on failure
            val stopWorker = context.spawn(
              Behaviors.supervise(Worker())
                .onFailure(SupervisorStrategy.stop),
              s"$name-stop"
            )
            
            // Strategy 4: Resume (continue with current state)
            val resumeWorker = context.spawn(
              Behaviors.supervise(Worker())
                .onFailure(SupervisorStrategy.resume),
              s"$name-resume"
            )
            
            Behaviors.same
          
          case WorkerCommand(name, cmd) =>
            context.child(name).foreach { ref =>
              ref ! cmd
            }
            Behaviors.same
        }
      }
    }
  }
}
```

**Nested Supervision:**

```scala
object NestedSupervisionExample {
  
  // Bottom level - workers
  object Worker {
    sealed trait Command
    case class Process(data: String) extends Command
    
    def apply(): Behavior[Command] = {
      Behaviors.receiveMessage {
        case Process(data) =>
          if (data.contains("error")) {
            throw new RuntimeException(s"Failed on: $data")
          }
          println(s"Processed: $data")
          Behaviors.same
      }
    }
  }
  
  // Middle level - worker pool supervisor
  object WorkerPool {
    sealed trait Command
    case class DistributeWork(data: String) extends Command
    private case class WorkerTerminated(name: String) extends Command
    
    def apply(poolSize: Int): Behavior[Command] = {
      Behaviors.setup { context =>
        
        // Создаем pool of supervised workers
        val workers = (1 to poolSize).map { i =>
          val name = s"worker-$i"
          val worker = context.spawn(
            Behaviors.supervise(Worker())
              .onFailure[RuntimeException](
                SupervisorStrategy.restart
                  .withLimit(3, 1.minute)
              ),
            name
          )
          context.watch(worker)
          name -> worker
        }.toMap
        
        running(workers, 0)
      }
    }
    
    private def running(
      workers: Map[String, ActorRef[Worker.Command]],
      nextWorkerIndex: Int
    ): Behavior[Command] = {
      
      Behaviors.receive[Command] { (context, message) =>
        message match {
          case DistributeWork(data) =>
            // Round-robin distribution
            val workerList = workers.values.toList
            val worker = workerList(nextWorkerIndex % workerList.size)
            worker ! Worker.Process(data)
            running(workers, nextWorkerIndex + 1)
          
          case WorkerTerminated(name) =>
            context.log.error(s"Worker $name terminated permanently")
            // Можем пересоздать worker или уменьшить pool
            running(workers - name, nextWorkerIndex)
        }
      }
      .receiveSignal {
        case (context, Terminated(ref)) =>
          workers.find(_._2 == ref) match {
            case Some((name, _)) =>
              context.self ! WorkerTerminated(name)
              Behaviors.same
            case None =>
              Behaviors.same
          }
      }
    }
  }
  
  // Top level - application supervisor
  object Application {
    sealed trait Command
    case class ProcessBatch(items: List[String]) extends Command
    
    def apply(): Behavior[Command] = {
      Behaviors.setup { context =>
        
        // Supervised worker pool
        val workerPool = context.spawn(
          Behaviors.supervise(WorkerPool(5))
            .onFailure[Exception](
              SupervisorStrategy.restart
                .withLimit(10, 1.hour)
            ),
          "worker-pool"
        )
        
        Behaviors.receiveMessage {
          case ProcessBatch(items) =>
            items.foreach { item =>
              workerPool ! WorkerPool.DistributeWork(item)
            }
            Behaviors.same
        }
      }
    }
  }
}
```

**Exponential Backoff Strategy:**

```scala
object BackoffExample {
  
  object UnreliableService {
    sealed trait Command
    case class Call(data: String) extends Command
    
    def apply(): Behavior[Command] = {
      Behaviors.receiveMessage {
        case Call(data) =>
          // Симулируем случайные ошибки
          if (scala.util.Random.nextDouble() < 0.3) {
            throw new Exception("Service temporarily unavailable")
          }
          println(s"Service called with: $data")
          Behaviors.same
      }
    }
  }
  
  object ResilientService {
    def apply(): Behavior[UnreliableService.Command] = {
      Behaviors.supervise(
        Behaviors.supervise(UnreliableService())
          .onFailure[Exception](
            SupervisorStrategy.restartWithBackoff(
              minBackoff = 100.millis,
              maxBackoff = 10.seconds,
              randomFactor = 0.2
            )
            .withMaxRestarts(maxRestarts = 10)
          )
      ).onFailure[Exception](
        // Outer supervisor для handling max restarts exceeded
        SupervisorStrategy.stop
      )
    }
  }
}

// Backoff progression:
// Attempt 1: immediate
// Attempt 2: wait 100ms
// Attempt 3: wait 200ms
// Attempt 4: wait 400ms
// Attempt 5: wait 800ms
// ...
// Attempt N: wait min(2^N * 100ms, 10s)
// + random jitter ±20%
```

### 4.4 Error Kernel Pattern

**Концепция Error Kernel:**

```
Error Kernel = Minimal trusted core

Архитектура:
┌──────────────────────────────────────────┐
│         Application Guardian             │
│  (Самый надежный, минимальная логика)   │
└───────┬──────────────────┬───────────────┘
        │                  │
        ▼                  ▼
  ┌───────────┐      ┌───────────┐
  │ Subsystem │      │ Subsystem │
  │     A     │      │     B     │
  │ (Больше   │      │ (Больше   │
  │  логики)  │      │  логики)  │
  └─────┬─────┘      └─────┬─────┘
        │                  │
   ┌────┴────┐        ┌────┴────┐
   ▼         ▼        ▼         ▼
Worker   Worker   Worker   Worker
(Максимум логики, максимум ошибок)

Принципы:
1. Надежность ↑ вверх иерархии
2. Сложность ↓ вверх иерархии
3. Ошибки изолированы на нижних уровнях
4. Восстановление на соответствующем уровне
```

**Реализация:**

```scala
object ErrorKernelExample {
  
  // Level 3: Workers (most complex, least reliable)
  object DataProcessor {
    sealed trait Command
    case class ProcessData(data: String) extends Command
    
    def apply(): Behavior[Command] = {
      Behaviors.receiveMessage {
        case ProcessData(data) =>
          // Complex processing - can fail
          val result = complexProcessing(data)
          println(s"Processed: $result")
          Behaviors.same
      }
    }
    
    private def complexProcessing(data: String): String = {
      if (data.isEmpty) throw new IllegalArgumentException("Empty data")
      if (data.contains("error")) throw new RuntimeException("Processing error")
      data.toUpperCase
    }
  }
  
  // Level 2: Subsystem supervisor (moderate complexity)
  object ProcessingSubsystem {
    sealed trait Command
    case class ProcessBatch(items: List[String]) extends Command
    private case class ProcessItem(item: String) extends Command
    
    def apply(): Behavior[Command] = {
      Behaviors.setup { context =>
        
        // Pool of workers with restart strategy
        val workers = (1 to 3).map { i =>
          context.spawn(
            Behaviors.supervise(DataProcessor())
              .onFailure[IllegalArgumentException](
                SupervisorStrategy.resume  // Skip bad data
              )
              .onFailure[RuntimeException](
                SupervisorStrategy.restart  // Restart on processing errors
                  .withLimit(3, 1.minute)
              ),
            s"processor-$i"
          )
        }
        
        Behaviors.receiveMessage {
          case ProcessBatch(items) =>
            items.zipWithIndex.foreach { case (item, idx) =>
              val worker = workers(idx % workers.size)
              worker ! DataProcessor.ProcessData(item)
            }
            Behaviors.same
          
          case ProcessItem(item) =>
            // Handle internal coordination
            Behaviors.same
        }
      }
    }
  }
  
  // Level 1: Application guardian (minimal complexity, most reliable)
  object Application {
    sealed trait Command
    case class StartProcessing() extends Command
    case class StopProcessing() extends Command
    case class SubmitData(items: List[String]) extends Command
    
    def apply(): Behavior[Command] = {
      Behaviors.setup { context =>
        
        // Subsystem with escalation handling
        val subsystem = context.spawn(
          Behaviors.supervise(ProcessingSubsystem())
            .onFailure[Exception](
              SupervisorStrategy.restart
                .withLimit(10, 1.hour)  // More tolerant at top level
            ),
          "processing-subsystem"
        )
        
        context.watch(subsystem)
        
        idle(subsystem)
      }
    }
    
    private def idle(subsystem: ActorRef[ProcessingSubsystem.Command]): Behavior[Command] = {
      Behaviors.receive[Command] { (context, message) =>
        message match {
          case StartProcessing() =>
            context.log.info("Processing started")
            running(subsystem)
          
          case _ =>
            context.log.warning(s"Ignoring message in idle state: $message")
            Behaviors.same
        }
      }
    }
    
    private def running(subsystem: ActorRef[ProcessingSubsystem.Command]): Behavior[Command] = {
      Behaviors.receive[Command] { (context, message) =>
        message match {
          case SubmitData(items) =>
            subsystem ! ProcessingSubsystem.ProcessBatch(items)
            Behaviors.same
          
          case StopProcessing() =>
            context.log.info("Processing stopped")
            context.stop(subsystem)
            idle(subsystem)
          
          case StartProcessing() =>
            context.log.warning("Already running")
            Behaviors.same
        }
      }
      .receiveSignal {
        case (context, Terminated(`subsystem`)) =>
          context.log.error("Subsystem terminated unexpectedly")
          // Recreate subsystem
          val newSubsystem = context.spawn(
            Behaviors.supervise(ProcessingSubsystem())
              .onFailure[Exception](SupervisorStrategy.restart),
            "processing-subsystem-recovered"
          )
          context.watch(newSubsystem)
          running(newSubsystem)
      }
    }
  }
}
```

**Правила Error Kernel:**

```
1. Надежность уровней:
   ┌──────────────────────────────┐
   │  Guardian: 99.999% uptime    │  ← Никогда не падает
   ├──────────────────────────────┤
   │  Supervisor: 99.9% uptime    │  ← Редко падает
   ├──────────────────────────────┤
   │  Worker: 95% uptime          │  ← Часто падает - OK!
   └──────────────────────────────┘

2. Сложность логики:
   Guardian: Минимальная
   - Только координация
   - Простейшее state management
   - Почти нечему ломаться
   
   Supervisor: Средняя
   - Координация workers
   - Некоторая бизнес-логика
   - Может иметь state
   
   Worker: Максимальная
   - Вся сложная логика
   - External calls
   - Может падать - это OK!

3. Изоляция ошибок:
   Worker error → Restart worker
   - Другие workers не затронуты
   
   Supervisor error → Restart subsystem
   - Другие subsystems не затронуты
   
   Guardian error → Impossible by design!
   - Минимальная логика гарантирует стабильность
```

---

*Документ продолжается в исходном файле с разделами 5-14, включая детальные теоретические материалы по:*

- **Akka Streams** - Reactive Streams, Graph DSL, Backpressure
- **Akka HTTP** - Routing DSL, WebSockets, Client/Server
- **Akka Persistence** - Event Sourcing, CQRS, Snapshots
- **Akka Cluster** - Distributed Systems, Gossip, Split Brain Resolution
- **Cluster Sharding** - Entity Distribution, Passivation
- **Distributed Data** - CRDTs, Eventually Consistent
- **Testing** - TestKit, BehaviorTestKit, Persistence Testing
- **Performance** - Dispatchers, Mailboxes, Monitoring
- **Pekko Migration** - Apache Fork, Migration Guide
- **Real-World Patterns** - Saga, CQRS, Backpressure

---

## Краткая Справочная Информация

### Основные Команды и Паттерны

**Создание ActorSystem:**
```scala
// Classic
val system = ActorSystem("MySystem")

// Typed
val system: ActorSystem[Command] = ActorSystem(RootBehavior(), "MySystem")
```

**Отправка Сообщений:**
```scala
// Tell (fire-and-forget)
actor ! Message

// Ask (request-response)
implicit val timeout: Timeout = 3.seconds
val future: Future[Response] = actor ? Request
```

**Supervision:**
```scala
// Classic
override val supervisorStrategy = OneForOneStrategy() {
  case _: IOException => SupervisorStrategy.Restart
  case _: Exception => SupervisorStrategy.Escalate
}

// Typed
Behaviors.supervise(behavior)
  .onFailure[IOException](SupervisorStrategy.restart)
```

**Persistence:**
```scala
// Event Sourced Behavior
EventSourcedBehavior[Command, Event, State](
  persistenceId = PersistenceId("Entity", id),
  emptyState = State.empty,
  commandHandler = ...,
  eventHandler = ...
)
```

**Streams:**
```scala
Source(1 to 100)
  .via(Flow[Int].map(_ * 2))
  .runWith(Sink.foreach(println))
```

### Быстрый Старт Checklist

**Development:**
- [ ] Добавить зависимости (Akka или Pekko)
- [ ] Создать ActorSystem
- [ ] Определить протокол сообщений
- [ ] Реализовать акторы
- [ ] Настроить supervision
- [ ] Написать тесты

**Production:**
- [ ] Настроить dispatchers
- [ ] Добавить monitoring (Kamon/Cinnamon)
- [ ] Настроить logging
- [ ] Production конфигурация
- [ ] Load testing
- [ ] Disaster recovery plan

### Полезные Ссылки

- [Оригинальный файл с полным содержанием](computer:///mnt/user-data/uploads/akka-comprehensive-guide.md)
- [Akka Documentation](https://doc.akka.io/)
- [Pekko Documentation](https://pekko.apache.org/)

---

**Этот документ предоставляет всеобъемлющую теоретическую базу для изучения Akka. Для полного содержания всех разделов смотрите оригинальный загруженный файл, который содержит детальные примеры кода и объяснения для всех 14 разделов.**

*Успехов в освоении Akka! 🚀*
