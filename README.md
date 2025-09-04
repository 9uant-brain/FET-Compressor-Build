## Why I made this
After finishing my RAT2 project, I built and even bought a few more pedals, but I still didn’t have a compressor pedal. The reason was simple: compressors tend to be deprioritized because they don’t reshape the sound dramatically — they mainly tame loud peaks and add a touch of sustain… or so I thought.

When I went deep into Jimi Hendrix’s recordings, I realized I needed a compressor. Even though he wasn’t known for using a compressor pedal, I thought it would help me imitate his sound more easily.

I could have just bought a low-cost pedal, but I decided to build my own instead. There were several reasons for that choice.

First, I wanted independent attack and release controls to get a truly controlled tone — and among budget pedals, that exact feature set is uncommon.

Second, I wanted a compressor versatile enough for both guitar and bass. For that, a dry/wet mix (parallel compression) is extremely useful. On bass, loud low frequencies can make a compressor clamp down so hard that the overall volume briefly dips after a strong note and then swells back — an audible “breathing” effect often called pumping. Blending in some dry signal keeps the original attack and reduces that dip-and-swell.

For these reasons —mainly cost-effectiveness and control over the feature set— I decided to build a compressor myself. I searched the internet for reference schematics and ultimately chose a design from PedalPCB as my starting point.


## Circuit overview


I like to describe a compressor this way: Imagine a helper sitting by the volume knob, watching the audio waveform just before it's played. If the signal gets too loud — above a certain threshold — he quickly turns the volume down.

So, I can explain this schematic same way. I highlighted that section which is called the ‘sidechain’; I likened it to a ‘helper’. It's the core section of compressors; you could even call it a compressor itself. I will focus on it in this chapter. And, Let’s look at the schematic below.

<p align='center'>
 <img src=asset/sch.jpg>
</p>

Maybe your first impression was, "This is quite a bit more complicated than the RAT2." And that's understandable—after all, the sidechain involves both detection and a self-feedback mechanism. Let me explain it briefly.

First, focus on IC2.1. Its output splits into two paths: one goes to the audio output, while the other feeds the sidechain. 
The level of this sidechain signal sets the basic compression intensity—in other words, a stronger sidechain signal allows quieter audio signals to be compressed more. When you turn up the COMPRESS potentiometer, the op-amp’s gain increases, so the signal becomes more compressed. This is because the signal ultimately controls the FET gate. As the voltage rises, the FET’s channel opens wider (Vgs moves closer to 0 from a negative value), which weakens the audio signal—since the channel connects to ground.

As you follow the path, you'll notice a split: one line goes to an inverting op-amp, the other directly to D2. At first, this structure might seem confusing. But after some thought, I figured out why it’s designed this way. It would be much easier to explain with a diagram.

### Rectification and Detection

<p align='center'>
 <img src=asset/waveform.jpg width="80%" height="80%">
</p>

Let's assume the audio input is a sine wave. That sine wave also travels through the sidechain path — we'll call that waveform A. A is then inverted through an inverting op-amp, becoming waveform B. When B passes through a diode, it becomes waveform C, since diodes only allow voltages above their forward voltage (VF) to pass — anything below VF is clipped.

The same thing happens with waveform D, except this time it's waveform A (not B) that's being clipped. Then, C and D are merged into waveform E at a common node. This final waveform E is what goes to the JFET gate.

In effect, this functions as a kind of rectifier. But why is this structure necessary?

Imagine we directly control the JFET gate with waveform A. In that case, only the upper half of the wave would cause compression (i.e., open the JFET channel). That’s because a JFET opens wider when Vgs gets closer to 0. Since the lower half of the sine wave is already negative, it would actually close the channel rather than open it. That’s why converting waveform A into waveform E is essential — it ensures the gate receives a proper, unipolar control signal.

Also, BAT43 diodes are used not only for rectification, but because they have a low forward voltage. If VF were too high, much of the signal would be lost — and the JFET only responds to relatively high gate signals.

I also measured the rectified output to observe its actual shape — the captured waveform is shown below.

<p align='center'>
 <img src=asset/waveform2.jpg width="30%" height="30%">
</p>

### Time domain

<p align='center'>
 <img src=asset/attack.jpg width="14%" height="14%">
</p>  

Look at the schematic — I’ve highlighted some essential components.
Now, let’s talk about time, because the ‘attack’ and ‘release’ potentiometers control exactly that — technically, the time constant.

Earlier, we discussed how the JFET gate voltage is sourced and transformed.
In this section, we'll focus on how quickly that voltage is charged and discharged.

<p align='center'>
 <img src=asset/constant.png width="60%" height="60%"> 
</p>  
To make it clearer, I simplified the previous schematic. The 'ATTACK' resistance determines how fast C6 charges, and the 'RELEASE' resistance determines how fast C6 discharges. If C6 charges quickly, the JFET reacts faster and compresses the signal more quickly.

These are the core timing parameters of a compressor. For example, attack time refers to how fast the circuit responds to a transient spike, while release time describes how quickly the circuit returns to normal after that spike.

Here’s a sonic example. The attack setting controls how much of the initial drum transient is allowed to pass through. The release setting determines how quickly the volume returns to normal. If the release is too fast, it can sound unnatural because the volume may rise too quickly before the drum hit ends. On the other hand, if the release is too slow, it can reduce the impact of the next drum hit by not fully recovering in time.

### JFET, the signal controller
<p align='center'>
 <img src=asset/fet.jpg width="14%" height="14%">
</p>  
This is the final section of the sidechain. It's no exaggeration to say that the entire sidechain circuit exists to serve these JFETs — since they directly control the audio signal. While Q1 is the one actually adjusting the signal, we'll start with Q2.

Q2 may seem less important than Q1, as it doesn't directly touch the signal path. However, it plays a critical supporting role by stabilizing Q1’s source voltage, which enables more accurate control.

When it comes to voltage stabilization, both Q2 and capacitor C7 are involved. C7 absorbs AC fluctuations at Q1’s source, helping the gate voltage (Vg) directly control the channel conductance. But C7 has high impedance at low frequencies and struggles to suppress slow fluctuations. That’s where Q2 comes into play.

Under normal conditions, a constant drain current (Id) flows through Q2. Since Q2 operates in a self-bias configuration, its source voltage (Vs) varies dynamically as Id changes — according to Ohm’s Law (V = IR). Current through R13 generates a voltage drop that defines Vs, which determines Vgs, which in turn controls the JFET channel conduction. In simple terms, Q2 acts as a current source. 
To make the following explanation clearer, I will refer to Q2’s source node as “Node B.”

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
Most of this schematic is identical to the original. I only made a few minor edits — changing some resistor values to match the parts I had in stock and updating the op-amp numbering.
I also added subboard designs for the audio jacks and footswitch.
Initially, I followed the same op-amp numbering as in the original, but that changed during PCB layout work. I’ll go into more detail about that later.

### PCB layout Creation 

This circuit has a lot of components, and since I didn’t want to use a larger enclosure, I decided to design it using SMD components. I was also curious to see whether I could solder SMD parts neatly.
I’ll first explain the older design, and then go over the revised version.


<p align='center'>
 <img src=asset/PCB1.png width="70%" height="70%">
</p>  

This was an abandoned design — I couldn’t complete it cleanly because I failed to connect all the pads neatly.

So, I started from scratch and approached the layout more strategically. I identified the key issues in the previous version and focused on resolving them. During that process, I also took into account the relative placement of components in the schematic.

First, I mistakenly placed SOIC packages on both sides of the board. I initially thought this would help reduce track congestion, but it actually made things worse. I had to frequently switch tracks between the top and bottom layers, which complicated routing even more.

Second, I made a mistake when placing the potentiometers. I focused too much on user-friendly knob positioning without considering signal flow or layout efficiency. For example, the wet (blend) knob should ideally be located near the audio input and positioned after the sidechain. But I didn’t realize that at first, and instead placed it at the top of the board for ergonomic reasons (2nd row: Wet, Tone, Volume).

----

<p align='center'>
 <img src=asset/PCB2.png width="70%" height="70%">
</p>  
This is the revised final design, now including a sub-board. Here’s what I changed:

1. I rotated some potentiometers to free up more space. In the previous design, their pins occupied the central area and interfered with routing.
2. I moved several components from the input section to the sub-board, which helped reduce congestion and secure more board space.
3. I changed the op-amp numbering. While positioning and tracing the layout, I realized that renumbering the op-amps would shorten trace lengths and make the routing cleaner.
4. I rearranged the potentiometer layout to strike a better balance between user-friendly knob placement and efficient PCB connectivity.


I learned an important lesson while revising this design: sometimes, you have to compromise between the ideal and reality. Specifically, I had to sacrifice the ideal, user-friendly positioning of the potentiometers (knobs) in favor of cleaner and more efficient trace routing.

At first glance, when I looked at the original pedal —the Ego76— I assumed they simply hadn’t prioritized user-friendly knob placement. But as I revised my own layout, my perspective changed. I realized it’s more likely that they made intentional trade-offs, just like I did.

In the image below, I’ve compared the knob positions between my design and the original pedal. Despite not referencing the original layout during my revision—mainly because my design uses a different row structure (3×2 vs. 2×3)—the final placements turned out to be quite similar. This suggests that shared physical constraints and ergonomic considerations can naturally lead to converging design choices, even when developed independently.

<p align='center'>
 <img src=asset/PCB3.jpg width="70%" height="70%">
</p>  

## Assembly
As usual, I ordered the PCB, soldered it, drilled the enclosure, and mounted the board inside. One notable difference this time was that I soldered some SMD parts for the first time. Rather than giving a long explanation, I’ll just show the pictures of the assembly process.

<p align='center'>
 <img src=asset/ssy1.jpg width="30%" height="30%">
</p>  

<p align='center'>
 <img src=asset/ssy2.jpg width="30%" height="30%">
</p>  

<p align='center'>
 <img src=asset/ssy3.jpg width="30%" height="30%">
</p>  

<p align='center'>
 <img src=asset/ssy4.jpg width="30%" height="30%">
</p>  

<p align='center'>
 <img src=asset/ssy5.jpg width="30%" height="30%">
</p>  

<p align='center'>
 <img src=asset/ssy6.png width="30%" height="30%">
</p>  

<p align='center'>
 <img src=asset/ssy7.png width="30%" height="30%">
</p>  

## Debugging, because most things won’t work at first

After finishing soldering, I immediately powered on the amp and plugged this thing in. It sounded fine at first, but very quiet—especially when I turned the wet knob all the way up. That suggested the compressed signal was being over-compressed, to the point where it was barely audible.

At first, I suspected a damaged MLCC decoupling capacitor leaking to ground, so I desoldered it. Indeed, the MLCC was causing some leakage, but even after removing it, the issue remained. I then noticed that only when the JFET was removed did the signal stop getting compressed. This meant that both the MLCC and the JFET were contributing to the problem.

Digging deeper into the JFET behavior, I found that the devices I used had lower Idss than expected. With such low Idss, the JFET couldn’t generate enough source voltage, so its channel wouldn’t fully close. As a result, the audio signal kept bleeding into ground.

To fix this, I tried replacing the trimpot from 10 kΩ to 50 kΩ, so the voltage could be pulled up higher. But it didn’t work—because the Idss was simply too low, I had to raise the trimpot to a much higher resistance (around 30–40 kΩ). At that point, the unwanted compression (signal flowing into ground) stopped, but as a side effect, the current hardly flowed at all, so compression itself no longer occurred.

I ordered more JFETs hoping to find ones with proper Idss, but that batch also turned out weak. Fortunately, I eventually found a suitable JFET in my inventory with sufficient Idss. Since it was a different model, its Vgs(off) was different, which resulted in a different compression intensity. Still, when I tried it, it worked quite well.

## Final Product and Demonstration
Here is the final product. I painted the enclosure and labeled the parameters using a waterslide decal that I printed myself. I also attached silver knobs. The photo of the finished unit is shown below.

<p align='center'>
 <img src=asset/FP.jpg width="50%" height="50%">
</p>  

---

Now, for the demonstration. For convenience, I used a saw wave as the input signal, since it is easy to observe variations. The test signal is 2 Vpp at 500 Hz, which is similar to a raw guitar signal. The demo video link is posted below.

[![Video Label](http://img.youtube.com/vi/7YFwYgtLydY/0.jpg)](https://youtu.be/7YFwYgtLydY)

In this video, you might have noticed how the parameters affect the signal, but here I will explain them more specifically.

``Attack`` controls how quickly the circuit reacts to the initial peak. In this case, a slower attack (knob turned up) appears to cause less compression, because the slower response delays the action of compression on the peak.

``Release`` controls how long the compression is maintained. In this example, a shorter release (knob turned down) results in less overall compression, because the gain reduction vanishes quickly and does not affect the following period.

``Compress`` (knob) behaves in a non-intuitive way: turning it up seems to result in less compression. In fact, it functions more like a threshold control. Increasing the compress setting allows more of the sidechain signal to pass through the diodes, which means lower-level audio is also affected. In simple terms, compress and threshold are inversely proportional.

Why does it seem that a higher compress knob setting causes less compression? Because once a signal level is above a certain threshold, it is compressed in proportion to its amplitude. Earlier I briefly explained how the threshold is defined. Therefore, if 90% of a signal is above the threshold, that 90% retains the same relative differences as in the original (non-compressed) signal.

## Conclusion
Even though there was no modification (unlike the RAT2 project), completing the final work was difficult. In particular, this point was challenging: since this is essentially a compressor that doesn’t completely reshape the sound, it was hard to figure out whether it was working properly and what the problems were. Even when it wasn’t working correctly, it still produced a decent sound—but not a properly compressed one, which is difficult to distinguish by ear alone. However, after overcoming those issues, I was able to fully explain how the sidechain and JFET operate. Most importantly, I learned once again that I can overcome difficulties as long as I don’t give up.

### **Thanks for reading this far!** 
