---
title: Testcontainer i springboot
date: 2025-03-26
author: Bjørn
tags:
  - springboot
  - testcontainers
---

# Testcontainers i Spring boot

Kodeeksemplene er laget med Spring Boot 3.4.3. 

## Konfigurasjon av testcontainer

Et sted i test-koden, lag en testkonfigurasjon for testcontaineren

```kotlin
import org.springframework.boot.test.context.TestConfiguration
import org.springframework.boot.testcontainers.service.connection.ServiceConnection
import org.springframework.context.annotation.Bean
import org.testcontainers.containers.PostgreSQLContainer
import org.testcontainers.utility.DockerImageName


@TestConfiguration(proxyBeanMethods = false)
class TestcontainerConfiguration {

    @Bean
    @ServiceConnection
    fun postgresContainer(): PostgreSQLContainer<*> {
        return PostgreSQLContainer(DockerImageName.parse("postgres:16"))
    }
}
```

Her bruker vi @ServiceConnection for å injekte alle connection paramtre til postgres-databasen og overstyre evt. parametre fra propertyfiler.

## Bruk i integrasjonstester
Enhetstester trenger ikke database, med unntak av JPA-tester. For integrasjonstester som starter opp hele Spring-konteksten for å kjøre tester, så ønsker vi å bruke testcontainers istedenfor en fysisk database som vi kanskje ikke har teknisk tilgang til fra byggeserver hvor testene kjøres fra.

I de klassene som trenger database, importer testcontainer-konfigurasjonen og Spring vil automatisk starte opp postgres-databasen før testen kjører.

```@SpringBootTest
@AutoConfigureMockMvc
@Import(TestcontainerConfiguration::class)
class SomeControllerIT {

    @Autowired
    private lateinit var repository: SomeRepository


}
```
