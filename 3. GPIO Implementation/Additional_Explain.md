# GPIO가 제어가 된 이유에 관하여

![image](https://github.com/user-attachments/assets/582b5f37-4b33-4b4d-a93c-41d3f7925f90)
GPIO Port C의 주소를 이용했다.  

더 디테일한 레지스터는 다음과 같다.
![image](https://github.com/user-attachments/assets/5c4000c7-64d6-4c29-a7a1-13d4c36b97c7)

복잡한 이유는 설정에따라 변하는 값이 있기 떄문이다. 

일단 GPIO가 무엇인지, 어떻게 쓰는지, 어디에 응용하는지 개념을 배운다.  
그리고 GPIO를 지원하는 칩을 찾고 그 칩을 어떻게 제어하는지 배운다. (With Datasheet)  
그래서 소스코드를 제어한다.

빠르게 하려면 예제코드를 보고 역으로 데이터시트를 추적한다. 

![image](https://github.com/user-attachments/assets/38867783-1c79-420b-9621-1538359a41b9)

GPIOC 클록을 제어하려면  

![image](https://github.com/user-attachments/assets/a865c151-a44e-4f98-977e-55cde2db3ba1)
![image](https://github.com/user-attachments/assets/778f4f93-ee6e-497b-8d51-819025541135)
![image](https://github.com/user-attachments/assets/56653d7f-dc41-4582-a4a9-8a5e2479bc81)

클록을 활성화하는 코드를보면

volatile unsigned int * reg = 0x40021018;  
*reg |= 16;  

GPIO init으로 옵션을 설정해주는데  
![image](https://github.com/user-attachments/assets/c95e2c61-381e-4930-bcf3-930681432cb9)
위 비트를 조합해서 설정을 한다.

volatile unsigned int * reg2 = 0x40011004;
*reg2 = (*reg2 & ~(15UL << 20U)) | (3U << 20U);

WritePin으로 핀을 끄고 키고를 제어했다.

![image](https://github.com/user-attachments/assets/81d35f24-a8f8-4bac-a093-62be24a43136)

리셋할때 16칸을 밀어버린 이유는 데이터시트를 보면 알 수 있다.  
