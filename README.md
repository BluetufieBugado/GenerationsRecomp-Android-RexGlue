# Sonic Generations Recomp to Android

This repo contains a recompiled version of Sonic Generations for Android, based on the Xbox 360 version; the original work was developed by [Player1444](https://github.com/Player124413/Sonic-Generations-recomp-android-and-pc-edition).
My work on this "fork" involved adding compatibility for Mali GPUs (documented ![here](docs/mali-g57.md) ) and creating a map to improve performance (or so I hope).
Currently, performance hovers around 2 to 4 fps in my tests, though I believe this may vary depending on the device.
<img width="2340" height="1080" alt="screenshot" src="https://github.com/user-attachments/assets/c7cfd37c-9350-43da-92ab-effe53651ceb" />

# Original Readme.md - from Player1444
sonicgenerations -- Android project sources
android/   launcher app (Gradle project)
port/      recompiled game (generated C++ + src + *.toml)
Build: run tools/android_sdk.sh (needs android/sdk), then
  cd android && ./gradlew assembleRelease -PrexName=sonicgenerations     -PrexPortDir=$PWD/../port -PrexSdkDir=$PWD/sdk/rexglue-sdk

I’m working on the Sonic Generations recompilation for PC and android, but I don't know what to do about the Android version—the game lags and has graphical glitches. Please help me fix the issues with the Android port; I would be incredibly grateful. Everything should work perfectly on PC, though.

To start the recompilation, you need to download Sonic Generations (USA, Europe).iso.
