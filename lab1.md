
# 실습 1. LED 5초 간격 점멸

## 1. 실습 목표

STM32의 GPIO 출력 기능을 이용하여 BlackPill 내장 LED를 5초 간격으로 켜고 끈다.

**학습 내용**
- GPIO Output
- Active-Low LED
- `HAL_GPIO_TogglePin()`
- `HAL_Delay()`
- `while(1)` 반복문

## 2. 준비물

- STM32F411CEU6 BlackPill
- ST-LINK
- USB-C 케이블

내장 LED를 사용하므로 별도의 LED나 저항은 필요하지 않다.

## 3. 핀맵

| 핀 | CubeMX 설정 | 용도 |
|---|---|---|
| PC13 | GPIO_Output | 내장 LED |
| PA13 | SWDIO | ST-LINK 데이터 |
| PA14 | SWCLK | ST-LINK 클록 |
| GND | GND | 공통 접지 |

BlackPill 내장 LED는 Active-Low 방식이다.

- PC13 = LOW → LED ON
- PC13 = HIGH → LED OFF

## 4. STM32CubeMX 설정

### System Core → SYS
- Debug: Serial Wire

### GPIO → PC13
- GPIO Mode: Output Push-Pull
- GPIO Output Level: High
- GPIO Pull-up/Pull-down: No Pull
- GPIO Speed: Low

### Clock Configuration
- Clock Source: HSI
- SYSCLK: 16MHz

설정 후 `GENERATE CODE`를 실행한다.

## 5. CubeIDE 코드

**수정 파일:** `Core/Src/main.c`

**교체 위치:** `/* USER CODE BEGIN 3 */`부터 `/* USER CODE END 3 */`까지 전체

```c
/* USER CODE BEGIN 3 */

  // PC13 LED 상태 반전
  HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);

  // 5000ms = 5초 대기
  HAL_Delay(5000);

}
/* USER CODE END 3 */
```

> `USER CODE BEGIN 3` 영역은 `while(1)` 내부에 위치한다. 위 코드에는 기존 `while(1)`을 닫는 `}`까지 포함되어 있으므로 해당 영역 전체를 교체한다.

## 6. 코드 설명

### HAL_GPIO_TogglePin()

```c
HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
```

지정한 GPIO 핀의 현재 출력 상태를 반전한다.

- HIGH → LOW
- LOW → HIGH

### HAL_Delay()

```c
HAL_Delay(5000);
```

5000ms 동안 대기한다.

1초 = 1000ms이므로 5000ms는 5초이다.

## 7. 예상 동작

| 시간 | PC13 | LED |
|---|---|---|
| 0~5초 | LOW | ON |
| 5~10초 | HIGH | OFF |
| 10~15초 | LOW | ON |
| 15~20초 | HIGH | OFF |

전체 점멸 주기는 약 10초이다.

## 8. 실습 결과 확인

1. CubeIDE에서 Build를 실행한다.
2. STM32CubeProgrammer에서 최신 ELF 파일을 선택한다.
3. ST-LINK로 프로그램을 기록한다.
4. 내장 LED가 5초 간격으로 켜지고 꺼지는지 확인한다.

## 9. 핵심 정리

- GPIO는 MCU 외부 핀의 입출력을 제어하는 기능이다.
- `HAL_GPIO_TogglePin()`은 출력 상태를 반전한다.
- `HAL_Delay()`는 지정한 시간 동안 프로그램 실행을 대기시킨다.
- PC13 내장 LED는 LOW에서 켜진다.
