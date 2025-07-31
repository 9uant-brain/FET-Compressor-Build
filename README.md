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


First, focus on IC2.1. Its output spilit into two path. One signal goes to audio out, another goes to sidechain. That signal strength determines basic intensity of compression. And, if you turn up the COMPRESS potentiometer, The IC's gain become stronger, so the singal compressed harder. Because ultimately, that signal voltage controls FET gate. 

As you follow down the path, could see there is splited path. One goes to inverting opamp, another straght to D2. These paths are quite felt weird first time. After much thought, I figured out why it is shaped like that. It would be much better explain it in diagram

