---
title: Required parameter i Kotlin-kode blir ikke håndhevet
date: 2025-03-28
author: Bjørn
tags:
  - kotlin
  - springboot
---

Oppdaget til min store forskrekkelse at en klasse som representerer request body til resttjenesten hadde en ikke-nullbar property `val ferdig: Boolean` i Kotlin-klassen. Lagde en test som ikke hadde denne propertyen i Json-grafen som ble sendt inn og forventet at den skulle feile med 400 Bad request. Det gjorde den ikke. I databasen hadde den fått veriden `false`.

Etter en del graving kom jeg fram til at en ikke-nullbar "primitiv" verdi i Kotlin, vil kompileres til en `boolean ferdig` primitiv i byte-koden. Java-primitiver har alltid en default-verdi, false for boolean, hvis en verdi ikke er blitt tilordnet. Det viser seg at desrialisering av Json derfor ikke feiler om propertyen mangler fordi den likevel har fått en verdi i objektet.

Løsningen er å konfigurerer Jackson object mapper til å feile i disse tilfellene:

```kotlin
fun create(): ObjectMapper {
        return JsonMapper.builder()
            .enable(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES)
            .build()
    }
```