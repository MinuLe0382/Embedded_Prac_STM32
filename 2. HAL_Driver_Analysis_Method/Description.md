# HAL_GPIO_WritePin Manual Implementation 

> 데이터 시트를 보면서 하나하나 연구하면서 해야함.
5줄의 코드를 쓰기 위해 많은 연구가 필요

> HAL_init()
![image](https://github.com/user-attachments/assets/25d3af63-2e58-441e-807c-96ea5d2d5504)
위 보드중 하나이면 HAL FLASH PREFETCH BUFFER ENABLE() 에 들어오라는 의미

![image](https://github.com/user-attachments/assets/cc390b30-e6d6-4ef3-b731-8a15c0dc4e75)
접근이 안되는 부분은 위와 같이 검은색으로 직관적으로 표현이 된다.
IDE가 강력한 이유는 이와 같이 디버거를 사용해서 BREAKPOINT를 이용해 쉽게 분석이 가능하다는 것.

위 함수는 아래와 같은 동작을 한다. 이것이 어떤 동작인지 쉽게 알려면
![image](https://github.com/user-attachments/assets/acb607d1-59e0-4dee-890e-79471bb40856)

Expressions 창에 FLASH를 쳐봐라
![image](https://github.com/user-attachments/assets/26e47ef6-884f-4853-aaf6-8776ffed7df4)

![image](https://github.com/user-attachments/assets/b39e841f-7c5a-47d5-b454-d2f1eb18406c)
>> FLASH는 메모리주소 40022000을 접근한다

![image](https://github.com/user-attachments/assets/4a997c84-dd14-479c-9ff9-7153667d0ba0)

즉 , (FLASH->ACR |= FLASH_ACR_PRFTBE) 의 의미는

*(0x40022000) = *(0x40022000) | 16 이다.

그러면 FLASH_TypeDef는 무엇인가?
![image](https://github.com/user-attachments/assets/3e8751ab-e99c-4b57-9bb0-db1daf694cde)
![image](https://github.com/user-attachments/assets/0640fb9b-a470-478b-82ef-98ec74f93d05)

FLASH 구조체 (uint32_t >> unsigned int >> 4바이트) (__IO >> volatile)

40022000

FLASH→ACR (40022000)

FLASH→KEYR (40022004)

40022000 ACR

40022004 KEYR

40022008 OPTKEYR

4002200C SR

40022010 CR

FLASH 라는 큰 덩어리의 구조체에 여러 레지스터가 존재한다. 

컴파일러는 최적화를 한다.
![image](https://github.com/user-attachments/assets/018aa9f5-d0c9-488a-b53f-d419834e1f82)

위 코드에서 실제로 컴파일러는 int a = 12; 만을 실행한다.

원하는 코드를 동작시키기 위해서는 __IO를 붙여 최적화를 방지해야한다.

volatile unsigned int * reg = 40022000;

*reg |= 16;

이제 데이터시트를 확인하면
![image](https://github.com/user-attachments/assets/5cce8b11-26c5-4d69-b80b-ba089d18cf5a)
![image](https://github.com/user-attachments/assets/80600ab7-4ed1-4db5-92c0-6f37e8670ec4)

16은 1 << 4 이므로 PRFTBE를 제어하는 것이다. 이는 플래시 메모리의 프리패치를 활성화 한다.

 프래패치는 os가 자주쓰는 명령어를 미리 메모리에 올려두어서 수행속도를 빠르게 하는 기능임
 >> CS지식을 잘 알고 있어야한다.
