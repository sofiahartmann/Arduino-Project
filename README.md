# Arduino-Project
## Video Walkthrough
https://youtu.be/TX0FOCcWjjQ 
## Resources
https://roboticsbackend.com/arduino-led-complete-tutorial/
https://chatgpt.com
https://chat.deepseek.com/
https://grok.com/
## Arduino Code
const int greenLedPin = 10;  // Pin for the green LED
const int redLedPin = 11;    // Pin for the red LED

void setup() {
  // Initialize serial communication
  Serial.begin(9600);

  // Set LED pins as output
  pinMode(greenLedPin, OUTPUT);
  pinMode(redLedPin, OUTPUT);

  // Ensure LEDs are off initially
  digitalWrite(greenLedPin, LOW);
  digitalWrite(redLedPin, LOW);
}

void loop() {
  // Check if data is available on the serial port
  if (Serial.available() > 0) {
    String command = Serial.readStringUntil('\n');  // Read the command
    command.trim();  // Remove any extra whitespace

    // Process the command
    if (command == "GREEN_ON") {
      digitalWrite(greenLedPin, HIGH);  // Turn on the green LED
      Serial.println("Green LED ON");
    } else if (command == "GREEN_OFF") {
      digitalWrite(greenLedPin, LOW);  // Turn off the green LED
      Serial.println("Green LED OFF");
    } else if (command == "RED_ON") {
      digitalWrite(redLedPin, HIGH);  // Turn on the red LED
      Serial.println("Red LED ON");
    } else if (command == "RED_OFF") {
      digitalWrite(redLedPin, LOW);  // Turn off the red LED
      Serial.println("Red LED OFF");
    } else {
      Serial.println("Unknown command");
    }
  }
}
## Visual Studio Code
import serial
import sounddevice as sd
import speech_recognition as sr

# Initialize the serial connection to the Arduino
try:
    arduino = serial.Serial('/dev/cu.usbmodem101', 9600, timeout=1)  # Replace with your Arduino's port
    print("Serial port opened successfully.")
except serial.SerialException as e:
    print(f"Error opening serial port: {e}")
    exit(1)

# Initialize the recognizer
recognizer = sr.Recognizer()

def listen_for_command():
    """
    Listen for audio input and transcribe it using Google Speech Recognition.
    """
    print("Listening for command...")
    duration = 5  # Record for 5 seconds
    sample_rate = 16000  # 16 kHz sample rate

    # Record audio using sounddevice
    print("Recording...")
    audio = sd.rec(int(duration * sample_rate), samplerate=sample_rate, channels=1, dtype='int16')
    sd.wait()  # Wait for recording to finish
    print("Recording finished.")

    # Convert the audio to the format required by speech_recognition
    audio_data = sr.AudioData(audio.tobytes(), sample_rate, 2)

    try:
        # Use Google Speech Recognition to transcribe the audio
        result = recognizer.recognize_google(audio_data).lower()
        print(f"You said: {result}")
        return result
    except sr.UnknownValueError:
        print("Sorry, I did not understand that.")
        return None
    except sr.RequestError:
        print("Could not request results; check your network connection.")
        return None

def send_command(command):
    """
    Send the command to the Arduino over serial.
    """
    try:
        arduino.write(command.encode())
        print(f"Sent: {command}")
    except serial.SerialException as e:
        print(f"Error sending command: {e}")

# Initial state - start with green light on
send_command("GREEN_ON\n")

def process_command(result):
    """
    Process the voice command and send the appropriate serial command.
    """
    if result:
        # Check for conditions to turn green off and red on
        if "shattered glass" in result or "door opening" in result:
            send_command("GREEN_OFF\n")
            send_command("RED_ON\n")
        # Check for reset condition to turn red off and green back on
        elif "reset" in result:
            send_command("RED_OFF\n")
            send_command("GREEN_ON\n")
        else:
            print("Unknown command")

if __name__ == "__main__":
    try:
        while True:
            # Listen for a command
            result = listen_for_command()

            # Process the command
            process_command(result)

            # Exit the loop if the user says "quit"
            if result and "quit" in result:
                print("Exiting...")
                break
    except KeyboardInterrupt:
        print("Program interrupted by user.")
    finally:
        # Close the serial connection
        arduino.close()
        print("Serial port closed.")
        
