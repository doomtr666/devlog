---
title: "Vibe Coding avec Gemini : Évaluation d'un Agent IA pour la Création d'un Site"
draft: false
tags: ["gemini", "ai", "développement", "vibe-coding", "hugo"]
---

Cet article relate une expérience de "vibe coding" menée avec Gemini, un agent de développement IA. Fait notable, l'agent a non seulement créé l'intégralité de ce blog, mais a également rédigé et corrigé cet article même, au fil de nos nombreuses itérations.

<!--more-->

## Introduction : L'Idée de Départ

Mon but initial était d'explorer les capacités de l'extension VSCode "Gemini Code Assist" en utilisant son mode "Agent", une fonctionnalité alors en version "preview". Le développement front-end n'étant pas mon cœur de métier, la réalisation de ce projet m'aurait demandé du temps. Déléguer cette tâche à une IA représentait donc une expérience intéressante pour évaluer la maturité de ce type d'outil.

J'ai donc adopté une approche de "vibe coding" : une approche exploratoire où l'on se laisse guider par l'outil, sans cahier des charges strict, pour découvrir les possibilités. Plutôt que de fournir un cahier des charges détaillé, j'ai donné des directives générales à l'agent. Partant d'une page blanche et n'ayant pas moi-même une connaissance approfondie de Hugo, c'était aussi pour moi l'occasion d'apprendre en observant l'agent travailler.

## L'Agent IA au Travail

L'agent Gemini est un grand modèle linguistique fonctionnant en ligne de commande. Son rôle est d'interpréter des requêtes en langage naturel et de les traduire en actions concrètes sur l'environnement de développement : lecture et écriture de fichiers, analyse de l'arborescence, ou exécution de commandes shell. Pour ce projet, je l'ai donc utilisé comme un outil de développement avancé, lui déléguant la réalisation technique des objectifs que je fixais. Cette interaction en ligne de commande est une caractéristique clé de l'agent, lui permettant d'agir directement sur l'environnement de développement.

## Pérégrinations

Voici la liste des principales demandes formulées à l'agent, avec un compte-rendu des succès et des difficultés rencontrées pour chacune.

1.  **Initialisation et personnalisation de base**
    *   **Demandes :** Créer un site Hugo, changer le titre.
    *   **Résultat :** Succès. Tâches simples et bien exécutées.

2.  **Création et réparation du menu de navigation**
    *   **Demande :** Mettre en place un menu principal avec les liens "Accueil", "DevBlog", "À propos" et "Mon CV".
    *   **Résultat :** Très laborieux. Ce fut l'un des premiers points de friction. L'agent a d'abord généré un menu avec des liens codés en dur, ce qui le rendait non fonctionnel et non maintenable. Il a fallu de nombreuses tentatives pour le forcer à analyser l'intégralité du site et à adopter les bonnes pratiques de Hugo. Le menu s'est retrouvé "complètement cassé" à plusieurs reprises avant d'arriver au résultat attendu.

3.  **Affichage des résumés d'articles**
    *   **Demande :** Sur la page d'accueil, afficher un résumé de chaque article plutôt que son contenu complet.
    *   **Résultat :** Succès après une itération. La première tentative de l'agent a échoué. Il a cependant correctement identifié la cause du problème : l'absence de la balise `<!--more-->` dans les articles, une fonctionnalité de Hugo que j'ignorais. Il a ensuite appliqué le correctif avec succès.

4.  **Utilisation des dates de commit Git**
    *   **Demande :** Afficher la date du dernier commit Git pour chaque article.
    *   **Résultat :** Mitigé. L'agent n'a pas pu générer la solution sans assistance. J'ai dû lui fournir la syntaxe de template Hugo exacte à utiliser.

5.  **Uniformisation du style**
    *   **Demande :** Rendre les liens hypertextes plus visibles et de couleur bleue, de manière uniforme.
    *   **Résultat :** Succès, mais poussif. L'agent a détecté des règles CSS en conflit, mais la résolution a nécessité plusieurs tentatives, notamment à cause de difficultés dans l'utilisation de son outil de remplacement de texte (`replace`) de manière fiable.

6.  **Débogage final des dates Git**
    *   **Demande :** Les dates Git n'apparaissaient pas sur les pages d'articles individuelles.
    *   **Résultat :** Très laborieux. Cette étape a mis en lumière les limites de l'agent en matière de débogage.
        *   **Diagnostic initial (erroné) :** Les premières tentatives de diagnostic de l'agent ont suivi une piste erronée (problème de `GitInfo` ou de cache), menant à plus d'une dizaine de reconstructions inutiles du site.
        *   **Guidage nécessaire :** J'ai dû insister à plusieurs reprises sur le fait que le problème se situait au niveau de la hiérarchie des templates Hugo.
        *   **Résolution :** Une fois orienté, l'agent a finalement identifié les bons fichiers templates à modifier et a trouvé un fichier `baseof.html` conflictuel à supprimer, résolvant enfin le problème.

## Conclusion : Un Développeur Débutant à l'Esprit Bien Câblé

Au final, que retenir de cette démarche ? Le résultat est pour le moins inégal, mais fascinant. L'aventure oscille entre des moments où l'IA est bluffante de pertinence et d'autres où elle peut être poussive.

Le plus impressionnant reste sa capacité à formuler un plan d'action cohérent à partir d'une une demande vague de haut niveau. Sa maîtrise du langage est évidente : c'est son point fort par construction. Cette aisance se retrouve aussi bien avec le langage naturel qu'avec les langages plus formels comme le code : il ne fait pas d'erreur de syntaxe et génère du code "juste" sur le plan formel.

Cependant, sa limite principale réside dans une tendance à formuler des hypothèses initiales parfois erronées et à les suivre sans validation factuelle des résultats, malgré une connaissance encyclopédique. Ces "trous" cognitifs se sont traduits par des fausses pistes (ex: cache navigateur) et des difficultés avec ses propres outils (`replace`, `web_fetch`, probablement lié à la sécurité). Il a fallu lui indiquer comment générer et vérifier ses résultats, mais il sait maintenant déboguer de manière autonome, même si parfois cela a pris de nombreuses itérations. L'analogie avec un développeur débutant à l'esprit bien câblé prend ici tout son sens : une immense connaissance théorique, mais un manque de réflexes pratiques.

Alors, aurais-je été plus vite sans lui ? Rien n'est moins sûr. Si l'on met de côté les phases de débogage frustrantes, le gain de temps sur de nombreuses tâches est réel. Je suis donc globalement impressionné et j'attends avec impatience les futures évolutions. Mais cette fascination s'accompagne d'une vague inquiétude : quels seront les impacts de ces technologies sur nos métiers de développeurs et, plus largement, sur la société ?