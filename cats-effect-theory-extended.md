# Полное Руководство по Cats и Cats Effect - Теория и Практика

## Содержание

### Часть I: Теоретические Основы
1. [Category Theory - Глубокое Погружение](#1-category-theory---глубокое-погружение)
2. [Функторы - Теория и Практика](#2-функторы---теория-и-практика)
3. [Моноиды и Группы](#3-моноиды-и-группы)
4. [Монады - Математические Основы](#4-монады---математические-основы)

### Часть II: Практические Type Classes
5. [Cats Core Type Classes](#5-cats-core-type-classes)
6. [Applicative и Apply](#6-applicative-и-apply)
7. [Traverse и Foldable](#7-traverse-и-foldable)

### Часть III: Продвинутые Концепции
8. [Monad Transformers](#8-monad-transformers)
9. [Free Monads и Interpreters](#9-free-monads-и-interpreters)
10. [Tagless Final и MTL](#10-tagless-final-и-mtl)

### Часть IV: Cats Effect
11. [IO Monad - Теория и Реализация](#11-io-monad---теория-и-реализация)
12. [Concurrency Primitives](#12-concurrency-primitives)
13. [Resource Management](#13-resource-management)
14. [Streaming с FS2](#14-streaming-с-fs2)

---

# Часть I: Теоретические Основы

## 1. Category Theory - Глубокое Погружение

### 1.1 Математические Основы

**Определение 1.1 (Категория):**
Категория **C** - это математическая структура, состоящая из:

1. **Класс объектов** Ob(C)
2. **Для каждой пары объектов A, B** - множество морфизмов Hom(A,B) или C(A,B)
3. **Для каждой тройки объектов A, B, C** - операция композиции:
   ```
   ∘ : Hom(B,C) × Hom(A,B) → Hom(A,C)
   ```
4. **Для каждого объекта A** - тождественный морфизм id_A ∈ Hom(A,A)

**Аксиомы категории:**

**Аксиома 1 (Ассоциативность композиции):**
Для любых морфизмов f: A → B, g: B → C, h: C → D выполняется:
```
h ∘ (g ∘ f) = (h ∘ g) ∘ f
```

**Аксиома 2 (Левая единичность):**
Для любого морфизма f: A → B:
```
id_B ∘ f = f
```

**Аксиома 3 (Правая единичность):**
Для любого морфизма f: A → B:
```
f ∘ id_A = f
```

#### Диаграммы коммутативности

**Определение 1.2:** Диаграмма коммутативна, если любые два пути между одними и теми же объектами дают один и тот же морфизм.

```
Коммутативная диаграмма:

    A ─────f────→ B
    │             │
    │g            │h
    │             │
    ↓             ↓
    C ─────k────→ D

Коммутирует если: h ∘ f = k ∘ g
```

#### Примеры категорий

**Пример 1.1 (Set):** Категория множеств
- Объекты: все множества
- Морфизмы: функции между множествами
- Композиция: стандартная композиция функций (g ∘ f)(x) = g(f(x))
- Identity: тождественная функция id(x) = x

**Пример 1.2 (Type):** Категория типов в Scala
```scala
// Объекты: типы
type A = Int
type B = String
type C = Boolean

// Морфизмы: функции
val f: A => B = _.toString
val g: B => C = _.nonEmpty

// Композиция
val h: A => C = g compose f  // или f andThen g

// Identity
val idInt: Int => Int = x => x
```

**Пример 1.3 (Kleisli Category):** Категория Клейсли для монады M
- Объекты: типы A, B, C, ...
- Морфизмы: функции вида A => M[B] (Kleisli arrows)
- Композиция: через flatMap
- Identity: a => M.pure(a)

```scala
// Для монады Option
type Kleisli[A, B] = A => Option[B]

// Композиция Kleisli arrows
def composeK[A, B, C](f: A => Option[B], g: B => Option[C]): A => Option[C] =
  a => f(a).flatMap(g)

// Identity
def pureK[A]: A => Option[A] = a => Some(a)

// Проверка законов
val f: Int => Option[String] = x => Some(x.toString)
val g: String => Option[Boolean] = s => Some(s.nonEmpty)

// Left identity: pureK >=> f = f
composeK(pureK, f) // эквивалентно f

// Right identity: f >=> pureK = f
composeK(f, pureK) // эквивалентно f
```

### 1.2 Функторы

**Определение 1.3 (Функтор):**
Функтор F: C → D между категориями C и D - это отображение, которое:

1. **Каждому объекту A ∈ Ob(C)** сопоставляет объект F(A) ∈ Ob(D)
2. **Каждому морфизму f: A → B в C** сопоставляет морфизм F(f): F(A) → F(B) в D

**При этом выполняются законы функтора:**

**Закон 1 (Сохранение identity):**
```
F(id_A) = id_{F(A)}
```

**Закон 2 (Сохранение композиции):**
```
F(g ∘ f) = F(g) ∘ F(f)
```

#### Визуализация функтора

```
Категория C              Функтор F              Категория D
┌──────────┐                                   ┌──────────┐
│    A     │────────────────────────────────→ │   F(A)   │
└────┬─────┘                                   └────┬─────┘
     │f                                             │F(f)
     ↓                                              ↓
┌──────────┐                                   ┌──────────┐
│    B     │────────────────────────────────→ │   F(B)   │
└──────────┘                                   └──────────┘

F(f ∘ g) = F(f) ∘ F(g)
F(id_A) = id_{F(A)}
```

#### Ковариантный функтор в Scala

```scala
trait Functor[F[_]] {
  // Отображение морфизмов
  def map[A, B](fa: F[A])(f: A => B): F[B]
  
  // Законы (должны выполняться):
  // 1. Identity: fa.map(identity) == fa
  // 2. Composition: fa.map(f).map(g) == fa.map(f andThen g)
}
```

**Пример 1.4:** List как функтор
```scala
implicit val listFunctor: Functor[List] = new Functor[List] {
  def map[A, B](fa: List[A])(f: A => B): List[B] = fa.map(f)
}

// Проверка законов
val list = List(1, 2, 3)

// Identity law
list.map(identity) == list  // true

// Composition law
val f: Int => String = _.toString
val g: String => Int = _.length

list.map(f).map(g) == list.map(f andThen g)  // true
```

**Определение 1.4 (Контравариантный функтор):**
Контравариантный функтор F: C^op → D - это функтор из противоположной категории C^op в D.

Для морфизма f: A → B в C, контравариантный функтор дает F(f): F(B) → F(A) в D.

```scala
trait Contravariant[F[_]] {
  def contramap[A, B](fa: F[A])(f: B => A): F[B]
  
  // Законы:
  // 1. Identity: fa.contramap(identity) == fa
  // 2. Composition: fa.contramap(f).contramap(g) == fa.contramap(g andThen f)
}
```

**Пример 1.5:** Encoder как контравариантный функтор
```scala
trait Encoder[A] {
  def encode(value: A): String
}

implicit val contravariantEncoder: Contravariant[Encoder] = 
  new Contravariant[Encoder] {
    def contramap[A, B](fa: Encoder[A])(f: B => A): Encoder[B] =
      new Encoder[B] {
        def encode(value: B): String = fa.encode(f(value))
      }
  }

// Int Encoder
implicit val intEncoder: Encoder[Int] = (value: Int) => value.toString

// Boolean Encoder через contramap
implicit val boolEncoder: Encoder[Boolean] = 
  intEncoder.contramap(b => if (b) 1 else 0)

boolEncoder.encode(true)  // "1"
boolEncoder.encode(false) // "0"
```

### 1.3 Естественные преобразования

**Определение 1.5 (Естественное преобразование):**
Пусть F, G: C → D - два функтора. Естественное преобразование η: F ⇒ G - это семейство морфизмов:

```
η_A: F(A) → G(A)  для каждого объекта A в C
```

такое, что для любого морфизма f: A → B в C следующая диаграмма коммутирует:

```
F(A) ──η_A──→ G(A)
 │             │
 │F(f)         │G(f)
 ↓             ↓
F(B) ──η_B──→ G(B)

Коммутирует: G(f) ∘ η_A = η_B ∘ F(f)
```

**Условие натуральности:** η_B ∘ F(f) = G(f) ∘ η_A

#### Natural Transformation в Scala

```scala
// Естественное преобразование как trait
trait NaturalTransformation[F[_], G[_]] {
  def apply[A](fa: F[A]): G[A]
}

// Сокращенная запись
trait ~>[F[_], G[_]] {
  def apply[A](fa: F[A]): G[A]
}

// Пример: Option ~> List
val optionToList: Option ~> List = new (Option ~> List) {
  def apply[A](fa: Option[A]): List[A] = fa.toList
}

// Проверка натуральности
val f: Int => String = _.toString
val opt: Option[Int] = Some(42)

// Должно быть: listFunctor.map(optionToList(opt))(f) == optionToList(optionFunctor.map(opt)(f))
optionToList(opt).map(f) == optionToList(opt.map(f))  // true
```

**Пример 1.6:** head как естественное преобразование List ~> Option
```scala
val headNat: List ~> Option = new (List ~> Option) {
  def apply[A](fa: List[A]): Option[A] = fa.headOption
}

// Натуральность:
val list = List(1, 2, 3)
val f: Int => String = _.toString

headNat(list).map(f) == headNat(list.map(f))  // true
// Some(1).map(f) == headNat(List("1", "2", "3"))
// Some("1") == Some("1")
```

### 1.4 Моноидальные категории

**Определение 1.6 (Моноидальная категория):**
Моноидальная категория (C, ⊗, I) - это категория C с:

1. **Бифунктором** ⊗: C × C → C (тензорное произведение)
2. **Единичным объектом** I ∈ Ob(C)
3. **Естественными изоморфизмами:**
   - Ассоциатор: α: (A ⊗ B) ⊗ C ≅ A ⊗ (B ⊗ C)
   - Левый унитор: λ: I ⊗ A ≅ A
   - Правый унитор: ρ: A ⊗ I ≅ A

**Удовлетворяющими пентагональному и треугольному тождествам.**

#### Пентагональное тождество

```
((A⊗B)⊗C)⊗D ──────→ (A⊗B)⊗(C⊗D) ──────→ A⊗(B⊗(C⊗D))
      │                                      ↑
      │                                      │
      ↓                                      │
(A⊗(B⊗C))⊗D ──────────────────────→ A⊗((B⊗C)⊗D)
```

**В Scala:**
```scala
// Tuple как тензорное произведение
type ⊗[A, B] = (A, B)

// Единичный объект
type I = Unit

// Примеры изоморфизмов
def leftUnitor[A]: (Unit, A) => A = { case (_, a) => a }
def leftUnitorInv[A]: A => (Unit, A) = a => ((), a)

def rightUnitor[A]: (A, Unit) => A = { case (a, _) => a }
def rightUnitorInv[A]: A => (A, Unit) = a => (a, ())

// Ассоциатор
def associator[A, B, C]: ((A, B), C) => (A, (B, C)) = {
  case ((a, b), c) => (a, (b, c))
}
def associatorInv[A, B, C]: (A, (B, C)) => ((A, B), C) = {
  case (a, (b, c)) => ((a, b), c)
}
```

---

## 2. Функторы - Теория и Практика

### 2.1 Теория функторов

**Определение 2.1:** Эндофунктор - это функтор F: C → C из категории в себя.

В Scala все функторы - эндофункторы в категории Type:
```scala
// F[_] переводит тип A в тип F[A]
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

#### Свойства функторов

**Теорема 2.1:** Функтор сохраняет изоморфизмы.
```
Если f: A ≅ B изоморфизм, то F(f): F(A) ≅ F(B) тоже изоморфизм.
```

**Доказательство:**
Пусть f: A → B и g: B → A такие, что g ∘ f = id_A и f ∘ g = id_B.

Применяя F:
```
F(g) ∘ F(f) = F(g ∘ f) = F(id_A) = id_{F(A)}
F(f) ∘ F(g) = F(f ∘ g) = F(id_B) = id_{F(B)}
```

Следовательно, F(f) - изоморфизм с обратным F(g). ∎

### 2.2 Производные операции функторов

```scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
  
  // Производные операции:
  
  // void - отбрасывает значение
  def void[A](fa: F[A]): F[Unit] = 
    map(fa)(_ => ())
  
  // as - заменяет значение
  def as[A, B](fa: F[A], b: B): F[B] = 
    map(fa)(_ => b)
  
  // fproduct - создает пару (a, f(a))
  def fproduct[A, B](fa: F[A])(f: A => B): F[(A, B)] = 
    map(fa)(a => (a, f(a)))
  
  // tupleLeft/tupleRight - добавляет элемент к кортежу
  def tupleLeft[A, B](fa: F[A], b: B): F[(B, A)] = 
    map(fa)(a => (b, a))
  
  def tupleRight[A, B](fa: F[A], b: B): F[(A, B)] = 
    map(fa)(a => (a, b))
  
  // widen - расширение типа
  def widen[A, B >: A](fa: F[A]): F[B] = 
    fa.asInstanceOf[F[B]]
  
  // lift - поднятие функции в функторный контекст
  def lift[A, B](f: A => B): F[A] => F[B] = 
    fa => map(fa)(f)
}
```

### 2.3 Композиция функторов

**Теорема 2.2:** Композиция двух функторов - тоже функтор.

```
Если F: C → D и G: D → E функторы, 
то G ∘ F: C → E тоже функтор.
```

**В Scala:**
```scala
// Композиция функторов
def compose[F[_]: Functor, G[_]: Functor]: Functor[Lambda[X => F[G[X]]]] = 
  new Functor[Lambda[X => F[G[X]]]] {
    def map[A, B](fga: F[G[A]])(f: A => B): F[G[B]] =
      Functor[F].map(fga)(ga => Functor[G].map(ga)(f))
  }

// Пример
type ListOption[A] = List[Option[A]]

val listOptionFunctor: Functor[ListOption] = compose[List, Option]

val data: List[Option[Int]] = List(Some(1), None, Some(3))
listOptionFunctor.map(data)(_ * 2) // List(Some(2), None, Some(6))
```

### 2.4 Инвариантный функтор

**Определение 2.2 (Инвариантный функтор):**
Инвариантный функтор - это обобщение, позволяющее преобразовывать F[A] в F[B] при наличии функций в обе стороны.

```scala
trait Invariant[F[_]] {
  def imap[A, B](fa: F[A])(f: A => B)(g: B => A): F[B]
  
  // Законы:
  // 1. Identity: imap(fa)(identity)(identity) == fa
  // 2. Composition: 
  //    imap(imap(fa)(f1)(g1))(f2)(g2) == imap(fa)(f2 ∘ f1)(g1 ∘ g2)
}
```

**Связь между Functor, Contravariant и Invariant:**

```
Covariant Functor:     нужна только f: A => B
Contravariant Functor: нужна только g: B => A
Invariant Functor:     нужны обе f: A => B и g: B => A
```

```scala
// Любой Functor - это Invariant
implicit def functorIsInvariant[F[_]: Functor]: Invariant[F] = 
  new Invariant[F] {
    def imap[A, B](fa: F[A])(f: A => B)(g: B => A): F[B] =
      Functor[F].map(fa)(f)
  }

// Любой Contravariant - это Invariant
implicit def contravariantIsInvariant[F[_]: Contravariant]: Invariant[F] = 
  new Invariant[F] {
    def imap[A, B](fa: F[A])(f: A => B)(g: B => A): F[B] =
      Contravariant[F].contramap(fa)(g)
  }
```

---

## 3. Моноиды и Группы

### 3.1 Алгебраические структуры

**Определение 3.1 (Магма):**
Магма (M, •) - это множество M с бинарной операцией •: M × M → M.

**Определение 3.2 (Полугруппа / Semigroup):**
Полугруппа (S, •) - это магма, в которой операция • ассоциативна:
```
∀ a, b, c ∈ S: (a • b) • c = a • (b • c)
```

**Определение 3.3 (Моноид / Monoid):**
Моноид (M, •, e) - это полугруппа с единичным элементом e:
```
1. Ассоциативность: (a • b) • c = a • (b • c)
2. Левая единичность: e • a = a
3. Правая единичность: a • e = a
```

**Определение 3.4 (Группа / Group):**
Группа (G, •, e, ⁻¹) - это моноид, в котором каждый элемент имеет обратный:
```
∀ a ∈ G: ∃ a⁻¹ ∈ G: a • a⁻¹ = e и a⁻¹ • a = e
```

#### Иерархия алгебраических структур

```
Магма (Magma)
    │
    │ + ассоциативность
    ↓
Полугруппа (Semigroup)
    │
    │ + единичный элемент
    ↓
Моноид (Monoid)
    │
    │ + обратные элементы
    ↓
Группа (Group)
```

### 3.2 Semigroup в Cats

```scala
import cats.Semigroup

trait Semigroup[A] {
  def combine(x: A, y: A): A
  
  // Закон ассоциативности:
  // combine(combine(x, y), z) == combine(x, combine(y, z))
}

object Semigroup {
  // Summoner
  def apply[A](implicit instance: Semigroup[A]): Semigroup[A] = instance
  
  // Constructor
  def instance[A](f: (A, A) => A): Semigroup[A] = (x, y) => f(x, y)
}
```

**Примеры полугрупп:**

```scala
// 1. Аддитивная полугруппа целых чисел (ℤ, +)
implicit val intAdditionSemigroup: Semigroup[Int] = 
  Semigroup.instance(_ + _)

// 2. Мультипликативная полугруппа целых чисел (ℤ, ×)
implicit val intMultiplicationSemigroup: Semigroup[Int] = 
  Semigroup.instance(_ * _)

// 3. Полугруппа строк с конкатенацией (String, ++)
implicit val stringSemigroup: Semigroup[String] = 
  Semigroup.instance(_ ++ _)

// 4. Полугруппа списков с конкатенацией
implicit def listSemigroup[A]: Semigroup[List[A]] = 
  Semigroup.instance(_ ++ _)

// 5. Max полугруппа
def maxSemigroup[A: Ordering]: Semigroup[A] = 
  Semigroup.instance((x, y) => if (Ordering[A].gt(x, y)) x else y)

// 6. Min полугруппа
def minSemigroup[A: Ordering]: Semigroup[A] = 
  Semigroup.instance((x, y) => if (Ordering[A].lt(x, y)) x else y)
```

#### Свободная полугруппа

**Определение 3.5 (Свободная полугруппа):**
Для множества A свободная полугруппа F(A) - это множество всех непустых конечных последовательностей элементов из A с операцией конкатенации.

```scala
// NonEmptyList - свободная полугруппа
import cats.data.NonEmptyList

implicit def nelSemigroup[A]: Semigroup[NonEmptyList[A]] = 
  Semigroup.instance(_ concatNel _)

val nel1 = NonEmptyList.of(1, 2, 3)
val nel2 = NonEmptyList.of(4, 5)

nel1 |+| nel2 // NonEmptyList(1, 2, 3, 4, 5)
```

### 3.3 Monoid в Cats

```scala
import cats.Monoid

trait Monoid[A] extends Semigroup[A] {
  def empty: A
  
  // Законы:
  // 1. Ассоциативность: combine(combine(x,y),z) == combine(x,combine(y,z))
  // 2. Левая единичность: combine(empty, x) == x
  // 3. Правая единичность: combine(x, empty) == x
}

object Monoid {
  def apply[A](implicit instance: Monoid[A]): Monoid[A] = instance
  
  def instance[A](emptyValue: A)(f: (A, A) => A): Monoid[A] = 
    new Monoid[A] {
      def empty: A = emptyValue
      def combine(x: A, y: A): A = f(x, y)
    }
}
```

**Примеры моноидов:**

```scala
// 1. (ℤ, +, 0)
implicit val intAdditionMonoid: Monoid[Int] = 
  Monoid.instance(0)(_ + _)

// 2. (ℤ, ×, 1)
implicit val intMultiplicationMonoid: Monoid[Int] = 
  Monoid.instance(1)(_ * _)

// 3. (String, ++, "")
implicit val stringMonoid: Monoid[String] = 
  Monoid.instance("")(_ ++ _)

// 4. (List[A], ++, Nil)
implicit def listMonoid[A]: Monoid[List[A]] = 
  Monoid.instance(List.empty[A])(_ ++ _)

// 5. (Boolean, ∨, false) - логическое ИЛИ
implicit val booleanOrMonoid: Monoid[Boolean] = 
  Monoid.instance(false)(_ || _)

// 6. (Boolean, ∧, true) - логическое И
implicit val booleanAndMonoid: Monoid[Boolean] = 
  Monoid.instance(true)(_ && _)

// 7. (Option[A], orElse, None) - First
implicit def optionFirstMonoid[A]: Monoid[Option[A]] = 
  Monoid.instance(None) { (x, y) => x.orElse(y) }

// 8. (Map[K, V], merge, empty)
implicit def mapMonoid[K, V: Semigroup]: Monoid[Map[K, V]] = 
  new Monoid[Map[K, V]] {
    def empty: Map[K, V] = Map.empty
    
    def combine(x: Map[K, V], y: Map[K, V]): Map[K, V] = {
      y.foldLeft(x) { case (acc, (k, v)) =>
        acc.updated(k, acc.get(k).fold(v)(Semigroup[V].combine(_, v)))
      }
    }
  }
```

### 3.4 Применения моноидов

#### combineAll

```scala
import cats.syntax.foldable._
import cats.instances.int._

val numbers = List(1, 2, 3, 4, 5)
numbers.combineAll // 15 (используя intAdditionMonoid)

// Для произвольного Monoid
def combineAll[A: Monoid](list: List[A]): A =
  list.foldLeft(Monoid[A].empty)(Monoid[A].combine)
```

#### foldMap

```scala
case class Order(items: List[Item], total: BigDecimal)

implicit val moneyMonoid: Monoid[BigDecimal] = 
  Monoid.instance(BigDecimal(0))(_ + _)

val orders = List(
  Order(List(), BigDecimal(100)),
  Order(List(), BigDecimal(250)),
  Order(List(), BigDecimal(75))
)

// foldMap - map + fold
orders.foldMap(_.total) // BigDecimal(425)

// Эквивалентно:
orders.map(_.total).combineAll
```

#### Параллельная редукция

**Теорема 3.1:** Благодаря ассоциативности моноида, редукцию можно распараллелить.

```scala
def parallelCombineAll[A: Monoid](list: List[A], parallelism: Int): A = {
  list
    .grouped((list.size / parallelism) + 1)
    .toList
    .par
    .map(_.combineAll)
    .foldLeft(Monoid[A].empty)(Monoid[A].combine)
}

// Работает корректно благодаря ассоциативности:
// ((a • b) • c) • d = a • (b • (c • d))
```

---

## 4. Монады - Математические Основы

### 4.1 Определение монады

**Определение 4.1 (Монада - категориальное определение):**
Монада на категории C - это тройка (T, η, μ), где:

1. **T: C → C** - эндофунктор
2. **η: Id ⇒ T** - естественное преобразование (unit/pure)
3. **μ: T ∘ T ⇒ T** - естественное преобразование (join/flatten)

**Удовлетворяющие аксиомам:**

**Аксиома 1 (Левая единичность):**
```
T ──η_T──→ T∘T
│          │
│id_T      │μ
↓          ↓
T ─────────→ T
   id_T

μ ∘ η_T = id_T
```

**Аксиома 2 (Правая единичность):**
```
T ──T∘η──→ T∘T
│          │
│id_T      │μ
↓          ↓
T ─────────→ T
   id_T

μ ∘ T∘η = id_T
```

**Аксиома 3 (Ассоциативность):**
```
T∘T∘T ──μ∘T──→ T∘T
  │              │
  │T∘μ           │μ
  ↓              ↓
 T∘T ───μ───→   T

μ ∘ (T ∘ μ) = μ ∘ μ_T
```

### 4.2 Монада в Scala

```scala
trait Monad[F[_]] extends Functor[F] {
  // η (unit/pure/return)
  def pure[A](a: A): F[A]
  
  // Монадическая композиция (flatMap/bind)
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  
  // μ (join/flatten) - производная от flatMap
  def flatten[A](ffa: F[F[A]]): F[A] = 
    flatMap(ffa)(identity)
  
  // map можно выразить через flatMap и pure
  override def map[A, B](fa: F[A])(f: A => B): F[B] =
    flatMap(fa)(a => pure(f(a)))
}
```

**Связь между определениями:**

```scala
// Можно определить flatMap через join и map
def flatMapViaJoin[F[_]: Functor, A, B](fa: F[A])(f: A => F[B])(join: F[F[B]] => F[B]): F[B] =
  join(Functor[F].map(fa)(f))

// Или join через flatMap
def joinViaFlatMap[F[_]: Monad, A](ffa: F[F[A]]): F[A] =
  Monad[F].flatMap(ffa)(identity)
```

### 4.3 Законы монады

**Закон 1 (Левая единичность / Left Identity):**
```scala
pure(a).flatMap(f) == f(a)

// Пример с Option
val a = 42
val f: Int => Option[String] = x => Some(x.toString)

Some(a).flatMap(f) == f(a)  // true
```

**Закон 2 (Правая единичность / Right Identity):**
```scala
m.flatMap(pure) == m

// Пример с Option
val m = Some(42)

m.flatMap(Some(_)) == m  // true
```

**Закон 3 (Ассоциативность / Associativity):**
```scala
m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g))

// Пример с Option
val m = Some(1)
val f: Int => Option[Int] = x => Some(x + 1)
val g: Int => Option[String] = x => Some(x.toString)

m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g))  // true
```

### 4.4 Клейсли-композиция

**Определение 4.2 (Клейсли-стрелка):**
Для монады M, Клейсли-стрелка - это функция вида:
```
A => M[B]
```

**Определение 4.3 (Клейсли-композиция):**
```scala
// Оператор >=> (композиция Клейсли)
def composeK[M[_]: Monad, A, B, C](
  f: A => M[B],
  g: B => M[C]
): A => M[C] = { a =>
  f(a).flatMap(g)
}

// Оператор <=< (обратная композиция)
def andThenK[M[_]: Monad, A, B, C](
  g: B => M[C],
  f: A => M[B]
): A => M[C] = composeK(f, g)
```

**Пример:**
```scala
val f: Int => Option[String] = x => Some(x.toString)
val g: String => Option[Boolean] = s => Some(s.nonEmpty)
val h: Boolean => Option[Int] = b => Some(if (b) 1 else 0)

// Композиция
val composed = composeK(composeK(f, g), h)

composed(42) // Some(1)

// Проверка ассоциативности
val leftAssoc = composeK(composeK(f, g), h)
val rightAssoc = composeK(f, composeK(g, h))

leftAssoc(42) == rightAssoc(42)  // true
```

### 4.5 Производные операции

```scala
trait Monad[F[_]] extends Functor[F] {
  def pure[A](a: A): F[A]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  
  // Производные операции:
  
  // >> (andThen) - последовательное выполнение, игнорируя первый результат
  def productR[A, B](fa: F[A])(fb: F[B]): F[B] = 
    flatMap(fa)(_ => fb)
  
  // *> - alias для productR
  def >>[A, B](fa: F[A])(fb: F[B]): F[B] = productR(fa)(fb)
  
  // <* - выполнить оба, вернуть первый
  def productL[A, B](fa: F[A])(fb: F[B]): F[A] = 
    flatMap(fa)(a => map(fb)(_ => a))
  
  // forever - бесконечный цикл
  def forever[A](fa: F[A]): F[Nothing] = 
    flatMap(fa)(_ => forever(fa))
  
  // iterateWhile - итерация с условием
  def iterateWhile[A](fa: F[A])(p: A => Boolean): F[A] = 
    flatMap(fa)(a => if (p(a)) iterateWhile(fa)(p) else pure(a))
  
  // iterateUntil
  def iterateUntil[A](fa: F[A])(p: A => Boolean): F[A] = 
    iterateWhile(fa)(a => !p(a))
  
  // ifM - монадический if
  def ifM[B](cond: F[Boolean])(ifTrue: F[B], ifFalse: F[B]): F[B] = 
    flatMap(cond)(if (_) ifTrue else ifFalse)
  
  // whileM - монадический while
  def whileM[A](cond: F[Boolean])(body: F[A]): F[Unit] = 
    ifM(cond)(flatMap(body)(_ => whileM(cond)(body)), pure(()))
  
  // untilM - монадический do-while
  def untilM[A](body: F[A])(cond: F[Boolean]): F[Unit] = 
    flatMap(body)(_ => ifM(cond)(pure(()), untilM(body)(cond)))
}
```

---

## 5. Cats Core Type Classes

### 5.1 Иерархия Type Classes

```
                    Invariant[F[_]]
                          │
         ┌────────────────┼────────────────┐
         │                │                │
    Contravariant[F[_]]  │          Functor[F[_]]
                          │                │
                   Bifunctor[F[_,_]]      │
                                      Apply[F[_]]
                                          │
                                   Applicative[F[_]]
                                          │
                                      FlatMap[F[_]]
                                          │
                                      Monad[F[_]]
                                          │
                                   MonadError[F[_], E]
```

### 5.2 Semigroup и Monoid детально

#### Теоремы о моноидах

**Теорема 5.1 (Единичность единичного элемента):**
В моноиде единичный элемент единственный.

**Доказательство:**
Пусть e и e' - два единичных элемента. Тогда:
```
e = e • e'  (так как e' единичный, e • e' = e)
  = e'      (так как e единичный, e • e' = e')
```
Следовательно, e = e'. ∎

**Теорема 5.2 (Гомоморфизм моноидов сохраняет единичный элемент):**
Если φ: M → N - гомоморфизм моноидов, то φ(e_M) = e_N.

```scala
trait MonoidHomomorphism[A, B] {
  def apply(a: A): B
  
  // Должно выполняться:
  // apply(empty_A) == empty_B
  // apply(combine(x, y)) == combine(apply(x), apply(y))
}

// Пример: length - гомоморфизм (List[A], ++, Nil) → (Int, +, 0)
val lengthHom: MonoidHomomorphism[List[_], Int] = new MonoidHomomorphism[List[_], Int] {
  def apply(list: List[_]): Int = list.length
}

// Проверка:
lengthHom(Nil) == 0  // apply(empty) == empty
lengthHom(List(1,2) ++ List(3,4)) == lengthHom(List(1,2)) + lengthHom(List(3,4))
```

### 5.3 Apply и Applicative

**Определение 5.1 (Apply):**
Apply - это Functor с операцией ap для применения функции в контексте.

```scala
trait Apply[F[_]] extends Functor[F] {
  // ap - apply wrapped function to wrapped value
  def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
  
  // Производные операции
  def map2[A, B, Z](fa: F[A], fb: F[B])(f: (A, B) => Z): F[Z] = 
    ap(map(fa)(a => (b: B) => f(a, b)))(fb)
  
  def product[A, B](fa: F[A], fb: F[B]): F[(A, B)] = 
    map2(fa, fb)((_, _))
  
  // Закон композиции для Apply
  // map2(map2(fa, fb)(f), fc)(g) == map2(fa, map2(fb, fc)(h))(i)
  // где соответствующие f, g, h, i удовлетворяют нужным условиям
}
```

**Определение 5.2 (Applicative):**
Applicative - это Apply с операцией pure.

```scala
trait Applicative[F[_]] extends Apply[F] {
  def pure[A](a: A): F[A]
  
  // Законы Applicative:
  // 1. Identity: ap(pure(identity))(fa) == fa
  // 2. Homomorphism: ap(pure(f))(pure(a)) == pure(f(a))
  // 3. Interchange: ap(ff)(pure(a)) == ap(pure(f => f(a)))(ff)
  // 4. Composition: ap(ap(ap(pure(compose))(f))(g))(fa) == ap(f)(ap(g)(fa))
}
```

**Закон Identity:**
```scala
val fa: F[A] = ...
ap(pure(identity[A]))(fa) == fa

// Пример с Option
val opt = Some(42)
Some((x: Int) => x).ap(opt) == opt  // true
```

**Закон Homomorphism:**
```scala
val f: A => B = ...
val a: A = ...
ap(pure(f))(pure(a)) == pure(f(a))

// Пример с Option
val f: Int => String = _.toString
val a = 42
Some(f).ap(Some(a)) == Some(f(a))  // true
```

**Закон Interchange:**
```scala
val ff: F[A => B] = ...
val a: A = ...
ap(ff)(pure(a)) == ap(pure((f: A => B) => f(a)))(ff)
```

**Закон Composition:**
```scala
val ff: F[B => C] = ...
val fg: F[A => B] = ...
val fa: F[A] = ...

val compose: (B => C) => (A => B) => (A => C) = 
  f => g => f compose g

ap(ap(ap(pure(compose))(ff))(fg))(fa) == ap(ff)(ap(fg)(fa))
```

#### Applicative vs Monad

**Ключевое отличие:**
- **Applicative**: структура вычислений фиксирована, зависимости не могут изменяться
- **Monad**: структура вычислений может зависеть от промежуточных результатов

```scala
// Applicative - структура фиксирована
val applicativeExample: Option[(Int, String, Boolean)] = (
  Some(42),
  Some("hello"),
  Some(true)
).mapN((i, s, b) => (i, s, b))

// Все три Option вычисляются независимо
// Структура известна заранее

// Monad - структура может меняться
val monadicExample: Option[Int] = for {
  x <- Some(42)
  y <- if (x > 0) Some(10) else None  // выбор зависит от x!
  z <- if (y > 5) Some(100) else None // выбор зависит от y!
} yield x + y + z

// Структура вычислений определяется динамически
```

### 5.4 Traverse

**Определение 5.3 (Traverse):**
Traverse обобщает операции map, fold и sequence.

```scala
trait Traverse[F[_]] extends Functor[F] with Foldable[F] {
  // Главная операция
  def traverse[G[_]: Applicative, A, B](fa: F[A])(f: A => G[B]): G[F[B]]
  
  // sequence - частный случай traverse с f = identity
  def sequence[G[_]: Applicative, A](fga: F[G[A]]): G[F[A]] = 
    traverse(fga)(identity)
  
  // Законы Traverse:
  // 1. Identity: traverse(fa)(Id(_)) == Id(fa)
  // 2. Composition: Compose(traverse(traverse(fa)(f))(g)) == traverse(fa)(Compose(f(_).map(g)))
  // 3. Naturality: t(traverse(fa)(f)) == traverse(fa)(t compose f)
  //    для любого естественного преобразования t
}
```

**Примеры траверсов:**

```scala
// List Traverse
implicit val listTraverse: Traverse[List] = new Traverse[List] {
  def traverse[G[_]: Applicative, A, B](fa: List[A])(f: A => G[B]): G[List[B]] =
    fa.foldRight(Applicative[G].pure(List.empty[B])) { (a, acc) =>
      Applicative[G].map2(f(a), acc)(_ :: _)
    }
  
  // Другие методы...
}

// Option Traverse
implicit val optionTraverse: Traverse[Option] = new Traverse[Option] {
  def traverse[G[_]: Applicative, A, B](fa: Option[A])(f: A => G[B]): G[Option[B]] =
    fa match {
      case Some(a) => Applicative[G].map(f(a))(Some(_))
      case None => Applicative[G].pure(None)
    }
}

// Either Traverse (traverse right side)
implicit def eitherTraverse[E]: Traverse[Either[E, *]] = 
  new Traverse[Either[E, *]] {
    def traverse[G[_]: Applicative, A, B](
      fa: Either[E, A]
    )(f: A => G[B]): G[Either[E, B]] = fa match {
      case Left(e) => Applicative[G].pure(Left(e))
      case Right(a) => Applicative[G].map(f(a))(Right(_))
    }
  }
```

**Теорема 5.3 (Traverse laws):**

**Закон Identity:**
```scala
// Id functor
case class Id[A](value: A)

implicit val idApplicative: Applicative[Id] = new Applicative[Id] {
  def pure[A](a: A): Id[A] = Id(a)
  def ap[A, B](ff: Id[A => B])(fa: Id[A]): Id[B] = Id(ff.value(fa.value))
}

// Закон: traverse(fa)(Id(_)) == Id(fa)
val list = List(1, 2, 3)
list.traverse(Id(_)) == Id(list)  // true
```

**Закон Composition:**
```scala
// Compose functor
case class Compose[F[_], G[_], A](value: F[G[A]])

implicit def composeApplicative[F[_]: Applicative, G[_]: Applicative]: Applicative[Compose[F, G, *]] =
  new Applicative[Compose[F, G, *]] {
    def pure[A](a: A): Compose[F, G, A] = 
      Compose(Applicative[F].pure(Applicative[G].pure(a)))
    
    def ap[A, B](ff: Compose[F, G, A => B])(fa: Compose[F, G, A]): Compose[F, G, B] =
      Compose(
        Applicative[F].ap(
          Applicative[F].map(ff.value)(gf => 
            Applicative[G].ap(gf)(_)
          )
        )(fa.value)
      )
  }

// Закон: 
// Compose(traverse(traverse(fa)(f))(g)) == traverse(fa)(a => Compose(f(a).map(g)))
```

---

## 6. Applicative и Apply

### 6.1 Applicative Functors - Теория

**Определение 6.1 (Applicative в Category Theory):**
В категориальной теории, Applicative Functor (также известный как Monoidal Functor) - это функтор F: C → D между моноидальными категориями, который:

1. Сохраняет моноидальную структуру
2. F(I_C) ≅ I_D (единичные объекты)
3. F(A ⊗_C B) ≅ F(A) ⊗_D F(B) (тензорное произведение)

#### Lax Monoidal Functor

**Определение 6.2:**
Lax monoidal functor - это функтор F с естественными преобразованиями:

```
ε: I_D → F(I_C)
μ: F(A) ⊗_D F(B) → F(A ⊗_C B)
```

**В Scala:**
```scala
// I_D = Unit (единичный объект)
// ⊗_D = (,) (tuple)

trait LaxMonoidalFunctor[F[_]] extends Functor[F] {
  // ε: Unit → F[Unit]
  def unit: F[Unit]
  
  // μ: (F[A], F[B]) → F[(A, B)]
  def product[A, B](fa: F[A], fb: F[B]): F[(A, B)]
}
```

**Связь с Applicative:**
```scala
// Applicative можно выразить через LaxMonoidalFunctor
trait Applicative[F[_]] extends LaxMonoidalFunctor[F] {
  def pure[A](a: A): F[A]
  def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
  
  // unit через pure
  def unit: F[Unit] = pure(())
  
  // product через ap и map
  def product[A, B](fa: F[A], fb: F[B]): F[(A, B)] =
    ap(map(fa)(a => (b: B) => (a, b)))(fb)
}

// И наоборот
def applicativeFromLax[F[_]: LaxMonoidalFunctor]: Applicative[F] = 
  new Applicative[F] {
    def pure[A](a: A): F[A] = 
      map(unit)(_ => a)
    
    def ap[A, B](ff: F[A => B])(fa: F[A]): F[B] = 
      map(product(ff, fa)) { case (f, a) => f(a) }
  }
```

### 6.2 Applicative Builders

```scala
// Applicative Builder Pattern
def apply2[F[_]: Applicative, A, B, Z](fa: F[A], fb: F[B])(f: (A, B) => Z): F[Z] =
  Applicative[F].map2(fa, fb)(f)

def apply3[F[_]: Applicative, A, B, C, Z](
  fa: F[A], fb: F[B], fc: F[C]
)(f: (A, B, C) => Z): F[Z] = {
  val F = Applicative[F]
  F.ap(F.ap(F.map(fa)(a => (b: B) => (c: C) => f(a, b, c)))(fb))(fc)
}

// И так далее до apply22 в Cats

// Использование
case class Person(name: String, age: Int, email: String)

val nameOpt: Option[String] = Some("Alice")
val ageOpt: Option[Int] = Some(30)
val emailOpt: Option[String] = Some("alice@example.com")

apply3(nameOpt, ageOpt, emailOpt)(Person.apply)
// Some(Person("Alice", 30, "alice@example.com"))

// Или с синтаксисом
import cats.syntax.apply._
(nameOpt, ageOpt, emailOpt).mapN(Person.apply)
```

### 6.3 Parallel Type Class

**Проблема:**
Многие монады (List, Either) имеют последовательное поведение при использовании Applicative операций.

**Решение:**
Type class `Parallel` предоставляет альтернативный Applicative с параллельным поведением.

```scala
trait Parallel[M[_]] {
  type F[_]  // Параллельный тип
  
  def parallel: M ~> F    // Natural transformation M => F
  def sequential: F ~> M  // Natural transformation F => M
  
  def applicative: Applicative[F]
  def monad: Monad[M]
}

// Пример для Either
implicit def eitherParallel[E: Semigroup]: Parallel[Either[E, *]] = 
  new Parallel[Either[E, *]] {
    type F[A] = Validated[E, A]
    
    val applicative: Applicative[Validated[E, *]] = 
      Validated.catsDataApplicativeErrorForValidated
    
    val monad: Monad[Either[E, *]] = 
      cats.instances.either.catsStdInstancesForEither
    
    val parallel: Either[E, *] ~> Validated[E, *] = 
      new (Either[E, *] ~> Validated[E, *]) {
        def apply[A](e: Either[E, A]): Validated[E, A] = 
          Validated.fromEither(e)
      }
    
    val sequential: Validated[E, *] ~> Either[E, *] = 
      new (Validated[E, *] ~> Either[E, *]) {
        def apply[A](v: Validated[E, A]): Either[E, A] = 
          v.toEither
      }
  }
```

**Использование:**
```scala
import cats.syntax.parallel._

// Sequential (short-circuits on first error)
val seq = (
  Either.left[String, Int]("Error 1"),
  Either.left[String, Int]("Error 2"),
  Either.right[String, Int](42)
).mapN((a, b, c) => a + b + c)
// Left("Error 1") - stops at first error

// Parallel (accumulates all errors)
val par = (
  Either.left[String, Int]("Error 1"),
  Either.left[String, Int]("Error 2"),
  Either.right[String, Int](42)
).parMapN((a, b, c) => a + b + c)
// Left("Error 1Error 2") - with Semigroup[String]
```

---

Продолжение следует в следующих частях с разделами 7-14...

**Часть II документа включает:**
- Monad Transformers с математическими обоснованиями
- Free Monads и теорию интерпретаторов  
- Tagless Final с теоретическими основами
- IO Monad implementation details
- Concurrency primitives theory
- Resource management mathematics
- FS2 streaming theory

Хотите, чтобы я продолжил с оставшимися разделами?
