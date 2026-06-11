# ACT Policy-Based Robot Imitation Learning System: Implementation and Principles

---

## 1. System Overview

This system is an **imitation learning system where a robot learns from human demonstrations and performs tasks autonomously**.

* When a human demonstrates picking up a pen and placing it into a container, the robot **learns from camera and sensor data**.
* The trained model **predicts future sequential actions (Action Chunks)** to drive the robot arm smoothly.

### Components

1. **Leader Arm (Teacher Arm)**
   * Records expert demonstrations → generates ground-truth actions (Action Labels)
   * Provides reference data for training the follower arm

2. **Follower Arm (Student Arm)**
   * Training phase: learns by replicating leader arm data
   * Execution phase: performs the actual pen-sorting task using the trained model

3. **Camera (Vision Sensor)**
   * Observes the environment → generates state observation data
   * Images → CNN → compressed into **feature vectors**

4. **ACT Policy Model**
   * Input: camera images + joint states
   * Output: future multi-step action sequence (Action Chunk)
   * Optional CVAE: extracts demonstration style (z) → enables diverse motion generation

In short:

> "A system where the robot watches a human sort pens, learns from the demonstration, and autonomously performs the same task smoothly and naturally using cameras and sensors."

---

## 2. Hardware Configuration and Roles

| Device           | Role                        | Description                                                                                                                                          |
| ---------------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Leader Arm**   | Demonstration data capture  | Records the expert's joint movements during manual operation, generating ground-truth action labels corresponding to each state at every time step.  |
| **Follower Arm** | Training and execution      | Training phase: reproduces demonstration data. Execution phase: carries out commands from the trained model.                                         |
| **Camera**       | Environment observation     | Observes the positions of pens and the container; converts images into feature vectors.                                                              |
| **Sensors**      | Joint state acquisition     | Measures leader arm joint angles, velocities, and grip state → embedded via linear layers.                                                           |

---

## 3. Dataset Construction

### 3-1. Environment Setup

* Arrange pens, container, and workspace
* Vary positions, layouts, and lighting conditions → enables the model to generalize

### 3-2. Recording Demonstrations

* Perform pen-sorting demonstrations using the leader arm
* Simultaneously:
  * Record joint sensor data
  * Capture camera footage
* Save the corresponding **ground-truth action labels (Action Labels)** for each frame

### 3-3. Generating State-Action Pairs

* Images → CNN → **Feature Vectors**
* Joint states → Linear layer → Embeddings
* Generate `(feature vector + joint embedding, Action Label)` pairs

**Feature Vector**
Compressed numerical representation of an image that the model uses to understand the visual scene.

**Joint Embedding**
A transformed representation of the robot's joint states (angles, positions, etc.) in a format the model can process effectively.

### 3-4. Dataset Structure Example

```text
dataset/
├── images/          # Camera frames
│   ├── frame_0001.png
│   ├── frame_0002.png
├── actions/         # Leader arm actions
│   ├── frame_0001.txt
│   ├── frame_0002.txt
└── metadata.csv     # Image ↔ action mapping
```

### 3-5. Purpose of Dataset Construction

* Obtain **accurate state-action mappings**
* Improve **generalization** → prevent the model from memorizing specific coordinates
* Enable learning of **sequential actions (Action Chunks)**

---

## 4. ACT Policy Learning Mechanism

### 4-1. Inputs and Outputs

| Element              | Description                                        |
| -------------------- | -------------------------------------------------- |
| Input (Observation)  | Camera images, robot joint states                  |
| Output (Action Chunk)| Future sequential action series                    |
| Ground-Truth Label   | Actual action sequence from expert demonstrations  |

**What is a Sequence?**
A sequence is a bundle of data arranged in chronological order.

### 4-2. Architecture

1. **Vision + Proprioception Encoder**
   * Images → CNN → Feature Vectors
   * Joint states → Linear layer → Embeddings
   * Two vectors combined → fed into Transformer

2. **Transformer-Based Sequence Model**
   * Understands context from past states → predicts future Action Chunks

3. **Optional CVAE**
   * Extracts demonstration style (z)
   * Enables diverse motion generation when needed → produces naturally flowing action sequences

### 4-3. Training Process

1. Collect demonstration data → generate state-action pairs
2. Encoder → produce state vectors
3. Decoder → predict future action sequence (Action Chunk)
4. Calculate loss → update model parameters
5. Repeat → model converges


### 4-4. Inference (Execution) Process

* Input current state → predict Action Chunk
* Temporal Ensembling → execute sequential actions smoothly
* Follower arm carries out the actual motions

<img width="1376" height="768" alt="Image" src="https://github.com/user-attachments/assets/91058674-7f47-4176-b486-565b58874d3e" />

---

## 5. Advantages of ACT Policy

* Capable of long-horizon sequence prediction → smooth, continuous motion
* Reduces error accumulation → more stable than single-step prediction
* Can be trained with relatively few demonstrations
* Transformer + optional CVAE → integrates visual and sequential information

---

## 6. Conclusion

* This system combines a **leader-follower robot structure, camera-based observation, and ACT Policy**
* Dataset construction and the learning architecture are central; **sequential action prediction and Transformer-based learning** make it effective for real-world robotic tasks
* The optional CVAE enables **diverse motion style generation**, improving generalization performance

---

## References and Image Sources
https://wikidocs.net/327258
