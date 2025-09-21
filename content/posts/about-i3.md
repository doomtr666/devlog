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

### Rendu : Une architecture flexible avec Vulkan

Le rendu dans "i3" est conçu pour être modulaire. J'ai mis en place une interface de `render_backend` générique qui abstrait l'implémentation concrète du rendu. Pour l'instant, le seul backend est basé sur Vulkan.

Pourquoi ce choix ? Vulkan offre un contrôle bas niveau sur le matériel, des performances optimales et une excellente portabilité sur mes cibles principales (Windows et Linux). Même si l'architecture permettrait d'ajouter d'autres backends comme DirectX 12, ce n'est pas une priorité. L'interface est surtout là pour la flexibilité future, si jamais le besoin de supporter d'autres plateformes se présentait.

Se concentrer sur Vulkan me permet de maîtriser en profondeur les pipelines graphiques modernes, la gestion de la mémoire et le multithreading, qui sont des compétences cruciales pour le développement de moteurs de jeu performants.

### Polyglotte : C, C++, C# - Le bon outil pour chaque tâche

"i3" est un projet polyglotte, avec une répartition actuelle de C (80.7%), C++ (16.0%), et C# (2.1%). Ce mélange n'est pas un hasard, c'est une philosophie : utiliser le bon langage pour le bon problème.

**C#** est utilisé pour les problématiques de haut niveau. C'est un choix classique dans l'industrie pour le développement d'outils (`tooling`) et pour la logique de jeu (`game logic`). Sa richesse fonctionnelle et sa simplicité d'utilisation en font un excellent candidat pour ces tâches.

**C** est au cœur du moteur pour les systèmes bas niveau. Plus qu'une simple question de performance, c'est une approche. Traiter les problèmes bas niveau avec un langage bas niveau. L'ABI C est un standard de fait, ce qui garantit une interopérabilité simple et robuste entre les différents modules du moteur. La simplicité du C force à se concentrer sur l'essentiel : la résolution du problème, sans se perdre dans les méandres du langage.

Et le **C++** ? Il est présent, mais son usage est limité. Je le considère comme un langage trop complexe dont l'ABI n'est pas standard. Le risque est de passer plus de temps à "faire du C++" (faut-il utiliser la *move semantics* ici ?) qu'à résoudre le problème fonctionnel.

### Build : Bazel, pour un build unifié et maîtrisé

Le système de build de "i3" est géré par Bazel (via Bazelisk). Oui, Bazel peut être complexe, mais il répond parfaitement aux objectifs du projet.

Le principal avantage est d'avoir un **build unifié** pour tous les langages : C, C++ et C#. De plus, il est relativement simple d'étendre le système avec des règles personnalisées pour intégrer des outils externes.

J'ai une aversion pour les méta-générateurs de build comme CMake, qui pallient l'absence d'un système de build standard en C/C++ (ce qui m'a déjà donné l'idée de proposer un `std::build` au comité C++, mais c'est une autre histoire, peut-être pour un futur article...). Pour moi, c'est une hérésie. Bazel, au contraire, offre une approche directe et cohérente, garantissant des builds reproductibles et rapides, ce qui est crucial pour un projet de cette complexité.

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