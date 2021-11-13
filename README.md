# 서보모터 제어 시스템

## 1. 설계 목적

- Servo plant model을 얻는다.
- 모터의 위치제어와 각속도를 조절할 수 있는 제어기를 설계한다.
- Simulink를 통해 Servo plant를 구성하고, 구형파를 입력했을 때 결과를 구하고 디자인 한다.

## 2. 설계 제한

### A. root locus method

### (1) 경제적 조건

- 모터에 인가되는 증폭기의 용량을 가능한 적게 하여 비용을 낮게 한다.(파라미터를 최대한 작게 한다.)

### (2) 생산성과 내구성

- 가능한 빠른 시간 내에 목표치(스텝 입력)에 도달해야 한다.
- 모터를 가능한 오래 사용할 수 있는 제어기를 설계해야 한다.(oscillation이 가능한 적게, 즉 settling time이 적게 설계한다.)

### (3) 기타

- 폐루프의 dominant pole의 damping ratio는 0.3-0.8사이의 값이어야 하며 settling time은 1/40 sec보다 작아야한다.

### B. Frequency domain method

### (1) 생산성 제한조건

- 가능한 빠른 시간 내에 목표치(스텝 입력)에 도달해야 한다.
- ramp input에 대해 가능한 오차가 적어야 한다.

### (2) 안정성

- 가능한 phase margin이 30도 이상 커야한다.

## 3. 설계결론

### A. root locus method

PI 제어기의 코드는 아래와 같습니다.

```matlab
KP = 7;
KI = 7;
p = 288.4 * [KP KI];
q = [1 40.41 0];
gs = tf(p, q);
disp(['KP = ',num2str(para(i,1)),',','KI = ', num2str(para(i,2))]);
sys = feedback(gs, [1])
rlocus(sys)
step(sys)
stepinfo(sys)
damp(sys)
```

근궤적도를 보면 극점은 좌반면에 있고 영점은 원점에 있으므로 안정적인 시스템입니다. 그리고 계단 입력을 보면 SettlingTime이 늦긴 하지만 목표값에 도달한 것을 볼 수 있습니다. 그러므로 PI 제어기가 적절한 제어기로 보입니다.

```matlab
sys = feedback(gs, [1])

sys =
 
     2019 s + 2019
  -------------------
  s^2 + 2059 s + 2019
 
연속시간 전달 함수입니다.
```

```matlab
rlocus(sys)
```

![RootLocus](https://user-images.githubusercontent.com/75241500/141605233-bc4dba31-176b-4745-81a8-03960e429ccb.png)


```matlab
step(sys)
```

![StepResponse](https://user-images.githubusercontent.com/75241500/141605117-b2b29b62-756d-47b6-afa5-2a0dee89a918.png)



```matlab
stepinfo(sys)

ans = 다음 필드를 포함한 struct:
        RiseTime: 0.0012
    SettlingTime: 0.0034
     SettlingMin: 0.9029
     SettlingMax: 0.9811
       Overshoot: 0
      Undershoot: 0
            Peak: 0.9811
        PeakTime: 0.0150

```

```matlab
damp(sys)

    극점           감쇠             주파수            시정수
                           (rad/seconds)    (seconds)

 -9.81e-01     1.00e+00       9.81e-01       1.02e+00
 -2.06e+03     1.00e+00       2.06e+03       4.86e-04
```

## B. Frequency domain method

A의 결과를 토대로 PI제어기를 사용하면 목표치에 빠르게 도달할 수 있고 안정도도 뛰어났다. 그럼 ramp input과 bode plot 대해 관찰해 보자.

```matlab
KP = 7;
KI = 7;
p = 288.4 * [KP KI];
q = [1 40.41 0];
gs = tf(p, q);
sys = feedback(gs, [1])
t = 0:0.04:8;
u = max(0,min(t-1,1));
lsim(sys,u,t)
margin(sys)
```

ramp input을 봤을 때 입력 신호과 유사한 것을 볼 수 있다. 또한 bode plot에서도 phase margin이 180이상이므로 안정한 것을 볼 수 있다.

```matlab
lsim(sys,u,t)
```

![RampResponse](https://user-images.githubusercontent.com/75241500/141605254-6dadf065-0b97-4dea-8aa7-47831970f3fb.png)


```matlab
margin(sys)
```
![BodeDiagram](https://user-images.githubusercontent.com/75241500/141605265-da0bdf65-9800-44a0-83be-2483d8ab3f41.png)
