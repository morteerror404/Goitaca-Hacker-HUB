Perfeito! Vou gerar **uma lista detalhada de componentes para EasyEDA**, organizada por módulo, e em seguida um **guia de conexões**, para que você possa montar o PCB ou protoboard sem confusão.

---

## **Lista de Componentes (EasyEDA)**

### 1️⃣ **Alimentação 3.3V**

| Componente      | Valor       | Encapsulamento / EasyEDA |
| --------------- | ----------- | ------------------------ |
| Capacitor       | 100nF       | 0402                     |
| Capacitor       | 1µF         | 0603                     |
| Capacitor       | 10µF        | 0805                     |
| LDO Regulador   | 3.3V, 500mA | SOT-223 / LM1117-3.3     |
| Barramento 3.3V | -           | Net / Power Rail         |

---

### 2️⃣ **MCU RP2040**

| Componente | Detalhe          | Encapsulamento    |
| ---------- | ---------------- | ----------------- |
| RP2040     | Microcontrolador | QFN-56            |
| LEDs       | Status RF        | 0805              |
| Resistores | 330Ω para LEDs   | 0603              |
| Botão      | User Switch      | THT / SMD pequeno |

---

### 3️⃣ **MCU ESP32-S3 DevKit**

| Componente      | Detalhe      | Encapsulamento |
| --------------- | ------------ | -------------- |
| ESP32-S3 DevKit | UART/SPI/ADC | Módulo DevKit  |
| Capacitores     | 100nF, 1µF   | 0603 / 0805    |

---

### 4️⃣ **RFFC5072 Mixer / PLL**

| Componente      | Detalhe        | Encapsulamento |
| --------------- | -------------- | -------------- |
| RFFC5072        | Mixer/PLL      | 32-QFN         |
| Capacitor       | 100nF          | 0402           |
| Capacitor       | 1µF            | 0603           |
| Capacitor XTAL  | 22pF           | 0402           |
| Resistor Pullup | 10kΩ           | 0603           |
| Filtro LPF      | 7th Order SAW  | FL1            |
| Filtro BPF      | SAW 30MHz-6GHz | FL2            |
| Filtro LPF      | 5th Order      | FL3            |

---

### 5️⃣ **LO / Oscilador Local**

| Componente    | Detalhe        | Encapsulamento |
| ------------- | -------------- | -------------- |
| PLL / VCO     | 1.5-4GHz       | Módulo SMT     |
| Capacitor     | 100pF DC Block | 0402           |
| Indutor       | 15nH           | 0402           |
| Capacitor GND | 1pF            | 0402           |
| Filtro SAW    | LPF            | SF2149E        |

---

### 6️⃣ **RF Front-Ends**

| Componente         | Detalhe               | Encapsulamento        |
| ------------------ | --------------------- | --------------------- |
| Antena             | Sub-1GHz / 30MHz-6GHz | PCB Trace / SMA       |
| Capacitor DC Block | 100pF                 | 0402                  |
| Indutor            | 22nH                  | 0402                  |
| Filtro SAW         | 30MHz-6GHz            | Johanson 3700BL15B050 |
| LPF IF             | 500MHz                | LFCN-500+             |
| Resistor Matching  | 50Ω                   | 0402                  |

---

### 7️⃣ **AT86RF215 Modem**

| Componente | Detalhe           | Encapsulamento |
| ---------- | ----------------- | -------------- |
| AT86RF215  | Sub-1GHz / 2.4GHz | QFN            |
| TCXO       | 26MHz             | SMD CMOS       |
| Capacitor  | 12pF              | 0402           |
| Indutor    | 15nH / 5.6nH      | 0402           |
| Capacitor  | 1.2pF / 1.8pF     | 0402           |

---

### 8️⃣ **PN532 NFC**

| Componente | Detalhe          | Encapsulamento |
| ---------- | ---------------- | -------------- |
| PN532      | NFC SPI/I2C/UART | SMT            |
| Antena     | 13.56MHz         | PCB trace      |
| L1         | 1.5µH            | 0603           |
| C1, C2     | 27pF             | 0402           |
| C3         | 100nF            | 0402           |
| C4         | 10µF             | 0805           |

---

## **Guia de Conexões / Interligações**

### 1️⃣ **Alimentação**

* 3.3V → RP2040, ESP32-S3, AT86RF215, RFFC5072, HF Front-End, TCXO, CLK Buffer, PN532
* GND comum para todos
* Decoupling próximo a cada IC (100nF + 1µF)

### 2️⃣ **Clock**

* TCXO 26MHz → Buffer → RP2040 XIN/XOUT, AT86RF215 CLK, RFFC5072 REF\_IN, HF Front-End
* External REF opcional → RFFC5072

### 3️⃣ **RP2040 → ESP32-S3**

* UART0 GP0 → ESP32 RX
* UART0 GP1 → ESP32 TX
* SPI0 GP2/GP3 → ESP32 SCK/MOSI
* SPI0 GP0/GP1 → ESP32 MISO/CS
* RESET\_OUT → ESP32 EN/Reset

### 4️⃣ **RP2040 → RFFC5072**

* SPI → Pin31/32
* GPIO → ENABLE / RESETB
* LO\_IN / RF\_IN / IF\_OUT → Módulos LO, RF, IF

### 5️⃣ **RP2040 → AT86RF215**

* SPI → Modem
* Clock → TCXO Buffer
* RF09 / RF24 → Matching Network → Antenas

### 6️⃣ **RP2040 → PN532**

* SPI/I2C/UART → RP2040
* Antena → L1 + C1/C2
* Alimentação → 3.3V + GND

### 7️⃣ **RF Front-End**

* Antenas → DC Block → Indutores → Filtros → ICs
* IF → LPF → Amplificador → ADC RP2040
