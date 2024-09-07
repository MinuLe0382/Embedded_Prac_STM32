# Why using Pushpull to control GPIO?

1. PushPull
![image](https://github.com/user-attachments/assets/afbb9a9d-40ec-4953-94a9-118f2a9a7b1b)

두개의 스위치가 있다고 생각해서 Vcc와 GND의 전압을 회로에 인가하는데 사용

2. Open Drain
![image](https://github.com/user-attachments/assets/0b01e07d-d47f-4286-99fd-90b8692622de)

외부회로에 GND 혹은 Float로 유지

![image](https://github.com/user-attachments/assets/f0180f15-faa8-4c2e-b970-6ac08d83a735)
우리가 제어하려는 LED는 3.3V에 연결되어있다. 반대쪽에 0V면 LED가 켜지고 3.3V면 LED가 꺼진다.
따라서 PushPull이 필요하다.
