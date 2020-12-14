# Smart-Dustbin - integrated application on Arduino IDE using GSM module
#include <SoftwareSerial.h>
#include <VarSpeedServo2.h>
#include <SharpDistSensor.h>

VarSpeedServo lid_servo;
VarSpeedServo bucket_servo;// create servo object to control a servo
// a maximum of eight servo objects can be created
const byte sensorPin1 = A0;
const byte sensorPin2 = A1;
const int lid = 8;
const int bucket = 11; // the digital pin used for the servo
const int inductive = 2;
const int capacitive = 7;
SoftwareSerial mySerial(12, 13);
const int trigger[] = {9, 6, 4};
const int echo[] = {10, 5, 3};
int distance_ir = 0;
int prev_distance = 0;
// defines variables
long duration;

long distance[4];
int detectiondelay;
boolean flag[] = {false, false, false};
int detect = 0;
void setup()
{
  pinMode(sensorPin1, INPUT);
    pinMode(sensorPin2, OUTPUT);
  lid_servo.attach(lid, 500, 2400);
  bucket_servo.attach(bucket, 500, 2400); // attaches the servo on pin 9 to the servo object
  lid_servo.write(0, 30, true);
  bucket_servo.write(0, 30, true);  // set the intial position of the servo, as fast as possible, wait until done
  mySerial.begin(9600);   // Setting the baud rate of GSM Module
  Serial.begin(9600);    // Setting the baud rate of Serial Monitor (Arduino)
  delay(100);


  pinMode(9, OUTPUT); // Sets the trigPin as an Output (9 is trigpin)
  pinMode(6, OUTPUT);
  pinMode(4, OUTPUT);


  pinMode(10, INPUT); // Sets the echoPin as an Input
  pinMode(5, INPUT);
  pinMode(3, INPUT);
}

void ultrasonic(int x, int y, int n)
{
  digitalWrite(x, LOW);
  delayMicroseconds(2);
  // Sets the trigPin on HIGH state for 10 micro seconds
  digitalWrite(x, HIGH);
  delayMicroseconds(10);
  digitalWrite(x, LOW);
  // Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(y, HIGH);


  distance[n] = duration * 0.0343 / 2; //as duration would be obtained in microseconds and speed of sound in dry air at 20 degree celsius is 343m/s

  Serial.print("Distance from sensor");
  Serial.print(n);
  Serial.print(" = ");
  Serial.print(distance[n]);
  Serial.print(" cm");
  Serial.println();

  if ((distance[n] < 7) && (!flag[n]))
  {
    SendMessage(n);
    flag[n] = true;
  }
  if (distance[n] > 30)
  {
    flag[n] = false;
  }

  delay(detectiondelay);    //to pause answer and calculation for 0.1 sec , 1000 means 1 second
}




void loop() {
  for (int i = 0; i < 3; i++)
  {
    ultrasonic(trigger[i], echo[i], i);
  }
  motor();
}

void SendMessage(int j)
{
  mySerial.println("AT+CMGF=1");    //Sets the GSM Module in Text Mode
  delay(1000);  // Delay of 1000 milli seconds or 1 second
  mySerial.println("AT+CMGS=\"+971564950944\"\r"); // Replace x with mobile number
  delay(1000);
  mySerial.println("Chamber");// The SMS text you want to send
  delay(100);
  mySerial.println(j);
  delay(100);
  mySerial.println(" filled. Please clear.");
  delay(100);
  mySerial.println((char)26);// ASCII code of CTRL+Z
  delay(1000);
}

void motor()
{
  digitalWrite(sensorPin2, LOW);
  delayMicroseconds(2);
  // Sets the trigPin on HIGH state for 10 micro seconds
  digitalWrite(sensorPin2, HIGH);
  delayMicroseconds(10);
  digitalWrite(sensorPin2, LOW);
  // Reads the echoPin, returns the sound wave travel time in microseconds
  int duration = pulseIn(sensorPin1, HIGH);


  float distance_ir = duration * 0.0343 / 2; //as duration would be obtained in microseconds and speed of sound in dry air at 20 degree celsius is 343m/s

  Serial.print("Distance from sensor_ir");
  //Serial.print(n);
  Serial.print(" = ");
  Serial.print(distance_ir);
  Serial.print(" cm");
  Serial.println();
  if ((distance_ir <= 25) && (prev_distance > 26 )) //detected
  {
    //delay(1000);
    double time_taken = millis();
    while ((millis() - time_taken) < 2500)
    {
      if (digitalRead(inductive))
      {
        detect = 1;
        break;
      }
      else if (digitalRead(capacitive) == 0)
      {
        detect = 2;
        break;
      }
    }
    if (detect == 0)
      detect = 3;
    switch (detect)
    {
      case 1:
        Serial.println("inside1");
        delay(1500);
        lid_servo.write(255, 30, true);
        delay(500);
        bucket_servo.write(125, 30, true);
        delay(500);
        bucket_servo.write(0, 30, true);
        detect = 0;
        break;

      case 2:
        delay(1500);
        Serial.println("inside2");
        lid_servo.write(128, 30, true);
        delay(500);
        bucket_servo.write(125, 30, true);
        delay(500);
        bucket_servo.write(0, 30, true);
        detect = 0;
        break;

      case 3:
        Serial.println("inside3");
        lid_servo.write(0, 30, true);
        Serial.println("inside4");
        delay(500);
        bucket_servo.write(125, 30, true);
        delay(500);
        bucket_servo.write(0, 30, true);
        detect = 0;
        break;
    }
    lid_servo.write(0, 30, true);
  }
  else
  {
    lid_servo.write(0, 30, true);
    bucket_servo.write(0, 30, true);
  }
  prev_distance = distance_ir;
}
