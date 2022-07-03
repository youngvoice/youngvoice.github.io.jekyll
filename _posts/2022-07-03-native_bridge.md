---
title: the mechanism of native bridge
description: design and implement and use of native bridge
categories:
 - android
tags: [native bridge]
---
# how to conclude nativebridge?
first stage: initialize  (LoadNativeBridge())
second stage: load lib(NativeBridgeLoadLibraryExt()), get_trampline



注意如下两个东西的区别：
libnativebridge.so 是nativebridge的接口
nativebridge的实现, xxxxx

# the first stage ---> initialize
Native Bridge is implemented as a part of ART in the Android architecture. There are two stages for the Native Bridge to be initialized:
1. In the first stage, it is common for all applications. The native bridge xxxxx is loaded in the system as part of the ART initialization process.
2. In the second stage, the native bridge xxxxx will be initialized and ready to be used for the applicatio, when an application with a different processor architecture native lib is started. It will be forked form Zygote, this is specific for individual applications.

When the application starts to load and execute a native lib from a different processor architecture, the Native bridge xxxxx will help to resolve the loading and executing of this library.

When the android system has just started, Native Bridge is in a kNotSetup state. 
$AOSP/art/libnativebridge
during the first stage, that is, initialization of ART, the nativebridge will be loaded into the system and the nativebridge status will changes from kNotSetup to KOpened.

In the second stage, that is, the system fork a new process from Zygote, the system will do some pre-initialization work for Native Bridge, the status changes to kPreInitialized. After the process is forked from Zygote, Native Bridge is initialized as part of the process creation and it's state changes to kInitialized.



the first stage function call:

|ART|android|
|---|-------|
|Runtime::init()| |
|LoadNativeBridge()[native_bridge_art_interface.cc]| |
| | android::LoadNativeBridge(native_bridge_art_callbacks)[art/libnativebridge/native_bridge.cc]|


the second stage function call:
during the creation of android applications.

Pre-initializing Native Bridge
|ART|native bridge|
|---|-------------|
|com_android_internal_os_Zygote_nativeForkAndSpecialize| |
| SpecializeCommon() | |
| | android::PreinitializeNativeBridge|

Initializing Native Bridge
|ART|native bridge|JNI|xxxxx|
|---|-------------|---|-------|
|continue in SpecializeCommon()| | | |
|gCallPostForkChildHooks() | | | |
|  | | ZygoteHooks_nativePostForkChild[art/runtime/native/dalvik_system_ZygoteHooks.cc]| |
|  | | InitNonZygoteOrPostFork[art/runtime/runtime.cc]| |
| InitializeNativeBridge[art/runtime/native_bridge_art_interface.cc] | | | |
| | InitializeNativeBridge | | |
| |  | |initialize |
| | SetupEnvironment | | |
| | | | getAppEnv|

# loading a native library

|App|Runtime|libart.so|nativeloader|nativebridge|xxxx|
|---|-------|---------|------------|------------|----|
|System.loadLibrary| | | | | |
| |LoadNativeLibrary | | | | |
| |  | OpenNativeLibrary| | | |
| |  |  | ... NativeBridgeLoadLibraryExt| | |
| |  |  | |callbacks->LoadLibraryExt |LoadLibraryExt |
| |  | library->FindSymbol("JNI_Onload") | |  | |
| |  | NativeBridgeGetTrampoline | |  | |
| |  |   | |callback->getTrampoline  |getTrampoline|
| |  |  |  |return stub address  ||


