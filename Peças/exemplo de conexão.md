# README - Sistema de Comunicação RF Integrado (Versão 3.1)

## Diagrama de Conexões Atualizado

```mermaid
graph TD
    %% =================== ALIMENTAÇÃO ===================
    subgraph "Sistema de Alimentação"
        USB["Micro USB 5V"] --> TP4056["Carregador TP4056"]
        TP4056 --> Bateria["Bateria LiPo 3.7V"]
        Bateria --> AMS1117_3V3["Regulador Lógico (AMS1117-3.3V)"]
        Bateria --> LDO_RF["Regulador RF (RT9193-3.3V)"]
        Bateria --> LDO_SDR["Regulador SDR (RT9193-3.3V)"]
    end

    %% =================== CONTROLADORES ===================
    subgraph "Controladores Principais"
        RP2040["RP2040\n(Core Principal)"]
        ESP32["ESP32-S3\n(Comunicações)"]
        
        RP2040 -->|UART0| ESP32
        ESP32 -->|UART0| RP2040
    end

    %% =================== MÓDULOS RF ===================
    subgraph "Módulos RF"
        CC1101["CC1101\n(Sub-1GHz)"]
        PN532["PN532\n(NFC)"]
        AT86RF215["AT86RF215\n(2.4/900MHz)"]
        LMS6002D["LMS6002D\n(SDR)"]
        RFFC5072["RFFC5072\n(Misturador)"]
    end

    %% =================== CONEXÕES SPI ===================
    RP2040 -->|SPI0| Módulos_RF
    subgraph Módulos_RF["Barramento SPI"]
        MOSI
        MISO
        SCK
    end
    
    MOSI --> CC1101 & PN532 & AT86RF215 & LMS6002D & RFFC5072
    MISO --> CC1101 & PN532 & AT86RF215 & LMS6002D & RFFC5072
    SCK --> CC1101 & PN532 & AT86RF215 & LMS6002D & RFFC5072

    %% =================== SELEÇÃO DE CHIP ===================
    RP2040 -->|GPIO0| CS_CC1101
    RP2040 -->|GPIO1| CS_PN532
    RP2040 -->|GPIO2| CS_AT86RF215
    RP2040 -->|GPIO3| CS_LMS6002D
    RP2040 -->|GPIO6| CS_RFFC5072

    %% =================== INFRAVERMELHO ===================
    ESP32 -->|GPIO4| IR_TX["Transmissor IR"]
    ESP32 -->|GPIO5| IR_RX["Receptor IR"]

    %% =================== ANTENAS ===================
    CC1101 --> ANT_433["Antena 433MHz"]
    PN532 --> ANT_NFC["Antena NFC"]
    AT86RF215 --> ANT_900["Antena 900MHz"]
    AT86RF215 --> ANT_24["Antena 2.4GHz"]
    LMS6002D --> ANT_SDR["Antena Wideband"]
    RFFC5072 --> ANT_LO["Antena LO"]

    %% =================== ALIMENTAÇÃO MÓDULOS ===================
    AMS1117_3V3 --> RP2040 & CC1101 & PN532
    LDO_RF --> AT86RF215 & RFFC5072
    LDO_SDR --> LMS6002D

    %% =================== ATERRAMENTO ===================
    GND((GND))
    RP2040 --> GND
    ESP32 --> GND
    CC1101 --> GND
    PN532 --> GND
    AT86RF215 --> GND
    LMS6002D --> GND
    RFFC5072 --> GND
```

## Melhorias Implementadas na Versão 3.1

### 1. Sistema de Alimentação Otimizado
- **Reguladores dedicados** para cada categoria de módulo:
  - Lógica digital (AMS1117-3.3V)
  - Circuitos RF (RT9193-3.3V)
  - Módulo SDR (RT9193-3.3V separado)
  
- **Proteção contra sobretensão** em todos os módulos RF
- **Monitoramento de bateria** integrado

### 2. Arquitetura de Comunicação
- **Barramento SPI compartilhado** com seleção individual de chip
- **UART dedicada** para comunicação entre RP2040 e ESP32-S3
- **Timing de comunicação** otimizado para cada módulo

### 3. Controle de Módulos RF
- **GPIO dedicados** para seleção de cada dispositivo SPI
- **Sequenciamento de inicialização** automático
- **Isolamento de sinais** entre módulos de diferentes bandas

### 4. Sistema de Antenas
- **Conexões otimizadas** para cada faixa de frequência
- **Casamento de impedância** verificado em todas as saídas RF
- **Rotas de sinal** separadas para evitar interferência

## Configuração Recomendada

```python
# Exemplo de inicialização dos módulos RF
def init_rf_modules():
    # Configuração do barramento SPI
    spi = SPI(0, baudrate=1000000, polarity=0, phase=0,
              sck=Pin(2), mosi=Pin(3), miso=Pin(4))
    
    # Pinos de seleção de chip
    cs_cc1101 = Pin(0, Pin.OUT)
    cs_pn532 = Pin(1, Pin.OUT)
    cs_at86 = Pin(2, Pin.OUT)
    
    # Inicialização dos módulos
    cc1101 = CC1101(spi, cs_cc1101)
    pn532 = PN532(spi, cs_pn532)
    at86 = AT86RF215(spi, cs_at86)
```

## Especificações Técnicas

| Módulo          | Frequência       | Interface | Consumo |
|-----------------|------------------|-----------|---------|
| CC1101          | 300-928 MHz      | SPI       | 30mA    |
| PN532           | 13.56 MHz        | SPI/I2C   | 50mA    |
| AT86RF215       | 400/900/2400 MHz | SPI       | 120mA   |
| LMS6002D        | 30MHz-6GHz       | SPI       | 200mA   |
| RFFC5072        | LO para SDR      | SPI       | 80mA    |

## Considerações de Projeto

1. **Layout de PCB**:
   - Manter traços SPI curtos e de igual comprimento
   - Separar áreas analógicas e digitais
   - Usar plano de terra contínuo

2. **Compatibilidade**:
   - Verificar versões dos módulos RF
   - Atualizar firmwares antes da integração

3. **Certificação**:
   - Testar conformidade RF para cada banda
   - Verificar regulamentações locais

> **Nota**: Recomenda-se o uso de ferramentas de análise de espectro durante a calibração dos módulos RF.