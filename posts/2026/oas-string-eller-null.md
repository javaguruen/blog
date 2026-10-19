---
title: OAS-dokumentasjon av nullbare typer
date: 2026-10-19
author: Bjørn
tags:
  - OAS
  - SpringDoc
---

# `"type": ["string", "null"]` i OpenAPI – hva betyr det for de som integrerer?

I OpenAPI 3.1 dukker det ofte opp properties som dette:

```json
"mellomnavn": {
  "type": [
    "string",
    "null"
  ]
}
```

Hva betyr det i praksis for de som skal integrere mot API-et?

## Type og `required` er to forskjellige ting

`"type": ["string", "null"]` betyr at **verdien** kan være en tekst eller `null`. Det sier ingenting om hvorvidt feltet er med i JSON-en. Det styres av `required`. Sammen gir de fire kombinasjoner:

| `required` | Type                 | Hva klienten kan få                          |
|------------|----------------------|----------------------------------------------|
| ja         | `string`             | Alltid med, alltid tekst                     |
| ja         | `["string", "null"]` | Alltid med, men kan være `null`              |
| nei        | `string`             | Kan mangle, men er tekst når den er med      |
| nei        | `["string", "null"]` | Kan mangle, være `null` eller være tekst     |

Et ikke-påkrevd felt med `["string", "null"]` (typisk en nullbar `String?` i Kotlin) havner i den siste raden. Klienten må da håndtere at feltet kan mangle, kan være `null` og kan være tekst.

I OpenAPI 3.0 skrev man det samme som `"type": "string"` sammen med `"nullable": true`.

## Hva betyr det for de som integrerer?

- **Lesing:** De må tåle `null` og ikke anta at verdien er en tekst. Kodegeneratorer lager typer som `String?`, `string | null` eller `Optional<String>`. Uten `null` i typen lager noen generatorer en type som ikke tillater null. Da feiler deserialiseringen hvis API-et likevel sender `null`.
- **Sending (i request):** De kan sende `"mellomnavn": null`, og API-et skal godta det. Når både «mangler» og `null` er gyldig, bør dokumentasjonen si om de betyr det samme. Det har betydning for PATCH, der «ikke send» ofte betyr «ikke endre» og `null` betyr «slett».
- **Validering:** Klienter som validerer mot skjemaet strengt, avviser `null` hvis typen bare er `"string"`. Med `["string", "null"]` går det gjennom.

## Sammenhengen med hvordan API-et serialiserer

Om `null` faktisk sendes, avhenger av JSON-oppsettet på serversiden. Med Jackson og `JsonInclude.Include.NON_NULL` blir felt med `null`-verdi utelatt i stedet for å sendes som `null`.

- I praksis får klientene da «mangler» eller tekst. `"null"` i typen er altså strengere enn det API-et faktisk sender, men ikke feil.
- Generert dokumentasjon kan endre seg med verktøyversjoner. Med springdoc-openapi 3.1.0 blir nullbare Kotlin-typer til `["string", "null"]`. Med 3.0.2 blir de bare `"string"`.

## Er det en breaking change?

- I **responser** utvider `null` hva klienten må håndtere. Formelt er det en endring som kan brekke klienter, og verktøy som oasdiff vil sannsynligvis flagge den. I praksis brekker den ingenting hvis API-et aldri sender `null` (som med `NON_NULL`).
- I **requests** godtar API-et mer enn før, og det brekker ikke klienter.

Er det motsatt, altså at du selv integrerer mot et API som har `["string", "null"]` i et svar, må du regne med at `null` kan komme. Da må feltet i din modell være nullbart, for eksempel `String?` i Kotlin.
