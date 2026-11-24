
# 실습 2-2. 버튼을 누를 때마다 LED ON/OFF 유지

## 1. 실습 목표

버튼을 한 번 누르면 LED가 켜진 상태로 유지되고, 다시 누르면 LED가 꺼진 상태로 유지되도록 구현한다.

버튼을 누르고 있는 동안 LED 상태가 계속 바뀌지 않도록 입력 상태 변화를 감지한다.

**학습 내용**
- GPIO 입력 상태 비교
- 버튼 눌림 이벤트 감지
- 이전 상태 저장
- `HAL_GetTick()`
- 디바운싱(Debouncing)
- `HAL_GPIO_TogglePin()`

## 2. 준비물

- STM32F411CEU6 BlackPill
- ST-LINK
- USB-C 케이블

## 3. 핀맵

| 핀 | CubeMX 설정 | 용도 |
|---|---|---|
| PA0 | GPIO_Input / Pull-up | 내장 KEY 버튼 |
| PC13 | GPIO_Output | 내장 LED |
| PA13 / PA14 | Serial Wire | ST-LINK |

## 4. STM32CubeMX 설정

실습 2-1과 동일하다.

- PA0: GPIO_Input
- PA0 Pull-up: Enable
- PC13: GPIO_Output
- PC13 Initial Output: High

추가적인 CubeMX 설정은 필요하지 않다.

## 5. CubeIDE 코드

**수정 파일:** `Core/Src/main.c`

이번 실습에서는 두 영역을 수정한다.

### STEP 1. 변수 초기화

**교체 위치:** `/* USER CODE BEGIN 2 */` ~ `/* USER CODE END 2 */`

```c
/* USER CODE BEGIN 2 */

  // 직전 버튼 입력 상태
  GPIO_PinState lastRawButton = GPIO_PIN_SET;

  // 디바운싱 후 확정된 버튼 상태
  GPIO_PinState stableButton = GPIO_PIN_SET;

  // 마지막 버튼 입력 변화 시각
  uint32_t lastChangeTime = HAL_GetTick();

/* USER CODE END 2 */
```

### STEP 2. 버튼 토글 로직

**교체 위치:** `/* USER CODE BEGIN 3 */` ~ `/* USER CODE END 3 */`

```c
/* USER CODE BEGIN 3 */

  // 현재 버튼 상태 읽기
  GPIO_PinState currentButton =
      HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0);

  uint32_t now = HAL_GetTick();

  // 버튼 상태가 바뀌면 시간 기록
  if (currentButton != lastRawButton)
  {
      lastChangeTime = now;
  }

  // 30ms 이상 같은 상태를 유지했는지 확인
  if ((uint32_t)(now - lastChangeTime) >= 30U)
  {
      if (currentButton != stableButton)
      {
          // 안정된 버튼 상태 갱신
          stableButton = currentButton;

          // 눌린 순간에만 LED 토글
          if (stableButton == GPIO_PIN_RESET)
          {
              HAL_GPIO_TogglePin(
                  GPIOC,
                  GPIO_PIN_13
              );
          }
      }
  }

  // 이전 버튼 상태 갱신
  lastRawButton = currentButton;

}
/* USER CODE END 3 */
```

## 6. 코드 설명

### 이전 상태 저장

```c
GPIO_PinState lastRawButton = GPIO_PIN_SET;
```

버튼의 이전 입력값을 저장한다.

### HAL_GetTick()

```c
uint32_t now = HAL_GetTick();
```

시스템 시작 이후 경과한 시간을 밀리초 단위로 가져온다.

### 디바운싱

```c
if ((uint32_t)(now - lastChangeTime) >= 30U)
```

버튼의 입력 상태가 최소 30ms 동안 안정적으로 유지되었는지 확인한다.

버튼은 물리적 접점이기 때문에 한 번 눌러도 짧은 시간 동안 HIGH와 LOW가 반복되는 채터링이 발생할 수 있다.

### LED 토글

```c
HAL_GPIO_TogglePin(
    GPIOC,
    GPIO_PIN_13
);
```

안정적으로 버튼이 눌린 순간에 한 번만 LED 상태를 반전한다.

## 7. 예상 동작

| 동작 | LED |
|---|---|
| 초기 상태 | OFF |
| 첫 번째 누름 | ON |
| 버튼 놓음 | ON 유지 |
| 두 번째 누름 | OFF |
| 버튼 놓음 | OFF 유지 |
| 세 번째 누름 | ON |

## 8. 실습 결과 확인

1. 코드를 Build하고 업로드한다.
2. 버튼을 짧게 한 번 누른다.
3. LED가 켜진 상태로 유지되는지 확인한다.
4. 버튼을 다시 누른다.
5. LED가 꺼진 상태로 유지되는지 확인한다.

## 9. 핵심 정리

- 버튼을 누르고 있는 상태와 누르는 순간은 다르다.
- 이전 상태와 현재 상태를 비교해 이벤트를 감지한다.
- 디바운싱으로 버튼 접점 채터링을 억제한다.
- `HAL_GetTick()`을 이용해 시간 조건을 판단할 수 있다.
