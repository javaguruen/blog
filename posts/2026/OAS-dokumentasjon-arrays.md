---
title: OAS-dokumentasjon av arrays
date: 2026-10-19
author: Bjørn
tags:
  - OAS
  - SpringDoc
  - Swagger
---

## Dokumentasjon av listen

I SpringBoot er det vanlig å annotere koden med Swagger-annotasjoner som Springdoc bruker til å generere OpenAPI Specification-filen (OAS-filen) til API-et ditt. I tillegg til Swagger-annotasjoner brukes også Spring-annotasjoner, Bean validation-annotasjoner, typesystemet til Kotlin (null/ikke null) og noen egne annotasjoner. OAS-filen blir da generert runtime.

Dette er ment å være en huskeliste for hvordan en skal dokumentere lister (Kotlin) som blir array i OAS. Der er det nemlig mulig å dokumentere både listen og elementene i listen.

Anta at vi har en klasse (navnet ikke viktig) som har en liste med beløper (beloep og valuta).

```kotlin
    @field:ArraySchema(
        arraySchema = Schema(description = "Liste over beløp og valuta for pantet."),
        schema = Schema(implementation = Beloep::class, description = "Pantets beløp og valuta."),
    )
    val beloeper: List<Beloep>,
```

Her er det verdt å merke seg:

1. Bruk annotasjonen `ArraySchema` for å dokumentere propertyen når det er en liste
2. I ArraySchema, så er `arraySchema`-attributten dokumentasjonen for selve listen beloeper
3. I ArraySchema, så er `schema`-attributten dokumentasjonen på elementene i listen, ikke selve listen

Dette vil resultere i en OAS-type med 
```json
    "beloeper": {
        "type": "array",
        "description": "Liste over beløp og valuta for pantet.",
        "items": {
            "$ref": "#/components/schemas/Beloep",
            "description": "Pantets beløp og valuta."
        }
    },
```

## Anngivelse av at listen skal inneholde minst ett element

Hvis du trenger å dokumentere at listen må inneholde minst ett element, kan du sette `minItems=1` i `ArraySchema`:

```kotlin
    @field:ArraySchema(
        arraySchema = Schema(description = "Liste over beløp og valuta for pantet."),
        schema = Schema(implementation = Beloep::class, description = "Pantets beløp og valuta."),
        minItems = 1
    )
    val beloeper: List<Beloep>,
```

Dette vil resultere i følgende OAS-type:

```json
    "beloeper": {
        "type": "array",
        "description": "Liste over beløp og valuta for pantet.",
        "items": {
            "$ref": "#/components/schemas/Beloep",
            "description": "Pantets beløp og valuta."
        },
        "minItems": 1
    },
```

Dette gir en riktig OAS-dokumentasjon, men gir ikke noen form for automatisk input-validering på requester som kommer inn. Validering av input på enkeltfelter gjøres vanligvis med beans validation hvor vi annoterer feltene med diverse jakarata.validation-annotasjoner som spesifiserer regler. For lister finnes det en annotasjon Size hvor man kan angi en eller begge av min og max antall elementer listen må inneholde.

Merk at hvis du bare angir min=1, slik som under:

```kotlin
    @field:ArraySchema(
        arraySchema = Schema(description = "Liste over beløp og valuta for pantet."),
        schema = Schema(implementation = Beloep::class, description = "Pantets beløp og valuta.")
    )
    @Field:Size(min=1)
    val beloeper: List<Beloep>,
```

Så vil det resultere i OAS-typen under hvor maxItems blir satt til Int.MAX_VALUE.

```json
    "beloeper": {
    "type": "array",
    "description": "Liste over beløp og valuta for pantet.",
    "items": {
        "$ref": "#/components/schemas/Beloep",
        "description": "Pantets beløp og valuta."
    },
    "maxItems": 2147483647,
    "minItems": 1
    },
```
Her ser vi at `@Size` alltid setter på en `maxItems = Int.MAX_VALUE` hvis ikke max er angitt i annotasjonen. Dette ser veldig rart ut i Redoc og Swagger-ui som vil vise `Array of objects (Beloep) [ 1 .. 2147483647 ] items`.

Hvis du ønsker å uttrykke at listen må inneholde minst ett element men ikke skal angi maks antall elementer, synes jeg OAS-dokumentasjonen blir bedre om du heller bruker `@NotEmpty` annotasjonen:

```kotlin
    @field:ArraySchema(
        arraySchema = Schema(description = "Liste over beløp og valuta for pantet."),
        schema = Schema(implementation = Beloep::class, description = "Pantets beløp og valuta.")
    )
    @Field:NotEmpty()
    val beloeper: List<Beloep>,
```

Det resulterer i at OAS-typen bare har satt en minItems som i swagger-ui og redoc blir visualisert som `not empty` eller `>=1`.

```json
    "beloeper": {
    "type": "array",
    "description": "Liste over beløp og valuta for pantet.",
    "items": {
        "$ref": "#/components/schemas/Beloep",
        "description": "Pantets beløp og valuta."
    },
    "minItems": 1
    },
```

## Bug i swagger før versjon 2.2.46. 
En bug i swagger før 2.2.46 gjør at description og minItems/maxItems på listen (beloeper) også blir generert på selve item, den "lekker ned" i items, altså:

```json
    "beloeper": {
    "type": "array",
    "description": "Liste over beløp og valuta for pantet.",
    "items": {
        "$ref": "#/components/schemas/Beloep",
        "description": "Liste over beløp og valuta for pantet.",
        "minItems": 1
    },
    "minItems": 1
    },
```

## Bug i swagger 2.2.46 og spring-doc 3.0.2

Vær obs på at om du oppgraderer swagger til 2.2.46 så har den endret hvordan required blir beregnet for felter. Spring-doc 3.0.2 og eldre tar ikke hensyn til denne endringen, så hvis du bruker de to bibliotekene, med de versjonene, sammen så vil ikke felter/properties bli merket som required selv om de er generert fra en Kotlin-type som ikke kan være null (beloap: Long). Løsningen er enten å manuelt sette på `required = true` i Schema-annotasjonen overalt i koden eller å oppgradere til en nyere Spring-doc, men det medfører andre endringer i API-dokumentasjonen

## Endringer på nullbare typer i springdoc 3.0.3 og senere

Merk at fra Springdoc 3.0.2 og eldre så vil valgfrie (kan være null) i OAS bli generert som f.eks
```json
    "mellomnavn": {
        "type": String
    }
```
Mellomnavn vil ikke stå i array `required` som betyr at den kan være null.

Dersom du oppgraderer til Spring-doc 3.0.3 eller senere vil typen til mellomnavn blir endret fra `string` til `string eller null`
```json
    "mellomnavn": {
        "type": [
            "string",
            "null"
        ]
    },
```

Dette betyr at mellomnavn kan forekommer i json med verdi null og den kan være fraværende fra json. Mange applikasjoner har konfigurert JsonMapperen sin til å ignorere felter som er null. Endringen er nok formelt sett mest korrekt, særlig for språk som skiller på null og undefined. Det er likevel en breaking change i API-et for klienter som er generert fra OAS-spesifikasjonen.