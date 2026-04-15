---
title: "A propos d'i3"
date: 2026-03-20
tags: ["Moteur 3D", "Vulkan", "Frame Graph", "Architecture"]
categories: ["Journal de Dev", "i3"]
draft: false
---

### L'héritage : De l'Insanity Engine à la liquidation

Tout ça part d'une vieille histoire. J'ai codé la "techno lourde" (moteur 3D, physique, réseau, serveur) pour un jeu sorti sur Steam : [Win That War!](https://store.steampowered.com/app/599040/Win_That_War/). 

<!--more-->
À l'époque, c'était du DX11, ça s'appelait l'**Insanity Engine**. J'étais le CTO du bouzin, à porter le projet via des levées de fonds et les joyeusetés habituelles (CIR, JEI) grâce à ma techno. Mais si le projet n'a pas été fructueux commercialement (on a fait des conneries, les ventes n'ont pas suivi et on a fini par liquider), je refuse de parler d'échec technique. On a livré un vrai jeu sur Steam. Et que ce soit à Rennes ou ailleurs, qui peut en dire autant ? Statistiquement, on parle d'un club qui regroupe moins de 0,001 % de la population mondiale : à bien y regarder, tu as 6 fois plus de chances de te faire foudroyer au moins une fois dans ta vie que de croiser un autre péon avec le même palmarès sur Steam. Depuis, la PI a été rachetée par Bidaj (aucune idée de qui ils sont) et je n'ai plus rien à voir là-dedans, mas je reste l'auteur du code.

Attention, si j'ai pondu le moteur, le jeu est le fruit du boulot phénoménal d'une équipe talentueuse. Et on en a bavé. Entre les galères techniques sur le réseau, les artistes qui en venaient presque aux mains pour des choix de DA, et des bugs proprement cringe... 

Côté rendu, j'ai aussi appris à gérer le feedback "créatif". Du genre le mec qui débarque devant ton écran et te lâche : "Ah c'est pas mal, mais je trouve que ça manque un peu de peps !". Vas-y pour convertir un "niveau de peps" en paramètres de BSDF et en tone mapping dans ton shader... 

Je me souviens notamment d'un bug sur le déplacement en vue planétaire : il fallait juste déplacer la caméra sur l'arc optimal entre deux points (segment du grand cercle). Sans déconner, ça a pris des semaines. "On comprend pas, on a refait la trigo et des fois ça merde encore". 

Mais c'était aussi le son au top et la musique vraiment cool, gros boulot de notre ingé son avec les **Bikini Machine**. Bref, avec le recul de ces 8 dernières années (ça a un peu vieilli, forcément), ça reste la meilleure expérience de dev et humaine de ma carrière.

Entre-temps, j'ai usé pas mal de temps perso sur **i2** (Insanity Engine 2). C'était mon bac à sable en OpenGL/C++, jamais rien publié, juste des tests dans tous les sens. Et puis est arrivé **Vulkan**. Open standard, puissant, performant... J'ai été séduit immédiatement. C'est là qu'est né **[i3](https://github.com/doomtr666/i3)**.

### Le voyage (et les sorties de route)

i3, c'est ma 4ème tentative pour la 3ème itération du moteur. Tout le problème de fond, ce que tu veux en tant qu'architecte, c'est d'avoir une liste de passes graphiques bien découplées et réutilisables pour avoir la latitude de composition et de réutilisation. 

Mais avec les API explicites comme Vulkan, tu dois poser tes barrières de synchro toi-même. Le souci, c'est que pour poser une barrière, tu as besoin de connaître l'état précédent de la ressource ET son état désiré avant ta passe. Et là, c'est la fin de ta modularité : adieu le découpage propre. Si ta passe B doit savoir exactement ce que la passe A a fait de la texture, tu es coincé. C'est un vrai problème d’architecture structurant.

Pour régler ça, j'ai tenté d'aller au bout de l'approche **[Vulkan-EZ](https://github.com/GPUOpen-LibrariesAndSDKs/V-EZ)** (V-EZ) d'AMD. L'idée est séduisante : proposer une abstraction de haut niveau des `cmd_buffers` Vulkan qui tracke l’usage des ressources et émet les barrières "par magie" interne. 

Sur le papier, c’est pas mal. Mais il y a un problème fondamental : peu importe comment tu tournes le truc, le calcul des barrières est forcément **séquentiel**. Puisqu’il dépend de l’ordre réel d’utilisation, même si tu enregistres tes buffers virtuels en parallèle, tu vas forcément avoir une phase séquentielle pour calculer les barrières et mettre à jour l’état des ressources. 

Certes, tu peux ensuite repasser en parallèle pour la traduction réelle vers Vulkan, mais le mal est fait : tes commandes logiques sont enregistrées **deux fois** (une fois par toi dans les buffers virtuels, une seconde par l'engine dans les vrais `VkCommandBuffer`). Résultat : beaucoup trop de mouvements mémoire, de l'overhead, et une performance forcément sous-optimale quoi que tu fasses.

Mais l'avantage indéniable, c'est que tu récupères ta modularité, puisque tu ne t'occupes plus des barrières. C'est ce point crucial qui m'a poussé à persister dans ces expérimentations.

J'ai exploré plusieurs pistes avant de me rendre à l'évidence :

1. **L'essai C# :** J'ai un gros faible pour ce langage, mais dès qu'on attaque le transfert de données massif vers le GPU, c'est trop mou. C'est super pour de la game logic, mais ça manque de punch à bas niveau.
2. **L'essai C :** Mes premières amours : c'est simple, le compilo est prédictible, ça sent bon le silicium. J'ai poussé l'approche V-EZ jusqu'au bout dans cette version (qui est d'ailleurs toujours sur mon GitHub). Mais entre le goulot d'étranglement du tracking séquentiel et les dépendances qui commençaient à me peser... ça stagnait. 
3. **Le hybride C#/C :** Du .NET 10 (JIT excellent) couplé à du C pur via l'ABI standard. Sur le papier, c'est efficace. En pratique, c'est un enfer de rigidité. Bref, c’est de l’optimisation prématurée : si transformer des appels critiques en natif a du sens pour un moteur mature, pour un projet en plein dev, c'est juste beaucoup trop complexe à gérer.

C'est là que je me suis dit : "Revoyons les **[Frame Graphs de DICE](https://www.gdcvault.com/play/1024612/FrameGraph-Extensible-Rendering-Pipelines-in)**, et tant qu'à faire, essayons Rust pour le faire". Et c'est là qu'est née la 4ème itération de **i3**.

J'avoue avoir été conquis par Rust : c'est un langage système, compilé, performant, avec les bonnes propriétés techniques pour ce genre de chantier. On en reparlera dans un article dédié, mais avec une gestion de dépendances moderne via **Cargo**, c'est l'outil parfait pour implémenter ce genre d'architecture.

### Le pivot : Le Frame Graph

C'est là que j'ai fini par accepter mon **renoncement** : le passage au **Frame Graph**. C'est la seule façon de résoudre l'équation modularité/barrières sans se taper le goulot d'étranglement de l'approche séquentielle. Pour le reste, j'ai fini par m'approprier le concept de DICE en y ajoutant ma "patte" personnelle, ce que je détaillerai d'ailleurs dans un prochain article.

Honnêtement, le travail de l'époque chez DICE sur Frostbite est une pièce d'ingénierie rarement égalée. Mais quand tu sais qu'ils étaient une armée de 200 personnes dont pas moins de 70 docteurs sur le pont pour le moteur au bas mot, tu restes forcément sceptique sur la possibilité de réaliser un truc pareil en tant "qu'homme providentiel" tout seul dans ton coin. 

Pourtant, c'est l'approche la plus solide à ce jour pour automatiser ce qui est pénible (barrières, transitions de layouts) sans pour autant perdre le contrôle fin sur l'exécution. On pourrait imaginer d'autres voies, comme un DSL avec une gestion du graphe façon "bytecode", voire même une compilation native via JIT LLVM. Certains travaux explorent même le support des **cycles** en traitant le Frame Graph comme un véritable **Control Flow Graph (CFG)** de compilateur, utilisant des **phi-nodes** pour gérer les dépendances circulaires — une approche idéale pour les algorithmes de **denoising** temporel. On quitte alors le confort du DAG pour entrer dans des territoires encore plus agressifs techniquement. En attendant, pour l'instant c'est bien ce fameux Frame Graph dans i3, bien aidé par l'IA (dont je reparlerai aussi dans un ou plusieurs articles dédiés).

Ceci est mon journal de bord. On y parlera technique, exploration et compromis d'architecture. Parce qu'entre une abstraction théorique élégante et la réalité physique de l'exécution sur GPU, il y a un gouffre. Dans le fond, i3 reste mon bac à sable, et le Frame Graph n'est qu'une brique parmi tant d'autres pour l'assembler.

---