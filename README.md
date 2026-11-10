# TurtleBot3 화재 탐색 및 진압 로봇

TurtleBot3에 자율 탐색(Frontier), Nav2, YOLO 화재 감지, Arduino Nano 기반 물 분사 장치를 결합한 화재 탐색·진압 프로젝트입니다.

로봇은 주변 공간을 자율 탐색하고 카메라 영상에서 화재를 감지합니다. 화재가 발견되면 이동과 방향 정렬을 수행한 뒤 Arduino Nano에 진압 명령을 전달합니다.

## 프로젝트 구성

- `ros2_ws/`
  - YOLOv8 모델 기반 화재 감지 노드
  - Frontier 자율 탐색 패키지
  - 전체 탐색·이동·진압 과정을 관리하는 제어 노드
  - ROS2 사용자 정의 메시지 및 서비스
- `turtlebot3_ws/`
  - TurtleBot3 bringup, Cartographer, Navigation2 설정
  - TurtleBot3 노드, 메시지, DynamixelSDK 및 모델 파일
- `sketch_mar11a.ino`
  - Arduino Nano용 물 분사 장치 제어 코드
- `ArduiNode/`
  - Arduino 연동 과정에서 사용된 서비스 인터페이스 및 빌드 기록

## 포함된 외부 프로젝트

이 저장소에는 프로젝트 재현을 위해 다음 오픈 소스 프로젝트의 소스가 포함되어 있습니다. 저작권과 라이선스는 각 디렉터리의 `LICENSE` 파일을 따릅니다.

- [frontier-exploration-ros2](https://github.com/mertgulerx/frontier-exploration-ros2) — 기준 커밋 `1992202`
- [ROBOTIS TurtleBot3](https://github.com/ROBOTIS-GIT/turtlebot3) — 기준 커밋 `1f67e8d`
- [ROBOTIS DynamixelSDK](https://github.com/ROBOTIS-GIT/DynamixelSDK) — 기준 커밋 `69b30fc`
- [ROBOTIS turtlebot3_msgs](https://github.com/ROBOTIS-GIT/turtlebot3_msgs) — 기준 커밋 `9209e01`

일부 설정 파일은 본 프로젝트의 로봇 구성에 맞게 수정되어 있습니다.

## 구동 전 확인 사항

### 실행 환경

- ROS 2 배포판과 각 워크스페이스의 패키지 버전이 호환되는지 확인합니다.
- Nav2, Cartographer, LiDAR 드라이버, 카메라 드라이버, `cv_bridge`, `rqt_image_view` 등 시스템 의존성을 설치해야 합니다.
- YOLO 실행을 위한 Python, OpenCV, PyTorch 및 `ultralytics` 환경이 필요합니다.
- `TURTLEBOT3_MODEL`과 `LDS_MODEL` 환경 변수를 실제 장비에 맞게 설정합니다.

### 파일 경로

일부 코드와 launch 파일에는 개발 당시의 절대 경로가 포함되어 있습니다. 실행 장비에서 다음 항목을 실제 위치에 맞게 수정하거나 동일한 경로에 배치해야 합니다.

- YOLO 가중치 `best.pt`
- TurtleBot3 Navigation2의 `burger.yaml`

### 카메라 토픽

화재 감지 노드는 `/image_raw`를 구독합니다. 카메라가 `/camera/image_raw` 등 다른 이름으로 영상을 발행한다면 토픽 리매핑이 필요합니다.

### USB 시리얼 포트

기본 설정에서는 LiDAR와 Arduino 노드가 모두 `/dev/ttyUSB0`을 사용할 수 있으므로 충돌 여부를 반드시 확인해야 합니다.

예상 구성은 다음과 같습니다.

```text
OpenCR       /dev/ttyACM0
LiDAR        /dev/ttyUSB0
Arduino Nano /dev/ttyUSB1 또는 /dev/ttyACM1
```

장치 번호는 재부팅 후 달라질 수 있으므로 가능하면 `/dev/serial/by-id/` 경로를 사용합니다. Arduino Nano는 115200 baud로 통신합니다.

### Arduino 및 진압 장치

- `sketch_mar11a.ino`를 Nano에 업로드합니다.
- 서보, 불꽃 센서, 펌프 제어 핀이 실제 배선과 일치하는지 확인합니다.
- 펌프는 Nano 핀에 직접 연결하지 말고 적절한 드라이버, 별도 전원 및 보호회로를 사용합니다.
- 물을 사용하기 전에 펌프 정지와 서비스 완료 응답이 정상 동작하는지 건식 테스트합니다.

### 실행 전 통신 확인

전체 시스템을 구동하기 전에 다음 항목이 정상적으로 제공되는지 확인합니다.

- 카메라 영상
- LiDAR `/scan`
- 지도 `/map`
- Odometry `/odom`
- TF 트리
- Nav2 이동 및 회전 액션
- TurtleBot3 속도 명령 `/cmd_vel`
- Arduino 진압 서비스 `extinguish_fire`

Arduino 연동 노드는 메인 launch에 포함되지 않을 수 있으므로 필요하면 별도로 실행합니다.

```bash
ros2 run turtlebot_control_node arduino_node
```

실제 물 분사 시험은 로봇의 주행·화재 감지·Arduino 통신을 각각 검증한 뒤 안전한 환경에서 진행하십시오.



이 GitHub 저장소는 포트폴리오 열람 목적으로 공개 처리되었습니다.

저장소에 포함된 외부 오픈소스 구성요소에는 각 구성요소의 기존 라이선스와 저작권 조건이 적용됩니다.

별도의 라이센스가 부여되지 않은 자체 제작 코드의 저작권은 원 저작자에게 있습니다.
