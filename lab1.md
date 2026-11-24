
# 실습 1. LED 5초 간격 점멸

## 1. 실습 목표
STM32의 GPIO 출력 기능을 사용하여 내장 LED를 5초 간격으로 켜고 끈다.

## 2. 핀맵

| 핀 | 설정 | 용도 |
|---|---|---|
| PC13 | GPIO_Output | 내장 LED |
| PA13 | SWDIO | ST-LINK |
| PA14 | SWCLK | ST-LINK |

> BlackPill 내장 LED는 Active-Low 방식이다.
> PC13 = LOW → LED ON
> PC13 = HIGH → LED OFF

## 3. CubeMX 설정

- MCU: STM32F411CEU6
- System Core → SYS → Debug: Serial Wire
- PC13: GPIO_Output
- GPIO Output Level: High
- GPIO Mode: Push-Pull
- Pull-up/Pull-down: No Pull
- GPIO Speed: Low
- Clock: HSI 16MHz

설정 후 `GENERATE CODE` 실행.

## 4. CubeIDE 코드

파일: `Core/Src/main.c`

위치: `while(1)` 내부의 `USER CODE BEGIN 3`

```c
/* USER CODE BEGIN 3 */

HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
HAL_Delay(5000);
```

기존 CubeMX의 `USER CODE END 3` 주석은 그대로 유지한다.

## 5. 주요 함수

| 함수 | 설명 |
|---|---|
| HAL_GPIO_TogglePin() | GPIO 출력 상태 반전 |
| HAL_Delay(5000) | 5000ms(5초) 대기 |
| GPIO_PIN_13 | GPIO 포트의 13번 핀 |

## 6. 예상 결과

1. LED가 켜진 상태로 5초 유지
2. LED가 꺼진 상태로 5초 유지
3. 위 동작 반복

전체 점멸 주기는 약 10초이다.
