# deej

deejxchanger is an **open-source hardware volume mixer and output switcher** for Windows. It is based on [deej](https://github.com/omriharel/deej) and lets you use real-life sliders (like a DJ!) to **seamlessly control the volumes of different apps** 
(such as your music player, the game you're playing and your voice chat session) without having to stop what you're doing.

**Join the [deej Discord server](https://discord.gg/nf88NJu) if you need help or have any questions!**

[![Discord](https://img.shields.io/discord/702940502038937667?logo=discord)](https://discord.gg/nf88NJu)

> **_New:_** [work-in-progress deej FAQ](./docs/faq/faq.md)!

deej consists of a [lightweight desktop client](#features) written in Go, and an Arduino-based hardware setup that's simple and cheap to build. [**Check out some versions built by members of our community!**](./community.md)

**[Download the latest release](https://github.com/omriharel/deej/releases/latest) | [Video demonstration](https://youtu.be/VoByJ4USMr8) | [Build video by Tech Always](https://youtu.be/x2yXbFiiAeI)**

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

deej is written in Go and [distributed](https://github.com/omriharel/deej/releases/latest) as a portable (no installer needed) executable.

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
- The middle pin of the two way switch is connected to 3.3V, the left and right ones are connected to Digital Pin 2 and 3 respectively (see second schematic)
- The board connects via a USB cable to the PC

#### Schematic

![Hardware schematic](assets/schematic.png)

<img width="482" height="451" alt="image" src="https://github.com/user-attachments/assets/7c05d57f-1c71-41e2-93da-0703767c968b" />


### Software

- The code running on the Arduino board is a [C program](./arduino/deej-5-sliders-vanilla/deej-5-sliders-vanilla.ino) constantly writing current slider values over its serial interface
- The PC runs a lightweight [Go client](./pkg/deej/cmd/main.go) in the background. This client reads the serial stream and adjusts app volumes according to the given configuration file

## Slider mapping (configuration)

deej uses a simple YAML-formatted configuration file named [`config.yaml`](./config.yaml), placed alongside the deej executable.

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

  

## Build your own!

Building deej is very simple. You only need a few relatively cheap parts - it's an excellent starter project (and my first Arduino project, personally). Remember that if you need any help or have a question that's not answered here, you can always [join the deej Discord server](https://discord.gg/nf88NJu).

Build deej for yourself, or as an awesome gift for your gaming buddies!

### FAQ

I've started a highly focused effort of writing a proper FAQ page for deej, covering many basic and advanced topics.

It is still _very much a work-in-progress_, but I'm happy to [share it in its current state](./docs/faq/faq.md) in hopes that it at least covers some questions you might have.

FAQ feedback in our [community Discord](https://discord.gg/nf88NJu) is strongly encouraged :)

### Build video

In case you prefer watching to reading, Charles from the [**Tech Always**](https://www.youtube.com/c/TechAlways) YouTube channel has made [**a fantastic video**](https://youtu.be/x2yXbFiiAeI) that covers the basics of building deej for yourself, including parts, costs, assembly and software. I highly recommend checking it out!

### Bill of Materials

- An Arduino Nano, Pro Micro or Uno board
  - I officially recommend using a Nano or a Pro Micro for their smaller form-factor, friendlier USB connectors and more analog pins. Plus they're cheaper
  - You can also use any other development board that has a Serial over USB interface
- A few slider potentiometers, up to your number of free analog pins (the cheaper ones cost around 1-2 USD each, and come with a standard 10K Ohm variable resistor. These _should_ work just fine for this project)
  - **Important:** make sure to get **linear** sliders, not logarithmic ones! Check the product description
  - You can also use circular knobs if you like
- A 3-Way Switch
- Some wires
- Any kind of box to hold everything together. **You don't need a 3D printer for this project!** It works fantastically with just a piece of cardboard or a shoebox. That being said, if you do have one, read on...

### Thingiverse collection

With many different 3D-printed designs being added to our [community showcase](./community.md), it felt right to gather all of them in a Thingiverse collection for you to browse. If you have access to a 3D printer, feel free to use one of the designs in your build.

**[Visit our community-created design collection on Thingiverse!](https://thingiverse.com/omriharel/collections/deej)**

> You can also [submit your own](https://discord.gg/nf88NJu) design to be added to the collection. Regardless, if you do upload your design to Thingiverse, _please add a `deej` tag to it so that others can find it more easily_.


### Build procedure

- Connect everything according to the [schematic](#schematic)
- Test with a multimeter to be sure your sliders are hooked up correctly
- Flash the Arduino chip with the sketch in [`arduino\deej-5-sliders-vanilla`](./arduino/deej-5-sliders-vanilla/deej-5-sliders-vanilla.ino)
  - _Important:_ If you have more or less than 5 sliders, you must edit the sketch to match what you have
- After flashing, check the serial monitor. You should see a constant stream of values separated by a pipe (`|`) character, e.g. `0|240|1023|0|483`
  - When you move a slider, its corresponding value should move between 0 and 1023
- Congratulations, you're now ready to run the deej executable!

## How to run

### Requirements

#### Windows

- Windows. That's it

#### Linux
- Linux is NOT supported by this modified version of Deej as it uses Windows-specific libraries for audio switching. For the normal, linux-compatible version, please check out the [original deej version](https://github.com/omriharel/deej) (Not supporting audio switching)


### Download and installation

- Head over to the [releases page](https://github.com/omriharel/deej/releases) and download the [latest version](https://github.com/omriharel/deej/releases/latest)'s executable and configuration file (`deejxchanger.exe` and `config.yaml`)
- Place them in the same directory anywhere on your machine
- (Optional, on Windows) Create a shortcut to `deejxchanger.exe` and copy it to `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup` to have deej run on boot

### Building from source

If you'd rather not download a compiled executable, or want to extend deej or modify it to your needs, feel free to clone the repository and build it yourself. All you need is a Go 1.14 (or above) environment on your machine. If you go this route, make sure to check out the [developer scripts](./pkg/deej/scripts).

Like other Go packages, you can also use the `go get` tool: `go get -u github.com/omriharel/deej`. Please note that the package code now resides in the `pkg/deej` directory, and needs to be imported from there if used inside another project.

If you need any help with this, please [join our Discord server](https://discord.gg/nf88NJu).

## Community

[![Discord](https://img.shields.io/discord/702940502038937667?logo=discord)](https://discord.gg/nf88NJu)

deej is a relatively new project, but a vibrant and awesome community is rapidly growing around it. Come hang out with us in the [deej Discord server](https://discord.gg/nf88NJu), or check out a whole bunch of cool and creative builds made by our members in the [community showcase](./community.md).

The server is also a great place to ask questions, suggest features or report bugs (but of course, feel free to use GitHub if you prefer).

### Donations

If you love deej and want to show your support for this project, you can do so using the link below. Please don't feel obligated to donate - building the project and telling your friends about it goes a very long way! Thank you very much.

[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/omriharel)

### Contributing

Please see [`docs/CONTRIBUTING.md`](./docs/CONTRIBUTING.md).

## License

deej is released under the [MIT license](./LICENSE).
