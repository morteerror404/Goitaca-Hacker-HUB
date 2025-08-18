# 📡 Módulo PN532 NFC - Guia de Montagem em PCB

*Solução completa NFC para aplicações 13.56MHz*

## 📦 Lista de Materiais (BOM)

| Ref    | Componente | Especificações           | Qtd |
| ------ | ---------- | ------------------------ | --- |
| U1     | CI PN532   | Controlador NFC 13.56MHz | 1   |
| L1     | Indutor    | 1.5µH 0805 ±1%           | 1   |
| C1, C2 | Capacitor  | 27pF NPO 0805            | 2   |
| C3     | Capacitor  | 100nF X7R 0805           | 1   |
| C4     | Capacitor  | 10µF Tântalo             | 1   |
| R1     | Resistor   | 220Ω 0805                | 1   |
| R2     | Resistor   | 10kΩ 0805                | 1   |
| LED1   | LED        | 0805 SMD (Vermelho)      | 1   |
| J1     | Conector   | 2x4 Pinos 2.54mm         | 1   |

## 🛠 Instruções de Montagem

### 1. Preparação da PCB

* Limpeza da placa com álcool isopropílico.
* Verifique se todos os pads estão livres de oxidação.

### 2. Componentes SMD (Perfil de Reflow Recomendado)

1. **Aplicação de pasta**: Estêncil de 0.1mm de espessura.
2. **Pick & Place**: Coloque os componentes 0402 primeiro.
3. **Reflow**:

   * Pré-aquecimento: 2°C/s até 150°C.
   * Soak: 60 segundos a 180°C.
   * Pico: 30 segundos a 230°C.
   * Resfriamento: Máximo 3°C/s.

### 3. Montagem da Antena (Passo Crítico!)

* **Desenho da Antena**:

  * Loop de 3-5 espirais, 1mm de largura.
  * Espaçamento entre trilhas: 0.5mm.
  * Diâmetro total: \~50mm.
* **Dicas**:

  * Use cálculos de indutância para ajustar C1/C2.
  * Mantenha área sólida de GND sob a antena para garantir um bom desempenho.

### 4. Soldagem dos Componentes Convencionais

1. Comece pelos componentes mais baixos, como os resistores.
2. Em seguida, monte os componentes mais altos, como os conectores.
3. Verifique cuidadosamente por pontes de solda com lupa.

### 🔧 Ajuste e Teste

1. **Tuning da Antena**:

   * Use um analisador de rede para ajustar os capacitores C1 e C2.
   * Frequência alvo: 13.56MHz ±0.5MHz.

2. **Teste Funcional**:

```bash
# Comando básico para teste (Linux)
nfc-poll
```

* Distância de leitura esperada: 3-5cm com cartão NFC.

## ⚠️ Problemas Comuns e Soluções

| Problema              | Solução                      |
| --------------------- | ---------------------------- |
| Leitura curta         | Ajuste C1/C2 ou verifique L1 |
| Não detecta           | Confira soldas do CI PN532   |
| Aquecimento excessivo | Verifique curto nos 3.3V     |

---

## 📚 Conexões e Fluxo do Sistema

Aqui está um diagrama atualizado das conexões para a montagem correta:

```mermaid
flowchart TD
    %% ==================== NÓ PRINCIPAL: PN532 ====================
    subgraph PN532["Módulo PN532 (NFC/RFID 13.56MHz)"]
        direction TB
        PN532_IC["PN532 (SoC)"] -->|SPI| SPI_Header["Header SPI (MISO/MOSI/SCK/SS)"]
        PN532_IC -->|I2C| I2C_Header["Header I2C (SDA/SCL)"]
        PN532_IC -->|UART| UART_Header["Header UART (TX/RX)"]
        PN532_IC -->|RF_OUT| Matching_Network["Rede de Casamento"]
    end

    %% ==================== REDE DE CASAMENTO ANTENA ====================
    subgraph Matching_Network["Rede de Casamento (Antena PCB)"]
        direction LR
        L1["L1: 1.5µH (0805)"] --> C1["C1: 27pF (NPO)"]
        C1 --> C2["C2: 27pF (NPO)"]
        C2 --> ANT["Antena PCB (Loop 13.56MHz)"]
    end

    %% ==================== ALIMENTAÇÃO ====================
    subgraph Power["Alimentação 3.3V"]
        VDDP["VDD 3.3V"] --> C3["C3: 100nF (X7R 0805)"]
        VDDP --> C4["C4: 10µF (Tantalum)"]
        C3 --> GND
        C4 --> GND
    end

    %% ==================== CONEXÕES EXTERNAS ====================
    SPI_Header -->|MISO/MOSI/SCK/SS| MCU["MCU (ESP32/RP2040)"]
    I2C_Header -->|SDA/SCL| MCU
    UART_Header -->|TX/RX| MCU
    GND -->|Plano de Terra| PCB_GND["GND (Multiplas Vias)"]

    %% ==================== COMPONENTES OPCIONAIS ====================
    PN532_IC --> LED1["LED1 (Status NFC)"]
    PN532_IC --> R1["R1: 220Ω (0805)"] --> LED1
    VDDP --> R2["R2: 10kΩ (Pull-Up SS)"] --> SPI_Header

    %% ==================== LEGENDA ====================
    legend[("
        Especificações Críticas:
        1. Antena PCB: Loop de 3-5 voltas, 1mm trace width
        2. L1/C1/C2: ±1% tolerância para tuning preciso
        3. GND: Plano sólido sob antena (sem splits!)
        4. Header SPI/I2C/UART: Jumpers para seleção de modo
    ")]
```

## 📈 Orçamento Estimado

Aqui está uma estimativa aproximada dos custos dos componentes principais para a montagem de uma unidade:

| Componente           | Estimativa de Custo (R\$) |
| -------------------- | ------------------------- |
| CI PN532             | 45,00                     |
| Indutor (L1)         | 2,00                      |
| Capacitores (C1, C2) | 1,50                      |
| Capacitor (C3)       | 1,00                      |
| Capacitor (C4)       | 2,00                      |
| Resistores (R1, R2)  | 0,50                      |
| LED1                 | 1,50                      |
| Conector (J1)        | 3,00                      |
| **Total Aproximado** | **56,50**                 |

Este orçamento é apenas uma estimativa e pode variar dependendo do fornecedor.

