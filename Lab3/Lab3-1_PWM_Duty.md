
# 실습 3-1. PWM 듀티비 25% ↔ 75% 반복

## 1. 실습 목표

STM32 TIM1 타이머를 사용하여 PWM 신호를 생성하고, 듀티비를 25%와 75%로 반복 변경한다.

외부 LED를 이용하여 듀티비에 따른 밝기 차이를 확인한다.

**학습 내용**
- PWM(Pulse Width Modulation)
- Timer Prescaler
- Counter Period
- CCR(Capture/Compare Register)
- Duty Cycle
- `HAL_TIM_PWM_Start()`
- `__HAL_TIM_SET_COMPARE()`

## 2. 준비물

- STM32F411CEU6 BlackPill
- ST-LINK
- 외부 LED 1개
- 1kΩ 저항 1개
- 브레드보드 및 점퍼선

## 3. 핀맵

| 핀 | CubeMX 설정 | 연결 대상 |
|---|---|---|
| PA8 | TIM1_CH1 | 외부 LED |
| GND | GND | LED 캐소드(-) |
| PA13 / PA14 | Serial Wire | ST-LINK |

### 외부 LED 연결

```text
BlackPill PA8
     |
   1kΩ 저항
     |
  LED (+)
  LED (-)
     |
    GND
```

> PWM 신호는 PA8에서 출력된다.
> PC13 내장 LED는 TIM1_CH1 PWM에 연결되지 않는다.

## 4. STM32CubeMX 설정

### Timers → TIM1

- Clock Source: Internal Clock
- Channel1: PWM Generation CH1

### Parameter Settings

| 항목 | 설정값 |
|---|---|
| Prescaler | 15 |
| Counter Mode | Up |
| Counter Period | 999 |
| Repetition Counter | 0 |
| PWM Mode | PWM Mode 1 |
| Pulse | 250 |
| Output Polarity | High |

### PA8

- Pin Function: TIM1_CH1
- Alternate Function: AF1_TIM1

### Clock Configuration

- HSI: 16MHz
- SYSCLK: 16MHz
- TIM1 Clock: 16MHz

설정 후 `GENERATE CODE` 실행.

## 5. PWM 주파수 계산

```text
Timer Clock = 16MHz

PSC = 15
ARR = 999

Counter Clock
= 16,000,000 / (15 + 1)
= 1,000,000Hz

PWM Frequency
= 1,000,000 / (999 + 1)
= 1,000Hz
```

PWM 주파수는 1kHz이고 한 주기는 1ms이다.

## 6. CubeIDE 코드

**수정 파일:** `Core/Src/main.c`

### STEP 1. PWM 출력 시작

**교체 위치:** `/* USER CODE BEGIN 2 */` ~ `/* USER CODE END 2 */`

```c
/* USER CODE BEGIN 2 */

  // TIM1 Channel 1 PWM 출력 시작
  HAL_TIM_PWM_Start(
      &htim1,
      TIM_CHANNEL_1
  );

/* USER CODE END 2 */
```

### STEP 2. 듀티비 반복 변경

**교체 위치:** `/* USER CODE BEGIN 3 */` ~ `/* USER CODE END 3 */`

```c
/* USER CODE BEGIN 3 */

  // PWM Duty Cycle 25%
  __HAL_TIM_SET_COMPARE(
      &htim1,
      TIM_CHANNEL_1,
      250
  );

  HAL_Delay(1000);

  // PWM Duty Cycle 75%
  __HAL_TIM_SET_COMPARE(
      &htim1,
      TIM_CHANNEL_1,
      750
  );

  HAL_Delay(1000);

}
/* USER CODE END 3 */
```

## 7. 코드 설명

### HAL_TIM_PWM_Start()

```c
HAL_TIM_PWM_Start(
    &htim1,
    TIM_CHANNEL_1
);
```

TIM1 Channel 1의 PWM 출력을 시작한다.

### __HAL_TIM_SET_COMPARE()

```c
__HAL_TIM_SET_COMPARE(
    &htim1,
    TIM_CHANNEL_1,
    250
);
```

CCR1의 비교값을 250으로 설정한다.

PWM Mode 1, Polarity High 기준으로 듀티비는 다음과 같다.

```text
Duty Cycle
= CCR / (ARR + 1) × 100%

CCR = 250 → 25%
CCR = 750 → 75%
```

### HAL_Delay()

```c
HAL_Delay(1000);
```

듀티비를 설정한 후 1초간 유지한다.

## 8. 예상 동작

| 시간 | CCR | 듀티비 |
|---|---|---|
| 0~1초 | 250 | 25% |
| 1~2초 | 750 | 75% |
| 2~3초 | 250 | 25% |
| 3~4초 | 750 | 75% |

외부 LED의 밝기가 약한 상태와 밝은 상태로 1초마다 바뀐다.

## 9. 핵심 정리

- TIM1 하드웨어가 PWM 신호를 생성한다.
- PSC와 ARR이 PWM 주파수를 결정한다.
- CCR은 PWM의 듀티비를 결정한다.
- CPU는 CCR을 변경하고 TIM1이 실제 파형을 출력한다.
