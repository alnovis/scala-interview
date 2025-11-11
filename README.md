# План подготовки к собеседованию Senior Scala Developer

## 📋 Общая структура (4-6 недель)

**Неделя 1-2**: Основы Scala + Функциональное программирование  
**Неделя 3-4**: Продвинутые темы + Экосистема  
**Неделя 5-6**: Системный дизайн + Mock интервью

---

## 📑 Оглавление

### [🎯 Неделя 1: Основы Scala](#-неделя-1-основы-scala)

#### [День 1-2: Базовый синтаксис и концепции](#день-1-2-базовый-синтаксис-и-концепции)

**📖 Теоретические материалы:**

1. [Collections (List, Map, Set, Vector, Array, Seq)](#1-collections-list-map-set-vector-array-seq)
   - [List - неизменяемый связный список](#list---неизменяемый-связный-список)
   - [Vector - индексированная последовательность](#vector---индексированная-последовательность)
   - [Array - изменяемый массив JVM](#array---изменяемый-массив-jvm)
   - [Set - неупорядоченная коллекция уникальных элементов](#set---неупорядоченная-коллекция-уникальных-элементов)
   - [Map - коллекция пар ключ-значение](#map---коллекция-пар-ключ-значение)
   - [Seq - абстрактная последовательность](#seq---абстрактная-последовательность)
   - [Seq vs List - ключевые отличия](#seq-vs-list---ключевые-отличия)
   - [Иерархия коллекций](#иерархия-коллекций)

2. [Immutability vs Mutability](#2-immutability-vs-mutability)
   - Преимущества immutability
   - Когда использовать mutable
   - Best practices

3. [Class, Object, Trait, Sealed Trait](#3-class-object-trait-sealed-trait)
   - Class - обычный класс
   - Object - singleton
   - Trait - интерфейс с реализацией
   - Sealed Trait - закрытая иерархия
   - Сравнительная таблица
   - Когда использовать

4. [Case Classes vs Classes](#4-case-classes-vs-classes)
   - [4.1. Structural Equality vs Referential Equality](#41-structural-equality-vs-referential-equality)
   - Определения и различия
   - Переопределение equals и hashCode
   - Контракт equals/hashCode
   - Проблемы в коллекциях
   - Best practices

5. [Pattern Matching](#5-pattern-matching)
   - Базовое pattern matching
   - Exhaustiveness checking
   - Guards (условия)
   - Extractors (unapply)
   - Nested pattern matching

6. [For-Comprehensions](#6-for-comprehensions)
   - Синтаксический сахар
   - Правила трансформации
   - С Option, Either, Future

7. [Implicit и Implicit Resolution](#7-implicit-и-implicit-resolution)
   - [7.1. Implicit Values (Неявные значения)](#71-implicit-values-неявные-значения)
   - [7.2. Implicit Parameters](#72-implicit-parameters)
   - [7.3. Implicit Resolution - правила поиска](#73-implicit-resolution---правила-поиска)
   - [7.4. Implicit Resolution - детальные правила](#74-implicit-resolution---детальные-правила)
   - [7.5. Implicit Scope и Package Objects](#75-implicit-scope-и-package-objects)
   - [7.6. Implicit Priority (приоритеты наследованием)](#76-implicit-priority-приоритеты-наследованием)
   - [7.7. Debugging Implicit Resolution](#77-debugging-implicit-resolution)
   - [7.8. Best Practices для Implicit](#78-best-practices-для-implicit)
   - [7.9. Практические примеры](#79-практические-примеры)

8. [Implicit Conversions и Implicit Parameters](#8-implicit-conversions-и-implicit-parameters)
   - [8.1. Implicit conversions](#81-implicit-conversions)
   - [8.2. Implicit parameters](#82-implicit-parameters)
   - [8.3. Implicit resolution rules](#83-implicit-resolution-rules)
   - [8.4. Type Class pattern](#84-type-class-pattern-с-implicit)

9. [Type Inference и Type Annotations](#9-type-inference-и-type-annotations)
   - Вывод типов
   - Когда нужны аннотации
   - Type ascription
   - Best practices

10. [Функции apply и unapply](#10-функции-apply-и-unapply)
    - Apply - вызов объектов как функций
    - Unapply - экстракторы
    - Типы unapply
    - Симметрия apply/unapply

11. [val, var, def, lazy val](#11-val-var-def-lazy-val---способы-определения-значений)
    - [11.1. val - Immutable значение](#111-val---immutable-значение-с-eager-evaluation)
    - [11.2. var - Mutable переменная](#112-var---mutable-переменная-с-eager-evaluation)
    - [11.3. def - Метод](#113-def---метод-с-by-name-evaluation)
    - [11.4. lazy val - Ленивое значение](#114-lazy-val---ленивое-immutable-значение)
    - [11.5. Сравнение производительности](#115-сравнение-производительности)
    - [11.6. Правила выбора](#116-правила-выбора)

12. [Variance - Covariance, Contravariance, Invariance](#12-variance---covariance-contravariance-invariance)
    - [12.1. Invariance](#121-invariance-инвариантность---по-умолчанию)
    - [12.2. Covariance (+A)](#122-covariance-ковариантность---a)
    - [12.3. Contravariance (-A)](#123-contravariance-контравариантность----a)
    - [12.4. Сочетание Variance](#124-сочетание-variance)
    - [12.5. Правила безопасности](#125-правила-безопасности-variance)
    - [12.6. Когда использовать](#126-когда-использовать-какую-variance)

**Практические задачи и вопросы**

#### [День 3-4: Функциональное программирование](#день-3-4-функциональное-программирование)

**📖 Теоретические материалы:**

13. [Теория категорий для функционального программирования](#13-теория-категорий-для-функционального-программирования)
    - [13.1. Категория (Category)](#131-категория-category)
    - [13.2. Функтор (Functor)](#132-функтор-functor)
      - [Математическое определение](#математическое-определение-функтора)
      - [Диаграммы функтора](#диаграмма-функтора)
      - [Законы функтора](#законы-функтора-формально)
      - [Проверка законов](#проверка-законов-функтора)
    - [13.3. Аппликативный функтор (Applicative)](#133-аппликативный-функтор-applicative)
    - [13.4. Монада (Monad)](#134-монада-monad)
    - [13.5. Natural Transformation](#135-natural-transformation-естественное-преобразование)
    - [13.6. Monoid](#136-monoid-моноид)
    - [13.7. Semigroup](#137-semigroup-полугруппа)
    - [13.8. Связь концепций](#138-связь-концепций---диаграмма-иерархии)

14. [Higher-Order Functions](#14-higher-order-functions-функции-высшего-порядка)
    - [14.1. Функции как параметры](#141-функции-как-параметры)
    - [14.2. map - преобразование элементов](#142-map---преобразование-элементов)
    - [14.3. flatMap - преобразование с распаковкой](#143-flatmap---преобразование-с-распаковкой)
    - [14.4. fold и reduce - агрегация](#144-fold-и-reduce---агрегация)
    - [14.5. Другие higher-order functions](#145-другие-higher-order-functions)

15. [Function Composition](#15-function-composition-композиция-функций)
    - [15.1. andThen - слева направо](#151-andthen---применение-слева-направо)
    - [15.2. compose - справа налево](#152-compose---применение-справа-налево)

16. [Currying и Partial Application](#16-currying-и-partial-application)
    - [16.1. Currying - преобразование функции](#161-currying---преобразование-функции)
    - [16.2. Partial Application - частичное применение](#162-partial-application---частичное-применение)

17. [Монады (Monad)](#17-монады-monad)
    - [17.1. Option - опциональные значения](#171-option---опциональные-значения)
    - [17.2. Either - обработка ошибок](#172-either---обработка-ошибок)
    - [17.3. Try - обработка исключений](#173-try---обработка-исключений)
    - [17.4. Future - асинхронные вычисления](#174-future---асинхронные-вычисления)

18. [For-Comprehensions как syntactic sugar](#18-for-comprehensions-как-syntactic-sugar)
    - [Правила desugaring](#правила-desugaring-развертывания)
    - [Практические примеры](#практические-примеры)

19. [Recursion vs Tail Recursion](#19-recursion-vs-tail-recursion)
    - [19.1. Обычная рекурсия](#191-обычная-рекурсия)
    - [19.2. Tail Recursion (@tailrec)](#192-tail-recursion-хвостовая-рекурсия)
    - [Паттерны tail recursion](#паттерны-tail-recursion)

20. [Lazy Evaluation (Stream/LazyList)](#20-lazy-evaluation-streamlazylist)
    - [20.1. Lazy evaluation - определение](#201-lazy-evaluation---что-это)
    - [20.2. LazyList](#202-lazylist-ранее-stream-в-scala-212)
    - [20.3. Преимущества lazy evaluation](#203-преимущества-lazy-evaluation)
    - [20.4. View - lazy обертка](#204-view---lazy-обертка-над-коллекциями)
    - [20.5. Memoization](#205-memoization-в-lazylist)

**Практические задачи и вопросы**

#### [День 5-7: Type System](#день-5-7-type-system)

**📖 Теоретические материалы:**

21. [Higher-Kinded Types (HKT)](#21-higher-kinded-types-hkt---типы-высшего-порядка)
    - [Виды (Kinds) типов](#виды-kinds-типов)
    - [F[_] vs F[A]](#f_-vs-fa)
    - [Практическое применение](#практическое-применение---абстракция-над-контейнерами)
    - [Ограничения HKT в Scala](#ограничения-hkt-в-scala)

22. [Type Bounds](#22-type-bounds-границы-типов)
    - [22.1. Upper Type Bound (<:)](#221-upper-type-bound-верхняя-граница---)
    - [22.2. Lower Type Bound (>:)](#222-lower-type-bound-нижняя-граница---)
    - [22.3. Сочетание Upper и Lower bounds](#223-сочетание-upper-и-lower-bounds)
    - [22.4. View Bounds](#224-view-bounds-устаревшие-в-scala-213)

23. [Type Classes](#23-type-classes-классы-типов)
    - [23.1. Проблема, которую решают type classes](#231-проблема-которую-решают-type-classes)
    - [23.2. Решение с Type Classes](#232-решение-с-type-classes)
    - [23.3. Улучшенный синтаксис](#233-улучшенный-синтаксис-interface-syntax)
    - [23.4. Type Class Laws](#234-type-class-laws-законы)
    - [23.5. Стандартные Type Classes](#235-стандартные-type-classes)
    - [23.6. Type Classes vs Inheritance](#236-type-classes-vs-inheritance)

24. [Context Bounds](#24-context-bounds-контекстные-границы)
    - [24.1. Базовый синтаксис](#241-базовый-синтаксис)
    - [24.2. Множественные context bounds](#242-множественные-context-bounds)
    - [24.3. Context bounds с Higher-Kinded Types](#243-context-bounds-с-higher-kinded-types)
    - [24.4. Доступ к implicit instance](#244-доступ-к-implicit-instance)
    - [24.5. Context bounds в классах](#245-context-bounds-в-классах)
    - [24.6. Практический пример](#246-практический-пример---generic-сортировка)

25. [Path-Dependent Types](#25-path-dependent-types-путе-зависимые-типы)
    - [25.1. Базовый пример](#251-базовый-пример)
    - [25.2. Практический пример - Graph](#252-практический-пример---graph)
    - [25.3. Type Projection](#253-type-projection----hash)
    - [25.4. Abstract Type Members](#254-abstract-type-members)
    - [25.5. Cake Pattern](#255-cake-pattern-dependency-injection)
    - [25.6. Type Refinement](#256-type-refinement)

26. [Phantom Types](#26-phantom-types-фантомные-типы)
    - [26.1. Базовый пример - Type-safe API](#261-базовый-пример---type-safe-api)
    - [26.2. Пример - Validated Data](#262-пример---validated-data)
    - [26.3. Пример - Builder Pattern](#263-пример---builder-pattern)
    - [26.4. Пример - Units of Measure](#264-пример---units-of-measure)
    - [26.5. Преимущества Phantom Types](#265-преимущества-phantom-types)

27. [Existential Types](#27-existential-types-экзистенциальные-типы)
    - [27.1. Базовый синтаксис](#271-базовый-синтаксис)
    - [27.2. Практический пример](#272-практический-пример---heterogeneous-collections)
    - [27.3. Existential types с Type Members](#273-existential-types-с-type-members)
    - [27.4. Bounded Existentials](#274-bounded-existentials)
    - [27.5. Захват экзистенциальных типов](#275-захват-экзистенциальных-типов)
    - [27.6. Java Interop](#276-java-interop)
    - [27.7. Когда использовать](#277-когда-использовать-existential-types)
    - [27.8. Deprecation в Scala 3](#278-deprecation-в-scala-3)

**Практические задачи и вопросы**


### [🚀 Неделя 2: Scala Collections + Concurrency](#-неделя-2-scala-collections--concurrency)

#### [День 1-3: Collections Deep Dive](#день-1-3-collections-deep-dive)
- Collection hierarchy
- Performance characteristics
- View и Parallel collections
- Custom collections

#### [День 4-7: Concurrency & Futures](#день-4-7-concurrency--futures)
- Future и Promise
- ExecutionContext
- Future composition
- Error handling
- Actor model basics

### [💎 Неделя 3: Продвинутые темы](#-неделя-3-продвинутые-темы)

#### [День 1-3: Cats / Scalaz](#день-1-3-cats--scalaz)
- Semigroup, Monoid
- Functor, Applicative, Monad
- Monad Transformers
- Validated vs Either
- IO Monad
- Free Monad
- Tagless Final

#### [День 4-7: Akka / Akka Streams](#день-4-7-akka--akka-streams)
- Actor model
- Actor lifecycle
- Supervision strategies
- Akka Streams
- Backpressure
- Graph DSL
- Akka HTTP basics

### [🏗️ Неделя 4: Архитектура и паттерны](#️-неделя-4-архитектура-и-паттерны)

#### [День 1-3: Design Patterns в Scala](#день-1-3-design-patterns-в-scala)
- Creational patterns
- Structural patterns
- Behavioral patterns
- Functional patterns
- Cake pattern
- Type classes pattern

#### [День 4-7: Testing](#день-4-7-testing)
- ScalaTest
- Property-based testing
- Mocking
- Integration testing

### [🗄️ Неделя 5: Базы данных и интеграции](#️-неделя-5-базы-данных-и-интеграции)

#### [День 1-4: Database access](#день-1-4-database-access)
- Slick
- Doobie
- Connection pools
- Transactions

#### [День 5-7: Message Queues & Integration](#день-5-7-message-queues--integration)
- Kafka
- RabbitMQ
- Redis
- HTTP clients
- gRPC
- JSON (Circe, Play JSON)

### [🏛️ Неделя 6: System Design + Interview Prep](#️-неделя-6-system-design--interview-prep)

#### [День 1-3: System Design](#день-1-3-system-design)
- Microservices architecture
- Event-driven architecture
- CQRS + Event Sourcing
- CAP theorem
- Distributed transactions (Saga)
- Load balancing
- Caching strategies

#### [День 4-7: Mock Interviews](#день-4-7-mock-interviews)
- Coding Practice (LeetCode/HackerRank)
- Scala-specific tasks

### [📚 Ресурсы для изучения](#-ресурсы-для-изучения)
- Книги (Must-read)
- Online курсы
- Документация
- Practice платформы

### [🎤 Типичные вопросы на собеседовании](#-типичные-вопросы-на-собеседовании)
- Scala Basics
- Functional Programming
- Type System
- Concurrency
- Architecture
- Performance

### [✅ Checklist перед собеседованием](#-checklist-перед-собеседованием)
- За неделю до
- За день до
- В день собеседования

### [💡 Советы](#-советы)
- Coding interview
- System design
- Behavioral questions (STAR method)

### [🎯 Финальный чек-лист навыков Senior Scala Developer](#-финальный-чек-лист-навыков-senior-scala-developer)
- Must-have (обязательно)
- Nice-to-have (желательно)
- Senior-level

---

## 🎯 Неделя 1: Основы Scala

### День 1-2: Базовый синтаксис и концепции

#### 📖 Теоретические материалы

---

##### 1. Collections (List, Map, Set, Vector, Array, Seq)

###### List - неизменяемый связный список
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

###### Vector - индексированная неизменяемая последовательность
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

###### Array - изменяемый массив JVM
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

###### Set - неупорядоченная коллекция уникальных элементов
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

###### Map - коллекция пар ключ-значение
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

###### Seq - абстрактная последовательность
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

###### Seq vs List - ключевые отличия

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

###### Иерархия коллекций
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

###### 7.1. Implicit Values (Неявные значения)

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

###### 7.2. Implicit Parameters

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

###### 7.3. Implicit Resolution - правила поиска

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

###### 7.4. Implicit Resolution - детальные правила

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

###### 7.5. Implicit Scope и Package Objects

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

###### 7.6. Implicit Priority (приоритеты наследованием)

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

###### 7.7. Debugging Implicit Resolution

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

###### 7.8. Best Practices для Implicit

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

###### 7.9. Практические примеры

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

##### 8. Implicit Conversions и Implicit Parameters

###### 8.1. Implicit Conversions

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

###### 8.2. Implicit Parameters

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

###### 8.3. Implicit resolution rules
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

###### 8.4. Type Class pattern с implicit

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

##### 11. val, var, def, lazy val - Способы определения значений

**Обзор:**

В Scala существует 4 основных способа определения значений, каждый с различной семантикой оценки (evaluation) и изменяемости (mutability).

**Сравнительная таблица:**

| Ключевое слово | Оценка | Изменяемость | Тип члена | Пример |
|----------------|---------|--------------|-----------|---------|
| **val** | Eager (немедленная) | Immutable | Значение | `val x = 42` |
| **var** | Eager (немедленная) | Mutable | Переменная | `var x = 42` |
| **def** | By-name (при каждом вызове) | N/A | Метод | `def x = 42` |
| **lazy val** | Lazy (при первом обращении) | Immutable | Значение | `lazy val x = 42` |

---

###### 11.1. val - Immutable значение с eager evaluation

```scala
val x = 42
// x = 43  // ERROR! Cannot reassign val

// Вычисляется немедленно при определении
val expensive = {
  println("Computing...")
  Thread.sleep(1000)
  42
}
// Напечатает "Computing..." немедленно

// Последующие обращения используют сохраненное значение
println(expensive)  // 42, без повторного вычисления
println(expensive)  // 42, без повторного вычисления
```

**Характеристики val:**
- ✅ **Immutable**: значение нельзя изменить после инициализации
- ✅ **Eager evaluation**: вычисляется сразу при определении
- ✅ **Вычисляется один раз**: результат кешируется
- ✅ **Thread-safe**: безопасно для многопоточного доступа
- ✅ **Referential transparency**: всегда возвращает одно и то же значение

**Когда использовать val:**
```scala
// ✅ Константы
val PI = 3.14159
val MAX_SIZE = 1000

// ✅ Immutable данные
val user = User("Alice", 30)
val config = Config.load()

// ✅ Результаты вычислений, которые не меняются
val result = expensiveComputation()
val list = List(1, 2, 3)

// ✅ Функциональный стиль (предпочтительно)
val doubled = numbers.map(_ * 2)
```

---

###### 11.2. var - Mutable переменная с eager evaluation

```scala
var x = 42
x = 43  // OK, можно переназначить

// Вычисляется немедленно
var counter = {
  println("Initializing counter...")
  0
}
// Напечатает "Initializing counter..." немедленно

counter = counter + 1  // можно изменять
println(counter)  // 1
```

**Характеристики var:**
- ❌ **Mutable**: значение можно изменить
- ✅ **Eager evaluation**: вычисляется сразу
- ❌ **Not thread-safe**: требует синхронизации в многопоточной среде
- ❌ **Breaks referential transparency**: может возвращать разные значения

**Когда использовать var:**
```scala
// ✅ Локальные счетчики и аккумуляторы
def sum(numbers: List[Int]): Int = {
  var total = 0  // локальный var - OK
  for (n <- numbers) {
    total += n
  }
  total
}

// ✅ Алгоритмы с in-place updates
def bubbleSort(arr: Array[Int]): Unit = {
  var swapped = true
  while (swapped) {
    swapped = false
    // сортировка с мутацией
  }
}

// ❌ ПЛОХО: var в классе (утечка mutability)
class BadCounter {
  var count = 0  // плохо - публичный var
}

// ✅ ХОРОШО: инкапсуляция mutability
class GoodCounter {
  private var count = 0  // приватный var
  def increment(): Unit = count += 1
  def getCount: Int = count  // публичный доступ только на чтение
}
```

**Best Practice:**
```scala
// Предпочитайте val вместо var
// ❌ ПЛОХО
var sum = 0
for (x <- list) {
  sum += x
}

// ✅ ХОРОШО
val sum = list.sum

// ❌ ПЛОХО
var result = List.empty[Int]
for (x <- list) {
  result = result :+ (x * 2)
}

// ✅ ХОРОШО
val result = list.map(_ * 2)
```

---

###### 11.3. def - Метод с by-name evaluation

```scala
def x = 42
// Это метод, не значение!

// Вычисляется при КАЖДОМ обращении
def expensive = {
  println("Computing...")
  Thread.sleep(1000)
  42
}

println(expensive)  // Напечатает "Computing...", подождет 1 сек
println(expensive)  // Напечатает "Computing..." СНОВА, подождет 1 сек
```

**Характеристики def:**
- 🔄 **By-name evaluation**: вычисляется при каждом вызове
- 🔄 **No caching**: результат НЕ кешируется
- ✅ **Параметры**: может принимать параметры
- ✅ **Полиморфизм**: может быть переопределен в подклассах

**Когда использовать def:**

```scala
// ✅ Вычисления с side-effects
def currentTime: Long = System.currentTimeMillis()
def randomNumber: Int = Random.nextInt(100)

println(currentTime)  // 1699887654321
println(currentTime)  // 1699887654322 - другое значение!

// ✅ Вычисления, зависящие от изменяемого состояния
class Counter {
  private var count = 0
  
  def next: Int = {  // def, потому что возвращает новое значение каждый раз
    count += 1
    count
  }
}

val counter = new Counter()
counter.next  // 1
counter.next  // 2

// ✅ Дорогие вычисления, которые могут не понадобиться
class DataProcessor {
  def heavyComputation: Result = {
    // Вычисляется только если вызвано
    expensiveOperation()
  }
}

// ✅ Переопределяемые методы
trait Shape {
  def area: Double  // может быть переопределен
}

class Circle(radius: Double) extends Shape {
  def area: Double = math.Pi * radius * radius
}
```

**def vs val:**
```scala
class Example {
  val eagerVal = {
    println("val evaluated")
    42
  }
  
  def byNameDef = {
    println("def evaluated")
    42
  }
  
  lazy val lazyValue = {
    println("lazy val evaluated")
    42
  }
}

val ex = new Example()
// Напечатает "val evaluated" сразу

println(ex.eagerVal)  // 42, ничего не печатает
println(ex.eagerVal)  // 42, ничего не печатает

println(ex.byNameDef)  // 42, печатает "def evaluated"
println(ex.byNameDef)  // 42, печатает "def evaluated" СНОВА

println(ex.lazyValue)  // 42, печатает "lazy val evaluated"
println(ex.lazyValue)  // 42, ничего не печатает
```

---

###### 11.4. lazy val - Ленивое immutable значение

```scala
lazy val x = 42

// Вычисляется при ПЕРВОМ обращении
lazy val expensive = {
  println("Computing...")
  Thread.sleep(1000)
  42
}

// Ничего не печатается до первого обращения
println("Before access")

println(expensive)  // Напечатает "Computing...", подождет, вернет 42
println(expensive)  // 42, без печати и ожидания (результат закеширован)
```

**Характеристики lazy val:**
- 💤 **Lazy evaluation**: вычисляется при первом обращении
- ✅ **Вычисляется один раз**: результат кешируется
- ✅ **Immutable**: нельзя переназначить
- ⚠️ **Thread-safe**: с синхронизацией (небольшой overhead)
- ⚠️ **Initialization overhead**: первое обращение медленнее

**Когда использовать lazy val:**

```scala
// ✅ Дорогие вычисления, которые могут не понадобиться
class Config {
  lazy val database = {
    println("Connecting to database...")
    Database.connect()  // дорогая операция
  }
  
  lazy val cache = {
    println("Initializing cache...")
    new Cache()  // может не понадобиться
  }
}

val config = new Config()
// Ничего не инициализируется

// Используем только если нужно
if (needsDatabase) {
  config.database  // инициализируется только здесь
}

// ✅ Разрешение циклических зависимостей
class Module {
  lazy val serviceA: ServiceA = new ServiceA(serviceB)
  lazy val serviceB: ServiceB = new ServiceB(serviceA)
}

// ✅ Ленивые бесконечные структуры данных
lazy val fibonacci: LazyList[BigInt] = 
  BigInt(0) #:: BigInt(1) #:: fibonacci.zip(fibonacci.tail).map { case (a, b) => a + b }

fibonacci.take(10).toList  // List(0, 1, 1, 2, 3, 5, 8, 13, 21, 34)

// ✅ Избежание null во время инициализации
trait Component {
  lazy val dependency: Dependency  // инициализируется позже
}
```

**lazy val в классах:**
```scala
class ExpensiveResource {
  println("Creating resource")
  
  val eagerField = {
    println("Eager field initialized")
    computeExpensive()
  }
  
  lazy val lazyField = {
    println("Lazy field initialized")
    computeExpensive()
  }
}

// При создании:
val resource = new ExpensiveResource()
// Напечатает:
// Creating resource
// Eager field initialized

// При первом обращении:
resource.lazyField
// Напечатает:
// Lazy field initialized
```

---

###### 11.5. Сравнение производительности

```scala
import scala.concurrent.duration._

// Бенчмарк
def benchmark[A](name: String)(f: => A): Unit = {
  val start = System.nanoTime()
  f
  val end = System.nanoTime()
  println(s"$name: ${(end - start).nanos.toMillis} ms")
}

class PerformanceTest {
  val valField = expensiveComputation()      // вычисляется при создании
  lazy val lazyField = expensiveComputation() // вычисляется при обращении
  def defField = expensiveComputation()       // вычисляется при каждом обращении
}

benchmark("Creating object") {
  new PerformanceTest()
  // val: дорого (вычисляется сразу)
  // lazy val: дешево (не вычисляется)
  // def: дешево (не вычисляется)
}

val obj = new PerformanceTest()

benchmark("First access") {
  obj.lazyField
  // val: дешево (уже вычислено)
  // lazy val: дорого (вычисляется сейчас) + overhead синхронизации
  // def: дорого (вычисляется)
}

benchmark("Second access") {
  obj.lazyField
  // val: дешево (используется кеш)
  // lazy val: дешево (используется кеш)
  // def: дорого (вычисляется снова!)
}
```

---

###### 11.6. Правила выбора

```scala
// Используйте val по умолчанию
val x = 42  // ✅ DEFAULT CHOICE

// Используйте var только когда действительно нужна мутация
var counter = 0  // ⚠️ Только если необходимо

// Используйте def для:
// - Вычислений с side effects
// - Параметризованных методов
// - Переопределяемых членов
def currentTime = System.currentTimeMillis()  // ✅ Side effect
def add(x: Int, y: Int) = x + y               // ✅ Параметры

// Используйте lazy val для:
// - Дорогих вычислений, которые могут не понадобиться
// - Разрешения циклических зависимостей
// - Ленивой инициализации ресурсов
lazy val config = loadHeavyConfig()  // ✅ Может не понадобиться
```

**Decision Tree (Дерево решений):**
```
Нужна мутация?
├─ Да → var (но лучше подумайте дважды!)
└─ Нет
   ├─ Дорогое вычисление, которое может не понадобиться?
   │  └─ Да → lazy val
   └─ Нет
      ├─ Нужно вычислять каждый раз (side effects)?
      │  └─ Да → def
      └─ Нет → val (DEFAULT)
```

**Примеры из реальной практики:**
```scala
class WebService {
  // val - конфигурация, загружается сразу
  val config: Config = Config.load()
  
  // lazy val - пул соединений, может не понадобиться в тестах
  lazy val connectionPool: ConnectionPool = ConnectionPool.create()
  
  // def - timestamp для каждого запроса
  def requestId: String = UUID.randomUUID().toString
  
  // var - счетчик запросов (приватный!)
  private var requestCount: Int = 0
  
  def handleRequest(request: Request): Response = {
    requestCount += 1  // мутация
    
    val userId = request.userId  // immutable локальная переменная
    val user = userService.getUser(userId)
    
    Response.success(user)
  }
}
```

---

##### 12. Variance - Covariance, Contravariance, Invariance

**Что такое Variance?**

Variance (вариантность) определяет, как параметризованные типы связаны с отношениями наследования их параметров.

Если `Dog <: Animal` (Dog является подтипом Animal), то:
- **Covariant** `+A`: `Container[Dog] <: Container[Animal]` ✅
- **Contravariant** `-A`: `Container[Animal] <: Container[Dog]` ✅ (обратное!)
- **Invariant** `A`: нет связи между `Container[Dog]` и `Container[Animal]` ❌

**Синтаксис в Scala:**
```scala
class Covariant[+A]      // ковариантный
class Contravariant[-A]  // контравариантный  
class Invariant[A]       // инвариантный (по умолчанию)
```

---

###### 12.1. Invariance (Инвариантность) - по умолчанию

```scala
class Box[A](val value: A)

class Animal
class Dog extends Animal
class Cat extends Animal

val dogBox: Box[Dog] = new Box(new Dog())
val animalBox: Box[Animal] = new Box(new Animal())

// ❌ НЕ КОМПИЛИРУЕТСЯ
// val animalBox2: Box[Animal] = dogBox
// Type mismatch: found Box[Dog], required Box[Animal]

// Box инвариантен - нет связи между Box[Dog] и Box[Animal]
```

**Почему инвариантность по умолчанию?**

```scala
class MutableBox[A](var value: A) {
  def set(newValue: A): Unit = value = newValue
  def get: A = value
}

// Если бы MutableBox был ковариантным:
val dogBox = new MutableBox[Dog](new Dog())
val animalBox: MutableBox[Animal] = dogBox  // предположим, что компилируется

// Мы могли бы положить Cat в Box[Dog]!
animalBox.set(new Cat())  // ❌ Нарушение типобезопасности!

// Теперь dogBox содержит Cat вместо Dog
val dog: Dog = dogBox.get  // ❌ Runtime exception!
```

**Когда использовать инвариантность:**
- Mutable контейнеры
- Типы, которые и читают, и пишут значение параметра типа

```scala
class MutableList[A] {  // Invariant - правильно
  def add(elem: A): Unit = ???
  def get(index: Int): A = ???
}

class Array[A] {  // В Scala Array инвариантен (в отличие от Java)
  def update(index: Int, elem: A): Unit = ???
  def apply(index: Int): A = ???
}
```

---

###### 12.2. Covariance (Ковариантность) - +A

```scala
class ReadOnlyBox[+A](private val value: A) {
  def get: A = value
  // НЕ МОЖЕМ: def set(newValue: A): Unit = ???
  // Ковариантный параметр не может появляться в contravariant position
}

val dogBox: ReadOnlyBox[Dog] = new ReadOnlyBox(new Dog())
val animalBox: ReadOnlyBox[Animal] = dogBox  // ✅ КОМПИЛИРУЕТСЯ!

// Можем читать как Animal
val animal: Animal = animalBox.get  // ✅ Dog является Animal
```

**Правило ковариантности:**

Если `A <: B`, то `F[A] <: F[B]` для ковариантного `F[+T]`

```
     Dog <: Animal
         ↓
Box[Dog] <: Box[Animal]
```

**Позиции параметра типа:**

```scala
class Example[+A] {
  // ✅ COVARIANT POSITION (output) - OK
  def get: A = ???
  def produce(): A = ???
  def container: List[A] = ???
  
  // ❌ CONTRAVARIANT POSITION (input) - ERROR!
  // def set(value: A): Unit = ???
  // def consume(value: A): Unit = ???
  
  // ✅ WORKAROUND: Lower type bound
  def set[B >: A](value: B): Unit = ???
}
```

**Примеры ковариантных типов в Scala:**

```scala
// List[+A] - ковариантный
val dogs: List[Dog] = List(new Dog())
val animals: List[Animal] = dogs  // ✅ OK

// Option[+A] - ковариантный
val dogOpt: Option[Dog] = Some(new Dog())
val animalOpt: Option[Animal] = dogOpt  // ✅ OK

// Vector[+A] - ковариантный
val dogVec: Vector[Dog] = Vector(new Dog())
val animalVec: Vector[Animal] = dogVec  // ✅ OK

// Immutable collections - обычно ковариантны
```

**Практический пример:**

```scala
sealed trait Animal {
  def name: String
}
case class Dog(name: String) extends Animal
case class Cat(name: String) extends Animal

// Ковариантный sealed trait
sealed trait AnimalList[+A] {
  def head: A
  def tail: AnimalList[A]
}

case object EmptyList extends AnimalList[Nothing] {
  def head = throw new NoSuchElementException
  def tail = throw new NoSuchElementException
}

case class NonEmpty[+A](head: A, tail: AnimalList[A]) extends AnimalList[A]

// Использование
val dogs: AnimalList[Dog] = NonEmpty(Dog("Rex"), EmptyList)
val animals: AnimalList[Animal] = dogs  // ✅ Ковариантность работает!

def printAnimals(animals: AnimalList[Animal]): Unit = {
  if (animals != EmptyList) {
    println(animals.head.name)
    printAnimals(animals.tail)
  }
}

printAnimals(dogs)  // ✅ Можем передать List[Dog] как List[Animal]
```

**Lower type bounds для методов:**

```scala
class ImmutableList[+A] {
  // ❌ Не компилируется:
  // def prepend(elem: A): ImmutableList[A] = ???
  
  // ✅ Компилируется с lower bound:
  def prepend[B >: A](elem: B): ImmutableList[B] = ???
}

val dogList: ImmutableList[Dog] = ???
val newList = dogList.prepend(new Cat())  // B = Animal
// newList: ImmutableList[Animal]
```

---

###### 12.3. Contravariance (Контравариантность) - -A

```scala
trait Printer[-A] {
  def print(value: A): Unit
}

val animalPrinter: Printer[Animal] = new Printer[Animal] {
  def print(animal: Animal): Unit = println(s"Animal: ${animal.name}")
}

val dogPrinter: Printer[Dog] = animalPrinter  // ✅ КОМПИЛИРУЕТСЯ!

// Это безопасно, потому что:
dogPrinter.print(new Dog("Rex"))
// Вызовет animalPrinter.print(new Dog("Rex"))
// Dog является Animal, поэтому все OK!
```

**Правило контравариантности:**

Если `A <: B`, то `F[B] <: F[A]` для контравариантного `F[-T]` (обратное направление!)

```
        Dog <: Animal
             ↓
Printer[Animal] <: Printer[Dog]  (обратное!)
```

**Интуиция:**

Контравариантность применяется к "потребителям" (consumers) значений.

Если функция может работать с `Animal`, она точно сможет работать с `Dog` (Dog это тоже Animal).

```scala
trait Function1[-T, +R] {  // T контравариантен, R ковариантен
  def apply(t: T): R
}

// Dog => String  может использоваться как  Animal => String
val dogToString: Dog => String = _.name
val animalToString: Animal => String = dogToString  // ✅ OK
```

**Позиции параметра типа:**

```scala
class Example[-A] {
  // ✅ CONTRAVARIANT POSITION (input) - OK
  def consume(value: A): Unit = ???
  def process(f: A => Unit): Unit = ???
  
  // ❌ COVARIANT POSITION (output) - ERROR!
  // def produce(): A = ???
  // def get: A = ???
  
  // ✅ WORKAROUND: Upper type bound
  def produce[B <: A](): B = ???
}
```

**Примеры контравариантных типов:**

```scala
// Ordering[-T] - контравариантный
val animalOrdering: Ordering[Animal] = Ordering.by(_.name)
val dogOrdering: Ordering[Dog] = animalOrdering  // ✅ OK

val dogs = List(Dog("Rex"), Dog("Max"))
dogs.sorted(dogOrdering)  // Используем Animal ordering для Dog

// CanEqual[-L, -R] в Scala 3
// JsonWriter[-A]
// Serializer[-A]
```

**Практический пример - JSON Writer:**

```scala
trait JsonWriter[-A] {
  def write(value: A): Json
}

val animalWriter: JsonWriter[Animal] = new JsonWriter[Animal] {
  def write(animal: Animal): Json = 
    Json.obj("name" -> animal.name, "type" -> "animal")
}

// Можем использовать для Dog
val dogWriter: JsonWriter[Dog] = animalWriter  // ✅ Контравариантность

dogWriter.write(new Dog("Rex"))
// Вызовет animalWriter.write(Dog("Rex")) - безопасно!

// Функция принимающая JsonWriter[Dog]
def serializeDogs(dogs: List[Dog], writer: JsonWriter[Dog]): List[Json] =
  dogs.map(writer.write)

// Можем передать JsonWriter[Animal]
serializeDogs(List(Dog("Rex")), animalWriter)  // ✅ OK
```

---

###### 12.4. Сочетание Variance

```scala
// Function1 использует оба вида variance
trait Function1[-T, +R] {
  def apply(t: T): R
}

// Если Dog <: Animal и String <: Any, то:
// (Animal => String) <: (Dog => Any)
//    ↑          ↑
//    |          +---- Covariant (выходной тип)
//    +--------------- Contravariant (входной тип)

val f1: Animal => String = (a: Animal) => a.name
val f2: Dog => Any = f1  // ✅ OK

// f2 может принимать Dog (Dog <: Animal)
// f2 может возвращать Any (String <: Any)
```

**Список сортировки:**

```scala
class List[+A] {
  // Метод sorted требует Ordering
  def sorted[B >: A](implicit ord: Ordering[B]): List[B] = ???
  //         ↑ lower bound
  //         Ordering контравариантен
}

val dogs: List[Dog] = List(Dog("Rex"), Dog("Max"))

implicit val animalOrdering: Ordering[Animal] = Ordering.by(_.name)

dogs.sorted  // ✅ Использует Ordering[Animal] для List[Dog]
// Работает благодаря:
// - List ковариантен (+A)
// - Ordering контравариантен (-T)
// - Lower bound B >: A
```

---

###### 12.5. Правила безопасности variance

**Liskov Substitution Principle (LSP):**

Если `S <: T`, то объект типа `T` может быть заменен объектом типа `S` без изменения корректности программы.

```scala
class Animal {
  def makeSound(): Unit = println("Some sound")
}

class Dog extends Animal {
  override def makeSound(): Unit = println("Woof!")
  def wagTail(): Unit = println("Wagging tail")
}

// Covariant - безопасно для read-only
class AnimalShelter[+A <: Animal](private val animal: A) {
  def getAnimal: A = animal  // ✅ Безопасно
}

val dogShelter: AnimalShelter[Dog] = new AnimalShelter(new Dog())
val animalShelter: AnimalShelter[Animal] = dogShelter  // ✅ Ковариантность

val animal: Animal = animalShelter.getAnimal  // ✅ Dog является Animal
animal.makeSound()  // ✅ Работает
```

**Heap pollution в Java (почему Array не должен быть ковариантным):**

```scala
// В Java массивы ковариантны - это ОШИБКА дизайна
// В Scala Array инвариантен - это правильно!

// Если бы Array был ковариантным (как в Java):
val dogs: Array[Dog] = Array(new Dog())
val animals: Array[Animal] = dogs  // В Java это OK, в Scala - ERROR!

animals(0) = new Cat()  // В Java: ArrayStoreException в runtime!
                        // В Scala: не компилируется - ERROR!

val dog: Dog = dogs(0)  // Теперь здесь Cat!
```

---

###### 12.6. Когда использовать какую variance

**Используйте Covariance (+A) когда:**
```scala
// ✅ Immutable контейнеры (только чтение)
class ImmutableList[+A]
class Option[+A]
class Try[+A]
class Future[+A]

// ✅ Producers (производители значений)
trait Producer[+A] {
  def produce(): A
}

// ✅ Return types в методах
def getAnimals: List[Animal] = List(new Dog(), new Cat())
```

**Используйте Contravariance (-A) когда:**
```scala
// ✅ Consumers (потребители значений)
trait Consumer[-A] {
  def consume(value: A): Unit
}

trait Ordering[-A]
trait JsonWriter[-A]
trait Serializer[-A]

// ✅ Параметры функций
def processAnimals(processor: Animal => Unit): Unit = ???
```

**Используйте Invariance (A) когда:**
```scala
// ✅ Mutable контейнеры
class Array[A]
class MutableList[A]

// ✅ Типы, которые и читают, и пишут
class Buffer[A] {
  def write(elem: A): Unit = ???
  def read(): A = ???
}

// ✅ Сомневаетесь - используйте invariance (безопасно по умолчанию)
```

**Mnemonic (мнемоника):**

- **+** (Plus/Positive) = **Producer** = Covariant = "gives out" (отдает)
- **-** (Minus/Negative) = **Consumer** = Contravariant = "takes in" (принимает)
- **No sign** = **Both** = Invariant = "both reads and writes" (и читает, и пишет)

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

- [Разница между val, var, def, lazy val?](#11-val-var-def-lazy-val---способы-определения-значений)
- [Что такое contravariance и covariance? Когда использовать +A и -A?](#12-variance---covariance-contravariance-invariance)
- [Как работает implicit resolution?](#72-implicit-resolution-разрешение-неявных-значений)
- [Разница между Seq, IndexedSeq, LinearSeq?](#1-collections-list-map-set-vector-array) (см. раздел Seq)

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

---

#### 📖 Теоретические материалы

---

##### 13. Теория категорий для функционального программирования

**Введение:**

Теория категорий - математическая дисциплина, изучающая абстрактные структуры и связи между ними. В функциональном программировании она предоставляет формальную основу для понимания композиции, абстракции и полиморфизма.

**Зачем теория категорий в Scala?**
- Понимание библиотек типа Cats и Scalaz
- Композиция и переиспользование кода
- Reasoning о программах на высоком уровне абстракции
- Гарантии корректности через типы
- Паттерны функционального программирования

---

###### 13.1. Категория (Category)

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

###### 13.2. Функтор (Functor)

**Математическое определение функтора:**

Функтор F из категории C в категорию D - это отображение, состоящее из двух компонентов:

1. **Отображение объектов**: Для каждого объекта A в C существует объект F(A) в D
2. **Отображение морфизмов**: Для каждого морфизма f: A → B в C существует морфизм F(f): F(A) → F(B) в D

Функтор должен удовлетворять двум аксиомам:
- **Сохранение identity**: F(idₐ) = id_F(A)
- **Сохранение композиции**: F(g ∘ f) = F(g) ∘ F(f)

**Диаграмма функтора:**

```
Категория C                    Категория D
                              
    A --------f------> B           F(A) -----F(f)-----> F(B)
    |                  |            |                     |
    |                  |     F      |                     |
   idₐ               idᵦ    =>    id_F(A)             id_F(B)
    |                  |            |                     |
    ↓                  ↓            ↓                     ↓
    A                  B           F(A)                 F(B)


Композиция морфизмов:

    A ----f----> B ----g----> C
         
    A ----------g∘f---------> C

Функтор сохраняет композицию:

Категория C:              Категория D:

    A ----f----> B              F(A) ----F(f)----> F(B)
              ∖  |                              ∖    |
            g∘f  | g                      F(g∘f)  | F(g)
                ∖|                                ∖  |
                 C                                 F(C)

F(g ∘ f) = F(g) ∘ F(f)
```

**В программировании:**

Функтор - это type constructor `F[_]` с операцией `map`, которая применяет функцию к значению внутри контейнера.

В контексте Scala:
- **Категория C** = категория типов Scala
- **Категория D** = та же категория (эндофунктор)
- **Объект A** = тип A (Int, String, User, etc.)
- **F(A)** = параметризованный тип F[A] (Option[A], List[A], etc.)
- **Морфизм f: A → B** = функция f: A => B
- **F(f): F(A) → F(B)** = функция map, которая применяет f внутри контекста F

**Trait Functor:**
```scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

**Законы функтора (формально):**

**1. Закон Identity (тождества):**
```scala
// Применение identity функции через map не должно изменять структуру
map(fa)(identity) == fa

// Или в инфиксной нотации:
fa.map(x => x) == fa
fa.map(identity) == fa

// Математически:
F(idₐ) = id_F(A)
```

**Пример:**
```scala
val list = List(1, 2, 3)

list.map(x => x) == list                    // true
list.map(identity) == list                  // true

val option = Some(42)
option.map(x => x) == option                // true

// Identity не меняет структуру:
List(1, 2, 3).map(identity)                 // List(1, 2, 3)
Some(42).map(identity)                      // Some(42)
None.map(identity)                          // None
```

**2. Закон Composition (композиции):**
```scala
// Последовательное применение двух функций должно быть эквивалентно
// применению их композиции
map(map(fa)(f))(g) == map(fa)(f andThen g)

// Или:
fa.map(f).map(g) == fa.map(f andThen g)
fa.map(f).map(g) == fa.map(x => g(f(x)))

// Математически:
F(g ∘ f) = F(g) ∘ F(f)
```

**Пример:**
```scala
val list = List(1, 2, 3)
val f: Int => Int = _ * 2        // умножить на 2
val g: Int => String = _.toString // преобразовать в строку

// Две последовательные map
val result1 = list.map(f).map(g)  
// List("2", "4", "6")

// Одна map с композицией
val result2 = list.map(f andThen g)
// List("2", "4", "6")

result1 == result2  // true

// Визуализация:
//     map(f)        map(g)
// List(1,2,3) -> List(2,4,6) -> List("2","4","6")
//
// эквивалентно:
//         map(f andThen g)
// List(1,2,3) -----------------> List("2","4","6")
```

**Диаграмма законов в коде:**

```
Закон Identity:

    F[A] ----map(id)----> F[A]
     |                     |
     |                     |
    id_F[A]              id_F[A]
     |                     |
     ↓                     ↓
    F[A] =============== F[A]


Закон Composition:

                 f              g
           A -------> B -------> C
           
           A ---------g∘f-------> C

Применяем F:

              map(f)        map(g)
       F[A] --------> F[B] --------> F[C]
        |                             |
        |        map(g∘f)             |
        +---------------------------+
        
       F[A].map(f).map(g) === F[A].map(f andThen g)
```

**Проверка законов в коде:**
```scala
import org.scalacheck.Prop.forAll
import org.scalacheck.Properties

object FunctorLaws extends Properties("Functor") {
  
  // Закон Identity для List
  property("identity law for List") = forAll { (list: List[Int]) =>
    val id = (x: Int) => x
    list.map(id) == list
  }
  
  // Закон Composition для List
  property("composition law for List") = forAll { (list: List[Int]) =>
    val f = (x: Int) => x * 2
    val g = (x: Int) => x + 1
    
    val left = list.map(f).map(g)
    val right = list.map(f andThen g)
    
    left == right
  }
  
  // Закон Identity для Option
  property("identity law for Option") = forAll { (opt: Option[Int]) =>
    opt.map(identity) == opt
  }
  
  // Закон Composition для Option
  property("composition law for Option") = forAll { (opt: Option[String]) =>
    val f = (s: String) => s.toUpperCase
    val g = (s: String) => s.length
    
    opt.map(f).map(g) == opt.map(f andThen g)
  }
}
```

**Интуиция за законами:**

**Identity закон** гарантирует, что функтор не добавляет никакой "дополнительной логики" при применении identity функции. Он просто честно применяет функцию к содержимому.

**Composition закон** гарантирует, что порядок применения map не имеет значения с точки зрения эффективности и результата. Это позволяет оптимизировать код через fusion (слияние последовательных map в одну).

**Почему законы важны:**

```scala
// Без законов функтора мы не могли бы делать такие оптимизации:

// Неоптимальный код:
list.map(f).map(g).map(h)  // 3 прохода по списку

// Оптимизированный код (благодаря закону композиции):
list.map(f andThen g andThen h)  // 1 проход по списку

// Компилятор или библиотека может автоматически применить эту оптимизацию,
// зная что функтор соблюдает законы
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

###### 13.3. Аппликативный функтор (Applicative)

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

---

###### 13.4. Монада (Monad)

**Математическое определение:** Монада в категории C - это тройка (T, η, μ) где:
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

**Определение:**

Монада - это аппликативный функтор с операцией `flatMap` (или `bind`), которая позволяет последовательно комбинировать вычисления в контексте.

**Законы монады:**
```scala
// 1. Левая идентичность: pure(a).flatMap(f) == f(a)
pure(42).flatMap(x => Some(x + 1)) == Some(43)

// 2. Правая идентичность: m.flatMap(pure) == m
Some(42).flatMap(x => pure(x)) == Some(42)

// 3. Ассоциативность: m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g))
Some(42).flatMap(f).flatMap(g) == Some(42).flatMap(x => f(x).flatMap(g))
```

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

---

###### 13.5. Natural Transformation (Естественное преобразование)

**Определение 1:** 
Натуральное преобразование η между функторами F и G - это семейство морфизмов:
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

**Определение 2:**
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

**В Scala:**
```scala
// Natural transformation as ~> (type lambda)
trait ~>[F[_], G[_]]:
  def apply[A](fa: F[A]): G[A]
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

###### 13.6. Monoid (Моноид)

**Математическое определение:** Моноид - это алгебраическая структура (M, •, e) где:
- M - множество
- • - бинарная ассоциативная операция: M × M → M
- e - нейтральный элемент

**Определение:**
Моноид - это алгебраическая структура с:
1. Бинарной ассоциативной операцией `combine`
2. Нейтральным элементом `empty`

**Законы моноида:**
1. Ассоциативность: `combine(x, combine(y, z)) == combine(combine(x, y), z)`
2. Идентичность: `combine(x, empty) == x == combine(empty, x)`

```scala
trait Monoid[A]:
  def empty: A                        // нейтральный элемент
  def combine(x: A, y: A): A          // бинарная операция

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

###### 13.7. Semigroup (Полугруппа)

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

###### 13.8. Связь концепций - диаграмма иерархии

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


##### 14. Higher-Order Functions (Функции высшего порядка)

**Определение:**

Higher-order function (функция высшего порядка) - это функция, которая:
1. Принимает другие функции как параметры, ИЛИ
2. Возвращает функцию как результат

###### 14.1. Функции как параметры

```scala
// Функция, принимающая другую функцию
def applyTwice(f: Int => Int, x: Int): Int = f(f(x))

val double = (x: Int) => x * 2
applyTwice(double, 3)  // double(double(3)) = double(6) = 12

// Более сложный пример
def repeat(n: Int)(action: => Unit): Unit = {
  (1 to n).foreach(_ => action)
}

repeat(3) {
  println("Hello!")
}
// Напечатает "Hello!" три раза
```

###### 14.2. map - преобразование элементов

```scala
// Сигнатура: def map[B](f: A => B): List[B]

val numbers = List(1, 2, 3, 4, 5)

// Преобразование каждого элемента
numbers.map(_ * 2)           // List(2, 4, 6, 8, 10)
numbers.map(_.toString)      // List("1", "2", "3", "4", "5")
numbers.map(x => x * x)      // List(1, 4, 9, 16, 25)

// map сохраняет структуру
Some(42).map(_ * 2)          // Some(84)
None.map(_ * 2)              // None

Right(42).map(_ * 2)         // Right(84)
Left("error").map(_ * 2)     // Left("error")

// map для Future
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

val futureNumber: Future[Int] = Future(42)
futureNumber.map(_ * 2)      // Future[Int] = 84 (когда завершится)
```

**Реализация map для своего типа:**

```scala
sealed trait Maybe[+A] {
  def map[B](f: A => B): Maybe[B] = this match {
    case Just(value) => Just(f(value))
    case Empty => Empty
  }
}
case class Just[A](value: A) extends Maybe[A]
case object Empty extends Maybe[Nothing]

Just(42).map(_ * 2)    // Just(84)
Empty.map(_ * 2)       // Empty
```

---

###### 14.3. flatMap - преобразование с "распаковкой"

```scala
// Сигнатура: def flatMap[B](f: A => F[B]): F[B]

val numbers = List(1, 2, 3)

// map создает вложенную структуру
numbers.map(x => List(x, x * 10))
// List(List(1, 10), List(2, 20), List(3, 30))

// flatMap "уплощает" результат
numbers.flatMap(x => List(x, x * 10))
// List(1, 10, 2, 20, 3, 30)

// flatMap = map + flatten
numbers.map(x => List(x, x * 10)).flatten
// List(1, 10, 2, 20, 3, 30)
```

**flatMap с Option:**

```scala
def parseIntOpt(s: String): Option[Int] = 
  try Some(s.toInt) catch { case _: Exception => None }

val stringOpt: Option[String] = Some("42")

// map создает Option[Option[Int]]
stringOpt.map(parseIntOpt)
// Some(Some(42))

// flatMap "распаковывает" внутренний Option
stringOpt.flatMap(parseIntOpt)
// Some(42)

val invalidOpt: Option[String] = Some("invalid")
invalidOpt.flatMap(parseIntOpt)
// None
```

**Цепочка операций с flatMap:**

```scala
case class User(id: Long, name: String)
case class Order(userId: Long, total: Double)

def getUser(id: Long): Option[User] = ???
def getOrders(user: User): Option[List[Order]] = ???
def calculateTotal(orders: List[Order]): Option[Double] = ???

// Вложенные flatMap
val result: Option[Double] = 
  getUser(1).flatMap { user =>
    getOrders(user).flatMap { orders =>
      calculateTotal(orders)
    }
  }

// Или с for-comprehension (синтаксический сахар для flatMap)
val result2: Option[Double] = for {
  user <- getUser(1)
  orders <- getOrders(user)
  total <- calculateTotal(orders)
} yield total
```

---

###### 14.4. fold и reduce - агрегация

**fold - свертка с начальным значением:**

```scala
// foldLeft: (B, (B, A) => B) => B
// Обходит слева направо

val numbers = List(1, 2, 3, 4, 5)

// Сумма
numbers.foldLeft(0)(_ + _)
// 0 + 1 = 1
// 1 + 2 = 3
// 3 + 3 = 6
// 6 + 4 = 10
// 10 + 5 = 15

// Произведение
numbers.foldLeft(1)(_ * _)  // 120

// Конкатенация строк
val words = List("Hello", "World", "!")
words.foldLeft("")(_ + " " + _)  // " Hello World !"

// foldRight: (B, (A, B) => B) => B
// Обходит справа налево
numbers.foldRight(0)(_ + _)
// 5 + 0 = 5
// 4 + 5 = 9
// 3 + 9 = 12
// 2 + 12 = 14
// 1 + 14 = 15

// Разница видна в порядке операций
List(1, 2, 3).foldLeft(0)(_ - _)   // ((0 - 1) - 2) - 3 = -6
List(1, 2, 3).foldRight(0)(_ - _)  // 1 - (2 - (3 - 0)) = 2
```

**Практические примеры fold:**

```scala
// Подсчет слов
val text = List("hello", "world", "scala", "hello")
val wordCount: Map[String, Int] = 
  text.foldLeft(Map.empty[String, Int]) { (acc, word) =>
    acc.updated(word, acc.getOrElse(word, 0) + 1)
  }
// Map("hello" -> 2, "world" -> 1, "scala" -> 1)

// Reverse списка
def reverse[A](list: List[A]): List[A] =
  list.foldLeft(List.empty[A])((acc, elem) => elem :: acc)

reverse(List(1, 2, 3, 4))  // List(4, 3, 2, 1)

// Filter через fold
def filter[A](list: List[A])(p: A => Boolean): List[A] =
  list.foldRight(List.empty[A]) { (elem, acc) =>
    if (p(elem)) elem :: acc else acc
  }

filter(List(1, 2, 3, 4, 5))(_ % 2 == 0)  // List(2, 4)
```

**reduce - свертка без начального значения:**

```scala
// reduce использует первый элемент как начальное значение

val numbers = List(1, 2, 3, 4, 5)

numbers.reduce(_ + _)   // 15
numbers.reduce(_ * _)   // 120

// Эквивалентно:
// numbers.tail.foldLeft(numbers.head)(_ + _)

// ⚠️ ОСТОРОЖНО: reduce бросает исключение на пустом списке
// List.empty[Int].reduce(_ + _)  // UnsupportedOperationException

// Безопасная альтернатива - reduceOption
List.empty[Int].reduceOption(_ + _)  // None
numbers.reduceOption(_ + _)          // Some(15)

// reduceLeft vs reduceRight
List(1, 2, 3).reduceLeft(_ - _)   // (1 - 2) - 3 = -4
List(1, 2, 3).reduceRight(_ - _)  // 1 - (2 - 3) = 2
```

---

###### 14.5. Другие полезные higher-order functions

```scala
val numbers = List(1, 2, 3, 4, 5)

// filter - отбор элементов
numbers.filter(_ % 2 == 0)  // List(2, 4)

// filterNot - отбор элементов НЕ удовлетворяющих условию
numbers.filterNot(_ % 2 == 0)  // List(1, 3, 5)

// find - поиск первого элемента
numbers.find(_ > 3)  // Some(4)
numbers.find(_ > 10) // None

// exists - проверка существования
numbers.exists(_ > 3)  // true
numbers.exists(_ > 10) // false

// forall - проверка для всех
numbers.forall(_ > 0)  // true
numbers.forall(_ > 3)  // false

// partition - разделение на два списка
numbers.partition(_ % 2 == 0)
// (List(2, 4), List(1, 3, 5))

// groupBy - группировка
numbers.groupBy(_ % 2)
// Map(0 -> List(2, 4), 1 -> List(1, 3, 5))

// collect - комбинация filter + map
numbers.collect {
  case x if x % 2 == 0 => x * 10
}
// List(20, 40)

// takeWhile / dropWhile
numbers.takeWhile(_ < 4)  // List(1, 2, 3)
numbers.dropWhile(_ < 4)  // List(4, 5)

// span - комбинация takeWhile + dropWhile
numbers.span(_ < 4)  // (List(1, 2, 3), List(4, 5))
```

---

##### 15. Function Composition (Композиция функций)

**Определение:**

Композиция функций - это создание новой функции путем последовательного применения других функций.

###### 15.1. andThen - применение слева направо

```scala
val f: Int => Int = _ * 2      // умножить на 2
val g: Int => String = _.toString  // преобразовать в строку

// andThen: сначала f, потом g
val h = f andThen g
h(21)  // f(21) = 42, g(42) = "42"

// Эквивалентно:
val h2 = (x: Int) => g(f(x))
```

###### 15.2. compose - применение справа налево

```scala
val f: Int => Int = _ * 2
val g: Int => String = _.toString

// compose: сначала g, потом f (обратный порядок!)
val parseAndDouble = f compose g.toInt
// parseAndDouble("21") не работает напрямую

// Правильный пример:
val parseInt: String => Int = _.toInt
val double: Int => Int = _ * 2

val parseAndDouble = double compose parseInt
parseAndDouble("21")  // parseInt("21") = 21, double(21) = 42
```

**Разница между andThen и compose:**

```scala
val f: Int => Int = _ + 1      // +1
val g: Int => Int = _ * 2      // *2

(f andThen g)(10)  // f(10) = 11, g(11) = 22
(f compose g)(10)  // g(10) = 20, f(20) = 21

// andThen читается слева направо (интуитивно)
// compose читается справа налево (математически)
```

**Цепочка композиций:**

```scala
val trim: String => String = _.trim
val toLower: String => String = _.toLowerCase
val capitalize: String => String = s => s.head.toUpper + s.tail

val normalize = trim andThen toLower andThen capitalize

normalize("  HELLO world  ")  // "Hello world"

// Или используя множественные andThen
val pipeline = List(trim, toLower, capitalize)
  .reduce(_ andThen _)

pipeline("  HELLO world  ")  // "Hello world"
```

**Композиция с Option:**

```scala
def parseIntOpt(s: String): Option[Int] = 
  try Some(s.toInt) catch { case _: Exception => None }

def isPositive(n: Int): Option[Int] = 
  if (n > 0) Some(n) else None

def sqrt(n: Int): Option[Double] = 
  Some(math.sqrt(n))

// Kleisli composition (композиция монадических функций)
def composeOpt[A, B, C](
  f: A => Option[B],
  g: B => Option[C]
): A => Option[C] = { a =>
  f(a).flatMap(g)
}

val parsePositive = composeOpt(parseIntOpt, isPositive)
parsePositive("42")   // Some(42)
parsePositive("-10")  // None
parsePositive("abc")  // None
```

**Function composition в функциональном стиле:**

```scala
// Вместо императивного:
def processData(data: String): String = {
  val trimmed = data.trim
  val lower = trimmed.toLowerCase
  val capitalized = lower.head.toUpper + lower.tail
  capitalized
}

// Функциональный стиль с композицией:
val processData: String => String = 
  ((_: String).trim) andThen 
  ((_: String).toLowerCase) andThen 
  (s => s.head.toUpper + s.tail)

// Или с явными функциями:
def trim(s: String): String = s.trim
def toLower(s: String): String = s.toLowerCase
def capitalize(s: String): String = s.head.toUpper + s.tail

val processData2 = trim _ andThen toLower andThen capitalize
```

---

##### 16. Currying и Partial Application

###### 16.1. Currying - преобразование функции

**Определение:**

Currying - это преобразование функции с несколькими аргументами в цепочку функций, каждая из которых принимает один аргумент.

```scala
// Обычная функция с двумя параметрами
def add(x: Int, y: Int): Int = x + y
add(2, 3)  // 5

// Curried функция
def addCurried(x: Int)(y: Int): Int = x + y
addCurried(2)(3)  // 5

// Можно применить частично
val add2 = addCurried(2) _  // Int => Int
add2(3)  // 5
add2(10) // 12
```

**Автоматический currying:**

```scala
// Метод curried преобразует обычную функцию в curried
val add: (Int, Int) => Int = _ + _
val addCurried = add.curried  // Int => Int => Int

addCurried(2)(3)  // 5

val increment = addCurried(1)
increment(10)  // 11

// uncurried - обратная операция
val addUncurried = addCurried.curried  // (Int, Int) => Int
addUncurried(2, 3)  // 5
```

**Зачем нужен currying:**

```scala
// 1. Создание специализированных функций
def multiply(x: Int)(y: Int): Int = x * y

val double = multiply(2) _     // Int => Int
val triple = multiply(3) _     // Int => Int
val quadruple = multiply(4) _  // Int => Int

List(1, 2, 3, 4, 5).map(double)  // List(2, 4, 6, 8, 10)

// 2. Конфигурация функций
def log(level: String)(message: String): Unit = 
  println(s"[$level] $message")

val info = log("INFO") _
val error = log("ERROR") _
val debug = log("DEBUG") _

info("Application started")    // [INFO] Application started
error("Connection failed")     // [ERROR] Connection failed

// 3. Создание DSL
def connect(host: String)(port: Int)(timeout: Int): Connection = ???

val localConnection = connect("localhost") _
val devConnection = localConnection(8080) _
val devConn = devConnection(5000)
```

---

###### 16.2. Partial Application - частичное применение

**Определение:**

Partial application - это фиксирование некоторых аргументов функции для создания новой функции с меньшим количеством параметров.

```scala
// Обычная функция
def sum(a: Int, b: Int, c: Int): Int = a + b + c

// Частичное применение с placeholder _
val sumWith5 = sum(5, _: Int, _: Int)
sumWith5(2, 3)  // 10

val sumWith5And2 = sum(5, 2, _: Int)
sumWith5And2(3)  // 10

// С curried функцией проще
def sumCurried(a: Int)(b: Int)(c: Int): Int = a + b + c

val partial1 = sumCurried(5) _           // (Int)(Int) => Int
val partial2 = sumCurried(5)(2) _        // Int => Int
val result = sumCurried(5)(2)(3)         // 10
```

**Практические примеры:**

```scala
// 1. Фильтрация с partial application
def filter[A](list: List[A], predicate: A => Boolean): List[A] = 
  list.filter(predicate)

val numbers = List(1, 2, 3, 4, 5)

val filterNumbers = filter(numbers, _: Int => Boolean)
filterNumbers(_ > 3)      // List(4, 5)
filterNumbers(_ % 2 == 0) // List(2, 4)

// 2. Преобразование с фиксированной конфигурацией
def convert(rate: Double, amount: Double): Double = amount * rate

val usdToEur = convert(0.85, _: Double)
val usdToGbp = convert(0.73, _: Double)

usdToEur(100)  // 85.0
usdToGbp(100)  // 73.0

// 3. HTTP requests с фиксированными заголовками
def makeRequest(
  url: String,
  headers: Map[String, String],
  body: String
): Response = ???

val authenticatedRequest = makeRequest(
  _: String,
  Map("Authorization" -> "Bearer token123"),
  _: String
)

authenticatedRequest("https://api.example.com/users", """{"name": "Alice"}""")
```

**Currying vs Partial Application:**

```scala
// Currying - преобразование структуры функции
def add(x: Int, y: Int): Int = x + y
val addCurried: Int => Int => Int = add.curried

// Partial application - фиксирование аргументов
val add5: Int => Int = add(5, _)
val add5Curried: Int => Int = addCurried(5)

// Оба дают одинаковый результат
add5(3)        // 8
add5Curried(3) // 8

// Но currying позволяет более гибкое использование
val addCurried2 = addCurried(2)      // Int => Int
val addCurried2And3 = addCurried(2)(3)  // Int
```

---

##### 17. Монады (Monad)

**Краткий обзор (детали в разделе 11.4):**

Монада - это паттерн для композиции вычислений в контексте. Любая монада должна иметь:
1. `pure` (или `apply`) - помещает значение в контекст
2. `flatMap` - позволяет последовательно композировать вычисления

###### 17.1. Option - монада для опциональных значений

```scala
// Option представляет значение, которое может отсутствовать
val some: Option[Int] = Some(42)
val none: Option[Int] = None

// flatMap для композиции
def div(x: Int, y: Int): Option[Int] = 
  if (y == 0) None else Some(x / y)

val result = for {
  a <- div(10, 2)   // Some(5)
  b <- div(20, 4)   // Some(5)
  c <- div(a + b, 2) // Some(5)
} yield c
// Some(5)

// Если хотя бы одна операция вернет None, весь результат будет None
val result2 = for {
  a <- div(10, 2)   // Some(5)
  b <- div(20, 0)   // None - деление на ноль!
  c <- div(a + b, 2) // не выполнится
} yield c
// None
```

###### 17.2. Either - монада для обработки ошибок

```scala
// Either[A, B] - либо Left(A) с ошибкой, либо Right(B) с результатом
def divide(x: Int, y: Int): Either[String, Int] = 
  if (y == 0) Left("Division by zero")
  else Right(x / y)

def sqrt(x: Int): Either[String, Double] = 
  if (x < 0) Left("Negative number")
  else Right(math.sqrt(x))

// Композиция с for-comprehension
val computation = for {
  a <- divide(10, 2)      // Right(5)
  b <- divide(20, 4)      // Right(5)
  c <- divide(a + b, 2)   // Right(5)
  d <- sqrt(c)            // Right(2.236...)
} yield d
// Right(2.23606...)

// При ошибке возвращается первый Left
val failed = for {
  a <- divide(10, 2)      // Right(5)
  b <- divide(20, 0)      // Left("Division by zero")
  c <- divide(a + b, 2)   // не выполнится
} yield c
// Left("Division by zero")
```

###### 17.3. Try - монада для обработки исключений

```scala
import scala.util.{Try, Success, Failure}

def parseIntTry(s: String): Try[Int] = Try(s.toInt)

def safeDivide(x: Int, y: Int): Try[Int] = Try(x / y)

// Композиция
val result = for {
  a <- parseIntTry("42")
  b <- parseIntTry("2")
  c <- safeDivide(a, b)
} yield c
// Success(21)

val failed = for {
  a <- parseIntTry("not a number")
  b <- parseIntTry("2")
  c <- safeDivide(a, b)
} yield c
// Failure(NumberFormatException)

// Обработка ошибок
result match {
  case Success(value) => println(s"Result: $value")
  case Failure(exception) => println(s"Error: ${exception.getMessage}")
}
```

###### 17.4. Future - монада для асинхронных вычислений

```scala
import scala.concurrent.{Future, ExecutionContext}
import scala.concurrent.ExecutionContext.Implicits.global

def fetchUser(id: Long): Future[User] = ???
def fetchOrders(userId: Long): Future[List[Order]] = ???
def calculateTotal(orders: List[Order]): Future[Double] = ???

// Композиция асинхронных операций
val totalFuture: Future[Double] = for {
  user <- fetchUser(1)
  orders <- fetchOrders(user.id)
  total <- calculateTotal(orders)
} yield total

// Обработка результата
totalFuture.onComplete {
  case Success(total) => println(s"Total: $total")
  case Failure(error) => println(s"Error: ${error.getMessage}")
}
```

---

##### 18. For-Comprehensions как syntactic sugar

**Правила desugaring (развертывания):**

```scala
// Одна generator линия + yield → map
for (x <- xs) yield f(x)
// становится:
xs.map(x => f(x))

// Несколько generators + yield → flatMap + map
for {
  x <- xs
  y <- ys
} yield f(x, y)
// становится:
xs.flatMap(x => ys.map(y => f(x, y)))

// С guard (if) → withFilter
for {
  x <- xs
  if condition(x)
} yield f(x)
// становится:
xs.withFilter(x => condition(x)).map(x => f(x))

// Без yield → foreach
for (x <- xs) action(x)
// становится:
xs.foreach(x => action(x))

// С присваиванием
for {
  x <- xs
  y = g(x)
} yield f(x, y)
// становится:
xs.map(x => (x, g(x))).map { case (x, y) => f(x, y) }
```

**Практические примеры:**

```scala
// Комплексный for-comprehension
for {
  x <- List(1, 2, 3)
  if x % 2 == 0
  y = x * 10
  z <- List(y, y * 2)
} yield z

// Развернутая версия:
List(1, 2, 3)
  .withFilter(x => x % 2 == 0)
  .map(x => (x, x * 10))
  .flatMap { case (x, y) => 
    List(y, y * 2).map(z => z)
  }
// List(20, 40)
```

---

##### 19. Recursion vs Tail Recursion

###### 19.1. Обычная рекурсия

```scala
// НЕ tail-recursive
def factorial(n: Int): Int = 
  if (n <= 1) 1
  else n * factorial(n - 1)

// Call stack:
// factorial(5)
//   5 * factorial(4)
//     5 * (4 * factorial(3))
//       5 * (4 * (3 * factorial(2)))
//         5 * (4 * (3 * (2 * factorial(1))))
//           5 * (4 * (3 * (2 * 1)))

factorial(5)     // 120
// factorial(10000)  // StackOverflowError!
```

**Проблема:** Каждый рекурсивный вызов добавляет новый stack frame. Для больших n - переполнение стека.

###### 19.2. Tail Recursion (хвостовая рекурсия)

```scala
// Tail-recursive - рекурсивный вызов в последней позиции
@scala.annotation.tailrec
def factorialTail(n: Int, acc: Int = 1): Int = 
  if (n <= 1) acc
  else factorialTail(n - 1, n * acc)

// Компилятор оптимизирует в цикл:
// var n = 5
// var acc = 1
// while (n > 1) {
//   acc = n * acc
//   n = n - 1
// }
// return acc

factorialTail(5)      // 120
factorialTail(10000)  // работает! Нет StackOverflow
```

**@tailrec аннотация:**

```scala
// Компилятор проверит, что функция действительно tail-recursive
@tailrec
def sum(list: List[Int], acc: Int = 0): Int = list match {
  case Nil => acc
  case head :: tail => sum(tail, acc + head)  // ✅ tail call
}

// Не скомпилируется - НЕ tail-recursive
// @tailrec
// def sumBad(list: List[Int]): Int = list match {
//   case Nil => 0
//   case head :: tail => head + sumBad(tail)  // ❌ NOT tail call
// }
// Error: could not optimize @tailrec annotated method
```

**Паттерны tail recursion:**

```scala
// 1. Аккумулятор
@tailrec
def length[A](list: List[A], acc: Int = 0): Int = list match {
  case Nil => acc
  case _ :: tail => length(tail, acc + 1)
}

// 2. Reverse с аккумулятором
@tailrec
def reverse[A](list: List[A], acc: List[A] = Nil): List[A] = list match {
  case Nil => acc
  case head :: tail => reverse(tail, head :: acc)
}

// 3. Fibonacci
@tailrec
def fibonacci(n: Int, prev: BigInt = 0, curr: BigInt = 1): BigInt = 
  if (n == 0) prev
  else fibonacci(n - 1, curr, prev + curr)

// 4. FoldLeft через tail recursion
@tailrec
def foldLeft[A, B](list: List[A], acc: B)(f: (B, A) => B): B = list match {
  case Nil => acc
  case head :: tail => foldLeft(tail, f(acc, head))(f)
}
```

**Mutual tail recursion:**

```scala
@tailrec
def isEven(n: Int): Boolean = 
  if (n == 0) true
  else isOdd(n - 1)

@tailrec
def isOdd(n: Int): Boolean = 
  if (n == 0) false
  else isEven(n - 1)

// ⚠️ Mutual recursion НЕ оптимизируется компилятором Scala
// Будет StackOverflowError на больших n
```

---

##### 20. Lazy Evaluation (Stream/LazyList)

###### 20.1. Lazy evaluation - что это?

```scala
// Eager evaluation - вычисляется сразу
val eagerList = List(1, 2, 3).map { x =>
  println(s"Computing $x")
  x * 2
}
// Напечатает:
// Computing 1
// Computing 2
// Computing 3

// Lazy evaluation - вычисляется по требованию
val lazyList = LazyList(1, 2, 3).map { x =>
  println(s"Computing $x")
  x * 2
}
// Ничего не печатает!

lazyList.take(2).toList
// Напечатает только:
// Computing 1
// Computing 2
```

###### 20.2. LazyList (ранее Stream в Scala 2.12)

```scala
// Создание LazyList
val lazy1 = LazyList(1, 2, 3)
val lazy2 = LazyList.from(1)  // бесконечная последовательность 1, 2, 3, ...

// #:: - ленивый cons operator
val lazy3 = 1 #:: 2 #:: 3 #:: LazyList.empty

// Бесконечные структуры данных
lazy val fibonacci: LazyList[BigInt] = 
  BigInt(0) #:: BigInt(1) #:: fibonacci.zip(fibonacci.tail).map {
    case (a, b) => a + b
  }

fibonacci.take(10).toList
// List(0, 1, 1, 2, 3, 5, 8, 13, 21, 34)

// Бесконечная последовательность простых чисел
def sieve(nums: LazyList[Int]): LazyList[Int] = 
  nums.head #:: sieve(nums.tail.filter(_ % nums.head != 0))

val primes = sieve(LazyList.from(2))
primes.take(10).toList
// List(2, 3, 5, 7, 11, 13, 17, 19, 23, 29)
```

###### 20.3. Преимущества lazy evaluation

```scala
// 1. Работа с бесконечными структурами
val naturals = LazyList.from(1)
naturals.take(5).toList  // List(1, 2, 3, 4, 5)

// 2. Избежание ненужных вычислений
val expensive = LazyList.from(1).map { x =>
  Thread.sleep(100)  // дорогая операция
  x * 2
}

expensive.take(3).toList  // вычисляются только 3 элемента
// Занимает ~300ms вместо бесконечности

// 3. Композиция трансформаций без промежуточных коллекций
val result = LazyList.from(1)
  .map(_ * 2)
  .filter(_ % 3 == 0)
  .map(_.toString)
  .take(5)
  .toList
// Вычисляется за один проход!

// 4. Short-circuit evaluation
val found = LazyList.from(1).find(_ > 1000000)
// Останавливается как только нашли первый элемент
```

###### 20.4. View - lazy обертка над коллекциями

```scala
val numbers = (1 to 1000000).toList

// Eager - создает промежуточные коллекции
val result1 = numbers
  .map(_ * 2)      // создается новый List
  .filter(_ % 3 == 0)  // создается еще один List
  .map(_.toString) // и еще один List
  .take(10)

// Lazy - используем view
val result2 = numbers.view
  .map(_ * 2)
  .filter(_ % 3 == 0)
  .map(_.toString)
  .take(10)
  .toList  // материализация только в конце

// view избегает создания промежуточных коллекций
// и может остановиться раньше (take(10))
```

###### 20.5. Memoization в LazyList

```scala
// LazyList кеширует вычисленные элементы
val expensive = LazyList.from(1).map { x =>
  println(s"Computing $x")
  x * 2
}

val first5 = expensive.take(5).toList
// Печатает: Computing 1, 2, 3, 4, 5

val first10 = expensive.take(10).toList
// Печатает только: Computing 6, 7, 8, 9, 10
// Элементы 1-5 уже закешированы!

// ⚠️ Осторожно: кеширование использует память
val hugeLazy = LazyList.from(1).take(1000000)
hugeLazy.last  // вычислит и закеширует все 1000000 элементов!
```

---

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

- [Что такое монада? Законы монад?](#134-монада-monad) (см. также раздел [17. Монады](#17-монады-monad))
- [Разница между map и flatMap?](#142-map---преобразование-элементов) и [14.3. flatMap](#143-flatmap---преобразование-с-распаковкой)
- [Что такое Applicative? Разница с Monad?](#133-аппликативный-функтор-applicative)
- [Как работает @tailrec?](#192-tail-recursion-хвостовая-рекурсия)

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

---

#### 📖 Теоретические материалы

---

##### 21. Higher-Kinded Types (HKT) - Типы высшего порядка

**Определение:**

Higher-Kinded Type (тип высшего порядка) - это тип, который абстрагируется над type constructor'ом, а не над конкретным типом.

**Виды (Kinds) типов:**

```scala
// Kind * - обычные типы (proper types)
// Примеры: Int, String, Boolean, User
// Можно создать значение этого типа

val x: Int = 42
val s: String = "hello"

// Kind * -> * - type constructor с одним параметром
// Примеры: List, Option, Future
// НЕЛЬЗЯ создать значение типа List (без параметра)
// Можно создать: List[Int], Option[String]

// val list: List = ???  // ❌ Ошибка компиляции
val list: List[Int] = List(1, 2, 3)  // ✅ OK

// Kind * -> * -> * - type constructor с двумя параметрами
// Примеры: Map, Either, Function1
type MyMap = Map[String, Int]
type MyEither = Either[String, Int]
type MyFunc = String => Int  // Function1[String, Int]

// Kind (* -> *) -> * - Higher-Kinded Type
// Абстракция над type constructor'ом
trait Functor[F[_]] {  // F[_] - это type constructor
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

**F[_] vs F[A]:**

```scala
// F[A] - конкретный тип (proper type), где F и A известны
def processOption(opt: Option[Int]): Int = opt.getOrElse(0)

// F[_] - type constructor, абстракция над структурой
trait Container[F[_]] {
  def wrap[A](value: A): F[A]
  def unwrap[A](fa: F[A]): Option[A]
}

// Можем реализовать для разных F
object OptionContainer extends Container[Option] {
  def wrap[A](value: A): Option[A] = Some(value)
  def unwrap[A](fa: Option[A]): Option[A] = fa
}

object ListContainer extends Container[List] {
  def wrap[A](value: A): List[A] = List(value)
  def unwrap[A](fa: List[A]): Option[A] = fa.headOption
}
```

**Практическое применение - Абстракция над контейнерами:**

```scala
// Пример 1: Функтор - работает с любым F[_]
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

implicit val listFunctor: Functor[List] = new Functor[List] {
  def map[A, B](fa: List[A])(f: A => B): List[B] = fa.map(f)
}

implicit val optionFunctor: Functor[Option] = new Functor[Option] {
  def map[A, B](fa: Option[A])(f: A => B): Option[B] = fa.map(f)
}

// Универсальная функция работающая с любым функтором
def increment[F[_]: Functor](fa: F[Int]): F[Int] = {
  val functor = implicitly[Functor[F]]
  functor.map(fa)(_ + 1)
}

increment(List(1, 2, 3))    // List(2, 3, 4)
increment(Some(5))          // Some(6)
```

**Пример 2: Монада**

```scala
trait Monad[F[_]] extends Functor[F] {
  def pure[A](a: A): F[A]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  
  // map можно выразить через flatMap и pure
  def map[A, B](fa: F[A])(f: A => B): F[B] = 
    flatMap(fa)(a => pure(f(a)))
}

implicit val listMonad: Monad[List] = new Monad[List] {
  def pure[A](a: A): List[A] = List(a)
  def flatMap[A, B](fa: List[A])(f: A => List[B]): List[B] = fa.flatMap(f)
}

// Универсальные функции для любой монады
def sequence[F[_]: Monad, A](list: List[F[A]]): F[List[A]] = {
  val m = implicitly[Monad[F]]
  list.foldRight(m.pure(List.empty[A])) { (fa, acc) =>
    m.flatMap(fa) { a =>
      m.map(acc)(list => a :: list)
    }
  }
}

sequence(List(Some(1), Some(2), Some(3)))  // Some(List(1, 2, 3))
sequence(List(Some(1), None, Some(3)))     // None
```

**Ограничения HKT в Scala:**

```scala
// ✅ Работает - F[_] с одним параметром
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

// ❌ Не работает напрямую - Either имеет два параметра
// trait EitherFunctor[Either[_, _]] - так нельзя

// ✅ Решение - type lambda (частичное применение типов)
type EitherString[A] = Either[String, A]

implicit val eitherFunctor: Functor[EitherString] = new Functor[EitherString] {
  def map[A, B](fa: Either[String, A])(f: A => B): Either[String, B] = 
    fa.map(f)
}

// Или с помощью kind-projector plugin:
// implicit def eitherFunctor[E]: Functor[Either[E, *]]
```

**Kind-projector plugin:**

```scala
// В build.sbt:
// addCompilerPlugin("org.typelevel" % "kind-projector" % "0.13.2" cross CrossVersion.full)

// Синтаксис с kind-projector
trait Bifunctor[F[_, _]] {
  def bimap[A, B, C, D](fab: F[A, B])(f: A => C, g: B => D): F[C, D]
}

implicit val eitherBifunctor: Bifunctor[Either] = new Bifunctor[Either] {
  def bimap[A, B, C, D](fab: Either[A, B])(f: A => C, g: B => D): Either[C, D] = 
    fab match {
      case Left(a) => Left(f(a))
      case Right(b) => Right(g(b))
    }
}

// Type lambda с kind-projector
type StringOr[A] = Either[String, A]  // старый способ

// С kind-projector можно писать:
// Either[String, *] или Either[String, ?]
```

---

##### 22. Type Bounds (Границы типов)

###### 22.1. Upper Type Bound (верхняя граница) - <:

Означает: "тип должен быть подтипом указанного типа"

```scala
// Базовая иерархия
trait Animal {
  def name: String
}
class Dog extends Animal {
  def name = "Dog"
  def bark(): Unit = println("Woof!")
}
class Cat extends Animal {
  def name = "Cat"
  def meow(): Unit = println("Meow!")
}

// Upper bound - принимаем только Animal и его подтипы
class Shelter[A <: Animal](animals: List[A]) {
  def names: List[String] = animals.map(_.name)
  
  // Можем вызывать методы Animal, так как A <: Animal
  def printNames(): Unit = animals.foreach(a => println(a.name))
}

// ✅ Работает
val dogShelter = new Shelter[Dog](List(new Dog, new Dog))
val catShelter = new Shelter[Cat](List(new Cat))
val animalShelter = new Shelter[Animal](List(new Dog, new Cat))

// ❌ Не компилируется
// val shelter = new Shelter[String](List("hello"))
// Error: String не является подтипом Animal
```

**Практическое применение upper bounds:**

```scala
// Пример 1: Сортировка
def sort[A <: Comparable[A]](list: List[A]): List[A] = 
  list.sorted

// Пример 2: Ограничение на числовые типы
def sum[A <: AnyVal](numbers: List[A])(implicit num: Numeric[A]): A = 
  numbers.sum

// Пример 3: С multiple bounds
trait Serializable
trait Printable {
  def print(): String
}

// A должен быть подтипом И Animal, И Serializable
class Zoo[A <: Animal with Serializable](animals: List[A])

// Или с контекстными границами:
class AdvancedZoo[A <: Animal : Ordering](animals: List[A]) {
  def sorted: List[A] = animals.sorted
}
```

###### 22.2. Lower Type Bound (нижняя граница) - >:

Означает: "тип должен быть супертипом указанного типа"

```scala
// Lower bound используется чаще всего с variance
class Box[+A] {
  // Без lower bound это не скомпилируется (variance problem)
  // def add(item: A): Box[A] = ???  // ❌ Error
  
  // С lower bound - ✅ OK
  def add[B >: A](item: B): Box[B] = new Box[B]
}

val animalBox: Box[Animal] = new Box[Dog]
// Можем добавить Cat (супертип Dog - это Animal)
val mixedBox: Box[Animal] = animalBox.add(new Cat)
```

**Практические примеры lower bounds:**

```scala
// Пример 1: Добавление элементов в коварiantную структуру
sealed trait MyList[+A] {
  def prepend[B >: A](elem: B): MyList[B]
}

case object Empty extends MyList[Nothing] {
  def prepend[B >: Nothing](elem: B): MyList[B] = Cons(elem, Empty)
}

case class Cons[A](head: A, tail: MyList[A]) extends MyList[A] {
  def prepend[B >: A](elem: B): MyList[B] = Cons(elem, this)
}

val dogList: MyList[Dog] = Cons(new Dog, Empty)
val animalList: MyList[Animal] = dogList.prepend(new Cat)  // ✅ OK

// Пример 2: Стандартная библиотека Scala
// def ::[B >: A](elem: B): List[B]

val dogs: List[Dog] = List(new Dog)
val animals: List[Animal] = new Cat :: dogs  // ✅ OK
```

###### 22.3. Сочетание Upper и Lower bounds:

```scala
class Container[A] {
  // Метод принимает супертип A, возвращает подтип A
  def transform[B >: A, C <: A](f: B => C): C = ???
}

// Пример с Option
sealed trait MyOption[+A] {
  // getOrElse принимает супертип A (для default value)
  def getOrElse[B >: A](default: => B): B
  
  // map сохраняет ковариантность
  def map[B](f: A => B): MyOption[B]
  
  // filter требует суперкласс A для предиката
  def filter(p: A => Boolean): MyOption[A]
}
```

###### 22.4. View Bounds (устаревшие в Scala 2.13+):

```scala
// Старый синтаксис (deprecated):
def printSorted[A <% Ordered[A]](list: List[A]): Unit = 
  list.sorted.foreach(println)

// Современный эквивалент - используйте Ordering:
def printSorted[A: Ordering](list: List[A]): Unit = 
  list.sorted.foreach(println)

// Или явно:
def printSorted[A](list: List[A])(implicit ord: Ordering[A]): Unit = 
  list.sorted.foreach(println)
```

---

##### 23. Type Classes (Классы типов)

**Определение:**

Type class - это паттерн, позволяющий добавлять новую функциональность к существующим типам без изменения их исходного кода. Это форма ad-hoc полиморфизма.

###### 23.1. Проблема, которую решают type classes:

```scala
// Представим, у нас есть разные типы:
case class User(name: String, age: Int)
case class Product(id: Long, name: String, price: Double)

// Мы хотим сериализовать их в JSON
// Вариант 1 (ООП) - добавить метод toJson:
trait JsonSerializable {
  def toJson: String
}

case class User(name: String, age: Int) extends JsonSerializable {
  def toJson: String = s"""{"name": "$name", "age": $age}"""
}

// ❌ Проблемы:
// 1. Нужно изменять исходный код классов
// 2. Не работает с типами из библиотек (Int, String, List)
// 3. Один класс = один способ сериализации
```

###### 23.2. Решение с Type Classes:

```scala
// Шаг 1: Определяем type class
trait JsonSerializer[A] {
  def toJson(value: A): String
}

// Шаг 2: Создаем instances для конкретных типов
object JsonSerializer {
  // Instance для Int
  implicit val intSerializer: JsonSerializer[Int] = 
    (value: Int) => value.toString
  
  // Instance для String
  implicit val stringSerializer: JsonSerializer[String] = 
    (value: String) => s""""$value""""
  
  // Instance для User
  implicit val userSerializer: JsonSerializer[User] = 
    (user: User) => s"""{"name": "${user.name}", "age": ${user.age}}"""
  
  // Generic instance для List
  implicit def listSerializer[A](implicit ser: JsonSerializer[A]): JsonSerializer[List[A]] = 
    (list: List[A]) => list.map(ser.toJson).mkString("[", ", ", "]")
}

// Шаг 3: Используем type class
def toJson[A](value: A)(implicit serializer: JsonSerializer[A]): String = 
  serializer.toJson(value)

// Или с context bound:
def toJson[A: JsonSerializer](value: A): String = 
  implicitly[JsonSerializer[A]].toJson(value)

// Использование:
import JsonSerializer._

toJson(42)                        // "42"
toJson("hello")                   // "\"hello\""
toJson(User("Alice", 30))         // {"name": "Alice", "age": 30}
toJson(List(1, 2, 3))             // [1, 2, 3]
```

###### 23.3. Улучшенный синтаксис (Interface Syntax):

```scala
// Добавим вспомогательные методы
object JsonSerializer {
  // Summoner method
  def apply[A](implicit instance: JsonSerializer[A]): JsonSerializer[A] = instance
  
  // Constructor method
  def instance[A](f: A => String): JsonSerializer[A] = 
    (value: A) => f(value)
}

// Syntax extension (неявное расширение)
implicit class JsonSyntax[A](value: A) {
  def toJson(implicit serializer: JsonSerializer[A]): String = 
    serializer.toJson(value)
}

// Теперь можно писать:
import JsonSerializer._

42.toJson                         // "42"
"hello".toJson                    // "\"hello\""
User("Bob", 25).toJson            // {"name": "Bob", "age": 25}
List(1, 2, 3).toJson              // [1, 2, 3]
```

###### 23.4. Type Class Laws (законы):

Многие type classes имеют законы, которым должны следовать instances:

```scala
// Пример: Monoid type class
trait Monoid[A] {
  def empty: A
  def combine(x: A, y: A): A
}

// Законы Monoid:
// 1. Ассоциативность: combine(x, combine(y, z)) == combine(combine(x, y), z)
// 2. Левая идентичность: combine(empty, x) == x
// 3. Правая идентичность: combine(x, empty) == x

implicit val intAdditionMonoid: Monoid[Int] = new Monoid[Int] {
  def empty: Int = 0
  def combine(x: Int, y: Int): Int = x + y
}

// Проверка законов:
// combine(1, combine(2, 3)) == combine(combine(1, 2), 3)  // 6 == 6 ✅
// combine(0, 5) == 5  // ✅
// combine(5, 0) == 5  // ✅

implicit val stringMonoid: Monoid[String] = new Monoid[String] {
  def empty: String = ""
  def combine(x: String, y: String): String = x + y
}
```

###### 23.5. Стандартные Type Classes:

```scala
// Ordering - упорядочивание
def sort[A: Ordering](list: List[A]): List[A] = list.sorted

// Numeric - числовые операции
def sum[A: Numeric](list: List[A]): A = list.sum

// Show (из Cats) - преобразование в строку
import cats.Show
import cats.implicits._

case class Person(name: String, age: Int)

implicit val personShow: Show[Person] = Show.show { person =>
  s"${person.name} is ${person.age} years old"
}

Person("Alice", 30).show  // "Alice is 30 years old"

// Eq (из Cats) - type-safe сравнение
import cats.Eq
import cats.syntax.eq._

implicit val personEq: Eq[Person] = Eq.fromUniversalEquals

Person("Alice", 30) === Person("Alice", 30)  // true
Person("Alice", 30) =!= Person("Bob", 25)    // true
```

###### 23.6. Type Classes vs Inheritance:

```scala
// ООП подход (наследование):
trait Printable {
  def print: String
}

case class User(name: String) extends Printable {
  def print: String = s"User: $name"
}

// ❌ Проблемы:
// - Нельзя добавить к существующим типам (Int, String)
// - Жесткая связь (coupling)
// - Только один способ реализации

// Type Class подход:
trait Printer[A] {
  def print(value: A): String
}

implicit val userPrinter: Printer[User] = 
  (user: User) => s"User: ${user.name}"

implicit val verboseUserPrinter: Printer[User] = 
  (user: User) => s"User Details: Name=${user.name}"

// ✅ Преимущества:
// - Работает с любыми типами
// - Слабая связь (loose coupling)
// - Множественные реализации (в разных scope)
// - Retroactive extension (добавление функциональности постфактум)
```

---

##### 24. Context Bounds (Контекстные границы)

**Определение:**

Context bound - это синтаксический сахар для implicit параметров с type classes.

###### 24.1. Базовый синтаксис:

```scala
// Полная форма с implicit параметром:
def show[A](value: A)(implicit shower: Show[A]): String = 
  shower.show(value)

// Context bound (сокращенная форма):
def show[A: Show](value: A): String = 
  implicitly[Show[A]].show(value)

// Или еще короче с summoner:
def show[A: Show](value: A): String = 
  Show[A].show(value)  // если есть def apply[A](implicit ev: Show[A])
```

###### 24.2. Множественные context bounds:

```scala
// Несколько type classes
def processData[A: Ordering : Numeric : Show](list: List[A]): String = {
  val sorted = list.sorted                    // использует Ordering
  val sum = sorted.sum                        // использует Numeric
  Show[A].show(sum)                          // использует Show
}

// Эквивалентно:
def processData[A](list: List[A])(
  implicit ord: Ordering[A], 
  num: Numeric[A], 
  show: Show[A]
): String = {
  val sorted = list.sorted(ord)
  val sum = num.plus(sorted.reduce(num.plus), num.zero)
  show.show(sum)
}
```

###### 24.3. Context bounds с Higher-Kinded Types:

```scala
// F[_] с context bound
def sequence[F[_]: Monad, A](list: List[F[A]]): F[List[A]] = {
  val m = implicitly[Monad[F]]
  list.foldRight(m.pure(List.empty[A])) { (fa, acc) =>
    m.flatMap(fa)(a => m.map(acc)(a :: _))
  }
}

// Использование:
sequence(List(Some(1), Some(2), Some(3)))  // Some(List(1, 2, 3))
sequence(List(Right(1), Right(2)))         // Right(List(1, 2))
```

###### 24.4. Доступ к implicit instance:

```scala
// Способ 1: implicitly
def method1[A: Ordering](list: List[A]): List[A] = {
  val ord = implicitly[Ordering[A]]
  list.sorted(ord)
}

// Способ 2: summoner (если определен apply)
trait Show[A] {
  def show(value: A): String
}

object Show {
  def apply[A](implicit instance: Show[A]): Show[A] = instance
}

def method2[A: Show](value: A): String = {
  Show[A].show(value)  // Вызов summoner
}

// Способ 3: явный implicit параметр (если нужно имя)
def method3[A: Ordering](list: List[A]): List[A] = {
  implicitly[Ordering[A]] match {
    case ord => list.sorted(ord)
  }
}
```

###### 24.5. Context bounds в классах:

```scala
// В классах context bounds работают так же
class Container[A: Ordering](elements: List[A]) {
  def sorted: List[A] = elements.sorted
  
  def max: A = elements.max
  
  def isSorted: Boolean = {
    val ord = implicitly[Ordering[A]]
    elements.sliding(2).forall { 
      case List(a, b) => ord.lteq(a, b)
      case _ => true
    }
  }
}

// Использование:
val intContainer = new Container(List(3, 1, 4, 1, 5))
intContainer.sorted   // List(1, 1, 3, 4, 5)
intContainer.max      // 5
```

###### 24.6. Практический пример - Generic сортировка:

```scala
case class Person(name: String, age: Int)

// Определяем разные Ordering instances
object Person {
  implicit val orderByName: Ordering[Person] = 
    Ordering.by(_.name)
  
  val orderByAge: Ordering[Person] = 
    Ordering.by(_.age)
}

// Generic функция с context bound
def sortAndPrint[A: Ordering : Show](items: List[A]): Unit = {
  val sorted = items.sorted
  sorted.foreach(item => println(Show[A].show(item)))
}

// Можем менять ordering через implicit scope
{
  import Person.orderByName
  sortAndPrint(persons)  // сортирует по имени
}

{
  implicit val ord = Person.orderByAge
  sortAndPrint(persons)  // сортирует по возрасту
}
```

---

##### 25. Path-Dependent Types (Путе-зависимые типы)

**Определение:**

Path-dependent type - это тип, который зависит от конкретного экземпляра (пути к значению), а не только от класса.

###### 25.1. Базовый пример:

```scala
class Outer {
  class Inner {
    def sayHi(): Unit = println("Hi from Inner")
  }
  
  def createInner(): Inner = new Inner
}

val outer1 = new Outer
val outer2 = new Outer

val inner1: outer1.Inner = outer1.createInner()  // path-dependent type
val inner2: outer2.Inner = outer2.createInner()

// inner1 и inner2 имеют РАЗНЫЕ типы!
// inner1: outer1.Inner
// inner2: outer2.Inner

// ❌ Не компилируется:
// val wrongInner: outer1.Inner = outer2.createInner()
// Error: type mismatch
```

###### 25.2. Практический пример - Graph:

```scala
class Graph {
  // Вложенные классы зависят от экземпляра Graph
  class Node(val value: Int) {
    def connectTo(other: Node): Edge = new Edge(this, other)
  }
  
  class Edge(val from: Node, val to: Node)
  
  def createNode(value: Int): Node = new Node(value)
}

val graph1 = new Graph
val graph2 = new Graph

val node1a: graph1.Node = graph1.createNode(1)
val node1b: graph1.Node = graph1.createNode(2)
val node2a: graph2.Node = graph2.createNode(3)

// ✅ OK - оба узла из graph1
val edge1: graph1.Edge = node1a.connectTo(node1b)

// ❌ Не компилируется - узлы из разных графов
// val invalidEdge: graph1.Edge = node1a.connectTo(node2a)
// Error: type mismatch - узлы должны быть из одного графа!

// Это type safety на уровне компилятора!
```

###### 25.3. Type Projection - # (hash):

Иногда нужно абстрагироваться от конкретного пути:

```scala
class Database {
  class Table(val name: String) {
    def query(sql: String): List[String] = List(s"Result from $name")
  }
}

// Path-dependent type:
def processTable(table: database.Table): Unit = ???  // привязан к database

// Type projection - любая Table из любой Database:
def processAnyTable(table: Database#Table): Unit = {
  println(table.query("SELECT *"))
}

val db1 = new Database
val db2 = new Database

val table1 = new db1.Table("users")
val table2 = new db2.Table("orders")

processAnyTable(table1)  // ✅ OK
processAnyTable(table2)  // ✅ OK
```

###### 25.4. Abstract Type Members:

```scala
trait Container {
  type Element  // абстрактный type member
  
  def add(elem: Element): Unit
  def get(): Element
}

class IntContainer extends Container {
  type Element = Int
  
  private var value: Int = 0
  
  def add(elem: Int): Unit = value = elem
  def get(): Int = value
}

class StringContainer extends Container {
  type Element = String
  
  private var value: String = ""
  
  def add(elem: String): Unit = value = elem
  def get(): String = value
}

// Path-dependent type с abstract type members:
def useContainer(container: Container)(elem: container.Element): Unit = {
  container.add(elem)
  println(container.get())
}

val intC = new IntContainer
useContainer(intC)(42)  // container.Element = Int

val strC = new StringContainer
useContainer(strC)("hello")  // container.Element = String

// ❌ Не компилируется:
// useContainer(intC)("hello")  // Error: type mismatch
```

###### 25.5. Cake Pattern (Dependency Injection):

```scala
// Компоненты системы
trait UserRepositoryComponent {
  val userRepository: UserRepository
  
  trait UserRepository {
    def findById(id: Long): Option[User]
  }
}

trait UserServiceComponent {
  this: UserRepositoryComponent =>  // self-type annotation
  
  val userService: UserService
  
  trait UserService {
    def getUser(id: Long): Option[User] = userRepository.findById(id)
  }
}

// Конкретные реализации
trait UserRepositoryComponentImpl extends UserRepositoryComponent {
  val userRepository = new UserRepositoryImpl
  
  class UserRepositoryImpl extends UserRepository {
    def findById(id: Long): Option[User] = Some(User(id, "Alice"))
  }
}

trait UserServiceComponentImpl extends UserServiceComponent {
  this: UserRepositoryComponent =>
  
  val userService = new UserServiceImpl
  
  class UserServiceImpl extends UserService
}

// Сборка приложения
object Application extends UserServiceComponentImpl 
                     with UserRepositoryComponentImpl

// Path-dependent types гарантируют корректность связей!
```

###### 25.6. Type Refinement:

```scala
trait Animal {
  type Food
  def eat(food: Food): Unit
}

class Dog extends Animal {
  type Food = Bone
  def eat(food: Bone): Unit = println(s"Eating bone")
}

class Cat extends Animal {
  type Food = Fish
  def eat(food: Fish): Unit = println(s"Eating fish")
}

case class Bone(size: Int)
case class Fish(species: String)

// Функция с type refinement
def feedAnimal(animal: Animal { type Food = Bone })(food: Bone): Unit = {
  animal.eat(food)
}

val dog = new Dog
feedAnimal(dog)(Bone(10))  // ✅ OK

val cat = new Cat
// feedAnimal(cat)(Fish("salmon"))  // ❌ Error: type mismatch
```

---

##### 26. Phantom Types (Фантомные типы)

**Определение:**

Phantom type - это type parameter, который не используется в runtime, но помогает обеспечить type safety на этапе компиляции.

###### 26.1. Базовый пример - Type-safe API:

```scala
// Состояния соединения
sealed trait ConnectionState
sealed trait Closed extends ConnectionState
sealed trait Open extends ConnectionState

// Connection с phantom type
class Connection[State <: ConnectionState] private (handle: String) {
  def getHandle: String = handle
}

object Connection {
  // Можем создать только закрытое соединение
  def create(handle: String): Connection[Closed] = 
    new Connection[Closed](handle)
  
  // Открыть можно только закрытое соединение
  def open(conn: Connection[Closed]): Connection[Open] = 
    new Connection[Open](conn.getHandle)
  
  // Закрыть можно только открытое соединение
  def close(conn: Connection[Open]): Connection[Closed] = 
    new Connection[Closed](conn.getHandle)
  
  // Отправить данные можно только через открытое соединение
  def send(conn: Connection[Open], data: String): Unit = {
    println(s"Sending: $data via ${conn.getHandle}")
  }
}

// Использование:
val closed = Connection.create("conn-123")
val open = Connection.open(closed)
Connection.send(open, "Hello!")
val closedAgain = Connection.close(open)

// ❌ Ошибки компиляции:
// Connection.send(closed, "data")     // Error: требуется Open, а у нас Closed
// Connection.open(open)                // Error: требуется Closed, а у нас Open
// Connection.close(closed)             // Error: требуется Open, а у нас Closed

// Type safety без runtime проверок!
```

###### 26.2. Пример - Validated Data:

```scala
// Состояния валидации
sealed trait ValidationState
sealed trait Unvalidated extends ValidationState
sealed trait Validated extends ValidationState

case class Email[State <: ValidationState](value: String)

object Email {
  // Создание невалидированного email
  def apply(value: String): Email[Unvalidated] = 
    new Email[Unvalidated](value)
  
  // Валидация
  def validate(email: Email[Unvalidated]): Option[Email[Validated]] = {
    if (email.value.contains("@")) 
      Some(new Email[Validated](email.value))
    else 
      None
  }
  
  // Отправка возможна только для валидированных email
  def send(email: Email[Validated], message: String): Unit = {
    println(s"Sending to ${email.value}: $message")
  }
}

// Использование:
val rawEmail = Email("user@example.com")
Email.validate(rawEmail) match {
  case Some(validEmail) => Email.send(validEmail, "Hello!")
  case None => println("Invalid email")
}

// ❌ Не компилируется:
// Email.send(rawEmail, "Hello!")  // Error: требуется Validated
```

###### 26.3. Пример - Builder Pattern:

```scala
// Этапы построения
sealed trait BuilderState
sealed trait WithName extends BuilderState
sealed trait WithAge extends BuilderState
sealed trait Complete extends BuilderState

class PersonBuilder[State <: BuilderState] private (
  name: Option[String] = None,
  age: Option[Int] = None
)

object PersonBuilder {
  def apply(): PersonBuilder[BuilderState] = 
    new PersonBuilder[BuilderState]()
  
  implicit class NameOps(builder: PersonBuilder[BuilderState]) {
    def withName(n: String): PersonBuilder[WithName] = 
      new PersonBuilder[WithName](Some(n), None)
  }
  
  implicit class AgeOps(builder: PersonBuilder[WithName]) {
    def withAge(a: Int): PersonBuilder[Complete] = 
      new PersonBuilder[Complete](builder.name, Some(a))
  }
  
  implicit class BuildOps(builder: PersonBuilder[Complete]) {
    def build(): Person = Person(builder.name.get, builder.age.get)
  }
}

case class Person(name: String, age: Int)

// Использование:
import PersonBuilder._

val person = PersonBuilder()
  .withName("Alice")
  .withAge(30)
  .build()

// ❌ Не компилируется - нарушен порядок:
// PersonBuilder().withAge(30)  // Error: нет метода withAge
// PersonBuilder().withName("Bob").build()  // Error: нет метода build
```

###### 26.4. Пример - Units of Measure:

```scala
// Единицы измерения
sealed trait Unit
sealed trait Meter extends Unit
sealed trait Kilometer extends Unit
sealed trait Second extends Unit

case class Quantity[U <: Unit](value: Double) {
  def +[U2 <: Unit](other: Quantity[U2])(implicit ev: U =:= U2): Quantity[U] = 
    Quantity[U](value + other.value)
  
  def *[U2 <: Unit](other: Quantity[U2]): Quantity[Unit] = 
    Quantity[Unit](value * other.value)
}

object Quantity {
  type Meters = Quantity[Meter]
  type Kilometers = Quantity[Kilometer]
  type Seconds = Quantity[Second]
  
  implicit class MeterOps(q: Meters) {
    def toKilometers: Kilometers = Quantity[Kilometer](q.value / 1000)
  }
  
  implicit class KilometerOps(q: Kilometers) {
    def toMeters: Meters = Quantity[Meter](q.value * 1000)
  }
}

import Quantity._

val distance1: Meters = Quantity[Meter](1000)
val distance2: Meters = Quantity[Meter](500)
val time: Seconds = Quantity[Second](10)

val totalDistance = distance1 + distance2  // ✅ OK - одинаковые единицы
val distanceKm = distance1.toKilometers     // ✅ OK - конвертация

// ❌ Не компилируется:
// val wrong = distance1 + time  // Error: разные единицы измерения
```

###### 26.5. Преимущества Phantom Types:

1. **Compile-time safety** - ошибки обнаруживаются при компиляции
2. **Zero runtime cost** - phantom types стираются после компиляции
3. **Self-documenting** - типы описывают состояние и правила
4. **Refactoring safety** - изменения автоматически проверяются компилятором

---

##### 27. Existential Types (Экзистенциальные типы)

**Определение:**

Existential type - это тип, который говорит "существует некоторый тип, но мы не знаем какой именно". В Scala записывается как `T forSome { type T }` или с использованием wildcards `_`.

###### 27.1. Базовый синтаксис:

```scala
// Полная форма:
val list1: List[T] forSome { type T } = List(1, 2, 3)

// Сокращенная форма с wildcard:
val list2: List[_] = List(1, 2, 3)

// Эквивалентно: "список чего-то, но мы не знаем чего"
```

###### 27.2. Практический пример - Heterogeneous Collections:

```scala
// Без existential types:
trait Animal {
  def makeSound(): Unit
}

class Dog extends Animal {
  def makeSound(): Unit = println("Woof")
  def wagTail(): Unit = println("Wagging tail")
}

class Cat extends Animal {
  def makeSound(): Unit = println("Meow")
  def purr(): Unit = println("Purr")
}

// Гомогенная коллекция (все Animal):
val animals: List[Animal] = List(new Dog, new Cat)
animals.foreach(_.makeSound())  // ✅ OK

// Но потеряли специфичные методы:
// animals.head.wagTail()  // ❌ Error

// С existential types - гетерогенная коллекция:
trait Box[A] {
  def get: A
}

class IntBox(value: Int) extends Box[Int] {
  def get: Int = value
}

class StringBox(value: String) extends Box[String] {
  def get: String = value
}

// Список Box с неизвестными типами содержимого
val boxes: List[Box[_]] = List(
  new IntBox(42),
  new StringBox("hello")
)

// Можем работать с Box, но не знаем конкретный тип T
boxes.foreach { box =>
  val value: Any = box.get  // Тип потерян, получаем Any
  println(value)
}
```

###### 27.3. Existential types с Type Members:

```scala
trait Processor {
  type Input
  type Output
  
  def process(input: Input): Output
}

class IntProcessor extends Processor {
  type Input = Int
  type Output = String
  
  def process(input: Int): String = input.toString
}

class StringProcessor extends Processor {
  type Input = String
  type Output = Int
  
  def process(input: String): Int = input.length
}

// Список процессоров с неизвестными Input/Output типами
val processors: List[Processor] = List(
  new IntProcessor,
  new StringProcessor
)

// Эквивалентно:
val processorsExistential: List[Processor { type Input; type Output }] = processors

// Или:
type AnyProcessor = Processor forSome { 
  type Input
  type Output  
}
val processors2: List[AnyProcessor] = processors
```

###### 27.4. Bounded Existentials:

```scala
// Existential type с ограничениями
trait Container[A] {
  def get: A
}

// "Контейнер чего-то, что является подтипом Number"
val numericContainers: List[Container[_ <: Number]] = List(
  new Container[Int] { def get = 42 },
  new Container[Double] { def get = 3.14 },
  new Container[java.lang.Long] { def get = 100L }
)

numericContainers.foreach { container =>
  val num: Number = container.get  // Мы знаем, что это Number
  println(num.doubleValue())
}

// Lower bound:
trait Producer[A] {
  def produce(): A
}

// "Производитель чего-то, что является супертипом String"
val stringProducers: List[Producer[_ >: String]] = List(
  new Producer[String] { def produce() = "hello" },
  new Producer[CharSequence] { def produce() = "world" },
  new Producer[Any] { def produce() = 42 }
)
```

###### 27.5. Захват экзистенциальных типов:

```scala
// Проблема: тип стирается
def printFirst(list: List[_]): Unit = {
  if (list.nonEmpty) {
    val first: Any = list.head  // Потеряли информацию о типе
    println(first)
  }
}

// Решение: использовать полиморфизм
def printFirstPoly[A](list: List[A]): Unit = {
  if (list.nonEmpty) {
    val first: A = list.head  // Сохранили тип
    println(first)
  }
}

// Или использовать pattern matching для "захвата" типа:
def processBox(box: Box[_]): Unit = box match {
  case b: Box[t] => // Захватили экзистенциальный тип как 't'
    val value: t = b.get
    println(s"Value: $value")
}
```

###### 27.6. Java Interop:

```scala
// Java generic wildcards становятся existential types в Scala

// Java: List<?> → Scala: List[_]
import java.util.{List => JList}

def processJavaList(list: JList[_]): Unit = {
  val size: Int = list.size()
  println(s"List size: $size")
}

// Java: List<? extends Number> → Scala: List[_ <: Number]
def sumJavaNumbers(list: JList[_ <: Number]): Double = {
  import scala.jdk.CollectionConverters._
  list.asScala.map(_.doubleValue()).sum
}

// Java: List<? super Integer> → Scala: List[_ >: Integer]
def addInteger(list: JList[_ >: Integer]): Unit = {
  list.add(42)
}
```

###### 27.7. Когда использовать Existential Types:

```scala
// ✅ Хорошее использование:

// 1. Java interop
def processJavaCollection(coll: java.util.Collection[_]): Int = coll.size()

// 2. Хранение гетерогенных данных
val cache: Map[String, Box[_]] = Map(
  "int" -> new IntBox(42),
  "string" -> new StringBox("hello")
)

// 3. API с неизвестными типами
trait Repository {
  type Entity
  def findAll(): List[Entity]
}

def countEntities(repo: Repository): Int = repo.findAll().size

// ❌ Плохое использование:

// Вместо existential лучше использовать полиморфизм:
// Плохо:
def processList(list: List[_]): Unit = ???

// Хорошо:
def processList[A](list: List[A]): Unit = ???
```

###### 27.8. Deprecation в Scala 3:

⚠️ **Важно**: Existential types объявлены deprecated в Scala 3 и заменены на:
- Wildcard types (`_`)
- Match types
- Opaque types

```scala
// Scala 2:
def foo: List[T] forSome { type T <: Animal }

// Scala 3:
def foo: List[_ <: Animal]  // Используйте wildcard
```

---

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

---

#### 📖 Теоретические материалы

---

##### 28. Future и Promise

**Определение:**

Future представляет результат асинхронного вычисления, которое может быть еще не завершено. Promise - это writable, single-assignment контейнер, который завершает Future.

###### 28.1. Future - чтение результата асинхронного вычисления

```scala
import scala.concurrent.{Future, ExecutionContext}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Success, Failure}

// Создание Future
val future1: Future[Int] = Future {
  Thread.sleep(1000)
  42
}

// Future уже запущен! Вычисление началось немедленно
println("Future created")  // печатается сразу

// Обработка результата - callback подход
future1.onComplete {
  case Success(value) => println(s"Got value: $value")
  case Failure(exception) => println(s"Failed: ${exception.getMessage}")
}

// Блокирующее ожидание (не рекомендуется в production!)
import scala.concurrent.Await
import scala.concurrent.duration._

val result = Await.result(future1, 5.seconds)  // 42
```

**Характеристики Future:**
- ✅ Immutable - нельзя изменить после создания
- ✅ Single-assignment - результат устанавливается один раз
- ✅ Неблокирующий - не блокирует поток при создании
- ⚠️ Eager evaluation - вычисление начинается сразу

**Создание Future разными способами:**

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

// 1. Future.apply - запускает асинхронное вычисление
val future1 = Future {
  expensiveComputation()
}

// 2. Future.successful - уже завершенный Future с результатом
val future2: Future[Int] = Future.successful(42)

// 3. Future.failed - уже завершенный Future с ошибкой
val future3: Future[Int] = Future.failed(new Exception("Error"))

// 4. Future.fromTry
import scala.util.Try
val tryValue: Try[Int] = Try(42)
val future4: Future[Int] = Future.fromTry(tryValue)

// 5. Из Promise (см. ниже)
```

**Обработка результата Future:**

```scala
val future = Future {
  Thread.sleep(1000)
  42
}

// 1. onComplete - callback вызывается при завершении
future.onComplete {
  case Success(value) => println(s"Success: $value")
  case Failure(ex) => println(s"Failure: ${ex.getMessage}")
}

// 2. foreach - только для успешного результата
future.foreach { value =>
  println(s"Got: $value")
}

// 3. failed - Future[Throwable] для обработки ошибок
future.failed.foreach { exception =>
  println(s"Failed with: ${exception.getMessage}")
}

// 4. Трансформация с map/flatMap (см. раздел композиция)
val doubled = future.map(_ * 2)  // Future[Int]
```

---

###### 28.2. Promise - запись результата в Future

**Определение:**

Promise - это writable Future. Вы создаете Promise, получаете из него Future для чтения, и затем завершаете Promise значением или ошибкой.

```scala
import scala.concurrent.{Future, Promise}
import scala.concurrent.ExecutionContext.Implicits.global

// Создание Promise
val promise = Promise[Int]()

// Получение Future из Promise
val future: Future[Int] = promise.future

// Promise еще не завершен
println(future.isCompleted)  // false

// Завершение Promise успехом
promise.success(42)
// ИЛИ завершение с ошибкой:
// promise.failure(new Exception("Error"))

// Теперь Future завершен
println(future.isCompleted)  // true

// Можно читать результат
future.foreach(value => println(value))  // 42

// ⚠️ Попытка завершить Promise дважды вызовет исключение
// promise.success(100)  // IllegalStateException!

// Безопасная попытка завершить
val success = promise.trySuccess(100)  // false (уже завершен)
```

**Когда использовать Promise:**

```scala
// 1. Интеграция с callback-based API
def asyncOperation(callback: Int => Unit): Unit = {
  // Симуляция асинхронной операции
  new Thread(() => {
    Thread.sleep(1000)
    callback(42)
  }).start()
}

def convertToFuture(): Future[Int] = {
  val promise = Promise[Int]()
  
  asyncOperation { result =>
    promise.success(result)  // Завершаем Promise в callback
  }
  
  promise.future
}

val future = convertToFuture()
future.foreach(println)  // 42 (через 1 секунду)

// 2. Ручное управление завершением
class DataFetcher {
  private val promise = Promise[String]()
  val dataFuture: Future[String] = promise.future
  
  def startFetching(): Unit = {
    // Асинхронная загрузка данных
    Future {
      Thread.sleep(2000)
      "Data loaded"
    }.onComplete {
      case Success(data) => promise.success(data)
      case Failure(ex) => promise.failure(ex)
    }
  }
}

val fetcher = new DataFetcher()
fetcher.startFetching()
fetcher.dataFuture.foreach(println)  // "Data loaded" (через 2 секунды)

// 3. Координация нескольких асинхронных операций
def raceCondition[A](future1: Future[A], future2: Future[A]): Future[A] = {
  val promise = Promise[A]()
  
  future1.onComplete(promise.tryComplete)
  future2.onComplete(promise.tryComplete)
  
  promise.future  // Вернет результат того Future, который завершится первым
}

val slow = Future { Thread.sleep(3000); "slow" }
val fast = Future { Thread.sleep(1000); "fast" }

raceCondition(slow, fast).foreach(println)  // "fast"
```

**Promise API:**

```scala
val promise = Promise[Int]()

// Завершение
promise.success(42)                    // Успех
promise.failure(new Exception("err"))  // Ошибка
promise.complete(Success(42))          // Try[Int]
promise.complete(Failure(exception))   // Try[Int]

// Безопасное завершение (возвращает Boolean)
promise.trySuccess(42)                 // true если завершен, false если уже был завершен
promise.tryFailure(exception)          // аналогично
promise.tryComplete(tryValue)          // аналогично

// Проверка состояния
promise.isCompleted                    // Boolean

// Получение Future
val future: Future[Int] = promise.future
```

**Future vs Promise - сравнение:**

| Аспект | Future | Promise |
|--------|--------|---------|
| **Назначение** | Read-only результат | Write-once контейнер |
| **Создание** | `Future { code }` | `Promise[T]()` |
| **Завершение** | Автоматически | Вручную через `success/failure` |
| **Получение значения** | `onComplete`, `map`, etc. | Через `promise.future` |
| **Использование** | Композиция вычислений | Интеграция с callback API |

---

##### 29. ExecutionContext

**Определение:**

ExecutionContext - это thread pool, который выполняет асинхронные задачи Future. Он определяет, на каких потоках будут выполняться вычисления.

###### 29.1. Зачем нужен ExecutionContext

```scala
// Future требует implicit ExecutionContext
import scala.concurrent.Future

// ❌ Не компилируется без ExecutionContext
// val future = Future { 42 }
// Error: Cannot find an implicit ExecutionContext

// ✅ С ExecutionContext
import scala.concurrent.ExecutionContext.Implicits.global
val future = Future { 42 }  // OK
```

**ExecutionContext решает:**
- На каком потоке выполнить вычисление
- Сколько потоков использовать
- Как управлять очередью задач
- Что делать при перегрузке

---

###### 29.2. Глобальный ExecutionContext

```scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future

// global - это ForkJoinPool с количеством потоков = количество CPU cores
val future = Future {
  println(s"Running on: ${Thread.currentThread().getName}")
  42
}

// ⚠️ Проблемы с global:
// 1. Разделяется всем кодом в приложении
// 2. Blocking операции блокируют потоки для всех
// 3. Нет изоляции между компонентами
```

---

###### 29.3. Создание собственных ExecutionContext

```scala
import scala.concurrent.{Future, ExecutionContext}
import java.util.concurrent.Executors

// 1. Fixed thread pool
val fixedThreadPool = ExecutionContext.fromExecutor(
  Executors.newFixedThreadPool(10)
)

implicit val ec1: ExecutionContext = fixedThreadPool

val future1 = Future {
  println(s"Thread: ${Thread.currentThread().getName}")
  "Result"
}(ec1)  // Явное указание ExecutionContext

// 2. Cached thread pool (создает новые потоки по необходимости)
val cachedThreadPool = ExecutionContext.fromExecutor(
  Executors.newCachedThreadPool()
)

// 3. Single thread executor
val singleThread = ExecutionContext.fromExecutor(
  Executors.newSingleThreadExecutor()
)

// 4. Custom thread pool с настройками
import java.util.concurrent.ThreadPoolExecutor
import java.util.concurrent.TimeUnit
import java.util.concurrent.LinkedBlockingQueue

val customPool = new ThreadPoolExecutor(
  5,                           // corePoolSize
  10,                          // maximumPoolSize
  60L,                         // keepAliveTime
  TimeUnit.SECONDS,            // unit
  new LinkedBlockingQueue[Runnable](100)  // workQueue
)

val customEc = ExecutionContext.fromExecutor(customPool)
```

---

###### 29.4. Специализированные ExecutionContext

```scala
// Для blocking операций - отдельный thread pool!
object Contexts {
  // Для CPU-bound задач
  implicit val cpuBound: ExecutionContext = 
    ExecutionContext.fromExecutor(
      Executors.newFixedThreadPool(Runtime.getRuntime.availableProcessors())
    )
  
  // Для IO-bound и blocking операций
  val blockingIo: ExecutionContext = 
    ExecutionContext.fromExecutor(
      Executors.newCachedThreadPool()  // Может создавать много потоков
    )
}

// Использование
import Contexts._

// CPU-bound задача - используем cpuBound (implicit)
val computation = Future {
  // Сложные вычисления
  (1 to 1000000).sum
}

// Blocking IO - явно используем blockingIo
val dbQuery = Future {
  // Blocking database call
  Thread.sleep(5000)
  "Result from DB"
}(blockingIo)  // Явно указываем ExecutionContext
```

---

###### 29.5. Blocking в ExecutionContext

**Проблема:**

```scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future

// ❌ ПЛОХО - блокирует поток из global pool
val future = Future {
  Thread.sleep(10000)  // Blocking operation!
  "result"
}

// Если все потоки заблокированы, новые Future не могут выполняться!
// Это может привести к deadlock
```

**Решение 1: Отдельный ExecutionContext для blocking:**

```scala
import scala.concurrent.{Future, ExecutionContext, blocking}
import java.util.concurrent.Executors

// Отдельный pool для blocking операций
val blockingEc = ExecutionContext.fromExecutor(
  Executors.newCachedThreadPool()
)

// Используем blocking EC для blocking операций
val future = Future {
  Thread.sleep(10000)
  "result"
}(blockingEc)
```

**Решение 2: blocking { } - уведомление ExecutionContext:**

```scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.{Future, blocking}

val future = Future {
  blocking {
    // ExecutionContext знает, что это blocking операция
    // и может добавить дополнительные потоки
    Thread.sleep(10000)
  }
  "result"
}
```

---

###### 29.6. Best Practices для ExecutionContext

```scala
// ✅ ХОРОШО: Разные ExecutionContext для разных типов задач
object ExecutionContexts {
  // Для быстрых CPU-bound операций
  implicit val default: ExecutionContext = 
    ExecutionContext.fromExecutor(
      Executors.newFixedThreadPool(
        Runtime.getRuntime.availableProcessors()
      )
    )
  
  // Для blocking IO
  val io: ExecutionContext = 
    ExecutionContext.fromExecutor(
      Executors.newCachedThreadPool()
    )
  
  // Для database операций
  val database: ExecutionContext = 
    ExecutionContext.fromExecutor(
      Executors.newFixedThreadPool(20)  // Размер connection pool
    )
}

// Использование
def fetchFromDb(id: Long): Future[User] = Future {
  // Database call
  queryDb(id)
}(ExecutionContexts.database)  // Явно указываем DB context

def computeExpensive(data: Data): Future[Result] = Future {
  // CPU-intensive computation
  process(data)
}  // Использует default (implicit)

// ✅ ХОРОШО: Закрытие ExecutionContext при завершении
val ec = ExecutionContext.fromExecutor(
  Executors.newFixedThreadPool(10)
)

// При shutdown приложения
ec match {
  case executor: ExecutionContextExecutor =>
    executor.shutdown()
    executor.awaitTermination(60, TimeUnit.SECONDS)
}

// ❌ ПЛОХО: Создание нового ExecutionContext для каждого Future
def badExample(): Future[String] = {
  val ec = ExecutionContext.fromExecutor(Executors.newSingleThreadExecutor())
  Future {
    "result"
  }(ec)
}
// Создает новый thread pool при каждом вызове!

// ✅ ХОРОШО: Переиспользование ExecutionContext
object Config {
  val ec: ExecutionContext = ExecutionContext.fromExecutor(
    Executors.newFixedThreadPool(10)
  )
}

def goodExample(): Future[String] = {
  Future {
    "result"
  }(Config.ec)
}
```

---

##### 30. Future Composition (Композиция Future)

**Определение:**

Future composition - это способы комбинирования нескольких Future в более сложные асинхронные вычисления.

###### 30.1. map - трансформация результата

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

val future: Future[Int] = Future { 42 }

// map трансформирует значение внутри Future
val doubled: Future[Int] = future.map(_ * 2)  // Future[Int] = 84
val asString: Future[String] = future.map(_.toString)  // Future[String] = "42"

// Цепочка map
val result: Future[String] = Future { 10 }
  .map(_ * 2)           // Future[Int] = 20
  .map(_ + 5)           // Future[Int] = 25
  .map(_.toString)      // Future[String] = "25"

result.foreach(println)  // "25"
```

---

###### 30.2. flatMap - последовательные асинхронные операции

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def getUser(id: Long): Future[User] = Future {
  Thread.sleep(100)
  User(id, "Alice")
}

def getOrders(userId: Long): Future[List[Order]] = Future {
  Thread.sleep(100)
  List(Order(1, userId, 100.0), Order(2, userId, 200.0))
}

// map создает Future[Future[List[Order]]]
val nested: Future[Future[List[Order]]] = getUser(1).map { user =>
  getOrders(user.id)
}

// flatMap "уплощает" вложенные Future
val orders: Future[List[Order]] = getUser(1).flatMap { user =>
  getOrders(user.id)
}

// Цепочка flatMap
val total: Future[Double] = 
  getUser(1).flatMap { user =>
    getOrders(user.id).map { orders =>
      orders.map(_.amount).sum
    }
  }

total.foreach(println)  // 300.0
```

---

###### 30.3. for-comprehension для Future

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def getUser(id: Long): Future[User] = ???
def getOrders(userId: Long): Future[List[Order]] = ???
def calculateTotal(orders: List[Order]): Future[Double] = ???

// Вместо вложенных flatMap
val result1: Future[Double] = 
  getUser(1).flatMap { user =>
    getOrders(user.id).flatMap { orders =>
      calculateTotal(orders)
    }
  }

// Используем for-comprehension (синтаксический сахар)
val result2: Future[Double] = for {
  user <- getUser(1)
  orders <- getOrders(user.id)
  total <- calculateTotal(orders)
} yield total

// ⚠️ ВАЖНО: операции выполняются последовательно!
// getOrders ждет завершения getUser
// calculateTotal ждет завершения getOrders
```

---

###### 30.4. Параллельное выполнение Future

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

// ❌ Последовательное выполнение (медленно)
val sequential: Future[(String, String, String)] = for {
  result1 <- Future { Thread.sleep(1000); "A" }
  result2 <- Future { Thread.sleep(1000); "B" }
  result3 <- Future { Thread.sleep(1000); "C" }
} yield (result1, result2, result3)
// Займет 3 секунды

// ✅ Параллельное выполнение (быстро)
val future1 = Future { Thread.sleep(1000); "A" }
val future2 = Future { Thread.sleep(1000); "B" }
val future3 = Future { Thread.sleep(1000); "C" }

val parallel: Future[(String, String, String)] = for {
  result1 <- future1
  result2 <- future2
  result3 <- future3
} yield (result1, result2, result3)
// Займет 1 секунду (параллельно)!

// Объяснение:
// Future запускается немедленно при создании (eager)
// future1, future2, future3 начинают выполняться сразу
// for-comprehension только ждет их результаты
```

---

###### 30.5. Future.sequence - List[Future] → Future[List]

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def fetchUrl(url: String): Future[String] = Future {
  Thread.sleep(1000)
  s"Content of $url"
}

val urls = List(
  "http://example.com/1",
  "http://example.com/2",
  "http://example.com/3"
)

// Создаем List[Future[String]]
val futures: List[Future[String]] = urls.map(fetchUrl)

// sequence превращает в Future[List[String]]
val allResults: Future[List[String]] = Future.sequence(futures)

allResults.foreach { results =>
  results.foreach(println)
}

// ⚠️ ВАЖНО: sequence ждет завершения ВСЕХ Future
// Если хотя бы один Future упадет с ошибкой, весь результат будет Failure
```

---

###### 30.6. Future.traverse - комбинация map + sequence

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def fetchUrl(url: String): Future[String] = ???

val urls = List("url1", "url2", "url3")

// Вместо map + sequence
val results1 = Future.sequence(urls.map(fetchUrl))

// Используем traverse (эффективнее)
val results2 = Future.traverse(urls)(fetchUrl)

// traverse = map + sequence в одной операции
// Future.traverse(urls)(fetchUrl) ≡ Future.sequence(urls.map(fetchUrl))
```

---

###### 30.7. Future.foldLeft и Future.reduceLeft

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

val futures = List(
  Future(1),
  Future(2),
  Future(3),
  Future(4)
)

// foldLeft - последовательная обработка с аккумулятором
val sum: Future[Int] = Future.foldLeft(futures)(0)(_ + _)
sum.foreach(println)  // 10

// Практический пример - последовательные операции
def updateRecord(id: Int): Future[Unit] = Future {
  println(s"Updating record $id")
  Thread.sleep(500)
}

val recordIds = List(1, 2, 3, 4, 5)

// Обновляем записи последовательно (не перегружаем базу)
val updates: Future[Unit] = Future.foldLeft(recordIds)(()) { (_, id) =>
  updateRecord(id).map(_ => ())
}
```

---

###### 30.8. zip и zipWith - комбинирование двух Future

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

val future1: Future[Int] = Future { 42 }
val future2: Future[String] = Future { "hello" }

// zip - комбинирует в кортеж
val zipped: Future[(Int, String)] = future1.zip(future2)
zipped.foreach { case (num, str) =>
  println(s"$num and $str")  // "42 and hello"
}

// zipWith - комбинирует с функцией
val combined: Future[String] = future1.zipWith(future2) { (num, str) =>
  s"$str $num"
}
combined.foreach(println)  // "hello 42"
```

---

###### 30.9. Композиция с andThen

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Success, Failure}

val future = Future { 42 }

// andThen - для side effects (логирование, метрики)
future
  .andThen { case Success(v) => println(s"Got value: $v") }
  .andThen { case Failure(e) => println(s"Failed: $e") }
  .map(_ * 2)
  .andThen { case Success(v) => println(s"After doubling: $v") }

// andThen НЕ трансформирует результат
// Он только выполняет side effect и возвращает тот же Future
```

---

##### 31. Error Handling в Future

**Проблема:**

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

val future: Future[Int] = Future {
  throw new Exception("Something went wrong!")
  42
}

// Ошибка не бросается сразу!
// Future захватывает exception

future.onComplete {
  case Success(value) => println(value)
  case Failure(exception) => println(s"Error: ${exception.getMessage}")
}
// Напечатает: "Error: Something went wrong!"
```

---

###### 31.1. recover - обработка ошибки с fallback значением

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def divide(a: Int, b: Int): Future[Int] = Future {
  if (b == 0) throw new ArithmeticException("Division by zero")
  a / b
}

// recover - возвращает значение при ошибке
val result: Future[Int] = divide(10, 0).recover {
  case _: ArithmeticException => 0  // Возвращаем 0 при делении на ноль
}

result.foreach(println)  // 0

// Более сложный пример
val userData: Future[User] = fetchUser(userId).recover {
  case _: NotFoundException => User.anonymous  // Default user
  case _: TimeoutException => User.cached      // Cached user
  case other => throw other  // Пробрасываем другие ошибки дальше
}
```

---

###### 31.2. recoverWith - обработка ошибки с альтернативным Future

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def fetchFromPrimary(): Future[String] = Future {
  throw new Exception("Primary failed!")
}

def fetchFromBackup(): Future[String] = Future {
  "Data from backup"
}

// recoverWith - запускает альтернативный Future при ошибке
val result: Future[String] = fetchFromPrimary().recoverWith {
  case _: Exception => fetchFromBackup()
}

result.foreach(println)  // "Data from backup"

// Цепочка fallback'ов
def fetchWithFallbacks(): Future[String] = 
  fetchFromPrimary()
    .recoverWith { case _ => fetchFromBackup() }
    .recoverWith { case _ => fetchFromCache() }
    .recover { case _ => "Default value" }
```

---

###### 31.3. fallbackTo - альтернативный Future

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

val primary = Future {
  throw new Exception("Failed!")
}

val backup = Future {
  "Backup result"
}

// fallbackTo - использует backup если primary упал
val result: Future[String] = primary.fallbackTo(backup)

result.foreach(println)  // "Backup result"

// ⚠️ ВАЖНО: backup Future запускается немедленно (параллельно)!
// Это отличается от recoverWith, где backup запускается только при ошибке
```

---

###### 31.4. transform и transformWith - универсальная трансформация

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Try, Success, Failure}

val future = Future { 42 / 0 }

// transform - трансформация Try[A] => Try[B]
val transformed: Future[String] = future.transform {
  case Success(value) => Success(s"Result: $value")
  case Failure(exception) => Success(s"Error: ${exception.getMessage}")
}

transformed.foreach(println)  // "Error: / by zero"

// transformWith - трансформация Try[A] => Future[B]
val transformedWith: Future[String] = future.transformWith {
  case Success(value) => Future.successful(s"Result: $value")
  case Failure(exception) => Future {
    // Логирование в базу
    logError(exception)
    s"Error: ${exception.getMessage}"
  }
}
```

---

###### 31.5. Паттерн Retry

```scala
import scala.concurrent.{Future, Promise}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Success, Failure}

def retry[A](f: => Future[A], retries: Int): Future[A] = {
  f.recoverWith {
    case exception if retries > 0 =>
      println(s"Failed, retrying... ($retries retries left)")
      retry(f, retries - 1)
    case exception =>
      Future.failed(exception)
  }
}

// Использование
def unreliableOperation(): Future[String] = Future {
  if (scala.util.Random.nextBoolean()) 
    throw new Exception("Random failure")
  "Success!"
}

val result = retry(unreliableOperation(), 3)

result.onComplete {
  case Success(value) => println(s"Finally succeeded: $value")
  case Failure(exception) => println(s"Failed after retries: ${exception.getMessage}")
}
```

---

###### 31.6. Паттерн Timeout

```scala
import scala.concurrent.{Future, Promise}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration._
import java.util.concurrent.TimeoutException

def withTimeout[A](future: Future[A], timeout: Duration): Future[A] = {
  val promise = Promise[A]()
  
  // Таймер для timeout
  val timer = new java.util.Timer()
  timer.schedule(
    new java.util.TimerTask {
      def run(): Unit = {
        promise.tryFailure(new TimeoutException(s"Timeout after $timeout"))
        timer.cancel()
      }
    },
    timeout.toMillis
  )
  
  // Гонка между future и timeout
  future.onComplete { result =>
    promise.tryComplete(result)
    timer.cancel()
  }
  
  promise.future
}

// Использование
val slowOperation = Future {
  Thread.sleep(5000)
  "Done"
}

val result = withTimeout(slowOperation, 2.seconds)

result.onComplete {
  case Success(value) => println(s"Completed: $value")
  case Failure(_: TimeoutException) => println("Timed out!")
  case Failure(ex) => println(s"Failed: ${ex.getMessage}")
}
```

---

##### 32. Actor Model Basics (Akka)

**Определение:**

Actor Model - это модель конкурентности, где actors (акторы) - это примитивы вычислений, которые:
- Обрабатывают сообщения последовательно (one message at a time)
- Имеют собственное изолированное состояние
- Взаимодействуют только через асинхронные сообщения
- Не разделяют память (no shared mutable state)

###### 32.1. Базовый Actor

```scala
import akka.actor.{Actor, ActorSystem, Props}

// 1. Определяем сообщения
case class Greeting(message: String)
case class Goodbye(message: String)

// 2. Создаем Actor
class GreeterActor extends Actor {
  def receive: Receive = {
    case Greeting(message) =>
      println(s"Received greeting: $message")
      sender() ! "Hello back!"  // Отправка ответа
    
    case Goodbye(message) =>
      println(s"Goodbye: $message")
      context.stop(self)  // Остановка актора
    
    case _ =>
      println("Unknown message")
  }
}

// 3. Создание ActorSystem и Actor
val system = ActorSystem("MySystem")
val greeter = system.actorOf(Props[GreeterActor], "greeter")

// 4. Отправка сообщений
greeter ! Greeting("Hi there!")  // Fire-and-forget (!)
greeter ! Goodbye("See you!")

// 5. Shutdown
system.terminate()
```

**Ключевые концепции:**

- `!` (tell) - fire-and-forget, асинхронная отправка без ожидания ответа
- `sender()` - ссылка на отправителя сообщения
- `context.stop(self)` - остановка актора
- `receive` - partial function для обработки сообщений

---

###### 32.2. Actor с состоянием

```scala
import akka.actor.{Actor, Props}

// Сообщения
case class Deposit(amount: Double)
case class Withdraw(amount: Double)
case object GetBalance

// Actor с mutable состоянием (изолированным!)
class BankAccountActor extends Actor {
  private var balance: Double = 0.0  // Mutable, но изолировано в акторе
  
  def receive: Receive = {
    case Deposit(amount) =>
      balance += amount
      println(s"Deposited $amount, new balance: $balance")
      sender() ! balance
    
    case Withdraw(amount) =>
      if (amount <= balance) {
        balance -= amount
        sender() ! Some(balance)
      } else {
        sender() ! None
      }
    
    case GetBalance =>
      sender() ! balance
  }
}

// Использование
val account = system.actorOf(Props[BankAccountActor], "account")

account ! Deposit(100.0)
account ! Withdraw(30.0)
account ! GetBalance
```

---

###### 32.3. Ask Pattern (?) - запрос с ответом

```scala
import akka.actor.{Actor, ActorSystem, Props}
import akka.pattern.ask
import akka.util.Timeout
import scala.concurrent.duration._
import scala.concurrent.Future

class CalculatorActor extends Actor {
  def receive: Receive = {
    case ("add", a: Int, b: Int) => sender() ! (a + b)
    case ("multiply", a: Int, b: Int) => sender() ! (a * b)
  }
}

val system = ActorSystem("MySystem")
val calculator = system.actorOf(Props[CalculatorActor], "calculator")

// Ask pattern требует implicit Timeout
implicit val timeout: Timeout = Timeout(5.seconds)
import system.dispatcher  // ExecutionContext

// ? возвращает Future
val resultFuture: Future[Any] = calculator ? ("add", 10, 20)

// Приведение типа
val result: Future[Int] = resultFuture.mapTo[Int]

result.foreach(println)  // 30

// ⚠️ Ask pattern менее эффективен чем tell (!)
// Используйте только когда действительно нужен ответ
```

---

###### 32.4. Actor Lifecycle

```scala
import akka.actor.{Actor, Props}

class LifecycleActor extends Actor {
  
  // Вызывается при старте актора
  override def preStart(): Unit = {
    println("Actor starting")
    // Инициализация ресурсов
  }
  
  // Вызывается после остановки актора
  override def postStop(): Unit = {
    println("Actor stopped")
    // Освобождение ресурсов
  }
  
  // Вызывается перед рестартом
  override def preRestart(reason: Throwable, message: Option[Any]): Unit = {
    println(s"Actor restarting due to: ${reason.getMessage}")
    super.preRestart(reason, message)
  }
  
  // Вызывается после рестарта
  override def postRestart(reason: Throwable): Unit = {
    println("Actor restarted")
    super.postRestart(reason)
  }
  
  def receive: Receive = {
    case "crash" => throw new Exception("Intentional crash")
    case msg => println(s"Received: $msg")
  }
}
```

---

###### 32.5. Supervision (Надзор)

**Стратегии надзора:**

```scala
import akka.actor.{Actor, Props, OneForOneStrategy, SupervisorStrategy}
import akka.actor.SupervisorStrategy._
import scala.concurrent.duration._

class Supervisor extends Actor {
  
  // Стратегия надзора
  override val supervisorStrategy: SupervisorStrategy =
    OneForOneStrategy(maxNrOfRetries = 3, withinTimeRange = 1.minute) {
      case _: ArithmeticException => Resume        // Продолжить
      case _: NullPointerException => Restart      // Перезапустить
      case _: IllegalArgumentException => Stop     // Остановить
      case _: Exception => Escalate                // Передать выше
    }
  
  def receive: Receive = {
    case props: Props => sender() ! context.actorOf(props)
  }
}

class WorkerActor extends Actor {
  def receive: Receive = {
    case "crash" => throw new Exception("Crash!")
    case msg => println(s"Processing: $msg")
  }
}

// Создание иерархии
val supervisor = system.actorOf(Props[Supervisor], "supervisor")
val worker = system.actorOf(Props[WorkerActor], "worker")

worker ! "crash"  // Actor будет перезапущен supervisor'ом
```

**Стратегии:**
- **Resume** - продолжить с текущим состоянием
- **Restart** - перезапустить актор (состояние сбрасывается)
- **Stop** - остановить актор
- **Escalate** - передать ошибку родительскому актору

---

###### 32.6. Actor vs Future - когда что использовать

**Future:**
```scala
// ✅ Используйте Future для:
// - Одноразовых асинхронных вычислений
// - Простой композиции операций
// - Когда не нужно состояние
val result = for {
  user <- fetchUser(id)
  orders <- fetchOrders(user.id)
} yield orders
```

**Actor:**
```scala
// ✅ Используйте Actor для:
// - Управления mutable состоянием
// - Длительно живущих сущностей
// - Сложной координации между компонентами
// - Когда нужны lifecycle hooks и supervision
class UserSessionActor extends Actor {
  private var state: SessionState = ???
  // Обработка сообщений с изменением состояния
}
```

**Сравнение:**

| Аспект | Future | Actor |
|--------|--------|-------|
| **Lifecycle** | Одноразовый | Долгоживущий |
| **Состояние** | Нет | Да (изолированное) |
| **Композиция** | map, flatMap, for | Иерархии, supervision |
| **Ошибки** | recover, fallbackTo | Supervision strategies |
| **Использование** | Async вычисления | Stateful entities |

---

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

