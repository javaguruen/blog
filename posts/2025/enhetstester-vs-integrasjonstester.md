---
title: Skille enhets- og integrasjonstester i maven
date: 2025-03-27
author: Bjørn
tags:
  - maven
  - test
---

# Hvordan skille mellom enhets- og integrasjonstester
Når vi skal bygge prosjektet med maven kan det være ønskelig å skille mellom de raske enhetstestene og de tregere integrasjonstestene. Med integrasjonstester mener jeg her tester som typisk trenger å starte hele spring-konteksten og ha tilgang til databasen, men som ikke kaller eksterne applikasjoner.

## Surefire
Enhetstester kjøres med `surefire` og vil feile på første test den kommer til som feiler. 
Surefire maven plugin er en standard plugin som ikke trenger å defineres i pom-filen, men om du ønsker å styre versjonsnummeret du bruker og evt. konfigurerer den, kan du legge inn følgende:

```xml
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>${maven-surefire-plugin.version}</version>
      </plugin>
```
Sjekk alltid opp siste versjon i maven central.

Det som er viktig å huske på er at surefire vil plukke opp test-klasser basert på klassenavnet. Den vil kjøre klasser har navn ette r følgende mønster:
- **/*Test.java
- **/Test*.java
- **/Tests*.java
- **/TestCase.java

Surefire vil trigge i maven life cycle `test`, altså når du skriver `maven test` eller en av life cyclene som kjører test som en del av sin egen. Du kan hoppe over surefire-testene ved å gi følgene argument til maven: `mvn install -DskipTests`.

## Failsafe
Failsafe vil, i motsetning til surefire, kjøre gjennom alle testene og ikke stoppe selv om en av dem feiler. Failsafe brukes typisk til integrasjonstester og kjøres i egen life cycle `mvn integration-test`. Jeg har konfigurert opp failsafe plugin på følgende måte:

```xml
      <plugin>
        <artifactId>maven-failsafe-plugin</artifactId>
        <version>${maven-failsafe-plugin.version}</version>
        <configuration>
          <skipTests>${skipITests}</skipTests>
          <argLine>
            <!-- For å slippe warnings om java agent -->
            -javaagent:${settings.localRepository}/org/mockito/mockito-core/${mockito.version}/mockito-core-${mockito.version}.jar
            -Xshare:off
          </argLine>

        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>integration-test</goal>
              <goal>verify</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
```
Jeg har lagt inn property for å hoppe over integrasjonstestene med argumentet `mvn install -DskipITests` (en stor 'I' til forskjell fra argumentet til enhetstestene).
Failsafe vil plukke opp tester som har følgende navnemønster:
- **/*IT.java
- **/IT*.java
- **/*ITCase.java

Så ved å sette opp både surefire og failsafe, og ha et bevisst forhold til hva navnet på testklassen er, så kan du skille mellom kjøring av de raske enhetstestene og de tregere integrasjonstestene hvis du ønsker det.