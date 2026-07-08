# Autonomous Maze-Solving Robot

A cool project featuring a 4WD autonomous robot built with an Arduino Mega. It uses an ultrasonic sensor mounted on a servo motor to scan its surroundings, detect walls, and smart-navigate its way through a maze without hitting obstacles.

---

## How It Works

* **The Goal:** The robot needs to navigate from the start of a maze to the end completely on its own. 
* **The Rules:** Once you turn it on, no touching! It gets 4 attempts to find the absolute fastest route through the maze.

---

## Hardware Setup & Pin Mapping 🔌

### 1. Control Logic (Arduino Mega)
Here is how everything connects to the board:

| Component | Component Pin | Arduino Mega Pin | What it does |
| :--- | :--- | :--- | :--- |
| **Ultrasonic Sensor** | Trig | Pin 11 | Sends out the sonar pulse |
| | Echo | Pin 12 | Listens for the bounce-back to calculate distance |
| **Servo Motor** | Signal | Pin 3 | Rotates the ultrasonic sensor left and right |
| **L298N Motor Driver**| ENA | Pin 5 | Controls Right Motor Speed (PWM) |
| | IN1 | Pin 7 | Controls Right Motor Direction A |
| | IN2 | Pin 8 | Controls Right Motor Direction B |
| | IN3 | Pin 9 | Controls Left Motor Direction A |
| | IN4 | Pin 10 | Controls Left Motor Direction B |
| | ENB | Pin 6 | Controls Left Motor Speed (PWM) |

### 2. The Power Network (Shared Rail Wiring)
To make sure all components communicate properly and stay powered, the wiring utilizes a shared rail approach:
* **Common Ground (GND):** The GND pins from the L298N driver, the Servo motor, and the Ultrasonic sensor are all tied together into a single wire line. This line connects to one of the **GND** pins on the Arduino Mega and the negative (**-ve**) terminal of the battery pack.
* **5V Power Rail:** The **+5V output** from the L298N driver powers the logic side. It links directly to the VCC of the Ultrasonic sensor, the VCC of the Servo, and connects straight into the **5V pin** on the Arduino Mega.
* **12V Motor Power:** The positive (**+ve**) terminal of the battery connects directly into the **+12V** terminal block on the L298N driver to power the motors.

### 3. Actuators (4WD Motor Wiring)
* The chassis uses 4 DC motors.
* **Right Side:** The two right-side motors are wired together in parallel into the **OUT1** and **OUT2** terminals of the L298N.
* **Left Side:** The two left-side motors are wired together in parallel into the **OUT3** and **OUT4** terminals of the L298N.

---

## Libraries Used 📚

This sketch relies on two primary libraries to keep the code efficient and non-blocking:
* **`Servo.h`**: To handle the precise 50Hz PWM signals required to position the servo mount.
* **`NewPing.h`**: A highly optimized library used to trigger and calculate distance from the ultrasonic sensor without lagging the execution loop.

---

## Software Logic & Code Structure

The robot runs on a continuous decision-making loop inside the microcontroller:

1. **Cruising (Forward):** The robot drives forward at a safe regular speed (`PWM = 75`).
2. **Wall Detection:** If the ultrasonic sensor detects a wall closer than **30 cm**, the robot immediately stops, backs up slightly to clear its turning radius, and halts again.
3. **Surrounding Scan:** The servo turns the sensor fully to the left ($180^\circ$) to measure clearance, then fully to the right ($0^\circ$) to check that side, and then centers back to $90^\circ$.
4. **Smart Turn:** * If one side is completely wide open (reads `0` or out of range), the robot instantly turns toward that open path.
   * If both sides have walls, it compares the distances and spins toward whichever side has more open space.

---

## Quick Testing Guide

| Scenario | What should happen? | Result |
| :--- | :--- | :--- |
| **Power On** | Servo centers to $90^\circ$ and motors wait before moving. | Success |
| **Path is Clear ($> 30\text{ cm}$)** | Robot moves straight ahead at cruising speed. | Success |
| **Approaching a Wall ($< 30\text{ cm}$)** | Stops, reverses for 200ms, and starts scanning left/right. | Success |
| **Left side is more open** | Right motor spins forward, left motor reverses (turns left). | Success |
| **Right side is more open** | Left motor spins forward, right motor reverses (turns right). | Success |
