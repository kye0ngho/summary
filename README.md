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

  그 bag이 정상으로 나오면 큰 산은 넘은 겁니다.
