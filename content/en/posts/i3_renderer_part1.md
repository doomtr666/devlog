---
title: "i3 Renderer (Part 1): IBL"
date: 2026-04-14
tags: ["3D Engine", "IBL", "Rendering", "Vulkan", "RayQuery", "Physically Based Rendering"]
categories: ["Dev Diary", "i3"]
draft: false
---

In my [previous article](../i3-frame-graph/), we dived into the logistical depths of the **[i3](https://github.com/doomtr666/i3)** **Frame Graph**. It's the foundation—the plumbing that allows for rendering orchestration without pulling your hair out over Vulkan synchronization barriers. But once the pipes are laid, you actually have to start drawing things on the screen.

Today, we're tackling a great classic of modern rendering: **Image-Based Lighting (IBL)**.

<!--more-->

### Ambition vs. Strategy: Incremental Rendering

I’ll be honest: I have a certain ambition for the final rendering in i3. I want things that pop, consistent reflections, and truly physical light management. But you don't birth a rendering engine in an afternoon, all by yourself in your garage.

My strategy is as follows: implement a collection of **composable** techniques. Currently, the engine's core is a classic **Clustered Deferred** pipeline (G-Buffers, Deferred Resolve, Tonemap). The only real "frill" so far has been **Normal Maps**, but without global illumination or reflections, the rendering still felt very "early 2000s."

We’re going to improve this progressively. No un-debuggable monolithic "Uber-Shaders," but bricks that we stack. And the first brick for ambient lighting is IBL.

### A Memory Lane: Split Sum Approximation

Revisiting IBL is a true trip down memory lane for me. I coded a version of this algorithm about 10 years ago for the **InsanityEngine**, the engine behind my game [Win That War!](https://store.steampowered.com/app/599040/Win_That_War/) released on Steam. It was the era of indie pioneers, and fighting with light integrals was already a national sport.

To start gently improving this rendering, I’m building on the "Split Sum Approximation" model by **Brian Karis** from Epic Games, which has now become the industry standard. His original SIGGRAPH 2013 course note, [**"Real Shading in Unreal Engine 4"**](https://cdn2.unrealengine.com/Resources/files/2013SiggraphPresentationsNotes-26915738.pdf), remains the key reference on the subject. If you're looking for more introductory basics, I highly recommend the excellent LearnOpenGL articles on [Diffuse Irradiance](https://learnopengl.com/PBR/IBL/Diffuse-irradiance) and [Specular IBL](https://learnopengl.com/PBR/IBL/Specular-IBL).

It’s efficient and robust, and it’s going to add some depth to the final result. But as usual, I wanted to add my own "touch" and a few technical specifics to make it more modern and tailored for i3.

<div style="text-align: center; margin: 2rem 0;">
  <img src="/images/i3_renderer_part1/hdri.png" alt="HDRI Source Poly Haven" style="max-width: 80%; border-radius: 8px;" />
  <p style="font-style: italic; font-size: 0.9rem; color: #666;">
    The source HDRI, from <a href="https://polyhaven.com/hdris">Poly Haven</a>.
  </p>
</div>

### Goodbye Cubemaps, Hello BC6H

Traditionally, IBL uses **Cubemaps**. They are convenient and widely supported, but they are also a bit of a pain to manage (6 faces, seam management, etc.).

In i3, we start with a classic **Equirectangular HDRI** (.hdr or .exr) file. But instead of converting it to a Cubemap, we keep it as is and compress it into **BC6H**.

This hardware compression format is dedicated to HDR: unlike traditional formats like BC1 or BC3 (which are limited to LDR), BC6H natively supports **16-bit float** data. This allows for preserving a huge dynamic range (essential for the sun) while reducing the texture size by 4 to 6 times.

No complex conversion, no losses due to re-sampling into 6 faces. We sample the equirect directly.

### The Dual-Octahedral Mapping

This is where things get interesting from a technical perspective. This approach comes directly from G-Buffer optimization and normal compression, popularized notably by work on Real-Time Global Illumination and engines like **Frostbite** or **Unity (HDRP)**.

#### Why abandon classic Octahedral?
The basic principle of Octahedral Mapping was formalized by Meyer et al. (2010) and then benchmarked by Cigolle et al. (2014) ([*"A Survey of Efficient Representations for Independent Unit Vectors"*](https://jcgt.org/published/0003/02/01/)).

The problem with a standard octahedral projection on a single map is **sampling discontinuity**. When using hardware bilinear filtering or mipmapping, the edges and diagonals of the unfolded octahedron do not physically correspond in texture space. This creates filtering artifacts that are impossible to correct cleanly without complex "border gutters."

#### The "DICE Trick": Dual-Octahedral
The trick (used in Frostbite) consists of using a **Hemi-Octahedral** encoding. We split the sphere into two hemispheres (North/South) and treat them separately. By placing these two projections side-by-side, we double the pixel density for the useful area and ensure a much cleaner hardware interpolation.

Here is how i3 encodes this in its baker (Rust) :

```rust
/// Encode direction sphere → Dual-Octahedral UV [0,1]².
pub fn hemi_octa_encode(d: [f32; 3]) -> (f32, f32) {
    let [x, y, z] = d;
    let l = x.abs() + y.abs() + z.abs();
    let ox = x / l;
    let oz = z / l;
    let u = ox * 0.5 + 0.5;

    // We split the V range in two: North [0, 0.5] and South [0.5, 1.0]
    let v_local = oz * 0.5 + 0.5;
    let v = if y >= 0.0 { v_local * 0.5 } else { 0.5 + v_local * 0.5 };
    (u, v)
}
```

#### Practical Advantages:
1.  **Uniformity**: Unlike a Lat-Long (Equirectangular) map where pixels pile up at the poles, here every pixel covers roughly the same solid angle.
2.  **Density**: By splitting it in two, we maximize texture square occupancy.
3.  **Speed**: The wrap/unfold functions are extremely lightweight in terms of shader instructions (just a few `abs` and `sign`).

<div style="text-align: center; margin: 2rem 0;">
  <div style="display: flex; gap: 10px; justify-content: center; align-items: flex-start;">
    <div style="width: 30%;">
      <img src="/images/i3_renderer_part1/diffuse.png" alt="Diffuse Irradiance Map" style="width: 100%; border-radius: 4px;" />
      <p style="font-size: 0.8rem;">Diffuse Irradiance</p>
    </div>
    <div style="width: 30%;">
      <img src="/images/i3_renderer_part1/specular.png" alt="Specular Map" style="width: 100%; border-radius: 4px;" />
      <p style="font-size: 0.8rem;">Prefiltered Specular</p>
    </div>
    <div style="width: 30%;">
      <img src="/images/i3_renderer_part1/lut.png" alt="BRDF LUT" style="width: 100%; border-radius: 4px;" />
      <p style="font-size: 0.8rem;">BRDF LUT</p>
    </div>
  </div>
  <p style="font-style: italic; font-size: 0.9rem; color: #666; margin-top: 10px;">
    The three maps resulting from the i3 bake: Irradiance (Dual-Octa), Prefiltered (Dual-Octa + Mips), and the traditional BRDF LUT.
  </p>
</div>

### Energy Balancing & Sun Extraction

The big problem with "raw" IBL is that it contains everything: the sky, the clouds, but also the **sun** (that massive "bright spot"). If you use IBL as is, you get correct ambient lighting, but you lose the sharp cast shadows and the control over the direction of the main light source.

In i3, I’ve developed an **Energy Balancing** algorithm during the bake (via `i3_baker`).

The baker scans the HDR to detect the main light source (the sun). Once detected, we set a threshold and "extract" the solar energy from the IBL. This allows combining IBL with a true directional light (Sun Light) in the engine without doubling the light energy, while maintaining the possibility of casting shadows.

This ensures that the sun's direct contribution is always balanced against the occluded ambient irradiance. The result: perfect consistency with the original HDR environment.

### Shadows at Last! (via RayQuery)

Since we’ve extracted the sun from the IBL to turn it into a classic directional light, we can finally have cast shadows.

For now, i3 skips classic Shadow Maps and directly uses **Ray Tracing** via **RayQuery**. It is worth noting here that the engine automatically synchronizes acceleration structures for native RT support. There is no fallback for non-RT hardware yet, but this keeps the pipeline clean and modern from the start. During the **Deferred Resolve** phase, we shoot a ray toward the sun for every visible pixel. It’s clean, avoids resolution artifacts, and leverages the RT (Ray Tracing) capabilities of modern cards.

<div style="text-align: center; margin: 2rem 0;">
  <img src="/images/i3_renderer_part1/i3_sponza.png" alt="i3 Sponza Render" style="max-width: 90%; border-radius: 8px;" />
  <p style="font-style: italic; font-size: 0.8rem; color: #666;">
    First example: the Sponza scene, from the <a href="https://github.com/KhronosGroup/glTF-Sample-Assets/tree/main">glTF Sample Assets</a>.
  </p>
</div>

<div style="text-align: center; margin: 2rem 0;">
  <img src="/images/i3_renderer_part1/i3_bistro.png" alt="i3 Bistro Render" style="max-width: 90%; border-radius: 8px;" />
  <p style="font-style: italic; font-size: 0.8rem; color: #666;">
    Second example: the Bistro scene (Amazon Lumberyard), from the NVIDIA ORCA archive.
  </p>
</div>

### What's Next?

This whole machinery is automated by the **i3_baker**. It's still a bit "flat," but it's progressing!

In upcoming articles, we’ll see how all this integrates into the global RenderGraph. I’m currently looking into **HiZ**, **Occlusion Culling**, and **GTAO** for the next steps of the journey.

See you soon for more adventures!
