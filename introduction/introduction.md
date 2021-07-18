# Introduction

Android is an operating system intended for and widely used in mobile devices. It is free and open-source software; its source code is known as Android Open Source Project (AOSP), which is primarily licensed under the Apache License. Many projects, the most recognised of which is LineageOS, fork AOSP and add their own patches on top. Generally there exist 3 different "types" of ROMs that you might have seen in XDA forums

* **AOSP based ROMs** like AospExtended/Evolution-X directly fork the AOSP source code and customize it. This has it's benefits like faster updates and more features, but there also exist disadvantages, that mostly boil down to ease of porting which we will get to.

* **LineageOS based ROMs** like LineageOS/crDroid are generally considered more stable and easy to port compared to AOSP ROMs. This is because Lineage is slower to incorporate AOSP updates into their repos. LineageOS itself also has less features compared to other ROMs, although many LOS based ROMs like crDroid are feature rich, just like AOSP ones.

* **CAF based ROMs** like AOSPA are based on the **C**ode **A**urora **F**orum (CAF) sources. This is where Qualcomm, the company that manufactures Snapdragon SoCs, uploades their sources for their chipsets. CAF based ROMs are, therefore, faster and more optimized for Qualcomm processors than AOSP or LineageOS. However they can be especially tedious to port so I would not recommend a beginner to attempt porting a CAF ROM. 

## Structure

The Android source code is too large to be kept in a single repository. For this reason it is split into many repos. Because cloning all them individually is a terrible idea, in every Android ROM there exists a central repo called the "manifest" or sometimes "android" that contains xml files that control which and where repos will be cloned. An example is the LineageOS manifest: https://github.com/LineageOS/android/blob/lineage-18.1/default.xml. 

At the beginning the AOSP remote URL is defined along with an android tag, which basically acts like a production ready "snapshot" of the source code:

```c
<remote  name="aosp"
           fetch="https://android.googlesource.com"
           review="android-review.googlesource.com"
           revision="refs/tags/android-11.0.0_r37" />
```

and after that, follow all the projects that Android depends on:

```c
<project path="hardware/qcom/bootctrl" name="platform/hardware/qcom/bootctrl" groups="pdk-qcom" remote="aosp" />
<project path="hardware/qcom/bt" name="platform/hardware/qcom/bt" groups="qcom,pdk-qcom" remote="aosp" />
<project path="hardware/qcom/camera" name="platform/hardware/qcom/camera" groups="qcom_camera,pdk-qcom" remote="aosp" />
...
```

If you are perceptive enough, you might have noticed that some paths have "LineageOS" prefix in their name:

```c
<project path="hardware/qcom/display" name="LineageOS/android_hardware_qcom_display" groups="pdk-qcom,qcom,qcom_display" />
```

These are the repos, LineageOS has forked and customized. All ROMs work similarly, most repos are cloned directly from AOSP while others are customized from the ROM developers to add features/fix bugs etc. Which repos are modified and what customizations are made, solely depends on the ROM at hand.

## Architecture

The basic architecture of the Android OS is the same across all ROM types. The following diagram shows the major components of the Android platform:

Don't worry if you don't understand something just yet. This tutorial has an overview for every category, but each of them will be explained in detail in a seperate document.

### **Linux Kernel**
The foundation of the Android platform is the Linux kernel. Android devices use a version of the Linux kernel with a few special additions. In Android system (as well as desktop systems) the kernel has the role of communicating with the hardware resources of the device and providing an interface for Android to take advantage of them.

### **Hardware abstraction layer (HAL)** 
The hardware abstraction layer (HAL) provides standard interfaces that expose device hardware capabilities to the higher-level Java API framework. The HAL consists of multiple library modules, each of which implements an interface for a specific type of hardware component, such as the camera or bluetooth module. When a framework API makes a call to access device hardware, the Android system loads the library module for that hardware component.

### **Native C/C++ Libraries**
Many core Android system components and services, such as ART and HAL, are built from native code that require native libraries written in C and C++. The Android platform provides Java framework APIs to expose the functionality of some of these native libraries to apps. For example, you can access OpenGL ES through the Android framework’s Java OpenGL API even though OpenGL is implemented as a C native library.

### **Framework**
In contrast to native libraries and HALs, which are generally written in C++, the framework is written in Java and provides a high-level interface to the underlying native libraries and HALs. The entire feature-set of the Android OS is available to you through framework APIs. For this reason, the application framework is used most often by app developers. 

### **Android Runtime** 
For devices running Android version 5.0 (API level 21) or higher, each app runs in its own process and with its own instance of the Android Runtime (ART). ART is written to run multiple virtual machines on low-memory devices by executing DEX files, a bytecode format designed specially for Android that's optimized for minimal memory footprint. Build tools, such as d8, compile Java sources into DEX bytecode, which can run on the Android platform.

### **Apps**
This is probably the most familiar part of Android. Android comes with a set of core apps for E-Mail, SMS messaging, calendars, internet browsing, contacts, and more. There are also system apps, that function both as apps for users and provide key capabilities like the Launcher3 app that defines the homescreen and app layout.

## Documentation

The Android OS is very large and complicated and this tutorial cannot include every single detail about it.
So I encourage everyone who is intersted in Android development, to take a look at the AOSP documentation after reading this guide: https://source.android.com/ which goes in depth, explaining the Android architecture. Some of the content will be adapted into this guide.

## Methodology

This guide is written with simplicity in mind. Anyone interested in Android developmenet should be able to comprehend and fully understand the topics discussed. For this reason there will be plenty of examples derived from real devices. My aim is to take a deep dive into the internal structure of Android, not to stratch the surface and provide a high level overview like most guides do. 
