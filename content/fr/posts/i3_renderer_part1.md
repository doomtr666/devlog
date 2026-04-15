---
title: "Rendu i3 (Partie 1) : L'IBL"
date: 2026-04-14
tags: ["Moteur 3D", "IBL", "Rendering", "Vulkan", "RayQuery", "Physically Based Rendering"]
categories: ["Journal de Dev", "i3"]
draft: false
---

Dans mon [article précédent](../i3-frame-graph/), on a plongé dans les entrailles logistiques du **Frame Graph** d'**[i3](https://github.com/doomtr666/i3)**. C'est la base, la tuyauterie qui permet d'orchestrer le rendu sans s'arracher les cheveux sur les barrières Vulkan. Mais une fois que la plomberie est en place, il faut bien commencer à dessiner des trucs à l'écran.

Aujourd'hui, on va s'attaquer à un grand classique du rendu moderne : l'**Image-Based Lighting (IBL)**.
<!--more-->

### Ambition vs Stratégie : Le rendu incrémental

Je vais être honnête : j'ai une certaine ambition sur le rendu final d'i3. J'ai envie de trucs qui claquent, de reflets cohérents et d'une gestion de la lumière proprement physique. Mais on ne pond pas un moteur de rendu en un après-midi tout seul dans son garage. 

Ma stratégie est donc la suivante : implémenter une collection de techniques **composables**. Actuellement, la base du moteur est un **Clustered Deferred** classique (G-Buffers, Deferred Resolve, Tonemap). La seule véritable fioriture jusqu'à présent, ce sont les **Normal Maps**, mais sans éclairage global ni réflexions, le rendu faisait encore très "début des années 2000".

On va donc améliorer cela progressivement. Pas de "Uber-Shader" monolithique indébuggable, mais des briques qu'on empile. Et la première brique de l'éclairage ambiant, c'est l'IBL.

### La Madeleine de Proust : Split Sum Approximation

Revisiter l'IBL, c'est pour moi une véritable madeleine de proust. J'avais codé une version de cet algorithme il y a environ 10 ans pour l'**InsanityEngine**, le moteur de mon jeu [Win That War!](https://store.steampowered.com/app/599040/Win_That_War/) sorti sur Steam. C'était l'époque des pionniers de l'indé, et se battre avec les intégrales de lumière était déjà un sport national.

Pour commencer en douceur à améliorer ce rendu, je repars sur le modèle devenu standard du "Split Sum Approximation" de **Brian Karis** chez Epic Games. L'article original de la conférence SIGGRAPH 2013, [**"Real Shading in Unreal Engine 4"**](https://cdn2.unrealengine.com/Resources/files/2013SiggraphPresentationsNotes-26915738.pdf), reste la référence sur le sujet. Si vous voulez des bases plus didactiques, je ne peux que vous conseiller les excellents articles de LearnOpenGL sur la [Diffuse Irradiance](https://learnopengl.com/PBR/IBL/Diffuse-irradiance) et la [Specular IBL](https://learnopengl.com/PBR/IBL/Specular-IBL).

C'est efficace, c'est robuste et ça va donner un peu de relief au résultat final. Mais comme d'habitude, j'ai voulu y ajouter ma "patte" et quelques spécificités techniques pour rendre ça plus moderne et adapté à i3.

<div style="text-align: center; margin: 2rem 0;">
  <img src="/images/i3_renderer_part1/hdri.png" alt="HDRI Source Poly Haven" style="max-width: 80%; border-radius: 8px;" />
  <p style="font-style: italic; font-size: 0.9rem; color: #666;">
    L'HDRI source provenant de <a href="https://polyhaven.com/hdris">Poly Haven</a>.
  </p>
</div>

### Adieu Cubemaps, Bonjour BC6H

Traditionnellement, l'IBL utilise des **Cubemaps**. C'est pratique, c'est supporté par tout le monde, mais c'est aussi un peu chiant à gérer (6 faces, gestion des jointures, etc.). 

Dans i3, on part d'un fichier **Equirectangulaire HDRI** (.hdr ou .exr) classique. Mais au lieu de le convertir en Cubemap, on le laisse tel quel et on le compresse en **BC6H**. 

C'est un format de compression matériel dédié au HDR : contrairement aux formats classiques comme le BC1 ou BC3 (limités au LDR), le BC6H supporte nativement les données **16-bit float**. Cela permet de garder une plage dynamique énorme (indispensable pour le soleil) tout en divisant le poids de la texture par 4 ou 6. 

Pas de conversion complexe, pas de pertes dues au ré-échantillonnage en 6 faces. On sample l'equirect directement.

### Le Dual-Octahedral Mapping

C'est là que ça devient intéressant d'un point de vue technique. Cette approche vient directement de l'optimisation des G-Buffers et de la compression des normales, popularisée notamment par les travaux sur le Real-Time Global Illumination et des moteurs comme **Frostbite** ou **Unity (HDRP)**.

#### Pourquoi abandonner l'Octahedral classique ?
Le principe de base de l'encodage octaédrique (Octahedral Mapping) a été formalisé par Meyer et al. (2010), puis benchmarké par Cigolle et al. (2014) ([*"A Survey of Efficient Representations for Independent Unit Vectors"*](https://jcgt.org/published/0003/02/01/)). 

Le problème d'une projection octaédrique standard sur une seule map, c'est la **discontinuité du sampling**. Lorsque vous faites du filtrage bilinéaire hardware ou du mipmapping, les bords et les diagonales de l'octaèdre déplié ne correspondent pas physiquement dans l'espace texture. Cela crée des artefacts de filtrage impossibles à corriger proprement sans "border gutters" complexes.

#### Le "DICE Trick" : Dual-Octahedral
L'astuce (utilisée chez Frostbite) consiste à utiliser un encodage **Hemi-Octahedral**. On divise la sphère en deux hémisphères (Nord/Sud) que l'on traite séparément. En mettant ces deux projections côte à côte, on double la densité de pixels pour la zone utile et on assure une interpolation hardware beaucoup plus propre.

Voici comment i3 encode cela dans son baker (Rust) :

```rust
/// Encode direction sphère → Dual-Octahedral UV [0,1]².
pub fn hemi_octa_encode(d: [f32; 3]) -> (f32, f32) {
    let [x, y, z] = d;
    let l = x.abs() + y.abs() + z.abs();
    let ox = x / l;
    let oz = z / l;
    let u = ox * 0.5 + 0.5;

    // On split le range V en deux : Nord [0, 0.5] et Sud [0.5, 1.0]
    let v_local = oz * 0.5 + 0.5;
    let v = if y >= 0.0 { v_local * 0.5 } else { 0.5 + v_local * 0.5 };
    (u, v)
}
```

#### Avantages concrets :
1.  **Uniformité** : Contrairement à une map Equirectangulaire où les pixels s'entassent aux pôles, ici chaque pixel couvre à peu près la même portion d'angle solide.
2.  **Densité** : En coupant en deux, on maximise l'occupation du carré de texture.
3.  **Vitesse** : Les fonctions de repliage sont extrêmement légères en instructions shader (quelques `abs` et `sign`).

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
    Les trois maps résultant du bake i3 : Irradiance (Dual-Octa), Prefiltered (Dual-Octa + Mips) et la BRDF LUT traditionnelle.
  </p>
</div>



### Energy Balancing & Sun Extraction

Le gros problème de l'IBL "brut", c'est qu'il contient tout : le ciel, les nuages, mais aussi le **soleil** (le "bright spot" énorme). Si on utilise l'IBL tel quel, on a un éclairage ambiant correct, mais on perd les ombres portées nettes et le contrôle sur la direction de la lumière principale.

Dans i3, j'ai mis au point un algorithme d'**Energy Balancing** lors du bake (via l'`i3_baker`).

Le baker scanne le HDR pour détecter la source de lumière principale (le soleil). Une fois détecté, on définit un seuil et on "extrait" l'énergie solaire de l'IBL. Cela permet de combiner l'IBL avec une vraie lumière directionnelle (Sun Light) dans le moteur, sans doubler l'énergie lumineuse et en conservant la possibilité de projeter des ombres.

Cela garantit que la contribution directe du soleil est toujours équilibrée par rapport à l'irradiance ambiante masquée. Résultat : une cohérence parfaite avec l'environnement HDR d'origine.

### Des ombres, enfin ! (via RayQuery)

Puisque nous avons extrait le soleil de l'IBL pour en faire une lumière directionnelle classique, nous pouvons enfin avoir des ombres portées. 

Pour l'instant, i3 fait l'impasse sur les Shadow Maps classiques et utilise directement du **Ray Tracing** via **RayQuery**. Notons ici que le moteur synchronise automatiquement les structures d'accelération pour un support RT natif. Il n'y a pas encore de fallback pour le matériel sans support RT, mais cela permet d'avoir un pipeline propre et moderne dès le départ. Lors de la phase de **Deferred Resolve**, on tire un rayon vers le soleil pour chaque pixel visible. C'est propre, ça évite les artefacts de résolution, et ça profite des capacités RT des cartes modernes.

<div style="text-align: center; margin: 2rem 0;">
  <img src="/images/i3_renderer_part1/i3_sponza.png" alt="Rendu i3 Sponza" style="max-width: 90%; border-radius: 8px;" />
  <p style="font-style: italic; font-size: 0.8rem; color: #666;">
    Premier exemple : la scène Sponza, issue des exemples <a href="https://github.com/KhronosGroup/glTF-Sample-Assets/tree/main">glTF Sample Assets</a>.
  </p>
</div>

<div style="text-align: center; margin: 2rem 0;">
  <img src="/images/i3_renderer_part1/i3_bistro.png" alt="Rendu i3 Bistro" style="max-width: 90%; border-radius: 8px;" />
  <p style="font-style: italic; font-size: 0.8rem; color: #666;">
    Second exemple : la scène Bistro (Amazon Lumberyard), issue de l'archive ORCA via NVIDIA.
  </p>
</div>

### Et après ?

Toute cette machinerie est automatisée par l'**i3_baker**. C'est encore un peu "plat", mais ça progresse !

Dans les prochains articles, on verra comment tout cela s'intègre dans le RenderGraph global. Je regarde d'ailleurs du côté du **HiZ**, l'**Occlusion Culling** et enfin le **GTAO** pour la suite du voyage.

À bientôt pour de nouvelles aventures !
