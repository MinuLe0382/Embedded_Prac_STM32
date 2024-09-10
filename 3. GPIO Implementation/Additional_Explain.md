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
