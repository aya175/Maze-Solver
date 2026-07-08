# Autonomous Maze-Solving Robot

An embedded systems project featuring an autonomous robot designed to navigate and solve physical mazes using real-time decision-making algorithms.

---

## Introduction

In today's era, the growth of autonomous robotics has reached extraordinary levels. Autonomous cars, also known as self-driving vehicles, are designed to navigate and reach predetermined destinations without human intervention. Such vehicles possess the ability to intelligently perform tasks on their own, without relying on human assistance, and must be able to navigate through mazes that are not designed for their use. 

This field of robotics is centered around decision-making algorithms, and maze-solving robotics is one of its most significant areas. This project focuses on a maze-solving algorithm and aims to develop a program that emulates all the necessary steps for solving a maze. Additionally, it involves constructing a robot and testing its performance in a real-world maze environment.

---

## The Maze & Evaluation Rules

* **Operation:** The robot must complete the autonomous ride using just one switch to start and one to reset it. Manual intervention is strictly prohibited after the start.
* **Evaluation:** The robot gets four attempts to find the fastest possible route from the start to the end of the maze. The robot that solves the maze in the shortest cumulative or single run time is designated the winner.

---

## System Specifications & Hardware Architecture

### Pin Mapping (Control Logic)

| Component | Component Pin | Arduino Mega Pin | Function |
| :--- | :--- | :--- | :--- |
| **Ultrasonic Sensor** | Trig | Pin 11 | Trigger pulse to start measurement |
| | Echo | Pin 12 | Receive pulse to calculate distance |
| **Servo Motor** | Signal (Orange/Yellow) | Pin 3 | Control the scanning angle |
| **L298N Driver** | ENA | Pin 5 | Right Motor Speed (PWM) |
| | IN1 | Pin 7 | Right Motor Direction A |
| | IN2 | Pin 8 | Right Motor Direction B |
| | IN3 | Pin 9 | Left Motor Direction A |
| | IN4 | Pin 10 | Left Motor Direction B |
| | ENB | Pin 6 | Left Motor Speed (PWM) |

### The Power Network (Shared Rail Approach)

* **Ground (GND) Merging:** The GND of the L298N, the GND of the Servo, and the GND of the Ultrasonic sensor are tied together into a shared wire. This common ground connects directly to an Arduino Mega GND pin and the negative (-ve) terminal of the battery to establish a shared reference point.
* **5V Power Merging:** The +5V regulated output from the L298N driver is merged with the VCC of the Ultrasonic sensor and the VCC of the Servo motor. This shared 5V line connects to the 5V pin of the Arduino Mega to power the logic side.
* **Main Power (12V):** The positive (+ve) terminal of the battery connects directly to the +12V power input terminal on the L298N driver.

### Actuators & Motors

* **Physical Configuration:** Four DC motors drive the chassis.
* **Wiring Setup:** The two DC motors on the right side are wired together in parallel to the OUT1 and OUT2 terminals of the L298N driver. The two DC motors on the left side are wired together in parallel to the OUT3 and OUT4 terminals.

---

## Peripherals & Libraries Used

* **GPIO (General Purpose Input/Output):** Configured via `pinMode()` to output direction controls to the H-bridge pins (`IN1` through `IN4`).
* **PWM (Pulse Width Modulation):** Handled via `analogWrite()` on pins 5 and 6 to regulate DC motor speed dynamically between regular cruising speed and precise adjustment turns.
* **Servo Library:** Utilizes `Servo.h` on pin 3 to dynamically sweep the ultrasonic sensor assembly $0^\circ$, $90^\circ$, and $180^\circ$.
* **NewPing Library:** Employs `NewPing.h` to optimize ultrasonic hardware handling, avoiding blocking delays and filtering echo calculations up to 400 cm.

---

## Software Architecture

### System State Machine
The software operates as a conditional state loop consisting of the following primary runtime states:
1. **FORWARD (Default):** Runs forward at a regular cruising speed (`MAX_REGULAR_MOTOR_SPEED = 75`) as long as the front path is clear.
2. **OBSTACLE DETECTED:** Activates when an obstacle drops closer than `DISTANCE_TO_CHECK = 30` cm.
3. **REVERSE/SAFETY:** Safely stops, backs away at adjustment speeds (`MAX_MOTOR_ADJUST_SPEED = 150`) to clear turning radius space, and halts again.
4. **SCANNING:** Sweeps the servo mount fully to the Left ($180^\circ$) and then Right ($0^\circ$) to record proximity paths.
5. **DECISION & TURNING:** Analyzes environmental variables (prioritizing open paths or sides with maximum clearance values) and executes a precise local pivot spin.

### Core Functions
* `void setup()`: Initialized pin configurations for the H-bridge, centers the servo to $90^\circ$, and ensures the platform starts stationary.
* `void loop()`: Continuously pings the front environment and runs the local branch handling if a wall threshold is crossed.
* `void rotateMotor(int rightMotorSpeed, int leftMotorSpeed)`: Directs directional logic vectors (`HIGH`/`LOW`) based on negative/positive sign arguments, and updates motor speeds using absolute magnitude values passed into `analogWrite()`.

---

## Testing Strategy

| Test Case ID | Scenario Description | Expected Behavior | Status |
| :--- | :--- | :--- | :--- |
| **TC_01** | Power On Initialization | Servo centers to $90^\circ$, motors stay idle for safety. | Pass |
| **TC_02** | Clearance Path Clear ($> 30\text{ cm}$) | Robot drives forward continuously at normal speed (75 PWM). | Pass |
| **TC_03** | Front Obstacle Found ($< 30\text{ cm}$) | Robot stops, reverses for 200ms, and sweeps servo left/right. | Pass |
| **TC_04** | Left Path Clearer / Open | Robot executes right motor forward, left motor reverse spin. | Pass |
| **TC_05** | Right Path Clearer / Open | Robot executes right motor reverse, left motor forward spin. | Pass |
