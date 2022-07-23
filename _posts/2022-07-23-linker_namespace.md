---
title: VNDK
description: VNDK and namespace
categories:
 - android
tags: [VNDK]
---

VNDK(Vendor Native Development Kit)
The Vendor Native Development Kit (VNDK) is a set of libraries exclusively for vendors to implement their HALs.

# Why VNDK?
Framework-only updates include the following challenges:
Dependency between framework modules and vendor modules.
Extensions to AOSP libraries. 

To address these challenges, Android 8.0 introduces several techniques such as VNDK (described in this section), HIDL, hwbinder, device tree overlay, and sepolicy overlay.

由于我们不是google，或者vendor，所以体会不到该问题的所有方面

VNDK concepts
* Framework shared libraries for vendor
two approaches to support vendor modules across multiple Android releases:
    1. stabilize the ABIs/APIs of the framework shared libraries.
    2. copy old framework shared libraries.


    framework shared libraries are classified into three sub-categories:
    1. LL-NDK Libraries that has stable API/ABI.
    2. Eligible VNDK Libraries(VNDK) that are safe to be copied twice.
    3. Framework-only Libraries (FWK-only)


* Same-Process HAL (SP-HAL)

    It is a set of predetermined HALs implemented as Vendor Shared Libraries and loaded into Framework Processes. SP-HALs must depend only on LL-NDK and VNDK-SP. VNDK-SP is a predefined subset of eligible VNDK libraries. VNDK-SP libraries are carefully reviewed to ensure double-loading VNDK-SP libraries into framework processes does not cause problems.


---

Linker Namespace 
# two challenge in Treble VNDK design
 * SP-HAL shared libraries and their dependencies, including VNDK-SP libraries, are loaded into framework processes. There should be some mechanisms to prevent symbol conflicts.
 * dlopen() and android_dlopen_ext() can introduce some runtime dependencies that aren't visible at build time and can be difficult to detect using static analysis.

# How does it work?
The dynamic linker is responsible for loading the shared libraries specified in DT_NEEDED entries or the shared libraries specified by the argument of dlopen() or android_dlopen_ext(). In both cases, the dynamic linker finds the linker namespace where the caller resides and tries to load the dependencies into the same linker namespace. If the dynamic linker can't load the shared library into the specified linker namespace, it asks the *linked linker namespace* for exported shared libraries.
# Configuration file format
## Directory-section mapping property
dir.name
A path to a directory that the [name] section applies to.


dir.system = /system/bin
dir.system = /system/xbin
[system] section applies to the executables that are loaded from either /system/bin or /system/xbin.

dir.vendor = /vendor/bin
[vendor] section applies to the executables that are loaded from /vendor/bin.

一个配置文件有多个section，每个section都对应一个目录下可执行的namespace配置
## Relation properties
links

we can also specify the only library as the requested library name for fallback link.
## Namespace properties
isolated
This indicates that only the shared libraries in search.paths or under permitted.paths can be loaded into the  namespace.

visible
A boolean value that indicates whether the program (other than libc) can obtain a linker namespace handle with android_get_exported_namespace() and open a shared library in the linker namespace by passing the handle to android_dlopen_ext().

search.paths
permitted.paths


# Linker namespace creation
In Android 11, linker configuration is created at runtime under /linkerconfig instead of using plain text files in ${android-src}/system/core/rootdir/etc. The configuration is generated at boot time based on the runtime environment, which includes following items:
Linker configuration is created by resolving dependencies between linker namespaces. For example, if there are any updates on the APEX modules that include dependency updates, linker configuration is generated reflecting these changes.

# Linker namespace isolation
## VNDK configuration
## VNDK Lite configuration

# LinkerConfig
# two special namespace
default
anonymous
android_init_anonymous_namespace().
init_default_namespaces().
#  how to change android 11 linker config
## how create the linker namespace?




# protobuf
https://cloud.tencent.com/developer/article/1484399
# Reference
https://source.android.com/devices/architecture/vndk
https://source.android.com/devices/architecture/vndk/linker-namespace
https://source.android.com/static/devices/architecture/images/VNDK.pdf
https://android.googlesourcecom/platform/systemlinkerconfig/+/refs/headsmaster
https://jackwish.net/blog/2017/namespace-based-dynamic-linking.html

