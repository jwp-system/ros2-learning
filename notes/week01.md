## Linux 기초 & ROS2 핵심 개념

목표 : 콘솔 이용 turtlesim 실행 하고 확인하기

1. Linux 기본기 
    
    ```bash
    ## 기본
    pwd; ls -la; cd; mkdir -p; cp; mv; rm -r    # 파일/폴더
    cat; less; head; tail -f; grep -rn "text"   # 파일 읽기와 검색
    chmod +x file; ls -l                        # 권한
    echo $PATH; export MY_VAR=1; printenv       # 환경변수
    which python3; python3 --version
    sudo apt update; apt search <이름>          # 패키지
    
    ## ROS2 기본
    echo $ROS_DISTRO                            # 패키지
    cat ~/.bashrc | tail -5                     # 자동 source 설정 확인
    printenv | grep -i ros                      # ROS 관련 환경변수 모아보기
    ```
    
    ROS2는 환경변수로 동작, 터미널에서 `ros2` 명령이 안된다면 
    `source /opt/ros/jazzy/setup.bash` 가 안 된 것임
    
2. 환경 설정과 turtlesim 실행
    
    공식 튜토리얼 : “Configuring environment” → “Using turtlesim, ros2, and rqt” 
    
    ```bash
    # 터미널 1
    ros2 run turtlesim turtlesim_node
    # 터미널 2
    ros2 run turtlesim turtle_teleop_key      # 이 터미널을 클릭해서 포커스를 둔 채 방향키
    # 터미널 3
    ros2 pkg list | grep turtlesim
    ros2 pkg executables turtlesim            # 이 패키지에서 실행 가능한 노드 목록
    rqt                                       # Plugins → Services → Service Caller 확인
    ```
    
    - 따라해 볼것
        - `/spawn` 서비스로 두 번째 거북이 생성 후 `/turtle2/cmd_ve`r 로 조작하기(튜토리얼의 remap 부분)
        
        ```bash
        ## 두번째 거북이 생성
        ros2 service call /spawn turtlesim/srv/Spawn "{x: 2.0, y: 2.0, theta: 0.0, name: 'turtle2'}"
        
        ## 두번째 거북이 조작
        ros2 run turtlesim turtle_teleop_key --ros-args --remap turtle1/cmd_vel:=turtle2/cmd_vel
        ```
        
3. Node & Topic
    
    공식 튜토리얼 : “Understanding nodes” → “Understanding topics” 
    
    ```bash
    # Node
    ros2 node list
    ros2 node info /turtlesim            # Publishers/Subscribers/Services/Actions 구조를 읽어보기
    ros2 run turtlesim turtlesim_node --ros-args --remap __node:=my_turtle
    ros2 node list                       # 이름이 바뀐 것을 확인
    
    # Topic
    ros2 topic list -t                   # 타입도 함께 표시
    ros2 topic echo /turtle1/cmd_vel     # 텔레옵 키를 누르며 관찰
    ros2 topic info /turtle1/cmd_vel
    ros2 interface show geometry_msgs/msg/Twist
    ros2 topic hz /turtle1/pose          # 발행 주기 측정
    ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist \
      "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
    ros2 topic pub --rate 1 /turtle1/cmd_vel geometry_msgs/msg/Twist \
      "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
    rqt_graph                            # 노드와 토픽의 연결 그림
    ```
    
    - 핵심 이해 : `Twist` 메시지는 로봇에게 “선속도(x,y,z)와 각속도(x,y,z)로 움직여라”를 전달하는 표준 형식, TurtleBot3의 `/cmd_vel`도 같은 메시지
    
    !image.png
    
    !image.png
    
4. Service & Parameter
    
    공식 튜토리어: “Understanding services” → “Understanding parameters” 
    
    - 핵심
        - service와 topic 사용 비교 → 지속적 스트림 vs 일회성 요청
        - parameter 파일 저장 사용 ?(설정 정보)
    
    ```jsx
    # Service
    ros2 service list -t
    ros2 service type /clear
    ros2 interface show std_srvs/srv/Empty
    ros2 service call /clear std_srvs/srv/Empty
    ros2 interface show turtlesim/srv/Spawn
    ros2 service call /spawn turtlesim/srv/Spawn "{x: 2, y: 2, theta: 0.2, name: ''}"
    
    # Parameter
    ros2 param list
    ros2 param get /turtlesim background_r
    ros2 param set /turtlesim background_r 150    # 배경색이 바뀌는지 확인
    ros2 param dump /turtlesim > turtlesim_params.yaml
    cat turtlesim_params.yaml
    ros2 param load /turtlesim turtlesim_params.yaml
    ros2 run turtlesim turtlesim_node --ros-args --params-file turtlesim_params.yaml
    ```
    
5. Action
    
    공식 튜토리얼: “Understanding actions” 
    
    ```jsx
    ros2 node info /turtlesim                      # Action Servers 항목 확인
    ros2 action list -t
    ros2 action info /turtle1/rotate_absolute
    ros2 interface show turtlesim/action/RotateAbsolute
    ros2 action send_goal /turtle1/rotate_absolute \
      turtlesim/action/RotateAbsolute "{theta: 1.57}" --feedback
    ```
    
    !image.png
    
    - 핵심
        - action은 `goal`(목표), `feedback`(진행 중 상태), `result`(최종 결과) 으로 구성, Redis Queue 작업(job)에 `진행률`과 `취소` 가 붙는 것과 비슷
        - service 와 action 사용 목적 ?
6. 로그 & Launch
    
    공식 튜토리얼: “Using rqt_console to view logs” → “Launching nodes” 
    
    ```jsx
    ros2 run rqt_console rqt_console
    ros2 run turtlesim turtlesim_node --ros-args --log-level WARN   # 로그 레벨 조절
    
    ## (참고) 튜토리얼의 launch 파일 예제 실행 -> 3주차 이후
    ros2 launch turtlesim multisim.launch.py
    ```
    
    !image.png
    
7. 데이터 기록/재생
    
    공식 튜토리얼: “Recording and playing back data”
    
    - /turtle1/cmd_vel을 기록 후 재생하면 같은 경로로 움직이는 것을 확인
    
    ```jsx
    # 거북이를 조작하며 기록
    ros2 bag record -o first_run /turtle1/cmd_vel /turtle1/pose
    # (텔레옵으로 거북이를 움직인 뒤 Ctrl+C)
    ros2 bag info first_run
    # 거북이를 clear 한 뒤 재생
    ros2 service call /clear std_srvs/srv/Empty
    ros2 bag play first_run
    ```
    

#### 완료 체크 리스트

- [ ]  터미널만으로 node/topic/service/param/action을 각각 조회하고 호출할 수 있다
- [ ]  `Twist` 메시지의 구조를 설명할 수 있다
- [ ]  `rqt_graph`를 보고 연결 구조를 설명할 수 있다
- [ ]  bag으로 기록한 데이터를 재생할 수 있다