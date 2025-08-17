Perfeito! Vamos organizar em duas partes:

1. **Diagramas individuais de cada módulo** (subgraphs separados com todos os componentes).
2. **Diagrama geral de interconexão** entre os módulos.

---

## **1️⃣ Diagramas Individuais de Módulos**

```mermaid
%% -------------------- Módulo RP2040 --------------------
graph TD
    subgraph MCU_RP2040["RP2040 - MCU Principal"]
        VCC_3V3 --> RP2040
        GND --> RP2040
        RP2040_GP0["GP0: UART0 TX / SPI0 RX / I2C0 SDA"]
        RP2040_GP1["GP1: UART0 RX / SPI0 CSn / I2C0 SCL"]
        RP2040_GP2["GP2: SPI0 SCK / I2C1 SDA"]
        RP2040_GP3["GP3: SPI0 MOSI / I2C1 SCL"]
        RP2040_GP12["XIN / GP12"]
        RP2040_GP13["XOUT / GP13"]
        RP2040_GP22["GPIO CTRL"]
        RP2040_GP26["ADC0"]
        RP2040_RST["RESET_OUT"]
    end
```

```mermaid
%% -------------------- Módulo ESP32 --------------------
graph TD
    subgraph MCU_ESP32["ESP32-S3 DevKit"]
        VCC_3V3 --> ESP32
        GND --> ESP32
        ESP32_GPIO10["GPIO10 - SPI CS"]
        ESP32_GPIO11["GPIO11 - SPI MOSI"]
        ESP32_GPIO12["GPIO12 - SPI SCK"]
        ESP32_GPIO13["GPIO13 - SPI MISO"]
        ESP32_GPIO17["GPIO17 - UART RX"]
        ESP32_GPIO18["GPIO18 - UART TX"]
        ESP32_EN["EN / Reset"]
    end
```

```mermaid
%% -------------------- Módulo RFFC5072 Mixer --------------------
graph TD
    subgraph MIXER_RFFC5072["RFFC5072 Mixer/PLL"]
        VDD["3.3V"] --> C9["100nF"] --> GND
        VDD --> C10["1µF"] --> GND
        LO_IN["LO IN 50Ω matched"]
        RF_IN["RF IN 50Ω matched"]
        IF_OUT["IF OUT 50Ω matched"]
        SCLK["SPI SCLK"]
        SDATA["SPI MOSI"]
        ENBL["ENABLE"]
        RESET["RESETB"]
    end
```

```mermaid
%% -------------------- Módulo LO --------------------
graph TD
    subgraph LO_Module["LO / PLL / VCO"]
        PLL["PLL/VCO 1.5-4GHz"]
        C_LO["100pF DC Block"]
        L_LO["15nH"]
        LO_IN["LO_IN"]
        C_GND["1pF"] --> GND
    end
```

```mermaid
%% -------------------- Módulo Front-End RF --------------------
graph TD
    subgraph RF_FrontEnds["Front-Ends RF"]
        RF_SUB1G["RF Sub-1GHz"]
        RF_6GHz["RF 30MHz-6GHz"]
        HF_FE["HF 15-30MHz"]
        SMA_SUB1G["SMA Sub-1GHz"]
        SMA_6GHz["SMA 30MHz-6GHz"]
        SMA_HF["SMA 15-30MHz"]
    end
```

```mermaid
%% -------------------- Módulo PN532 --------------------
graph TD
    subgraph PN532_Module["PN532 NFC Reader"]
        VDDP["3.3V"] --> C3["100nF"] --> GND
        VDDP --> C4["10µF"] --> GND
        ANT["Antenna PCB 13.56MHz"]
        L1["1.5µH"]
        C1["27pF"]
        C2["27pF"]
        SPI_MCU["SPI MCU"]
        I2C_MCU["I2C MCU"]
        UART_MCU["UART MCU"]
    end
```

```mermaid
%% -------------------- Módulo Filtros --------------------
graph TD
    subgraph FILTERS["Filtros SAW / LPF / BPF"]
        FL1["LPF 7th Order SAW"]
        FL2["BPF 30MHz-6GHz SAW"]
        FL3["LPF 5th Order"]
        FL_LO["SAW LPF LO"]
    end
```

---

## **2️⃣ Diagrama de Conexões Entre Módulos**

```mermaid
graph TD
    %% Alimentação
    VCC_3V3 --> MCU_RP2040
    VCC_3V3 --> MCU_ESP32
    VCC_3V3 --> MIXER_RFFC5072
    VCC_3V3 --> LO_Module
    VCC_3V3 --> RF_FrontEnds
    VCC_3V3 --> PN532_Module

    GND --> MCU_RP2040
    GND --> MCU_ESP32
    GND --> MIXER_RFFC5072
    GND --> LO_Module
    GND --> RF_FrontEnds
    GND --> PN532_Module

    %% Clock
    TCXO["26MHz TCXO"] --> MCU_RP2040_GP12
    TCXO --> MCU_RP2040_GP13
    TCXO --> MIXER_RFFC5072
    TCXO --> AT86RF215

    %% RP2040 -> Mixer / Front-End
    MCU_RP2040_GP0 --> MIXER_RFFC5072_SCLK
    MCU_RP2040_GP3 --> MIXER_RFFC5072_SDATA
    MCU_RP2040_GP22 --> MIXER_RFFC5072_ENBL
    MCU_RP2040_GP22 --> MIXER_RFFC5072_RESET

    %% Mixer -> Filtros -> Front-End
    MIXER_RFFC5072_LO_IN --> LO_Module
    MIXER_RFFC5072_RF_IN --> RF_FrontEnds_RF_6GHz
    MIXER_RFFC5072_IF_OUT --> FILTERS_FL3 --> MCU_RP2040_GP26

    %% RP2040 -> ESP32
    MCU_RP2040_GP0 --> MCU_ESP32_GPIO17
    MCU_RP2040_GP1 --> MCU_ESP32_GPIO18

    %% PN532 -> MCU
    PN532_Module_SPI_MCU --> MCU_RP2040_GP0
    PN532_Module_I2C_MCU --> MCU_RP2040_GP1
    PN532_Module_UART_MCU --> MCU_RP2040_GP0
```