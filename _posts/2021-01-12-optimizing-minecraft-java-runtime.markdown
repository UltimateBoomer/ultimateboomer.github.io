---
title: "Optimizing Minecraft Java Runtime"
description: "Test"
date: 2021-1-12
published: true
---

I tested some ways to optimize Minecraft and improve the default Java Edition setup.

## TL;DR

In short, these are my recommended setups for Java Edition.

**Note**: There are some older Forge versions that do not work because they are incompatible with OpenJ9 and newer Java versions.

#### High performance option - For systems that run Minecraft well already

1. Download an [AdoptOpenJDK](https://adoptopenjdk.net/releases.html) JRE

- Choose **OpenJDK 15**, and select **HotSpot** as the JVM
- If your mods don't support Java 15, choose Java 11 instead
- Download the zip version, there is no need to use the installer

2. Unpack the JRE in a directory of your choice

- For ease of use put it in the same directory as the built-in JRE, which is under `C:/Program Files (x86)/Minecraft Launcher/runtime`

3. In your launcher profile, change the Java executable to: `(Path to new JRE)/bin/javaw.exe`
4. Change the JVM arguments to: `-XmsNG -XmxNG -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -XX:+AlwaysPreTouch`

- Replace `N` with the amount of memory to allocate

#### Low memory option - For systems with low memory size

1. Download an [AdoptOpenJDK](https://adoptopenjdk.net/releases.html) JRE

- Choose **OpenJDK 15**, and select **OpenJ9** as the JVM
- If your mods don't support Java 15, choose Java 11 or 8 instead
- Download the zip version, there is no need to use the installer

2. Unpack the JRE in a directory of your choice

- For ease of use put it in the same directory as the built-in JRE, which is under `C:/Program Files (x86)/Minecraft Launcher/runtime`

3. In your launcher profile, change the Java executable to: `(Path to new JRE)/bin/javaw.exe`
4. Change the JVM arguments to: `-XmsNG -XmxNG -Xgcpolicy:gencon`

- Replace `N` with the amount of memory to allocate

## Garbage Collection Improvements with Newer Java Versions

The biggest improvement with the newer Java versions is the new garbage collector, _[Shenandoah](https://wiki.openjdk.java.net/display/shenandoah/Main)_. _Shenandoah_ is designed for short pause times over high throughout, so it is suitable for games like Minecraft. GC pauses are one major source of stutters in Minecraft, which is shown as the orange lines in Optifine's FPS graph. The stock garbage collector used is G1GC, which causes noticeable stutters during gameplay. With the new garbage collector, this stutter is pretty much gone. This won't remove all stutters from the game however.

It's not completely scientific, but here is the general comparison between the typical Minecraft setup and this improved setup.

**Stock setup** - Frequent stutters:
![Stock Perf](/assets/optimizing-minecraft-java-runtime/perf2.png)

**Stock setup, fixed 4 GB allocated** - GC stutters are less frequent but longer:
![Stock 4GB Perf](/assets/optimizing-minecraft-java-runtime/perf3.png)

**Java 15, fixed 4 GB allocated and using Shenandoah GC** - GC stutters are virtually gone:
![New Perf](/assets/optimizing-minecraft-java-runtime/perf1.png)

_Shenandoah_ is available on AdoptOpenJDK for Java 11 and higher. It can be used by replacing the _G1GC_ arguments with `-XX:+UseShenandoahGC`. There is more room for tuning, such as using `-XX:+AlwaysPreTouch` which may give a small performance uplift. You can read more about _Shenandoah_ and ways to optimize it in its [OpenJDK wiki page](https://wiki.openjdk.java.net/display/shenandoah/Main).

There's another GC that is similar to Shenandoah, which is _[ZGC](https://wiki.openjdk.java.net/display/zgc/Main)_. Out of the box _ZGC_ isn't suitable for Minecraft as it creates long periods of stutters. I haven't tested this GC much, so I can't say much about its performance after it is tuned properly.

## Memory Allocation

There are two types of memory used by Java: _on-heap_ and _off-heap_ memory. You can change the amount of memory used by Minecraft using the JVM parameters `-Xms` and `-Xmx`. These are the amount of _on-heap_ memory that will be used by Minecraft, which includes the world, integrated server, and anything that Minecraft needs to run. _On-heap_ memory is subject to garbage collection.

Minecraft also use _off-heap_ memory, which are allocated for resources like textures. This means that Minecraft will **use more memory that the amount allocated to it**. Here I have loaded up a complete 256x resource pack. I allocated 4 GB to Minecraft, yet task manager shows the game (OpenJDK Platform Binary) using over 12 GB of memory.

![Memory](/assets/optimizing-minecraft-java-runtime/mem.png)

What's more, some of the memory usage isn't shown in the Minecraft process. You can see in the screenshot that 67% of my 32 GB of ram is being used, and that doesn't seem to add up. I assume it's because some memory are used by the GPU process which isn't shown in the task manager.

Another reason to not allocate to much memory is that each garbage collection will take more time, creating bigger individual stutters.

So, the takeaway here is that you shouldn't allocate more memory than Minecraft needs. For vanilla and most modded setups, 4 GB should be fine. Only increase the allocated RAM if Minecraft is throwing out of memory errors or the garbage collector is running too frequently.

## Hotspot vs. OpenJ9

There are two different JVM options, **Hotspot** and **OpenJ9**. HotSpot is the traditional JVM used by Java applications like Minecraft. OpenJ9 is a newer JVM that is said to have better startup times and lower memory footprint than HotSpot. With newer OpenJ9 releases, [the issue with static initialization is now fixed](https://github.com/iczero/fabric-openj9compat) and it should compatible with any Minecraft version. Here is the memory usage of HotSpot vs. OpenJ9. The render distance set to 16 chunks, and no resource packs above 16x are used. Both setups have 4 GB of memory allocated.

**HotSpot**:
![HotSpot Mem](/assets/optimizing-minecraft-java-runtime/memtest2.png)

**OpenJ9**:
![OpenJ9 Mem](/assets/optimizing-minecraft-java-runtime/memtest1.png)

As you can see, **Openj9 uses a lot less memory than HotSpot**. Even with the `-Xms4G` argument it doesn't use more memory than it needs. This shows that if your computer doesn't have enough RAM to run Minecraft, using OpenJ9 is a good option.

Even though OpenJ9 has an impressive memory efficiency, the major downside is that **OpenJ9 is slower than HotSpot, and doesn't have a comparable garbage collector to _Shenandoah_**. When using OpenJ9, I noticed that I was getting much more FPS fluctuations than HotSpot. For the garbage collector options, the best GC from my testing is _gencon_, which is only in the G1 levels of performance. Furthermore, if you are using high resolution resource packs, the gap in memory usage closes as more memory is taken up by the textures. If your computer has plenty of RAM, just go with HotSpot + Shenandoah which will yield more performance over memory efficiency.
