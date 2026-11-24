
# 실습 2-1. 버튼을 누르면 LED 켜기

## 1. 실습 목표

GPIO 입력 기능을 활용하여 버튼을 누르고 있는 동안 내장 LED가 켜지고, 버튼을 놓으면 LED가 꺼지도록 구현한다.

**학습 내용**
- GPIO Input
- Pull-up 저항
- Active-Low 입력
- `HAL_GPIO_ReadPin()`
- `HAL_GPIO_WritePin()`
- `if / else` 조건문

## 2. 준비물

- STM32F411CEU6 BlackPill
- ST-LINK
- USB-C 케이블

BlackPill에 내장된 KEY 버튼과 LED를 사용하므로 외부 부품은 필요하지 않다.

## 3. 핀맵

| 핀 | CubeMX 설정 | 용도 |
|---|---|---|
| PA0 | GPIO_Input / Pull-up | 내장 KEY 버튼 |
| PC13 | GPIO_Output | 내장 LED |
| PA13 / PA14 | Serial Wire | ST-LINK |

### 버튼과 LED의 논리

| 상태 | 입력 또는 출력 |
|---|---|
| 버튼 누르지 않음 | PA0 = HIGH |
| 버튼 누름 | PA0 = LOW |
| LED 켜짐 | PC13 = LOW |
| LED 꺼짐 | PC13 = HIGH |

## 4. STM32CubeMX 설정

### PA0 설정
- GPIO Mode: Input
- GPIO Pull-up/Pull-down: Pull-up

### PC13 설정
- GPIO Mode: Output Push-Pull
- GPIO Output Level: High
- GPIO Pull-up/Pull-down: No Pull

### Clock Configuration
- HSI 16MHz 유지

설정 후 `GENERATE CODE`를 실행한다.

## 5. CubeIDE 코드

**수정 파일:** `Core/Src/main.c`

**교체 위치:** `/* USER CODE BEGIN 3 */` ~ `/* USER CODE END 3 */`

```c
/* USER CODE BEGIN 3 */

  // PA0 버튼 상태 읽기
  GPIO_PinState button =
      HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0);

  if (button == GPIO_PIN_RESET)
  {
      // 버튼이 눌렸을 때 LED ON
      HAL_GPIO_WritePin(
          GPIOC,
          GPIO_PIN_13,
          GPIO_PIN_RESET
      );
  }
  else
  {
      // 버튼을 놓았을 때 LED OFF
      HAL_GPIO_WritePin(
          GPIOC,
          GPIO_PIN_13,
          GPIO_PIN_SET
      );
  }

}
/* USER CODE END 3 */
```

## 6. 코드 설명

### HAL_GPIO_ReadPin()

```c
GPIO_PinState button =
    HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0);
```

PA0의 현재 입력 상태를 읽는다.

반환값은 다음 두 가지이다.

- `GPIO_PIN_SET`: HIGH
- `GPIO_PIN_RESET`: LOW

### HAL_GPIO_WritePin()

```c
HAL_GPIO_WritePin(
    GPIOC,
    GPIO_PIN_13,
    GPIO_PIN_RESET
);
```

PC13 핀을 LOW로 만들어 내장 LED를 켠다.

### Pull-up

PA0에 Pull-up을 설정하면 버튼을 누르지 않았을 때 기본 입력 상태가 HIGH로 유지된다.

버튼을 누르면 PA0이 GND와 연결되어 LOW가 된다.

## 7. 예상 동작

| 버튼 상태 | PA0 | PC13 | LED |
|---|---|---|---|
| 안 누름 | HIGH | HIGH | OFF |
| 누름 | LOW | LOW | ON |
| 놓음 | HIGH | HIGH | OFF |

버튼을 계속 누르고 있는 동안 LED가 켜진 상태를 유지한다.

## 8. 실습 결과 확인

1. Build 후 최신 ELF를 업로드한다.
2. 버튼을 누르지 않았을 때 LED가 꺼져 있는지 확인한다.
3. 버튼을 누르면 LED가 켜지는지 확인한다.
4. 버튼을 놓으면 LED가 꺼지는지 확인한다.

## 9. 핵심 정리

- GPIO Input은 외부 신호의 HIGH/LOW 상태를 읽는다.
- Pull-up은 입력 신호가 떠 있는 상태를 방지한다.
- PA0 버튼은 누르면 LOW가 된다.
- `if / else` 조건문으로 입력에 따른 출력을 제어한다.
