---
title: "i3 : Mon parcours dans le développement de moteurs de jeu"
draft: true
tags: ["moteur-de-jeu", "vulkan", "développement"]
---

"i3", c'est ma troisième tentative de moteur de jeu. Après "Insanity Engine" (qui a même fini sur Steam), "i3" est mon nouveau terrain de jeu. Un projet expérimental, éducatif, pour repousser les limites du dev de jeux. Pas de blabla, on apprend en faisant.
<!--more-->

## i3 : Le concept, sans fioritures

"i3" n'est pas juste du code. C'est une plateforme pour tester des techniques de rendu modernes et des architectures de moteur. L'objectif ? Apprendre. Mettre les mains dans le cambouis avec les API graphiques bas niveau et les systèmes complexes. Ma quête perso : une base solide et flexible pour mes futurs jeux.

## Sous le capot : La technique, cash

### Rendu : Vulkan, ça envoie du lourd

Au cœur de "i3", un moteur de rendu Vulkan. Pourquoi Vulkan ? Contrôle total sur le GPU, performances au top, rendu efficace. Ça force à comprendre les pipelines graphiques, la gestion mémoire, le multithreading. DirectX 12 ? Peut-être un jour, pour le fun.

### Polyglotte : C, C++, C#, le trio gagnant

"i3" mélange les langages : C (80.7%), C++ (16.0%), C# (2.1%). C et C++ pour la perf pure, là où ça compte (rendu, logique moteur). C# pour les outils, le scripting, ou la logique de jeu moins critique. Efficace, quoi.

### Build : Bazel, le maniaque de la propreté

Bazel (avec Bazelisk) gère le build. Pourquoi ? Parce que c'est propre, reproductible et rapide. Idéal pour un projet complexe et multi-langages comme "i3". Ça assure des builds cohérents, même quand le code grossit.

## Ce que i3 sait faire (déjà)

Encore expérimental, mais "i3" montre déjà les muscles :

*   **`vk_draw_cubes`** : Un cube, en Vulkan. La base, mais ça marche.
*   **`game_draw_cubes`** : Le moteur en action, pour rendre des objets dans un jeu.

Ces démos valident le cœur du moteur.

## Les galères et les leçons

Développer un moteur from scratch, surtout avec Vulkan, c'est pas une partie de plaisir. Pipelines graphiques, synchro mémoire, interopérabilité... Chaque problème est une leçon. On plonge dans les systèmes bas niveau, on comprend les graphiques. Ça pique, mais on progresse.

## La suite : Pas de plan fixe, mais des idées

La feuille de route de "i3" est vivante, elle évolue avec l'expérimentation. Quelques pistes :

*   Plus de rendu (lumières avancées, post-processing).
*   Un moteur physique digne de ce nom.
*   Des outils de dev de jeux plus poussés.
*   Explorer DirectX 12.

## Conclusion

"i3", c'est ma preuve que la curiosité paie. Un projet pour maîtriser les graphiques modernes et l'architecture moteur. C'est en cours, mais la base est là pour de futures explorations. Et qui sait, de nouveaux jeux.