---
title: Cors-konfigurasjon i Spring Boot
date: 2025-09-19
author: Bjørn
tags:
  - cors
  - spring boot
---

# Cors-konfigurasjon i Spring Boot

Dersom du har et REST API i SpringBoot med en GUI som bruker API-et, så vil nettleseren automatisk gjøre en "pre-flight" request (et kapp med verb OPTIONS) før den sender data til et API på en annen server. Basert på headerne som kommer i retur vil den etterpå gjennomføre POST- eller PUT-kallet. Som API-eier kan du da f.eks. bestemme at kun nettsider fra godkjente servere får bruke API-et. Dette kan være et viktig middel for å bekjempe falske phishing-sider.


Første konfigurasjonsendring som må gjøres er at OPTIONS-kall kommer uten autentiseringsheader og dette må tillates. Linjen 
```kotlin
authorize(HttpMethod.OPTIONS, "/**", permitAll)
```
kan da legges inn i metoden under hvor hvert enkelt endepunkt konfigureres med roller/scopes som kreves for å ha lov til å kalle endepunktet. Denne linjen vil tillate OPTIONS-kall på alle endepunkter uten autentisering.


Metoden `corsConfigurationSource()` lager en bønne med Cors-konfigurasjon. I eksempelet under åpner egentlig for alle metoder, på alle endepunkter fra alle servere og er den mest åpne konfigurasjonen du kan lage. Den legger heller ikke begrensning på headere som kan sendes inn og tillater sending av credentials i OPTIONS-kall. Hvis ikke du trenger en så åpen konfigurasjon, så bør du begrense den mer.

```kotlin

@Configuration
class SecurityConfig{
    @Bean
    fun corsConfigurationSource(): CorsConfigurationSource {
        val configuration = CorsConfiguration()
        configuration.allowedOriginPatterns = listOf("*")
        configuration.allowedMethods = listOf("GET", "POST", "PUT", "DELETE", "OPTIONS", "HEAD", "PATCH")
        configuration.allowedHeaders = listOf("*")
        configuration.allowCredentials = true

        val source = UrlBasedCorsConfigurationSource()
        source.registerCorsConfiguration("/**", configuration)
        return source
    }

    @Bean
    fun securityFilterChain(http: HttpSecurity, authenticationEntryPoint: AuthenticationEntryPoint): SecurityFilterChain {

        http.cors { corsConfigurer -> corsConfigurer.configurationSource(corsConfigurationSource())}
        http {
            authorizeHttpRequests {
                authorize(HttpMethod.OPTIONS, "/**", permitAll)
                ... vanlig konfigurasjon av alle endepunktene
                authorize(anyRequest, denyAll)
            }
        }
        return http.build()
    }

}
```

