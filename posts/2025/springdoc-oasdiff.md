---
title: Generere oas-filen fra springdoc i en pipeline
date: 2025-08-27
author: Bjørn
tags:
  - springboot
  - oas
  - springdoc
---

Som en del av å bruke oasdiff i en github workflow for å sjekke om det er noen breaking API-endringer trenger jeg å sammenligne OAS-filen i min (feature) branch med den som er i produksjon. Teknologi-stacken som jeg bruker er Spring Boot, Maven og [Springdoc](https://springdoc.org/) for OAS-generering. Springdoc bruker en kombinasjon av annotasjoner og informasjon fra Spring-bønner til å generere en OAS-fil for API-et. Fordi den bruker informasjon fra Spring, så blir filen generert runtime, ikke compile-time eller build-time. 

Før jeg kan sammenligne OAS-filen slik den er i min branch med hvordan den er i produksjon for å avdekke om det er noen breaking changes i API-et, må jeg få generert OAS-filen. Her er en beskrivelse av hvordan jeg har satt opp det i Maven.

## Springdoc openapi maven plugin
Springdoc openapi maven plugin genererer oas-filen i maven sin `integration-test`-fase. I konfigurasjonen har angitt URL-en til den genererte OAS-filen på den kjørende applikasjonen. Det er også angitt hva filen skal hete lokalt. Merk at filen blir lagret i target-mappen, så det relative filnavnet fra der applikasjonen er sjekket ut er `target/openapi.json` i mitt tilfelle.

```xml
<plugin>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-maven-plugin</artifactId>
    <version>${maven-springdoc-openapi-plugin.version}</version>
    <executions>
        <execution>
            <id>integration-test</id>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <apiDocsUrl>http://localhost:8080/todo/v3/api-docs</apiDocsUrl>
        <outputFileName>openapi.json</outputFileName>
    </configuration>
</plugin>
```

Merk at denne pluginen krever at applikasjonen kjører, det fikser vi med Spring Boot maven plugin.

## Spring Boot maven plugin
Denne pluginen har jeg brukt til å starte applikasjonen i `pre-integration-test`-fasen og stoppe den i `post-integration-test`-fasen. I tillegg setter jeg spring-profilen til å være oasdiff, mer om det under...

```xml 
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>${project.parent.version}</version>
    <configuration>
        <jvmArguments>-Dspring.application.admin.enabled=true -Dspring.profiles.active=oasdiff</jvmArguments>
    </configuration>
    <executions>
        <execution>
            <id>pre-integration-test</id>
            <goals>
                <goal>start</goal>
            </goals>
        </execution>
        <execution>
            <id>post-integration-test</id>
            <goals>
                <goal>stop</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## Egen maven og spring-profil
En ting jeg så når jeg prøvde ut dette var at jeg ikke vile generere OAS-filen hver gang jeg kjørte integrasjontester, og det tok en del tid å generere filen fordi applikasjonen benytter testcontainers (postgres) i integrasjontestene med tilhørende kjøring av databasepatcher vha. Flyway. Jeg ønsket heller ikke å kjøre enhets- og integrasjonstester i den arbeidsflyten hvor jeg skal sjekke for breaking changes i API-et. Å skippe surefire- og failsafe-testene gjøres på standard-måten (-DskipTests).

Løsningen for meg var som følgende:

1. Å lage en `application-oasdiff.properties` og konfigurere datasourcen til å være H2
2. Å lage en Maven-profil som heter `oasdiff`. I denne profilen har jeg lagt til H2-dependencies og lagt inn de overnevnte pluginene.

```xml
<profiles>
    <profile>
        <id>oasdiff</id>
        <dependencies>
            <!-- H2 for database -->
            <dependency>
                <groupId>com.h2database</groupId>
                <artifactId>h2</artifactId>
                <scope>runtime</scope>
                <version>${h2.version}</version>
            </dependency>
        </dependencies>
        <build>
        <!-- Plugins som kjører for kun denne profilen -->
        <plugins>
        ... se eksemplene over.
        </plugins>
    </profile>
</profiles>
```

Den store fordelen med å lage en egen (maven) profil for dette er at jeg trenger ikke å blande inn dependencies og plugins og ekstra arbdeid i det som har med å teste og bygge applikasjonen å gjøre og jeg kan kjøre den ønskede profilen bare når jeg ønsker det.

## Github action
I github-actionen for å generere OAS-filen kjører jeg da maven med den nye profilen og hopper over både surefire- og failsafe-testene. 

```yaml
- name: Generate oasfile
  run: |
    : # Generate oasfile
    mvn verify -Poasdiff -DskipTests -DskipITests
```
