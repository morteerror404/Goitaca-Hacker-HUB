show! Separei em **tabelas por funcionalidade**, já no formato de pré-BOM para você levar ao KiCad/EasyEDA. Removi o FPGA e distribuí os barramentos com o **RP2040 como mestre SPI único** e **ESP32-S3 via UART/I²C**.
Colunas: **Ref** (placeholder), **Qtd**, **Componente**, **Especificação**, **Pacote**, **Notas**.

> Dica: marque os “Ref” na ordem que for esquematizando (U1, U2…; Lx/Cx por bloco). Onde pus “(ajustar)” é para fechar valor na bancada com VNA/medição.

---

# Controle & Comunicação (Core)

| Ref   | Qtd | Componente                 | Especificação       | Pacote       | Notas                                              |
| ----- | --: | -------------------------- | ------------------- | ------------ | -------------------------------------------------- |
| U1    |   1 | **RP2040**                 | MCU dual-core + USB | QFN-56       | Mestre SPI (AT86RF215/LMS/CC1101/PN532), I²C, UART |
| U2    |   1 | **Flash QSPI**             | 16–32 MB ≥133 MHz   | WSON-8/SOP-8 | Firmware/Logs; ver footprint alternativo           |
| Y1    |   1 | **Cristal RP2040**         | 12 MHz, ±20 ppm     | 3225         | Carga 18–22 pF (C0G)                               |
| U3    |   1 | **ESP32-S3 (WROOM/1U)**    | Wi-Fi/BLE           | Módulo       | UART com RP2040; se 1U, usar antena chip           |
| AE1   |   1 | **Antena 2.4 GHz (se 1U)** | Chip/PCB            | SMD          | Rede π de ajuste opcional                          |
| JTAG1 |   1 | **Header SWD RP2040**      | 2×5/2×7             | TH/SMD       | Depuração                                          |
| UART1 |   1 | **Header UART**            | 1×4                 | TH           | Console de debug                                   |

---

# Alimentação & Proteção

| Ref   |   Qtd | Componente                   | Especificação              | Pacote     | Notas                                    |
| ----- | ----: | ---------------------------- | -------------------------- | ---------- | ---------------------------------------- |
| J1    |     1 | **USB-C**                    | CC resistors 5.1 k         | –          | Alimentação/USB                          |
| U4    |     1 | **TP4056**                   | Charger Li-ion c/ proteção | SOP-8      | DW01 + 8205A (ou módulo discreto na PCB) |
| BT1   |     1 | **Células 18650 (paralelo)** | 1–2 ×                      | Holder/PCB | BMS/PCM se fora do TP4056                |
| U5    |     1 | **Buck-Boost 3.3 V**         | 2–3 A                      | QFN/TSSOP  | TPS630xx/MPM; eficiência                 |
| Lp1   |     1 | **Indutor potência**         | p/ buck-boost              | SMD        | Conforme design do regulador             |
| U6–U8 |   2–3 | **LDO low-noise 3.3 V**      | PSRR alto                  | SOT-23/DFN | Trilhos RF/PLL/TCXO dedicados            |
| TVS1  |     1 | **TVS USB**                  | ESD proteção               | SMD        | Linha D+/D− e VBUS                       |
| PTC1  |     1 | **Fusível resetável**        | 0.5–1 A                    | SMD        | Entre VBUS e sistema                     |
| Cbulk |   3–5 | **Capacitores bulk**         | 47–220 µF                  | SMD        | Próximo a reguladores                    |
| Cdec  | 30–60 | **Bypass**                   | 100 nF + 1–10 µF           | 0402/0603  | Por CI e por trilho                      |
| FBx   |  6–12 | **Ferrite beads**            | 100–600 Ω@100 MHz          | 0402/0603  | Em trilhos sensíveis/clock               |

---

# Clocks & Referência (comuns ao SDR)

| Ref     |     Qtd | Componente            | Especificação              | Pacote    | Notas                                 |
| ------- | ------: | --------------------- | -------------------------- | --------- | ------------------------------------- |
| Y2      |       1 | **TCXO p/ AT86RF215** | 26 MHz (ou 32 MHz), ±2 ppm | 2520/3225 | Trilho limpo via LDO dedicado         |
| Y3      |       1 | **TCXO p/ LMS6002D**  | 30.72 MHz, ±2 ppm          | 2520/3225 | Se usar LMS; LDO separado recomendado |
| Osc-ref | 1 (opt) | **Ref externa**       | 10 MHz in (SMA)            | –         | Para calibração/experimentos          |

---

# SDR – Caminho AT86RF215 (Sub-GHz + 2.4 GHz)

| Ref   |   Qtd | Componente               | Especificação         | Pacote    | Notas                            |
| ----- | ----: | ------------------------ | --------------------- | --------- | -------------------------------- |
| U10   |     1 | **AT86RF215**            | Dual-band transceiver | QFN       | SPI do RP2040; IRQ/GPIO mapeados |
| Y2    |     1 | **TCXO**                 | 26/32 MHz             | 2520/3225 | (já listado em Clocks)           |
| NET09 | 1 set | **Matching 863–928 MHz** | L/C C0G               | 0402/0603 | **Ajustar** com VNA              |
| NET24 | 1 set | **Matching 2.4 GHz**     | L/C C0G               | 0402/0603 | **Ajustar** com VNA              |
| FL09  |     1 | **Filtro/BPF 900**       | SAW/Cerâmico          | SMD       | Opcional, melhora seletividade   |
| FL24  |     1 | **Filtro/BPF 2.4**       | SAW/Cerâmico          | SMD       | Opcional                         |
| JRF1  |     1 | **Conector RF 900**      | u.FL/SMA              | –         | Saída/entrada RX/TX              |
| JRF2  |     1 | **Conector RF 2.4**      | u.FL/SMA              | –         | Saída/entrada RX/TX              |
| EN215 |     1 | **Pull-ups/downs**       | EN/RESET              | 0402      | Sequenciamento                   |

---

# SDR – Caminho LMS6002D + RFFC5072 (Wideband) *(opcional avançado, sem FPGA)*

| Ref       |   Qtd | Componente         | Especificação           | Pacote    | Notas                             |
| --------- | ----: | ------------------ | ----------------------- | --------- | --------------------------------- |
| U20       |     1 | **LMS6002D**       | Transceptor 0.3–3.8 GHz | QFN       | SPI do RP2040; IQ diff RX/TX      |
| U21       |     1 | **RFFC5072**       | Síntese/LO, mixer       | QFN       | SPI do RP2040                     |
| Y3        |     1 | **TCXO 30.72 MHz** | ±2 ppm                  | 2520/3225 | Trilho LDO dedicado (baixo ruído) |
| FL-LO     |     1 | **LPF LO**         | Ordem 5–7               | SMD       | Isolamento LO                     |
| FL-RF     |     1 | **BPF RF**         | Banda alvo              | SMD       | Selecionar por uso                |
| FL-IF     |     1 | **LPF IF**         | 5ª ordem                | SMD       | IQ/baseband                       |
| NET-IQ    | 1 set | **Matching IQ**    | 50 Ω diff               | 0402      | Curto e simétrico                 |
| JRF-RX/TX |   2–3 | **Conectores RF**  | SMA/u.FL                | –         | RX, TX (e LO/IF se expostos)      |
| SH-LMS    |     1 | **Blindagem RF**   | Can metálico            | –         | Sobre LMS/RFFC                    |
| EN-LMS    |     1 | **Pulls**          | EN/RESET                | 0402      | Sequenciamento                    |

> Sem FPGA, limite a taxa/stream. Use **DMA + PIO** do RP2040 e/ou buffers no ESP32-S3 para bursts controlados.

---

# 433 MHz – CC1101 (Link dedicado de baixo consumo)

| Ref    |   Qtd | Componente         | Especificação | Pacote    | Notas                      |
| ------ | ----: | ------------------ | ------------- | --------- | -------------------------- |
| U30    |     1 | **CC1101**         | 300–928 MHz   | QFN       | SPI do RP2040, CS dedicado |
| Y30    |     1 | **Cristal CC1101** | 26 MHz        | 3225      | Ajustar carga              |
| NET433 | 1 set | **Matching 433**   | L/C C0G       | 0402/0603 | **Ajustar**                |
| BAL433 |     1 | **Balun RF**       | 50 Ω          | SMD       | Adaptação dif→single       |
| JRF433 |     1 | **Conector RF**    | u.FL/SMA      | –         | Antena λ/4 externa         |

---

# NFC – PN532 (13.56 MHz)

| Ref     | Qtd | Componente               | Especificação | Pacote | Notas                                 |
| ------- | --: | ------------------------ | ------------- | ------ | ------------------------------------- |
| U40     |   1 | **PN532**                | NFC/RFID      | QFN    | Interface I²C **ou** SPI (escolher 1) |
| AN-NFC  |   1 | **Antena NFC**           | PCB coil      | –      | Desenho do loop na PCB                |
| L-NFC   |   1 | **Indutor sintonia**     | \~1–2 µH      | SMD    | Ajustar p/ 13.56 MHz                  |
| C-NFC   | 2–3 | **Capacitores sintonia** | 10–100 pF     | C0G    | **Ajustar** ressonância               |
| SEL-IF  |   1 | **Jumpers IF**           | I²C/SPI/UART  | SMD    | Seleção de interface                  |
| ESD-NFC |   1 | **ESD array**            | 2–4 linhas    | SMD    | Proteção p/ laço antena               |

---

# IR – Transmissão/Recepção

| Ref      | Qtd | Componente            | Especificação | Pacote | Notas                        |
| -------- | --: | --------------------- | ------------- | ------ | ---------------------------- |
| D50      |   1 | **LED IR**            | 940 nm        | TH/SMD | Driver por transistor        |
| Q50      |   1 | **Transistor driver** | NPN/P-MOSFET  | SOT-23 | Corrente do LED              |
| R50      |   1 | **Resistor série**    | 10–100 Ω      | 0603   | Ajuste de corrente           |
| RX50     |   1 | **Fotodiodo/TSOP**    | 38–56 kHz     | SMD/TH | Com demodulador, se preferir |
| Rbias/Cf |   2 | **Bias/filtro**       | –             | 0603   | Conformer ao receptor        |

---

# UI – Display, Botões e LEDs

| Ref  | Qtd | Componente           | Especificação    | Pacote               | Notas                            |
| ---- | --: | -------------------- | ---------------- | -------------------- | -------------------------------- |
| U60  |   1 | **OLED SSD1306**     | 0.96″ I²C        | Módulo SMD/footprint | Pode ser footprint direto na PCB |
| SWx  | 2–3 | **Botões tácteis**   | Reset/Boot/Menu  | SMD                  | Com ESD se externo               |
| ENC1 |   1 | **Encoder rotativo** | Push opcional    | TH                   | UI local                         |
| LEDx | 3–4 | **LEDs**             | Power/Status/RF  | 0603                 | Com resistores 1–4.7 k           |
| BZ1  |   1 | **Buzzer**           | 3.3 V com driver | SMD                  | Opcional                         |

---

# Conectores & Barramentos

| Ref     | Qtd | Componente         | Especificação  | Pacote | Notas                  |
| ------- | --: | ------------------ | -------------- | ------ | ---------------------- |
| PMOD1–2 | 1–2 | **Headers PMOD**   | 2×6/2×8        | TH     | SPI/I²C/UART + 3V3/GND |
| JRF\*   | 2–6 | **Conectores RF**  | SMA fêmea/u.FL | –      | Por banda/caminho      |
| DBG     |   1 | **Test-pads**      | Vários nós     | –      | Alimentação/LO/RF/IF   |
| PWR     |   1 | **Header bateria** | JST-PH         | TH     | Isolar quando USB      |

---

# Proteções & Layout RF (itens de apoio)

| Ref      | Qtd | Componente          | Especificação | Pacote | Notas               |
| -------- | --: | ------------------- | ------------- | ------ | ------------------- |
| SH-RF    | 1–3 | **Blindagem**       | Can metálico  | –      | Sobre LMS/RFFC/LO   |
| GND-VIAS |   – | **Vias de costura** | 1 mm pitch    | –      | Em bordas CPWG      |
| COAX     |   – | **Cabos coax**      | RG-178/u.FL   | –      | Para bancada/testes |

---

## Barramentos (mapeamento sugerido, sem FPGA)

| Função    | Barramento         | Pinos chave                         |
| --------- | ------------------ | ----------------------------------- |
| AT86RF215 | **SPI0** do RP2040 | SCK/MOSI/MISO + CS\_215 + IRQ\_215  |
| LMS6002D  | **SPI0** (shared)  | CS\_LMS + IRQ\_LMS (dedicar linhas) |
| RFFC5072  | **SPI0** (shared)  | CS\_RFFC                            |
| CC1101    | **SPI0** (shared)  | CS\_CC1101 + GDO0/GDO2              |
| PN532     | **I²C0** (ou SPI1) | SDA/SCL (+ SEL IF)                  |
| OLED      | **I²C0**           | Endereço 0x3C (típico)              |
| ESP32-S3  | **UART0**          | TX/RX + RTS/CTS (opcional)          |

> Se possível, mantenha **SPI1** livre para expansões/PMOD. Use **CS dedicados** e **pull-ups/downs** para boot-safe.

---
Perfeito — abaixo criei **duas tabelas** que conectam o que já temos (as listas de componentes) diretamente às **funcionalidades do projeto**.
A **primeira tabela** é de alto nível (bloco funcional → componentes principais/support → barramento / ref rápido).
A **segunda tabela** mapeia **cada componente-chave** para a(s) funcionalidade(s) específicas, seu papel, o ref sugerido nas tabelas anteriores e se é **Essencial / Módulo / Opcional**.

Se quiser eu já converto isso em CSVs separados por bloco para importar no KiCad/EasyEDA.

# 1) Mapa funcional — blocos do sistema → componentes principais / suporte / barramento

| Bloco funcional                                        | Componentes principais                                                      | Componentes de suporte / passivos                                             | Barramento / Conector / Ref                          |
| ------------------------------------------------------ | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------- |
| Controle central / firmware                            | RP2040 (U1), Flash QSPI (U2), cristal 12MHz (Y1)                            | Headers SWD/JTAG, pads de teste, caps de bypass                               | SPI (mestre), I²C, UART; Ref: U1/U2/Y1               |
| Comunicação externa / host                             | ESP32-S3 (U3), antena 2.4GHz (AE1)                                          | Header UART, conector USB-C                                                   | UART entre RP2040 ↔ ESP32; USB-C para PC/energia     |
| Alimentação & gestão                                   | Buck-Boost 3.3V (U5), TP4056 (U4), LDOs RF (U6–U8), PTC, TVS                | Indutor de potência, capacitores bulk, ferrites                               | Conector bateria JST (BT1), USB-C (J1)               |
| SDR (AT86RF215) — Sub-GHz + 2.4 GHz (CORE)             | AT86RF215 (U10), TCXO 26/32MHz (Y2)                                         | Matching L/C (NET09/NET24), filtros BPF/SAW (FL09/FL24), u.FL/SMA (JRF1/JRF2) | SPI (RP2040 SPI0) + CS\_215, IRQ\_215                |
| SDR wideband (LMS6002D + RFFC5072) — opcional avançado | LMS6002D (U20), RFFC5072 (U21), TCXO 30.72MHz (Y3)                          | Filtros LO/RF/IF, blindagem (SH-LMS), conectores SMA                          | SPI (RP2040 SPI0) + CS\_LMS, CS\_RFFC; IQ diff lines |
| Link 433 MHz dedicado (baixo consumo)                  | CC1101 (U30), cristal 26MHz (Y30)                                           | Matching 433, balun (BAL433), conector JRF433                                 | SPI (RP2040 SPI0) + CS\_CC1101                       |
| NFC / RFID                                             | PN532 (U40), antena PCB (AN-NFC)                                            | L/C de sintonia, jumper interface (I²C/SPI)                                   | I²C (ou SPI) com RP2040; antena PCB dedicada         |
| IR TX/RX                                               | LED IR (D50), driver (Q50), receptor TSOP (RX50)                            | Resistores/RC de filtro                                                       | GPIOs do RP2040 (PWM para TX)                        |
| UI\_local / Status                                     | OLED SSD1306 (U60), botões (SWx), encoder (ENC1), LEDs (LEDx), buzzer (BZ1) | Resistores, pads                                                              | I²C (OLED), GPIOs/pullups para botões/LEDs           |
| Conectividade RF física / testes                       | SMA/u.FL (JRF\*), test pads (DBG), coax                                     | Acopladores/atenuadores (opcionais)                                           | Acesso físico para medição e antenas                 |
| Proteção / EMC                                         | TVS, ferrites, PTC, blindagens                                              | Planos GND, vias de costura, keep-out                                         | Distribuído pelo PCB                                 |

# 2) Mapa por componente (com papel, ref e status)

| Componente                             | Papel / Funcionalidade                                                     |              Ref sugerido | Barramento / IO                 | Status (Essencial / Módulo / Opcional) | Observações rápidas                                |
| -------------------------------------- | -------------------------------------------------------------------------- | ------------------------: | ------------------------------- | -------------------------------------- | -------------------------------------------------- |
| RP2040                                 | MCU mestre — controla SPI/I²C/UART, sequencing, UI, lógica SDR de controle |                        U1 | SPI0 (mestre), I²C0, UART0      | Essencial                              | Centraliza o controle; otimizar firmware (DMA/PIO) |
| Flash QSPI                             | Armazenamento de firmware e logs                                           |                        U2 | QSPI                            | Essencial                              | 16–32MB recomendado                                |
| Cristal 12MHz (RP2040)                 | Clock do RP2040                                                            |                        Y1 | —                               | Essencial                              | Capas 18–22 pF                                     |
| ESP32-S3                               | Comunicação Wi-Fi/BLE / uplink de dados / offloading                       |                        U3 | UART (RP2040 ↔ ESP32)           | Essencial                              | Usado para host GUI/stream e rede                  |
| Buck-Boost 3.3V                        | Fornece 3.3V estável sob bateria                                           |                        U5 | Alimentação do sistema          | Essencial                              | Substitui AMS1117; melhor eficiência               |
| TP4056 (+proteção)                     | Carregamento de bateria Li-ion                                             |                        U4 | Alimentação bateria             | Essencial (se portátil)                | Se a bateria for externa, manter BMS/PCM           |
| LDO low-noise (RF)                     | Alimentação limpa para TCXOs / RF chips                                    |                     U6–U8 | Trilho 3.3V → LDO               | Essencial                              | Um por bloco RF crítico (TCXO / LMS / AT86)        |
| AT86RF215                              | Transceptor sub-GHz & 2.4 GHz — Core SDR-Lite                              |                       U10 | SPI0 + CS\_215 + IRQ            | Essencial (no CORE escolhido)          | Matching e filtros obrigatórios para desempenho    |
| TCXO 26/32MHz                          | Referência estável p/ AT86RF215                                            |                        Y2 | Alimentação LDO dedicado        | Essencial ao AT86                      | Baixo ruído, PSRR crítico                          |
| Matching 900 / 2.4                     | Ajuste de antena / impedância                                              |             NET09 / NET24 | RF lines                        | Essencial                              | Valores finalizados via VNA                        |
| Filtros BPF/SAW                        | Melhora seletividade / rejeição                                            |               FL09 / FL24 | RF lines                        | Opcional/recomendado                   | Muito útil em ambiente ruidoso                     |
| Conectores RF (u.FL/SMA)               | Antenas externas / testes                                                  |          JRF1/JRF2/JRF433 | RF ports                        | Essencial                              | Preferir SMA para bancada, u.FL para compacto      |
| LMS6002D                               | Transceptor wideband — alternativa PRO                                     |                       U20 | SPI0 + IQ diff                  | Módulo/Alternativa                     | Requer blindagem e grande esforço RF               |
| RFFC5072                               | Sintetizador LO para LMS (wideband)                                        |                       U21 | SPI0                            | Módulo/Alternativa                     | Necessário se usar LMS6002D                        |
| CC1101                                 | Link 433 MHz de baixo consumo                                              |                       U30 | SPI0 + CS\_CC1101               | Módulo                                 | Excelente para enlaces simples/pouca energia       |
| PN532                                  | NFC 13.56 MHz — leitura/gravação                                           |                       U40 | I²C (ou SPI)                    | Módulo                                 | Antena PCB desenhada na placa                      |
| Antena NFC (loop)                      | Ler/escrever tags                                                          |                    AN-NFC | PCB coil                        | Módulo                                 | Requer sintonia L/C fina                           |
| LED IR / driver / receptor             | Emulação IR TX/RX                                                          |              D50/Q50/RX50 | GPIO/PWM                        | Módulo/Opcional                        | Útil para testes e controle remoto                 |
| OLED SSD1306                           | Exibição local                                                             |                       U60 | I²C                             | Módulo                                 | Serve para status e menus sem PC                   |
| Botões / Encoder / LEDs / Buzzer       | UI local                                                                   |         SWx/ENC1/LEDx/BZ1 | GPIOs                           | Módulo                                 | Simples e barato; facilita uso em campo            |
| TVS / Ferrites / PTC                   | Proteção ESD/EMI e fusível resetável                                       |         TVS1 / FBx / PTC1 | Linha de alimentação / USB / RF | Essencial                              | Protege entradas expostas e USB                    |
| Blindagem RF (shield cans)             | Reduz interferência entre blocos RF                                        |                     SH-RF | Sobre LMS/RFFC/AT86             | Opcional recomendada                   | Se usar LMS, torna-se quase obrigatório            |
| Test pads / SMA de teste / acopladores | Debug e calibração                                                         |               DBG / JRF\* | RF / TTL / I²C / SPI            | Essencial para desenvolvimento         | Facilita ajuste de matching e medição              |
| Capacitores / resistores / indutores   | Bypass, matching, redes                                                    | Cdec / Cbulk / Lp1 / Rxxx | Distribuído                     | Essencial                              | Quantidades grandes — já listadas no BOM           |
| Planos GND e vias de costura           | Integridade RF e retorno                                                   |                         — | PCB layout                      | Essencial                              | 4 camadas recomendado                              |
