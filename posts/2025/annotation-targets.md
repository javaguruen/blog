---
title: Annotation targets i Kotlin
date: 2025-07-21
author: Bjørn
tags:
  - Kotlin
---

Annotasjoner angir en eller flere @AnnotationTarget som spesifiserer hvor det er lov å plassere annotasjonen.

```kotlin
@Target(
    AnnotationTarget.FIELD,
    AnnotationTarget.PROPERTY_GETTER,
    AnnotationTarget.VALUE_PARAMETER,
)
annotation class Ann
```

Når Kotlin-compiler lager bytekode blir det generert mye automatisk. For klassen under blir det generert constructor, felter og getter-metoder (ingen setter når det er val). Når du setter på annotasjoner i Kotlin-koden, bør du bruke use-site targets for å angi hvor i bytekoden du vil at annotasjonen skal havne.

```kotlin
class ExampleKotlin(@field:Ann val foo: String,
              @get:Ann val bar: String,
              @param:Ann val quux: String)
```

Her er brukt tre use-site targets, `field:`, `get:` og `param:`:
* `@field:Ann val foo: String` angir at feltet foo i klassen (i bytekoden) vil blir annotert. 
* `@get:Ann val bar: String` angir at generert getter `String getBar()` vil bli annotert.
* `@param:Ann val quux: String` angir at contructor-parameter quux vil annotert. 


Dette tilsvarer å skrive en Java-klasse slik:

```java
public class ExampleJava {

  @Ann
  private final String foo;
  private final String bar;
  private final String quux;

  public ExampleJava(
      String foo,
      String bar,
      @Ann
      String quux
  ) {
    this.foo = foo;
    this.bar = bar;
    this.quux = quux;
  }

  public String getFoo() {
    return foo;
  }

  @Ann
  public String getBar() {
    return bar;
  }

  public String getQuux() {
    return quux;
  }
}
```




https://kotlinlang.org/docs/annotations.html#annotation-use-site-targets

https://www.baeldung.com/kotlin/annotations


## Spring
https://docs.spring.io/spring-framework/reference/languages/kotlin/annotations.html

If you use bean validation on classes with properties or a primary constructor with parameters, you may need to use annotation use-site targets, such as @field:NotNull or @get:Size(min=5, max=15), as described in this Stack Overflow response.

