---
title: "The Principles of Overclocking"
date: 2021-10-03
categories: jekyll update
published: false
---

Overclocking is something that is pretty easy to get into, yet once you start to overclock competitively it can get very complicated. In this article I'll talk about many things that will be quite important if you want to tweak the performance of your hardware. This applies to more than overclocking, but also undervolting as well. If you already know a lot about overclocking, these principles will probably seem like common sense. But there are still far too many misconceptions people have, and a lot of them diminish the efficiency of overclocking. With these principles, you will be able to get the most out of your system.

## Variables

With any kind of processor, memory or other kind of overclockable chip, there are always these factors that need to be considered.

#### Frequency/clock

This is the clock speed of the processor, often measured in MHz or GHz. It is roughly 1:1 with performance given no other bottlenecks. Upping frequency increases the current of the processor linearly.

#### Voltage

This is the voltage sent to the circuits in the processor. Generally a higher voltage allows a higher frequency to be stabilzed, but for some systems too high of a voltage can reduce the stable frequency range.

#### Temperature

The temperature of a processor affects its current draw because higher temperatures leads to greater leakage current. It also affects stability, but the effect varies a lot between different systems. These are the reasons why cooling processors to extremely low temperatures can allow it to clock much higher.

#### Heat Dissipation

The cooling system will greatly affect how much you can overclock a processor because of the benefits of a lower temperature.

#### Power

This is probably the most important factor to pay attention to. Power depends on the frequency, voltage, and temperature, and this makes the power consumption increase polynomially for a linear increase in performance. It's also why undervolting makes a processor much more efficient.

## The Voltage/Frequency Curve

Knowing the V/F curve is very important as it comes up very often in modern processor boost algorithms. 

The V/F curve is basically a curve of the possible voltage and frequency pairs that the processor can run at. The reason it looks like a curve is explained by that increasesing frequency requires an increase in voltage, and at higher frequencys a greater voltage bump is needed to stabilize the overclock. 

### Shifts to the V/F Curve

Shifting the curve up or to the right decreases the voltage supplied for any given frequency, and shifting the curve down or to the left increases the voltage supplied for any given frequency.

The former can reduce stability, but **it is always more efficient**. If you want the most efficient clock configuration, whether if you want to overclock or not, always set either a positive clock offset or a negative voltage offset. The only reason to go the other way is if the stock settings are unstable, which points to a whole other problem entirely.

### Power limits

Power limits are often implemented in conjunction with a V/F curve to limit the power consumption of a processor.

If you set a positive clock offset while keeping the power limit in place, it will result in a higher clock with a lower voltage being used at the same power consumption, thus improving efficiency.

To undervolt a processor if you can customize the power limit, simply set a positive clock offset and reduce the power limit.

## Degradation

