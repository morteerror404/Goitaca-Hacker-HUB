Perfeito — fiz a cotação dos **3 orçamentos em BRL** usando a taxa de câmbio atual de referência (1 USD = **R\$ 5,3931**, taxa de 14 de agosto de 2025). Fonte da taxa: TradingEconomics / XE (consultas na web). ([Trading Economics][1], [Xe][2])

**Observações importantes (leia antes):**

* Valores abaixo são **estimativas** convertidas da faixa em USD que definimos anteriormente. Use-as como planejamento inicial.
* **Excluem**: custo da placa PCB (fabricação), montagem/SMT, frete, impostos de importação (II/IPI/ICMS), eventuais custos de conformidade, e custos de bancada (VNA, analisador, etc.).
* Câmbio flutua — a cotação usada foi a de 14/08/2025; se for comprar, confirme o câmbio do dia e possíveis taxas do cartão/fornecedor.
* Todos os valores são **por unidade** (1 placa / 1 conjunto de componentes).

---

# 1) ORÇAMENTO A — CORE (perfil AT86RF215)

(este é o *núcleo essencial* para a V1 — RP2040 + ESP32-S3 + AT86RF215 + alimentação + componentes de suporte)

* Faixa USD (orig): **US\$ 35 — US\$ 65**
* Conversão usada: **1 USD = R\$ 5,3931**. ([Trading Economics][1])

**Total estimado (BRL):** **R\$ 188,76 — R\$ 350,55**
(US\$ 35 → R\$ 188,76 ; US\$ 65 → R\$ 350,55)

---

# 2) ORÇAMENTO B — MÓDULOS (itens que estarão no projeto mas montados como módulos na própria PCB)

(Ex.: CC1101 433 MHz, PN532 NFC, OLED, IR, headers, filtros, etc.)

* Faixa USD (orig): **US\$ 12 — US\$ 20**
* **Total estimado (BRL):** **R\$ 64,72 — R\$ 107,86**
  (US\$ 12 → R\$ 64,72 ; US\$ 20 → R\$ 107,86)

---

# 3) ORÇAMENTO C — OPCIONAIS (upgrades/expansões não necessários para funcionamento básico)

(Ex.: blindagens, FPGA removido — aqui seriam itens como iCE40UP5K se quisesse, LNAs/PAs, GPS, acopladores, etc.)

* Faixa USD (orig): **US\$ 8 — US\$ 35**
* **Total estimado (BRL):** **R\$ 43,14 — R\$ 188,76**
  (US\$ 8 → R\$ 43,14 ; US\$ 35 → R\$ 188,76)

---

# 4) (BÔNUS) CORE — Alternativa PRO (substitui/expande o CORE com LMS6002D + RFFC5072)

(Se você optar pelo caminho *LMS6002D + RFFC5072* em vez do AT86RF215, estimativa adicional conforme tabela anterior)

* Faixa USD (orig adicional): **US\$ 40 — US\$ 70** (adicional sobre o CORE comum)
* **TOTAL CORE (LMS path) estimado (BRL)**: **R\$ 404,48 — R\$ 728,07**

  * Explicação: somei a faixa adicional (US\$ 40–70) ao CORE original (US\$ 35–65) e converti. (US\$ 75—135 total → R\$ 404,48 — R\$ 728,07)

---

# Resumo compacto (por unidade)

| Orçamento              |        USD (faixa) |     BRL (faixa, convertido) |
| ---------------------- | -----------------: | --------------------------: |
| CORE (AT86RF215)       |  US\$ 35 — US\$ 65 | **R\$ 188,76 — R\$ 350,55** |
| MÓDULOS                |  US\$ 12 — US\$ 20 |  **R\$ 64,72 — R\$ 107,86** |
| OPCIONAIS              |   US\$ 8 — US\$ 35 |  **R\$ 43,14 — R\$ 188,76** |
| CORE (ALTERNATIVO LMS) | US\$ 75 — US\$ 135 | **R\$ 404,48 — R\$ 728,07** |

(Taxa usada: **1 USD = R\$ 5,3931** — 14/08/2025). ([Trading Economics][1], [Xe][2])