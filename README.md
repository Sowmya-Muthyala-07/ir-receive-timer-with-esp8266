#include <IRremoteESP8266.h>
#include <IRrecv.h>
#include <IRutils.h>


const uint16_t recv_pin = 18;  // Adjust the GPIO pin number for your ESP32 setup
IRrecv irrecv(recv_pin);
decode_results results;


// Timer variables
hw_timer_t * timer = NULL;
unsigned long timerInterval = 1000; // Timer interval in milliseconds
unsigned long lastMillis = 0;


void IRAM_ATTR onTimer() {
  // Code to execute on timer interrupt
  Serial.println("Timer triggered");
}


void setup() {
  Serial.begin(115200);
  irrecv.enableIRIn();  // Start the IR receiver


  // Set up timer to trigger every 1000ms
  timer = timerBegin(0, 80, true);  // Timer 0, divide by 80 for 1ms interval
  timerAttachInterrupt(timer, &onTimer, true);  // Trigger onTimer() every time
  timerAlarmWrite(timer, timerInterval * 1000, true);  // Set timer interval (in microseconds)
  timerAlarmEnable(timer);  // Enable timer


  Serial.println("Ready to receive IR signals...");
}


void loop() {
  if (irrecv.decode(&results)) {
    // Print the HEX code and protocol of the received IR signal
    Serial.print("IR Code Received: ");
    Serial.println(resultToHexidecimal(&results));  // Print received IR code in HEX format
    Serial.print("Protocol: ");
    Serial.println(typeToString(results.decode_type));  // Print protocol type as a string


    // Resume the receiver for the next IR signal
    irrecv.resume();
  }


  // Check if the timer has triggered
  unsigned long currentMillis = millis();
  if (currentMillis - lastMillis >= timerInterval) {
    lastMillis = currentMillis;
    // Place code here that you want to run periodically
    Serial.println("Timer tick (non-interrupt)...");
  }
}
