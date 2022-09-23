# STM32F103_ADC_DMA
Using ADC trigged by TIM3 Update Event and DMA

O exercício hoje consiste em usar o Timer3 como TRIGGER (disparador) para a conversão ADC1 num instante de tempo pré-determinado, e para inverter o estado de um LED instantes antes dessa conversão ADC iniciar.
A figura abaixo explica melhor o que será feito.
![TIM3](https://user-images.githubusercontent.com/114233216/191957019-21e098ff-0f1b-481a-91f7-9467e5ae4fba.png)

O Timer3 foi configurado para operar numa frequência de 8MHz. Isso significa que ele incrementa em uma unidade a cada 125nS.
Além disso, ele foi configurado para contar de 0 até 999, resultando em 1000 incrementos. Dessa forma, o intervalo de tempo total para o Timer3 iniciar e terminar um ciclo de contagem é de 125nS * 1000 = 125uS.

Quando o contador do TIM3 está em 998, ocorre um evento Toggle on Match na saída de comparação 1 deste timer. Essa saída é representada fisicamente na blupill (STM32F103C8T6) no pino PA6. 

Quando o contador do TIM3 está em 999 ocorre um Update Event. Nesse momento o trigger é disparado e uma conversão ADC via DMA é iniciada. Após isso o TIM3 reseta e começa a contagem novamente, indefinidamente. Assim, toda vez que ocorre um Update Event no TIM3, uma conversão ADC é realizada e seu resultado é colcoado em um buffer via DMA, de forma que o núcleo do STM32 fica livre para realizar outras operações.

O Toggle on Match na porta PA6 é usado para observar via osciloscópio o extao momento em que cada conversão ADC é iniciada. (Na verdade o toggle on match ocorre um ciclo de coclock antes do início da conversão ADC).

No código que eu criei, há um buffer com 10 posições de memória, o que nos possibilita armazenar o resultado de 10 conversões ADC. Assim, a cada 10 conversões ADC o buffer é totalmente preenchido. Para observar isso concetei um LED à porta PB7. Esse LED tem seu estado invertido toda vez que o buffer é preenchido. 

Usei um osciloscópio para analisar essas formas de onda.
![WhatsApp Image 2022-09-23 at 08 22 45](https://user-images.githubusercontent.com/114233216/191957668-b03443ad-00c9-4997-8321-65dcb7733db5.jpeg)

A forma de onda azul (tanto a borda superior como a borda inferior) indica o início de cada conversão ADC. 

A forma de onda amarela (tanto a borda superior como a borda inferior) indica que o buffer foi totalmente preenchido, ou seja, ele tem o resultado de 10 conversões analógicas.


Para determinar a inversão do estado do LED em PB7 foi usada uma função para indicar o fim das conversões ADC:

void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc1){
HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_7);
}
