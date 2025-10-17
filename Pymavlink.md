# MAVLink 통신을 위한 Python 라이브러리 (Pymavlink) 가이드

## 1. Pymavlink란?

- **Pymavlink**는 Python으로 작성된 **MAVLink 프로토콜** 처리 라이브러리입니다.
- **MAVLink**는 드론, 자율주행 차량 등 무인 시스템 간 통신을 위한 경량 프로토콜입니다.
- Pymavlink는 **저수준** MAVLink 메시지 처리를 지원하며, **MAVLink 1**과 **MAVLink 2**를 모두 지원합니다.

**주요 사용 사례:**
- **GCS (Ground Control Station)**: MAVProxy
- **개발자 API**: DroneKit
- **컴퓨터 기반 MAVLink 애플리케이션**

**Python 3.5 이상**에서 동작.

---

## 2. Pymavlink 설치

### 2.1 표준 MAVLink 방언 설치

**pip를 사용해 간단히 설치:**
```bash
pip install pymavlink
```

**참고**: PyPi의 Pymavlink는 **ArduPilot/mavlink** 포크를 기반으로 하며, 기본 MAVLink 저장소와 약간 다를 수 있습니다.

 

## 3. Pymavlink 주요 모듈

**모듈 경로:** `/home/dronedev/.local/lib/python3.10/site-packages/pymavlink`

| 모듈 | 설명 |
|------|------|
| **mavutil** | 통신 채널 설정, 메시지 송수신, 기본 자동 조종 속성 쿼리 (초보자 추천) |
| **dialects/v10, v20** | MAVLink 1/2 dialects별 메시지 인코딩/디코딩, 서명 처리 |
| **mavwp** | 웨이포인트, 지오펜스, 랠리 포인트 관리 |
| **mavparm** | MAVLink 매개변수 관리 |
| **mavextra** | 단위 변환 등 유틸리티 함수 |

---

## 4. Pymavlink 사용법

### 4.1 연결 설정

**mavutil.mavlink_connection**을 사용해 통신 채널 설정
**지원 채널:** 직렬 포트, UDP, TCP, 파일

#### 예제: UDP로 SITL 시뮬레이터 연결
```python
from pymavlink import mavutil

# UDP 포트(14560)에서 연결 시작
connection = mavutil.mavlink_connection('udpin:localhost:14560')

# 첫 번째 HEARTBEAT 메시지 대기
connection.wait_heartbeat()
print(f"Heartbeat from system {connection.target_system}, component {connection.target_component}")
```

**연결 문자열 형식:** `[protocol:]address[:port]`

| 프로토콜 | 예시 |
|----------|------|
| USB | `/dev/ttyUSB0` |
| UDP | `udpin:localhost:14540` |
| TCP | `tcp:127.0.0.1:5760` |

---

## 5. ArduCopter SITL 옵션 (사용 빈도 순)

| 순위 | 옵션 | 설명 | 사용 예 |
|------|------|------|---------|
| 1 | `--model \| -M MODEL` | 시뮬레이션 모델 지정 | `./arducopter -M quad` |
| 2 | `--home\|-O HOME` | 시작 위치 지정 | `./arducopter -O 37.5843,126.925,0,0` |
| 3 | `--speedup \| -s SPEEDUP` | 시뮬레이션 속도 | `./arducopter -s 2` |
| 4 | `--instance \| -I N` | SITL 인스턴스 번호 | `./arducopter -I 0` |
| 5 | `--console \| -C` | 콘솔 모드 | `./arducopter -C` |

**전체 실행 예시:**
```bash
./arducopter -S --model copter --speedup 1 --defaults parameters/copter.parm -I0 --home 36.5960,126.2922,0,0
```

---

## 6. MAVProxy 옵션 (사용 빈도 순)

| 순위 | 옵션 | 설명 | 사용 예 |
|------|------|------|---------|
| 1 | `--master=DEVICE[,BAUD]` | 주 연결 포트 | `--master=tcp:127.0.0.1:5760` |
| 2 | `--out=DEVICE[,BAUD]` | 추가 출력 포트 | `--out=udp:127.0.0.1:14550` |
| 3 | `--console` | GUI 콘솔 활성화 | `--console` |
| 4 | `--map` | 지도 모듈 | `--map` |

**전체 실행 예시:**
```bash
mavproxy.py --master tcp:127.0.0.1:5760 --out udp:127.0.0.1:14550 --out udp:127.0.0.1:14560
```

---

## 7. MAVProxy 주요 명령어

| 명령어 | 사용법 | 설명 | 예시 |
|--------|--------|------|------|
| `status` | `status` | 비행체 상태 표시 | `status` |
| `param` | `param set <NAME> <VALUE>` | 파라미터 설정 | `param set ARMING_CHECK 1` |
| `mode` | `mode GUIDED` | 비행 모드 변경 | `mode GUIDED` |
| `arm` | `arm throttle` | 모터 활성화 | `arm throttle` |
| `takeoff` | `takeoff 50` | 50m 이륙 | `takeoff 50` |
| `guided` | `guided <lat> <lon> <alt>` | 특정 위치 이동 | `guided 37.5843 126.925 50` |
| `land` | `land` | 착륙 | `land` |
| `wp` | `wp load mission.txt` | 임무 로드 | `wp load mission.txt` |

**SITL 테스트 시퀀스:**
```
status
mode GUIDED
arm throttle
takeoff 50
guided 36.5960 126.2922 50
land
disarm
```

---

## 8. Screen 프로그램 활용 (일괄 실행)

**설치:**
```bash
sudo apt-get install screen
```

**ardu-sim.sh 스크립트 생성:**
```bash
cd ardu-sim/
gedit ardu-sim.sh
```

**스크립트 내용:**
```bash
#!/bin/bash
screen -S vehicle -d -m bash -c "./arducopter -S --model copter --speedup 1 --defaults parameters/copter.parm -I0 --home  37.5843,126.925,0,0"
screen -S proxy -d -m bash -c "mavproxy.py --master tcp:127.0.0.1:5760 --out udp:127.0.0.1:14550 --out udp:127.0.0.1:14560"
screen -S QGC -d -m bash -c "./QGroundControl-x86_64.AppImage"
```

**실행:**
```bash
chmod +x ardu-sim.sh
./ardu-sim.sh
```

**Screen 관리:**
| 명령어 | 설명 |
|--------|------|
| `screen -list` | 실행 중인 세션 목록 |
| `screen -r proxy` | proxy 콘솔 열기 |
| `screen -S vehicle -X quit` | vehicle 세션 종료 |
| `killall screen` | 모든 세션 종료 |

---

## 9. Python 연결 코드 (listener.py)

```python
import pymavlink.mavutil
import time

# 기체에 접속 (14560 포트)
vehicle = pymavlink.mavutil.mavlink_connection(device="udpin:127.0.0.1:14560")

# 접속 신호 대기 (5초 타임아웃)
vehicle.wait_heartbeat(timeout=5)

# 접속 성공 확인 (SYSID > 0)
print("Connected to the vehicle")
print("Target system:", vehicle.target_system, "Target component:", vehicle.target_component)
```

---
## mav_message.py
#### 예제: UDP로 mAVlink 데이터 보기
```python
import time
import pymavlink.mavutil as utility #/home/dronedev/.local/lib/python3.8/site-packages/pymavlink --> mavutil.py 
import pymavlink.dialects.v20.all as dialect #/home/dronedev/.local/lib/python3.8/site-packages/pymavlink/dialects/v20  -->all.py

# connect to vehicle   mavutil.py  1079 line ->def mavlink_connection()
vehicle = utility.mavlink_connection(device="udpin:127.0.0.1:14560") #mavproxy.py --master tcp:127.0.0.1:5760 --out udp:127.0.0.1:14550 --out udp:127.0.0.1:14560
#vehicle = utility.mavlink_connection('/dev/ttyUSB0', 57600)	#Telemetry # cd /dev/  여기서 맞는 포트 확인가능하세요
#vehicle = utility.mavlink_connection('/dev/ttyACM0', 57600)	#usb # cd /dev/ 여기서 자기컴에 맞는 포트 확인가능하세요

# wait for a heartbeat
vehicle.wait_heartbeat()  # mavutil.py 539 line -> def wait_hearbeat()

# inform user
print("Connected to system:", vehicle.target_system, ", component:", vehicle.target_component)

# infinite loop



while True:

    # try to receive a message
    try:

        # receive a message /type = mavlink common lists https://mavlink.io/en/messages/common.html
        #message = vehicle.recv_match(type=dialect.MAVLink_system_time_message.msgname, blocking=True)
        message = vehicle.recv_match()        
        # convert received message to dictionary
        message = message.to_dict()
        print(message)
       
    except: 
        time.sleep(0.1)
 ```
       
## 🚀 전체 시스템 실행 순서

1. **스크립트 실행:** `./ardu-sim.sh`
   - SITL (5760): ArduCopter 시뮬레이터
   - MAVProxy: 프록시 (14550: QGC, 14560: Python)
   - QGC: 지상 관제소

2. **Python 코드 실행:** `python listener.py`

3. **MAVProxy 콘솔:** `screen -r proxy`

 

---
 
