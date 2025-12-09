+++
author = "Xavier Amado"
title = "Spark: Atmospheric Scattering"
date = "2025-12-08"
description = "Atmospheric Scattering implementation"
tags = [
    "spark",
    "rendering",
    "graphics"
]
categories = [
    "spark",
]
series = ["Spark"]
+++

Last weekend I decided to spend some time implementing an Atmospheric Scattering component in Spark, based on "A Scalable and Production Ready Sky and Atmosphere Rendering Technique" by Sébastien Hillaire (https://sebh.github.io/publications/egsr2020.pdf). 

<!--more-->

The paper proposes a technique evolved from Bruneton's 2014 paper, making it faster and solving some edge cases. His implementation is basically what was implemented in UE5.

The technique works by raymarching the atmosphere in incremental steps, evaluating the medium at each step and generating several LUTs for later lookup. Transmittance, Multiscattering (for simulating multi-scattering effects, added realism), Skyview (used later to render the actual sky) and a 3D one generated from the camera's perspective to get an "idea" of how thick atmosphere is, this is effectively the "horizon fog".

To generate these LUTs I've used compute shaders for everything, and even though I'm currently doing it every frame, these should really be recomputed only if the atmosphere changes significantly, for example by modifying a parameter in the editor, or a night-day transition system.

I must say I'm quite pleased with the result, but I still have some work to do.

{{< youtube cZC9dSQ6vlU >}}

Next steps? Well you will notice that the sun becomes a huge blob, this is due to the fact that I never paid much attention to real photometric light units, and the sun is approximately 100000 lux (compared to the silly intensity of "2" that I'm using for the directional light, IBL, etc.).

This makes the bloom post process go crazy. Also my tonemapping and auto-exposure is a bit of a joke. So this all needs to be reworked.

So I'm going to be working on implementing a real pre-exposure (EV100) and get some real physically based camera settings and physically based light units, plus making sure all my luminance is calculated properly.

