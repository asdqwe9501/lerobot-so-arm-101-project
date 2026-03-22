# LEROBOT SOARM101 PROJECT
![Image](https://github.com/user-attachments/assets/bbfb79ad-0a2f-4016-ad69-124146166fcf)

---


# LeRobot SO-ARM 101 Setup Guide

이 문서는 **LeRobot SO-ARM 101 (Leader / Follower)** 로봇을 조립 및 설정한 뒤,


**모터 인식 → 캘리브레이션 → 원격 조정(Teleoperation)** 까지 수행하는 전체 과정을 정리한 가이드입니다.


| Leader-Arm Axis      | Motor | Gear Ratio |
|-----------------------|-------|------------|
| Base / Shoulder Pan   | 1     | 1 / 191    |
| Shoulder Lift         | 2     | 1 / 345    |
| Elbow Flex            | 3     | 1 / 191    |
| Wrist Flex            | 4     | 1 / 147    |
| Wrist Roll            | 5     | 1 / 147    |
| Gripper               | 6     | 1 / 147    |

로봇 조립 및 모터에 관한 내용 
https://huggingface.co/docs/lerobot/so101


---

## 개발 환경

| 항목        | 내용                              |
| --------- | ------------------------------- |
| OS        | Windows 10 / 11                 |
| 실행 환경     | **Visual Studio Code (VSCode)** |
| Python 버전 | 3.10 이상 권장                      |
| 패키지 설치    | `lerobot`           |
| 실행 터미널    | VSCode 내장 터미널 (PowerShell)      |

>  *콘다 환경에서도 가능하지만, 본 실습은 VSCode 기본 Python 환경에서 진행되었습니다.*
> 
> 패키지 설치 방법 https://huggingface.co/docs/lerobot/installation
> 

---

## 1. 포트 찾기

먼저 로봇의 시리얼 포트를 확인합니다. 
컨트롤 보드에 USB포트와 어댑터를 연결한 후 파이썬 터미널에 아래 명령어를 실행합니다.


영상 참조 https://huggingface.co/docs/lerobot/so101

```bash
lerobot-find-port
```

* 예: `COM3`, `COM4`

---

## 2. 모터 설정 (Motor Setup)

순서에 따라 모터들을 컨트롤 보드에 연결한 후 각 모터 ID를 순서대로 등록합니다.


영상 참조 https://huggingface.co/docs/lerobot/so101


### Follower Arm 설정

```bash
lerobot-setup-motors --robot.type=so101_follower --robot.port=COM3
```

**출력 예시**

```
Connect the controller board to the 'gripper' motor only and press enter.
'gripper' motor id set to 6
...
'shoulder_pan' motor id set to 1
```

---

### Leader Arm 설정

```bash
lerobot-setup-motors --teleop.type=so101_leader --teleop.port=COM4
```

**출력 예시**

```
Connect the controller board to the 'gripper' motor only and press enter.
'gripper' motor id set to 6
...
'shoulder_pan' motor id set to 1
```

---

## 3. 캘리브레이션 (Calibration)

모든 관절의 최소/최대/중앙 위치를 기록하여 동작 범위를 보정합니다.
명령어 실행 한 후 Calibration 영상에 따라 각 모터들은 움직여줍니다. 


영상 참조 https://huggingface.co/docs/lerobot/so101

### 🦿 Follower Arm Calibration

```bash
lerobot-calibrate --robot.type=so101_follower --robot.port=COM3 --robot.id=ty_follower_arm
```

**결과 예시**

```
shoulder_pan    |    767 |   1978 |   3310
shoulder_lift   |    903 |    926 |   3278
elbow_flex      |    896 |   3109 |   3122
wrist_flex      |    844 |   2855 |   3194
wrist_roll      |    112 |   2088 |   3989
gripper         |   2046 |   2046 |   2047
Calibration saved to:
C:\Users\곽동현\.cache\huggingface\lerobot\calibration\robots\so101_follower\ty_follower_arm.json
```

---

### Leader Arm Calibration

```bash
lerobot-calibrate --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=ty_leader_arm
```

**결과 예시**

```
shoulder_pan    |    730 |   1999 |   3238
shoulder_lift   |   2034 |   2042 |   4419
elbow_flex      |      0 |   2030 |   4095
wrist_flex      |    334 |   2361 |   2706
wrist_roll      |    128 |   2035 |   3971
gripper         |   2046 |   2047 |   2053
Calibration saved to:
C:\Users\곽동현\.cache\huggingface\lerobot\calibration\teleoperators\so101_leader\ty_leader_arm.json
```

---

## 4. 원격 조정 (Teleoperation)

리더 암(Leader Arm)의 움직임을 팔로워 암(Follower Arm)에 실시간으로 전달합니다.


영상 참조 https://huggingface.co/docs/lerobot/so101

```bash
lerobot-teleoperate \
  --robot.type=so101_follower  --robot.port=COM3 --robot.id=ty_follower_arm \
  --teleop.type=so101_leader   --teleop.port=COM4 --teleop.id=ty_leader_arm
```

* 리더 암을 움직이면 팔로워 암이 동일한 동작을 수행합니다.
* 연결 성공 시 콘솔에 `"connected"` 메시지가 표시됩니다.

![Image](https://github.com/user-attachments/assets/a6bd1568-5d39-4819-bbc1-994f7ba60087)

---

## 캘리브레이션 데이터 저장 경로

| 구분           | 경로                                                                                       |
| ------------ | ---------------------------------------------------------------------------------------- |
| Follower Arm | `~/.cache/huggingface/lerobot/calibration/robots/so101_follower/ty_follower_arm.json`    |
| Leader Arm   | `~/.cache/huggingface/lerobot/calibration/teleoperators/so101_leader/ty_leader_arm.json` |

---

## 최종 점검

* 각 관절의 움직임이 자연스러운지 확인
* 리더 ↔ 팔로워 실시간 반응 테스트
* 문제가 있을 경우 캘리브레이션 재진행

---

## Act Model
![Image](https://github.com/user-attachments/assets/5ded7f40-a024-43bf-a796-51d15e0cda69)
https://github.com/user-attachments/assets/c66a5c10-1176-42af-a635-e11d7ce82278


ACT Policy-based Robotic Sorting Task

In this project, we implemented a robotic sorting task using the ACT Policy model. The robot was trained to pick up two pens placed on a desk and move them into a container. The overall workflow consisted of dataset collection, additional data collection, model training, and execution of the trained robot.

To create the initial dataset, demonstration data was collected through direct human control of the leader arm. The follower arm replicated the movements of the leader arm in real time. During this process, two cameras—a front-facing camera and a top-down camera—were used to capture the task environment and record visual observations along with the robot's motion data.

```
lerobot-record --robot.type=so101_follower --robot.port=COM3 --robot.id=my_awesome_follower_arm \
--robot.cameras="{ front: {type: opencv, index_or_path: 0, width: 640, height: 360, fps: 30},
top: {type: opencv, index_or_path: 1, width: 640, height: 360, fps: 30}}" \
--teleop.type=so101_leader --teleop.port=COM4 --teleop.id=my_awesome_leader_arm \
--display_data=false --dataset.repo_id=taeyoungss/Model_dataset2 \
--dataset.num_episodes=20 --dataset.single_task="Grab the pens and move it into the box"
```

After generating the initial dataset, additional demonstrations of the same task were collected to increase data diversity and improve the robustness of the model. The --resume=true option was used to append new data to the existing dataset without overwriting previous recordings.

```
lerobot-record --robot.type=so101_follower --robot.port=COM3 --robot.id=my_awesome_follower_arm \
--robot.cameras="{ front: {type: opencv, index_or_path: 0, width: 480, height: 640, fps: 30},
top: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}}" \
--teleop.type=so101_leader --teleop.port=COM4 --teleop.id=my_awesome_leader_arm \
--display_data=false --dataset.repo_id=taeyoungss/act_cleandesk \
--dataset.num_episodes=24 --dataset.single_task="Grab the stick and move it into the box" \
--resume=true
```

The collected dataset was then used to train the ACT Policy model. After training, the learned model was deployed on the robot and tested in a real-world environment to evaluate its performance.

```
lerobot-record \
--robot.type=so101_follower \
--robot.port=COM3 \
--robot.id=my_awesome_follower_arm \
--robot.cameras="{ front: {type: opencv, index_or_path: 0, width: 640, height: 360, fps: 15},
top: {type: opencv, index_or_path: 1, width: 640, height: 360, fps: 15}}" \
--display_data=false \
--dataset.repo_id=taeyoungss/eval_Act_Model2 \
--dataset.num_episodes=5 \
--dataset.reset_time_s=5 \
--dataset.single_task="Grab the stick and move it into the box" \
--policy.path=taeyoungss/Act_Model
```

Experimental results showed that the robot was able to successfully detect, grasp, and move the pens into the container. The overall success rate was approximately 70%. Failures mainly occurred due to slippage during grasping or when the pens were placed in positions not sufficiently represented in the training dataset.

This experiment demonstrates that the ACT Policy model can be effectively applied to simple robotic manipulation and organization tasks. Future improvements, such as increasing the dataset size and extending the training duration, are expected to further enhance the robot’s performance and reliability.




You can download and watch the model demonstration video from the link below.


https://huggingface.co/taeyoungss




# References 

https://hyundoil.tistory.com/469


https://github.com/huggingface/lerobot


https://huggingface.co/docs/lerobot/so101

https://www.youtube.com/watch?v=ElZvzKRt9g8&t=150s

https://roboseasy.github.io/docs/physical-ai/lerobot/so-arm/software/record-replay

