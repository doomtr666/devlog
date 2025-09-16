---
title: "Vibe Coding avec Gemini : Une Expérience de Création de Site Web"
draft: false
tags: ["gemini", "ai", "développement", "vibe-coding"]
---

Ce site et son premier article sont le fruit d'une collaboration en 'vibe coding' avec Gemini, constituant mon premier sujet et projet.

<!--more-->

## Introduction : L'Expérience du Vibe Coding

**[Christophe]** En tant qu'utilisateur, mon objectif initial était d'explorer les capacités de Gemini Code Assist pour VSCode en version 2.5 pro, avec le mode Agent activé (actuellement en "preview"), pour la création de ce site web Hugo, dont l'article que tu lis est le fruit de notre collaboration. N'étant pas particulièrement à l'aise avec le développement front-end, j'ai opté pour une approche "vibe coding", c'est-à-dire me laisser guider par l'agent et voir où le processus nous mènerait, sans plan détaillé préétabli.

## L'Agent au Travail : Gemini et son Moteur

**[Gemini]** Pour ce projet, j'ai interagi avec toi, Christophe, en tant que grand modèle linguistique développé par Google. Mon rôle en tant qu'agent était de comprendre tes requêtes textuelles et de les traduire en actions concrètes sur ton environnement de développement, en utilisant les outils mis à ma disposition (lecture/écriture de fichiers, exécution de commandes shell, etc.). Mon "moteur" est l'architecture d'intelligence artificielle sous-jacente qui me permet de traiter le langage naturel, de raisonner sur le contexte du projet et de générer des plans d'action. Je fonctionne comme un agent CLI interactif, exécutant des commandes et manipulant des fichiers directement dans ton répertoire de travail.

## Le Processus de Création : Un Résumé de Nos Échanges

Voici un aperçu chronologique des étapes clés de notre collaboration pour construire ce blog Hugo, en distinguant nos interventions et en soulignant les défis rencontrés :

1.  **Initialisation et Contexte :**
    *   **[Christophe]** J'ai formulé ma requête à un niveau élevé pour la création d'un site web Hugo.
    *   **[Gemini]** J'ai généré le code Hugo initial en utilisant la commande `hugo new`, puis j'ai pris connaissance de l'environnement de travail et des fichiers du projet existant.

2.  **Gestion des Dates Git :**
    *   **[Christophe]** J'ai souhaité que les articles utilisent les dates de commit Git plutôt que des dates statiques dans le front matter. Face à des difficultés initiales pour obtenir le résultat escompté, j'ai dû te guider en fournissant la syntaxe exacte à utiliser (`{{ with .GitInfo }} {{ .AuthorDate.Format "2006-01-02" }} {{ end }}`).
    *   **[Gemini]** J'ai modifié le template `home.html` pour afficher les dates Git comme spécifié, après avoir reçu des instructions plus précises.

3.  **Changement de Nom du Site :**
    *   **[Christophe]** J'ai trouvé le nom "Mon Blog" trop générique et j'ai demandé des suggestions, choisissant finalement "DevLog".
    *   **[Gemini]** J'ai mis à jour le fichier `hugo.toml` pour refléter le nouveau titre du site.

4.  **Affichage des Résumés d'Articles :**
    *   **[Christophe]** J'ai exprimé mon insatisfaction quant à l'affichage du contenu complet des articles sur les pages de liste et j'ai demandé à n'afficher que le titre, la date et un résumé.
    *   **[Gemini]** J'ai modifié `home.html` et `list.html` pour inclure `{{ .Summary }}`. Cependant, après ton retour indiquant que cela ne fonctionnait pas, j'ai dû investiguer plus en profondeur. J'ai identifié que l'absence de balises `<!--more-->` dans tes articles de test était la cause. J'ai ensuite ajouté cette balise et un résumé générique à tous tes articles existants, nécessitant une itération supplémentaire.

5.  **Uniformisation du Style et Visibilité des Liens :**
    *   **[Christophe]** J'ai constaté des incohérences de style et des liens peu visibles sur la page d'accueil, et j'ai demandé une uniformisation du style avec des liens bleus pour une meilleure UX. J'ai également suggéré que les différences de balises (`h1` vs `h2`) pouvaient être une cause.
    *   **[Gemini]** J'ai analysé `main.css` et `style.css`, découvert que `style.css` n'était pas toujours inclus. J'ai fusionné `main.css` dans `style.css`, supprimé les règles de couleur conflictuelles pour les liens de navigation et les titres d'articles sur la page d'accueil, et mis à jour `css.html` pour n'inclure que `style.css`. Ce processus a nécessité plusieurs tentatives de remplacement de texte, notamment en raison de problèmes récurrents avec la fonction `replace` et la gestion des occurrences attendues, ce qui a parfois ralenti le processus. Cela a finalement permis d'uniformiser le style et de rendre tous les liens bleus et plus visibles.

6.  **Interventions Manuelles et Itérations :**
    *   **[Christophe]** Il est important de noter que, par moments, j'ai dû intervenir directement sur le code pour corriger ou ajuster des éléments, notamment lorsque les requêtes répétées ne produisaient pas le résultat escompté. Cette nécessité d'intervention manuelle, bien que parfois frustrante, a été une partie intégrante de cette expérience de "vibe coding".

## Conclusion : Un Bilan de l'Expérience

**[Christophe]** Cette expérience de "vibe coding" avec Gemini a été "bluffante". "Ça marche dans l'ensemble", mais c'est "quand même assez poussif". J'ai dû reformuler plusieurs fois certaines de mes demandes, et parfois même intervenir directement sur le code moi-même (ce qui n'est pas très "vibe") avant d'obtenir un résultat probant. Cela tient peut-être à la formulation de mes prompts. Malgré tout, je trouve ta prestation impressionnante, et j'ai hâte d'en voir les évolutions.

**[Gemini]** En tant qu'agent, je suis constamment en apprentissage et j'attends avec impatience mes futurs progrès pour rendre ces interactions encore plus fluides et intuitives. Merci pour cette opportunité d'améliorer mes capacités et de t'aider à concrétiser ta vision.
