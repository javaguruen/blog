---
title: Forskjellen på fields og properties i en Kotlin-klasse
date: 2026-03-31
author: Bjørn
tags:
  - Kotlin
---

I mange sammenhenger brukes field og property om en annen når vi snakker om klasser, både i Kotlin og Java.

Jeg spurte AI-en følgene:

> I en Kotlin klasse hva er forskjellen på field og property?

Synes svaret var så oversiktlig og greit at jeg tok vare på det her...


I Kotlin er property et høynivå-konsept som kombinerer et backing field, getter og setter. Et field er den underliggende lagringen i minnet.

## Property
```kotlin
class Person {
    var name: String = "Bjørn"  // Dette er en property
}
```
En property gir deg automatisk genererte getter/setter. Fra Java-siden ser du getName() og setName().

## Field
```kotlin
class Person {
    var name: String = "Bjørn"
        get() = field.uppercase()       // 'field' refererer til backing field
        set(value) {
            field = value.trim()        // 'field' refererer til backing field
        }
}
```

Et field er den faktiske minneplassen som holder verdien. I Kotlin er ikke fields direkte tilgjengelige — du får tilgang via det spesielle field-nøkkelordet kun inne i en custom accessor:

## Oppsummering

Viktig distinksjon

|                  | Property          | Field                                                          |
| ---------------- | ----------------- | -------------------------------------------------------------- |
| Definert av      | Deg i Kotlin-kode | Kompilatoren (hvis nødvendig)                                  |
| Tilgjengelig fra | Overalt           | Kun i accessors via `field`                                    |
| Kan ha logikk    | Ja (get/set)      | Nei                                                            |
| Alltid til stede | Ja                | Nei — kompilatoren lager bare et backing field hvis det trengs |

Når finnes ikke backing field?

Hvis en property beregner verdien sin uten å lagre noe, lages det ikke noe backing field:

```kotlin
class Rectangle(val width: Int, val height: Int) {
    val area: Int
        get() = width * height   // Ingen backing field — beregnes hver gang
}
```

Kort sagt: du jobber alltid med properties i Kotlin, mens fields er implementasjonsdetaljer som Kotlin håndterer for deg.