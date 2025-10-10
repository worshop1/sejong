#공식문서 
https://ardupilot.org/mavproxy/
MAVProxy는 강력한 명령줄 기반 "개발자" 지상국 소프트웨어


## MAVProxy의 주요 기능

MAVProxy는 명령줄 기반의 지상관제시스템(Ground Control Station, GCS)으로, MAVLink 프로토콜을 사용하는 무인기 시스템의 운용 및 개발을 지원한다. 다음은 MAVProxy의 주요 특징이다.

1. 명령줄 기반 구조
MAVProxy는 콘솔 환경에서 동작하는 명령줄 기반 애플리케이션으로, 사용자 인터페이스는 텍스트 입력을 중심으로 구성되어 있다. 기본 GUI 기능은 별도의 플러그인 형태로 제공된다.

2. 네트워크 분산 운용 지원
동일한 MAVLink 데이터를 UDP 네트워크를 통해 다수의 지상국으로 중계할 수 있으며, 여러 컴퓨터 간의 협업 운용이 가능하다.

3. 높은 이식성과 호환성
Python, pyserial, select() 함수 호출이 가능한 POSIX 기반 운영체제(예: Linux, macOS, Windows 등)에서 실행할 수 있어, 다양한 환경에서 활용할 수 있다.

4. 경량 설계
프로그램 구조가 단순하고 효율적으로 설계되어 있어, 성능이 제한된 소형 노트북이나 임베디드 시스템에서도 원활히 동작한다.

5. 모듈형 확장 구조
콘솔, 이동 지도, 조이스틱, 안테나 트래커 등 다양한 기능을 모듈로 추가할 수 있는 구조를 채택하고 있어, 사용자 요구에 따라 손쉽게 기능 확장이 가능하다.

6. 명령 자동 완성 기능
탭(Tab) 키를 이용한 명령 자동 완성 기능을 제공하여, 사용자의 입력 효율성과 정확성을 향상시킨다.


# 홈디렉토리 이동  


cd ~


## home/dronedev/ardu-sim/parameters 생성

mkdir -p ardu-sim/parameters 

## binary 와 default parameter 복사

cp -a $HOME/ardupilot/build/sitl/bin/. $HOME/ardu-sim/ 

## copter & plane & rover default parameters 복사 

cp $HOME/ardupilot/Tools/autotest/default_params/copter.parm $HOME/ardu-sim/parameters/copter.parm

cp $HOME/ardupilot/Tools/autotest/default_params/plane.parm $HOME/ardu-sim/parameters/plane.parm

cp $HOME/ardupilot/Tools/autotest/default_params/rover.parm $HOME/ardu-sim/parameters/rover.parm


## 새로운 커멘트 창 열고 비행체가 임무수행할 logs폴더 생성

cd ~

mkdir -p ardu-sim/logs

cd ardu-sim/logs/

cd $HOME/ardu-sim

## 명지전문대학교 드론 준비
./arducopter -S --model copter --speedup 1 --defaults parameters/copter.parm -I0 --home 36.9675,127.8690,0,0 

# 비행체를 개별적으로 실행.

./arducopter -S --model copter --speedup 1 --defaults parameters/copter.parm -I0 --home 37.5838,126.9253,0,0 # multicopter 실행

./arduplane -S --model plane --speedup 1 --defaults parameters/plane.parm -I0 --home 37.5838,126.9253,0,0 #plane 실행 

./ardurover -S --model rover --speedup 1 --defaults parameters/rover.parm -I0 --home 37.5838,126.9253,0,0 #rover 실행


 #GCS 실행
mavproxy.py --master tcp:127.0.0.1:5760 --map --console

#옵션
--master: 드론과 통신할 주요 연결을 설정합니다. 예: --master tcp:127.0.0.1:5760 또는 --master /dev/ttyACM0와 같이 사용할 수 있습니다.
--out: 추가 출력 연결을 설정합니다. 이렇게 하면 MAVProxy에서 수신한 데이터를 다른 시스템으로 전달할 수 있습니다. 예: --out 127.0.0.1:14550와 같이 사용할 수 있습니다.
--baudrate: 시리얼 연결의 전송 속도를 설정합니다. 예: --baudrate 57600과 같이 사용할 수 있습니다.
--aircraft: 비행 데이터를 저장할 디렉터리를 설정합니다. 예: --aircraft MyDrone와 같이 사용할 수 있습니다.
--sitl: SITL 인스턴스에 대한 연결 정보를 설정합니다. 예: --sitl 127.0.0.1:5501과 같이 사용할 수 있습니다.
--console: MAVProxy 콘솔을 활성화합니다. 콘솔은 상태 정보와 함께 명령 입력을 허용합니다.
--map: 지도 기능을 활성화합니다. 이를 통해 드론의 위치와 경로를 시각적으로 모니터링할 수 있습니다.
--speech: 음성 합성 기능을 사용하여 상태 정보를 오디오로 듣습니다.
--logfile: 특정 로그 파일을 설정합니다. 예: --logfile MyLogFile.bin과 같이 사용할 수 있습니다.
--load-module: MAVProxy 시작 시 추가 모듈을 로드합니다. 예: --load-module module_name과 같이 사용할 수 있습니다.

# Example
mavproxy.py --master tcp:127.0.0.1:5760 --map #지도표현
mavproxy.py --master tcp:127.0.0.1:5760 --consol #GUI 상태창
mavproxy.py --master tcp:127.0.0.1:5760 --version #mavproxy version
mavproxy.py --master tcp:127.0.0.1:5760 --out udp:127.0.0.1:14550 #MAVLink 패킷을 원격 장치로 전달 - QgroundControl 등 다른 GCS 가능
mavproxy.py --master tcp:127.0.0.1:5760 --out remote_IP:14550


# QgroundControl 설치순서
sudo usermod -a -G dialout $USER # dialout 그룹에 추가합니다
sudo apt-get remove modemmanager -y
sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y # 비디오 및 오디오 스트리밍을 처리하기 
sudo apt install libqt5gui5 -y # 그래픽 사용자 인터페이스(GUI)를 생성하는 
sudo apt install libfuse2 -y # 다양한 파일 시스템을 구현하기 


# QgroundControl 다운로드 
wget https://d176tv9ibo4jno.cloudfront.net/latest/QGroundControl.AppImage #wget 명령으로 QGC 다운로드

chmod +x ./QGroundControl.AppImage # 파일 사용권한 설정 옆같이 해도 실행됨. chmod +x $HOME/Downloads/QGroundControl.AppImage

# chmod 
chmod ugo +/- rwx 파일명 또는 디렉토리명: 이 옵션은 파일 또는 디렉토리의 권한을 변경합니다. u는 소유자, g는 그룹, o는 다른 사용자를 나타냅니다. r은 읽기, w는 쓰기, x는 실행 권한을 나타냅니다. +는 권한을 추가하고 -는 권한을 제거합니다.
chmod a +/- rwx 파일명 또는 디렉토리명: 이 옵션은 모든 사용자에 대한 권한을 변경합니다. a는 all을 나타냅니다.
chmod 0xxx 파일명 또는 디렉토리명: 이 옵션은 파일 또는 디렉토리의 권한을 8진수 숫자로 변경합니다. 각 자릿수는 읽기, 쓰기 및 실행 권한을 나타냅니다.
chmod -R: 이 옵션은 디렉토리와 그 하위 디렉토리의 모든 파일에 대해 권한을 재귀적으로 변경합니다.



# Mavproxy + Ardupilot + QgroundControl 실행순서

# copter 객체 실행
./arducopter -S --model copter --speedup 1 --defaults parameters/copter.parm -I0 --home 37.5838,126.9253,0,0


#QgroundControl 폴더로이동 14550port로 실행

cd Downloads
./QGroundControl.AppImage

# mavproxy에서 out을 u에:127.0.0.1:14550 QGC로 
mavproxy.py --master tcp:127.0.0.1:5760 --out udp:127.0.0.1:14550

