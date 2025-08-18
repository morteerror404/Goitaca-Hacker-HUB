Aqui estão os três diagramas Mermaid solicitados, com detalhamento completo de pinagens e conexões:

---

### **1. Diagrama de Integração AT86RF215-ZU + PN532 + RFFC5072**
```mermaid
flowchart TD
  %% ==================== Componentes Principais ====================
  subgraph AT86RF215["AT86RF215-ZU (Sub-1GHz/2.4GHz)"]
    RFP24["P4: RFP24"] --> LFCN["Filtro LFCN-1000+"]
    RFN24["P5: RFN24"] --> LFCN
    SPI_AT86["P26-28: MOSI/MISO/SCK\nP27: SELN"] --> SPI_BUS
    IRQ_AT86["P29: IRQ"] --> IRQ_BUS
  end

  subgraph PN532["PN532 (NFC/RFID)"]
    SPI_PN532["SPI: MOSI/P23, MISO/P24, SCK/P25\nSS/P22"] --> SPI_BUS
    IRQ_PN532["P6: IRQ"] --> IRQ_BUS
  end

  subgraph RFFC5072["RFFC5072 (Mixer RF)"]
    SPI_RFFC["SPI: MOSI/P12, MISO/P13, SCK/P14\nSS/P11"] --> SPI_BUS
    IF_OUT["P18: IF_OUT"] --> LPF["Filtro LPF5"]
  end

  %% ==================== Conexões Externas ====================
  LFCN --> SMA1["SMA Sub-1GHz"]
  LPF --> SMA2["SMA 30MHz-6GHz"]
  SPI_BUS --> LevelShifter["Conversor de Nível 3.3V↔5V"]
  IRQ_BUS --> MCU["MCU (GPIOs)"]
```

---

### **2. Diagrama de Conexões com Componentes Externos**
```mermaid
flowchart TD
  %% ==================== Componentes Principais ====================
  subgraph AT86RF215["AT86RF215-ZU"]
    RFP24 --> PA["PA RFX2401C (Sub-1GHz)"]
    FEA24["P1: FEA24"] --> PA_EN
    CLKO["P20: CLKO"] --> TCXO["TCXO 26MHz"]
  end

  subgraph PN532
    UART_PN532["P2/P3: TX/RX"] --> UART_ESP["ESP32 (UART1)"]
    VCC_PN532["VCC"] --> LDO["LDO 3.3V"]
  end

  subgraph RFFC5072
    LO_IN["P16: LO_IN"] --> SYN["Sintetizador ADF4351"]
    ANA_VDD["P20: ANA_VDD"] --> LDO
  end

  %% ==================== Subsistemas Externos ====================
  subgraph Power["Alimentação"]
    LDO --> BAT["Bateria 5V"]
    LDO --> AT86RF215
    LDO --> PN532
    LDO --> RFFC5072
  end

  subgraph Antennas["Antenas"]
    PA --> ANT1["Antena Sub-1GHz"]
    RFFC5072 --> ANT2["Antena 6GHz"]
  end

  subgraph MCU["Controladores"]
    ESP32 --> |SPI| AT86RF215
    RP2040 --> |SPI| RFFC5072
  end
```

---

### **3. Conexões com RP2040 e ESP32**
```mermaid
flowchart TD
  %% ==================== Microcontroladores ====================
  subgraph RP2040
    GPIO0["GPIO0 (SCK)"] --> SCK_BUS
    GPIO1["GPIO1 (MOSI)"] --> MOSI_BUS
    GPIO2["GPIO2 (MISO)"] --> MISO_BUS
    GPIO3["GPIO3 (CS_AT86)"] --> CS_AT86
    GPIO4["GPIO4 (CS_RFFC)"] --> CS_RFFC
  end

  subgraph ESP32
    GPIO18["GPIO18 (SCK)"] --> SCK_BUS
    GPIO23["GPIO23 (MOSI)"] --> MOSI_BUS
    GPIO19["GPIO19 (MISO)"] --> MISO_BUS
    GPIO5["GPIO5 (CS_PN532)"] --> CS_PN532
    GPIO16["GPIO16 (IRQ_AT86)"] --> IRQ_AT86
  end

  %% ==================== Módulos RF ====================
  subgraph AT86RF215
    SPI_AT86["P26-28: SCK/MOSI/MISO\nP27: SELN"] --> SCK_BUS & MOSI_BUS & MISO_BUS
    IRQ_AT86["P29: IRQ"] --> IRQ_AT86
  end

  subgraph PN532
    SPI_PN532["P22-25: SS/SCK/MOSI/MISO"] --> CS_PN532 & SCK_BUS & MOSI_BUS & MISO_BUS
  end

  subgraph RFFC5072
    SPI_RFFC["P11-14: SS/SCK/MOSI/MISO"] --> CS_RFFC & SCK_BUS & MOSI_BUS & MISO_BUS
  end

  %% ==================== Barramentos ====================
  SCK_BUS --> |Pull-up 10kΩ| VCC_3V3
  MOSI_BUS --> |Pull-up 10kΩ| VCC_3V3
  MISO_BUS --> |Pull-up 10kΩ| VCC_3V3
```
---

### **Detalhes Críticos:**
1. **Compatibilidade de Níveis Lógicos**:
   - Todos os módulos operam em 3.3V (checar se PN532 suporta 5V).
   - Usar *level shifters* se necessário (ex: ESP32 ↔ PN532).

2. **SPI Compartilhado**:
   - Cada módulo deve ter um pino CS dedicado.
   - Configurar *clock speed* adequado (ex: AT86RF215 suporta até 10MHz).

3. **Gerenciamento de IRQ**:
   - Priorizar IRQ do AT86RF215 (baixa latência para comunicações RF).

4. **Alimentação**:
   - Separar trilhas de 3.3V para RF (filtradas com LC) e digital.

5. **Antenas**:
   - Impedância controlada (50Ω) e afastamento de fontes de ruído.
