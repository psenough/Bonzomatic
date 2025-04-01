# Bonzomatic Network

## What's this?
This is a live-coding tool, where you can write a 2D fragment/pixel shader while it is running in the background.
It's a fork of [Bonzomatic](https://github.com/Gargaj/Bonzomatic) that add network communication (WebSocket Server).
The goal is to send a shader as text across a network and receive it somewhere else for display.
The shader is send when you ask for compilation (ctrl+r) and also at regular updates (interval can be set in config.json)
Bonzomatic can be launched as "sender" or as "grabber" if you want to send your shader or visualize a distant shader.

![Screenshot](https://i.imgur.com/8K8IztLl.jpg)

## Keys
- F2: toggle texture preview
- F3: in a grabber bonzo to toggle following the coder's caret or allowing free scrolling
- F4: reset shader time to zero
- F5 or Ctrl-R: recompile shader
- F11 or Ctrl/Cmd-f: hide shader overlay
- Alt-F4 or Shift+Escape: exbobolate your planet

## Requirements
On Windows, both DirectX 9 and 11 are supported

For the OpenGL version (for any platform), at least OpenGL 4.1 is required.

Sender and grabber must launch the same version (DX9/DX11/Ogl)
DX9/DX11 versions are not recommanded if you launch several grabbing Bonzomatic on the same PC

## Using Network

### Testing on Alkama Server

#### To start your own live session:
- double-click on Bonzomatic.exe
- In the dialogue, choose "Sender" as network setting
- put "ws://drone.alkama.com:9000" as server name (it should already be there)
- choose a room name (can be anything)
- choose your pseudo (please, dont use the pseudo from someone else :D )
- If someone is connected to that room, they should see your shader live

#### To watch a live session from someone on Alkama's server:
- double-clic on Bonzomatic.exe
- In the dialogue, choose "Grabber" as network setting
- put "ws://drone.alkama.com:9000" as server name (it should already be there)
- put the room name chosen by the other person
- put the pseudo of the other person
- If someone is sending to that room/pseudo you should see it

### Local Testing
- Launch a simple local server by double-clicking on BonzoServer.exe (it's in the same folder as Bonzomatic)
- double-clic on Bonzomatic.exe
- In the dialogue, choose "Grabber" as network setting
- put "ws://127.0.0.1:8000" as server name
- clic on "Run""
- Open a new Bonzomatic.exe, do the same settings but select "Sender" as network setting
- If you change the code in bonzomatic window from the sender, you should see the result updated on both bonzomatic.

### Hosting you own server
You can find a server (coded in Go by Alkama) ready to be deployed here:
https://github.com/alkama/BonzomaticServer

If you want to code your own server, connection is made using websockets.
A simple echo server that broadcast received messages to everyone connected is enough for now.

## Configuration
You can configure Bonzomatic by tweaking the `config.json` that is next to the binary executable.
You can also pass command line options:
- configfile=config2.json
- skipdialog
- serverURL=ws://127.0.0.1:8000/roomname/nickname
- networkMode=grabber

The config.json file can have the following contents: (all fields are optional)
``` javascript
{
  "window":{ // default window size / state, if there's a setup dialog, it will override it
    "width":1280,
    "height":720,
    "fullscreen":false,
    "resizable": true,
    "hideConsole": false,
  },
  "font":{ // all paths in the file are also relative to the binary, but again, can be absolute paths if that's more convenient
    "file": "ProFontWindows.ttf",
    "size":16,
  },
  "codernamefont": { // alternative font that is used to display the coder's name
    "file": "ProFontWindows.ttf",
    "size": 16
  },
  "rendering":{
    "fftSmoothFactor": 0.9, // 0.0 means there's no smoothing at all, 1.0 means the FFT is completely smoothed flat
    "fftAmplification": 1.0, // 1.0 means no change, larger values will result in brighter/stronger bands, smaller values in darker/weaker ones
    "fftCapturePlaybackDevices": true, // If true, will capture from playback devices (desktop sound) instead of capture devices (mic)
    "fftCaptureDeviceSearchString":  "", // If not empty, will search a device name containing this string
    "fftPeakNormalization": true, // If true, FFT will be normalized so as not to depend on the audio level
    "fftPeakMinValue": 0.01, // Minimum audio peak that will be scaled
    "fftPeakSmoothing": 0.995 // If audio peak drops, we smooth the transition to adjust it slowly
  },
  "textures":{ // the keys below will become the shader variable names
    "texChecker":"textures/checker.png",
    "texNoise":"textures/noise.png",
    "texTex1":"textures/tex1.jpg",
  },
  "gui":{
    "outputHeight": 200,
    "opacity": 192, // 255 means the editor occludes the effect completely, 0 means the editor is fully transparent
    "texturePreviewWidth": 64,
    "spacesForTabs": false,
    "tabSize": 8,
    "visibleWhitespace": true,
    "autoIndent": "smart", // can be "none", "preserve" or "smart"
    "scrollXFactor": 1.0, // if horizontal scrolling is too slow you can speed it up here (or change direction)
    "scrollYFactor": 1.0, // if vertical scrolling is too slow you can speed it up here (or change direction)
    "alwaysdisplaycodername": true // if network is enabled, decide if coder's name is always displayed even when hidding editor text
  },
  "midi":{ // the keys below will become the shader variable names, the values are the CC numbers
    "fMidiKnob": 16, // e.g. this would be CC#16, i.e. by default the leftmost knob on a nanoKONTROL 2
  },
  // this section is if you want to enable NDI streaming; otherwise just ignore it
  "ndi":{
    "enabled": true,
    "connectionString": "<ndi_product something=\"123\"/>", // metadata sent to the receiver; completely optional
    "identifier": "hello!", // device identifier; must be unique, also helps source discovery/identification in the receiver if there are multiple sources on the network
    "frameRate": 60.0, // frames per second
    "progressive": true, // progressive or interleaved?
  },
  // this section is there is you want to use networking
  "network": {
    "enabled": true,
    "serverURL": "ws://127.0.0.1:8000",
    "networkMode": "grabber", // can be sender or grabber
    "udpateInterval": 0.3, // duration between two shader sending
    "SyncTimeWithSender": true, // grabber will have same shader time than sender
    "SendMidiControls": true, // sender will broadcast it's midi control's
    "GrabMidiControls": true // grabber will apply recieved midi control's
  },
  // this section is if you want to customise colors to your liking
  "theme":{
    "text": "FFFFFF", // color format is "RRGGBB" or "AARRGGBB" in hexadecimal
    "comment": "00FF00",
    "number": "FF8000",
    "op": "FFCC00",
    "keyword": "FF6600",
    "type": "00FFFF",
    "builtin": "44FF88",
    "preprocessor": "C0C0C0",
    "selection": "C06699CC", // background color when selecting text
    "charBackground":   "C0000000", // if set, this value will be used (instead of gui opacity) behind characters
  },
  "postExitCmd":"copy_to_dropbox.bat" // this command gets ran when you quit Bonzomatic, and the shader filename gets passed to it as first parameter. Use this to take regular backups.
}
```
### Automatic shader backup
If you want the shader to be backed up once you quit Bonzomatic, you can use the above `postExitCmd` parameter in the config, and use a batch file like this:
```
@echo off
REM ### cf. https://stackoverflow.com/a/23476347
for /f "tokens=2 delims==" %%a in ('wmic OS Get localdatetime /value') do set "dt=%%a"
set "YY=%dt:~2,2%" & set "YYYY=%dt:~0,4%" & set "MM=%dt:~4,2%" & set "DD=%dt:~6,2%"
set "HH=%dt:~8,2%" & set "Min=%dt:~10,2%" & set "Sec=%dt:~12,2%"
copy %1 X:\MyShaderBackups\%YYYY%%MM%%DD%-%HH%%Min%%Sec%.glsl
```
This will copy the shader timestamped into a specified folder.

## Building
As you can see you're gonna need [CMAKE](https://cmake.org/) for this, but don't worry, a lot of it is automated at this point.

### Windows
Use at least [Visual C++ 2010](https://support.microsoft.com/ru-ru/help/2977003/the-latest-supported-visual-c-downloads). For the DX9/DX11 builds, obviously you'll be needing a [DirectX SDK](https://www.microsoft.com/en-us/download/details.aspx?id=6812), though a lot of it is already in the Windows 8.1 SDK as well.
You will need to install NDI SDK and install Visual Studio with components ATL and MFC
Then:
```mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ../
cmake --build .```

### OSX/macOS
```cmake``` should take care of everything:
```
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ../
cmake --build .
```
The Bonzomatic.app bundle, resulting from the compilation, should be found in `./build/Bonzomatic.app`. You can place it anywhere.
We do NOT recommend putting it in /Applications. Bonzomatic is looking for config.json files and resources living at the same level of the app.

### Linux
You'll need ```xorg-dev``` and ```libglu1-mesa-dev```; after that ```cmake``` should take care of the rest:
```
apt install xorg-dev libglu1-mesa-dev cmake
cd Bonzomatic
cmake .
make
make install
```

### OpenBSD
[Xenocara](https://xenocara.org) contains all required components.  Hack away with
```
cmake .
make
```
or use the port
```
cd /usr/ports/graphics/bonzomatic
make install
```

## Organizing a competition
If you want to organize a competition using Bonzomatic at your party, here's a handy-dandy guide on how to get started:
https://github.com/Gargaj/Bonzomatic/wiki/How-to-set-up-a-Live-Coding-compo

## Credits and acknowledgements
### Original / parent project authors
- "Bonzomatic" by Gargaj (https://github.com/Gargaj/Bonzomatic)
- "ScintillaGL" project by Mykhailo Parfeniuk (https://github.com/sopyer/ScintillaGL)
- Riverwash LiveCoding Tool by Michał Staniszewski and Michal Szymczyk (http://www.plastic-demo.org)

### Libraries and other included software
- Scintilla editing component by the Scintilla Dev Team (https://www.scintilla.org)
- OpenGL Extension Wrangler Library by Nigel Stewart (http://glew.sourceforge.net)
- mini_al by David Reid (https://github.com/dr-soft/mini_al)
- KISSFFT by Mark Borgerding (https://github.com/mborgerding/kissfft)
- STB Image and Truetype libraries by Sean Barrett (https://nothings.org)
- GLFW by whoever made GLFW (https://www.glfw.org/faq.html)
- JSON++ by Hong Jiang (https://github.com/hjiang/jsonxx)
- NDI(tm) SDK by NewTek(tm) (https://www.newtek.com/ndi.html)
- Mongoose Embedded Web Server Library by Cesanta (https://github.com/cesanta/mongoose)

These software are available under their respective licenses.

The network part has been written by NuSan and is public domain.
The remainder of this project code was (mostly, I guess) written by Gargaj / Conspiracy and is public domain.
OSX / macOS maintenance and ports by Alkama / Tpolm + Calodox; Linux maintenance by PoroCYon / K2.

## Contact / discussion forum
If you have anything to say, do it at https://www.pouet.net/topic.php?which=9881 or [![Join the chat at https://gitter.im/Gargaj/Bonzomatic](https://badges.gitter.im/Gargaj/Bonzomatic.svg)](https://gitter.im/Gargaj/Bonzomatic?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
