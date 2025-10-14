---
title: Kjør LLM lokalt med Ollama
date: 2025-10-14
author: Bjørn
tags:
  - springboot
  - testcontainers
---

# Kjøre en LLM lokalt med Ollama

Installer Ollama fra `https://ollama.com`. Dette installerer en CLI. Bekreft at den er installert ved å kjøre `ollama --version` i en terminal.

På `https://ollama.com/search` får du en god oversikt over modeller som kan kjøres. Kjør en modell helt enkelt med `ollama run deepseek-r1:8b`.

```json
curl -v -X POST http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-r1:8b",
    "prompt": "Explain quantum computing in simple terms.",
    "stream": false
}' > response.json
```