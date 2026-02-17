---
title: "A Start on Dante - A journey into the world of Digital AV"
date: 2026-02-17T23:43:05+01:00
draft: false
---

For over 15 years I have been using analog audio equipment and last month i made the move to Dante. Learning Dante has been a journey, and I wanted to share my experience with you. I will go through the setup, the issues I faced, and the good things about Dante.

<!--more-->

At home I have for many years used a condenser microphone, mixers and what not at my desk. I started doing that when i was kinda irritated at the quality of the built in microphones on headsets. I started out in 2012 with a RODE NT1-A and a Behringer Xenyx 1002B mixer.

Over the years I have upgraded the setup more and more. In 2019 I moved to a MOTU AVB Ultralite which i was in love with! It did everything in a small box and had great sound quality. Back then my requirements was:

- 2 microphone inputs
- 2 headphone outputs
- 4 line inputs
  - Windows Output 1 to the mixer: Voice chat (Discord, Teams, etc.)
  - Windows Output 2 to the mixer: Music (Spotify), Gaming etc.
- Built in Compressor. With a compressor i could hook it on the Microphone but even more important on the Voice chat output. This way i could set the compressor to make the voice chat sound more consistent and less noisy. Which means people on Discord cant yell your ears off, and others that are quiet can be heard better.

A few years later i moved my microphone to a Shure SM7B instead of the NT1-A. It is smaller in size and formed in another way that makes the microphone fill a lot less.

# So what now? 
I have in many years wanted to try Dante. I have been interested in it for a long time, but never had the need to try it out. It is a big investment to get into Dante, and I have been hesitant to spend that kind of money on something that I might not like. And it is hard to get started, because at the very end. Everything starts and ends in analog anyways.

*Spoiler alert*: A lot is working, some is... an issue

**What is Dante?**
Dante is a digital audio networking protocol developed by Audinate. It allows for the transmission of high-quality, low-latency audio over standard Ethernet networks. Dante is widely used in professional audio applications, such as live sound, recording studios, and broadcast environments. It enables the connection of multiple audio devices, such as microphones, mixers, and speakers, over a single network infrastructure, simplifying setup and reducing the need for traditional analog audio cables.

This is awesome for me, since i wanted to get rid of all the analog cables, the AV Rack that uses a lot of space. In the current installation i'm starting to question if i really get more space. 

You might be thinking: You are using words like: **"Professional Audio"**, **"live sound"**, **"recording studios"** and **"broadcast environments"** it sounds a bit overkill? That's because it is.

## The Setup
I looked into all the different Dante devices, a lot is recommending the Dante AVIO adapters. I found a australian company that creates their version of a lot of Dante stuff called Turtle AV. They have their version of the AVIO adapters. Many on Reddit recommended them, and they are even a bit cheaper than the AVIO adapters.

I bought the following devices:
- 1x TurtleAV Dual XLR Input for the microphones
  - Shure SM7B
- 1x TurtleAV Dual XLR Output for the headphones
- 1x TurtleAV USB-C Audio Interface for the computer
- M32C mixer (which has Dante built in)

My idea is that the M32C is the center of the setup. Connect the microphones to the Dual XLR inputs (Shure SM7B in my DBX286s and RODE GO). Connect the headphones to the Dual XLR outputs. Connect the computer to the USB-C Audio Interface. Then connect everything together with Ethernet cables and configure Dante to route the audio signals as needed.

My dream is that the M32C can be moved to a closet or something as it dosent really do anything other than be a DSP and only needs power and a network connection. 

So with the setup out of the way Nikolaj, this blog post is over right? Everything is working, and there is no issues, right? Well, not really - lets finish the setup and we can talk about the issues (spoiler alert) and the good things about Dante.

# Installation and Configuration
To use Dante you need a piece of software called Dante Controller. It is used to route the audio signals between the different devices. It is a bit of a learning curve to get used to it, but once you get the hang of it, it is pretty easy to use. I came from a UltraLite AVB which already had a routing matrix built in, so for me it was not that big of a jump.

The Dante Controller is where you route stuff, which means taking an input. Lets say the Turtle AV XLR Input 1. You can say that you want that to go to the M32C or even just the USB-C Adapter directly. You dont need a mixer per se.

But since i have the M32C all inputs come into the M32C as Input 1-10, and the outputs from the M32C go to the Turtle AV XLR Output and the USB-C Adapter. 

Everything is up and running and is working, but there are some issues that i have faced along the way.

# The Issues
In more or less order of importance:
- I like my SteelSeries Arctis Nova Pro which has 3.5mm Jack in and 3.5mm Jack out. I have been using it for years and it is really comfortable and has great sound quality. There is a lot of noise when the AVIO adapter is connected to the headphones, which weirdly enough isnt there in my BeyerDynamic DT 770 Pro headphones. I have tried different cables, different ports on the AVIO adapter, different settings in Dante, but nothing seems to fix the issue. 
  - I know that the SteelSeries is not really designed for professional audio. But for me its good for the long gaming sessions, the wireless connection and the comfort. I dont want to have to switch headphones all the time, and i dont want to have to use a different pair of headphones for gaming and music.
- I planned everything, made sure i bought everything to replace my current setup..... i just forgot to count the number of needed ethernet ports.
- All Dante devices has their own VLAN. I need to be on the same L2 to use the Dante Controller.
- Some of the Dante devices handle stuff differently. Turtle AV has on the device possiblities to turn up or down for the volume. NA-2I2O-DLINE just blasts 24dB out, and you cant do shit about it. That makes a lot of noise in my SteelSeries DAC as that is waaaay to much for it.
- Getting to learn the M32C was a journey. MOTU and their AVB UltraLite was a lot more UX Friendly.
  - I miss the Webinterface and not a software on a computer..

# The Good Things
- The sound quality is amazing. The latency is very low, and the flexibility of routing audio signals is fantastic. I can easily switch between different audio sources and destinations without having to unplug and replug cables.
- The M32C is a great mixer with a lot of features. It has a lot of built in effects and processing options, and the Dante integration works well.

# Can I use this?
I think Dante can be used in a lot of scenarios without going completely overkill. You can have fx have the following setup:
- AVIO Dual XLR input PoE+
- AVIO Dual XLR output PoE+

And with those to extend a analog line between different rooms using existing networking etc. I think that is really cool, since you can have a lot of flexibility in how you route the audio signals. You can have a microphone in one room, and have it go to a speaker in another room without having to run long analog cables. You can also have a lot of different audio sources and destinations, and route them as needed. 