---
title: "i3: Wanderings at the Edge of the Frame Graph"
date: 2026-03-20
tags: ["3D Engine", "Vulkan", "Frame Graph", "Architecture"]
categories: ["Dev Log", "i3"]
draft: false
---

### The Legacy: From the Insanity Engine to Liquidation

It all starts with an old story. I coded the "heavy tech" (3D engine, physics, networking, server) for a game released on Steam: [Win That War!](https://store.steampowered.com/app/599040/Win_That_War/). Back then, it was DX11, and we called it the **Insanity Engine**. I was the CTO of the whole thing, carrying the project through fundraising and the usual administrative joys (CIR, JEI) thanks to my tech. But even if the project wasn't commercially successful (we made our share of mistakes, sales didn't follow, and we ended up liquidating), I refuse to call it a technical failure. We shipped a real game on Steam. And whether in Rennes or elsewhere, who can say the same? Statistically, it's a club that comprises less than 0.001% of the global population: if you look closely, you're 6 times more likely to get struck by lightning at least once in your life than to meet another average joe with the same track record on Steam. Since then, the IP was bought out by Bidaj (no idea who they are), and I have nothing to do with it anymore, though I remain the author of the code.

Mind you, while I birthed the engine, the game was the result of phenomenal work by a talented team. And we went through hell. Between the technical struggles on the network, artists almost coming to blows over art direction choices, and some genuinely cringe-worthy bugs... 

On the rendering side, I also learned to manage "creative" feedback. You know, like the guy who shows up behind your screen and drops: "Oh, it's not bad, but I think it lacks a little punch!". Good luck translating an undefined "level of punch" into BSDF parameters and shader tone mapping... 

I specifically remember a bug regarding planetary view movement: we just needed to move the camera along the optimal arc between two points (a great circle segment). No kidding, that took weeks. "We don't get it, we redid the trigonometry and sometimes it still breaks". 

But it was also top-tier audio and really cool music, massive work from our sound engineer with the **Bikini Machine**. In short, looking back over these past 8 years (it's aged a bit, inevitably), it remains the best developmental and human experience of my career.

Meanwhile, I spent quite a bit of personal time on **i2** (Insanity Engine 2). It was my OpenGL/C++ sandbox, never published anything, just testing out ideas all over the place. And then came **Vulkan**. Open standard, powerful, performant... I was hooked immediately. That's where **[i3](https://github.com/doomtr666/i3)** was born.

### The Journey (and the Detours)

i3 is my 4th attempt at the 3rd iteration of the engine. The fundamental issue—what you want as an architect—is to have a list of well-decoupled and reusable graphics passes to maintain the freedom of composition and reusability. 

But with explicit APIs like Vulkan, you have to place your synchronization barriers yourself. The problem is that to place a barrier, you need to know the previous state of the resource AND its desired state before your pass. And there goes your modularity: goodbye to clean separation. If pass B must know exactly what pass A did with the texture, you're stuck. It's a real, structural architectural problem.

To solve this, I tried to fully lean into AMD's **[Vulkan-EZ](https://github.com/GPUOpen-LibrariesAndSDKs/V-EZ)** (V-EZ) approach. The idea is appealing: offer a high-level abstraction over Vulkan `cmd_buffers` that tracks resource usage and emits barriers through internal "magic". 

On paper, it's not bad. But there’s a fundamental flaw: no matter how you spin it, barrier computation is inherently **sequential**. Because it depends on the actual order of use, even if you record your virtual buffers in parallel, you'll inevitably have a sequential phase to calculate barriers and update resource states. 

Sure, you can then switch back to parallel for the actual translation to Vulkan, but the damage is done: your logical commands are recorded **twice** (once by you in virtual buffers, and a second time by the engine into the real `VkCommandBuffer`s). Result: far too much memory movement, overhead, and inevitably, sub-optimal performance, no matter what you do.

But the undeniable advantage is that you get your modularity back, since you no longer deal with barriers. That crucial point pushed me to persist in these experiments.

I explored several avenues before facing the facts:

1. **The C# Attempt:** I have a huge soft spot for this language, but as soon as we tackle massive data transfers to the GPU, it's too sluggish. It's great for game logic, but it lacks low-level punch.
2. **The C Attempt:** My first love: it's simple, the compiler is predictable, it smells like silicon. I pushed the V-EZ approach to the limit in this version (which is still sitting over on my GitHub, actually). But between the sequential tracking bottleneck and dependencies starting to weigh me down... it stagnated. 
3. **The C#/C Hybrid:** .NET 10 (excellent JIT) coupled with pure C via the standard ABI. On paper, it's effective. In practice, it's an inflexible nightmare. In short, it’s premature optimization: while transforming critical calls into native code makes sense for a mature engine, for an actively developing project, it's just way too complex to manage.

That's when I thought: "Let's revisit **[DICE's Frame Graphs](https://www.gdcvault.com/play/1024612/FrameGraph-Extensible-Rendering-Pipelines-in)**, and while we're at it, let's try Rust to build it." And that’s how the 4th iteration of **i3** was born.

I admit, I was won over by Rust: it's a systems language, compiled, highly performant, with the right technical properties for this kind of undertaking. We'll discuss it more in a dedicated article, but paired with modern dependency management via **Cargo**, it's the perfect tool for implementing this kind of architecture.

### The Pivot: The Frame Graph

That's when I finally accepted my **compromise**: migrating to the **Frame Graph**. It's the only way to solve the modularity/barriers equation without suffering the bottleneck of a sequential approach. For the rest, I ended up appropriating DICE's concept while adding my own personal "touch", which I'll detail in a future article, by the way.

Honestly, the work done by DICE back then on Frostbite is a rarely equaled piece of engineering. But when you know they were an army of 200 people with no less than 70 PhDs on deck for the engine at the very least, you understandably remain skeptical about the possibility of pulling off such a feat as a lone "providential man" in your corner. 

Still, it's the most solid approach to date for automating the tedious stuff (barriers, layout transitions) without sacrificing fine-grained control over execution. We could imagine other paths, like a DSL with "bytecode"-style graph management, or even native compilation via LLVM JIT. Some research even explores supporting **cycles** by treating the Frame Graph as a true compiler **Control Flow Graph (CFG)**, using **phi-nodes** to handle circular dependencies—an ideal approach for temporal **denoising** algorithms. We then leave the comfort of the DAG to enter even more technically aggressive territories. In the meantime, for now, i3 indeed uses this famous Frame Graph, well-assisted by AI (which I will also discuss in one or more dedicated articles).

This is my logbook. We'll talk about tech, exploration, and architectural trade-offs. Because between an elegant theoretical abstraction and the physical reality of execution on a GPU, there is a gulf. Deep down, i3 remains my sandbox, and the Frame Graph is just one building block among many needed to assemble it.
