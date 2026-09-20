sonicgenerations -- Android project sources
android/   launcher app (Gradle project)
port/      recompiled game (generated C++ + src + *.toml)
Build: run tools/android_sdk.sh (needs android/sdk), then
  cd android && ./gradlew assembleRelease -PrexName=sonicgenerations     -PrexPortDir=$PWD/../port -PrexSdkDir=$PWD/sdk/rexglue-sdk

I’m working on the Sonic Generations recompilation for PC and android, but I don't know what to do about the Android version—the game lags and has graphical glitches. Please help me fix the issues with the Android port; I would be incredibly grateful. Everything should work perfectly on PC, though.

To start the recompilation, you need to download Sonic Generations (USA, Europe).iso.
