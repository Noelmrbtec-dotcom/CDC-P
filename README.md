# CDC-P v3.0 - Concurrency Deterministic Control with Preemption

## Um RTOS determinístico com prioridade tridimensional, auto-regulagem temporal, diagnóstico de pane e imunidade a condições de corrida. Tudo em 1188 palavras de ROM e 78 bytes de RAM.

[![Platform](https://img.shields.io/badge/platform-PIC16F628A-blue)](https://www.microchip.com/en-us/product/PIC16F628A)
[![ROM](https://img.shields.io/badge/ROM-1188%20words%20(58%25)-green)]()
[![RAM](https://img.shields.io/badge/RAM-78%20bytes%20(35%25)-brightgreen)]()
[![Stack](https://img.shields.io/badge/Stack-4%20of%208%20levels-orange)]()
[![Overhead](https://img.shields.io/badge/Overhead-112%C2%B5s%20(1.4%25)-blue)]()
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

---

## Visão Geral

O CDC-P (Concurrency Deterministic Control with Preemption) é um RTOS (Real-Time Operating System) guiado por tempo (Time-Triggered) projetado para sistemas embarcados de recursos restritos. Ele prova que é possível obter concorrência real, resposta a eventos urgentes e tolerância a falhas sem sacrificar o determinismo ou a eficiência.

Diferentemente de RTOS preemptivos tradicionais como FreeRTOS e ThreadX, o CDC-P não usa preempção interruptiva, que é a fonte primária de condições de corrida e deadlocks. Em vez disso, utiliza preempção cooperativa por período variável (CDC-P) e urgência situacional por flags binárias (URG-S). O sistema é imune a condições de corrida, deadlocks e inversão de prioridade por construção arquitetural, não por mecanismos de correção posteriores.

O CDC-P é autoconsciente: diagnostica pane, isola tarefas problemáticas e sinaliza para hardware externo quando atinge um estado crítico. Tudo isso cabendo em 1188 palavras de ROM e 78 bytes de RAM em um microcontrolador PIC16F628A de 8 bits com clock de 4 MHz.

> *"A simplicidade é um pré-requisito para a confiabilidade."* — Edsger W. Dijkstra

---

## Características Principais

### Prioridade Tridimensional

Nenhum RTOS comercial oferece isso. A prioridade no CDC-P é uma matriz de três dimensões que o programador deve dominar.

A primeira dimensão é a Espacial, que é estática e definida em tempo de projeto. Ela controla a posição fixa de cada tarefa no despachador. A Task10, por exemplo, sempre executa antes da Task1 porque está posicionada no topo da fila.

A segunda dimensão é a Temporal, que é configurável e pode ser alterada em tempo de execução. Ela controla o período de cada tarefa através da variável taskX. Uma tarefa com task1=2 executa a cada 2 ticks, ou seja, a cada 16ms com o tick padrão de 8ms.

A terceira dimensão é a Situacional, que é dinâmica e acionada por eventos. Ela utiliza flags binárias de urgência (URG-S) que permitem que qualquer ISR ou tarefa sinalize que outra tarefa precisa executar imediatamente na mesma iteração do despachador, independentemente de sua posição ou período configurado.

| Dimensão | Tipo | O que controla | Exemplo |
|:---|:---|:---|:---|
| 1. Espacial | Estática (projeto) | Posição fixa no despachador | Task10 antes de Task1 |
| 2. Temporal | Configurável | Período da tarefa (taskX) | task1=2 (16ms) |
| 3. Situacional | Dinâmica (eventos) | Flag de urgência (URG-S) | Botão pressionado -> resposta imediata |

### Mecanismos de Resposta a Eventos

O CDC-P oferece três níveis de resposta a eventos urgentes, cada um com sua latência característica e uso apropriado.

O URG-S (Urgência Situacional) é o mecanismo mais rápido. Ele responde na mesma iteração do despachador, com zero ticks de latência. A duração é de uma única execução, após a qual a flag é automaticamente limpa. É ideal para eventos pontuais como "Acorda! Algo aconteceu AGORA!" — por exemplo, um botão pressionado ou um byte crítico recebido na serial.

O CDC-P (Preempção Cooperativa por Período Variável) responde na próxima iteração do despachador, com 1 tick de latência. A duração persiste até que seja explicitamente desativado. É ideal para emergências persistentes como "Fique em alerta!" — por exemplo, uma condição de sobrecarga que dura vários ciclos.

O Tick Dinâmico é o mecanismo mais abrangente. Ele altera a frequência de todo o sistema imediatamente e persiste até ser revertido. É ideal para situações de sobrecarga sistêmica como "Todos acelerem!" — por exemplo, quando o buffer serial está cheio e precisa ser processado rapidamente.

| Mecanismo | Latência | Duração | Uso |
|:---|:---|:---|:---|
| URG-S | Mesma iteração (0 ticks) | Única execução | Eventos pontuais: "Acorda! Algo aconteceu AGORA!" |
| CDC-P | Próxima iteração (1 tick) | Persiste até desativar | Emergências persistentes: "Fique em alerta!" |
| Tick Dinâmico | Imediato | Persiste até reverter | Sobrecarga sistêmica: "Todos acelerem!" |

### Urgência Situacional (URG-S)

O URG-S implementa resposta imediata a eventos pontuais usando flags binárias. Uma flag é um único bit em uma variável de 16 bits (flags_urgencia), onde cada bit corresponde a uma tarefa. Quando uma ISR ou tarefa detecta um evento urgente, seta a flag correspondente. No início da próxima iteração do despachador, o bloco URG-S verifica cada flag e executa a tarefa correspondente antes de qualquer outra, limpando a flag em seguida.

O overhead adicional do URG-S é de apenas 12 a 32 microssegundos em relação à versão sem o mecanismo. Isso representa entre 0.15% e 0.40% do tick de 8ms.

Para chamadas dentro de ISRs, utiliza-se a função set_urgent_isr(), que já está em contexto de interrupção e não precisa desabilitar interrupções. Para chamadas dentro de tarefas normais, utiliza-se a função set_urgent(), que protege contra re-entrância desabilitando interrupções durante a operação.

É fundamental entender que o URG-S deve ser usado apenas para eventos pontuais — coisas que acontecem uma única vez, como a borda de um sinal ou uma interrupção. Usar o URG-S com condições contínuas, como verificar se um LED está aceso a cada iteração, faria a flag ser setada repetidamente, burlando o mecanismo de auto-regulagem e distorcendo o período natural da tarefa.

### Preempção Cooperativa (CDC-P)

O CDC-P implementa um mecanismo de preempção que não interrompe o fluxo de controle. Quando uma tarefa é designada como preemptiva via CDC_EnablePreempt(), seu período é reduzido para 1 tick. A tarefa passa a ser verificada em todas as iterações subsequentes do despachador, mas sempre em sua posição fixa na fila.

Não há interrupção de outras tarefas. Não há salvamento e restauração de contexto. Não há condições de corrida. O overhead é zero — a preempção custa exatamente uma atribuição de variável para reduzir o período e outra para restaurá-lo. A ativação é feita com CDC_EnablePreempt(numero_da_tarefa) e a desativação com CDC_DisablePreempt().

### 7 Camadas de Proteção

O CDC-P implementa sete camadas de proteção que atuam em ordem, desde a mais branda (auto-regulagem) até a mais drástica (reset do sistema).

A primeira camada é a auto-regulagem temporal. Cada tarefa detecta seu próprio atraso e aumenta seu período para reduzir a carga do sistema. É um mecanismo de adaptação que previne sobrecargas antes que elas se tornem críticas.

A segunda camada é o bloqueio por violação de deadline. Se o período de uma tarefa atinge um limite máximo configurável, a tarefa é isolada para proteger o sistema. O semáforo sema_taskX é setado para 1, impedindo que a tarefa execute.

A terceira camada é o diagnóstico de pane, executado pela Task10, também chamada de Xerife. Ela monitora os contadores de reincidência de bloqueio e, quando qualquer tarefa atinge o limite configurado (sistema_em_pane, tipicamente 5), decreta pane operacional, sinaliza para hardware externo através do pino monitor_tasks e isola permanentemente a tarefa problemática.

A quarta camada é a recuperação progressiva, executada pela Task9, também chamada de Síndico. Ela verifica todas as tarefas bloqueadas, reduz seus períodos em uma unidade e as desbloqueia, dando uma segunda chance. Se a própria Task9 falhar, a quinta camada entra em ação.

A quinta camada é o fail-safe final. Se a Task9 exceder seu limite de tolerância, o sistema executa um reset completo. É a última linha de defesa: se o médico ficar doente, o paciente é colocado em coma induzido para evitar um mal maior.

A sexta camada é a aceleração automática, implementada pela Task8. Durante a preempção ativa, o tick do sistema é reduzido de 8ms para 2ms, fazendo o sistema hiperventilar na urgência e retornar à respiração normal quando a crise passa.

A sétima camada é o próprio URG-S, que oferece resposta imediata a eventos urgentes com latência de mesma iteração do despachador.

| Camada | Mecanismo | Função |
|:---:|:---|:---|
| 1 | Auto-regulagem temporal | Cada tarefa se desacelera se atrasar |
| 2 | Bloqueio por deadline | Isolamento de tarefas que violam limites |
| 3 | Diagnóstico (Task10 - Xerife) | Detecta pane e ISOLA tarefas problemáticas |
| 4 | Recuperação (Task9 - Síndico) | Reabilita tarefas bloqueadas progressivamente |
| 5 | Fail-safe final | Reset se o mecanismo de recuperação falhar |
| 6 | Aceleração automática (Task8) | Tick reduz de 8ms para 2ms na urgência |
| 7 | URG-S | Resposta IMEDIATA a eventos urgentes |

### Tick Dinâmico em Tempo de Execução

O CDC-P permite alterar o tick do sistema em tempo de execução, sem reinicialização ou perda de estado. A função ajustar_tick() reconfigura o Timer0 e a constante _segundo. A estrutura do código não muda — o despachador, as tarefas, os semáforos, a Task9, a Task10, tudo permanece idêntico.

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

O despachador do CDC-P consome apenas 112 microssegundos para percorrer todo o loop quando nenhuma tarefa está pronta para executar. Isso significa que 98.6% do tempo de CPU está disponível para as tarefas da aplicação. Em um tick de 8ms (8000 microssegundos), sobram 7888 microssegundos livres.

| Recurso | Usado | Livre |
|:---|:---:|:---:|
| ROM | 1188 words (58%) | 860 words (42%) |
| RAM (main) | 78 bytes (35%) | 146 bytes (65%) |
| RAM (pior caso) | 84 bytes (38%) | 140 bytes (62%) |
| Stack | 4 níveis (50%) | 4 níveis (50%) |
| Overhead por tick | 112 microssegundos | 7888 microssegundos livres |

### Comparação com RTOS Preemptivos (Estimativas)

| Métrica | CDC-P (PIC16 4MHz) | FreeRTOS (ARM Cortex-M) | Vantagem CDC-P |
|:---|:---:|:---:|:---:|
| ROM (10 tarefas) | 1.6 KB | ~10-15 KB | 6-10x menos |
| RAM (10 tarefas) | 78 bytes | ~2000-3000 bytes | 25-40x menos |
| Stack (10 tarefas) | 4 níveis | ~2000+ bytes | ~500x menos |
| Prioridade | Tridimensional | Unidimensional | Mais expressiva |
| Diagnóstico de pane | Nativo (Task10) | Inexistente | Presente |
| Resposta a eventos | Imediata (URG-S) | Via kernel (latência) | Mais rápido |
| Condições de corrida | IMUNE | Possíveis | 100% seguro |

### Estimativas em Outras Plataformas

| Plataforma | Clock | Ganho vs PIC16 | Tick Mínimo |
|:---|:---:|:---:|:---:|
| ATmega328P (Arduino Uno) | 16 MHz | 4x | 1ms |
| STM32F103 (Blue Pill) | 72 MHz | 60-70x | 50 microssegundos |
| RP2040 (Raspberry Pi Pico) | 133 MHz | 120x | 50 microssegundos |
| CH32V003 (RISC-V) | 48 MHz | 48x | 100 microssegundos |
| CH32V307 (RISC-V) | 144 MHz | 144x | 50 microssegundos |
| ESP32 (Xtensa LX6) | 240 MHz | 200-240x | 10 microssegundos |

Em todas as plataformas, o CDC-P ocupa menos de 5% dos recursos disponíveis.

---

## Estrutura do Sistema

### Ordem do Despachador (Uma Iteração = 1 Tick)

O despachador do CDC-P é um loop while(true) que executa em uma ordem fixa e imutável. A cada iteração, que corresponde a um tick (8ms padrão), o despachador percorre quatro blocos hierárquicos.

O primeiro bloco é o URG-S (Urgência Situacional). Ele verifica se alguma flag de urgência foi setada por uma ISR ou tarefa. Se houver flags setadas, executa as tarefas correspondentes imediatamente, antes de qualquer outra coisa. A ordem de verificação das flags segue a mesma ordem fixa do despachador: urg_task1 é verificado antes de urg_task2, e assim por diante.

O segundo bloco é o CDC-P (Preempção Temporal). Ele verifica se há uma tarefa em modo preemptivo que foi acionada por um evento externo. Se houver, executa essa tarefa imediatamente após o bloco URG-S.

O terceiro bloco é a Camada de Supervisão, que é imutável e nunca deve ser alterada pelo usuário. Contém a Task10 (Xerife), responsável pelo diagnóstico de pane e isolamento de tarefas problemáticas, e a Task9 (Síndico), responsável pela recuperação progressiva de tarefas bloqueadas.

O quarto bloco é a Camada de Aplicação, que é adaptável e deve ser modificada pelo usuário para cada projeto. Contém as Tasks 1 a 8, que executam a lógica da aplicação em ordem fixa.

### As 10 Tarefas

| Tarefa | Nome | Função | Período | Bloqueio |
|:---|:---|:---|:---:|:---:|
| Task10 | Xerife | Diagnóstico de pane e isolamento | 1 tick | NUNCA |
| Task9 | Síndico | Recuperação de tarefas bloqueadas | 1 tick | Reset |
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
2. Defina as tolerâncias de deadline (tempo_maximo_taskX)
3. Escreva a lógica de cada taskX_func()
4. Configure os pinos de I/O em config_io()
5. Escolha o tick padrão (2ms, 4ms, 8ms ou 50ms)
6. Compile e grave no microcontrolador

### Modelo Padrão de Tarefa

Toda tarefa no CDC-P deve seguir este modelo. As três linhas finais (cálculo de jitter, auto-regulagem e bloqueio) são obrigatórias e nunca devem ser removidas. A lógica da aplicação deve ser inserida no início da função, e nunca deve conter delay_ms() ou loops infinitos.

O cálculo de jitter (timer_ex_task1) mede quantos períodos a tarefa atrasou. A auto-regulagem (if timer_ex_task1) aumenta o período para reduzir a carga do sistema. O bloqueio (if task1 >= tempo_maximo_task1) isola a tarefa se o atraso persistir além do limite configurado.

---

## Regras de Dimensionamento Temporal

### Fórmula do Tempo Máximo por Tarefa

O tempo máximo que cada tarefa pode consumir é calculado dividindo o tempo livre do tick pelo número de tarefas que podem executar simultaneamente naquele tick. O overhead do despachador (112 microssegundos) é subtraído do tick total.

A fórmula é: Tempo máximo por tarefa em microssegundos = (Tick em microssegundos - 112) / N, onde Tick em microssegundos é 8000 para o tick padrão de 8ms, 112 é o overhead do despachador em microssegundos, e N é o número de tarefas que executam no mesmo tick.

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

A fórmula é: Ciclos por tick = (Clock_Hz / 4) x (Tick_ms / 1000). Para PIC16, 1 ciclo de instrução = 4 ciclos de clock (Fosc/4). Para ARM/RISC-V, 1 ciclo de instrução = 1 ciclo de clock.

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

Esta tabela ajuda a decidir se uma tarefa cabe em um determinado tick. VERDE significa que cabe com folga e pode usar taskX=1. AMARELO significa que cabe, mas ocupa parte significativa do tick e requer cuidado com outras tarefas. VERMELHO significa que não cabe e requer aumento do período, do tick, ou divisão da tarefa.

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

O URG-S deve ser usado exclusivamente para eventos que ocorrem uma única vez, como a borda de um sinal, uma interrupção de hardware, ou uma condição excepcional que muda de estado. O gatilho deve ser uma transição, não um nível.

Exemplos de uso correto: verificar se um botão foi pressionado (transição de não pressionado para pressionado), se um byte crítico foi recebido na serial, ou se um sensor ultrapassou um limite (transição de normal para alarme).

Exemplos de uso incorreto: verificar se um LED está aceso (condição contínua), se uma temperatura está acima de um valor (condição contínua), ou se uma flag está ligada (condição contínua). Estes casos disparam o URG-S a cada iteração, burlando o mecanismo de auto-regulagem.

### CDC-P — Para Emergências PERSISTENTES

O CDC-P deve ser usado para condições que persistem no tempo e exigem que uma tarefa específica execute com frequência máxima enquanto a condição durar. Quando a condição cessa, a preempção deve ser desativada.

Exemplo de uso correto: ativar durante uma condição anormal que persiste no tempo, como uma sobrecarga detectada ou um modo de emergência acionado por hardware.

Exemplo de uso incorreto: usar para eventos que ocorrem uma única vez — para isso, use o URG-S.

---

## Contrato Não-Bloqueante

O CDC-P opera sob um contrato fundamental: toda task_func() deve executar sua lógica e retornar ao despachador. Se o programador inserir funções bloqueantes como delay_ms(), loops infinitos ou espera por flags sem timeout, estará violando este contrato. Nestes casos, o sistema trava — não por falha do CDC-P, mas por violação das regras fundamentais de uso.

Em um sistema corretamente projetado segundo os princípios do CDC-P, cada tarefa é um circuito combinacional que recebe entradas, produz saídas e termina. Esta disciplina de projeto é o que garante o determinismo absoluto.

Não use delay_ms() dentro de tarefas. Não use loops infinitos while(1). Não espere por flags sem timeout. Se precisar de operações longas, aumente o período da tarefa (taskX maior), use máquina de estados para processamento parcelado, aumente o tick do sistema (50ms), ou divida o trabalho em múltiplas tarefas menores.

---

## Fundamentação Teorica

O CDC-P é fundamentado nos princípios de Edsger W. Dijkstra, um dos fundadores da ciência da computação moderna. Seus três princípios mais importantes estão incorporados na arquitetura do sistema.

O primeiro princípio, "Simplicidade é um pré-requisito para a confiabilidade" (1972), está encarnado no kernel do CDC-P, que é um loop com ifs, sem estruturas de dados complexas, sem alocação dinâmica e sem recursão.

O segundo princípio, "O teste prova a presença de bugs, não a ausência" (1970), é atendido pelo determinismo absoluto da ordem de execução, que permite a reprodução exata de qualquer falha. O que falhou uma vez falhará sempre da mesma forma, tornando a depuração trivial.

O terceiro princípio, "Go To Statement Considered Harmful" (1968), é levado às últimas consequências. O CDC-P abole o GOTO em todas as suas formas modernas: o GOTO textual (código espaguete), o GOTO temporal (preempção interruptiva), o GOTO semântico (callbacks aninhados) e o GOTO arquitetural (barramentos de eventos pub/sub).

---

## Exemplos de Dimensionamento

### Automação Residencial (PIC16 4MHz, Tick 8ms)

Um sistema típico de automação residencial com leitura de sensores, controle PID, display LCD e comunicação serial. O pior caso ocorre quando o LCD é atualizado no mesmo tick que as outras tarefas, consumindo 88% do tick. A gravação em EEPROM deve ser parcelada para caber no orçamento de tempo.

| Tarefa | WCET (ciclos) | Período | Status |
|:---|:---:|:---:|:---:|
| 4 botões | 50 | 16ms | OK |
| Sensor temp. | 27 | 8ms | OK |
| PID temp. | 315 | 16ms | OK |
| LCD 16x2 | 6450 | 1s | OK |
| Serial | 205 | 24ms | OK |
| TOTAL pior caso | 7084 | — | 88% do tick |

### Controlador de Motor (STM32F103 72MHz, Tick 1ms)

Um controlador de motor com leitura de encoder, duas malhas PID (velocidade e corrente), geração de PWM e proteção contra sobrecorrente. Com clock de 72MHz, o sistema tem 72000 ciclos por tick, e o pior caso consome apenas 1.3% desse tempo.

| Tarefa | WCET (ciclos) | Período | Status |
|:---|:---:|:---:|:---:|
| Encoder | 50 | 1ms | OK |
| PID veloc. | 315 | 1ms | OK |
| PID corr. | 315 | 1ms | OK |
| PWM 6 canais | 200 | 1ms | OK |
| Proteção | 30 | 1ms | OK |
| TOTAL pior caso | 964 | — | 1.3% do tick |

### Drone/Quadrimotor (STM32F4 168MHz, Tick 500 microssegundos)

Um sistema de controle de voo com leitura de IMU de 6 eixos, três malhas PID (atitude, yaw e altitude), mixer de motores e geração de PWM para ESCs. Com tick de 500 microssegundos, a taxa de atualização é de 2000 Hz, e o pior caso consome apenas 1.5% do tick.

| Tarefa | WCET (ciclos) | Período | Status |
|:---|:---:|:---:|:---:|
| IMU 6 eixos | 200 | 500 microssegundos | OK |
| PID atitude | 400 | 500 microssegundos | OK |
| PID yaw | 200 | 500 microssegundos | OK |
| PID altitude | 200 | 500 microssegundos | OK |
| Mixer motores | 150 | 500 microssegundos | OK |
| PWM ESCs | 100 | 500 microssegundos | OK |
| TOTAL pior caso | 1280 | — | 1.5% do tick |

### Estação Meteorológica IoT (ESP32 240MHz, Tick 1ms)

Uma estação meteorológica com sensores I2C, anemômetro, pluviômetro, cálculo de médias, display E-Ink, envio de dados por WiFi via MQTT e logging em cartão SD. O pior caso ocorre uma vez por minuto quando WiFi, SD e display executam juntos, consumindo 37% do tick.

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

Um sistema de alarme com 8 sensores de porta, sensor PIR, teclado matricial, verificação de zonas, controle de sirene e display LCD 20x4. O pior caso ocorre quando o LCD é atualizado, consumindo 28% do tick.

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

A imunidade do CDC-P a condições de corrida não é acidental — é arquitetural. Como as tarefas não são interrompidas, duas tarefas nunca acessam a mesma variável ao mesmo tempo. A ordem de execução é fixa e imutável. Não há mutexes, não há locks múltiplos, não há como duas tarefas ficarem esperando uma pela outra.

Em um RTOS preemptivo, um loop infinito em uma tarefa de alta prioridade trava o sistema porque o kernel sempre entrega o processador à tarefa pronta de maior prioridade. No CDC-P, um loop infinito trava o sistema de forma detectável: a Task9 e a Task10 não executam, e o watchdog externo reseta o sistema. É uma falha estrondosamente visível, não uma falha silenciosa e intermitente.

| Tipo de Falha | RTOS Preemptivo | CDC-P |
|:---|:---|:---|
| Loop infinito em tarefa | Trava se alta prioridade; falha silenciosa se baixa | Trava DETECTAVELMENTE (watchdog) |
| Deadlock | Possível e INDETECTÁVEL pelo kernel | IMUNE (sem mutex) |
| Condição de corrida | Possível e INTERMITENTE | IMUNE (tarefas não interrompem) |
| Inversão de prioridade | Possível e CATASTRÓFICA | IMUNE (sem preempção) |
| Transparência da falha | BAIXA (sistema "quase funciona") | ALTA (para completamente) |

---

## Glossário de Termos

| Termo | Significado |
|:---|:---|
| Tick | Período do despachador (8ms padrão). Uma unidade de tempo do sistema. |
| Iteração | Uma passada completa pelo loop while(true). Equivale a 1 tick. |
| Ciclo de clock | Período do oscilador do MCU (250ns a 4MHz). |
| Ciclo de instrução | Tempo de 1 instrução assembly (1 microssegundo a 4MHz no PIC16). |
| Período (taskX) | Número de ticks entre inícios de execuções consecutivas da tarefa. |
| WCET | Worst-Case Execution Time. Tempo máximo que uma tarefa pode levar. |
| URG-S | Urgência Situacional. Resposta imediata a eventos pontuais. |
| CDC-P | Preempção Cooperativa por Período Variável. Resposta sustentada a emergências. |
| Xerife (Task10) | Tarefa de diagnóstico de pane e isolamento de tarefas problemáticas. |
| Síndico (Task9) | Tarefa de recuperação progressiva de tarefas bloqueadas. |

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

> *"252 words e 3 bytes. Esse é o preço da terceira dimensão. O URG-S não é um luxo — é uma necessidade que coube no troco do pão."*

> *"112 microssegundos. É isso que custa a ordem no universo do CDC-P. Os outros 7888 microssegundos são seus para fazer o que quiser. O CDC-P cobra barato pela paz de espírito."*

---

**Agora o mundo pode conhecê-lo.**
