# summary
  대회 전 준비물

  우리가 만들어둬야 할 것은:

  1. raw bag 녹화 커맨드
  2. offline replay + FAST-LIO 실행 절차
  3. bag 진단 스크립트
  4. trajectory/PCD preview 스크립트
  5. 정상/실패 판정 기준
  6. 최종 nav map 생성 커맨드

  이미 준비된 스크립트:

  scripts/diagnose_fastlio_bag.py
  scripts/plot_fastlio_bag.py
  scripts/extract_laser_map_from_bag.py
  scripts/preview_pcd_map.py

  가장 중요한 오늘의 결론

  대회장에서 우리가 믿어야 하는 건 live PCD가 아닙니다.

  믿을 것: /livox/lidar + /livox/imu raw bag
  검증할 것: /Odometry trajectory
  최종 산출물: 검증 통과한 PCD map

  그러니까 여기서 먼저 CustomMsg 짧은 주행 테스트를 성공시켜야 합니다. 그 다음 raw bag replay로 같은
  맵을 만들 수 있는지 확인하면 대회 운영 방향이 잡힙니다.


› 그럼 그 목표까지 지금 어느정도 달성했어


• 현재 달성도는 **약 60~70%**입니다.

  완료된 것:

  1. PCD 후처리/crop/voxel 문제가 아님을 확인
  2. /Odometry, /tf 자체가 발산한다는 것 확인
  3. static bag은 정상, moving bag에서 발산 확인
  4. mid360_debug_ bag 분석 완료
  5. FAST-LIO 발산 증거 확보
  6. bag 진단 스크립트 작성
  7. odometry 시각화 스크립트 작성
  8. PCD preview 스크립트 작성
  9. /Laser_map -> PCD 추출 확인
  10. PointCloud2 timestamp/backlog가 유력 원인이라는 증거 확보
  11. CustomMsg 경로로 전환 패치 완료
  12. fast_lio, livox_ros_driver2 빌드 완료

  아직 못 끝낸 핵심:

  1. CustomMsg 상태에서 새 주행 bag이 정상인지 검증
  2. raw bag replay로 offline FAST-LIO map 생성 검증
  3. navigation에 쓸 수 있는 최종 PCD 생성 검증

  현재 가장 중요한 남은 증명은 이겁니다:

  CustomMsg로 바꾼 뒤 FAST-LIO가 정상 주행거리 수준으로 나오는가?

  이게 성공하면 목표의 85~90%까지 갑니다.

  그 다음:

  raw bag replay -> PCD 생성 -> bounds/preview 정상

  까지 성공하면 대회 첫날 운영 전략까지 확정이라 **거의 100%**입니다.

  지금 상태를 한 줄로 말하면:

  원인 후보는 거의 잡았고, 진단/시각화/패치는 준비됐지만, CustomMsg 재주행 검증이 아직 남았다.

  다음에 해야 할 바로 그 테스트:

  ros2 topic info /livox/lidar

  여기서:

  Type: livox_ros_driver2/msg/CustomMsg

  확인 후 30초 정도 천천히 녹화:

  ros2 bag record /livox/lidar /livox/imu /Odometry /tf -o bags/custommsg_debug_$(date +
  %Y%m%d_%H%M%S)


  # To-do List 
# 대회장 전 준비 TODO

## 1. CustomMsg 입력 검증

  목표: PointCloud2 timestamp 문제를 피하고 FAST-LIO가 정상 동작하는지 확인한다.

  ### 확인

  ```bash
  ros2 topic info /livox/lidar

  정상 기대:

  Type: livox_ros_driver2/msg/CustomMsg

  ### 짧은 테스트 bag 저장

  cd /home/jairlab/go2_ws
  source /opt/ros/humble/setup.bash
  source install/setup.bash

  ros2 bag record /livox/lidar /livox/imu /Odometry /tf \
    -o bags/custommsg_debug_$(date +%Y%m%d_%H%M%S)

  ### 분석

  python3 scripts/diagnose_fastlio_bag.py bags/custommsg_debug_xxx --sample-clouds 12

  python3 scripts/plot_fastlio_bag.py bags/custommsg_debug_xxx \
    --output maps/custommsg_debug_plot.png

  ### 성공 기준

  /Odometry range가 실제 주행거리 수준
  frame displacement max가 수 m 이하
  z가 수십 km로 떨어지지 않음
  /livox/lidar point offset range가 약 0~100ms 수준

  ———

  ## 2. Raw Bag Backup 절차 검증

  목표: 대회장에서 live mapping이 실패해도 raw bag으로 offline map을 다시 만들 수 있게 한다.

  ### raw bag 저장

  ros2 bag record /livox/lidar /livox/imu \
    -o bags/raw_custommsg_test_$(date +%Y%m%d_%H%M%S)

  ### offline replay

  터미널 1:

  ros2 bag play bags/raw_custommsg_test_xxx -r 0.3

  터미널 2:

  cd /home/jairlab/go2_ws
  source /opt/ros/humble/setup.bash
  source install/setup.bash

  ./scripts/start_fastlio_mid360.sh

  ### 성공 기준

  FAST-LIO가 replay 속도를 따라감
  /Odometry 발산 없음
  PCD map bounds가 실제 공간 크기 수준

  ———

  ## 3. PCD Map 생성 및 검증

  목표: FAST-LIO 결과가 navigation에 쓸 수 있는 map인지 확인한다.

  ### /Laser_map에서 PCD 추출

  python3 scripts/extract_laser_map_from_bag.py \
    bags/custommsg_debug_xxx \
    maps/site_raw_map.pcd

  또는 FAST-LIO 직접 저장 결과 확인:

  /home/jairlab/go2_ws/maps/raw_map.pcd

  ### PCD preview 생성

  python3 scripts/preview_pcd_map.py \
    maps/site_raw_map.pcd \
    maps/site_raw_map_preview.png

  ### 성공 기준

  x/y bounds가 실제 주행거리 수준
  z bounds가 수 m~십수 m 수준
  km 단위 발산 없음
  복도 layer가 위아래로 중복되지 않음
  맵이 실제 복도 구조처럼 이어짐

  ———

  ## 4. Navigation용 Map 변환

  목표: TRG planner에 넘길 crop/voxel map을 만든다.

  ### 예시

  python3 src/go2_competition_nav/go2_competition_nav/prepare_pcd_map.py \
    maps/site_raw_map.pcd \
    maps/site_nav_map.pcd \
    --voxel 0.10 \
    --crop-min <xmin> <ymin> <zmin> \
    --crop-max <xmax> <ymax> <zmax>

  ### 성공 기준

  불필요한 먼 outlier 제거됨
  복도 구조 유지됨
  포인트 수가 planner가 처리 가능한 수준
  TRG planner 입력으로 사용 가능

  ———

  ## 5. TRG Planner 주행 검증

  목표: 생성한 PCD map으로 실제 로봇이 주행 가능한지 확인한다.

  ### 확인할 것

  map frame / odom frame / base_link frame 연결 정상
  로봇 초기 위치가 map 안에 있음
  planner가 path 생성
  local controller가 path를 따라감
  장애물/벽 근처에서 충돌 없이 주행

  ### 성공 기준

  짧은 목표점 도달 성공
  복도 직진 성공
  회전 포함 경로 성공
  긴 경로 주행 성공

  ———

  ## 6. 반복성 검증

  목표: 한 번 성공이 아니라 반복 가능한 절차인지 확인한다.

  ### 최소 반복 테스트

  1. static 30초
  2. 짧은 복도 30~60초
  3. 회전 포함 복도 1~2분
  4. 실제 대회 동선 유사 경로

  ### 각 테스트 후 확인

  python3 scripts/diagnose_fastlio_bag.py <bag> --sample-clouds 12
  python3 scripts/plot_fastlio_bag.py <bag> --output <plot.png>
  python3 scripts/preview_pcd_map.py <pcd> <preview.png>

  ———

  ## 7. 대회장용 운영 스크립트 만들기

  목표: 대회장에서 커맨드 실수를 줄인다.

  만들 스크립트:

  scripts/record_raw_bag.sh
  scripts/record_debug_bag.sh
  scripts/replay_fastlio_map.sh
  scripts/diagnose_bag.sh
  scripts/make_nav_map.sh

  각 스크립트에 포함할 것:

  source /opt/ros/humble/setup.bash
  source /home/jairlab/go2_ws/install/setup.bash
  bag/map 이름 timestamp 자동 생성
  필수 topic 명확히 지정

  ———

  ## 8. MID360 네트워크 체크

  목표: 대회장에서 Livox driver bind failed 방지.

  ### 확인

  ip -br addr

  MID360 연결 NIC에 있어야 함:

  192.168.123.5/24

  없으면:

  sudo ip addr add 192.168.123.5/24 dev enp46s0

  ### driver 확인

  ./scripts/start_mid360_fastlio_driver.sh

  정상 기준:

  bind failed 없음
  /livox/lidar publish 됨
  /livox/imu publish 됨

  ———

  ## 9. 대회장 첫날 권장 운영 순서

  1. MID360 네트워크 확인
  2. /livox/lidar type이 CustomMsg인지 확인
  3. static raw bag 30초 저장
  4. 짧은 복도 raw/debug bag 저장
  5. 실제 동선 raw/debug bag 저장
  6. offline FAST-LIO replay로 map 생성
  7. diagnose/plot/PCD preview로 정상 판정
  8. nav map 생성
  9. TRG planner 짧은 주행 테스트
  10. 긴 경로 주행 테스트

  ———

  ## 최종 성공 기준

  CustomMsg 기반 FAST-LIO가 발산하지 않음
  raw bag replay로 map 재생성 가능
  PCD bounds가 실제 공간 크기 수준
  TRG planner가 생성 map으로 실제 주행 성공
  대회장에서 raw bag backup 절차 확보



  그 bag이 정상으로 나오면 큰 산은 넘은 겁니다.
