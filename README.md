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

\\\\Maybe your first impression was 'this is quite complicated than RAT2.'. Because, essentially sidechain is detection and self feedback logic. I will explain it briefly.  


First, focus on IC2.1. Its output spilit into two path. One signal goes to audio out, another goes to sidechain. That signal strength determines basic intensity of compression. And, if you turn up the COMPRESS potentiometer, The IC's gain become stronger, so the singal compressed harder. Because ultimately, that signal voltage controls FET gate. If this voltage gets higher, the FET's channel is opened wider as Vgs gets higher(getting closer to 0). This make audio signal weaker, because channel connects to GND.

As you follow down the path, could see there is splited path. One goes to inverting opamp, another straght to D2. These paths are quite felt weird first time. After much thought, I figured out why it is shaped like that. It would be much better explained in diagram.

### Rectifying, Detecting

<p align='center'>
 <img src=asset/waveform.jpg>
</p>

Let's assume audio input is sinewave. Then sinewave also goes to sidechain path. That's waveform A. A inverted through inverting opamp, that's B. If B passed through diode, become C. As diodes allow to pass voltages above Voltage Forward, the singals below than VF are cut. Same thing happens on D, difference is A has been cut down instead of B. C and D merged into E because they meet at the node. And that signal goes to FET gate. 

It works as kind of retifier. But, why this structure is needed? First, lets assume that we directly control fet gate with waveform A. It will compress(open channel) only upper side of singal. Because JFET open its channel only when Vgs getting lower than 0. As lower side means minus voltage, jfet will close the channel rather than open it. So, it's essential to convert waveform from A to E. 

Also, BAT43 diodes are used, not only for retifying but because they have lower VF. If VF was too high, most of signal gonna be cut. And FET only works when high signals come in.

### Time domian.

<p align='center'>
 <img src=asset/attack.jpg>
</p>  

Look at the scheamtic. I hightlighted some components. They are essential. Now, let's talk about time. Because 'attack','release' potentiometers are all about time. Technically, 'time constant'. 
We discussed about how FET gate voltage is sourced, transformed. In this section, we discuss about how fast that voltage charged, discharged. 

<p align='center'>
 <img src=asset/constant.jpg>
</p>  
To make it clearer, I simplifed previous schematic. Resistance 'ATTACK' determines how fast charge C6. Resistance 'RELEASE' determines how fast discharge C6. If C6 was charged fast, quicker FET compresses signal.Because  These are core parameters of compressor. B
