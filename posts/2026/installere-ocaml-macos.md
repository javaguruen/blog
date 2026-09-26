---
title: Installere OCaml på macos
date: 2026-09-26
author: Bjørn
tags:
  - OCaml
---

## Installere via opam
Ocaml har en pakkemanager, opam, som kan brukes til å installere både Ocaml, biblioteker og verktøy.

```
brew install opam
```
Første gang må du initialisere ~/.opam. For å slippe å skrive `eval $(opam env)` hver gang du skal bruke ocaml, kan du velge alternativet hvor du lar opam oppdatere  shellet ditt (~/.zshrc for min del).
```
opam init

Do you want opam to configure zsh?
> 1. Yes, update ~/.zshrc
  2. Yes, but don't setup any hooks. You'll have to run eval $(opam env) whenever you change your current 'opam switch'
  3. Select a different shell
  4. Specify another config file to update instead
  5. No, I'll remember to run eval $(opam env) when I need opam
```

Nå kan du installere:
- UTop, en moderne REPL (REPL: Read-Eval-Print Loop).
- Dune, et byggesystem for OCaml prosjekter (kjørbare filer biblioteker, kjøre tester mm.).
- ocaml-lsp-server Language Server Protocol for editor-støtte (VS Code, Vim, eller Emacs).
- odoc Generere dokumentasjon fra OCaml-kode.
- OCamlFormat formatterer OCaml-kode

```
opam install dune ocaml-lsp-server odoc ocamlformat utop
```

Start utop ved å skrive `utop` og avslutt med å skrive `#quit;;` inkludert '#'-tegnet.

I VS Code kan du installere plugin `OCaml Platform` fra OCaml Labs hvis du vil jobbe i VS Code istedenfor i utop.

