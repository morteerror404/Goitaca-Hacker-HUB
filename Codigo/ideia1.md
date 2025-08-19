## **1️⃣ RP2040 – Controle de Periféricos e Interface**

### Bibliotecas sugeridas (C/C++ com SDK do Pico)

```c
#include "pico/stdlib.h"
#include "hardware/spi.h"
#include "hardware/i2c.h"
#include "hardware/adc.h"
#include "ssd1306.h"   // Biblioteca OLED
#include <stdio.h>

// Pinos
#define SPI_MISO 0
#define SPI_MOSI 3
#define SPI_SCK  2
#define SPI_CS   1
#define OLED_SDA 4
#define OLED_SCL 5

// Modos de operação
typedef enum {
    MODE_SDR,
    MODE_JAMMER,
    MODE_DERRUTE
} operation_mode_t;

operation_mode_t current_mode = MODE_SDR;

// Função de leitura de ADC (ex.: IF_OUT do mixer)
uint16_t read_adc(uint adc_channel) {
    adc_select_input(adc_channel);
    return adc_read();
}

// Função para atualizar display OLED
void display_mode(operation_mode_t mode) {
    ssd1306_clear();
    switch(mode) {
        case MODE_SDR:
            ssd1306_draw_string(0,0,"Modo: SDR");
            break;
        case MODE_JAMMER:
            ssd1306_draw_string(0,0,"Modo: Jammer");
            break;
        case MODE_DERRUTE:
            ssd1306_draw_string(0,0,"Modo: Derraute");
            break;
    }
    ssd1306_display();
}

// Loop principal do RP2040
void main_loop() {
    while(1) {
        // Leitura ADC do IF_OUT
        uint16_t if_signal = read_adc(0);
        // Atualização da interface
        display_mode(current_mode);
        sleep_ms(100);
    }
}

int main() {
    stdio_init_all();
    i2c_init(i2c0, 400*1000);
    spi_init(spi0, 1*1000*1000);
    ssd1306_init();
    main_loop();
}
```

---

## **2️⃣ ESP32-S3 – Controle de Modo, Sequenciamento e IR**

### Bibliotecas sugeridas (ESP-IDF)

```c
#include "driver/gpio.h"
#include "driver/rmt.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

#define PIN_IR_TX 23
#define PIN_MODE_BUTTON 21
#define PIN_JAMMER_ENABLE 10
#define PIN_JAMMER_RESET 11

typedef enum {
    MODE_SDR,
    MODE_JAMMER,
    MODE_DERRUTE
} operation_mode_t;

operation_mode_t current_mode = MODE_SDR;

// Configuração RMT para IR
void ir_init() {
    rmt_config_t tx_config = RMT_DEFAULT_CONFIG_TX(PIN_IR_TX, RMT_CHANNEL_0);
    rmt_config(&tx_config);
    rmt_driver_install(RMT_CHANNEL_0, 0, 0);
}

// Função de mudança de modo via botão
void check_mode_button() {
    if(gpio_get_level(PIN_MODE_BUTTON) == 0) {
        current_mode++;
        if(current_mode > MODE_DERRUTE) current_mode = MODE_SDR;
        vTaskDelay(200 / portTICK_PERIOD_MS);
    }
}

// Sequenciamento do Jammer
void jammer_sequence() {
    if(current_mode == MODE_JAMMER) {
        gpio_set_level(PIN_JAMMER_ENABLE, 1);
        gpio_set_level(PIN_JAMMER_RESET, 0);
    } else {
        gpio_set_level(PIN_JAMMER_ENABLE, 0);
        gpio_set_level(PIN_JAMMER_RESET, 1);
    }
}

void app_main(void)
{
    gpio_reset_pin(PIN_MODE_BUTTON);
    gpio_set_direction(PIN_MODE_BUTTON, GPIO_MODE_INPUT);

    gpio_reset_pin(PIN_JAMMER_ENABLE);
    gpio_set_direction(PIN_JAMMER_ENABLE, GPIO_MODE_OUTPUT);

    gpio_reset_pin(PIN_JAMMER_RESET);
    gpio_set_direction(PIN_JAMMER_RESET, GPIO_MODE_OUTPUT);

    ir_init();

    while(1) {
        check_mode_button();
        jammer_sequence();
        vTaskDelay(50 / portTICK_PERIOD_MS);
    }
}
```

---

## **3️⃣ Conexão entre RP2040 e ESP32-S3**

* **UART** para transmissão de dados de telemetria:

  * RP2040 TX → ESP32 RX
  * RP2040 RX ← ESP32 TX
* **GPIO de controle**:

  * ESP32 controla EN/RESET do Jammer/SDR
* **SPI / I²C compartilhado**:

  * RP2040 → AD9361 / RFFC5072A
  * ESP32 → não precisa acessar SPI dos RFs, apenas sequenciamento

---

## **4️⃣ Interface Gráfica do Usuário**

* Tela OLED 128x64
* Menu hierárquico simples:

  ```
  Modo Atual: SDR
  [1] SDR
  [2] Jammer
  [3] Derraute
  ```
* Botão muda o modo (debounce implementado no ESP32)
* LEDs indicam:

  * Verde → SDR
  * Amarelo → Derraute
  * Vermelho → Jammer
