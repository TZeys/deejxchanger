# deejxchanger

deejxchanger is an **open-source hardware volume mixer and output switcher** for Windows. It is lightly based on [deej](https://github.com/omriharel/deej) and lets you use real-life sliders (like a DJ!) to **seamlessly control the volumes of different apps** 
(such as your music player, the game you're playing and your voice chat session) without having to stop what you're doing.


deejxchanger consists of a [lightweight desktop client](#features) written in Go, and an Arduino-based hardware setup that's simple and cheap to build.

**[Download the latest release](https://github.com/TZeys/deejxchanger/releases/tag/V0.01) | [Deej based Build-Tutorial](https://youtu.be/x2yXbFiiAeI)**

<img width="1133" height="658" alt="image" src="https://github.com/user-attachments/assets/7bdbb748-7808-4be8-85c5-94f46c1e4c20" />


> _**Psst!** [No 3D printer? No problem!](./assets/build-shoebox.jpg)_ You can build deej on some cardboard, a shoebox or even a breadboard :)

## Table of contents

- [Features](#features)
- [How it works](#how-it-works)
  - [Hardware](#hardware)
    - [Schematic](#schematic)
  - [Software](#software)
- [Slider mapping (configuration)](#slider-mapping-configuration)
- [Build your own!](#build-your-own)
  - [FAQ](#faq)
  - [Build video](#build-video)
  - [Bill of Materials](#bill-of-materials)
  - [Thingiverse collection](#thingiverse-collection)
  - [Build procedure](#build-procedure)
- [How to run](#how-to-run)
  - [Requirements](#requirements)
  - [Download and installation](#download-and-installation)
  - [Building from source](#building-from-source)
- [Community](#community)
- [License](#license)

## Features

deejxchanger is written in Go and [distributed](https://github.com/TZeys/deejxchanger/releases/tag/V0.01) as a portable (no installer needed) executable.

- Bind apps to different sliders
  - Bind multiple apps per slider (i.e. one slider for all your games)
  - Bind the master channel
  - Bind "system sounds" (on Windows)
  - Bind specific audio devices by name (on Windows)
  - Bind currently active app (on Windows)
  - Bind all other unassigned apps
- Control your microphone's input level
- Lightweight desktop client, consuming around 10MB of memory
- Runs from your system tray
- Helpful notifications to let you know if something isn't working

> **Looking for the older Python version?** It's no longer maintained, but you can always find it in the [`legacy-python` branch](https://github.com/omriharel/deej/tree/legacy-python).

## How it works

### Hardware

- The sliders are connected to 5 (or as many as you like) analog pins on an Arduino Nano/Uno board. They're powered from the board's 5V output (see schematic)
- The middle pin of the two way switch is connected to GND, the left and right ones are connected to Digital Pin 2 and 3 respectively (see second schematic)
- The board connects via a USB cable to the PC

#### Schematic

![Hardware schematic](assets/schematic.png)

<img width="699" height="639" alt="image" src="https://github.com/user-attachments/assets/f85efef9-2e24-4cd5-9384-5cca5352501e" />



### Software

- The code running on the Arduino board is a [C program](https://github.com/TZeys/deejxchanger/releases/tag/V0.01) constantly writing current slider values over its serial interface
- The PC runs a lightweight [Go client](./pkg/deej/cmd/main.go) in the background. This client reads the serial stream and adjusts app volumes according to the given configuration file

## Slider mapping (configuration)

deejxchanger uses a simple YAML-formatted configuration file named [`config.yaml`](./config.yaml), placed alongside the deej executable.

The config file determines which applications (and devices) are mapped to which sliders, and which parameters to use for the connection to the Arduino board, as well as other user preferences.

**This file auto-reloads when its contents are changed, so you can change application mappings on-the-fly without restarting deej.**

It looks like this:

```yaml
# process names are case-insensitive
# you can use 'master' to indicate the master channel, or a list of process names to create a group
# you can use 'mic' to control your mic input level (uses the default recording device)
# you can use 'deej.unmapped' to control all apps that aren't bound to any slider (this ignores master, system, mic and device-targeting sessions) (experimental)
# windows only - you can use 'deej.current' to control the currently active app (whether full-screen or not) (experimental)
# windows only - you can use a device's full name, i.e. "Speakers (Realtek High Definition Audio)", to bind it. this works for both output and input devices
# windows only - you can use 'system' to control the "system sounds" volume
# important: slider indexes start at 0, regardless of which analog pins you're using!
slider_mapping:
  0: discord.exe
  1: deej.unmapped
  2: spotify.exe
  3: brave.exe
  4: master

# set this to true if you want the controls inverted (i.e. top is 0%, bottom is 100%)
invert_sliders: false

# settings for connecting to the arduino board
com_port: COM4
baud_rate: 9600

# audio output device names for switching via serial commands
# "WORKS 1" will switch to output_one_device
# "WORKS 2" will switch to output_two_device
# important: make sure that the names below are correctly spelled as well as in the correct format! You will get an error message if these names are incorrect. 
# To find the exact name, please go to: Right Click Sound in your taskbar -> Windows Legacy -> Playback Devices
# You will see your playback devices with their names, and their driver device (Below the name i.e NVIDIA High Definition Audio or Realtek(R) Audio)
# format "NAME (DRIVER-NAME)"

output_one_device: "Headphones (3- AIR 192 4)"
output_two_device: "Speakers (NVIDIA High Definition Audio)"

# adjust the amount of signal noise reduction depending on your hardware quality
# supported values are "low" (excellent hardware), "default" (regular hardware) or "high" (bad, noisy hardware)
noise_reduction: default
```

- `master` is a special option to control the master volume of the system _(uses the default playback device)_
- `mic` is a special option to control your microphone's input level _(uses the default recording device)_
- `deej.unmapped` is a special option to control all apps that aren't bound to any slider ("everything else")
- On Windows, `deej.current` is a special option to control whichever app is currently in focus
- On Windows, you can specify a device's full name, i.e. `Speakers (Realtek High Definition Audio)`, to bind that device's level to a slider. This doesn't conflict with the default `master` and `mic` options, and works for both input and output devices.
  - Be sure to use the full device name, as seen in the menu that comes up when left-clicking the speaker icon in the tray menu
- `system` is a special option on Windows to control the "System sounds" volume in the Windows mixer
- All names are case-**in**sensitive, meaning both `chrome.exe` and `CHROME.exe` will work
- You can create groups of process names (using a list) to either:
    - control more than one app with a single slider
    - choose whichever process in the group that's currently running (i.e. to have one slider control any game you're playing)
- For audio output switching change the names of `output_one_device` and `output_two_device` respectively. the naming and format is very specific. please refer to the config.yaml explanation as well as the example below.
    - FORMAT: "NAME (DRIVER_NAME)" (with a space between NAME and (DRIVER_NAME)!)
  
<img width="751" height="67" alt="image" src="https://github.com/user-attachments/assets/24085e1e-1827-4ad9-a855-39f0bfeb9532" />

  

## License

deej is released under the [MIT license](./LICENSE).
