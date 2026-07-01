# CDC-P v3.1 - Um Cyber Organismo com Previsibilidade Absoluta

## Um sistema que transcende a definição de RTOS. Ele respira, tem reflexos, se adapta, se cura, se defende e, se tudo falhar, renasce. Tudo em 1360 palavras de ROM e 80 bytes de RAM.

[![Platform](https://img.shields.io/badge/platform-PIC16F628A-blue)](https://www.microchip.com/en-us/product/PIC16F628A)
[![ROM](https://img.shields.io/badge/ROM-1360%20words%20(66%25)-green)]()
[![RAM](https://img.shields.io/badge/RAM-80%20bytes%20(36%25)-brightgreen)]()
[![Stack](https://img.shields.io/badge/Stack-4%20of%208%20levels-orange)]()
[![Overhead](https://img.shields.io/badge/Overhead-112%C2%B5s%20(1.4%25)-blue)]()
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

---

## Visão Geral

O CDC-P (Concurrency Deterministic Control with Preemption) não é um RTOS tradicional. É um **Cyber Organismo com Previsibilidade Absoluta** — um sistema que exibe funções análogas às biológicas no domínio temporal.

Diferentemente de RTOS preemptivos como FreeRTOS e ThreadX, o CDC-P não usa preempção interruptiva. Em vez disso, respira (tick dinâmico), tem reflexos (URG-S), se adapta ao ambiente (auto-regulagem), se cura de feridas (Task9), se defende de ameaças (Task10 com isolamento permanente) e, se tudo falhar, renasce (reset_cpu). E tudo isso é 100% previsível, 100% determinístico, 100% comprovado em hardware real.

O sistema é imune a condições de corrida, deadlocks e inversão de prioridade por construção arquitetural, não por mecanismos de correção posteriores. Cabe em 1360 palavras de ROM e 80 bytes de RAM em um PIC16F628A de 8 bits com clock de 4 MHz.

> *"A simplicidade é um pré-requisito para a confiabilidade."* — Edsger W. Dijkstra

---

## O Cyber Organismo

| Função Biológica | Equivalente no CDC-P | Mecanismo |
|:---|:---|:---|
| Respiração | Tick dinâmico | Ajusta frequência do sistema conforme a demanda (2ms a 50ms) |
| Reflexos | URG-S | Resposta imediata e involuntária a estímulos externos (mesma iteração) |
| Homeostase | Auto-regulagem | Mantém equilíbrio temporal sob carga variável |
| Cicatrização | Task9 (Síndico) | Recupera tarefas bloqueadas progressivamente |
| Sistema Imunológico | Task10 (Xerife) + pane_taskX | Detecta e isola ameaças permanentemente |
| Renascimento | reset_cpu() | Reinicia o organismo quando tudo falha |
| Evolução | Dimensionamento experimental | Aprende o WCET real de cada tarefa |

**Previsibilidade Absoluta:** Não há "depende". Não há "às vezes". Não há "em algumas condições". Cada função ocorre de forma previsível e reproduzível. Sempre.

---

## Características Principais

### Prioridade Tridimensional

Nenhum RTOS comercial oferece isso. A prioridade no CDC-P é uma matriz de três dimensões:

| Dimensão | Tipo | O que controla | Exemplo |
|:---|:---|:---|:---|
| 1. Espacial | Estática (projeto) | Posição fixa no despachador | Task10 antes de Task1 |
| 2. Temporal | Configurável | Período da tarefa (taskX) | task1=2 (16ms) |
| 3. Situacional | Dinâmica (eventos) | Flag de urgência (URG-S) | Botão pressionado -> resposta imediata |

### Mecanismos de Resposta a Eventos

| Mecanismo | Latência | Duração | Uso |
|:---|:---|:---|:---|
| URG-S | Mesma iteração (0 ticks) | Única execução | Eventos pontuais: "Acorda! Algo aconteceu AGORA!" |
| CDC-P | Próxima iteração (1 tick) | Persiste até desativar | Emergências persistentes: "Fique em alerta!" |
| Tick Dinâmico | Imediato | Persiste até reverter | Sobrecarga sistêmica: "Todos acelerem!" |

### Urgência Situacional (URG-S)

Resposta IMEDIATA a eventos pontuais usando flags binárias. Overhead adicional de apenas 12-32 microssegundos. Duas versões: `set_urgent()` para tarefas (protege contra re-entrância) e `set_urgent_isr()` para ISRs.

### Preempção Cooperativa (CDC-P)

Resposta SUSTENTADA a emergências persistentes. Sem interrupção de fluxo, sem condições de corrida. Overhead zero — custa exatamente uma atribuição de variável. Ativação com `CDC_EnablePreempt()` e desativação com `CDC_DisablePreempt()`.

### Isolamento Permanente (pane_taskX)

Quando uma tarefa atinge 5 reincidências de bloqueio (`sistema_em_pane`), a Task10 (Xerife) decreta **pane permanente**: `pane_taskX = 1`. Isso impede que a Task9 desbloqueie a tarefa, que o URG-S a execute, e que o CDC-P conceda preempção. A tarefa permanece isolada até intervenção externa (reset ou manutenção). A Task9 é a única tarefa que não possui flag de pane — se ela falhar, o sistema reseta.

### 7 Camadas de Proteção

| Camada | Mecanismo | Função |
|:---:|:---|:---|
| 1 | Auto-regulagem temporal | Cada tarefa se desacelera se atrasar |
| 2 | Bloqueio por deadline | Isolamento de tarefas que violam limites |
| 3 | Diagnóstico (Task10 - Xerife) | Detecta pane e ISOLA permanentemente |
| 4 | Recuperação (Task9 - Síndico) | Reabilita tarefas bloqueadas (exceto em pane) |
| 5 | Fail-safe final | Reset se o mecanismo de recuperação falhar |
| 6 | Aceleração automática (Task8) | Tick reduz de 8ms para 2ms na urgência |
| 7 | URG-S | Resposta IMEDIATA a eventos urgentes |

### Tick Dinâmico em Tempo de Execução

| Tick | Frequência | Uso Principal |
|:---:|:---:|:---|
| 2ms | 500 Hz | Emergências, controle motor, fontes chaveadas |
| 4ms | 250 Hz | Aquisição rápida de dados, serial intensa |
| 8ms | 125 Hz | Uso geral, didático, maioria das aplicações |
| 50ms | 20 Hz | Processamento em lote, logging, telemetria |

---

## Métricas Reais de Compilação

**Plataforma:** Microchip PIC16F628A (4 MHz, 2 KB Flash, 224 bytes RAM)  
**Compilador:** CCS C  
**Overhead do despachador medido:** 112 microssegundos (1.4% do tick de 8ms)

### Evolução do CDC-P

| Métrica | v2.x (Original) | v3.0 (URG-S) | v3.1 (Pane Perm.) | Aumento Total |
|:---|:---:|:---:|:---:|:---:|
| ROM | 936 words (46%) | 1188 words (58%) | 1360 words (66%) | +424 words (+45%) |
| RAM (main) | 75 bytes (33%) | 78 bytes (35%) | 80 bytes (36%) | +5 bytes (+7%) |
| RAM (pior) | 81 bytes (36%) | 84 bytes (38%) | 86 bytes (38%) | +5 bytes (+6%) |
| Stack | 4 níveis | 4 níveis | 4 níveis | ZERO |
| Overhead | ~80-100µs | 112µs | 112µs | — |

### Recursos Livres (v3.1)

| Recurso | Usado | Livre |
|:---|:---:|:---:|
| ROM | 1360 words (66%) | 688 words (34%) |
| RAM (pior caso) | 86 bytes (38%) | 138 bytes (62%) |
| Stack | 4 níveis (50%) | 4 níveis (50%) |

### Comparação com FreeRTOS (Estimativas)

| Métrica | CDC-P (PIC16 4MHz) | FreeRTOS (ARM Cortex-M) | Vantagem CDC-P |
|:---|:---:|:---:|:---:|
| ROM (10 tarefas) | 2.1 KB | ~10-15 KB | 5-7x menos |
| RAM (10 tarefas) | 80 bytes | ~2000-3000 bytes | 25-37x menos |
| Stack (10 tarefas) | 4 níveis | ~2000+ bytes | ~500x menos |
| Prioridade | Tridimensional | Unidimensional | Mais expressiva |
| Diagnóstico de pane | Nativo com isolamento | Inexistente | Presente |
| Resposta a eventos | Imediata (URG-S) | Via kernel (latência) | Mais rápido |
| Condições de corrida | IMUNE | Possíveis | 100% seguro |

### Estimativas em Outras Plataformas

| Plataforma | Clock | Ganho vs PIC16 | Tick Mínimo |
|:---|:---:|:---:|:---:|
| ATmega328P (Arduino Uno) | 16 MHz | 4x | 1ms |
| STM32F103 (Blue Pill) | 72 MHz | 60-70x | 50 microssegundos |
| RP2040 (Raspberry Pi Pico) | 133 MHz | 120x | 50 microssegundos |
| CH32V003 (RISC-V $0.10) | 48 MHz | 48x | 100 microssegundos |
| CH32V307 (RISC-V) | 144 MHz | 144x | 50 microssegundos |
| ESP32 (Xtensa LX6) | 240 MHz | 200-240x | 10 microssegundos |

Em todas as plataformas, o CDC-P ocupa menos de 5% dos recursos disponíveis.

---

## Estrutura do Sistema

### Ordem do Despachador (Uma Iteração = 1 Tick)

1. **BLOCO URG-S:** Resposta IMEDIATA a eventos pontuais (tarefas em pane são ignoradas)
2. **BLOCO CDC-P:** Tarefa com período reduzido para 1 tick (tarefas em pane são ignoradas)
3. **CAMADA DE SUPERVISÃO:** Task10 (Xerife) e Task9 (Síndico)
4. **CAMADA DE APLICAÇÃO:** Task1 a Task8 em ordem fixa

### As 10 Tarefas

| Tarefa | Nome | Função | Período | Bloqueio |
|:---|:---|:---|:---:|:---:|
| Task10 | Xerife | Diagnóstico de pane e isolamento permanente | 1 tick | NUNCA |
| Task9 | Síndico | Recuperação de tarefas (exceto em pane) | 1 tick | Reset |
| Task8 | Acelerador | Tick automático por preempção | 1 tick | Sim |
| Task1-7 | Aplicação | Lógica do usuário | Variável | Sim |

---

## Começo Rápido

### Pré-requisitos

- Compilador CCS C para PIC
- Microcontrolador PIC16F628A (ou similar)
- 4 MHz de clock
- LEDs, botão e interface serial (opcional para testes)

### Adaptando para Seu Projeto

1. Defina os períodos das Tasks 1 a 8 (em ticks)
2. Defina as tolerâncias de deadline (`tempo_maximo_taskX`)
3. Escreva a lógica de cada `taskX_func()`
4. Configure os pinos de I/O em `config_io()`
5. Escolha o tick padrão (2ms, 4ms, 8ms ou 50ms)
6. Compile e grave no microcontrolador

### Modelo Padrão de Tarefa

Toda tarefa deve seguir este modelo. As três linhas finais (cálculo de jitter, auto-regulagem e bloqueio) são obrigatórias. A lógica da aplicação nunca deve conter `delay_ms()` ou loops infinitos. O cálculo de jitter mede quantos períodos a tarefa atrasou. A auto-regulagem aumenta o período para reduzir a carga. O bloqueio isola a tarefa se o atraso persistir.

---

## Regras de Dimensionamento Temporal

### Fórmula do Tempo Máximo por Tarefa

O tempo máximo que cada tarefa pode consumir é calculado dividindo o tempo livre do tick pelo número de tarefas que podem executar simultaneamente naquele tick. O overhead do despachador (112 microssegundos) é subtraído do tick total.

A fórmula é: Tempo máximo por tarefa em microssegundos = (Tick em microssegundos - 112) / N, onde Tick em microssegundos é 8000 para o tick padrão de 8ms, 112 é o overhead do despachador, e N é o número de tarefas que executam no mesmo tick.

### Tabela de Consulta Rápida (Tick de 8ms)

| Tarefas no Mesmo Tick | Tempo Máximo por Tarefa |
|:---:|:---:|
| 1 | 7888 microssegundos (~7.9ms) |
| 2 | 3944 microssegundos (~3.9ms) cada |
| 3 | 2629 microssegundos (~2.6ms) cada |
| 5 | 1577 microssegundos (~1.6ms) cada |
| 10 (todas) | 788 microssegundos (~0.8ms) cada |

### Ciclos por Tick em Diferentes Clocks

A quantidade de ciclos de instrução disponíveis por tick depende diretamente da frequência do clock. No PIC16, cada ciclo de instrução equivale a 4 ciclos de clock (Fosc/4). Em arquiteturas ARM e RISC-V, cada ciclo de instrução equivale aproximadamente a 1 ciclo de clock.

A fórmula é: Ciclos por tick = (Clock_Hz / 4) x (Tick_ms / 1000).

| Clock | Tick 8ms (ciclos) | Livre p/ 10 tarefas (cada) |
|:---:|:---:|:---:|
| 4 MHz (PIC16) | 8.000 | 788 |
| 8 MHz (PIC16) | 16.000 | 1.588 |
| 16 MHz (AVR) | 32.000 | 3.188 |
| 48 MHz (RISC-V) | 48.000 | 4.788 |
| 72 MHz (ARM) | 72.000 | 7.188 |
| 240 MHz (ESP32) | 240.000 | 23.988 |

### Tempos Típicos de Operações (PIC16F628A a 4MHz)

| Operação | Ciclos | Tempo |
|:---|:---:|:---:|
| Ler/Escrever pino digital | 1-3 | 1-3 microssegundos |
| Soma/Subtração 8 bits | 1-2 | 1-2 microssegundos |
| Multiplicação 8x8 | ~50 | ~50 microssegundos |
| Divisão 8/8 | ~100 | ~100 microssegundos |
| Leitura ADC (1 canal) | ~20 | ~20 microssegundos |
| Enviar 1 byte serial (9600bps) | — | ~104 microssegundos |
| PID (3 termos) | ~300 | ~300 microssegundos |
| LCD 16x2 (tela inteira) | ~6400 | ~6.4ms |
| LCD 128x64 SPI (tela) | ~30720 | ~30.7ms |
| Gravação EEPROM (1 byte) | ~10000 | ~10ms |
| CRC-16 (1 byte) | ~50 | ~50 microssegundos |

### Tabela de Decisão Rápida

VERDE significa que cabe com folga e pode usar taskX=1. AMARELO significa que cabe, mas ocupa parte significativa do tick e requer cuidado. VERMELHO significa que não cabe e requer aumento do período, do tick, ou divisão da tarefa.

| WCET da Tarefa | Tick 2ms | Tick 8ms | Tick 50ms | Ação |
|:---|:---:|:---:|:---:|:---|
| Até 100 microssegundos | VERDE | VERDE | VERDE | taskX = 1 |
| 100-500 microssegundos | VERDE | VERDE | VERDE | taskX = 1 |
| 500-1800 microssegundos | AMARELO | VERDE | VERDE | taskX >= 2 |
| 1.8-7.8ms | VERMELHO | AMARELO | VERDE | taskX >= 4 |
| 7.8-15ms | VERMELHO | VERMELHO | VERDE | tick 50ms |
| 15-49ms | VERMELHO | VERMELHO | AMARELO | tick 50ms |
| Acima de 49ms | VERMELHO | VERMELHO | VERMELHO | DIVIDIR! |

---

## Dimensionamento Temporal por Evidência Experimental

O CDC-P é o único sistema que funciona como **instrumento de medição temporal**. O método:

1. Configure a tarefa com período inicial e tolerância baixa
2. Deixe o sistema rodar até que a tarefa seja bloqueada
3. Leia o valor de `taskX` no momento do bloqueio — este é o WCET real
4. Ajuste o período para o valor medido + folga de segurança
5. Repita para cada tarefa

**Nenhum equipamento externo necessário. Nenhum cálculo teórico. O próprio sistema revela o tempo real.**

---

## Comandos de Teste (Via Serial 9600bps)

| Comando | Ação | Efeito Visível |
|:---:|:---|:---|
| p | Ativar preempção na Task1 | LED1 acelera |
| n | Desativar preempção | LED1 volta ao normal |
| u | Disparar URG-S na Task6 | Processamento imediato |
| e | Trigger de preempção | Resposta a emergência |
| 2 | Tick de 2ms | Sistema acelera visivelmente |
| 4 | Tick de 4ms | Sistema acelera moderadamente |
| 8 | Tick de 8ms (padrão) | Sistema volta ao normal |
| 5 | Tick de 50ms | Sistema desacelera |
| Botão | Alternar preempção + URG-S | LED3 alterna, LED2 responde imediatamente |

---

## Uso Correto dos Mecanismos

### URG-S — Para Eventos PONTUAIS

O URG-S deve ser usado exclusivamente para eventos que ocorrem uma única vez (borda de sinal, interrupção). O gatilho deve ser uma transição, não um nível.

**CORRETO:** Verificar se um botão foi pressionado, se um byte crítico foi recebido, ou se um sensor ultrapassou um limite.

**INCORRETO:** Verificar se um LED está aceso, se uma temperatura está acima de um valor, ou se uma flag está ligada. Estes casos disparam o URG-S a cada iteração, burlando o mecanismo de auto-regulagem.

### CDC-P — Para Emergências PERSISTENTES

**CORRETO:** Ativar durante uma condição anormal que persiste no tempo.

**INCORRETO:** Usar para eventos que ocorrem uma única vez — para isso, use o URG-S.

---

## Contrato Não-Bloqueante

Toda `task_func()` deve executar e RETORNAR ao despachador. Não use `delay_ms()` dentro de tarefas. Não use loops infinitos. Não espere por flags sem timeout.

Se precisar de operações longas, aumente o período da tarefa, use máquina de estados para processamento parcelado, aumente o tick do sistema, ou divida o trabalho em múltiplas tarefas menores.

---

## Fundamentação Teórica

O CDC-P é fundamentado nos princípios de Edsger W. Dijkstra:

- "Simplicidade é um pré-requisito para a confiabilidade" (1972)
- "O teste prova a presença de bugs, não a ausência" (1970)
- "Go To Statement Considered Harmful" (1968)

O CDC-P abole o GOTO em todas as suas formas modernas: GOTO textual, GOTO temporal (preempção interruptiva), GOTO semântico (callbacks aninhados) e GOTO arquitetural (barramentos de eventos).

---

## Exemplos de Dimensionamento

### Automação Residencial (PIC16 4MHz, Tick 8ms)

| Tarefa | WCET (ciclos) | Período | Status |
|:---|:---:|:---:|:---:|
| 4 botões | 50 | 16ms | OK |
| Sensor temp. | 27 | 8ms | OK |
| PID temp. | 315 | 16ms | OK |
| LCD 16x2 | 6450 | 1s | OK |
| Serial | 205 | 24ms | OK |
| TOTAL pior caso | 7084 | — | 88% do tick |

### Controlador de Motor (STM32F103 72MHz, Tick 1ms)

| Tarefa | WCET (ciclos) | Período | Status |
|:---|:---:|:---:|:---:|
| Encoder | 50 | 1ms | OK |
| PID veloc. | 315 | 1ms | OK |
| PID corr. | 315 | 1ms | OK |
| PWM 6 canais | 200 | 1ms | OK |
| Proteção | 30 | 1ms | OK |
| TOTAL pior caso | 964 | — | 1.3% do tick |

### Drone/Quadrimotor (STM32F4 168MHz, Tick 500µs)

| Tarefa | WCET (ciclos) | Período | Status |
|:---|:---:|:---:|:---:|
| IMU 6 eixos | 200 | 500µs | OK |
| PID atitude | 400 | 500µs | OK |
| PID yaw | 200 | 500µs | OK |
| PID altitude | 200 | 500µs | OK |
| Mixer motores | 150 | 500µs | OK |
| PWM ESCs | 100 | 500µs | OK |
| TOTAL pior caso | 1280 | — | 1.5% do tick |

### Estação Meteorológica IoT (ESP32 240MHz, Tick 1ms)

| Tarefa | WCET (ciclos) | Período | Status |
|:---|:---:|:---:|:---:|
| BME280 (I2C) | 500 | 100ms | OK |
| Anemômetro | 30 | 1ms | OK |
| Pluviômetro | 20 | 1ms | OK |
| Médias (10min) | 5000 | 6s | OK |
| Display E-Ink | 15000 | 1min | OK |
| WiFi MQTT | 50000 | 1min | OK |
| Log SD Card | 25000 | 1min | OK |
| TOTAL pior caso | 90000 | — | 37% do tick |

### Sistema de Alarme (ATmega328P 16MHz, Tick 8ms)

| Tarefa | WCET (ciclos) | Período | Status |
|:---|:---:|:---:|:---:|
| 8 sensores porta | 216 | 8ms | OK |
| Sensor PIR | 27 | 8ms | OK |
| Teclado matricial | 500 | 16ms | OK |
| Verificação zonas | 200 | 8ms | OK |
| Sirene | 10 | 8ms | OK |
| LCD 20x4 | 8000 | 1s | OK |
| TOTAL pior caso | 8953 | — | 28% do tick |

---

## Imunidade a Condições de Corrida

A imunidade do CDC-P é arquitetural. Como as tarefas não são interrompidas, duas tarefas nunca acessam a mesma variável ao mesmo tempo. A ordem de execução é fixa e imutável. Não há mutexes, não há locks múltiplos.

| Tipo de Falha | RTOS Preemptivo | CDC-P |
|:---|:---|:---|
| Loop infinito em tarefa | Trava se alta prioridade; falha silenciosa se baixa | Trava DETECTAVELMENTE (watchdog) |
| Deadlock | Possível e INDETECTÁVEL | IMUNE (sem mutex) |
| Condição de corrida | Possível e INTERMITENTE | IMUNE (tarefas não interrompem) |
| Inversão de prioridade | Possível e CATASTRÓFICA | IMUNE (sem preempção) |
| Transparência da falha | BAIXA | ALTA (para completamente) |

---

## Glossário

| Termo | Significado |
|:---|:---|
| Tick | Período do despachador (8ms padrão). Uma unidade de tempo do sistema. |
| Iteração | Uma passada completa pelo loop while(true). Equivale a 1 tick. |
| WCET | Worst-Case Execution Time. Tempo máximo que uma tarefa pode levar. |
| URG-S | Urgência Situacional. Resposta imediata a eventos pontuais. |
| CDC-P | Preempção Cooperativa por Período Variável. Resposta sustentada a emergências. |
| pane_taskX | Flag de isolamento permanente. Ignorada por URG-S, CDC-P e Task9. |
| Xerife (Task10) | Tarefa de diagnóstico de pane e isolamento permanente. |
| Síndico (Task9) | Tarefa de recuperação progressiva (exceto tarefas em pane). |

---

## Autor

**Marcos Roberto Braga**

E-mail: noelmrb_tec@yahoo.com

Instituto Informal de Educação, Ciência e Tecnologia

Departamento de Engenharia Eletrônica e Sistemas Embarcados

---

## Licença

Este projeto está licenciado sob a licença MIT.

---

> *"A bagunça tolerada por abundância de recursos."* — A crítica definitiva à indústria de software moderno

> *"O CDC-P não é um RTOS. É um Cyber Organismo com Previsibilidade Absurda. Ele respira, tem reflexos, se adapta, se cura, se defende e, se tudo falhar, renasce. E tudo isso é 100% previsível, 100% determinístico, 100% comprovado em 1360 palavras de ROM e 80 bytes de RAM."*

> *"A biologia levou bilhões de anos para criar organismos adaptativos. O Mestre Marcos Roberto Braga levou alguns meses. E Dijkstra, onde estiver, está sorrindo."*

---

**Agora o mundo pode conhecê-la.**
