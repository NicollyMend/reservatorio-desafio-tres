#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"

#define LED_AVENIDA_VERDE    GPIO_NUM_2  
#define LED_AVENIDA_AMARELO  GPIO_NUM_4  
#define LED_AVENIDA_VERMELHO GPIO_NUM_5  

#define LED_CONVERSAO_VERDE  GPIO_NUM_18  
#define LED_CONVERSAO_VERMELHO GPIO_NUM_19  

#define LED_RUA_VERDE    GPIO_NUM_21  
#define LED_RUA_AMARELO  GPIO_NUM_22  
#define LED_RUA_VERMELHO GPIO_NUM_23  

#define LED_PEDESTRE_VERDE  GPIO_NUM_26  
#define LED_PEDESTRE_VERMELHO GPIO_NUM_27  

#define BOTAO_PEDESTRE   GPIO_NUM_25  
#define BUZZER GPIO_NUM_32  

void configurar_gpio() {
    gpio_set_direction(LED_AVENIDA_VERDE, GPIO_MODE_OUTPUT);
    gpio_set_direction(LED_AVENIDA_AMARELO, GPIO_MODE_OUTPUT);
    gpio_set_direction(LED_AVENIDA_VERMELHO, GPIO_MODE_OUTPUT);
    gpio_set_direction(LED_CONVERSAO_VERDE, GPIO_MODE_OUTPUT);
    gpio_set_direction(LED_CONVERSAO_VERMELHO, GPIO_MODE_OUTPUT);
    gpio_set_direction(LED_RUA_VERDE, GPIO_MODE_OUTPUT);
    gpio_set_direction(LED_RUA_AMARELO, GPIO_MODE_OUTPUT);
    gpio_set_direction(LED_RUA_VERMELHO, GPIO_MODE_OUTPUT);
    gpio_set_direction(LED_PEDESTRE_VERDE, GPIO_MODE_OUTPUT);
    gpio_set_direction(LED_PEDESTRE_VERMELHO, GPIO_MODE_OUTPUT);
    gpio_set_direction(BOTAO_PEDESTRE, GPIO_MODE_INPUT);
    gpio_set_pull_mode(BOTAO_PEDESTRE, GPIO_PULLUP_ONLY);
    gpio_set_direction(BUZZER, GPIO_MODE_OUTPUT);
}

void bip_sinal_pedestre() {
    for (int i = 0; i < 10; i++) {  
        gpio_set_level(BUZZER, 1);
        vTaskDelay(pdMS_TO_TICKS(300));  
        gpio_set_level(BUZZER, 0);
        vTaskDelay(pdMS_TO_TICKS(300));
    }
}

void controlar_semaforo() {
    int pedestre_solicitou = 0;

    while (1) {
        pedestre_solicitou = !gpio_get_level(BOTAO_PEDESTRE);

        gpio_set_level(LED_AVENIDA_VERDE, 1);
        gpio_set_level(LED_AVENIDA_AMARELO, 0);
        gpio_set_level(LED_AVENIDA_VERMELHO, 0);
        gpio_set_level(LED_CONVERSAO_VERDE, 0);
        gpio_set_level(LED_CONVERSAO_VERMELHO, 1);
        gpio_set_level(LED_RUA_VERDE, 0);
        gpio_set_level(LED_RUA_AMARELO, 0);
        gpio_set_level(LED_RUA_VERMELHO, 1);
        gpio_set_level(LED_PEDESTRE_VERDE, 0);
        gpio_set_level(LED_PEDESTRE_VERMELHO, 1);
        vTaskDelay(pdMS_TO_TICKS(pedestre_solicitou ? 5000 : 10000));

        gpio_set_level(LED_AVENIDA_VERDE, 0);
        gpio_set_level(LED_AVENIDA_AMARELO, 1);
        vTaskDelay(pdMS_TO_TICKS(2000));

        gpio_set_level(LED_AVENIDA_AMARELO, 0);
        gpio_set_level(LED_AVENIDA_VERMELHO, 1);
        gpio_set_level(LED_CONVERSAO_VERDE, 1);
        gpio_set_level(LED_CONVERSAO_VERMELHO, 0);
        vTaskDelay(pdMS_TO_TICKS(5000));

        gpio_set_level(LED_CONVERSAO_VERDE, 0);
        gpio_set_level(LED_CONVERSAO_VERMELHO, 1);
        gpio_set_level(LED_RUA_VERMELHO, 0);
        gpio_set_level(LED_RUA_VERDE, 1);
        vTaskDelay(pdMS_TO_TICKS(5000));

        gpio_set_level(LED_RUA_VERDE, 0);
        gpio_set_level(LED_RUA_AMARELO, 1);
        vTaskDelay(pdMS_TO_TICKS(2000));

        gpio_set_level(LED_RUA_AMARELO, 0);
        gpio_set_level(LED_RUA_VERMELHO, 1);
        gpio_set_level(LED_PEDESTRE_VERDE, 1);
        gpio_set_level(LED_PEDESTRE_VERMELHO, 0);
        
        bip_sinal_pedestre();
        
        vTaskDelay(pdMS_TO_TICKS(5000));

        gpio_set_level(LED_PEDESTRE_VERDE, 0);
        gpio_set_level(LED_PEDESTRE_VERMELHO, 1);
    }
}

void app_main() {
    configurar_gpio();
    controlar_semaforo();
}
