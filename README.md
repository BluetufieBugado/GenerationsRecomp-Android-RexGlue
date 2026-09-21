# Sonic Generations Recomp to Android

This repo contains a recompiled version of Sonic Generations for Android, based on the Xbox 360 version; the original work was developed by [Player1444](https://github.com/Player124413/Sonic-Generations-recomp-android-and-pc-edition).
My work on this "fork" involved adding compatibility for Mali GPUs (documented ![here](docs/mali-g57.md) ) and creating a map to improve performance (or so I hope).
Currently, performance hovers around 2 to 4 fps in my tests, though I believe this may vary depending on the device.
<img width="2340" height="1080" alt="screenshot" src="https://github.com/user-attachments/assets/c7cfd37c-9350-43da-92ab-effe53651ceb" />

# A straightforward summary of what needs to be done to improve (currently)

1- Render below native resolution (e.g., half-res). Currently, the minimum is 1x (full 360 resolution). The Mali GPU (specifically) struggles with fragment processing; rendering at a lower resolution and then upscaling would cut the GPU workload in half or more (improving performance for everyone). This offers the biggest potential gain but requires engine-level changes.

2 - Change texture formats (DXT → ETC2). The game currently uses a texture format the Mali GPU doesn't support, so the port converts everything to a format that is four times larger. ETC2 is natively supported by Mali; textures would return to their original size, and the upload/resolution load would be reduced fourfold. (This would boost performance for everyone, not just Mali users; I imagine Snapdragon devices lacking good custom driver support could also benefit.)

3 - Fix the vertex memexport fallback. In my tests, Sonic sometimes appeared with a glitched hand; this is a direct symptom—since the feature had to be skipped at boot, that specific code path is broken. Reimplementing it using compute shaders would resolve this issue (it wouldn't increase FPS, but it would fix a graphical glitch).

4 - Speed ​​up CPU emulation. The ~370ms/frame spent on guest code is a major issue. This requires deep-level work—such as a better recompiler or profile-guided optimization—which falls into SDK territory.

5 - Change a default setting (affinity). The current default causes glitches on big.LITTLE architectures; flipping it won't boost FPS, but it will eliminate bugs.

# Disclaimer and RexGlue

This project was written using RexGlue, which means it is less efficient than a project built with XenonRecomp and xenos recomp tools.
This marks the limit of my contribution; I don't think I'll be able to do much more than this with the RexGlue version.

# Original Readme.md - from Player1444
sonicgenerations -- Android project sources
android/   launcher app (Gradle project)
port/      recompiled game (generated C++ + src + *.toml)
Build: run tools/android_sdk.sh (needs android/sdk), then
  cd android && ./gradlew assembleRelease -PrexName=sonicgenerations     -PrexPortDir=$PWD/../port -PrexSdkDir=$PWD/sdk/rexglue-sdk

I’m working on the Sonic Generations recompilation for PC and android, but I don't know what to do about the Android version—the game lags and has graphical glitches. Please help me fix the issues with the Android port; I would be incredibly grateful. Everything should work perfectly on PC, though.

To start the recompilation, you need to download Sonic Generations (USA, Europe).iso.
