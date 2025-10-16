MAVLink 통신을 위한 Python 라이브러리 (Pymavlink)

1. Pymavlink란?

Pymavlink는 Python으로 작성된 MAVLink 프로토콜 처리 라이브러리입니다.

MAVLink는 드론, 자율주행 차량 등 무인 시스템 간 통신을 위한 경량 프로토콜입니다.

Pymavlink는 저수준 MAVLink 메시지 처리를 지원하며, MAVLink 1과 MAVLink 2를 모두 지원합니다.

주요 사용 사례:

GCS (Ground Control Station): MAVProxy

개발자 API: DroneKit

컴퓨터 기반 MAVLink 애플리케이션

Python 3.5 이상에서 동작.

2. Pymavlink 설치

2.1 표준 MAVLink 방언 설치

pip를 사용해 간단히 설치:

bash

복사

pip install pymavlink

참고: PyPi의 Pymavlink는 ArduPilot/mavlink 포크를 기반으로 하며, 기본 MAVLink 저장소와 약간 다를 수 있습니다.

2.2 사용자 정의 MAVLink 방언 생성

사용자 정의 방언이 필요한 경우, mavgen을 사용해 직접 생성해야 합니다.

절차:

mavgen 설치:

bash

복사

python3 -m pip install -r pymavlink/requirements.txt

사용자 정의 방언 XML 파일을 준비.

mavgen으로 Python 라이브러리 생성:

bash

복사

mavgen.py --lang=Python --output=pymavlink/dialects/v20 my_dialect.xml

생성된 .py 파일을 적절한 디렉토리에 복사:

MAVLink 2: pymavlink/dialects/v20

MAVLink 1: pymavlink/dialects/v10

Pymavlink 설치:

bash

복사

python setup.py install --user

3. Pymavlink 주요 모듈

모듈 경로:  /home/dronedev/.local/lib/python3.10/site-packages/pymavlink

Pymavlink는 다양한 모듈로 구성되어 있으며, 주요 모듈은 다음과 같습니다:

mavutil:

통신 채널 설정, 메시지 송수신, 기본 자동 조종 속성 쿼리.

초보자에게 가장 추천되는 시작점.

dialects/v10, dialects/v20:

MAVLink 1 및 MAVLink 2 dialects별 모듈.

메시지 인코딩/디코딩, 서명 처리 기능 제공.

mavwp:

웨이포인트, 지오펜스, 랠리 포인트 관리.

mavparm:

MAVLink 매개변수 관리.

mavextra:

단위 변환 등 유틸리티 함수.

4. Pymavlink 사용법

4.1 연결 설정

mavutil.mavlink_connection을 사용해 통신 채널을 설정합니다.

지원 채널: 직렬 포트, UDP, TCP, 파일.

예제: UDP로 SITL 시뮬레이터 연결

python

복사

from pymavlink import mavutil

# UDP 포트(14540)에서 연결 시작

connection = mavutil.mavlink_connection('udpin:localhost:14560')

# 첫 번째 HEARTBEAT 메시지 대기

connection.wait_heartbeat()

print(f"Heartbeat from system {connection.target_system}, component {connection.target_component}")

mavudp 클래스는 mavutil.py에 정의된 UDP 소켓을 관리하는 클래스입니다. mavfile을 상속하며, MAVLink 메시지를 송수신합니다.

connection.target_system과 connection.target_component는 mavudp에 직접 정의되지 않고, 상위 클래스 mavfile에서 관리됩니다. mavfile은 HEARTBEAT 메시지를 받아 target_system과 target_component를 설정합니다

연결 문자열 형식

형식: [protocol:]address[:port]

예시:

USB: /dev/ttyUSB0

UDP: udpin:localhost:14540

TCP: tcp:127.0.0.1:5760

 

< ./arducopter 의 옵션 >

사용예:  ./arducopter -S --model copter --speedup 1 --defaults parameters/copter.parm -I0 --home 36.5960,126.2922,0,0

arducopter 명령어는 ArduPilot의 SITL(Software In The Loop) 시뮬레이터를 실행하는 데 사용되며, 다양한 옵션을 제공합니다. 옵션의 사용 빈도는 프로젝트나 사용 사례에 따라 다르지만, 일반적으로 개발자 및 테스트 환경에서 자주 사용되는 옵션을 빈도가 높은 순으로 설명하겠습니다. 이는 ArduPilot 커뮤니티의 문서, 포럼, 그리고 일반적인 사용 패턴을 기반으로 추정한 것입니다.

1. --model | -M MODEL

설명: 시뮬레이션에 사용할 모델(예: 쿼드콥터, 헥사콥터 등)을 지정합니다. 예를 들어, quad는 쿼드콥터, hexa는 헥사콥터를 의미합니다.

사용 예: ./arducopter -M quad

빈도 이유: SITL 시뮬레이션의 핵심은 특정 비행체 모델을 테스트하는 것이므로, 모델 선택은 거의 모든 실행에서 필수적입니다.

참고: 지원 모델은 ArduPilot 문서에서 확인 가능하며, quad, hexa, octa, plane 등이 일반적입니다.

2. --home|-O HOME

설명: 시뮬레이션 시작 위치를 지정합니다. 위도(latitude), 경도(longitude), 고도(altitude), 요(yaw)를 입력하거나, 특정 위치 이름을 사용할 수 있습니다.

사용 예: ./arducopter -O -35.363261,149.165230,584,353

빈도 이유: 드론의 시작 위치는 비행 경로 및 임무 계획에 직접적인 영향을 미치므로, 실제 환경을 시뮬레이션하거나 특정 지역을 테스트할 때 자주 설정됩니다.

참고: 위치 이름(예: "Canberra")을 사용할 경우, 내부적으로 좌표로 변환됩니다.

3. --speedup | -s SPEEDUP

설명: 시뮬레이션 속도를 조정합니다. 기본값은 1(실시간)이며, 값이 클수록 시뮬레이션이 빨라집니다.

사용 예: ./arducopter -s 2 (2배속)

빈도 이유: 긴 비행 시뮬레이션을 빠르게 테스트하거나, 개발 중 시간을 절약하기 위해 자주 사용됩니다.

참고: 너무 높은 값은 시스템 성능에 따라 불안정할 수 있습니다.

4. --instance | -I N

설명: SITL 인스턴스 번호를 지정합니다. 각 인스턴스는 포트 번호에 10*instance를 추가하여 여러 SITL 인스턴스를 동시에 실행할 수 있습니다.

사용 예: ./arducopter -I 0 또는 ./arducopter -I 1

빈도 이유: 다중 드론 시뮬레이션(예: Swarm 테스트)이나 병렬 테스트를 위해 필수적이며, 포트 충돌을 방지하는 데 사용됩니다.

참고: 기본 포트는 5670이며, -I 1은 5680번대 포트를 사용합니다.

5. --console | -C

설명: TCP 포트 대신 콘솔을 통해 시뮬레이터와 상호작용합니다. 주로 디버깅이나 간단한 테스트에 사용됩니다.

사용 예: ./arducopter -C

빈도 이유: 초보자나 간단한 테스트 환경에서 GUI 없이 빠르게 결과를 확인하려는 경우 자주 사용됩니다.

참고: 콘솔 모드는 Mission Planner와 같은 GCS(Ground Control Station) 연결을 대체합니다.

6. --wipe | -w

설명: EEPROM(비휘발성 메모리)을 초기화하여 모든 파라미터를 기본값으로 재설정합니다.

사용 예: ./arducopter -w

빈도 이유: 새로운 설정으로 테스트를 시작하거나, 이전 설정으로 인한 문제를 제거하려는 경우 자주 사용됩니다.

참고: 파라미터가 초기화되므로 주의가 필요합니다.

7. --defaults path

설명: 사용자 지정 파라미터 파일(기본값)을 로드합니다. 이를 통해 특정 설정으로 시뮬레이션을 시작할 수 있습니다.

사용 예: ./arducopter --defaults ./my_params.parm

빈도 이유: 반복적인 테스트에서 동일한 파라미터 세트를 사용하거나, 특정 하드웨어 설정을 시뮬레이션할 때 유용합니다.

참고: .parm 파일은 ArduPilot 파라미터 형식을 따라야 합니다.

8. --fg | -F ADDRESS 및 --enable-fgview

설명: FlightGear 시뮬레이터와 연동하여 3D 비행 시각화를 활성화합니다. --fg는 FlightGear의 IP 주소를 지정합니다(기본값: 127.0.0.1).

사용 예: ./arducopter -F 127.0.0.1 --enable-fgview

빈도 이유: 비행 동작을 시각적으로 확인하거나, 비행체의 물리적 움직임을 테스트할 때 사용됩니다. 특히 개발 초기 단계에서 유용합니다.

참고: FlightGear가 설치되어 있어야 하며, 네트워크 설정이 필요할 수 있습니다.

9. --serial0 device ~ --serial9 device

설명: 시뮬레이터의 시리얼 포트(예: SERIAL0~SERIAL9)에 사용할 장치를 지정합니다. 주로 외부 장치(예: GPS, 텔레메트리) 시뮬레이션에 사용됩니다.

사용 예: ./arducopter --serial0 tcp:5760

빈도 이유: GCS 연결(Mission Planner, QGroundControl)이나 외부 모듈 시뮬레이션에서 필수적입니다.

참고: tcp:port, udp:port, 또는 실제 장치 경로(예: /dev/ttyUSB0)를 지정할 수 있습니다.

10. --rate | -r RATE

설명: SITL의 프레임 속도(Hz)를 설정합니다. 기본값은 보통 200Hz이며, 높은 값은 더 정밀한 시뮬레이션을 제공하지만 CPU 부하가 증가합니다.

사용 예: ./arducopter -r 400

빈도 이유: 특정 하드웨어 성능이나 실시간 제어 테스트를 위해 프레임 속도를 조정하는 경우가 많습니다.

참고: 시스템 성능에 따라 적절한 값을 선택해야 합니다.

< mavproxy.py 의 옵션 >

참조 : https://ardupilot.org/mavproxy/

사용예: mavproxy.py --master tcp:127.0.0.1:5760 --out udp:127.0.0.1:14550 --out udp:127.0.0.1:14560

mavproxy.py는 ArduPilot와 같은 자동 조종 소프트웨어와 통신하기 위한 MAVLink 프록시 및 지상 관제소 도구입니다. 제공된 --help 출력을 바탕으로, 사용 빈도가 높은 옵션부터 순서대로 설명하겠습니다. 빈도 추정은 ArduPilot 커뮤니티의 일반적인 사용 사례, 문서, 포럼을 기반으로 하며, SITL(Software In The Loop) 환경에서의 사용을 고려했습니다.

사용 빈도가 높은 옵션 (상위 10개)

1. --master=DEVICE[,BAUD]

설명: MAVLink 주 연결 포트를 지정하며, 선택적으로 보율(baud rate)을 설정합니다. 장치는 시리얼 포트, TCP, UDP 등이 될 수 있습니다.

사용 예: --master=/dev/ttyUSB0,115200 또는 --master=127.0.0.1:5760

빈도 이유: MAVProxy의 핵심은 비행체(또는 SITL)와의 MAVLink 통신이므로, 모든 실행에서 필수적입니다.

참고: SITL에서는 127.0.0.1:5760이 기본적으로 사용됩니다.

2. --out=DEVICE[,BAUD]

설명: 추가 MAVLink 출력 포트를 지정하여 데이터를 다른 GCS(예: Mission Planner, QGroundControl)나 장치로 전달합니다.

사용 예: --out=127.0.0.1:14550

빈도 이유: 외부 GCS와 연결하거나 다중 클라이언트로 데이터를 스트리밍할 때 자주 사용됩니다.

참고: 여러 --out 옵션을 추가하여 다중 출력 포트를 설정할 수 있습니다.

3. --console

설명: GUI 콘솔을 활성화하여 비행체 상태, MAVLink 메시지, 명령을 실시간으로 확인합니다.

사용 예: mavproxy.py --master=127.0.0.1:5760 --console

빈도 이유: 디버깅, 상태 모니터링, 간단한 테스트에서 직관적으로 사용되며, 가벼운 인터페이스를 제공합니다.

참고: 콘솔은 CPU 사용량이 적고, SITL 환경에서 자주 사용됩니다.

4. --map

설명: 지도 모듈을 활성화하여 비행체의 위치, 경로, 임무를 실시간으로 시각화합니다.

사용 예: mavproxy.py --master=127.0.0.1:5760 --map

빈도 이유: 비행 경로 확인, 임무 계획 테스트, 위치 기반 디버깅에 필수적입니다.

참고: wxPython 및 인터넷 연결이 필요하며, SITL에서 비행체 이동을 시각화할 때 유용합니다.

5. --baudrate=BAUDRATE

설명: 기본 시리얼 포트 보율을 설정합니다. --master 또는 --out에서 보율을 지정하지 않을 경우 적용됩니다.

사용 예: --baudrate=115200

빈도 이유: 실제 하드웨어(예: Pixhawk)와 통신하거나 SITL 환경에서 안정적인 연결을 위해 자주 설정됩니다.

참고: 일반적인 보율은 57600, 115200, 921600입니다.

6. --sitl=SITL

설명: SITL 시뮬레이터와 연결할 출력 포트를 지정합니다.

사용 예: --sitl=127.0.0.1:5501

빈도 이유: SITL 환경에서 MAVProxy와 시뮬레이터를 연결하는 데 자주 사용됩니다.

참고: ArduPilot SITL은 기본적으로 127.0.0.1:5501을 사용합니다.

7. --streamrate=STREAMRATE

설명: MAVLink 메시지 스트림의 전송 속도(Hz)를 설정합니다.

사용 예: --streamrate=10

빈도 이유: 네트워크 부하를 관리하거나 특정 데이터 업데이트 빈도를 맞추기 위해 사용됩니다.

참고: 높은 값은 대역폭을 많이 소모할 수 있으므로 주의가 필요합니다.

8. --logfile=LOGFILE

설명: MAVLink 데이터를 기록할 로그 파일 경로를 지정합니다.

사용 예: --logfile=/home/dronedev/log.tlog

빈도 이유: 비행 데이터를 저장하여 나중에 분석하거나 디버깅할 때 유용합니다.

참고: .tlog 파일은 Mission Planner 또는 QGroundControl에서 재생 가능합니다.

9. --load-module=LOAD_MODULE

설명: 추가 모듈(예: adsb, fence, rally)을 로드합니다. 여러 모듈을 쉼표로 구분하거나 반복 사용 가능합니다.

사용 예: --load-module=fence,adsb

빈도 이유: 지오펜싱, ADS-B 통합, 랠리 포인트 등 특정 기능을 테스트할 때 사용됩니다.

참고: 사용 가능한 모듈은 MAVProxy/modules 디렉토리에서 확인할 수 있습니다.

10. --aircraft=AIRCRAFT

설명: 로그와 상태 데이터를 저장할 항공기별 디렉토리 이름을 지정합니다.

사용 예: --aircraft=MyDrone

빈도 이유: 여러 비행체를 관리하거나 로그를 체계적으로 정리할 때 유용합니다.

참고: 지정된 이름으로 디렉토리가 생성되며, 로그와 설정이 저장됩니다.

 

<Mavproxy 주요 명령어 사용법>

명령어는 콘솔에서 직접 입력하며, 일부 명령어는 하위 명령어나 인자를 필요로 합니다. 아래는 제공된 목록 중 주요 명령어의 사용법입니다.

1. status

설명: 비행체의 현재 상태(모드, 배터리, GPS, 속도 등)를 표시합니다.

사용법: status

예시:

text

복사

status

출력: Mode: GUIDED, Battery: 11.1V, GPS: 3D Lock, Sats: 10

용도: 비행체의 실시간 상태를 빠르게 확인.

2. param

설명: 비행체의 파라미터를 조회, 설정, 저장합니다.

하위 명령:

param show <파라미터 이름>: 특정 파라미터 값 확인.

param set <파라미터 이름> <값>: 파라미터 값 설정.

param save <파일 경로>: 파라미터를 파일로 저장.

param load <파일 경로>: 파라미터 파일 로드.

사용법:

text

복사

param show ARMING_CHECK 

param set ARMING_CHECK 1 

param save /home/dronedev/params.txt

예시:

text

복사

param set THR_MIN 130

최소 스로틀 값을 130으로 설정.

용도: 비행체 설정(예: 모터 출력, PID 값) 조정.

3. mode

설명: 비행 모드를 설정하거나 현재 모드를 확인합니다(예: STABILIZE, GUIDED, AUTO).

사용법:

mode: 현재 모드 확인.

mode <모드 이름>: 모드 변경.

예시:

text

복사

mode GUIDED mode

GUIDED 모드로 전환 후 현재 모드 확인.

용도: 테스트나 임무에 맞는 비행 모드 전환.

4. arm

설명: 비행체를 활성화(arming)하여 모터를 준비 상태로 만듭니다.

사용법:

arm throttle: 모터 활성화.

예시:

text

복사

arm throttle

용도: 비행 시작 전 모터 활성화.

5. disarm

설명: 비행체를 비활성화(disarming)하여 모터를 정지합니다.

사용법: disarm

예시:

text

복사

disarm

용도: 비행 종료 또는 안전을 위해 모터 정지.

6. takeoff

설명: 비행체를 지정된 고도로 이륙시킵니다(GUIDED 또는 AUTO 모드에서 동작).

사용법: takeoff <고도(m)>

예시:

text

복사

mode GUIDED 

arm throttle takeoff 50

GUIDED 모드로 전환, 모터 활성화, 50m 고도로 이륙.

용도: 자동 이륙 테스트.

7. wp

설명: 임무(waypoints)를 관리합니다(로드, 저장, 목록 표시 등).

하위 명령:

wp list: 현재 임무 목록 표시.

wp load <파일 경로>: 임무 파일 로드.

wp save <파일 경로>: 임무를 파일로 저장.

wp loop: 임무를 반복 실행.

사용법:

text

복사

wp load /home/dronedev/mission.txt wp list

예시:

text

복사

wp load /home/dronedev/mission.txt mode AUTO

임무 파일을 로드하고 AUTO 모드로 실행.

용도: 사전 정의된 경로로 비행 테스트.

8. guided

설명: GUIDED 모드에서 특정 위치(위도, 경도, 고도)로 비행체를 이동시킵니다.

사용법: guided <위도> <경도> <고도(m)>

예시:

text

복사

mode GUIDED guided -35.363261 149.165230 50

GUIDED 모드로 전환 후 지정된 좌표와 50m 고도로 이동.

용도: 특정 위치로의 이동 테스트.

9. land

설명: 비행체를 현재 위치에서 착륙시킵니다.

사용법: land

예시:

text

복사

land

용도: 비행 종료 또는 안전 착륙.

10. rc

설명: RC 채널 값을 수동으로 설정하여 비행체를 제어합니다(예: 스로틀, 롤, 피치).

사용법: rc <채널 번호> <PWM 값>

예시:

text

복사

rc 3 1500

3번 채널(스로틀)을 PWM 1500으로 설정(중립).

용도: 수동 제어 또는 특정 채널 테스트.

기타 유용한 명령어

reboot: 비행체의 자동 조종 장치를 재부팅.

사용법: reboot

position: 현재 위치(위도, 경도, 고도)를 표시.

사용법: position

set: MAVProxy 내부 설정을 변경(예: set streamrate 10으로 스트림 속도 설정).

사용법: set <설정 이름> <값>

module: 추가 모듈을 로드하거나 관리.

사용법: module load fence (지오펜싱 모듈 로드)

watch: 특정 MAVLink 메시지를 실시간으로 모니터링.

사용법: watch GPS_RAW_INT

명령어 사용 예시 (SITL 환경)

SITL 실행:

bash

복사

./arducopter -M quad -I 0

MAVProxy 실행:

bash

복사

mavproxy.py --master=127.0.0.1:5760 --console --map

콘솔에서 명령어 입력:

text

복사

status # 상태 확인 

param set ARMING_CHECK 1 # ARMING_CHECK 파라미터 설정 

mode GUIDED # GUIDED 모드로 전환 

arm throttle # 모터 활성화 

takeoff 50 # 50m 이륙 

guided -35.363261 149.165230 50 # 특정 위치로 이동 

land # 착륙 disarm # 모터 비활성화

# ardu-sim 폴더에서 기체 시뮬 모듈 실행 (5760 Port )

 ./arducopter -S --model copter --speedup 1 --defaults parameters/copter.parm -I0 --home 36.5960,126.2922,0,0

 #mavproxy 실행 ( --master 5760 / QGC , consol -> 14550 / python code ->14560 

mavproxy.py --master tcp:127.0.0.1:5760 --out udp:127.0.0.1:14550 --out udp:127.0.0.1:14560 

#QgroundControl 실행 (QGC 설치된경로)

 ./QGroundControl.AppImage 

< screen 프로그램 활용 >

screen은 리눅스 기반 시스템에서 터미널 세션을 관리하는 유용한 도구입니다. 

여러 터미널 창을 사용하지 않고도 한 터미널에서 여러 작업을 동시에 수행하거나, 원격 서버에 연결할 때 세션을 유지하고자 할 때 사용할 수 있습니다.

 screen을 사용하면 세션을 생성, 관리 및 종료할 수 있으며, 각 세션을 독립적으로 관리할 수 있습니다

매번 함께 실행되는 consol 창을 hidden으로 함께처리하여 Back으로 sessio처리함.

sudo apt-get install screen

cd ardu-sim/

gedit ardu-sim.sh

ardu-sim.sh 파링 편집

#!/bin/bash 

screen -S vehicle -d -m bash -c "./arducopter -S --model copter --speedup 1 --defaults parameters/copter.parm -I0 --home 36.5960,126.2922,0,0 " 

screen -S proxy -d -m bash -c "mavproxy.py --master tcp:127.0.0.1:5760 --out udp:127.0.0.1:14550 --out udp:127.0.0.1:14560" 

screen -S QGC -d -m bash -c "./QGroundControl.AppImage"

저장  (Ctrl+S)

#실행 권한설정

sudo chmod +x ardu-sim.sh

#해당폴더에서 

./ardu-sim.sh

Screen 사용예시

screen -list #현재 실행중인 screen session 보여줌 

screen -r proxy # proxy consol 창 오픈 

screen -r vehicle #vehicle 콘솔 보임 

screen -S vehicle -X quit # vehicle세션 종료

screen -S proxy -X quit # proxy 세션 종료 

killall screen # 모든세션 종료

# VisualStudio  listern.py 파일명 생성

import pymavlink.mavutil

import time

# 기체에 접속

vehicle = pymavlink.mavutil.mavlink_connection(device="udpin:127.0.0.1:14560")

# 접속 신호 시간

vehicle.wait_heartbeat(timeout=5)

# 접속 기체의 SYSID SYSID>0 면 성공.

print("Connected to the vehicle")

print("Target system:", vehicle.target_system, "Target component:", vehicle.target_component)
