---
title: "i3 : Pérégrinations aux frontières du Render Graph"
date: 2026-03-19
tags: ["Moteur 3D", "Vulkan", "Render Graph", "Architecture"]
categories: ["Journal de Dev", "i3"]
draft: true
---

### L'héritage : De l'Insanity Engine à la liquidation

Tout ça part d'une vieille histoire. J'ai codé la "techno lourde" (moteur 3D, physique, réseau, serveur) pour un jeu sorti sur Steam : [Win That War!](https://store.steampowered.com/app/599040/Win_That_War/). À l'époque, c'était du DX11, ça s'appelait l'**Insanity Engine**. J'étais le CTO du merdier, à porter le projet via des levées de fonds et les joyeusetés habituelles (CIR, JEI) grâce à ma techno. Mais si le projet n'a pas été fructueux commercialement (on a fait des conneries, les ventes n'ont pas suivi et on a fini par liquider), je refuse de parler d'échec technique. On a livré un vrai truc sur Steam. Et que ce soit à Rennes ou ailleurs, qui peut en dire autant ? Statistiquement, on parle d'un club qui regroupe moins de 0,001 % de la population mondiale. Depuis, la PI a été rachetée par Bidaj (aucune idée de qui ils sont) et je n'ai plus rien à voir là-dedans, mais je reste l'auteur du code.

Attention, si j'ai pondu le moteur, le jeu est le fruit du boulot phénoménal d'une équipe talentueuse. Et on en a chié. Entre les galères techniques sur le réseau, les artistes qui se foutaient presque sur la gueule pour des choix de DA, et des bugs proprement cringe... 

Côté rendu, j'ai aussi appris à gérer le feedback "créatif". Du genre le mec qui débarque devant ton écran et te lâche : "Ah c'est pas mal, mais je trouve que ça manque un peu de peps !". Démerde-toi pour convertir un "niveau de peps" en paramètres de BSDF et en tone mapping dans ton shader... 

Je me souviens notamment d'un bug sur le déplacement en vue planétaire : il fallait juste déplacer la caméra sur l'arc optimal entre deux points (segment du grand cercle). Sans déconner, ça a pris des semaines. "On comprend pas, on a refait la trigo et des fois ça merde encore". 

Mais c'était aussi le son au top et la musique vraiment cool, gros boulot de notre ingé son avec les **Bikini Machine**. Bref, avec le recul de ces 8 dernières années (ça a un peu vieilli, forcément), ça reste la meilleure expérience de dev et humaine de ma carrière.

Entre-temps, j'ai usé pas mal de temps perso sur **i2** (Insanity Engine 2). C'était mon bac à sable en OpenGL, jamais rien publié, juste des tests dans tous les sens. Et puis est arrivé **Vulkan**. Open standard, puissant, performant... J'ai été séduit immédiatement. C'est là qu'est né **i3**.

### Le voyage (et les sorties de route)

i3, c'est ma troisième tentative. On s'en branle un peu du langage, j'aurais pu faire ça en C++ ou n'importe quoi d'autre. Si je suis passé sur Rust, c'est purement utilitaire : j'ai besoin d'un langage système avec une vraie gestion de dépendances qui ne me demande pas d'y passer mes nuits. La sécurité mémoire ou le threading ? Très honnêtement, j'en ai rien à branler. Ce qui compte, c'est ce qu'il y a sous le capot.

1. **L'essai C# :** J'adore ce langage, mais dès qu'on attaque le transfert de données massif vers le GPU, c'est trop mou.
2. **L'essai C :** J'ai tenté une approche type *AMD V-EZ* (virtualisation des listes de commandes). Mais entre la misère des dépendances et le blocage psychologique de retomber dans le vieux papier de DICE sur les Render Graphs... la parallélisation est forcément suboptimale. C'est une misère pour un homme seul.

### Le pivot : Le Render Graph "avec quelques twists"

Le point marquant de cette troisième itération, c'est mon **renoncement** : je suis passé au **Render Graph**. Mais pas l'usine à gaz classique. J'ai injecté quelques twists pour éviter de reproduire les structures trop rigides que j'ai croisées par le passé. 

L'idée, c'est d'automatiser ce qui est pénible dans Vulkan (barrières, transitions de layouts) sans pour autant perdre le contrôle fin sur l'exécution. Je commence enfin à être content du résultat.

Ceci est mon journal de bord. On ne va pas parler de la "beauté du code", mais de pérégrinations techniques aux frontières de l'inconnu, là où on forge des outils qui fonctionnent vraiment.

---