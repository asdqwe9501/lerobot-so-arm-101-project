# LEROBOT SOARM101 PROJECT
![Image](https://github.com/user-attachments/assets/bbfb79ad-0a2f-4016-ad69-124146166fcf)

---


# LeRobot SO-ARM 101 Setup Guide

This document is a complete guide covering the entire process from assembling and configuring the
**LeRobot SO-ARM 101 (Leader / Follower)** robot to performing

**motor detection → calibration → teleoperation**.



| Leader-Arm Axis      | Motor | Gear Ratio |
|-----------------------|-------|------------|
| Base / Shoulder Pan   | 1     | 1 / 191    |
| Shoulder Lift         | 2     | 1 / 345    |
| Elbow Flex            | 3     | 1 / 191    |
| Wrist Flex            | 4     | 1 / 147    |
| Wrist Roll            | 5     | 1 / 147    |
| Gripper               | 6     | 1 / 147    |

For robot assembly and motor details:  https://huggingface.co/docs/lerobot/so101


---

## Development Environment

| Item        | Details                              |
| --------- | ------------------------------- |
| OS        | Windows 10 / 11                 |
| IDE     | Visual Studio Code  |
| Python version | 3.10 or higher recommended                      |
| Package    | Lerobot           |
| Terminal    | VSCode PowerShell      |

>  * Conda environments are also supported, but this guide was written using VSCode's default Python environment.*
>
> 
> Package installation: https://huggingface.co/docs/lerobot/installation
> 

---

## 1. Finding the Port

First, identify the serial port of the robot.
Connect the USB port and adapter to the control board, then run the following command in the Python terminal.


Reference: https://huggingface.co/docs/lerobot/so101

```bash
lerobot-find-port
```

* Example: `COM3`, `COM4`

---

## 2. Motor Setup

Connect the motors to the control board in order, then register each motor ID sequentially.


Reference: https://huggingface.co/docs/lerobot/so101


### Follower Arm setup

```bash
lerobot-setup-motors --robot.type=so101_follower --robot.port=COM3
```

**Example output**

```
Connect the controller board to the 'gripper' motor only and press enter.
'gripper' motor id set to 6
...
'shoulder_pan' motor id set to 1
```

---

### Leader Arm setup

```bash
lerobot-setup-motors --teleop.type=so101_leader --teleop.port=COM4
```

**Example output**

```
Connect the controller board to the 'gripper' motor only and press enter.
'gripper' motor id set to 6
...
'shoulder_pan' motor id set to 1
```

---

## 3. Calibration

Records the minimum, maximum, and center positions of each joint to calibrate the range of motion.
After running the command, move each motor according to the calibration video.


Reference: https://huggingface.co/docs/lerobot/so101

###  Follower Arm Calibration

```bash
lerobot-calibrate --robot.type=so101_follower --robot.port=COM3 --robot.id=ty_follower_arm
```

**Example output**

```
shoulder_pan    |    767 |   1978 |   3310
shoulder_lift   |    903 |    926 |   3278
elbow_flex      |    896 |   3109 |   3122
wrist_flex      |    844 |   2855 |   3194
wrist_roll      |    112 |   2088 |   3989
gripper         |   2046 |   2046 |   2047
Calibration saved to:
C:\Users\username\.cache\huggingface\lerobot\calibration\robots\so101_follower\ty_follower_arm.json
```

---

### Leader Arm Calibration

```bash
lerobot-calibrate --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=ty_leader_arm
```

**Example output**

```
shoulder_pan    |    730 |   1999 |   3238
shoulder_lift   |   2034 |   2042 |   4419
elbow_flex      |      0 |   2030 |   4095
wrist_flex      |    334 |   2361 |   2706
wrist_roll      |    128 |   2035 |   3971
gripper         |   2046 |   2047 |   2053
Calibration saved to:
C:\Users\username\.cache\huggingface\lerobot\calibration\teleoperators\so101_leader\ty_leader_arm.json
```

---

## 4. Teleoperation

Transmits the movements of the Leader Arm to the Follower Arm in real time.


Reference: https://huggingface.co/docs/lerobot/so101

```bash
lerobot-teleoperate \
  --robot.type=so101_follower  --robot.port=COM3 --robot.id=ty_follower_arm \
  --teleop.type=so101_leader   --teleop.port=COM4 --teleop.id=ty_leader_arm
```

- Moving the leader arm causes the follower arm to replicate the same motion.
- Upon successful connection, a `"connected"` message will appear in the console

![Image](https://github.com/user-attachments/assets/a6bd1568-5d39-4819-bbc1-994f7ba60087)

---

## Calibration Data Storage Path

| Type           | Path                                                                                       |
| ------------ | ---------------------------------------------------------------------------------------- |
| Follower Arm | `~/.cache/huggingface/lerobot/calibration/robots/so101_follower/ty_follower_arm.json`    |
| Leader Arm   | `~/.cache/huggingface/lerobot/calibration/teleoperators/so101_leader/ty_leader_arm.json` |

---

## Final Checklist

- Verify that each joint moves smoothly
- Test real-time response between leader ↔ follower
- If issues occur, redo the calibration

---

## Act Model
![Image](https://github.com/user-attachments/assets/5ded7f40-a024-43bf-a796-51d15e0cda69)
![Image](https://github.com/user-attachments/assets/eb2c0fe5-aa5d-40b3-b602-832857d1e9a1)


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


### SmolVLA Pick-and-Place Project

### Project Overview

The earlier ACT experiment achieved about a 70% success rate on its pick-and-place task. Building on that, this project tested whether switching to a SmolVLA policy — with the object changed to a red ball — could handle the same pick-and-place task, this time guided by a natural-language instruction ("grab the red ball and put it in the box") rather than a fixed motion.

### What is SmolVLA?

SmolVLA is a compact Vision-Language-Action model built by Hugging Face for real-world robot control. In plain terms, it's a small AI model that looks at camera images, reads a text instruction, and outputs a sequence of motor commands — all in one forward pass. It's built from three pieces working together:


A vision-language backbone (a small pretrained vision-language model) that turns camera frames and the instruction text into a shared understanding — essentially "seeing" the scene and "reading" the goal at the same time.
An action expert, a lightweight network that translates that understanding into continuous robot joint movements using a technique called flow matching, which lets it generate smooth, precise motion trajectories rather than jerky discrete steps.
Efficiency-focused design: because it's "Smol" (small), it can run on a single consumer GPU or even a CPU, and it supports asynchronous inference — the robot can keep moving while the next action chunk is being computed, so there's less lag between "thinking" and "doing."


The key advantage over the earlier ACT policy is generalization through language: instead of memorizing "move to this fixed spot," SmolVLA learns to associate the words "red ball" and "box" with visual features, so it can find and grasp the object wherever it happens to be placed.

### Data & Training

Fifty demonstration episodes were collected via teleoperation (a human guiding the leader arm), recorded from both a front-facing and a top-down camera, all paired with the same instruction string. The pretrained lerobot/smolvla_base checkpoint was then fine-tuned on this dataset for 30,000 steps. Image color/saturation augmentation was deliberately turned off, since it was found to interfere with the model's ability to correctly ground the word "red" to the object's actual color.

### Results

The trained policy was tested over 5 real-robot trials and succeeded in 4 of them (~80% success rate) — an improvement over the ~70% baseline from the ACT experiment. Notably, the ball's position on the desk was varied between trials, and the robot still located and grasped it correctly each time — evidence that the grasp was driven by recognizing the object visually, not by memorizing a fixed location. The one failure was a case where the arm froze near the start of the episode and never recovered, rather than a targeting or grasping mistake.

### Next Steps

Planned extensions include training on multiple object colors so the color word in the instruction actually determines the target (true language grounding, not just a single fixed phrase), and integrating Whisper speech-to-text so the instruction can be spoken aloud instead of typed.

You can download and watch the model demonstration video from the link below.


https://huggingface.co/taeyoungss




# References 

https://hyundoil.tistory.com/469


https://github.com/huggingface/lerobot


https://huggingface.co/docs/lerobot/so101

https://www.youtube.com/watch?v=ElZvzKRt9g8&t=150s

https://roboseasy.github.io/docs/physical-ai/lerobot/so-arm/software/record-replay

