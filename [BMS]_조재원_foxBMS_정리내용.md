
# foxBMS 2 하드웨어 아키텍처 보고서  
**버전:** foxBMS 2 - 1.9.0

---

## 0. BMS? (Battery Management System)

**BMS(Battery Management System)**는 배터리의 안전하고 효율적인 사용을 위해 **모니터링, 보호, 제어, 진단 및 통신** 기능을 수행하는 전자 시스템.  
전기차, 에너지 저장 시스템(ESS), 드론, 항공기, 전동 공구 등 **모든 이차전지 기반 시스템**에서 핵심 제어장치 역할.

### 주요 기능

- **셀 전압 및 온도 감시**: 각 셀의 전압 및 온도를 실시간으로 측정해 과충전/과방전, 과열을 방지  
- **SOC/SOH/SOE 추정**: 배터리의 충전 상태(State of Charge), 건강 상태(State of Health), 사용 가능한 에너지(State of Energy) 계산  
- **충방전 제어**: 릴레이/컨택터를 통해 충전기 및 부하 제어  
- **셀 밸런싱**: 셀 간 전압 불균형을 맞추기 위한 액티브/패시브 밸런싱 수행  
- **이상 감지 및 보호**: 과전압, 저전압, 고온, 절연 결함(IMD) 등의 이상 조건을 탐지하고 시스템을 보호  
- **통신 기능**: VCU, 충전기, 클라우드, 로거와 통신하여 상태 전송 및 명령 수신 (CAN, UART 등)  
- **데이터 로깅 및 진단**: 오류 이벤트 기록, 사용 이력 저장 및 분석 지원  

### 구성 요소

- **센서**: 전류, 전압, 온도 측정용  
- **MCU (Microcontroller)**: 연산 및 제어 핵심  
- **Slave Board**: 셀 단위 측정  
- **Relay/Contactor**: 전류 흐름 제어  
- **통신 인터페이스**: CAN, isoSPI, UART 등  

### BMS가 왜 중요한가?

- 🔒 **배터리 화재 및 폭발 예방**  
- ⚡ **충전 시간 최적화 및 수명 연장**  
- 📊 **상태 기반 유지보수 (Predictive Maintenance)**  
- 🚗 **전기차, ESS, 항공 등 고신뢰성 시스템의 핵심 안전 컴포넌트**  

---

## 1. Battery Module  
foxBMS 기준 각 모듈은 직렬 셀 그룹으로 구성.  

- 온도 센서(NTC 등)는 각 셀 근처에 배치되어 Slave 보드에 연결.  
- 셀 전압은 접촉단자 또는 와이어링 하니스를 통해 Slave 보드에 공급.  

---

## 2. BMS-Slave Unit  
**모델 예시:** SLAVE BOARD v1.0.x  

- 전압/온도 측정: 최대 18셀까지 측정 가능 (LTC6811 사용)  
- 패시브 밸런싱 회로 내장 (최대 50mA 수준, 저항 기반)  
- SPI/isoSPI 통신: Master와 절연된 통신 구성  
- EMC 최적화 회로 포함  
- 여러 개의 Slave를 데이지 체인 연결 가능  

---

## 3. 통신 보드 (Interface Board)  
**공식 명칭:** INTERFACE BOARD v1.0.x  
**주요 칩:** LTC6820 (isoSPI ↔ SPI 변환)  

- Slave → Master 데이터 릴레이  
- 전기적 절연 기능 내장 (GND 격리)  
- Master와 신뢰성 높은 통신 유지  
- 선택적으로 다른 프로토콜(UART, LIN 등)도 지원 가능  

---

## 4. BMS-Master Unit  
**공식 보드:** MASTER TMS570 BOARD v1.1.1  
**주요 SoC:** TI TMS570LC4357 (ARM Cortex-R5)  

- 배터리 상태 집계 (SOC, SOH, SOE 계산)  
- 릴레이 제어 및 보호 로직 실행 (예: Precharge, OVP, UVP)  
- 로깅 및 오류 코드 저장  
- CAN 인터페이스 통해 외부 시스템과 통신  
- 내부에 Watchdog, CRC, EEPROM 내장됨  

---

## 5. Extension Board (확장 보드)  
**명칭:** EXTENSION BOARD v1.0.x  
**목적:** IO 확장 및 사용자 정의 기능 구현  

- 전류 센서 (Shunt, Hall)  
- 디지털 입력/출력  
- 펌프, 팬 등 냉각 제어  
- 추가 통신 포트 (UART, CAN 등)  

---

## 6. 외부 시스템 연동 (CAN/UART 등)  
foxBMS는 기본적으로 CAN 통신 기반 구조.  

- VCU (Vehicle Control Unit): 충전/방전 요청, Fault 연동  
- 충전기: 상태 요청 및 충전 제어 명령 전송  
- HMI / 디스플레이: 사용자 정보 제공  
- 데이터 로거 / 클라우드: 진단 및 유지보수용 로그 업로드  
