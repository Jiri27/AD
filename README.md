#include <stdbool.h>
#include <stm8s.h>
#include <stdio.h>
#include "main.h"
#include "milis.h"
//#include "delay.h"
#include "uart1.h"
#include "daughterboard.h" 
#include "adc_helper.h"

void init(void)
{
    CLK_HSIPrescalerConfig(CLK_PRESCALER_HSIDIV1);      // taktovani MCU na 16MHz
    init_milis();

    init_uart1();
    
    ADC2_SchmittTriggerConfig(ADC2_SCHMITTTRIG_CHANNEL14, DISABLE);
    ADC2_SchmittTriggerConfig(ADC2_SCHMITTTRIG_CHANNEL15, DISABLE);
    
    ADC2_PrescalerConfig(ADC2_PRESSEL_FCPU_D4);     //device 4   = D4
    //ZVOLÍM ZAROVNANI VYSLEDKU -> VĚTŠINOU DO PRAVA
    ADC2_AlignConfig(ADC2_ALIGN_RIGHT);
    //NASTAVÍM MULTIPLEXER NA NĚKTERÝ KANÁL
    ADC2_Select_Channel(ADC2_CHANNEL_14);
    //ROZBĚHNU ADC2
    ADC2_Cmd(ENABLE);
    //POČKÁM CHVILI

}



int main(void){
  
    uint32_t time = 0;
    uint16_t vref, vtemp, temp;
    init();

    while (1) {
        if (milis() - time > 1000) {
            time = milis();
            ADC_get(CHANNEL_VTEMP);
            ADC_get(CHANNEL_VTEMP);
            ADC_get(CHANNEL_VTEMP);
            ADC_get(CHANNEL_VTEMP);
            ADC_get(CHANNEL_VTEMP);
            vtemp= (ADC_get(CHANNEL_VTEMP)*5000L + 512) / 1023;
            vref= ((uint32_t) ADC_get(CHANNEL_VREF)*5000 +512) /1023 ;
            temp = (100L*vtemp - 40000L + 195/2) / 195;
            printf("%u mV %u mV T = %u,%u ˚C\n", vtemp, vref, temp/10, temp%10);
        }       // pyserial-miniterm /dev/ttyACM0 115200  <-- rychlost
    }
}

/*-------------------------------  Assert -----------------------------------*/
#include "__assert__.h"
