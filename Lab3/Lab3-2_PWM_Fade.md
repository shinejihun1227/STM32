
# 실습 3-2. PWM LED Fade In / Fade Out

## 1. 실습 목표

TIM1 PWM의 듀티비를 점진적으로 증가·감소시켜 LED가 서서히 밝아졌다가 어두워지는 효과를 구현한다.

**학습 내용**
- PWM 듀티비 제어
- CCR 값 변경
- `for` 반복문
- Fade In / Fade Out
- CPU와 Timer의 역할 구분

## 2. 준비물

- STM32F411CEU6 BlackPill
- ST-LINK
- 외부 LED
- 1kΩ 저항
- 브레드보드 및 점퍼선

## 3. 핀맵

| 핀 | CubeMX 설정 | 연결 대상 |
|---|---|---|
| PA8 | TIM1_CH1 | 외부 LED |
| GND | GND | 외부 LED 접지 |
| PA13 / PA14 | Serial Wire | ST-LINK |

### LED 연결

```text
PA8 → 1kΩ 저항 → LED (+) → LED (-) → GND
```

실습 3-1과 동일한 배선을 사용한다.

## 4. STM32CubeMX 설정

실습 3-1의 TIM1 설정을 그대로 유지한다.

| 항목 | 값 |
|---|---|
| Clock Source | Internal Clock |
| Channel1 | PWM Generation CH1 |
| Prescaler | 15 |
| Counter Mode | Up |
| Counter Period | 999 |
| PWM Mode | PWM Mode 1 |
| Pulse | 0 (초기값 권장) |
| Output Polarity | High |

### PWM 조건

```text
Timer Clock = 16MHz
Prescaler = 15
ARR = 999

PWM Frequency = 1kHz
PWM Period = 1ms
```

추가 주변장치 설정은 필요하지 않다.

## 5. CubeIDE 코드

**수정 파일:** `Core/Src/main.c`

### STEP 1. PWM 출력 시작

**교체 위치:** `/* USER CODE BEGIN 2 */` ~ `/* USER CODE END 2 */`

```c
/* USER CODE BEGIN 2 */

  // TIM1 Channel 1 PWM 시작
  HAL_TIM_PWM_Start(
      &htim1,
      TIM_CHANNEL_1
  );

/* USER CODE END 2 */
```

### STEP 2. Fade In / Fade Out

**교체 위치:** `/* USER CODE BEGIN 3 */` ~ `/* USER CODE END 3 */`

```c
/* USER CODE BEGIN 3 */

  // Fade In: LED 점점 밝아지기
  for (uint32_t duty = 0; duty <= 1000; duty += 10)
  {
      __HAL_TIM_SET_COMPARE(
          &htim1,
          TIM_CHANNEL_1,
          duty
      );

      HAL_Delay(10);
  }

  // Fade Out: LED 점점 어두워지기
  for (int32_t duty = 1000; duty >= 0; duty -= 10)
  {
      __HAL_TIM_SET_COMPARE(
          &htim1,
          TIM_CHANNEL_1,
          duty
      );

      HAL_Delay(10);
  }

}
/* USER CODE END 3 */
```

## 6. 코드 설명

### Fade In

```c
for (uint32_t duty = 0; duty <= 1000; duty += 10)
```

CCR 값을 0부터 1000까지 10씩 증가시킨다.

```text
CCR: 0 → 10 → 20 → ... → 990 → 1000
Duty: 0% → 1% → 2% → ... → 99% → 100%
```

각 단계에서 10ms를 대기한다.

따라서 약 1초에 걸쳐 LED가 점점 밝아진다.

### Fade Out

```c
for (int32_t duty = 1000; duty >= 0; duty -= 10)
```

CCR 값을 1000부터 0까지 10씩 감소시킨다.

```text
CCR: 1000 → 990 → 980 → ... → 10 → 0
Duty: 100% → 99% → 98% → ... → 1% → 0%
```

LED가 약 1초 동안 점점 어두워진다.

### __HAL_TIM_SET_COMPARE()

CCR 값을 변경해 TIM1 PWM의 High 유지 시간을 조절한다.

PWM 주파수는 1kHz로 유지된다.

## 7. 예상 동작

| 단계 | CCR | Duty Cycle | LED |
|---|---|---|---|
| 시작 | 0 | 0% | OFF |
| 밝아지는 중 | 250 | 25% | 어두움 |
| 중간 | 500 | 50% | 중간 |
| 밝아지는 중 | 750 | 75% | 밝음 |
| 최대 | 1000 | 100% | 최대 |
| 어두워지는 중 | 750 | 75% | 밝음 |
| 중간 | 500 | 50% | 중간 |
| 어두워지는 중 | 250 | 25% | 어두움 |
| 종료 | 0 | 0% | OFF |

### 전체 반복 시간

```text
Fade In  ≈ 1초
Fade Out ≈ 1초

전체 반복 ≈ 2초
```

## 8. 실습 결과 확인

1. Build 후 최신 ELF 파일을 업로드한다.
2. 외부 LED가 처음에는 꺼져 있는지 확인한다.
3. LED 밝기가 서서히 증가하는지 확인한다.
4. 최대 밝기 이후 서서히 감소하는지 확인한다.
5. 같은 동작이 계속 반복되는지 확인한다.

## 9. 핵심 정리

- PWM 출력은 TIM1 하드웨어가 생성한다.
- CPU는 CCR 값을 점진적으로 변경한다.
- CCR 변경에 따라 듀티비가 달라진다.
- 듀티비가 증가하면 외부 LED의 평균 밝기가 증가한다.
- `for` 반복문으로 점진적인 밝기 변화를 구현할 수 있다.
