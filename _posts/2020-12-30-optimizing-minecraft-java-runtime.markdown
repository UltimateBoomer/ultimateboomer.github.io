---
layout: post
title: "Optimizing Minecraft Java Runtime"
description: "Test"
date: 2020-12-30
categories: minecraft
published: false
---
Java, being a cross-platform language that runs on a JVM, has many optimizations that have been added over time. The latest Java LTS version is Java 15, and many applications are now built on Java 11. However, Minecraft still runs on Java 8, which is fairly old at this point and lacks some performance-improving features.

This article shows how to optimize Minecraft's Runtime Environment and some things to look out for. The guide should apply to pretty much any version of Minecraft, except for a few older Forge versions.

## TL;DR

This is my ideal Minecraft setup.

**Note**: There are some older Forge versions like 1.12.2 that do not work because it is incompatible with newer Java versions. In that case, you would need to find or build a Java 8 JRE that includes Shenandoah.

1. Download an [AdoptOpenJDK](https://adoptopenjdk.net/releases.html) JRE
- Choose **OpenJDK 15**, and select **HotSpot** as the JVM
- Download the zip version, there is no need to use the installer
2. Unpack the JRE in a directory of your choice
- For ease of use I would put it in the same directory as the built-in JRE, which is under C:/Program Files (x86)/Minecraft Launcher/runtime
3. In your Minecraft Launcher profile, change the Java executable to: (Path to new JRE)/bin/javaw.exe
4. Change the JVM arguments to: `-Xms4G -Xmx4G -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -XX:+AlwaysPreTouch`
  
## Garbage Collection Improvements with Newer Java Versions

The biggest improvement with newer Java versions is the new garbage collector, *[Shenandoah](https://wiki.openjdk.java.net/display/shenandoah/Main)*. *Shenandoah* is designed for short pause times over high throughout, so it is suitable for games like Minecraft. GC pauses are one major source of stutters in Minecraft, which is shown as the orange lines in Optifine's FPS graph. The stock garbage collector used is G1GC, which causes noticeable stutters during gameplay. With the new garbage collector, this stutter is pretty much gone. This won't remove all stutters from the game however.

It's not completely scientific, but I've tested the typical Minecraft setup with the new setup. The render distance is set at 16 chunks, and the framerate is capped to 100 so the FPS graph won't scroll too quickly. Here are the results. 

**Stock setup (G1GC)** - Frequent stutters: 
![Stock Perf](/assets/optimizing-minecraft-java-runtime/perf2.png)

**Stock setup with fixed 4 GB allocated** - GC stutters are less frequent but longer:
![Stock 4GB Perf](/assets/optimizing-minecraft-java-runtime/perf3.png)

**Java 15 setup with fixed 4GB allocated and Shenandoah GC** - GC stutters are virtually gone:
![New Perf](/assets/optimizing-minecraft-java-runtime/perf1.png)

*Shenandoah* is available on AdoptOpenJDK for Java 11 and higher. It can be used by replacing the *G1GC* arguments with `-XX:+UseShenandoahGC`. There is more room for tuning, such as using `-XX:+AlwaysPreTouch` which may give a small performance uplift. You can read more about *Shenandoah* in its [OpenJDK wiki page](https://wiki.openjdk.java.net/display/shenandoah/Main).

There's another GC that is similar to Shenandoah, which is *[ZGC](https://wiki.openjdk.java.net/display/zgc/Main)*. Out of the box *ZGC* isn't suitable for Minecraft as it creates long periods of stutters. I haven't tested this GC much, so I can't say much about its performance after it is tuned.

## Memory Allocation

There are two types of memory used by Java: *on-heap* and *off-heap* memory. You may already know that you can change the amount of memory used by Minecraft using `-Xms` and `-Xmx`. These are the amount of *on-heap* memory that will be used by Minecraft, which includes the world, integrated server, and anything that Minecraft needs to run. *On-heap* memory is subject to garbage collection, and the more you allocate the longer the stutter cause by garbage collection will be. You can see this happening in the previous screenshots with the stock setup vs. stock setup with 4 GB allocated.

Minecraft also use *off-heap* memory, which are allocated for resources like textures. This means that Minecraft will **use more memory that the amount allocated to it**. Here I have loaded up a complete 256x resource pack. I allocated 4 GB to Minecraft, yet task manager shows the game (OpenJDK Platform Binary) using over 12 GB of memory!

![Memory](/assets/optimizing-minecraft-java-runtime/mem.png)

What's more, some of the memory usage isn't even shown in the Minecraft process. You can see in the screenshot that 67% of my 32 GB of ram is being used (*~24 GB*), and that doesn't seem to add up. I assume it's because some memory are used by the GPU process which isn't shown in the task manager.

So, the takeaway here is that you shouldn't allocate more memory than Minecraft needs. For example, if you have 16 GB of ram and other programs are using 6 GB in total, don't allocate the remaining 10 GB to Minecraft.

## Hotspot vs. OpenJ9

There are two different JVM options, **Hotspot** and **OpenJ9**. HotSpot is the traditional JVM used by Java applications like Minecraft. OpenJ9 is a newer JVM that is said to have better startup times and lower memory footprint than HotSpot. With newer OpenJ9 releases, [the issue with static initialization is now fixed](https://github.com/iczero/fabric-openj9compat) and it should compatible with any Minecraft version. Here is the memory usage of HotSpot vs. OpenJ9. The render distance set to 16 chunks, and no resource packs above 16x are used. Both setups have 4 GB of memory allocated.

**HotSpot**:
![HotSpot Mem](/assets/optimizing-minecraft-java-runtime/memtest2.png)

**OpenJ9**:
![OpenJ9 Mem](/assets/optimizing-minecraft-java-runtime/memtest1.png)

As you can see, **Openj9 uses a lot less memory than HotSpot**. Even with the `-Xms4G` argument it doesn't use more memory than it needs. This shows that if your computer doesn't have enough RAM to run Minecraft, using OpenJ9 is a good option. However, if you are using high resolution resource packs, the gap in memory usage closes as more memory is used by the textures.

Even though OpenJ9 has an impressive memory efficiency, the major downside is that **OpenJ9 doesn't have a comparable garbage collector to *Shenandoah***. The best GC for Minecraft in my testing is *gencon* (apart from *metronome* which is Linux/AIX only), which is only in the G1GC levels of performance. If your computer has plenty of RAM, just go for HotSpot+Shenandoah.
