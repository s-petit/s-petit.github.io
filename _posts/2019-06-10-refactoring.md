---
layout: post
title: "Le refactoring, un éternel et indispensable recommencement"
tags: [refactoring, pratiques, craftmanship]
---



— INTRO — JE VEUX PARLER DE REFACTO

Dans cet article, j'aimerai parler d'un sujet qui m'est cher, la pratique du refactoring. Souvent mise en valeur par de nombreux ouvrages ou des communautés à forte sensibilité craft / clean code, au travers de discussions et autres katas, je me suis rendu compte au travers de mes expériences que cette pratique était bien trop peu pratiquée. Nous essaierons de comprendre pourquoi, ainsi que les conséquences d'un projet où le refactoring est peu pratiqué. Enfin, je donnerai quelques pistes faciles afin de 

— POURQUOI 1— 

1. Ben parce que je trouve que le gens n'en font pas assez (pas l'envie ou le réflexe)
2. Constaté lors de code review / pair / PR review / Diff historique GIT
3. Le code a besoin de refactoring a chaque itération, malgré le mal que l'on se donne
4. 
5.  La dette se creuse bien plus vite —> plus une consequence qu'un pourquoi

— POURQUOI 2— 

1. Manque de temps
2. Manque de culture
3. Parfois pas fun a faire (tests ?)



— QUAND LE FAIRE — 

— COMMENT FAIRE — simplement prendre du recul et se relire avant de livrer + un peu de boy scout si nécessaire...

Et pourtant, un refactoring régulier est indispensable dans la vie d'un projet. Sans cette gymnastique quotidienne, une code base peut rapidement devenir difficile à lire, à maintenir, à faire évoluer. Et c'est un cercle vicieux. Nos pairs anglophones utilisent parfois le terme "calisthenics" ou de kata et je trouve que ces analogies sont tout a fait pertinentes. Il faut faire quotidiennement cet exercice pour que notre code base garde la forme. Sinon on rentre dans un cerncle vicieux et cela va de mal en pis. 

Souvent, les bonnes pratiques sont appliquées par des développeurs ou des entreprises qui ont une culture et une sensibilité sur leurs bienfaits. Si le refactoring n'est pas spécialement encouragé par une entreprise, cela n'incitera pas spécialement un développeur qui n'a pas les bons réflexes ou qui n'a pas le temps

Qu'est ce que le refactoring au final ? ou QUAND plutot que quoi (sinon ca peut aller loin)

De nombreuses définitions existent, qui sont toutes intéressantes. Ma vision du refactoring est très basique : Au moment où l'on s'apprête à livrer du code

## Idées

**Souci:**

Dette technique qui se creuse quotidiennement

Des devs qui n'ont pas forcement le réflexe de refactorer si necessaire

Nouvelle feature, On se retrouve souvent ralenti ou embeté par le design existant, qui ne correspond plus exactement avec le nouveau besoin. Le code que l'on veut faire evoluer est parfois difficile a bouger -> design, couplage...

trop de pression / timing

**Symptomes**

Classes Longues

Classes difficiles a lire

Tests fragiles

Classes difficiles a faire evoluer

Duplication de code

Nommages pas clairs

Trop de  Responsabilite  - cohesion

Couplage fort

Code pas forcement placé dans la bonne classe

En general, on a pas le reflexe de créer de nouvelles classes

**Solution**

Warning compilateur

Warning IDE

Linters, Sonar...



**Comportement (le plus important)**

Culture / Qualité / Clean code

Se relire regulierement

Diffs / PR / code review

Se demander si on est satisfait de ce qu'on livre

Se demander si on ne peut pas ameliorer ce qu'on livre

**Prerequis :**

Couverture de tests complete

**Choses possibles a faire**

Splitter les classes



**Divers **

Ce qui est parfait aujourd'hui ne l'est plus demain

Attentions au tests, qui sont plus souvent delaisses et ont besoin de refacto

Ne pas refactorer si celui est trop couteux / risqué

Prendre en compte le contexte metier / delais / sprint

Faire le refacto en amont dans une PR separé pour ne pas tout mélanger

Parfois, le refacto peut aider, car rend la feature beaucoup plus simple et rapide a dev -> investissement gagnant

Dans chaque sprint / estimation on devrait inclure un cout de refactoring

regle du boy scout

encourager ses pairs a refactorer



pour refactorer il faut obligatoirement



\- recul

\- contexte





-> ne jamais refactorer de facon prématurée car on s'ecarte du besoin

-> on peut se perdre en route ou se tromper

-> optim prematurée

-> livrer la valeur métier d'abord

-> le faire a la fin, pendant ou juste avant la code review

-> TDD... refactos tres unitaires ou localisés OK mais pas trop car on peut se tromper de direction

----



Le Refactoring, une tache quotidienne

Le Refactoring, un combat de tous les instants

Dans cet article, j'explique pourquoi le refactoring est si important et si peu pratiqué, et qu il devrait etre pris en compte en continu