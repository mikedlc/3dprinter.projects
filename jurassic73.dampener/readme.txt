https://www.printables.com/model/881578-arduino-servo-controlled-enclosure-exhaust-port-da/files

Description

This is a servo controlled duct I use for my printer exhaust (120mm fan to 4" flex duct) which goes in to an attic space. The hot/cold air from the attic gets pulled into my enclsoure and my home from time to time with air pressure differences. This dampener helps only allow flow through this duct while my printer and exhaust fan is running.
 

This was necessitated by the fact that regular gravity dampeners would not move(even the lightest) with the low flow of my fan on it's lowest setting. This servo control gurantees the flap moves as required to cut off flow when not printing.

This is controlled using an Arduino Pro Micro board via a GPIO pin from a Raspberry Pi running OctoPrint and the PSU Control plug-in.

The Arduino must be powered from a USB port on the Pi so you have a common ground reference point for the GPIO pin to be read.


https://www.youtube.com/watch?time_continue=1&v=QUaQtuo-cK0&embeds_referring_euri=https%3A%2F%2Fwww.printables.com%2F&source_ve_path=Mjg2NjY

I modeled this using a sub micro servo - Hitec HS81.
 

Wire servo to pins RAW(red), GND(black), A1(yellow) on the Arduino.


Used (7) 6mm M3 screws for the servo mounting, flap mounting and cover.

Fast Close Code:

#include <Servo.h>

// Define the servo motor object
Servo servo;

// Define the pin for input
const int gpio_input = 14;

// Define the pin for servo control - A1 marked on the board = pin 19
const int servo_pin = 19;

// Define the angles for original position and rotated position
const int open_flap = 40;
const int close_flap = 132;

// Variable to store the current state of the gpio_input
int gpio_input_state = 0;
int last_gpio_input_state = 0;

// Variable to store the current servo position
int currentAngle = close_flap;

void setup() {
 // Attach the servo to servo_pin
 servo.attach(servo_pin);
 // Set initial position of the flap to close
 servo.write(close_flap);
 // wait while servo moves (ms) else this will process "instantly" and detatch before the servo can complete it's move
 delay(500);
 // we don't need to hold the servo in this position in this use so detatch from servo pin to avoid servo chatter/twitches
 servo.detach();
 
 // Set gpio_input as input - this is what we use from the Raspberry Pi to control the flap from Octoprint(running on said Pi) Plug-in PSU Control. Our Arduino is powered by the Pi via a USB port on the Pi which also gives us a common ground so we can read this pin. We need the common ground for that point of reference.
 pinMode(gpio_input, INPUT);
}

void loop() {
 // Read the state of the gpio_input
 gpio_input_state = digitalRead(gpio_input);
 
 // Check for changes in gpio_input_ state - if no changes, do nothing - if a change, move the flap
 if (gpio_input_state != last_gpio_input_state) {
   if (gpio_input_state == LOW) {
     // Close flap
     currentAngle = close_flap;
     // Attach the servo to servo_pin
     servo.attach(servo_pin);
     // move servo to close the flap
     servo.write(close_flap);
     // wait while servo moves (ms)
     delay(500);
     // detatch from the servo pin
     servo.detach();

   } else {
     // Open flap
     currentAngle = open_flap;
     // Attach the servo to servo_pin
     servo.attach(servo_pin);
     // move servo to open the flap
     servo.write(open_flap);
     // wait while servo moves (ms)
     delay(500);
     // detatch from the servo pin
     servo.detach();
   }

   // Update last gpio_input_ state after processing the change so next loop processes accordingly with updated flap position
   last_gpio_input_state = gpio_input_state;
 } else {
   // If gpio_input_ state has not changed, wait for another half second
   delay(500);
 }
}

Slow Close Code: (trial and error with movement_increment and movement_delay can yeild a quieter motion.

#include <Servo.h>

// Define the servo motor object
Servo servo;

// Define the pin for input
const int gpio_input = 14;

// Define the pin for servo control
const int servo_pin = 19;

// Define the angles for original position and rotated position
const int open_flap = 40;
const int close_flap = 135;

// Variable to store the current state of the gpio_input
int gpio_input_state = 0;
int last_gpio_input_state = 0;

// Variable to store the current servo position
int currentAngle = close_flap;

// Movement increment
int movement_increment = 1;

// Movement delay
int movement_delay = 30;

void setup()

{
 // Attach the servo to servo_pin
 servo.attach(servo_pin);
 // Set initial position of the flap to close upon power up
 servo.write(close_flap);
 // wait while servo moves (ms) else this line will process "instantly" and detach before the servo can even start it's move
 delay(500);
 // we don't need to hold the servo in this position in this use so detach from servo pin to avoid servo chatter/twitches
 servo.detach();

 // Set gpio_input as input
 pinMode(gpio_input, INPUT);
}

void loop()
{
 // Read the state of the gpio_input
 gpio_input_state = digitalRead(gpio_input);

 // Check for changes in gpio_input_ state - if no changes, do nothing - if a change, move the flap
 if (gpio_input_state != last_gpio_input_state)
 {
   if (gpio_input_state == LOW)
   {
     // Attach the servo to servo_pin
     servo.attach(servo_pin);
     // Close flap gradually by increasing angle from open_flap to close_flap
     for (int angle = currentAngle; angle < close_flap; angle += movement_increment)
     {
       // move servo to the desired position
       servo.write(angle);
       // wait for movement delay
       delay(movement_delay);
     }
     currentAngle = close_flap; // Update current angle
   }
   else
   {
     // Attach the servo to servo_pin
     servo.attach(servo_pin);
     // Open flap gradually by decreasing angle from close_flap to open_flap
     for (int angle = currentAngle; angle > open_flap; angle -= movement_increment)
     {
       // move servo to the desired position
       servo.write(angle);
       // wait for movement delay
       delay(movement_delay);
     }
     currentAngle = open_flap; // Update current angle
   }
   delay(200);
 }

 // Detach from the servo pin after the movement is completed
 servo.detach();

 // Update last gpio_input_ state after processing the change so next loop processes accordingly with updated flap position
 last_gpio_input_state = gpio_input_state;
}ß