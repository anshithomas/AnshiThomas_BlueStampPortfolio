# Gesture Controlled Robot
In this project, I am building a hand-gesture controlled robot. Users can control the robot through the hand module. For example, if the user tilts their hand forward, the robot will move forwards. The project contains two main modules: the main car robot and a hand module that is used to control the car. I am using an Arduino Nano 33 BLE Sense with an accelerometer on the hand module to pick up tilt in different directions, and transmits that data wirelessly through a HC-05 Bluetooth module to a second HC-05 module on the robot itself. An Arduino Uno clone reads the incoming signals and and controls a motor driver shield connected to four DC motors, translating hand tilts into forward, backward, left, and right movement. I plan to add a speed boost mode modification, allowing the user to trigger a temporary motor speed increase through an additional gesture.

<!--- You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions: -->
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Anshi T. | Jumeirah College Dubai | Aerospace Engineering | Incoming Senior

<!--- **Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.** --->

![Headstone Image](logo.svg)
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/nz8sKrfr17A?si=N2Qhwa4aTsRMOLTy" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

The second milestone connected both halves of the robot together in software: gesture/tilt detection, wireless communication between the hand and car components, and motor response on the car, working as one system.

The hand controller uses an Arduino Nano 33 BLE Sense paired with an MPU6050 sensor to read tilt angles in real time. Once tilt passes a set threshold, the Nano sends a direction command over Bluetooth through a HC-05 Bluetooth module. A second HC-05, connected to the robot's Arduino Uno, picks up that command and converts it into signals for the L298N motor driver, which drives the robot and keeps updating as the hand position changes.

Most of the difficulty was debugging across three layers: sensor, code, and wiring. The MPU6050 wouldn't connect at first, because the library I was using ran a strict device-ID check that clone sensor boards fail. Switching to a different library fixed it, but only after I confirmed with an I2C scanner that the sensor was actually responding. Tilt direction was also backwards for a while (for example, tilting forward triggered a left turn) which turned out to be the sensor's axes sitting rotated relative to how it was worn on the hand; remapping the axes in code solved it. A similar problem happened with the motors, where the physical wiring didn't line up with the pin numbers in the code, causing only one side of the robot’s wheels to turn. Checking my code and referring to the physical wiring of my robot helped me debug this problem. 

With the base project finished, the next phase is implementing modifications to the robot!


# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/RlMRTpZOGCI?si=E5vtUMKn8YLxuwlT" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

My first milestone consisted of assembling and wiring the robot chassis. I bolted four DC motors to the bottom frame and wired them into an L298N motor driver, which was in turn connected to an Arduino Uno responsible for direction and speed control. Power was supplied by two separate 9V batteries: one connected directly to the motor driver, the other to the Uno's barrel jack. Components were mounted with screws or tape depending on the part. To test if the car robot was working as intended, I programmed the robot to drive autonomously in a square pattern, which it did successfully. 

Sourcing parts was the biggest challenge during this milestone. The chassis kit arrived without screws, the Nano's micro-USB port broke off, and the battery clip was fitted with an incompatible connector. Since I am an international student, it takes a long time to reorder the parts I need. So I had to find local versions or alternatives. I used a different chassis kit, sourced a new Nano, and used electrical tape to connect the battery clip to the motor driver. Another challenge I faced was wiring the hand module correctly. At first, I used a voltage divider which I quickly realised I didn’t need when I found out the Nano also operates at 3.3V. 

Next, I will be working on the programming of the robot. I have already connected the HC-05 bluetooth modules and gotten the accelerometer connected: the car’s bluetooth module is receiving gesture/tilt data from the hand’s bluetooth module. I will work on actually making the robot move based on the gesture data it receives from the hand module. 

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code

Arudino Nano 33 BLE Sense Code

```c++
#include <Wire.h>
#include <MPU6050_light.h> //This was the library I used after other ones didn't work. Best for if your MPU6050 is a clone!

MPU6050 mpu(Wire);

const float TILT_THRESHOLD = 35.0; //Controls the angle that the hand component must be turned for the signal to send. Measured in degrees. 
char lastCommand = 'S';

void setup() {
  Serial.begin(9600); //I set the baud rate to 9600 but you can change this depending on your parts.
  Serial1.begin(38400);
  while (!Serial) delay(10);

  Wire.begin();
  mpu.begin();
  Serial.println("Calibrating, keep sensor still...");
  mpu.calcOffsets();
  Serial.println("Master ready");
}

void loop() {
  mpu.update();

  float fwdBack = mpu.getAngleX();
  float leftRight = mpu.getAngleY();

  char command = 'S';

  if (fwdBack > TILT_THRESHOLD) {
    command = 'F';
  } else if (fwdBack < -TILT_THRESHOLD) {
    command = 'B';
  } else if (leftRight > TILT_THRESHOLD) {
    command = 'L';
  } else if (leftRight < -TILT_THRESHOLD) {
    command = 'R';
  }

  if (command != lastCommand) {
    Serial1.println(command);
    Serial.print("Sent: ");
    Serial.println(command);
    lastCommand = command;
  }

  delay(100);
}
```

Arduino UNO Code

```c++
#include <SoftwareSerial.h>

const int HC05_RX = 3;
const int HC05_TX = 2;
SoftwareSerial HC05(HC05_RX, HC05_TX);

// Left motor pair
const int ENA = 9;
const int IN1 = 12;
const int IN2 = 11;

// Right motor pair
const int ENB = 6;
const int IN3 = 7;
const int IN4 = 8;

const int SPEED = 200; //Controls the speed. Max is 255

void setup() {
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  stopMotors();

  Serial.begin(9600);
  HC05.begin(38400);
  Serial.println("Slave ready");
}

void loop() {
  if (HC05.available()) {
    String message = HC05.readStringUntil('\n');
    message.trim();
    Serial.print("Received: ");
    Serial.println(message);

    if (message == "F") moveForward();
    else if (message == "B") moveBackward();
    else if (message == "L") turnLeft();
    else if (message == "R") turnRight();
    else stopMotors(); // "S" or anything unrecognised
  }
}

void moveForward() {
  digitalWrite(IN1, HIGH); digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH); digitalWrite(IN4, LOW);
  analogWrite(ENA, SPEED); analogWrite(ENB, SPEED);
}

void moveBackward() {
  digitalWrite(IN1, LOW); digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW); digitalWrite(IN4, HIGH);
  analogWrite(ENA, SPEED); analogWrite(ENB, SPEED);
}

void turnLeft() {
  digitalWrite(IN1, HIGH); digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW); digitalWrite(IN4, HIGH);
  analogWrite(ENA, SPEED); analogWrite(ENB, SPEED);
}

void turnRight() {
  digitalWrite(IN1, LOW); digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH); digitalWrite(IN4, LOW);
  analogWrite(ENA, SPEED); analogWrite(ENB, SPEED);
}

void stopMotors() {
  analogWrite(ENA, 0);
  analogWrite(ENB, 0);
}
```

# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Car Chassis Kit | Base of the robot | $39.99 | <a href="https://www.amazon.com/dp/B0DJ7BT1V5?ref=cm_sw_r_cso_cp_apin_dp_RCSYWRX92M0H5DJ6HNQA"> Link </a> |
| Screwdriver kit | Includes multiple types of screwdriver ends for different screws | $5.94 | <a href="https://www.amazon.com/Small-Screwdriver-Set-Mini-Magnetic/dp/B08RYXKJW9/"> Link </a> |
| Elegoo Uno R3 (Arduino Uno Clone) | Microcontroller board for the main robot | $14.98 | <a href="https://www.amazon.com/ELEGOO-Board-ATmega328P-ATMEGA16U2-Compliant/dp/B01EWOE0UU"> Link </a> |
| Electronics Kit | General kit for robotics | $14.00 | <a href="https://www.amazon.com/Smraza-Electronics-Potentiometer-tie-Points-Breadboard/dp/B0B62RL725"> Link </a> |
| Breadboard Kit | To build circuits without soldering | $8.79 | <a href="https://www.amazon.com/Breadboards-Solderless-Breadboard-Distribution-Connecting/dp/B07DL13RZH"> Link </a> |
| Arduino Nano 33 BLE Sense | Smaller microcontroller board for the gesture module | $39.70 | <a href="https://www.amazon.com/Arduino-Nano-Sense-headers-ABX00070/dp/B0BQHZ88WD"> Link </a> |
| Micro USB Cable | Allows connection between components & laptop | $5.00 | <a href="https://www.amazon.com/Charging-Transfer-Android-Trustable-MYFON/dp/B098DW7485"> Link </a> |
| Accelerometer | Measures the acceleration of the robot | $9.00 | <a href="https://www.amazon.com/dp/B0D2TJVMNY"> Link </a> |
| HC-05 Bluetooth Serial Pass-through Module | Allows a bluetooth connection between the car robot & hand module | $9.00 | <a href="https://www.amazon.com/DSD-TECH-HC-05-Pass-through-Communication/dp/B01G9KSAF6"> Link </a> |
| Breadboard Power Supply | Allows power to be supplied to the breadboards | $8.00 | <a href="https://www.amazon.com/ALAMSCN-Solderless-Breadboard-Battery-Arduino/dp/B08JYPMCZY"> Link </a> |
| 9V Batteries | Power supply | $8.69 | <a href="https://www.amazon.com/Amazon-Basics-Performance-All-Purpose-Batteries/dp/B00MH4QM1S/"> Link </a> |
| Velcro Tape | Wraps around a hand for the hand module | $8.00 | <a href="https://www.amazon.com/Art3d-Sticky-Double-Sided-Command-Adhesive/dp/B0B58FGF8H"> Link </a> |
| Digital Multimeter | Measures resistance, voltage, and current | $9.99 | <a href="https://www.amazon.com/dp/B0CXM242J1"> Link </a> |


# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
