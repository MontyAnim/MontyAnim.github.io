---
title: "Stylized Water Shader"
date: 2025-11-22
draft: true
tech_stack: ["HLSL", "Unity", "Shader Graph"]
hero_video_id: ""
thumbnail_url: "/images/thumbnails/water-shader.gif"
summary: "Custom stylized water shader with animated waves, foam, and caustics for Unity games."
project_type: "visual"
tags: ["shaders", "unity", "vfx", "hlsl"]
---

## Overview

A performant stylized water shader designed for Unity games, featuring customizable waves, dynamic foam, and animated caustics. Optimized for mobile and PC platforms while maintaining visual quality.

## Visual Goals

The shader needed to:
- Match the game's stylized art direction
- Run smoothly on mobile devices (60 FPS+)
- Support various water types (ocean, rivers, ponds)
- Integrate seamlessly with the existing lighting system

## Technical Implementation

### Core Features

1. **Animated Wave System**
   - Gerstner waves for realistic movement
   - Customizable amplitude, frequency, and speed
   - Normal map blending for detail

2. **Dynamic Foam**
   - Depth-based foam at intersections
   - Animated foam texture scrolling
   - Adjustable foam spread and intensity

3. **Caustics**
   - Animated caustic patterns
   - Depth-faded for realism
   - Performance-optimized lookup textures

4. **Color Gradients**
   - Depth-based color transitions
   - Support for shallow/deep water colors
   - Transparency falloff

### Shader Graph Architecture

The shader uses Unity's Shader Graph with custom HLSL functions for performance-critical calculations:

```hlsl
void CalculateWaves_float(float3 worldPos, float time, 
                          out float3 offset, out float3 normal)
{
    float2 wave1 = GerstnerWave(worldPos.xz, _Wave1Dir, 
                                _Wave1Amplitude, _Wave1Frequency, time);
    float2 wave2 = GerstnerWave(worldPos.xz, _Wave2Dir, 
                                _Wave2Amplitude, _Wave2Frequency, time);
    
    offset = float3(wave1.x + wave2.x, wave1.y + wave2.y, 0);
    normal = CalculateWaveNormal(worldPos, offset);
}
```

## Performance Optimization

Key optimizations implemented:
- **LOD System:** Reduced wave complexity at distance
- **Texture Atlasing:** Combined foam and caustic textures
- **Shader Variants:** Separate quality tiers (Low/Medium/High)
- **Mobile Optimizations:** Simplified calculations for mobile GPUs

### Performance Metrics
- **PC (High):** 0.8ms per frame
- **PC (Medium):** 0.5ms per frame  
- **Mobile (Low):** 0.3ms per frame

## Visual Breakdown

### Layer 1: Base Color & Transparency
![Base layer showing depth-based color gradient](#)

### Layer 2: Wave Animation
![Gerstner waves creating surface movement](#)

### Layer 3: Foam & Edge Detection
![Foam appearing at intersections and wave peaks](#)

### Layer 4: Caustics
![Animated light patterns beneath the surface](#)

## Demo

Interactive shader demo (click and drag to rotate):

{{</* webgl demo="water-shader" title="Interactive Water Shader" height="600px" */>}}

## Source Code

Complete shader implementation:

{{</* gist MontyAnim YOUR_WATER_SHADER_GIST_ID */>}}

## Results

The shader has been integrated into production and is currently used in:
- Ocean scenes (12+ instances)
- River systems (procedurally placed)
- Environmental water features

**Performance Impact:** Less than 1% GPU time in typical scenes

## Artistic Control

Artists can adjust:
- Wave height, speed, and direction
- Foam thickness and spread
- Water clarity and color
- Caustic intensity and animation speed
- Reflection and refraction strength

## Lessons Learned

1. **Profile Early:** Mobile performance constraints shaped early design decisions
2. **Artist-Friendly:** Exposed parameters need clear names and logical ranges
3. **Visual Reference:** Having real water footage helped match the stylized look
4. **Iteration:** The shader went through 8 major versions before final approval

---

**Skills Demonstrated:** HLSL, Shader Graph, Unity, Performance Optimization, VFX