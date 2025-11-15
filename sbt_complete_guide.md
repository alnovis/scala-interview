# Полный план подготовки по SBT (Scala Build Tool)

## Оглавление
1. [Введение в SBT](#введение-в-sbt)
2. [Установка и настройка](#установка-и-настройка)
3. [Структура проекта](#структура-проекта)
4. [Базовые концепции](#базовые-концепции)
5. [Конфигурация проекта](#конфигурация-проекта)
6. [Управление зависимостями](#управление-зависимостями)
7. [Задачи и команды](#задачи-и-команды)
8. [Плагины](#плагины)
9. [Мультипроектная структура](#мультипроектная-структура)
10. [Тестирование](#тестирование)
11. [Публикация артефактов](#публикация-артефактов)
12. [Продвинутые темы](#продвинутые-темы)
13. [Оптимизация и best practices](#оптимизация-и-best-practices)
14. [Практические проекты](#практические-проекты)

---

## Введение в SBT

### Что такое SBT?

SBT (Simple Build Tool) — это инструмент сборки для проектов на Scala и Java. Это де-факто стандарт для Scala проектов.

**Основные возможности:**
- Компиляция кода
- Управление зависимостями
- Запуск тестов
- Упаковка артефактов
- Публикация библиотек
- Инкрементальная компиляция
- Параллельное выполнение задач
- REPL интеграция

**Преимущества SBT:**
- Написан на Scala (конфигурация на Scala)
- Мощная система плагинов
- Инкрементальная компиляция (компилирует только измененные файлы)
- Интерактивный режим
- Поддержка мультипроектов
- Интеграция с IDE (IntelliJ IDEA, VS Code)

**Сравнение с другими инструментами:**
```
Maven:
+ Более простая конфигурация (XML)
- Менее гибкий
- Медленнее компиляция

Gradle:
+ Гибкий (Groovy/Kotlin DSL)
- Меньше оптимизаций для Scala
- Меньше специфичных для Scala плагинов

SBT:
+ Оптимизирован для Scala
+ Инкрементальная компиляция
+ Мощные возможности настройки
- Крутая кривая обучения
- Специфичный синтаксис
```

### Архитектура SBT

```
SBT Архитектура:
┌─────────────────────────────────────┐
│      build.sbt (конфигурация)       │
├─────────────────────────────────────┤
│           Settings & Tasks          │
├─────────────────────────────────────┤
│        Zinc (инкр. компилятор)      │
├─────────────────────────────────────┤
│      Coursier (resolving deps)      │
├─────────────────────────────────────┤
│         Scala Compiler API          │
└─────────────────────────────────────┘
```

---

## Установка и настройка

### Установка SBT

**macOS (через Homebrew):**
```bash
brew install sbt
```

**Linux (Ubuntu/Debian):**
```bash
echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add
sudo apt-get update
sudo apt-get install sbt
```

**Windows (через Scoop):**
```bash
scoop install sbt
```

**Проверка установки:**
```bash
sbt version
# sbt version in this project: 1.9.7
# sbt script version: 1.9.7
```

### Глобальная конфигурация

**~/.sbt/1.0/global.sbt** — глобальные настройки для всех проектов:
```scala
// Установка proxy
Global / useCoursier := true

// JVM опции
Global / javaOptions ++= Seq(
  "-Xms512M",
  "-Xmx2G",
  "-Xss2M"
)

// Настройка логирования
Global / logLevel := Level.Info

// Добавление глобальных плагинов
// ~/.sbt/1.0/plugins/plugins.sbt
```

**~/.sbt/repositories** — настройка репозиториев:
```
[repositories]
  local
  maven-central
  typesafe-ivy-releases: https://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/[revision]/[type]s/[artifact](-[classifier]).[ext]
  sonatype-snapshots: https://oss.sonatype.org/content/repositories/snapshots
  my-maven-proxy: https://my.maven.proxy/
```

### JVM настройки

**.jvmopts** в корне проекта:
```
-Xms512M
-Xmx4G
-Xss2M
-XX:+UseG1GC
-XX:ReservedCodeCacheSize=256M
```

**.sbtopts** в корне проекта:
```
-J-Xms512M
-J-Xmx4G
-J-Xss2M
-Dsbt.color=always
-Dsbt.coursier=true
```

---

## Структура проекта

### Минимальная структура

```
my-project/
├── build.sbt              # Основная конфигурация
├── project/
│   ├── build.properties   # Версия SBT
│   └── plugins.sbt        # Плагины
└── src/
    ├── main/
    │   ├── scala/         # Исходный код Scala
    │   ├── java/          # Исходный код Java
    │   └── resources/     # Ресурсы
    └── test/
        ├── scala/         # Тесты
        └── resources/     # Ресурсы для тестов
```

### Полная структура

```
my-project/
├── build.sbt                      # Основная конфигурация
├── project/
│   ├── build.properties           # Версия SBT
│   ├── plugins.sbt                # Плагины
│   ├── Dependencies.scala         # Управление зависимостями
│   └── Settings.scala             # Общие настройки
├── src/
│   ├── main/
│   │   ├── scala/
│   │   │   └── com/example/
│   │   │       └── Main.scala
│   │   ├── java/
│   │   └── resources/
│   │       ├── application.conf
│   │       └── logback.xml
│   ├── test/
│   │   ├── scala/
│   │   │   └── com/example/
│   │   │       └── MainSpec.scala
│   │   └── resources/
│   │       └── application-test.conf
│   └── it/                        # Интеграционные тесты
│       └── scala/
├── target/                        # Скомпилированные файлы
├── lib/                           # Неуправляемые зависимости
├── docs/                          # Документация
├── scripts/                       # Скрипты
├── .gitignore
├── README.md
└── LICENSE
```

### project/build.properties

```properties
# Указывает версию SBT для проекта
sbt.version=1.9.7
```

### Создание нового проекта

**Вручную:**
```bash
mkdir my-project
cd my-project
mkdir -p src/{main,test}/{scala,resources}
mkdir project
echo 'sbt.version=1.9.7' > project/build.properties
touch build.sbt
```

**Используя sbt new:**
```bash
# Создание проекта из шаблона
sbt new scala/scala-seed.g8

# Другие популярные шаблоны
sbt new scala/scala3.g8              # Scala 3 проект
sbt new playframework/play-scala-seed.g8  # Play Framework
sbt new http4s/http4s.g8             # http4s
sbt new holdenk/sparkProjectTemplate.g8   # Apache Spark
```

---

## Базовые концепции

### Settings, Tasks и Commands

**Settings** — неизменяемые значения, вычисляемые один раз при загрузке проекта:
```scala
// В build.sbt
name := "my-project"
version := "1.0.0"
scalaVersion := "2.13.12"
```

**Tasks** — вычисления, которые выполняются каждый раз при вызове:
```scala
// Предопределенные tasks
compile  // Компиляция кода
test     // Запуск тестов
run      // Запуск приложения
clean    // Очистка target

// Определение своей task
val hello = taskKey[Unit]("Prints hello")
hello := {
  println("Hello from SBT!")
}
```

**Commands** — более низкоуровневые операции:
```scala
commands += Command.command("greet") { state =>
  println("Hello!")
  state
}
```

### Scopes

Scopes определяют контекст для settings и tasks.

**Три оси scope:**
1. **Projects** — подпроект (для мультипроектов)
2. **Configurations** — конфигурация (Compile, Test, Runtime)
3. **Tasks** — привязка к task

```scala
// Примеры scopes
name                              // Глобальный scope
Compile / name                    // Scope конфигурации Compile
Test / name                       // Scope конфигурации Test
Compile / compile / scalacOptions // Опции компилятора для compile task
```

### Зависимости между tasks

```scala
// task B зависит от task A
val taskA = taskKey[String]("Task A")
val taskB = taskKey[String]("Task B")

taskA := {
  println("Running Task A")
  "Result A"
}

taskB := {
  val resultA = taskA.value  // Сначала выполнится taskA
  println(s"Running Task B with: $resultA")
  s"Result B depends on $resultA"
}
```

### Операторы

```scala
// := - присвоение
name := "my-project"

// += - добавление одного элемента
libraryDependencies += "org.typelevel" %% "cats-core" % "2.10.0"

// ++= - добавление нескольких элементов
libraryDependencies ++= Seq(
  "org.typelevel" %% "cats-core" % "2.10.0",
  "org.typelevel" %% "cats-effect" % "3.5.2"
)

// ~= - трансформация значения
version := version.value + "-SNAPSHOT"
scalacOptions ~= (_ filterNot (_ == "-Xfatal-warnings"))

// in - указание scope (устаревший синтаксис)
scalacOptions in Compile += "-deprecation"
// Новый синтаксис:
Compile / scalacOptions += "-deprecation"
```

---

## Конфигурация проекта

### Базовая конфигурация build.sbt

```scala
// Информация о проекте
ThisBuild / organization := "com.example"
ThisBuild / version      := "1.0.0"
ThisBuild / scalaVersion := "2.13.12"

lazy val root = (project in file("."))
  .settings(
    name := "my-project",
    
    // Основные настройки
    scalaVersion := "2.13.12",
    
    // Опции компилятора Scala
    scalacOptions ++= Seq(
      "-encoding", "UTF-8",
      "-deprecation",
      "-feature",
      "-unchecked",
      "-Xlint",
      "-Ywarn-dead-code",
      "-Ywarn-numeric-widen",
      "-Ywarn-value-discard"
    ),
    
    // Опции компилятора Java
    javacOptions ++= Seq(
      "-source", "11",
      "-target", "11",
      "-Xlint:unchecked",
      "-Xlint:deprecation"
    ),
    
    // JVM опции для запуска
    javaOptions ++= Seq(
      "-Xms512M",
      "-Xmx2G"
    ),
    
    // Fork отдельный JVM процесс для run и test
    Compile / run / fork := true,
    Test / fork := true,
    
    // Параллельное выполнение тестов
    Test / parallelExecution := true,
    
    // Настройки для scaladoc
    Compile / doc / scalacOptions ++= Seq(
      "-groups",
      "-implicits"
    )
  )
```

### Scala версии и кросс-компиляция

```scala
// Одна версия Scala
scalaVersion := "2.13.12"

// Кросс-компиляция для нескольких версий
crossScalaVersions := Seq("2.12.18", "2.13.12", "3.3.1")

// Условные настройки в зависимости от версии
libraryDependencies ++= {
  CrossVersion.partialVersion(scalaVersion.value) match {
    case Some((2, n)) if n <= 12 =>
      Seq("org.scala-lang.modules" %% "scala-collection-compat" % "2.11.0")
    case _ =>
      Seq.empty
  }
}

// Команда для сборки под все версии
// sbt +compile
// sbt +test
// sbt +publishLocal
```

### Resolvers (репозитории)

```scala
// Добавление дополнительных репозиториев
resolvers ++= Seq(
  "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots",
  "Sonatype OSS Releases" at "https://oss.sonatype.org/content/repositories/releases",
  "Typesafe Releases" at "https://repo.typesafe.com/typesafe/releases/",
  Resolver.mavenLocal,
  Resolver.mavenCentral
)

// Частный репозиторий с аутентификацией
credentials += Credentials(
  "Sonatype Nexus Repository Manager",
  "my.artifact.repo.net",
  "admin",
  "password"
)

// Или из файла
credentials += Credentials(Path.userHome / ".sbt" / ".credentials")
```

### Конфигурации (Compile, Test, Runtime)

```scala
// Compile - основной код
Compile / sourceDirectory := baseDirectory.value / "src" / "main"

// Test - тесты
Test / sourceDirectory := baseDirectory.value / "src" / "test"

// Создание своей конфигурации
lazy val IntegrationTest = config("it") extend Test
configs(IntegrationTest)
inConfig(IntegrationTest)(Defaults.testSettings)

// Использование
IntegrationTest / sourceDirectory := baseDirectory.value / "src" / "it"
```

---

## Управление зависимостями

### Синтаксис зависимостей

```scala
libraryDependencies += groupID % artifactID % revision
libraryDependencies += groupID %% artifactID % revision  // автоматически добавляет Scala версию
libraryDependencies += groupID %%% artifactID % revision // для Scala.js

// Примеры
libraryDependencies += "org.typelevel" %% "cats-core" % "2.10.0"
// Эквивалентно: "org.typelevel" % "cats-core_2.13" % "2.10.0"

// Зависимость только для тестов
libraryDependencies += "org.scalatest" %% "scalatest" % "3.2.17" % Test

// Зависимость для нескольких конфигураций
libraryDependencies += "ch.qos.logback" % "logback-classic" % "1.4.11" % "compile->default,test"
```

### Управление версиями зависимостей

**project/Dependencies.scala:**
```scala
import sbt._

object Dependencies {
  // Версии
  object Versions {
    val cats       = "2.10.0"
    val catsEffect = "3.5.2"
    val http4s     = "0.23.23"
    val circe      = "0.14.6"
    val zio        = "2.0.19"
    val scalaTest  = "3.2.17"
  }

  // Библиотеки
  val cats = Seq(
    "org.typelevel" %% "cats-core"   % Versions.cats,
    "org.typelevel" %% "cats-effect" % Versions.catsEffect
  )

  val http4s = Seq(
    "org.http4s" %% "http4s-dsl"          % Versions.http4s,
    "org.http4s" %% "http4s-ember-server" % Versions.http4s,
    "org.http4s" %% "http4s-ember-client" % Versions.http4s,
    "org.http4s" %% "http4s-circe"        % Versions.http4s
  )

  val circe = Seq(
    "io.circe" %% "circe-core"    % Versions.circe,
    "io.circe" %% "circe-generic" % Versions.circe,
    "io.circe" %% "circe-parser"  % Versions.circe
  )

  val zio = Seq(
    "dev.zio" %% "zio"         % Versions.zio,
    "dev.zio" %% "zio-streams" % Versions.zio
  )

  val testing = Seq(
    "org.scalatest" %% "scalatest" % Versions.scalaTest % Test
  )

  // Все зависимости
  val all = cats ++ http4s ++ circe ++ zio ++ testing
}
```

**Использование в build.sbt:**
```scala
import Dependencies._

lazy val root = (project in file("."))
  .settings(
    libraryDependencies ++= Dependencies.all
  )
```

### Исключение транзитивных зависимостей

```scala
// Исключение конкретной зависимости
libraryDependencies += "org.apache.spark" %% "spark-core" % "3.5.0" exclude("org.slf4j", "slf4j-log4j12")

// Исключение по группе
libraryDependencies += "org.apache.spark" %% "spark-core" % "3.5.0" excludeAll(
  ExclusionRule(organization = "org.slf4j")
)

// Исключение на уровне проекта
excludeDependencies ++= Seq(
  ExclusionRule("org.slf4j", "slf4j-log4j12")
)
```

### Разрешение конфликтов версий

```scala
// Принудительная версия
dependencyOverrides += "com.fasterxml.jackson.core" % "jackson-databind" % "2.15.2"

// Стратегия разрешения конфликтов
conflictManager := ConflictManager.latestRevision

// Проверка дерева зависимостей
// sbt dependencyTree
// sbt "whatDependsOn org.scala-lang scala-library 2.13.12"
```

### Неуправляемые зависимости

```scala
// JAR файлы в директории lib/
Compile / unmanagedJars += baseDirectory.value / "lib" / "custom.jar"

// Все JAR из директории
Compile / unmanagedJars ++= {
  val base = baseDirectory.value
  val libs = base / "lib"
  (libs ** "*.jar").classpath
}
```

---

## Задачи и команды

### Основные встроенные задачи

```scala
// Компиляция
compile          // Компилировать main источники
Test / compile   // Компилировать test источники

// Запуск
run              // Запустить main класс
Test / run       // Запустить test main класс
runMain com.example.Main  // Запустить конкретный класс

// Тестирование
test             // Запустить все тесты
testOnly com.example.MySpec  // Запустить конкретный тест
testQuick        // Запустить только изменённые тесты

// Консоль
console          // Scala REPL с classpath проекта
Test / console   // REPL с test classpath
consoleQuick     // REPL без компиляции

// Очистка
clean            // Удалить target
cleanFiles       // Список файлов для очистки

// Упаковка
package          // Создать JAR
publishLocal     // Опубликовать в локальный Ivy
publish          // Опубликовать в удалённый репозиторий

// Документация
doc              // Генерировать ScalaDoc

// Информация
projects         // Список проектов
settings         // Список всех settings
tasks            // Список всех tasks
inspect <key>    // Подробности о setting/task
```

### Создание своих задач

```scala
// Простая задача
val hello = taskKey[Unit]("Выводит приветствие")

hello := {
  println("Привет из SBT!")
}

// Задача с зависимостями
val sayHello = taskKey[String]("Возвращает приветствие")
val sayGoodbye = taskKey[Unit]("Прощается")

sayHello := {
  val n = name.value  // Зависимость от setting
  s"Hello, $n!"
}

sayGoodbye := {
  val greeting = sayHello.value  // Зависимость от task
  println(greeting)
  println("Goodbye!")
}

// Задача с параметрами через inputTask
val greet = inputKey[Unit]("Приветствует по имени")

import complete.DefaultParsers._

greet := {
  val args = spaceDelimited("<arg>").parsed
  val names = if (args.isEmpty) Seq("World") else args
  names.foreach(name => println(s"Hello, $name!"))
}
```

### Продвинутые задачи

```scala
// Динамическая задача
val complexTask = taskKey[String]("Сложная задача")

complexTask := Def.taskDyn {
  val sv = scalaVersion.value
  if (sv.startsWith("2.13"))
    Def.task { "Scala 2.13" }
  else if (sv.startsWith("3"))
    Def.task { "Scala 3" }
  else
    Def.task { "Other" }
}.value

// Задача с побочными эффектами
val generateConfig = taskKey[File]("Генерирует конфигурационный файл")

generateConfig := {
  val file = (Compile / resourceManaged).value / "generated.conf"
  val contents = s"""
    |version = "${version.value}"
    |scalaVersion = "${scalaVersion.value}"
    |""".stripMargin
  IO.write(file, contents)
  file
}

// Добавление файла в ресурсы
Compile / resourceGenerators += generateConfig.taskValue
```

### Команды

```scala
// Простая команда
commands += Command.command("greet") { state =>
  println("Hello from command!")
  state  // Возврат state обязателен
}

// Команда с аргументами
commands += Command.args("echo", "<args>") { (state, args) =>
  println(args.mkString(" "))
  state
}

// Команда с парсингом
import complete.DefaultParsers._

commands += Command("deploy")(_ => spaceDelimited("<env>")) { (state, args) =>
  val env = args.headOption.getOrElse("dev")
  println(s"Deploying to $env")
  // Выполнение других команд
  s"project root" :: s"test" :: s"publish" :: state
}
```

### Последовательное выполнение команд

```bash
# Точка с запятой - последовательно
sbt "clean; compile; test"

# && - выполнить следующую, если предыдущая успешна
sbt clean && sbt test

# Через alias
addCommandAlias("validate", ";clean;compile;test")
# Использование: sbt validate
```

---

## Плагины

### Добавление плагинов

**project/plugins.sbt:**
```scala
// Assembly - создание fat JAR
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "2.1.5")

// Native Packager - создание native пакетов
addSbtPlugin("com.github.sbt" % "sbt-native-packager" % "1.9.16")

// ScalaFmt - форматирование кода
addSbtPlugin("org.scalameta" % "sbt-scalafmt" % "2.5.2")

// Scoverage - покрытие кода
addSbtPlugin("org.scoverage" % "sbt-scoverage" % "2.0.9")

// WartRemover - статический анализ
addSbtPlugin("org.wartremover" % "sbt-wartremover" % "3.1.5")

// Scalafix - рефакторинг и linting
addSbtPlugin("ch.epfl.scala" % "sbt-scalafix" % "0.11.1")

// BuildInfo - генерация build информации
addSbtPlugin("com.eed3si9n" % "sbt-buildinfo" % "0.11.0")

// Revolver - hot reload для разработки
addSbtPlugin("io.spray" % "sbt-revolver" % "0.10.0")
```

### SBT Assembly

Создание "fat JAR" со всеми зависимостями.

**Настройка:**
```scala
// В build.sbt
import sbtassembly.AssemblyPlugin.autoImport._

lazy val root = (project in file("."))
  .settings(
    assembly / assemblyJarName := s"${name.value}-${version.value}.jar",
    
    // Стратегия объединения при конфликтах
    assembly / assemblyMergeStrategy := {
      case PathList("META-INF", xs @ _*) => MergeStrategy.discard
      case "reference.conf" => MergeStrategy.concat
      case x => MergeStrategy.first
    },
    
    // Исключение зависимостей из JAR
    assembly / assemblyExcludedJars := {
      val cp = (assembly / fullClasspath).value
      cp.filter(_.data.getName == "scala-library.jar")
    },
    
    // Main class
    assembly / mainClass := Some("com.example.Main")
  )
```

**Использование:**
```bash
sbt assembly
# Создаст target/scala-2.13/my-project-1.0.0.jar

java -jar target/scala-2.13/my-project-1.0.0.jar
```

### SBT Native Packager

Создание пакетов для различных платформ.

```scala
enablePlugins(JavaAppPackaging)

// Для Docker
enablePlugins(DockerPlugin)

dockerBaseImage := "openjdk:11-jre-slim"
dockerExposedPorts := Seq(8080)
dockerRepository := Some("myregistry.com")

Docker / packageName := "my-app"
Docker / version := "1.0.0"

// Команды Docker
// sbt docker:publishLocal
// sbt docker:publish

// Для Debian/Ubuntu
enablePlugins(DebianPlugin)

maintainer := "Your Name <your.email@example.com>"
packageSummary := "My application"
packageDescription := "Detailed description of my application"

// sbt debian:packageBin

// Для RPM (RedHat/CentOS)
enablePlugins(RpmPlugin)

// sbt rpm:packageBin

// Universal ZIP
enablePlugins(UniversalPlugin)

// sbt universal:packageBin
```

### ScalaFmt

Автоматическое форматирование кода.

**.scalafmt.conf:**
```conf
version = "3.7.17"
runner.dialect = scala213

maxColumn = 120
align.preset = more
align.multiline = false

rewrite.rules = [
  RedundantBraces,
  RedundantParens,
  SortModifiers,
  PreferCurlyFors
]

newlines.source = keep
```

**Использование:**
```bash
sbt scalafmt           # Форматировать main код
sbt Test/scalafmt      # Форматировать test код
sbt scalafmtCheck      # Проверить форматирование
sbt scalafmtAll        # Форматировать весь код
```

### Scoverage

Измерение покрытия кода тестами.

```scala
// В project/plugins.sbt уже добавлен плагин

// Настройки в build.sbt
coverageEnabled := true
coverageMinimumStmtTotal := 80
coverageFailOnMinimum := true

coverageExcludedPackages := "<empty>;.*\\.generated\\..*"
```

**Использование:**
```bash
sbt clean coverage test coverageReport

# Отчёт в target/scala-2.13/scoverage-report/index.html
```

### BuildInfo

Генерация информации о сборке.

```scala
enablePlugins(BuildInfoPlugin)

buildInfoKeys := Seq[BuildInfoKey](
  name,
  version,
  scalaVersion,
  sbtVersion,
  BuildInfoKey.action("buildTime") {
    System.currentTimeMillis()
  }
)

buildInfoPackage := "com.example.info"
buildInfoOptions += BuildInfoOption.ToJson
```

**Использование в коде:**
```scala
package com.example

object Main extends App {
  println(s"Name: ${info.BuildInfo.name}")
  println(s"Version: ${info.BuildInfo.version}")
  println(s"Scala: ${info.BuildInfo.scalaVersion}")
  println(s"Build time: ${info.BuildInfo.buildTime}")
}
```

### Создание своего плагина

**project/CustomPlugin.scala:**
```scala
import sbt._
import sbt.Keys._

object CustomPlugin extends AutoPlugin {
  override def trigger = allRequirements

  object autoImport {
    val customTask = taskKey[Unit]("Custom task")
    val customSetting = settingKey[String]("Custom setting")
  }

  import autoImport._

  override lazy val projectSettings = Seq(
    customSetting := "default value",
    customTask := {
      val log = streams.value.log
      val setting = customSetting.value
      log.info(s"Custom task with setting: $setting")
    }
  )
}
```

---

## Мультипроектная структура

### Базовая структура

```
my-project/
├── build.sbt
├── project/
├── core/
│   └── src/
│       └── main/scala/
├── api/
│   └── src/
│       └── main/scala/
└── app/
    └── src/
        └── main/scala/
```

### Определение подпроектов

```scala
// build.sbt

lazy val commonSettings = Seq(
  organization := "com.example",
  version := "1.0.0",
  scalaVersion := "2.13.12",
  scalacOptions ++= Seq("-deprecation", "-feature")
)

// Core модуль - базовая функциональность
lazy val core = (project in file("core"))
  .settings(
    commonSettings,
    name := "my-project-core",
    libraryDependencies ++= Seq(
      "org.typelevel" %% "cats-core" % "2.10.0"
    )
  )

// API модуль - зависит от core
lazy val api = (project in file("api"))
  .settings(
    commonSettings,
    name := "my-project-api",
    libraryDependencies ++= Seq(
      "org.http4s" %% "http4s-dsl" % "0.23.23"
    )
  )
  .dependsOn(core)

// App модуль - главное приложение
lazy val app = (project in file("app"))
  .settings(
    commonSettings,
    name := "my-project-app",
    mainClass := Some("com.example.Main"),
    libraryDependencies ++= Seq(
      "ch.qos.logback" % "logback-classic" % "1.4.11"
    )
  )
  .dependsOn(api, core)

// Root проект - агрегирует все подпроекты
lazy val root = (project in file("."))
  .settings(
    name := "my-project",
    publish / skip := true  // Не публиковать root
  )
  .aggregate(core, api, app)
```

### Зависимости между проектами

```scala
// Зависимость от compile классов
lazy val api = project
  .dependsOn(core)

// Зависимость от test классов
lazy val integrationTests = project
  .dependsOn(core % "test->test")

// Множественные зависимости
lazy val app = project
  .dependsOn(
    core % "compile->compile;test->test",
    api
  )

// Зависимость от другого проекта с другой версией
lazy val legacy = project
  .dependsOn(core % "provided")
```

### Агрегация

```scala
// Агрегация - команды применяются ко всем подпроектам
lazy val root = (project in file("."))
  .aggregate(core, api, app)

// При выполнении:
// sbt compile  - скомпилирует все три проекта
// sbt test     - запустит тесты во всех проектах
```

### Навигация между проектами

```bash
# В SBT shell
sbt
> projects                 # Список всех проектов
> project core            # Переключиться на core
> project root            # Переключиться на root
> core/compile            # Скомпилировать core не переключаясь
> api/test                # Запустить тесты в api
```

### Общие настройки

**project/CommonSettings.scala:**
```scala
import sbt._
import sbt.Keys._

object CommonSettings {
  val settings = Seq(
    organization := "com.example",
    scalaVersion := "2.13.12",
    
    scalacOptions ++= Seq(
      "-encoding", "UTF-8",
      "-deprecation",
      "-feature",
      "-unchecked"
    ),
    
    Test / parallelExecution := false,
    
    resolvers ++= Seq(
      Resolver.mavenLocal,
      Resolver.sonatypeRepo("releases")
    )
  )

  val testSettings = Seq(
    libraryDependencies ++= Seq(
      "org.scalatest" %% "scalatest" % "3.2.17" % Test,
      "org.scalatestplus" %% "mockito-4-11" % "3.2.17.0" % Test
    )
  )
}
```

**Использование:**
```scala
import CommonSettings._

lazy val core = project
  .settings(settings)
  .settings(testSettings)

lazy val api = project
  .settings(settings)
  .settings(testSettings)
  .dependsOn(core)
```

### Пример реального проекта

```scala
lazy val core = (project in file("modules/core"))
  .settings(commonSettings)
  .settings(
    name := "myapp-core",
    libraryDependencies ++= Dependencies.core
  )

lazy val database = (project in file("modules/database"))
  .settings(commonSettings)
  .settings(
    name := "myapp-database",
    libraryDependencies ++= Dependencies.database
  )
  .dependsOn(core)

lazy val api = (project in file("modules/api"))
  .settings(commonSettings)
  .settings(
    name := "myapp-api",
    libraryDependencies ++= Dependencies.http4s
  )
  .dependsOn(core, database)

lazy val worker = (project in file("modules/worker"))
  .settings(commonSettings)
  .settings(
    name := "myapp-worker",
    libraryDependencies ++= Dependencies.kafka
  )
  .dependsOn(core, database)

lazy val app = (project in file("modules/app"))
  .enablePlugins(JavaAppPackaging, DockerPlugin)
  .settings(commonSettings)
  .settings(
    name := "myapp",
    mainClass := Some("com.example.Main"),
    dockerExposedPorts := Seq(8080),
    dockerBaseImage := "openjdk:11-jre-slim"
  )
  .dependsOn(api, worker)

lazy val root = (project in file("."))
  .settings(
    name := "myapp-root",
    publish / skip := true
  )
  .aggregate(core, database, api, worker, app)
```

---

## Тестирование

### ScalaTest

```scala
// build.sbt
libraryDependencies += "org.scalatest" %% "scalatest" % "3.2.17" % Test

// Настройки тестирования
Test / testOptions += Tests.Argument(TestFrameworks.ScalaTest, "-oD")
Test / parallelExecution := true
Test / fork := true
```

**Примеры тестов:**
```scala
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers

class CalculatorSpec extends AnyFlatSpec with Matchers {
  "A Calculator" should "add two numbers" in {
    val result = Calculator.add(2, 3)
    result shouldBe 5
  }

  it should "subtract two numbers" in {
    Calculator.subtract(5, 3) shouldBe 2
  }

  it should "multiply two numbers" in {
    Calculator.multiply(4, 3) shouldBe 12
  }
}
```

### Specs2

```scala
libraryDependencies += "org.specs2" %% "specs2-core" % "4.20.3" % Test
```

```scala
import org.specs2.mutable.Specification

class CalculatorSpec extends Specification {
  "Calculator" should {
    "add two numbers" in {
      Calculator.add(2, 3) must_=== 5
    }

    "subtract two numbers" in {
      Calculator.subtract(5, 3) must_=== 2
    }
  }
}
```

### MUnit

```scala
libraryDependencies += "org.scalameta" %% "munit" % "0.7.29" % Test

testFrameworks += new TestFramework("munit.Framework")
```

```scala
class CalculatorSuite extends munit.FunSuite {
  test("add two numbers") {
    assertEquals(Calculator.add(2, 3), 5)
  }

  test("subtract two numbers") {
    assertEquals(Calculator.subtract(5, 3), 2)
  }
}
```

### ZIO Test

```scala
libraryDependencies ++= Seq(
  "dev.zio" %% "zio-test" % "2.0.19" % Test,
  "dev.zio" %% "zio-test-sbt" % "2.0.19" % Test
)

testFrameworks += new TestFramework("zio.test.sbt.ZTestFramework")
```

```scala
import zio.test._

object CalculatorSpec extends ZIOSpecDefault {
  def spec = suite("CalculatorSpec")(
    test("add two numbers") {
      assertTrue(Calculator.add(2, 3) == 5)
    },
    test("subtract two numbers") {
      assertTrue(Calculator.subtract(5, 3) == 2)
    }
  )
}
```

### Интеграционные тесты

**Создание конфигурации для интеграционных тестов:**
```scala
lazy val IntegrationTest = config("it") extend Test

lazy val root = project
  .configs(IntegrationTest)
  .settings(
    inConfig(IntegrationTest)(Defaults.testSettings),
    IntegrationTest / scalaSource := baseDirectory.value / "src" / "it" / "scala"
  )
```

**Структура:**
```
src/
├── main/scala/
├── test/scala/          # Юнит-тесты
└── it/scala/            # Интеграционные тесты
```

**Запуск:**
```bash
sbt it:test
```

### Тестовые фикстуры

```scala
// ScalaTest
class DatabaseSpec extends AnyFlatSpec with BeforeAndAfterAll with Matchers {
  var db: Database = _

  override def beforeAll(): Unit = {
    db = Database.connect("jdbc:h2:mem:test")
    db.migrate()
  }

  override def afterAll(): Unit = {
    db.close()
  }

  "Database" should "save user" in {
    val user = User("Alice", "alice@example.com")
    db.save(user)
    db.find(user.id) shouldBe Some(user)
  }
}
```

### Property-based тестирование

```scala
libraryDependencies += "org.scalacheck" %% "scalacheck" % "1.17.0" % Test
```

```scala
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers
import org.scalatestplus.scalacheck.ScalaCheckPropertyChecks

class StringSpec extends AnyFlatSpec with Matchers with ScalaCheckPropertyChecks {
  "String concatenation" should "be associative" in {
    forAll { (s1: String, s2: String, s3: String) =>
      ((s1 + s2) + s3) shouldBe (s1 + (s2 + s3))
    }
  }

  "String reverse" should "be involutive" in {
    forAll { (s: String) =>
      s.reverse.reverse shouldBe s
    }
  }
}
```

---

## Публикация артефактов

### Настройка публикации

```scala
// Базовая информация
ThisBuild / organization := "com.example"
ThisBuild / organizationName := "Example Company"
ThisBuild / organizationHomepage := Some(url("https://example.com"))

ThisBuild / scmInfo := Some(
  ScmInfo(
    url("https://github.com/example/myproject"),
    "scm:git@github.com:example/myproject.git"
  )
)

ThisBuild / developers := List(
  Developer(
    id    = "yourname",
    name  = "Your Name",
    email = "your.email@example.com",
    url   = url("https://yourwebsite.com")
  )
)

ThisBuild / description := "My awesome Scala library"
ThisBuild / licenses := List("Apache 2" -> url("http://www.apache.org/licenses/LICENSE-2.0.txt"))
ThisBuild / homepage := Some(url("https://github.com/example/myproject"))
```

### Публикация в Sonatype/Maven Central

```scala
// project/plugins.sbt
addSbtPlugin("com.github.sbt" % "sbt-pgp" % "2.2.1")
addSbtPlugin("org.xerial.sbt" % "sbt-sonatype" % "3.9.21")

// build.sbt
publishTo := sonatypePublishToBundle.value

publishMavenStyle := true

sonatypeCredentialHost := "s01.oss.sonatype.org"
sonatypeRepository := "https://s01.oss.sonatype.org/service/local"

credentials += Credentials(
  "Sonatype Nexus Repository Manager",
  "s01.oss.sonatype.org",
  sys.env.getOrElse("SONATYPE_USERNAME", ""),
  sys.env.getOrElse("SONATYPE_PASSWORD", "")
)
```

**Процесс публикации:**
```bash
# 1. Подписать артефакты
sbt publishSigned

# 2. Выпустить на staging
sbt sonatypeBundleRelease

# 3. Или всё вместе
sbt publishSigned sonatypeBundleRelease
```

### Публикация в локальный репозиторий

```bash
# Публикация в ~/.ivy2/local
sbt publishLocal

# Публикация в ~/.m2/repository (Maven формат)
sbt publishM2
```

### Семантическое версионирование

```scala
// project/plugins.sbt
addSbtPlugin("com.github.sbt" % "sbt-dynver" % "5.0.1")

// build.sbt
ThisBuild / dynverSeparator := "-"

// Версия будет автоматически определена из git тегов
// v1.0.0 -> 1.0.0
// v1.0.0-3-g1234abc -> 1.0.0-3-g1234abc
// v1.0.0-3-g1234abc-dirty -> 1.0.0-3-g1234abc-20230101-123456-dirty
```

### Cross-publishing

```scala
// Публикация для нескольких версий Scala
crossScalaVersions := Seq("2.12.18", "2.13.12", "3.3.1")

// Публикация для всех версий
// sbt +publishSigned
```

### GitHub Packages

```scala
ThisBuild / githubOwner := "your-username"
ThisBuild / githubRepository := "your-repo"

publishTo := Some(
  "GitHub Package Registry" at "https://maven.pkg.github.com/your-username/your-repo"
)

credentials += Credentials(
  "GitHub Package Registry",
  "maven.pkg.github.com",
  "your-username",
  sys.env.getOrElse("GITHUB_TOKEN", "")
)
```

---

## Продвинутые темы

### Инкрементальная компиляция

SBT использует Zinc для инкрементальной компиляции.

```scala
// Настройки инкрементальной компиляции
Compile / incOptions := incOptions.value.withApiDebug(false)

// Очистка кеша компиляции
// sbt clean
// sbt "reload plugins"
```

**Понимание зависимостей:**
- **API зависимости** — изменения в сигнатурах
- **Внутренние зависимости** — изменения в реализации

### Кеширование

```scala
// Глобальные настройки кеша
Global / useCoursier := true

// Очистка Coursier кеша
// cs clean <org>:<name>:<version>
```

### Scope делегирование

```scala
// SBT ищет настройки в следующем порядке:
// 1. Текущий project / конфигурация / task
// 2. Текущий project / конфигурация / Global
// 3. Текущий project / Global / task
// 4. Global / конфигурация / task
// 5. и т.д.

// Пример
Compile / scalacOptions := Seq("-Xfatal-warnings")
// Применяется к Compile конфигурации

Test / scalacOptions := (Compile / scalacOptions).value :+ "-Xlog-implicits"
// Test наследует настройки Compile и добавляет свои
```

### Custom конфигурации

```scala
// Создание своей конфигурации
lazy val CustomConfig = config("custom") extend Compile

lazy val root = project
  .configs(CustomConfig)
  .settings(
    inConfig(CustomConfig)(Defaults.compileSettings ++ Seq(
      scalaSource := baseDirectory.value / "src" / "custom" / "scala",
      scalacOptions ++= Seq("-Xexperimental")
    ))
  )

// Использование
// sbt custom:compile
```

### Source generators

```scala
// Генерация кода во время компиляции
Compile / sourceGenerators += Def.task {
  val file = (Compile / sourceManaged).value / "Generated.scala"
  val content =
    s"""package generated
       |
       |object Generated {
       |  val version = "${version.value}"
       |  val buildTime = ${System.currentTimeMillis()}L
       |}
       |""".stripMargin
  IO.write(file, content)
  Seq(file)
}.taskValue
```

### Conditional compilation

```scala
// Компиляция в зависимости от условий
libraryDependencies ++= {
  CrossVersion.partialVersion(scalaVersion.value) match {
    case Some((2, n)) if n >= 13 =>
      Seq("org.scala-lang.modules" %% "scala-parallel-collections" % "1.0.4")
    case Some((3, _)) =>
      Seq("org.scala-lang.modules" %% "scala-parallel-collections" % "1.0.4")
    case _ =>
      Seq.empty
  }
}

// Conditional settings
lazy val root = project.settings(
  Seq(
    scalacOptions ++= {
      if (insideCI.value) Seq("-Xfatal-warnings")
      else Seq.empty
    }
  )
)
```

### Профили сборки

```scala
// Разные настройки для разных окружений
lazy val DevProfile = config("dev").extend(Compile)
lazy val ProdProfile = config("prod").extend(Compile)

lazy val root = project
  .configs(DevProfile, ProdProfile)
  .settings(
    inConfig(DevProfile)(
      Seq(
        scalacOptions ++= Seq("-Xlog-implicits"),
        javaOptions ++= Seq("-Xmx1G")
      )
    ),
    inConfig(ProdProfile)(
      Seq(
        scalacOptions ++= Seq("-opt:l:inline", "-opt-inline-from:**"),
        javaOptions ++= Seq("-Xmx4G")
      )
    )
  )
```

### Макросы и метапrogramming

```scala
// Для Scala 2 макросов
libraryDependencies ++= {
  CrossVersion.partialVersion(scalaVersion.value) match {
    case Some((2, n)) if n >= 13 =>
      Seq("org.scala-lang" % "scala-reflect" % scalaVersion.value)
    case _ =>
      Seq.empty
  }
}

// Paradise плагин для Scala 2.12
libraryDependencies ++= {
  CrossVersion.partialVersion(scalaVersion.value) match {
    case Some((2, n)) if n <= 12 =>
      Seq(compilerPlugin("org.scalamacros" % "paradise" % "2.1.1" cross CrossVersion.full))
    case _ =>
      Seq.empty
  }
}

// Для Scala 2.13+
scalacOptions ++= {
  CrossVersion.partialVersion(scalaVersion.value) match {
    case Some((2, n)) if n >= 13 =>
      Seq("-Ymacro-annotations")
    case _ =>
      Seq.empty
  }
}
```

---

## Оптимизация и Best Practices

### Ускорение сборки

**1. Использование Coursier:**
```scala
// Уже включено по умолчанию в SBT 1.3+
ThisBuild / useCoursier := true
```

**2. Параллельное выполнение:**
```scala
Global / concurrentRestrictions := Seq(
  Tags.limit(Tags.CPU, Runtime.getRuntime.availableProcessors()),
  Tags.limit(Tags.Network, 10),
  Tags.limit(Tags.Test, 1)
)
```

**3. Увеличение памяти:**
```bash
# .sbtopts
-J-Xmx4G
-J-Xss4M
-J-XX:ReservedCodeCacheSize=512M
-J-XX:+UseG1GC
```

**4. Отключение документации при разработке:**
```scala
Compile / doc / sources := Seq.empty
Compile / packageDoc / publishArtifact := false
```

**5. Отключение ненужных задач:**
```scala
// При локальной разработке
publishArtifact := false
```

### Структурирование build.sbt

**Плохо:**
```scala
// Всё в одном файле
lazy val root = project
  .settings(
    name := "myproject",
    version := "1.0.0",
    libraryDependencies += "org.typelevel" %% "cats-core" % "2.10.0",
    libraryDependencies += "org.typelevel" %% "cats-effect" % "3.5.2",
    // ... 100 строк настроек
  )
```

**Хорошо:**
```scala
// build.sbt - минимальный
lazy val root = project
  .settings(CommonSettings.settings)
  .settings(
    name := "myproject",
    libraryDependencies ++= Dependencies.all
  )

// project/CommonSettings.scala
object CommonSettings {
  val settings = Seq(...)
}

// project/Dependencies.scala
object Dependencies {
  val all = Seq(...)
}
```

### Управление зависимостями

**Хорошие практики:**
```scala
// 1. Явные версии
val catsVersion = "2.10.0"
libraryDependencies += "org.typelevel" %% "cats-core" % catsVersion

// 2. Группировка связанных зависимостей
object Dependencies {
  val http4s = Seq(
    "org.http4s" %% "http4s-dsl",
    "org.http4s" %% "http4s-ember-server",
    "org.http4s" %% "http4s-ember-client"
  ).map(_ % "0.23.23")
}

// 3. Исключение транзитивных зависимостей
libraryDependencies += "com.example" %% "library" % "1.0.0" excludeAll(
  ExclusionRule("org.slf4j")
)

// 4. Проверка обновлений
// sbt dependencyUpdates
```

### Настройки компилятора

**Рекомендуемые опции для Scala 2.13:**
```scala
scalacOptions ++= Seq(
  "-encoding", "UTF-8",
  "-deprecation",
  "-feature",
  "-unchecked",
  "-language:higherKinds",
  "-language:implicitConversions",
  "-Xlint",
  "-Ywarn-dead-code",
  "-Ywarn-value-discard",
  "-Ywarn-numeric-widen",
  "-Ywarn-unused:imports",
  "-Ywarn-unused:patvars",
  "-Ywarn-unused:privates",
  "-Ywarn-unused:locals",
  "-Xfatal-warnings"  // Предупреждения как ошибки
)

// Отключить -Xfatal-warnings для консоли
Compile / console / scalacOptions := (Compile / console / scalacOptions).value
  .filterNot(Set("-Xfatal-warnings", "-Xlint"))
```

**Для Scala 3:**
```scala
scalacOptions ++= Seq(
  "-encoding", "UTF-8",
  "-deprecation",
  "-feature",
  "-unchecked",
  "-language:higherKinds",
  "-Xfatal-warnings",
  "-Ykind-projector"
)
```

### Continuous Build

```bash
# Автоматическая перекомпиляция при изменении файлов
sbt ~compile

# Автоматический запуск тестов
sbt ~test

# Автоматический запуск приложения (требует sbt-revolver)
sbt ~reStart
```

### CI/CD интеграция

**GitHub Actions пример:**
```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup JDK
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '11'
          cache: 'sbt'
      
      - name: Run tests
        run: sbt clean coverage test coverageReport
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: target/scala-2.13/scoverage-report/scoverage.xml
  
  publish:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup JDK
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '11'
          cache: 'sbt'
      
      - name: Publish
        run: sbt publishSigned
        env:
          SONATYPE_USERNAME: ${{ secrets.SONATYPE_USERNAME }}
          SONATYPE_PASSWORD: ${{ secrets.SONATYPE_PASSWORD }}
```

---

## Практические проекты

### Проект 1: HTTP сервис с http4s

**build.sbt:**
```scala
ThisBuild / organization := "com.example"
ThisBuild / version := "1.0.0"
ThisBuild / scalaVersion := "2.13.12"

lazy val root = (project in file("."))
  .enablePlugins(JavaAppPackaging, DockerPlugin)
  .settings(
    name := "http4s-service",
    
    libraryDependencies ++= Seq(
      // HTTP4s
      "org.http4s" %% "http4s-ember-server" % "0.23.23",
      "org.http4s" %% "http4s-ember-client" % "0.23.23",
      "org.http4s" %% "http4s-dsl" % "0.23.23",
      "org.http4s" %% "http4s-circe" % "0.23.23",
      
      // JSON
      "io.circe" %% "circe-generic" % "0.14.6",
      "io.circe" %% "circe-parser" % "0.14.6",
      
      // Logging
      "ch.qos.logback" % "logback-classic" % "1.4.11",
      
      // Testing
      "org.scalatest" %% "scalatest" % "3.2.17" % Test,
      "org.http4s" %% "http4s-ember-client" % "0.23.23" % Test
    ),
    
    // Docker настройки
    Docker / packageName := "http4s-service",
    Docker / version := version.value,
    dockerBaseImage := "openjdk:11-jre-slim",
    dockerExposedPorts := Seq(8080),
    dockerUpdateLatest := true
  )
```

### Проект 2: Библиотека с cross-compilation

**build.sbt:**
```scala
ThisBuild / organization := "com.example"
ThisBuild / version := "1.0.0"
ThisBuild / scalaVersion := "2.13.12"
ThisBuild / crossScalaVersions := Seq("2.12.18", "2.13.12", "3.3.1")

lazy val core = (project in file("core"))
  .settings(
    name := "mylib-core",
    
    libraryDependencies ++= Seq(
      "org.typelevel" %% "cats-core" % "2.10.0",
      "org.scalatest" %% "scalatest" % "3.2.17" % Test
    ),
    
    // Настройки публикации
    publishTo := sonatypePublishToBundle.value,
    publishMavenStyle := true,
    licenses := Seq("Apache-2.0" -> url("http://www.apache.org/licenses/LICENSE-2.0")),
    
    scmInfo := Some(
      ScmInfo(
        url("https://github.com/example/mylib"),
        "scm:git@github.com:example/mylib.git"
      )
    ),
    
    developers := List(
      Developer(
        id = "yourname",
        name = "Your Name",
        email = "your@email.com",
        url = url("https://yourwebsite.com")
      )
    )
  )

lazy val root = (project in file("."))
  .aggregate(core)
  .settings(
    name := "mylib",
    publish / skip := true
  )
```

### Проект 3: Многомодульное приложение

```scala
// build.sbt
ThisBuild / organization := "com.example"
ThisBuild / version := "1.0.0"
ThisBuild / scalaVersion := "2.13.12"

lazy val commonSettings = Seq(
  scalacOptions ++= Seq(
    "-deprecation",
    "-feature",
    "-Xlint",
    "-Xfatal-warnings"
  ),
  
  Test / parallelExecution := false
)

// Модели данных
lazy val models = (project in file("modules/models"))
  .settings(commonSettings)
  .settings(
    name := "myapp-models",
    libraryDependencies ++= Seq(
      "io.circe" %% "circe-core" % "0.14.6",
      "io.circe" %% "circe-generic" % "0.14.6"
    )
  )

// Слой базы данных
lazy val database = (project in file("modules/database"))
  .settings(commonSettings)
  .settings(
    name := "myapp-database",
    libraryDependencies ++= Seq(
      "org.tpolecat" %% "doobie-core" % "1.0.0-RC4",
      "org.tpolecat" %% "doobie-hikari" % "1.0.0-RC4",
      "org.postgresql" % "postgresql" % "42.6.0"
    )
  )
  .dependsOn(models)

// HTTP API
lazy val api = (project in file("modules/api"))
  .settings(commonSettings)
  .settings(
    name := "myapp-api",
    libraryDependencies ++= Seq(
      "org.http4s" %% "http4s-ember-server" % "0.23.23",
      "org.http4s" %% "http4s-dsl" % "0.23.23",
      "org.http4s" %% "http4s-circe" % "0.23.23"
    )
  )
  .dependsOn(models, database)

// CLI приложение
lazy val cli = (project in file("modules/cli"))
  .enablePlugins(JavaAppPackaging)
  .settings(commonSettings)
  .settings(
    name := "myapp-cli",
    mainClass := Some("com.example.cli.Main"),
    libraryDependencies ++= Seq(
      "com.github.scopt" %% "scopt" % "4.1.0"
    )
  )
  .dependsOn(models, database)

// Веб приложение
lazy val web = (project in file("modules/web"))
  .enablePlugins(JavaAppPackaging, DockerPlugin)
  .settings(commonSettings)
  .settings(
    name := "myapp-web",
    mainClass := Some("com.example.web.Main"),
    
    Docker / packageName := "myapp-web",
    dockerBaseImage := "openjdk:11-jre-slim",
    dockerExposedPorts := Seq(8080),
    
    libraryDependencies ++= Seq(
      "ch.qos.logback" % "logback-classic" % "1.4.11"
    )
  )
  .dependsOn(api, models, database)

// Корневой проект
lazy val root = (project in file("."))
  .aggregate(models, database, api, cli, web)
  .settings(
    name := "myapp",
    publish / skip := true
  )
```

### Проект 4: Библиотека с плагином

**project/MyPlugin.scala:**
```scala
package myplugin

import sbt._
import sbt.Keys._

object MyPlugin extends AutoPlugin {
  override def trigger = allRequirements

  object autoImport {
    val greet = taskKey[Unit]("Greets the user")
    val greeting = settingKey[String]("The greeting to use")
  }

  import autoImport._

  override lazy val projectSettings = Seq(
    greeting := "Hello",
    greet := {
      val log = streams.value.log
      val msg = greeting.value
      val projName = name.value
      log.info(s"$msg from $projName!")
    }
  )
}
```

**build.sbt:**
```scala
lazy val plugin = (project in file("plugin"))
  .enablePlugins(SbtPlugin)
  .settings(
    name := "sbt-myplugin",
    organization := "com.example",
    version := "1.0.0",
    sbtPlugin := true,
    
    scriptedLaunchOpts := { 
      scriptedLaunchOpts.value ++
      Seq("-Xmx1024M", "-Dplugin.version=" + version.value)
    },
    scriptedBufferLog := false
  )

lazy val root = (project in file("."))
  .aggregate(plugin)
  .settings(
    publish / skip := true
  )
```

---

## Полезные ресурсы

### Официальная документация
- [SBT Reference Manual](https://www.scala-sbt.org/1.x/docs/)
- [SBT by Example](https://www.scala-sbt.org/1.x/docs/sbt-by-example.html)
- [SBT Best Practices](https://github.com/sbt/sbt/wiki/Best-Practices)

### Книги
- "sbt in Action" - Joshua Suereth and Matthew Farwell
- "Scala for the Impatient" (Chapter on SBT) - Cay Horstmann

### Плагины
- [SBT Plugin Best Practices](https://www.scala-sbt.org/1.x/docs/Plugins-Best-Practices.html)
- [Community Plugins](https://www.scala-sbt.org/1.x/docs/Community-Plugins.html)

### Сообщество
- [SBT GitHub Discussions](https://github.com/sbt/sbt/discussions)
- [Scala Users Forum](https://users.scala-lang.org/)
- Stack Overflow (тег sbt)

### Полезные команды (шпаргалка)

```bash
# Основные команды
sbt compile          # Компиляция
sbt run             # Запуск
sbt test            # Тесты
sbt clean           # Очистка
sbt console         # REPL

# Dependency management
sbt "show libraryDependencies"  # Показать зависимости
sbt dependencyTree              # Дерево зависимостей
sbt dependencyUpdates           # Проверить обновления
sbt evicted                     # Показать конфликты версий

# Publishing
sbt publishLocal    # Публикация локально
sbt publishSigned   # Публикация с подписью

# Мультипроекты
sbt projects        # Список проектов
sbt "project core"  # Переключиться на проект
sbt core/compile    # Скомпилировать проект

# Continuous build
sbt ~compile        # Авто-компиляция
sbt ~test           # Авто-тестирование

# Информация
sbt settings        # Все settings
sbt tasks           # Все tasks
sbt "inspect <key>" # Информация о key
```

---

## План обучения (рекомендуемый порядок)

### Неделя 1: Основы
- Установка и настройка SBT
- Структура проекта
- Базовые концепции (settings, tasks, scopes)
- Простая конфигурация build.sbt
- **Практика:** Создать простой проект, скомпилировать и запустить

### Неделя 2: Управление зависимостями
- Добавление зависимостей
- Resolvers и репозитории
- Управление версиями
- Транзитивные зависимости
- **Практика:** Добавить зависимости для http4s проекта

### Неделя 3: Задачи и команды
- Встроенные задачи
- Создание своих задач
- Зависимости между задачами
- **Практика:** Создать custom tasks для генерации кода

### Неделя 4: Плагины
- Использование популярных плагинов
- Assembly, Native Packager
- ScalaFmt, Scoverage
- **Практика:** Настроить Docker packaging

### Неделя 5: Мультипроектная структура
- Определение подпроектов
- Зависимости между проектами
- Агрегация
- Общие настройки
- **Практика:** Создать многомодульное приложение

### Неделя 6-7: Продвинутые темы
- Тестирование
- Публикация артефактов
- Custom конфигурации
- Source generators
- **Практика:** Опубликовать библиотеку в Maven Central

### Неделя 8: Оптимизация
- Ускорение сборки
- Best practices
- CI/CD интеграция
- **Практика:** Настроить GitHub Actions

---

## Заключение

SBT — мощный и гибкий инструмент сборки для Scala проектов. Хотя кривая обучения может быть крутой, понимание его концепций открывает огромные возможности для автоматизации и управления проектами.

### Ключевые моменты:
1. **Settings** вычисляются один раз, **tasks** — каждый раз при вызове
2. **Scopes** определяют контекст для settings и tasks
3. **Плагины** расширяют функциональность SBT
4. **Мультипроектная структура** помогает организовать большие кодовые базы
5. **Инкрементальная компиляция** значительно ускоряет разработку

### Дальнейшие шаги:
- Изучите исходный код популярных Scala проектов
- Создайте свой плагин
- Участвуйте в сообществе
- Автоматизируйте рутинные задачи

Удачи в освоении SBT! 🚀
