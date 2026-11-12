# ULTRASONIC_Project

## HC-SR04 원리

![HC-ST04](IMG/HC-SR04(3).png) <br>
![HC-ST04](IMG/HC-SR04(4).png) <br>
![HC-ST04](IMG/HC-SR04(1).png) <br>
![HC-ST04](IMG/HC-SR04(2).png) <br>

- TOF(Time-of-Flight)
![HC-ST04](IMG/HC-SR04(5).png) <br>

```
음속 = 약 340 m/s = 0.34 mm/µs
거리 = 음속 × 시간 / 2  (왕복이므로)
     = 0.34 × t(µs) / 2 = 0.17 × t(µs)  [mm]
```

##CUBE MX 설정

![HC-ST04](IMG/HC-SR04(10).png) <br>

##코드/시뮬레이션
###TRIG 신호 측정

```
/* USER CODE BEGIN PD */
#define HIGH 1
#define LOW 0
long unsigned int echo_time;
int dist;
/* USER CODE END PD */

/* USER CODE BEGIN 0 */
void timer_start(void)
{
   HAL_TIM_Base_Start(&htim1);
}

void delay_us(uint16_t us)
{
   __HAL_TIM_SET_COUNTER(&htim1, 0);              // 카운터 0 부터 시작
   while((__HAL_TIM_GET_COUNTER(&htim1)) < us);   // 1 count = 1 µs
}

void trig(void)
{
   HAL_GPIO_WritePin(TRIGER1_GPIO_Port, TRIGER1_Pin, HIGH);
   delay_us(10);                                  // 10 µs HIGH
   HAL_GPIO_WritePin(TRIGER1_GPIO_Port, TRIGER1_Pin, LOW);
}
/* USER CODE END 0 */
```

###HAL_Delay 수정
```
  while (1)
  {
    trig();
    //HAL_Delay(200);
    delay_us(200);
  }
```

![HC-ST04](IMG/HC-SR04(1).jpg) <br>

###ECHO 측정
```
  while (1)
  {
    trig();
    //HAL_Delay(200);
    delay_us(200);
  }
```
![HC-ST04](IMG/HC-SR04(2).jpg) <br>
대상이 센서와 가까울 때
![HC-ST04](IMG/HC-SR04(3).jpg) <br>
대상이 센서와 멀 때

###거리 측정 시뮬레이션(PUTTY 사용)
![HC-ST04](IMG/HC-SR04(11).jpg) <br>

```
/* USER CODE BEGIN Includes */
#include <stdio.h>
/* USER CODE END Includes */
```
*studio.h 함수가 없을 시 아래와 같은 에러가 나타나며 printf() 함수를 사용하지 못함. printf() 함수가 선언된 헤더 파일을 불러와야 사용할 수 있음.

../Core/Src/main.c:175:3: note: include '<stdio.h>' or provide a declaration of 'printf'

```
/* USER CODE BEGIN 0 */
#ifdef __GNUC__
#define PUTCHAR_PROTOTYPE int __io_putchar(int ch)
#else
#define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)
#endif /* __GNUC__ */

PUTCHAR_PROTOTYPE
{
  if (ch == '\n')
    HAL_UART_Transmit (&huart2, (uint8_t*) "\r", 1, 0xFFFF);
  HAL_UART_Transmit (&huart2, (uint8_t*) &ch, 1, 0xFFFF);

  return ch;
}

uint32_t echo(void) {
    uint32_t start_tick, start_time, end_time;

    // ECHO HIGH 대기 (최대 10ms)
    start_tick = HAL_GetTick();
    while (HAL_GPIO_ReadPin(ECHO1_GPIO_Port, ECHO1_Pin) == GPIO_PIN_RESET) {
        if ((HAL_GetTick() - start_tick) > 10) return 0;
    }
    start_time = __HAL_TIM_GET_COUNTER(&htim1);

    // ECHO LOW 대기 (최대 30ms)
    start_tick = HAL_GetTick();
    while (HAL_GPIO_ReadPin(ECHO1_GPIO_Port, ECHO1_Pin) == GPIO_PIN_SET) {
        if ((HAL_GetTick() - start_tick) > 30) return 0;
    }
    end_time = __HAL_TIM_GET_COUNTER(&htim1);

    return end_time - start_time;
}
/* USER CODE END 0 */
```

```
  /* USER CODE BEGIN 2 */
  timer_start();
  printf("Ranging with HC-SR04\n");
  /* USER CODE END 2 */

  while (1)
  {
    trig();
    echo_time = echo();
    if (echo_time != 0) {
        dist = (int)(17 * echo_time / 100);
        printf("Distance = %d(mm)\n", dist);
    }
    else printf("Out of Range!\n");
    HAL_Delay(250);
  }
```

##PUTTY 결과

![HC-ST04](IMG/HC-SR04(7).PNG) <br>
![HC-ST04](IMG/HC-SR04(8).PNG) <br>
