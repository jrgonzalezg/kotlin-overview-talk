### Kotlin: A language for productive developers

Olmo Gallegos & Juan Ramón González González

---

## Introduction

- The Kotlin language
  - Basics
  - Null-safety
  - Extension functions
  - Kotlin on Android? (small block about anko and other concrete details on Android Dev.)
- Functional Programming (FP) basics
  - Lambda expressions
  - FP operators available in Kotlin?
- Advanced topics
  - Algebraic Data Types (ADTs)
  - Domain Specific Languages (DSLs)
  - Coroutines
  - Small intro to HKT and its adaptation to Kotlin?
- To know more

---

## The Kotlin programming language

- Expressive
- Type-safe
- Functional (we’ll focus on that later)
- Created by JetBrains
- Official ! First class support in Android since Google I/O 2017
- Made by and for Java “sufferers”

---

## Multiple targets

- Goal: Make it possible to use Kotlin for all parts of any project
- Kotlin/JVM
  - All features and the whole standard library, production-ready
  - 100% Interoperable with Java and Android
- JavaScript
  - All language features since Kotlin 1.1 and a large part of the standard library
  - JavaScript interoperability (including Node.js and browser support)
- Kotlin / Native
  - Machine code through LLVM
  - Mac OS X 10.10 and later (x86-64), x86-64 Ubuntu Linux (14.04, 16.04 and later), Apple iOS (arm64), Raspberry Pi, cross-compiled on Linux host, more to come...
  - Not yet feature complete

---

## Features

- Data classes
- Inheritance (open, closed classes)
- Interfaces with attrs, default methods
- Generics with default template methods
- Default values for attributes
- Smart casts
- with, apply, let functions
- Coroutines! Since Kotlin 1.1
- Multiple inheritance
- Testability (JUnit4 100% supported)

---

## Data classes

- Java

```java
public class Book {
  int id;
  String title;
  int coverId;

  public int getId() {
    return id;
  }

  public void setId(int id) {
    this.id = id;
  }

  public String getTitle() {
    return title;
  }

  public void setTitle(String title) {
    this.title = title;
  }

  public int getCoverId() {
    return coverId;
  }

  public void setCoverId(int coverId) {
    this.coverId = coverId;
  }
}
```

- Kotlin

```kotlin
data class Book(val id: Int, val title: String, val coverId: Int)
```

---

## Null-Safety

Java (no optionals - available in Java 8)

```java
Book book = null;
book.getTitle();
```

Kotlin - not using optionals

```kotlin
val book: Book = null
print(book.title)
```

Kotlin - using optionals

```kotlin
val book: Book? = null
print(book.title)
```

Optionals usage

```kotlin
val book: Book? = null
print(book?.title)
```

```kotlin
val book: Book? = null
print(book!!.title)
```

---

## Inheritance

Given the previous example, we can't inherit from Book

```kotlin
data class Book(val id: Int, val title: String, val coverId: Int)
```

```kotlin
class Novel : Book(id = 0, title = "", coverId = 0)
```

Book needs to be declared as open class for inheritance to work

```kotlin
open class Book(val id: Int, val title: String, val coverId: Int)
```

---

## Interfaces with attributes and default methods

In Kotlin, interfaces can have attributes

Java

```java
public interface Request<T> {
  T client;

  void execute();
}
```

Kotlin

```kotlin
interface Request<T> {
  val client : T

  fun execute()
}
```

---

## Interfaces with attributes and default methods

Java: Default template methods are supported in abstract classes

Kotlin: Just add a method body

```java
public abstract class Request<T> {
  T client;

  void execute() {
    // Default implementation
  }
}
```

Kotlin

```kotlin
interface Request<T> {
  val client : T

  fun execute() {
    // Default implementation
  }
}
```

---

## Default values for attributes

Java: Telescoping constructor problem - Builder pattern

Kotlin: Attributes with default values to the rescue!

```kotlin
data class Book(val id: Int = 0, val title: String = "", val coverId: Int = 0)

fun initBooks() {
  val books = listOf(
    Book(id = 1),
    Book(id = 2, title = "Moby dick"),
    Book(id = 3, title = "Sherlock Holmes", coverId = 254),
    Book()
  )

  books.forEach {
    print(it.title)
  }
}
```

---

## Smart casts

Null safety

```kotlin
val book : Book?

book.title

book?.title

if (book != null) {
  book.title
}
```

Smart type casts

```kotlin
open class Book(val id: Int = 0, val title: String = "", val coverId: Int = 0)

class Novel : Book(id = 0, title = "", coverId = 0) {
  var summary: String = ""
}

val novel: Book = Novel()

(novel as Novel).summary = "Once upon a time..."

if (novel is Novel) {
  novel.summary = "Once upon a time..."
}
```

---

## Algebraic Data Types (ADTs) - Product Types

- Seen in the wild as: Tuples, Records, POJOs…
- Directly supported in most languages
- Kotlin: Ideally as a data class

```kotlin
data class Book(val id: Int, val title: String, val coverId: Int)
```

- Always contains an id, a title and a coverId
- There are as many possible inhabitants of the type as the product of the number of values in each of the composing subtypes

---

## Algebraic Data Types (ADTs) - Sum Types

- Also named: Coproducts, Tagged Types, Disjoint Unions or Variant Types
- No direct support on many languages
- Kotlin: Use sealed classes

```kotlin
sealed class BookResult {
    class  BookFound(val book: Book) : BookResult()
    object BookNotFound : BookResult()
    object BookUnavailable : BookResult()
}
```

- It can contain only one of the enclosed types and nothing else
- As many possible inhabitants as the sum of inhabitants on each subtype

---

## Algebraic Data Types (ADTs) - Example

- Based on https://fsharpforfunandprofit.com/ddd/

```kotlin
typealias ContactName = String
data class Contact(val name: ContactName, val primaryContactInfo: ContactInfo, val secondaryContactInfo: ContactInfo? = null)

sealed class ContactInfo {
    class EmailContactInfo(val email: String): ContactInfo()
    class PostalContactInfo(val address: String): ContactInfo()
}

abstract fun sendEmail(emailContactInfo: EmailContactInfo) // Somewhere

fun sendMessageUsingPrimaryContactInfo(contact: Contact): Unit =
  when (contact.primaryContactInfo) {
    is ContactInfo.EmailContactInfo -> sendEmail(contact.primaryContactInfo)
    is ContactInfo.PostalContactInfo -> TODO() // NotImplementedError
  }
```

- Combines product and sum types to better model the domain
- Ideal for domain-driven design
- A Contact requires a name and at least a primaryContactInfo
- It is not possible to create a contact with no primary contact info (or null)
- when branches are autocompleted, include smarcasts and the compiler ensures you handle all ContactInfo options => Less tests
- TODO() => Runtime Exception, add to static code analysis (detekt in Kotlin, checkstyle, findbugs)

---

## Domain Specific Languages (DSLs)

- Life is Great and Everything Will Be Ok, Kotlin is Here (Google I/O '17 - https://www.youtube.com/watch?v=fPzxfeDJDzY)

```kotlin
val db: SQLiteDatabase = // ...
db.transaction {
    delete("users", "first_name = ?", arrayOf("jake"))
    // ….
}

inline fun SQLiteDatabase.transaction(body: SQLiteDatabase.() -> Unit) {
    beginTransaction()
    try {
        body()
        setTransactionSuccessful()
    } finally {
        endTransaction()
    }
}
```

---

## Coroutines

- Computations that can be suspended without blocking a thread
- Very lightweight: Millions of coroutines can run on a few threads
- A suspended coroutines does not consume a thread
- Allow to express an asynchronous computation on a sequential way
- No need to use callbacks
- Experimental feature on Kotlin 1.1 (API may still change)
- Implemented through compiler transformations, no OS / VM support  needed
- Language support: Suspending functions (suspend keyword)
- Low level core API in the Kotlin Standard Library
- High level APIs that can be used directly in the user code

---

## Coroutines

https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md

```kotlin
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

---

## Coroutines

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async(CommonPool) { doSomethingUsefulOne() }
        val two = async(CommonPool) { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}
```

It produces something like this:

The answer is 42

Completed in 1017 ms

---

## To know more

- References
  - Documentation - Kotlin Programming Language
    - https://kotlinlang.org/docs/
  - Kotlin and Android | Android Developers
  - https://developer.android.com/kotlin/index.html
  - Books
    - Kotlin for Android Developers - Antonio Leiva
      - https://leanpub.com/kotlin-for-android-developers/
    - Kotlin in Action - Dmitry Jemerov and Svetlana Isakova
      - https://www.manning.com/books/kotlin-in-action
  - Lambda World - October 26th & 27th, 2017: http://www.lambda.world/
    - Hadi Hariri - VP of Developer Advocacy, JetBrains - Kotlin talk

---

## To know more

- HelloKotlin
  - Some learning and feature testing with the Kotlin language
  - Model View Presenter (MVP)
  - Repository Pattern
  - Many tests (reword this)
  - Single God Package, Thermosiphon Samples
  - https://github.com/voghDev/HelloKotlin
- OpenLibraryApp
  - Sample Android App to query and display Open Library data
  - Dependency Injection (DI) with Dagger 2
  - Clean Architecture, Repository Pattern, MVP
  - Domain-driven design, Package by feature
  - Functional Programming (FP), Coroutines
  - https://github.com/jrgonzalezg/OpenLibraryApp
