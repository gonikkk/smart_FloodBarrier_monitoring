# Flood Barrier (스마트 차수막 시스템 & IoT 모니터링)

![Flood Barrier Demo](docs/images/flood_barrier_demo.gif)

**STM32**로 수위를 감지하여 차수막을 제어하고, **Raspberry Pi**를 통해 데이터를 저장(DB) 및 시각화(GUI)하는 통합 침수 방지 시스템입니다.

기존의 **자동/수동 제어**와 **시각·청각 경고** 기능에 더해, **실시간 데이터 로깅 및 PC 모니터링** 기능을 추가하여 원격 관리 효율성을 높였습니다.

---

## 📂 프로젝트 구조

~~~text
.
├── raspberry/          # [NEW] 라즈베리파이 호스트 애플리케이션
│   ├── uart_receiver.py   # UART 수신 및 MariaDB 저장
│   └── water_gui.py       # Tkinter 기반 모니터링 GUI
└── stm32/              # STM32 펌웨어 소스코드
    ├── Core/              # 메인 로직 (수위 감지, 모터 제어 등)
    ├── Drivers/           # HAL 드라이버
    └── ...
~~~

---

## 💡 프로젝트 개요

- **목적**
  - 집중호우 시 자동으로 수위를 감지하여 차수막을 작동(STM32).
  - 수위 데이터와 장치 상태를 기록하고 관리자가 직관적으로 확인(Raspberry Pi).

- **핵심 기능**
  1. **임베디드 제어 (STM32)**: 수위 센서 감지, 서보모터 제어, LED/부저 경고.
  2. **데이터 통신 (UART)**: STM32 → Raspberry Pi로 실시간 상태 전송 (`RAIN`, `LEVEL`, `SERVO`).
  3. **데이터 관리 (MariaDB)**: 수신된 데이터를 DB에 영구 저장.
  4. **모니터링 (GUI)**: 저장된 로그를 조회하고 현재 상태를 대시보드 형태로 제공.

---

## 🛠 하드웨어 구성

| 구분 | 주요 부품 | 역할 |
| --- | --- | --- |
| **제어부 (STM32)** | STM32 Board, 수위 센서 | 물 높이 측정 및 차수막 구동 |
| | 서보모터, LCD, LED, 부저 | 차수막 물리 제어 및 현장 알림 |
| | IR 리모컨 | 현장 수동 제어 |
| **모니터링부 (RPi)** | **Raspberry Pi 4/5** | 데이터 수집 서버 및 GUI 디스플레이 |
| **연결** | UART (TX/RX) | STM32와 라즈베리파이 간 통신 |

---

## 🚀 설치 및 실행 방법

### 1. STM32 (펌웨어)
1. `stm32/` 폴더 내 프로젝트를 **STM32CubeIDE** 또는 **EWARM**에서 엽니다.
2. 빌드 후 보드에 다운로드합니다.
3. UART 핀(TX/RX)을 라즈베리파이와 연결합니다. (GND 연결 필수)

### 2. Raspberry Pi (서버 및 GUI)

**필수 환경 설정**
- Python 3
- MariaDB (MySQL)

**라이브러리 설치**
~~~bash
pip install pyserial mysql-connector-python tk
~~~

**DB 설정 (MariaDB)**
~~~sql
-- 아래 계정 정보는 소스코드 기준입니다 (수정 가능)
CREATE DATABASE sensordb;
CREATE USER 'sensoruser'@'localhost' IDENTIFIED BY 'sensorpass';
GRANT ALL PRIVILEGES ON sensordb.* TO 'sensoruser'@'localhost';
FLUSH PRIVILEGES;
~~~

**실행 순서**
1. **데이터 수신부 실행** (백그라운드 또는 별도 터미널)
   ~~~bash
   python3 raspberry/uart_receiver.py
   ~~~
   > STM32로부터 데이터를 받아 DB에 적재합니다.

2. **모니터링 GUI 실행**
   ~~~bash
   python3 raspberry/water_gui.py
   ~~~
   > DB 데이터를 5초 주기로 불러와 화면에 표시합니다.

---

## 💻 주요 기능 상세

### 1. 수위 감지 & 자동 제어 (STM32)
- 아날로그 센서값 기반 `NORMAL` / `WARNING` / `FLOOD` 3단계 분류.
- `FLOOD` 단계 도달 시 차수막 자동 상승 및 경보 울림.

### 2. 실시간 모니터링 (GUI)
- **Dashboard**: 현재 수위(mm), 위험 단계, 서보모터 상태(ON/OFF) 실시간 표시.
- **Log Viewer**: 과거 시간대별 수위 변화 및 작동 이력을 테이블 형태로 조회.

### 3. 데이터베이스 연동
- 수신된 데이터(`RAIN=xx,LEVEL=xx,SERVO=xx`)를 파싱하여 `water_log` 테이블에 타임스탬프와 함께 저장.

---

## 📷 시연 장면

### 임베디드 동작 (STM32)
| 물 유입 감지 | 수위 감소 / 복귀 | 리모컨 수동 제어 | 수위 상승 / 경고 |
| --- | --- | --- | --- |
| ![](images/Animation.gif) | ![](images/Animation2.gif) | ![](images/Animation3.gif) | ![](images/Animation4.gif) |

### 모니터링 시스템 (Raspberry Pi)
> *(여기에 GUI 실행 스크린샷을 추가하면 좋습니다)*
- **실시간 대시보드**: 현재 수위 및 시스템 상태 확인
- **DB 로그**: 데이터 이력 조회

---

## 🔮 향후 개선 아이디어
- 웹 서버(Flask/Django)를 구축하여 모바일 웹에서 원격 제어 기능 추가.
- 수위 데이터를 그래프로 시각화(Matplotlib)하여 추세 분석 기능 추가.
