## Why I made this
After finishing my RAT2 project, I built and even bought a few more pedals. But I still didn’t have a compressor pedal. The reason was simple: compressors tend to be deprioritized because they don’t shape the sound dramatically. They mainly tame loud peaks while slightly boosting quieter parts—or so I thought.

However, when I got hooked on Jimi Hendrix’s music, I realized I needed a compressor. His playing focuses on a sophisticated yet raw electric guitar tone, and compression plays a key role in achieving that.

I could have just bought a cheap Chinese pedal, but I decided to build my own instead. There were several reasons for that choice.

First, I wanted a compressor with attack and release controls to get a truly clean guitar tone—but even China-made pedals with these features tend to be quite expensive.

Second, I wanted a compressor versatile enough for both guitar and bass. For that, a dry/wet mix (also known as parallel compression) was essential. Bass guitars typically have very strong signals, and if a compressor isn’t designed specifically to handle them, it can cause a “pumping” effect. This happens when the compressor clamps down too aggressively on the signal and then releases, creating an audible swell or “pump.” By blending in some dry signal with the compressed tone, this effect can be reduced or avoided.

For these reasons (mostly due to cost-effectiveness), I decided to build a compressor myself. Then, I searched the internet for reference schematics and finally found one on 'pedalpcb.com'.


## Circuit overview


I like to describe a compressor this way: Imagine a helper sitting by the volume knob, watching the audio waveform just before it's played. If the signal gets too loud — above a certain threshold — he quickly turns the volume down.

So, I can explain this schematic same way. I highlighted that section which is called as 'sidechain', I metaphored it as 'a helper'. It's the core section of compressors; you could even call it a compressor itself. I will focus on it in this chapter. And, let's look into schematic below.

<p align='center'>
 <img src=asset/sch.jpg>
</p>

Maybe your first impression was, "This is quite a bit more complicated than the RAT2." And that's understandable—after all, the sidechain involves both detection and a self-feedback mechanism. Let me explain it briefly.

First, focus on IC2.1. Its output splits into two paths: one goes to the audio output, and the other feeds into the sidechain. The strength of this signal determines the basic compression intensity. When you turn up the COMPRESS potentiometer, the op-amp’s gain increases, so the signal becomes more compressed. This is because the signal ultimately controls the FET gate. As the voltage rises, the FET’s channel opens wider (Vgs moves closer to 0 from a negative value), which weakens the audio signal—since the channel connects to ground.

As you follow the path, you'll notice a split: one line goes to an inverting op-amp, the other directly to D2. At first, this structure might seem confusing. But after some thought, I figured out why it’s designed this way. It would be much easier to explain with a diagram.

### Rectifying, Detecting

<p align='center'>
 <img src=asset/waveform.jpg width="80%" height="80%">
</p>

Let's assume the audio input is a sine wave. That sine wave also travels through the sidechain path — we'll call that waveform A. A is then inverted through an inverting op-amp, becoming waveform B. When B passes through a diode, it becomes waveform C, since diodes only allow voltages above their forward voltage (VF) to pass — anything below VF is clipped.

The same thing happens with waveform D, except this time it's waveform A (not B) that's being clipped. Then, C and D are merged into waveform E at a common node. This final waveform E is what goes to the FET gate.

In effect, this functions as a kind of rectifier. But why is this structure necessary?

Imagine we directly control the FET gate with waveform A. In that case, only the upper half of the wave would cause compression (i.e., open the FET channel). That’s because a JFET opens wider when Vgs gets closer to 0. Since the lower half of the sine wave is already negative, it would actually close the channel rather than open it. That’s why converting waveform A into waveform E is essential — it ensures the gate receives a proper, unipolar control signal.

Also, BAT43 diodes are used not only for rectification, but because they have a low forward voltage. If VF were too high, much of the signal would be lost — and the FET only responds to relatively high gate signals.

Also I measured the rectified output to observe its actual shape — the captured waveform is shown below.

<p align='center'>
 <img src=asset/waveform2.jpg width="30%" height="30%">
</p>

### Time domain

<p align='center'>
 <img src=asset/attack.jpg width="14%" height="14%">
</p>  

Look at the schematic — I’ve highlighted some essential components.
Now, let’s talk about time, because the ‘attack’ and ‘release’ potentiometers control exactly that — technically, the time constant.

Earlier, we discussed how the FET gate voltage is sourced and transformed.
In this section, we'll focus on how quickly that voltage is charged and discharged.

<p align='center'>
 <img src=asset/constant.png width="60%" height="60%"> 
</p>  
To make it clearer, I simplified the previous schematic. The 'ATTACK' resistance determines how fast C6 charges, and the 'RELEASE' resistance determines how fast C6 discharges. If C6 charges quickly, the FET reacts faster and compresses the signal more quickly.

These are the core timing parameters of a compressor. For example, attack time refers to how fast the circuit responds to a transient spike, while release time describes how quickly the circuit returns to normal after that spike.

Here’s a sonic example. The attack setting controls how much of the initial drum transient is allowed to pass through. The release setting determines how quickly the volume returns to normal. If the release is too fast, it can sound unnatural because the volume may rise too quickly before the drum hit ends. On the other hand, if the release is too slow, it can reduce the impact of the next drum hit by not fully recovering in time.

### JFET, the signal controller
<p align='center'>
 <img src=asset/fet.jpg width="14%" height="14%">
</p>  
This is the final section of the sidechain. It's no exaggeration to say that the entire sidechain circuit exists to serve these JFETs — since they directly control the audio signal. While Q1 is the one actually adjusting the signal, we'll start with Q2.

Q2 may seem less important than Q1, as it doesn't directly touch the signal path. However, it plays a critical supporting role by stabilizing Q1’s source voltage, which enables more accurate control.

When it comes to voltage stabilization, both Q2 and capacitor C7 are involved. C7 absorbs AC fluctuations at Q1’s source, helping the gate voltage (Vg) directly control the channel conductance. But C7 has high impedance at low frequencies and struggles to suppress slow fluctuations. That’s where Q2 comes into play.

Under normal conditions, a constant drain current (Id) flows through Q2. Since Q2 operates in a self-bias configuration, its source voltage (Vs) varies dynamically as Id changes — per Ohm's Law (V = IR), current through R13 generates a voltage drop that defines Vs. Vs determines Vgs, which in turn governs how open the JFET channel is. For clarity, let’s call Q2’s source node “Node B.”

Now let’s walk through what happens during a signal peak.

When a transient spike hits, Q1’s channel conducts more current (Id), causing its source voltage (Node A) to rise. Node A also happens to be Q2’s gate. So, when Node A rises, Q2’s gate-to-source voltage (Vgs) briefly becomes positive — meaning the gate is forward-biased, much like a forward-biased diode. This allows current to flow into the gate, which then travels through Q2’s channel.

This has a subtle but important effect:
Q1’s Id is effectively shared with Q2, and as current flows into Q2’s channel, a portion of Node A’s voltage is transferred to Node B (again, V = IR). This doesn’t just “share” voltage — it actively raises Q2’s source voltage (Vs). As Vs increases, Q2’s Vgs drops (becomes less negative), narrowing the channel and reducing Id. This decrease in Id means less current flows through TR1, leading to additional voltage drop at Node A.

So why is this mechanism needed if we already have a sidechain controlling the JFET gates?

Because this feedback path reacts faster than the main sidechain. The sidechain signal must pass through multiple stages — op-amps, rectifiers, diodes — introducing latency. In contrast, this gate injection feedback loop responds immediately to transient spikes, suppressing them before the main sidechain even kicks in.
## Design and Implementation
### Schematic Creation
<p align='center'>
 <img src=asset/FET_SCH.png width="70%" height="70%">
</p>  
Mostly, this schematic is same as original schematic. I just edited some parts, slightly changed resistors for fitting my inventory, changed opamp numbers. And, some subboard designs with audio jacks, footswtich. At first, I followed same opamp number as orignal one, it was edited while I designing PCB layout. I will discuss about it later. 
### PCB layout Creation 
This circuit have lots of components, and I don't want to use larger enclousure, I decided to design it with SMD component. Also I wondered I could solder neatly SMD components. 
<p align='center'>
 <img src=asset/PCB2.png width="70%" height="70%">
</p>  
This was abandoned design. 
