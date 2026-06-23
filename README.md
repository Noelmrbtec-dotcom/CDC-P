# CDC-P v3.0 - Concurrency Deterministic Control with Preemption

[![Platform](https://img.shields.io/badge/platform-PIC16F628A-blue)](https://www.microchip.com/en-us/product/PIC16F628A)
[![ROM](https://img.shields.io/badge/ROM-1188%20words%20(58%25)-green)]()
[![RAM](https://img.shields.io/badge/RAM-78%20bytes%20(35%25)-brightgreen)]()
[![Stack](https://img.shields.io/badge/Stack-4%20of%208%20levels-orange)]()
[![Overhead](https://img.shields.io/badge/Overhead-112%C2%B5s%20(1.4%25)-blue)]()
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

**Um RTOS determinístico com prioridade tridimensional, auto-regulagem temporal, diagnóstico de pane e imunidade a condições de corrida. Tudo em 1188 palavras de ROM e 78 bytes de RAM.**

---

## 📖 Visão Geral

O **CDC-P** (*Concurrency Deterministic Control with Preemption*) é um RTOS (*Real-Time Operating System*) guiado por tempo (*Time-Triggered*) projetado para sistemas embarcados de recursos restritos. Ele prova que é possível obter concorrência real, resposta a eventos urgentes e tolerância a falhas **sem sacrificar o determinismo ou a eficiência**.

Diferentemente de RTOS preemptivos tradicionais (FreeRTOS, ThreadX), o CDC-P:

- ❌ **NÃO** usa preempção interruptiva (fonte de condições de corrida e deadlocks)
- ✅ **USA** preempção cooperativa por período variável (CDC-P)
- ✅ **USA** urgência situacional por flags binárias (URG-S)
- ✅ **É IMUNE** a condições de corrida, deadlocks e inversão de prioridade
- ✅ **É AUTOCONSCIENTE**: diagnostica pane e isola tarefas problemáticas

> *"A simplicidade é um pré-requisito para a confiabilidade."* — Edsger W. Dijkstra

---

## 🎯 Características Principais

### Prioridade Tridimensional

| Dimensão | Tipo | O que controla |
|:---|:---|:---|
| **1. Espacial** | Estática | Posição fixa no despachador |
| **2. Temporal** | Configurável | Período da tarefa (`taskX`) |
| **3. Situacional** | Dinâmica | Flag de urgência por evento (URG-S) |

### 7 Camadas de Proteção

1. **Auto-regulagem temporal** — Cada tarefa se desacelera se atrasar
2. **Bloqueio por deadline** — Isolamento de tarefas que violam limites
3. **Diagnóstico de pane** — Task10 (Xerife) detecta e sinaliza falhas
4. **Recuperação progressiva** — Task9 (Síndico) reabilita tarefas bloqueadas
5. **Fail-safe final** — Reset se o mecanismo de recuperação falhar
6. **Aceleração automática** — Tick reduz de 8ms para 2ms durante emergências
7. **URG-S** — Resposta IMEDIATA (mesma iteração) a eventos urgentes

### Tick Dinâmico em Tempo de Execução

| Tick | Frequência | Uso |
|:---:|:---:|:---|
| 2ms | 500 Hz | Emergências, controle motor |
| 4ms | 250 Hz | Aquisição rápida de dados |
| **8ms** | **125 Hz** | **Padrão do sistema** |
| 50ms | 20 Hz | Tarefas volumosas, logging |

---

## 📊 Métricas Reais (PIC16F628A)

┌─────────────────┬──────────────┬──────────────┐
│ RECURSO         │ USADO        │ LIVRE         │
├─────────────────┼──────────────┼───────────────┤
│ ROM             │ 1188 words   │ 860 words     │
│ RAM (main)      │ 78 bytes     │ 146 bytes     │
│ RAM (pior caso) │ 84 bytes     │ 140 bytes     │
│ Stack           │ 4 níveis     │ 4 níveis      │
│ Overhead/tick   │ 112µs        │ 7888µs livres │
└─────────────────┴──────────────┴───────────────┘

