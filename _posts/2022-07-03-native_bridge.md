---
title: the mechanism of native bridge
description: design and implement and use of native bridge
categories:
 - android
tags: [native bridge]
---

                                                                                         
                                                                                         
                 +------------------+                                                           
                 |                  |                                                           
                 |  Native Bridge   |                                                           
                 |                  |<----------------------------------------+                 
                 |                  |                                         |                 
                 +--------+---------+                                         |                 
                          |                                                   |                 
                          |                                                   |                 
                NativeBridgeCallbacks                                         |                 
                          |                                                   |                 
                          |                                                   |                 
                          v                                                   |                 
                 +------------------+                               +---------+--------+        
                 |                  |                               |                  |        
                 |     xxxxxx       |------------------------------>|    ART           |        
                 |                  |                               |                  |        
                 |                  |                               |                  |        
                 +------------------+                               +------------------+        
                                                                                         
                                      NativeBridgeRuntimeCallbacks                              
                                                                                         


                 +--------------------------------------------------------------------+
                 |                                                                    |
                 |                                                                    |
                 |                                                                    |
                 |                       Applications                                 |
                 |                                                                    |
                 +--------------------------------------------------------------------+
                 |                                                                    |
                 |                                                                    |
                 |                                                                    |
                 |                   Application Framework                            |
                 |                                                                    |
                 +-------------------------------+------------------------------------+
                 |                               |                                    |
                 |                               |                                    |
                 |                               |       Native Bridge                |
                 |                               |                                    |
                 |                               +------------------------------------+
                 |                               |         xxxxxxxx                   |
                 |                               |                                    |
                 |                               +------------------------------------+
                 |                                                                    |
                 |                    Libraries/ART                                   |
                 |                                                                    |
                 |                                                                    |
                 |                                                                    |
                 |                                                                    |
                 +--------------------------------------------------------------------+

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





```c++
// art/libnativebridge/include/nativebridge/native_bridge.h
// Runtime interfaces to native bridge.
struct NativeBridgeRuntimeCallbacks {
  // Get shorty of a Java method. The shorty is supposed to be persistent in memory.
  //
  // Parameters:
  //   env [IN] pointer to JNIenv.
  //   mid [IN] Java methodID.
  // Returns:
  //   short descriptor for method.
  const char* (*getMethodShorty)(JNIEnv* env, jmethodID mid);

  // Get number of native methods for specified class.
  //
  // Parameters:
  //   env [IN] pointer to JNIenv.
  //   clazz [IN] Java class object.
  // Returns:
  //   number of native methods.
  uint32_t (*getNativeMethodCount)(JNIEnv* env, jclass clazz);

  // Get at most 'method_count' native methods for specified class 'clazz'. Results are outputed
  // via 'methods' [OUT]. The signature pointer in JNINativeMethod is reused as the method shorty.
  //
  // Parameters:
  //   env [IN] pointer to JNIenv.
  //   clazz [IN] Java class object.
  //   methods [OUT] array of method with the name, shorty, and fnPtr.
  //   method_count [IN] max number of elements in methods.
  // Returns:
  //   number of method it actually wrote to methods.
  uint32_t (*getNativeMethods)(JNIEnv* env, jclass clazz, JNINativeMethod* methods,
                               uint32_t method_count);
};
```
These callback functions provide a way for native methods to call JNI native functions. We will see how this callback data structure is passed to the actual Native Bridge implementation later. In our case, the actual implementation is the xxxxx library.






```c++
// art/libnativebridge/include/nativebridge/native_bridge.h

// Native bridge interfaces to runtime.
struct NativeBridgeCallbacks {
  // Version number of the interface.
  uint32_t version;

  // Initialize native bridge. Native bridge's internal implementation must ensure MT safety and
  // that the native bridge is initialized only once. Thus it is OK to call this interface for an
  // already initialized native bridge.
  //
  // Parameters:
  //   runtime_cbs [IN] the pointer to NativeBridgeRuntimeCallbacks.
  // Returns:
  //   true if initialization was successful.
  bool (*initialize)(const struct NativeBridgeRuntimeCallbacks* runtime_cbs,
                     const char* private_dir, const char* instruction_set);

  // Load a shared library that is supported by the native bridge.
  //
  // Parameters:
  //   libpath [IN] path to the shared library
  //   flag [IN] the stardard RTLD_XXX defined in bionic dlfcn.h
  // Returns:
  //   The opaque handle of the shared library if sucessful, otherwise NULL
  //
  // Starting with v3, NativeBridge has two scenarios: with/without namespace.
  // Use loadLibraryExt instead in namespace scenario.
  void* (*loadLibrary)(const char* libpath, int flag);

  // Get a native bridge trampoline for specified native method. The trampoline has same
  // sigature as the native method.
  //
  // Parameters:
  //   handle [IN] the handle returned from loadLibrary
  //   shorty [IN] short descriptor of native method
  //   len [IN] length of shorty
  // Returns:
  //   address of trampoline if successful, otherwise NULL
  void* (*getTrampoline)(void* handle, const char* name, const char* shorty, uint32_t len);

  // Check whether native library is valid and is for an ABI that is supported by native bridge.
  //
  // Parameters:
  //   libpath [IN] path to the shared library
  // Returns:
  //   TRUE if library is supported by native bridge, FALSE otherwise
  //
  // Starting with v3, NativeBridge has two scenarios: with/without namespace.
  // Use isPathSupported instead in namespace scenario.
  bool (*isSupported)(const char* libpath);

  // Provide environment values required by the app running with native bridge according to the
  // instruction set.
  //
  // Parameters:
  //   instruction_set [IN] the instruction set of the app
  // Returns:
  //   NULL if not supported by native bridge.
  //   Otherwise, return all environment values to be set after fork.
  const struct NativeBridgeRuntimeValues* (*getAppEnv)(const char* instruction_set);

  // Added callbacks in version 2.

  // Check whether the bridge is compatible with the given version. A bridge may decide not to be
  // forwards- or backwards-compatible, and libnativebridge will then stop using it.
  //
  // Parameters:
  //   bridge_version [IN] the version of libnativebridge.
  // Returns:
  //   true if the native bridge supports the given version of libnativebridge.
  bool (*isCompatibleWith)(uint32_t bridge_version);

  // A callback to retrieve a native bridge's signal handler for the specified signal. The runtime
  // will ensure that the signal handler is being called after the runtime's own handler, but before
  // all chained handlers. The native bridge should not try to install the handler by itself, as
  // that will potentially lead to cycles.
  //
  // Parameters:
  //   signal [IN] the signal for which the handler is asked for. Currently, only SIGSEGV is
  //                 supported by the runtime.
  // Returns:
  //   NULL if the native bridge doesn't use a handler or doesn't want it to be managed by the
  //   runtime.
  //   Otherwise, a pointer to the signal handler.
  NativeBridgeSignalHandlerFn (*getSignalHandler)(int signal);

  // Added callbacks in version 3.

  // Decrements the reference count on the dynamic library handler. If the reference count drops
  // to zero then the dynamic library is unloaded.
  //
  // Parameters:
  //   handle [IN] the handler of a dynamic library.
  //
  // Returns:
  //   0 on success, and nonzero on error.
  int (*unloadLibrary)(void* handle);

  // Dump the last failure message of native bridge when fail to load library or search symbol.
  //
  // Parameters:
  //
  // Returns:
  //   A string describing the most recent error that occurred when load library
  //   or lookup symbol via native bridge.
  const char* (*getError)();

  // Check whether library paths are supported by native bridge.
  //
  // Parameters:
  //   library_path [IN] search paths for native libraries (directories separated by ':')
  // Returns:
  //   TRUE if libraries within search paths are supported by native bridge, FALSE otherwise
  //
  // Starting with v3, NativeBridge has two scenarios: with/without namespace.
  // Use isSupported instead in non-namespace scenario.
  bool (*isPathSupported)(const char* library_path);

  // Initializes anonymous namespace at native bridge side.
  // NativeBridge's peer of android_init_anonymous_namespace() of dynamic linker.
  //
  // The anonymous namespace is used in the case when a NativeBridge implementation
  // cannot identify the caller of dlopen/dlsym which happens for the code not loaded
  // by dynamic linker; for example calls from the mono-compiled code.
  //
  // Parameters:
  //   public_ns_sonames [IN] the name of "public" libraries.
  //   anon_ns_library_path [IN] the library search path of (anonymous) namespace.
  // Returns:
  //   true if the pass is ok.
  //   Otherwise, false.
  //
  // Starting with v3, NativeBridge has two scenarios: with/without namespace.
  // Should not use in non-namespace scenario.
  bool (*initAnonymousNamespace)(const char* public_ns_sonames, const char* anon_ns_library_path);

  // Create new namespace in which native libraries will be loaded.
  // NativeBridge's peer of android_create_namespace() of dynamic linker.
  //
  // Parameters:
  //   name [IN] the name of the namespace.
  //   ld_library_path [IN] the first set of library search paths of the namespace.
  //   default_library_path [IN] the second set of library search path of the namespace.
  //   type [IN] the attribute of the namespace.
  //   permitted_when_isolated_path [IN] the permitted path for isolated namespace(if it is).
  //   parent_ns [IN] the pointer of the parent namespace to be inherited from.
  // Returns:
  //   native_bridge_namespace_t* for created namespace or nullptr in the case of error.
  //
  // Starting with v3, NativeBridge has two scenarios: with/without namespace.
  // Should not use in non-namespace scenario.
  struct native_bridge_namespace_t* (*createNamespace)(const char* name,
                                                       const char* ld_library_path,
                                                       const char* default_library_path,
                                                       uint64_t type,
                                                       const char* permitted_when_isolated_path,
                                                       struct native_bridge_namespace_t* parent_ns);

  // Creates a link which shares some libraries from one namespace to another.
  // NativeBridge's peer of android_link_namespaces() of dynamic linker.
  //
  // Parameters:
  //   from [IN] the namespace where libraries are accessed.
  //   to [IN] the namespace where libraries are loaded.
  //   shared_libs_sonames [IN] the libraries to be shared.
  //
  // Returns:
  //   Whether successed or not.
  //
  // Starting with v3, NativeBridge has two scenarios: with/without namespace.
  // Should not use in non-namespace scenario.
  bool (*linkNamespaces)(struct native_bridge_namespace_t* from,
                         struct native_bridge_namespace_t* to, const char* shared_libs_sonames);

  // Load a shared library within a namespace.
  // NativeBridge's peer of android_dlopen_ext() of dynamic linker, only supports namespace
  // extension.
  //
  // Parameters:
  //   libpath [IN] path to the shared library
  //   flag [IN] the stardard RTLD_XXX defined in bionic dlfcn.h
  //   ns [IN] the pointer of the namespace in which the library should be loaded.
  // Returns:
  //   The opaque handle of the shared library if sucessful, otherwise NULL
  //
  // Starting with v3, NativeBridge has two scenarios: with/without namespace.
  // Use loadLibrary instead in non-namespace scenario.
  void* (*loadLibraryExt)(const char* libpath, int flag, struct native_bridge_namespace_t* ns);

  // Get native bridge version of vendor namespace.
  // The vendor namespace is the namespace used to load vendor public libraries.
  // With O release this namespace can be different from the default namespace.
  // For the devices without enable vendor namespaces this function should return null
  //
  // Returns:
  //   vendor namespace or null if it was not set up for the device
  //
  // Starting with v5 (Android Q) this function is no longer used.
  // Use getExportedNamespace() below.
  struct native_bridge_namespace_t* (*getVendorNamespace)();

  // Get native bridge version of exported namespace. Peer of
  // android_get_exported_namespace(const char*) function.
  //
  // Returns:
  //   exported namespace or null if it was not set up for the device
  struct native_bridge_namespace_t* (*getExportedNamespace)(const char* name);

  // If native bridge is used in app-zygote (in doPreload()) this callback is
  // required to clean-up the environment before the fork (see b/146904103).
  void (*preZygoteFork)();
};
```
this is used as the Native Bridge interface of its actual implementation. In our case, this implementation is the xxxxx library. 
