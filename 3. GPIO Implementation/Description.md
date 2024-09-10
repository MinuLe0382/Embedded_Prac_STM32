![image](https://github.com/user-attachments/assets/efb72114-9398-4c42-9f43-8dde859e171b)

---

![image](https://github.com/user-attachments/assets/ab36816d-ed14-4d83-aabd-3a657684c78d)
 
#define __HAL_RCC_GPIOC_CLK_ENABLE()   do { \  
                                        __IO uint32_t tmpreg; \ << 변수를 선언  
                                        SET_BIT(RCC->APB2ENR, RCC_APB2ENR_IOPCEN);\  << 여기가 제일 중요하다  
                                        /* Delay after an RCC peripheral clock enabling */\  
                                        tmpreg = READ_BIT(RCC->APB2ENR, RCC_APB2ENR_IOPCEN);\ << 변수에 비트를 읽어온다.  
                                        UNUSED(tmpreg); \  
                                      } while(0U)  

                                      
막상 unused를 보면 

#define UNUSED(X) (void)X      /* To avoid gcc/g++ warnings */ 로 아무것도 없다  
단지 경고를 뜨지 않기 위해 쓴다.  

![image](https://github.com/user-attachments/assets/fe99a16b-73c1-49ad-badc-420bdcafabcb)

((REG) |= (BIT)) << OR연산임  

---

SET_BIT(RCC->APB2ENR, RCC_APB2ENR_IOPCEN); 그럼 이건 무슨 연산인가?  
![image](https://github.com/user-attachments/assets/d4fc900a-61b6-47f3-adab-646aeaf9a1b2)

*(0x40021018) |= 16 (1을 옆으로 4칸 밀어서 16이다.)

---

이는 GPIO Ports Clock Enable인데 왜 클록을 킨거지?

## **Clock의 개념**
컴퓨터가 동작하는 기준이다. STM은 외부클록과 MCU내부 클록이 있다. 내부클록은 좀 불안정해서 외부클록을 쓰는데 아래 그림이 외부클록이다. 
크리스탈 부품은 전류를 넣으면 일정하게 클록을 발생시킨다. 
![image](https://github.com/user-attachments/assets/26b7728a-7ed1-453f-a170-5c7a6e51f0c8)


1초에 8000번을 뛴다고할때 장치마다 필요한 클록이 다르다. 빠른애들끼리의 버스에 모아놓고 느린애들끼리 버스에 모아놓는다.

![image](https://github.com/user-attachments/assets/d23b0071-b8d8-40d8-b637-6de42959db1f)
AHB가 빠른버스, APB가 느린버스임

중심이 되는 심장의 클록을 나누어서쓰기도 하고 뻥튀기해서 쓰기도하고 그런다.
GPIOC는 APB로부터 클록을 받는다.

HAL_GPIO_WritePin(GPIO_LED_GPIO_Port, GPIO_LED_Pin, GPIO_PIN_SET); 는 무슨 코드일까?

![image](https://github.com/user-attachments/assets/384c0b15-749b-4d53-973f-46da60cd8b54)

assert param은 아무것도 안함
HAL_GPIO_WritePin은 3개의 매개변수를 받는다.
![image](https://github.com/user-attachments/assets/5d4f91f8-4f18-4c73-95c7-51256effd955)

*(0x40011000), 8192(13임)
1 << 13 C13포트를 제어한다.
GPIO_PIN_SET은 Decimal로 보면 1이다.

if (PinState != GPIO_PIN_RESET) 핀상태가 0이 아니면
GPIOx -> BSRR = GPIO_Pin; GPIO의 BSRR레지스터에 13을 넣는다. 즉 13번째 비트를 컨트롤한다.
(여기서 GPIOx -> BSRR의 주소는 0x40011010)
![image](https://github.com/user-attachments/assets/58c519be-338d-4655-b12e-82603bee075c)

그런데 만일 PinState가 0이면
GPIOx->BSRR = (uint32_t)GPIO_Pin << 16u;
*(0x40011010) =  (8192 << 16); 이며 이는 데이터시트를 봐야 설명 가능하다.
시트를 보면 BSy가 켜는거고 리셋을 하려면 BRy를 켜야하는데 16칸 밀면 되는 것이다.

>> 그러면 여기서 BS13을 다시켠다고 하면 BR13이 1인채로 BS13도 1인가? 데이터시트에 보면 둘다 1인경우 BS13을 우선한다고했다.

일단 여기까지 MX_GPIO_Init()를 직접 구현하면
volatile unsigned int * reg = 0x40021018;
*reg |= 16; 으로 클록을 켜준다.

while문 이전에 
volatile unsigned int * reg = 0x40011010; 을 선언한다. GPIOPORT의 BSRR
while문 안에서

pinwrite를 다음과 같이 바꿀 수 있다.
*reg = 0x2000; (8192랑 같은값) (led 끄는거임)
키는건 *reg = (0x2000 << 16); 회로도를 보면 high일때 led가 꺼지는 것을 기억하라

---
![image](https://github.com/user-attachments/assets/5876273f-c999-47de-ab07-7833b19bdb9c)
일단 GPIO_InitTypeDef GPIO_InitStruct = {0}; 에서 구조체는 다음과 같다.

![image](https://github.com/user-attachments/assets/8e536e25-53d3-45b3-a94b-a36bb0446828)
GPIO_LED_Pin //PC13 1<<13 8192 0x2000
GPIO_MODE_OUTPUT_PP // 1
GPIO_PULLDOWN // 2
GPIO_SPEED_FREQ_HIGH // 3

**HAL_GPIO_Init()**

while문 하나로 끝난다.

while (((GPIO_Init->Pin) >> position) != 0x00u)  
// GPIO_Init에 &GPIO_InitStruct가 들어왔음을 기억하라.  
// GPIO_Init->Pin = 10000000000000 이므로 position(초기값 0 while문 돌때마다 하나씩 증가함)만큼 오른쪽으로 밀라는 의미  
{  
  ![image](https://github.com/user-attachments/assets/6d270e1f-5b0e-4afc-ac24-c73e46c31140)
  ![image](https://github.com/user-attachments/assets/8e76c37e-59d1-478a-87b5-6b7a6e0d58d2)
  ![image](https://github.com/user-attachments/assets/2ed97218-e99e-439c-8ebf-b25c0044887c)
  GPIO_Init->Pin은 고정이 되어있는 상태에서 ioposition과 and 연산을 한다. 첫번째 반복문에서 ioposition은 1이고 iocurrent는 0이다.  
  즉 position이 13은 되어야 ioposition과 iocurrent가 같아진다.  
  ![image](https://github.com/user-attachments/assets/1a2e8ede-c35c-49e0-83b0-c5e121e2ab84)
  switch는 mode값에 따라 동작하는데 목적은 config값을 조절하기 위해서이다. (초기값은 pp이다.)
  ![image](https://github.com/user-attachments/assets/1cef979e-341d-4217-a99e-5a0ea1166ef8)
  ![image](https://github.com/user-attachments/assets/8b2b6acc-c746-4324-99ff-ff8400668a2e)
  일단 config는 3으로 설정이 된다.  
  ![image](https://github.com/user-attachments/assets/7209880c-0fff-4795-add3-792919fdd32f)
  switch문을 빠져나오면 위의 코드가 실행된다.  
  
  ---
  
  configregister에 어떤주소를 넣을지를 결정한다. 일단 1 << 13 과 1 << 8을 비교하는데 8핀 이상이면 &GPIOx->CRH 주소(0x40011004)
  registeroffset에는 20이 들어간다.
  ![image](https://github.com/user-attachments/assets/cd4ae6b0-2b6d-4c4a-bcd6-64255c6ae17e)
  ![image](https://github.com/user-attachments/assets/df4f54e6-d7d9-48ef-aa59-77814c7b5822)
  *(0x40011004) = VAL; 이다. VAL은 (((READ_REG(REG)) & (~(CLEARMASK))) | (SETMASK))
  ((이미 설정된 값) & (~변화시키고 싶은 비트)) | (3 << 20))
  ![image](https://github.com/user-attachments/assets/9395fa46-f977-40e3-9d72-73fee7c111b8)
  위 수식이 코드의 결론임

  GPIO_InitStruct.Pin = GPIO_LED_Pin;  
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;  
  GPIO_InitStruct.Pull = GPIO_PULLDOWN;  
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;  
  HAL_GPIO_Init(GPIO_LED_GPIO_Port, &GPIO_InitStruct);  
  위코드 다 지우고 아래 코드로 대체 가능  
  ![image](https://github.com/user-attachments/assets/7c1e97f7-8f3a-41d3-b880-15d222211adc)
  
  ---
  
  position ++  
}
1이 오른쪽으로 밀리면서 0이될거고 그러면 while문이 종료된다. 
