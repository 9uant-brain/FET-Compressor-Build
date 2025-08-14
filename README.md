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
This is last section of sidechain. It is no exaggeration to say that the whole sidechian is for these JFETs. Because the JFET directly adjust audio signal. Q1 is the one adjusting the signal, but I will explain about Q2 first. 

Q2 is looked like less important than Q1, as Q2 doesn't control the signal directly. However, Q2 pave the way to enable Q1 controls the signal precisely, because Q2 stablize Q1's source voltage. 

When it comes to voltage stablization, Q2 and also C7 are important. C7 absorbs Q1's source AC voltage, enable Vg directly control Q1's channel. But, that capacitor has high impedence at lower frequencies. Therefore, C7 can't handle low frequency fluctuation well, at this moment, Q2 comes into play. Here is how it works.

In its normal state, a current (Id) constantly flows through Q2. Since Q2 operates in self-bias mode, Vs varies continuously as Id changes — per Ohm's law (V = IR), Id flowing through R13 generates Vs. This Vs determines Vgs, which controls how open the channel is. The channel openness, in turn, affects Id. Let's refer to Q2’s source node as 'Node B'.

When peak signal came into, larger current flow out from Q1's channel, then it rise Vs of Q1(node A). Also, node A is Q2's gate. Then, instantly Q2's Vgs goes over 0, which means, the gate become forward bias, a current can flow through Q1's gate. It is samething happen in forward biased diode. And, let's draw whole picture of it. 
Initially, Q1 Id instantly more flows.
