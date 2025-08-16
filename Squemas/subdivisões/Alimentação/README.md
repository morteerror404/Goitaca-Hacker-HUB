### Gráfico em Mermaid

```mermaid
graph TD
    A[Micro USB] -->|VCC| B[Fuse 3A (U4)]
    B --> C[D1 (1N5819W SL)]
    C -->|Catodo| D[V_SYS]
    E[BAT (18650 Paralelo)] --> F[D2 (1N5819W SL)]
    F -->|Catodo| D
    D --> G[LDO (MCP1700-3.3)]
    G --> H[3.3V_OUT]
    H --> I[Jumper P2 (H-Fem-12)]
    I --> J[Sistema (3.3V In)]
    B --> K[Barramento 5V (P2 Pin 1)]
    K --> L[Barramento 5V (P2 Pin 2)]
    L --> M[GND]
    D --> N[TP5100 (U12) VIN]
    E --> O[TP5100 (U12) BAT]
    N --> P[TP5100]
    O --> P
    P -->|CHRG#| Q[LED Azul (LED2) via R5 330Ω]
    P -->|STDBY#| R[LED Verde (LED1) via R6 330Ω]
    A -->|GND| M
    H --> M
    J --> M
```

### Readme para Documentação

# Documentação do Circuito de Alimentação USB/Bateria

## Descrição Geral
Este circuito foi projetado para fornecer uma fonte de alimentação híbrida, utilizando uma entrada USB de 5V e duas baterias 18650 conectadas em paralelo como backup. O sistema prioriza a alimentação via USB quando conectado, gerando um barramento de 5V e uma saída regulada de 3.3V. Quando a USB é desconectada, a alimentação é automaticamente transferida para a bateria, mantendo a saída de 3.3V. O carregamento das baterias ocorre apenas quando o sistema está desligado e a USB está conectada.

## Componentes Principais
- **Micro USB-B 5P C40942**: Entrada de alimentação 5V.
- **Fuse 3A (U4)**: Proteção contra sobrecorrente na entrada USB.
- **D1 e D2 (1N5819W SL)**: Diodos Schottky para comutação automática entre USB e bateria.
- **TP5100 (U12)**: Controlador de carga para baterias 18650 em modo 1S.
- **LDO (MCP1700-3.3)**: Regulador de tensão para gerar 3.3V a partir de V_SYS.
- **Jumper P2 (H-Fem-12)**: Conector para barramento 5V e interruptor manual de 3.3V.
- **Baterias 18650 (BH-18650-A1BJ021)**: Duas células em paralelo para ~3.7V nominais.
- **LEDs**: Azul (LED2) para carregamento, Verde (LED1) para carga completa.
- **Resistores**: R5 e R6 (330Ω) para limitar corrente dos LEDs.

## Funcionamento
- **USB Conectada, Sistema Ligado**: A USB (5V) alimenta V_SYS via D1 (~4.7V após queda), o LDO gera 3.3V, e a bateria é isolada por D2. O TP5100 carrega as baterias.
- **USB Desconectada, Sistema Ligado**: A bateria alimenta V_SYS via D2 (~3.4V após queda), e o LDO mantém 3.3V.
- **Sistema Desligado (Jumper P2 Aberto)**: Sem saída 3.3V, mas o TP5100 carrega as baterias se a USB estiver conectada.
- **Barramento 5V**: Disponível apenas com USB conectada, via P2.

## Instruções de Montagem
1. Conecte o Micro USB ao fusível 3A (U4) e ao ânodo de D1.
2. Ligue o cátodo de D1 e D2 a V_SYS, com o ânodo de D2 conectado às baterias em paralelo.
3. Conecte V_SYS à entrada do LDO e a saída (3.3V_OUT) ao jumper P2.
4. Ligue as baterias ao TP5100 (pino BAT) e configure o modo 1S via jumper U11.
5. Conecte os LEDs e resistores conforme o esquema.
6. Adicione capacitores de 1µF na entrada e saída do LDO para estabilização.

## Considerações
- Certifique-se de que as baterias 18650 tenham tensões balanceadas antes de conectar em paralelo.
- Verifique a capacidade do LDO para manter 3.3V com entrada mínima de ~3.4V (dropout baixo é essencial).
- O jumper P2 pode ser substituído por um switch dedicado para maior praticidade.

## Data de Criação
16 de agosto de 2025, 14:42 (horário de Brasília).