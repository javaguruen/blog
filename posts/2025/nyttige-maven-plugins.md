---
title: Nyttige maven plugins
date: 2025-11-20
author: Bjørn
tags:
  - maven
---

# Nyttige maven plugins
Her er en del nyttige maven plugins som jeg bruker aktivt i prosjektene mine på jobb. Denne bloggen er skrevet i november 2025, så versjonsnumre på plugins nevnt her er sannsynligvis utdaterte. Sjekk https://central.sonatype.com/ for syste versjon før du tar noen av disse i bruk.

## Generelt om bruk av denne type plugins
De fleste plugins har flere goals man kan kjøre og en del konfigurasjon man kan sette, jeg viser bare ett eksempel som er hvordan jeg bruker de forskjellige plugin-ene. Sjekk alltid nettsiden til plugin-en for dokumentasjon over goals og mulige konfigurasjoner for å finne det som passer for deg.

De fleste eksempler som viser bruk av plugin viser XML-en som kan legges inn i pom.xml-filen.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>versions-maven-plugin</artifactId>
            <version>2.19.1</version>
            <configuration>
                <excludes>
                    <exclude>org.apache.commons:commons-collections4</exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```
`mvn versions:display-property-updates -DallowAnyUpdates=false -Dversions.outputFile=versionUpdate.txt`

Jeg gjør sjelden dette fordi jeg føler at det gjør en stor XML-fil enda større, disse plugin-ene har kke noe med bygging av prosjektet å gjøre og hver gang de kommer med ny versjon så må det en Pull request (PR) til og en ny commit.

Hvis ikke det er veldig kompleks konfigurasjon føler jeg det er lettere å kjøre fra kommandolinjen ved å angi groupId, versjonsnummer og artifactId fullt ut:
```
mvn org.codehaus.mojo:versions-maven-plugin:2.19.1:display-property-updates -DallowAnyUpdates=false -Dversions.outputFile=versionUpdate.txt
```
Da trenger man ikke legge plugin-en inn i pom.xml-filen.

## Owasp dependency plugin
Jeg bruker [Owasp dependency plugin](https://dependency-check.github.io/DependencyCheck/) for å finne biblioteker som har kjente sårbarheter. 

`mvn org.owasp:dependency-check-maven:12.1.9:aggregate -Dformats=JSON,HTML`

OWASP Dependency-Check Maven Plugin bruker to API-er for å hente data om kjente sårbarheter og analysere sårbarheter og bibliotekene:
* The National Vulnerability Database (NVD) som er et register over kjente sårbarheter.
* Sonatype OSS Index som har mye informasjon om open source-biblioteker
Før kunne man bruke disse API-ene fritt, men NVD trottler requester som ikke har en API-key og Sonatype krever pålogging. Denne [bloggen](https://rieckpil.de/fix-sonatype-oss-index-errors-for-owasp-maven-plugin/) beskriver veldig godt hvordan du kan registere deg begge steder og så credentials som kan brukes når du kjører plugin-en på ditt prosjekt. Med api og credentials blir kommandolinen slik:

`mvn org.owasp:dependency-check-maven:12.1.9:aggregate -Dformats=JSON,HTML -DossIndexUsername=epost@epost.com -DossIndexPassword=topp_hemmelig_passord -DnvdApiKey=dinApiKey`

Jeg kjører typisk aggregate som vil samle funnene fra alle sub-modulene (hvis du har multi-module maven proisjekt) til én rapport. Raporten (både json og html) ligger i target-mappen `target/dependency-check-report.html`.

Dersom ett eller flere biblioteker har kjente sårbarheter, så er sannsynligvis første tiltak å se om sårbarheten er patchet i en nyere versjon, som bringer oss videre til neste plugin....

## Versions plugin
For å finne ut hvilke biblioteker jeg bruker som har nyere versjoner bruker jeg 
[Versions plugin](https://www.mojohaus.org/versions/versions-maven-plugin/index.html). Den har utallige mål og konfigurasjonsmuligheter. I mine maven-prosjekter bruker jeg alltid properties til å sette versjonsnummer på de bibliotekene jeg drar inn direkte. De direkte avhengighetene jeg legger inn er bare toppen av isfjellet av biblioteker som er med i en applikajson. Hvert bibliotek jeg inkluderer har som regel en lang liste av transitive avhengigheter som den tar med seg. Det kan være en risikosport å oppgradere en transitiv avhengighet til en nyere versjon enn biblioteket er beregnet for, så i utgangspunktet vil jeg kun sjekke om de bibliotekene jeg eksplisitt inkluderer direkte finnes i nyere versjoner. Unntaket er om Owasp plugin-en i forrige avsnitt har avdekket en sårbarhet i et transitivt inkludert bibliotek men det biblioteket som tar den med ikke har kommet med en ny versjon enda.

```
mvn org.codehaus.mojo:versions-maven-plugin:2.19.1:display-property-updates -DallowAnyUpdates=false -Dversions.outputFile=versionUpdate.txt
```
Denne kommandoen viser bare oppdateringer på de biblotekene som har versjoner i properties, oppdaterer ikke pom-filen direkte med nyeste versjon og lager en rapport i filen versionUpdate.txt. Vær obs på at om du har et multi modul maven prosjekt, så lager den en rapportfil for hver submodul...

## Oga maven plugin
Dersom versions plugin-en i avsnittet over viser at det ikke finnes noen oppdaterte versjoner av bibliotekene dine, så kan du stort sett være fornøyd med egen innsats, men det kan hende du blir lurt. Noen ganger så omstrukturerer open source-prosjektene koden sin og deler opp kodebasen i mindre moduler eller bare endrer navn på noe. Det kan hende at biblioteket du bruker har fortsatt sin eksistens under et nytt navn (groupId og/eller artifactId) og at du burde endre til det nye navnet og benytte siste versjon.

Det finnes en plugin som kan hjelpe deg å finne ut av dette: [oga-maven-plugin](https://github.com/jonathanlermitage/oga-maven-plugin).

```
mvn biz.lermitage.oga:oga-maven-plugin:1.9.4:check -DfailOnError=false
```

Du kan jo velge selv om den skal feile hvis den finner noen slike biblioteker eller ikke. Det kan også hende at i forbindelse med en ny major release så har de endret litt på navnene, men du skal bli igjen på den nåværende (major) versjonen en stund til. Det er da mulig å lage seg en liten json-fil (f.eks. `ogaIgnoreList.json`) med de bibliotekene som den skal ignprere/ikke feil for: 

```json
{
  "ignoreList": [
    {
      "item": "org.apache.httpcomponents:httpclient"
    }
  ]
}

mvn biz.lermitage.oga:oga-maven-plugin:1.9.4:check -DignoreListFile=ogaIgnoreList.json
```
Da vil den ikke lenger feile pga dette biblioteket.
