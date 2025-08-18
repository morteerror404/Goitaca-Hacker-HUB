![Goitacá](foto.png)

### **GOITACÁ HACKER-HUB: O GUERREIRO DA SELVA DIGITAL**  
*(Flipper Zero? Brinquedo de playboy. HackRF? Sucata superfaturada. Nós somos o cacique do espectro!)*  

---

### **🦅 SANGUE, FERRO E FREQUÊNCIA**  
Este não é um "dispositivo". É uma **arma de guerra digital**, forjada com:  
- **Garras de SDR** (30MHz–6GHz) – CaribouLite + LMS6002D rasgando o espectro  
- **Veneno de Deauth** – ESP32 + CC1101 envenenando redes Wi-Fi  
- **Feitiço NFC/IR** – PN532 cuspindo tags clonadas como maldições  
- **Coração Aberto** – Headers expostos pra quem tem coragem de mexer  
---

### **⚔️ TABELA DA HUMILHAÇÃO**  

| **ARMAMENTO**        | **GOITACÁ HACKER-HUB**                     | **FLIPPER ZERO (Criança)**    | **HACKRF (Velho Decrépito)**  |  
|----------------------|--------------------------------------------|-------------------------------|------------------------------ |  
| **ALCANCE**          | 13MHz-6GHz                                 | Brinca no sub-GHz             | Até 6GHz mas só olhando       |  
| **PODER**            | RP2040+ESP32 (dois cérebros)               | STM32 capenga 125kHz ou NFC   | LPC4320 cansado               |  
| **LIBERDADE**        | Headers PMOD/I2C/SPI (Expansão sem limites)| Gaiola de plástico            | Sucata sem GPIO               |  
| **MAGIA NEGRA**      | NFC+IR+RF+Deauth (kit completo)            | NFC de mentirinha             | Só escuta, não ataca          |  
| **BATERIA**          | 18650 + USB-C (Caça por dias)              | Bateria de celular velho      | Cordão umbilical USB          | 
| **PREÇO**            | Custo Brasil (mas vale cada centavo)       | Preço de iPhone usada         | Vende rim pra comprar         |  

---

### **💥 NA PRÁTICA ISSO SIGNIFICA:**  
- **Passa catraca** sem pagar (Todo mundo sabe que tu não aguenta pular a roleta)  
- **Derriba Wi-Fi** alheio (igual Anatel que só derruba pirata, ou ao menos deveria)  
- **Controla tudo por IR** (até a TV da sogra)  
- **Fareja espectro** como cachorro fareja um pinguço.

---

### **🌳 MITOLOGIA HACKER**  
- **O Curupira Digital**: Pés virados confundem wardrivers.  
- **O Deus Tupã do RF**: Troveja nas frequências.
- **A Onça-Pintada do Espectro**: Ataca silencioso, mata rápido.

---

### **🩸 VEREDICTO FINAL**  
Enquanto os gringos:  
- Brincam de hacker com Flipper Zero  
- Pagam caro num HackRF que só escuta  

O **GOITACÁ HACKER-HUB** é:  
✅ **Nascido para florestas digitais**  
✅ **Alimentado a café e ódio**  
✅ **Pronto pra revolução eletromagnética**  

---  
**GOITACÁ HACKER-HUB: BIXO-PAPÃO DAS FREQUÊNCIAS.**  

*(Documentação completa e esquemáticos disponíveis pra quem tem coragem de fazer)*

# Projeto SDR Modular com Raspberry Pi Pico e ESP32-S3

Este projeto descreve um sistema **modular de rádio definido por software (SDR)** utilizando o **Raspberry Pi Pico (RP2040)** como unidade de controle principal e o **ESP32-S3** como co-processador de comunicação.

O sistema integra múltiplos transceptores (LMS6002D, AT86RF215 e PN532), permitindo cobertura desde **NFC (13.56 MHz)** até **3.8 GHz**, com módulos periféricos de interface, armazenamento e expansão.

---

## 🎯 Objetivo

Construir um **sistema SDR compacto e eficiente**, com:

* Controle centralizado no RP2040 (Raspberry Pi PICO).
* Comunicação sem fio (Wi-Fi/Bluetooth) via ESP32-S3 (Dev Kit).
* Suporte a múltiplos transceptores (NFC, sub-GHz, 2.4GHz, wideband).
* Arquitetura modular, expansível e portátil.

---

## 🧩 Estrutura Geral do Projeto

### 1. Core (Essenciais)

* **RP2040** → MCU central (coordena todos os transceptores).
* **ESP32-S3** → Rede e comunicação externa.
* **LMS6002D** → SDR de alta faixa (30 MHz – 3.8 GHz).
* **AT86RF215** → Transceptor duplo (sub-GHz + 2.4 GHz).
* **RFFC5072** → Conversor/mixer de frequência.
* **CC1101** → Sub-GHz dedicado.
* **PN532** → NFC (13.56 MHz).
* **Circuito de alimentação** (bateria, TP4056, LDOs).

### 2. Módulos (Complementares)

* **Display OLED (SSD1306 ou similar)**.
* **Buzzer piezo**.
* **Botões e encoder rotativo**.
* **Conectores SMA/u.FL para antenas externas**.
* **USB-C (alimentação, debug, dados)**.
* **Slot microSD (armazenamento)**.
* **LEDs indicadores RGB**.

### 3. Opcionais (Expansões)

* **Bateria Li-ion/LiPo + proteção**.
* **Blindagem metálica RF**.
* **Filtros SAW/cerâmicos dedicados**.
* **Antenas otimizadas por banda**.
* **GPS/GNSS para georreferenciamento**.
* **Módulos de amplificação (LNA/PA)**.

---

## ⚡ Arquitetura de Controle

```mermaid
flowchart TD
    RP2040 -->|SPI/I2C/UART| ESP32-S3
    RP2040 -->|SPI| LMS6002D
    RP2040 -->|SPI| AT86RF215
    RP2040 -->|SPI| CC1101
    RP2040 -->|I2C| PN532
    RP2040 -->|I2C| OLED
    RP2040 -->|GPIO| Buzzer
    RP2040 -->|GPIO| Botões, LEDs

    LMS6002D -->|LO| RFFC5072
    AT86RF215 --> SMA
    CC1101 --> Antena433
    PN532 --> AntenaNFC
    ESP32-S3 -->|UART/WiFi/BLE| Host
```

---

## 🔗 Tabelas de Conexões

### 1. RP2040 (Controle Central)

| Recebe...            | Conecta no...                      |
| -------------------- | ---------------------------------- |
| SPI0 (MOSI/MISO/SCK) | LMS6002D, AT86RF215, CC1101, PN532 |
| I2C0                 | OLED, PN532                        |
| UART0                | ESP32-S3                           |
| GPIOs                | Botões, LEDs, Buzzer               |

---

### 2. ESP32-S3 (Comunicação)

| Recebe...       | Conecta no...       |
| --------------- | ------------------- |
| UART0           | RP2040              |
| Wi-Fi/Bluetooth | Host externo        |
| GPIOs           | IR TX/RX (opcional) |

---

### 3. LMS6002D (SDR)

| Recebe...       | Conecta no...           |
| --------------- | ----------------------- |
| SPI             | RP2040                  |
| CLK (30.72 MHz) | Oscilador               |
| TX/RX diff.     | Antena wideband via SMA |
| LO              | RFFC5072                |

---

### 4. AT86RF215

| Recebe...   | Conecta no...                |
| ----------- | ---------------------------- |
| SPI         | RP2040                       |
| RF09 / RF24 | Matching → Antenas SMA       |
| TCXO 26 MHz | Oscilador + rede de matching |

---

### 5. CC1101

| Recebe...   | Conecta no...              |
| ----------- | -------------------------- |
| SPI         | RP2040                     |
| RF          | Antena λ/4 433 MHz + balun |
| XTAL 26 MHz | Oscilador                  |

---

### 6. PN532 (NFC)

| Recebe...      | Conecta no...       |
| -------------- | ------------------- |
| I2C ou SPI     | RP2040              |
| Antena NFC PCB | Rede LC de matching |

---

### 7. RFFC5072 (Mixer)

| Recebe...        | Conecta no...  |
| ---------------- | -------------- |
| SPI              | RP2040         |
| LO/RF/IF         | LMS6002D       |
| Filtros externos | LPF/BPF na PCB |

---

### 8. Alimentação

| Fonte        | Destino                  |
| ------------ | ------------------------ |
| USB-C 5V     | TP4056                   |
| Bateria 3.7V | LDOs (3.3V digital / RF) |
| Reguladores  | Todos os transceptores   |

---

## 📌 Resumo

* **Controle central**: RP2040 + ESP32-S3.
* **Transceptores**: LMS6002D, AT86RF215, PN532.
* **Mixer**: RFFC5072 para extensão de frequência.
* **Periféricos**: Display, buzzer, LEDs, botões, microSD.
* **Expansões**: Antenas otimizadas, blindagem RF, GPS, PA/LNA.