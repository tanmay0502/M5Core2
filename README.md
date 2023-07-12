# Navigation System for Visually Disabled

This repository contains the code for a navigation system designed to assist visually impaired individuals in navigating the IIT Bhilai campus. The system provides step-by-step directions from the EE lab to the director's office, with information about different paths near B117.

## Introduction

The Navigation System utilizes the M5Core2 device, which is a compact development kit with a touchscreen display. It combines visual and audio feedback to guide users through the campus. The directions and number of steps are storde in SD card attached to M5Core2.

## Getting Started

### Prerequisites

To run this code, you will need the following:

- M5Core2 device (https://m5stack.com/products/m5core2)
- M5Burner tool (https://m5stack.com/pages/m5burner)
- UIFlow Desktop IDE (https://flow.m5stack.com/)

### Installation

1. Install M5Burner by downloading it from the official M5Stack website.
2. Connect the M5Core2 device to your computer via USB.
3. Open M5Burner and select "M5Core2" in the Core2 and Tough section.
4. Install UIFlow by clicking the "Burn" button and following the on-screen instructions.
5. Once the M5Core2 restarts and enters USB mode, you are ready to download the code.

### Downloading the Code

1. Download the UIFlow Desktop IDE from the official M5Stack website.
2. Connect the M5Core2 device to your computer via USB.
3. Open the UIFlow Desktop IDE and select the corresponding port for the M5Core2.
4. Copy the code below and paste it into the UIFlow Desktop IDE.
5. Click the "Run" button in the UIFlow Desktop IDE to upload and run the code on the M5Core2.

# Code for the Navigation System goes here
```python
from m5stack import *
from m5stack_ui import *
from uiflow import *
import wifiCfg
from easyIO import *
import network

# Initialize M5Stack components and variables
Battery = str(map_value((power.getBatVoltage()), 3.7, 4.1, 0, 100))
wcLabel = M5Label("Starting up...", x=1, y=0, color=0x000, font=FONT_MONT_26, parent=None)
battery = M5Label("Battery: "+ Battery + "%", x=222, y=220, color=0x000, font=FONT_MONT_14, parent=None)
previousSound = "none"
previousSound2 = "none"

# Function to vibrate the device
def vibrate():
    power.setVibrationEnable(True)
    wait(0.5)
    power.setVibrationEnable(False)
    return

# Function to play a sound file
def playSound(mp3):
    global previousSound
    global previousSound2
    if previousSound != mp3:
        vibrate()
        speaker.playWAV('/sd/voice/' + mp3 + ".wav", volume=10)
        previousSound2 = previousSound
        previousSound = mp3
        return
    return

# Event handler for touch_button0 press
def touch_button0_pressed():
    playSound(previousSound)

# Display welcome message and play initial sound
speaker.playTone(220, 1/2, volume=6)
wait_ms(2000)
wcLabel.set_text("Welcome to IIT Bhilai")
playSound("welcome")
previousSound = "welcome"

# Main loop
while True:
    # Check if button A is pressed to power off the device
    if btnA.isPressed():
        power.powerOff()

    # Initialize the screen and display WiFi scanning message
    screen = M5Screen()
    screen.clean_screen()
    screen.set_screen_bg_color(0xffffff)
    power.setVibrationIntensity(50)
    wifi_label = M5Label("Scanning for WiFi...", x=1, y=220, color=0x000, font=FONT_MONT_10, parent=None)
    Battery2 = str(map_value((power.getBatVoltage()), 3.7, 4.1, 0, 100))
    battery2 = M5Label("Battery: "+ Battery2 + "%", x=222, y=220, color=0x000, font=FONT_MONT_14, parent=None)
    touch_button0 = M5Btn(text=' ', x=-40, y=-80, w=400, h=400, bg_c=0xFFFFFF, text_c=0x000000, font=FONT_MONT_14, parent=None)

    # Check if button B is pressed to play the previous sound
    if btnB.wasPressed():
        playSound(previousSound)

    # WiFi scanning and filtering
    wifi = network.WLAN(network.STA_IF)
    wifi.active(True)
    scan_results = wifi.scan()
    filtered_results = [result for result in scan_results if result[0].decode() == "IITBhilai"]
    sorted_results = sorted(filtered_results, key=lambda x: x[3], reverse=True)

    # Display up to 5 best WiFi networks found
    for i in range(min(5, len(sorted_results))):
        ssid = sorted_results[i][0].decode()
        rssi = sorted_results[i][3]
        # Placeholder values for MAC addresses (for security reasons)
        mac = "1:1:1:1:1:1"  # Placeholder for the first MAC address
        if i == 1:
            mac = "2:2:2:2:2:2"  # Placeholder for the second MAC address
        elif i == 2:
            mac = "3:3:3:3:3:3"  # Placeholder for the third MAC address
        # Add more elif statements for additional MAC addresses

        label = M5Label("yo", x=1, y=i*35, color=0x000, font=FONT_MONT_10, parent=None)
        label.set_text("SSID: " + ssid + "\nRSSI: " + str(rssi) + "\nMAC: " + mac)

    wifi_label.set_text("WiFi scan complete!")

    # Perform actions based on the detected WiFi network MAC address
    mac2 = ":".join("{:02x}".format(x) for x in sorted_results[0][1])
    # Placeholder values for MAC addresses (for security reasons)
    if mac2 == "1:1:1:1:1:1" and prev != mac2:  # Placeholder for the first MAC address
        playSound("director")
    elif mac2 == "2:2:2:2:2:2" and prev != mac2:  # Placeholder for the second MAC address
        if previousSound2 != "117":
            playSound("117")
            if previousSound2 == "eeTo117":
                playSound("117toDirector")
            elif previousSound2 == "welcome" or previousSound2 == "director":
                playSound("117toEE")
    elif mac2 == "3:3:3:3:3:3" and prev != mac2:  # Placeholder for the third MAC address
        playSound("ee")
        if previousSound2 == "welcome":
            playSound("eeTo117")

    prev = mac2
    wait_ms(3000)

