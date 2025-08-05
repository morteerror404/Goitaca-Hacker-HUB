# Projeto SDR Modular com Raspberry Pi Pico e ESP32

Este projeto descreve um sistema modular de rádio definido por software (SDR) utilizando o Raspberry Pi Pico (RP2040) como unidade de controle principal e o ESP32-S3 como controlador periférico. O sistema integra módulos como CaribouLite, LMS6002D, NFC, IR e display OLED, com interfaces bem definidas para comunicação e expansão.

## Objetivo
Desenvolver um sistema SDR modular, compacto e eficiente, com controle centralizado pelo RP2040, comunicação sem fio via ESP32-S3 e funcionalidades avançadas de RF, incluindo suporte a NFC e SDR completo.

---

## Estrutura Geral do Projeto

### 1. Módulos Principais
- **Raspberry Pi Pico (RP2040)**: Unidade de controle principal, responsável pela lógica geral e comunicação com periféricos via SPI, I2C e UART.
- **ESP32-S3**: Controlador periférico para comunicação sem fio (CC1101, NFC, IR).
- **CaribouLite Clone**: Módulo SDR com FPGA iCE40LP1K, AT86RF215 e RFFC5072 para operação de 30 MHz a 6 GHz.
- **LMS6002D**: Transceptor RF completo, controlado via SPI pelo RP2040.
- **Display OLED (SSD1306)**: Exibição de status e dados do sistema via I2C.
- **I/O Geral**: Botões, LEDs, buzzer e headers para expansão (I2C, UART, SPI, PMOD, SMA).

### 2. Alimentação
- **Entrada**: Micro USB 5V.
- **Carregador**: TP4056 com proteção.
- **Bateria**: 2x 18650 em paralelo.
- **Reguladores**:
  - AMS1117 (3.3V) para RP2040.
  - LDOs dedicados para ESP32, OLED e LMS6002D.

**Nota**: A tensão nominal de 3.7V das baterias pode ser insuficiente para reguladores lineares. Recomenda-se incluir um conversor boost ou usar LDOs de baixa queda.

### 3. Arquitetura de Controle
```
RP2040 (Master MCU)
│
├── SPI/I2C/UART --> ESP32-S3 (Slave Periférico)
│   ├── SPI --> CC1101 (433 MHz)
│   ├── I2C --> PN532 (NFC)
│   └── GPIO --> IR TX/RX
├── SPI --> FPGA (CaribouLite)
│   └── SPI/CTRL --> AT86RF215 / RFFC5072
├── SPI --> LMS6002D
├── I2C --> SSD1306 OLED
├── GPIO --> LEDs, Buzzer, Botões
└── Headers: UART, I2C, SPI, PMOD, LMS
```

---

### 1. **RP2040 (Controle Principal)**
- **Função**: Serve como o microcontrolador central que gerencia a comunicação e a inicialização dos outros módulos. Controla os chips selecionados (CS) para CC1101, PN532, AT86RF215 e LMS6002D via GPIOs e gerencia a interface SPI (MOSI, MISO, SCK).
- **Detalhes**: Coordena a sequência de inicialização e sincroniza com o ESP32 via UART.

### 2. **ESP32-S3 (Comunicação)**
- **Função**: Responsável pela comunicação externa, incluindo transmissão (IR_TX) e recepção (IR_RX) via infravermelho, além de sincronização com o RP2040 via UART0.
- **Detalhes**: Atua como uma interface de comunicação adicional, possivelmente para redes ou controle remoto.

### 3. **FPGA iCE40LP1K (Controle SDR)**
- **Função**: Controla o módulo SDR (LMS6002D e AT86RF215) via SPI e gerencia o RFFC5072 (sintetizador de frequência) através de GPIOs (ENABLE e SLEEP).
- **Detalhes**: Processa sinais de radiofrequência e suporta operações de software-defined radio (SDR).

### 4. **CC1101**
- **Função**: Módulo de rádio transceptor de baixa potência, operando na faixa de 433MHz, com interface SPI para o RP2040.
- **Detalhes**: Utiliza uma antena λ/4 de 433MHz e um balun (T1) para matching, ideal para comunicações de curto alcance.

### 5. **PN532**
- **Função**: Controlador NFC/RFID, suportando comunicação via SPI, I2C ou UART com o RP2040, operando a 13.56MHz.
- **Detalhes**: Conectado a uma antena PCB com componentes de ajuste (L1, C1, C2) para leitura/escrita de tags NFC.

### 6. **AT86RF215**
- **Função**: Transceptor de rádio multibanda, operando nas faixas de 900MHz e 2.4GHz, com interface SPI para RP2040 e FPGA.
- **Detalhes**: Possui redes de matching (MATCH09, MATCH24) e antenas SMA dedicadas para cada faixa, usado em comunicações de longo alcance.

### 7. **LMS6002D**
- **Função**: Chip SDR de alta performance, gerenciado via SPI pelo RP2040, com interfaces diferenciais RX/TX.
- **Detalhes**: Suporta uma ampla faixa de frequências com uma antena wideband SMA, ideal para aplicações de rádio definidas por software.

### 8. **RFFC5072**
- **Função**: Sintetizador de frequência controlado via SPI pelo MCU, usado para gerar sinais LO, RF e IF com filtros (LPF, BPF).
- **Detalhes**: Integra-se ao sistema SDR via FPGA, ajustando frequências para o AT86RF215 e LMS6002D.

### 9. **Alimentação (TP4056, AMS1117-3.3V, RT9193-3.3V)**
- **Função**: Gerencia a alimentação do sistema, convertendo 5V do USB via TP4056 para carregar a bateria LiPo 3.7V, e regulando 3.3V para lógica (AMS1117) e RF/SDR (RT9193).
- **Detalhes**: Fornece energia estável com capacitores de desacoplamento (100nF, 10µF) para todos os módulos.

### Resumo
- **Controle**: RP2040 (central) e ESP32 (comunicação externa).
- **RF/NFC**: CC1101 (433MHz), PN532 (13.56MHz), AT86RF215 (900MHz/2.4GHz), LMS6002D (wideband).
- **SDR**: FPGA e RFFC5072 (processamento e síntese de frequência).
- **Alimentação**: TP4056, AMS1117, RT9193 (gerenciamento de energia).

Essa separação reflete as responsabilidades distintas de cada módulo no sistema descrito.

Com base nos diagramas fornecidos e na separação das funções, implementei as conexões de cada componente em uma estrutura de tabela com duas colunas essenciais: **"Recebe..."** (o que o componente recebe) e **"Conecta no..."** (aonde ele se conecta). Abaixo estão as tabelas para cada módulo, refletindo as conexões descritas nos diagramas Mermaid.

---

### 1. **RP2040 (Controle Principal)**
| Recebe...             | Conecta no...            |
|-----------------------|--------------------------|
| GPIO0                 | CS_CC1101                |
| GPIO1                 | CS_PN532                 |
| GPIO2                 | CS_AT86RF215             |
| GPIO3                 | CS_LMS6002D              |
| SPI0 (MOSI)           | CC1101, PN532, AT86RF215, LMS6002D |
| SPI0 (MISO)           | CC1101, PN532, AT86RF215, LMS6002D |
| SPI0 (SCK)            | CC1101, PN532, AT86RF215, LMS6002D |
| UART0_RX              | ESP32 (UART0_TX)         |

---

### 2. **ESP32-S3 (Comunicação)**
| Recebe...             | Conecta no...            |
|-----------------------|--------------------------|
| GPIO4                 | IR_TX                    |
| GPIO5                 | IR_RX                    |
| UART0_TX              | RP2040 (UART0_RX)        |

---

### 3. **FPGA iCE40LP1K (Controle SDR)**
| Recebe...             | Conecta no...            |
|-----------------------|--------------------------|
| SPI                   | AT86RF215                |
| GPIO8                 | RFFC5072 (ENABLE)        |
| GPIO9                 | RFFC5072 (SLEEP)         |

---

### 4. **CC1101**
| Recebe...             | Conecta no...            |
|-----------------------|--------------------------|
| SPI                   | RP2040                   |
| RF_N/RF_P             | BALUN (T1)               |
| VCC (3.3V)            | C7 (100nF), C8 (10µF)    |
| XTAL (26MHz)          | C5 (12pF), C6 (12pF)     |

---

### 5. **PN532**
| Recebe...             | Conecta no...            |
|-----------------------|--------------------------|
| SPI                   | RP2040                   |
| I2C                   | RP2040                   |
| UART                  | RP2040                   |
| Antena PCB (13.56MHz) | L1 (1.5µH), C1 (27pF), C2 (27pF) |
| VDD (3.3V)            | C3 (100nF), C4 (10µF)    |

---

### 6. **AT86RF215**
| Recebe...             | Conecta no...            |
|-----------------------|--------------------------|
| SPI                   | RP2040, FPGA             |
| RF09_N/RF09_P         | MATCH09 (L2 15nH, C13 1.8pF) |
| RF24_N/RF24_P         | MATCH24 (L3 5.6nH, C14 1.2pF) |
| TCXO (26MHz)          | C11 (12pF), C12 (12pF)   |

---

### 7. **LMS6002D**
| Recebe...             | Conecta no...            |
|-----------------------|--------------------------|
| SPI                   | RP2040                   |
| CLK (30.72MHz)        | C15 (10pF), C16 (10pF)   |
| RX (Differential)     | -                        |
| TX (Differential)     | -                        |
| VDD (3.3V)            | C17 (100nF), C18 (10µF)  |

---

### 8. **RFFC5072**
| Recebe...             | Conecta no...            |
|-----------------------|--------------------------|
| SPI                   | MCU                      |
| LO (50Ω matched)      | FL1 (LPF 7th Order)      |
| RF (50Ω matched)      | FL2 (BPF)                |
| IF (50Ω matched)      | FL3 (LPF 5th Order)      |
| VDD (3.3V)            | C9 (100nF), C10 (1µF)    |
| ENABLE                | FPGA (GPIO8)             |
| SLEEP                 | FPGA (GPIO9)             |

---

### 9. **Alimentação (TP4056, AMS1117-3.3V, RT9193-3.3V)**
| Recebe...             | Conecta no...            |
|-----------------------|--------------------------|
| Micro USB 5V          | TP4056                   |
| TP4056                | Bateria (LiPo 3.7V)      |
| Bateria (3.7V)        | AMS1117-3.3V (Lógica)    |
| Bateria (3.7V)        | RT9193-3.3V (RF)         |
| Bateria (3.7V)        | RT9193-3.3V (SDR)        |

---

### Notas
- As conexões de terra (GND) foram omitidas nas tabelas por serem implícitas nos capacitores e outros componentes, mas estão presentes nos diagramas corrigidos.
- Para RX/TX diferenciais no LMS6002D, não há conexão específica listada, pois são interfaces externas que dependem do design do sistema.
- "MCU" em RFFC5072 refere-se a um microcontrolador genérico (pode ser RP2040 ou outro, dependendo do contexto).

