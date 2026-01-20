---
title: Åpne et prosjekt i IntelliJ fra kommandolinjen
date: 2026-01-20
author: Bjørn
tags:
  - intellij
---

Når du skal åpne et prosjekt i IntelliJ er det litt tungvindt å først starte IntelliJ for så å åpne prosjektet eller mappen med filer du vil jobbe med. Særlig hvis du akkurat har brukt kommandolinjen til å klone ut et nytt prosjekt fra git.

Følgende endringer kan gjøres i `.zshrc` (eller tilsvarende for det shellet du bruker):

1. Legg IntelliJ til i path: `export PATH="$PATH:/Applications/IntelliJ IDEA.app/Contents/MacOS"``
2. Legg til følgende funksjon (under path-linjene):

```
function idea() {
    open -a "IntelliJ IDEA" "$1"
}
```
Del to er for å få den til å kjøer i bakgrunnen.

Står du i en mappe med prosjekt eller filer kan du nå starte IntelliJ med prosjektet åpent fra kommandolinjen ved å bare skrive `idea pom.xml` eller `idea .` hvis du bare vil åpne mappen med filer du står i.
