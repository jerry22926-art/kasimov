# Sensor Code Guide

현재 센서 코드는 라인트레이서 센서 4개와 측면 적외선 센서 2개를 기준으로 작성했다.


---

## 1. 파일 구성

| 파일명 | 역할 |
|---|---|
| `Linetracer_sensing.ino` | 라인트레이서 4개 값을 읽고, 탈출 액션까지 판단 |
| `Linetracer_sensing_simple.ino` | 라인트레이서 4개 raw 값만 읽음 |
| `IR_sensing.ino` | 좌/우 적외선 센서 값을 읽고, 적 위치 상태까지 판단 |
| `IR_sensing_simple.ino` | 좌/우 적외선 센서 digital 값만 읽음 |

---

## 2. 센서 연결

### 라인트레이서 HW-870

| 위치 | Arduino 핀 |
|---|---|
| Front | A0 |
| Rear | A1 |
| Left | A2 |
| Right | A3 |

연결 방식:

- HW-870 VCC -> Arduino 5V
- HW-870 GND -> Arduino GND
- HW-870 AO -> Arduino A0~A3
- HW-870 DO -> 사용하지 않음

현재 테스트 결과, 검은색 라인에서 센서값이 더 크게 나왔다.

그래서 판단 포함 버전에서는 `sensorValue > LINE_THRESHOLD`이면 라인을 감지한 것으로 본다.

---

### 측면 적외선 HW-201

| 위치 | Arduino 핀 |
|---|---|
| Left IR | D2 |
| Right IR | D3 |

연결 방식:

- HW-201 VCC -> Arduino 5V
- HW-201 GND -> Arduino GND
- HW-201 OUT -> Arduino D2, D3

HW-201은 보통 감지 시 `LOW(0)`, 미감지 시 `HIGH(1)`이 나온다.  
다만 실제 모듈에 따라 다를 수 있으므로 테스트할 때 시리얼 모니터로 확인해야 한다.

---

## 3. 어떤 파일을 쓰면 되는가

### 센서 코드에서 상태까지 판단하는 경우

사용 파일:

- `Linetracer_sensing.ino`
- `IR_sensing.ino`

메인에서 주로 쓰는 함수:

- `decideLineEscapeAction()`
- `detectIrTarget()`

이 방식은 센서 코드가 센서값을 읽고, 필요한 상태값까지 반환한다.

라인트레이서에서 반환되는 상태:

- `ACTION_NO_LINE`
- `ACTION_ESCAPE_BACKWARD`
- `ACTION_ESCAPE_FORWARD`
- `ACTION_ESCAPE_TURN_RIGHT`
- `ACTION_ESCAPE_TURN_LEFT`
- `ACTION_ESCAPE_BACKWARD_RIGHT`
- `ACTION_ESCAPE_BACKWARD_LEFT`
- `ACTION_ESCAPE_FORWARD_RIGHT`
- `ACTION_ESCAPE_FORWARD_LEFT`
- `ACTION_ESCAPE_EMERGENCY`

적외선에서 반환되는 상태:

- `IR_TARGET_NONE`
- `IR_TARGET_LEFT`
- `IR_TARGET_RIGHT`
- `IR_TARGET_BOTH`

모터 쪽에서는 이 상태값에 맞게 모터 동작을 연결하면 된다.

---

### 모터 쪽에서 직접 상태를 판단하는 경우

사용 파일:

- `Linetracer_sensing_simple.ino`
- `IR_sensing_simple.ino`

메인에서 주로 쓰는 함수:

- `readLineSensors()`
- `readSideIrSensors()`

이 방식은 센서 코드가 판단하지 않고 센서값만 넘긴다.

라인트레이서 값:

- `lineValues.front`
- `lineValues.rear`
- `lineValues.left`
- `lineValues.right`

적외선 값:

- `irValues.left`
- `irValues.right`

모터 쪽에서 이 값을 이용해 threshold, 상태, 모터 동작을 직접 정하면 된다.

---

## 4. 통합 시 주의사항

Arduino 코드에서는 `setup()`과 `loop()` 함수가 각각 하나씩만 있어야 한다.

따라서 4개의 `.ino` 파일을 그대로 한 번에 합치면 안 된다.

최종 통합할 때는 사용할 방식에 따라 필요한 함수와 구조체만 메인 코드로 옮겨서 쓰면 된다.

판단 포함 버전을 사용할 경우:

- `decideLineEscapeAction()`
- `detectIrTarget()`

Simple 버전을 사용할 경우:

- `readLineSensors()`
- `readSideIrSensors()`

---

## 5. 정리

모터 쪽에서 센서 판단 결과만 받아서 쓰고 싶으면 아래 파일을 쓰면 된다.

- `Linetracer_sensing.ino`
- `IR_sensing.ino`

모터 쪽에서 raw 값만 받아서 직접 판단하고 싶으면 아래 파일을 쓰면 된다.

- `Linetracer_sensing_simple.ino`
- `IR_sensing_simple.ino`

팀에서 최종 제어 구조를 어떻게 잡느냐에 따라 둘 중 하나를 선택하면 된다.
