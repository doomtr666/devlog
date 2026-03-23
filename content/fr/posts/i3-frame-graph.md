---
title: "Frame Graph : L'Orchestrateur de l'Ombre"
date: 2026-03-23
tags: ["Moteur 3D", "Vulkan", "Frame Graph", "Rust", "Architecture"]
categories: ["Journal de Dev", "i3"]
draft: true
---

Dans mon article précédent, je vous parlais du fameux "renoncement" : le passage au **Frame Graph**. C'est un grand mot, un peu à la mode dans le milieu du moteur 3D, mais derrière le buzzword se cache une réalité brutale. Quand tu passes sur des API explicites comme Vulkan, tu te retrouves avec la gestion manuelle des barrières de synchronisation. Et là, c'est le début des emmerdes.

Imagine que tu es un chef d'orchestre, mais qu'au lieu de juste battre la mesure, tu doives aller voir chaque musicien individuellement pour lui dire : "Toi, tu t'arrêtes pile à la mesure 12, tu poses ton violon, tu vérifies qu'il est bien sec, et seulement ALORS le trompettiste peut commencer sa note". Tu multiplies ça par 100 passes de rendu et des milliers de ressources, et tu as une recette parfaite pour un mal de crâne carabiné et des bugs de synchro indébuggables.

### Le problème du couplage implicite

Le vrai souci, c'est le couplage. Si ta passe de *Lighting* a besoin de savoir exactement dans quel état la passe de *GBuffer* a laissé la texture de profondeur pour poser sa barrière, tu as créé un lien invisible mais indestructible entre les deux. Ton architecture devient rigide comme un vieux bout de bois. Tu veux rajouter une passe de *Decals* au milieu ? Vas-y, bon courage pour refaire toute la chaîne de synchro à la main.

### Le concept : Déclarer, Compiler, Exécuter

Pour s'en sortir, on sépare le "quoi" du "comment". C'est un cycle en trois temps :

```mermaid
graph LR
    D[1. DECLARE<br/>Les passes disent ce qu'elles veulent] --> C[2. COMPILE<br/>Le moteur calcule les barrières et le planning]
    C --> E[3. EXECUTE<br/>Le GPU fait le boulot en parallèle]
```

C'est là qu'entre en scène le **Frame Graph**. L'idée, c'est de dire : "Passes, dites-moi juste ce que vous consommez et ce que vous produisez. Moi, le moteur, je m'occupe des barrières, du planning et de la logistique".

### L'Invariant du Nœud : La Loi de i3

Pour que i3 ne devienne pas une usine à gaz, j'ai posé un principe simple mais non négociable : **L'Invariant du Nœud**. 
Un nœud (une passe) est une séquence de commandes GPU ininterruptible. Si une passe a besoin d'une ressource dans deux états différents (genre j'écris dans une texture en Compute puis je la lis dans un Shader), alors ce n'est pas une passe, c'est **deux passes**. Point.

Ça peut paraître rigide, mais c'est ce qui permet au compilateur du graphe de raisonner proprement. Le graphe devient un arbre de nœuds, et la résolution des barrières devient un problème purement logique, presque comme dans un compilateur de langage.

### Rust et le "Scoped Symbol Table"

Pour implémenter ça, je me suis pas mal inspiré de la théorie des compilateurs (coucou les SSA et les Phi-nodes). Dans i3, tout est un "Symbole" : une image, un buffer, mais aussi des données CPU comme la caméra ou les réglages de rendu.

Grâce à une structure de table de symboles scopée (merci la puissance de Rust pour gérer ça proprement), une passe peut "publier" une ressource et une autre peut la "consommer". Le compilateur regarde tout ça, aplatit l'arbre, trie les dépendances (Topological Sort pour les intimes) et hop : il te sort un plan d'exécution optimal.

### Parallèle par design

Le vrai kiff, c'est la parallélisation. Comme le graphe connaît toutes les dépendances, il sait exactement quelles passes peuvent tourner en même temps sans se marcher sur les pieds. 
J'utilise **Rayon** pour ça. Si tu as 8 cœurs, i3 va enregistrer tes commandes sur 8 threads en parallèle. Pas parce que je l'ai forcé, mais parce que l'architecture du graphe le permet nativement. On est loin des goulots d'étranglement de mes essais précédents en C ou en C#.

### Et la performance dans tout ça ?

C'est là que le bât blesse souvent avec les abstractions de haut niveau : l'overhead. Mais le Frame Graph de i3 est conçu pour être ultra-rapide. La phase de compilation prend moins de 100 microsecondes pour une centaine de passes. On ne re-record pas tout à chaque fois pour rien. On déclare léger, on compile vite, on enregistre une fois.

On profite aussi de `VK_KHR_dynamic_rendering` et `VK_KHR_synchronization2` (Vulkan 1.3 baseline, s'il vous plaît) pour garder un backend propre et moderne. Adieu les objets `VkRenderPass` et `VkFramebuffer` qui pesaient trois tonnes, on est sur de l'explicite fluide.

### Conclusion (provisoire)

Le Frame Graph, c'est l'orchestrateur invisible. C'est lui qui permet à i3 d'être à la fois modulaire (je peux rajouter des passes comme des Legos) et performant (tout est synchronisé au millimètre). 

Bien sûr, tout n'est pas rose. Il y a encore des questions ouvertes sur le batching optimal des barrières ou la gestion des dépendances cycliques (pour le denoising temporel par exemple). Mais c'est ça qui est beau dans le dev moteur : on repousse les frontières de l'infini, un symbole à la fois.

Dans le prochain épisode, on parlera de la "bouffe" du GPU : le **Baker**. Parce que c'est bien beau d'avoir un super orchestre, mais si les musiciens n'ont pas de partitions lisibles, on va pas aller loin !
