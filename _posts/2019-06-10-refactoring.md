---
layout: post
title: "Le refactoring, un éternel et indispensable recommencement"
tags: [refactoring, pratiques, craftmanship]
---



— INTRO — JE VEUX PARLER DE REFACTO

Dans cet article, j'aimerai parler d'un sujet qui m'est cher, le refactoring. Cette pratique, souvent mise en valeur par de nombreux ouvrages et communautés, a pour but d'améliorer la qualité de son code à posteriori, en améliorant notamment sa simplicité ou sa lisibilité. La finalité étant de rendre le code plus robuste et facile à faire évoluer. 



est quotidiennement, au travers de discussions et autres katas, je me suis rendu compte au travers de mes expériences que cette pratique était bien trop peu pratiquée / pas assez régulièrement. Nous essaierons de comprendre pourquoi, ainsi que les conséquences d'un projet où le refactoring est peu pratiqué. Enfin, je donnerai quelques pistes faciles afin de 

— POURQUOI 1— 

1. Ben parce que je trouve que le gens n'en font pas assez (pas l'envie ou le réflexe)
2. Constaté lors de code review / pair / PR review / Diff historique GIT
3. Le code a besoin de refactoring a chaque itération, malgré le mal que l'on se donne
4. 
5.  La dette se creuse bien plus vite —> plus une consequence qu'un pourquoi

**Souci:**

Dette technique qui se creuse quotidiennement

Des devs qui n'ont pas forcement le réflexe de refactorer si necessaire

Nouvelle feature, On se retrouve souvent ralenti ou embeté par le design existant, qui ne correspond plus exactement avec le nouveau besoin. Le code que l'on veut faire evoluer est parfois difficile a bouger -> design, couplage...

trop de pression / timing

— POURQUOI 2— 

1. Manque de temps + long + cher (mais investissment long terme)
2. Manque de culture
3. Parfois pas fun a faire (tests ?)
4. La peur de casser

- POURQUOI FAIRE - 

 ameliorer la qualité du livrable (lisibilité, maintenabilité)
 le souci du travail bien fait
 relecture
 prendre du recul sur ce que l'on a fait
 Gagner du temps lors des code review
 Attenuer la dette (on crée de la dette quoiqu'il arrive)
 Rendre la code base plus facile a gerer pour les futurs devs

 - JUSQU OU ALLER - bon et mauvais refacto, pieges a eviter.

 - Trop refactorer ( rapport temps / complexité / benefice)
 - Refactorer sur des hypotheses ou du futur
 - Refactorer avant d'avoir mis en place la solution basique qui marche, refactorer trop tot
 - La simplicité et la lisbilité avant tout

 -- SYMPTOMES D'UN CODE NON REFACTORE 

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


— QUAND LE FAIRE — 

En fin de développement, quand la fonctionnalité est fonctionnelle et testée
Juste avant, pendant la code review

Idéalement, il faut en faire tout le temps, en continu. 

Refactorer sur du present et non du passé et du futur. Tout est question de contexte.

Pourquoi cela n'a pas été fait par le passé ? Probablement une raison contextuelle
Ne pas penser au futur, seulement au présent
Penser a la solution technique ou au design avant d'avoir tout fait marcher fonctionnellement (D.Knuth optimisation prematuree)
Ne pas refacto avant d'avoir fini la fonctionnalité ! Un seul grain de sel ou imprevu dans votre perception de l'US peut rendre votre travail caduc (anecdote sur le refacto des ratios...)
-> Refacto pour la probable futur story dans 1 ou 2 sprint NON (le besoin aura peut etre changé ou été descopé + meilleur moyen de se planter pour les memes raison que si dessus) 

Etre conscient que le refacto est temporaire, il sera en partie caduc lorsqu'une nouvelle fonctionnalité impactera le code refactoré -> Changement de contexte
Le code sera quand meme bien plus facile a retoucher et ameliorer, et le refacto du refacto en sera plus facile

Si on a le temps bien sur


— COMMENT FAIRE — simplement prendre du recul et se relire avant de livrer + un peu de boy scout si nécessaire...


pour refactorer il faut obligatoirement

\- recul

\- contexte

**Prerequis :**

Couverture de tests complete


Y'a t-il une bonne facon de faire, comment faire pour refactorer ? Ai je les epaules pour le faire ?

Simplement prendre du recul, de la hauteur. Se relire
On trouve forcément des noms de methodes a ameliorer. Des choses a rendre plus lisibles, de la duplication a virer
!!!! ON REFACTO AUSSI LES TESTS !! -> Ce test est-il toujours pertinent ? Puis-je mutualiser des choses ?

Etape supérieure : Revoir le design du module fonctionnelle, faire de l'abstraction, faire un refacto qui touche des choses
hors scope de l'US, regle du boy scout (on trouve un truc moche facile a refaire)

Faut-il le faire ? Ca depend

- Depend du risque et de la criticité
- Depend du coût / complexité
- Tests obligatoires avant pour limiter les regressions


**Comportement (le plus important)**

Culture / Qualité / Clean code

Se relire regulierement

Diffs / PR / code review

Se demander si on est satisfait de ce qu'on livre

Se demander si on ne peut pas ameliorer ce qu'on livre

-- TIPS pour faire du bon refacto


**Solution**

Relecture

Warning compilateur

Warning IDE

Linters, Sonar...


**Choses possibles a faire**

Splitter les classes



faire des katas ? -> Gilded Rose



--- CONCLUSION

---------------------------------------------------------------------------------------





Et pourtant, un refactoring régulier est indispensable dans la vie d'un projet. Sans cette gymnastique quotidienne, une code base peut rapidement devenir difficile à lire, à maintenir, à faire évoluer. Et c'est un cercle vicieux. Nos pairs anglophones utilisent parfois le terme "calisthenics" ou de kata et je trouve que ces analogies sont tout a fait pertinentes. Il faut faire quotidiennement cet exercice pour que notre code base garde la forme. Sinon on rentre dans un cerncle vicieux et cela va de mal en pis. 

Souvent, les bonnes pratiques sont appliquées par des développeurs ou des entreprises qui ont une culture et une sensibilité sur leurs bienfaits. Si le refactoring n'est pas spécialement encouragé par une entreprise, cela n'incitera pas spécialement un développeur qui n'a pas les bons réflexes ou qui n'a pas le temps

Qu'est ce que le refactoring au final ? ou QUAND plutot que quoi (sinon ca peut aller loin)

De nombreuses définitions existent, qui sont toutes intéressantes. Ma vision du refactoring est très basique : Au moment où l'on s'apprête à livrer du code

## Idées









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

il m'arrive souvent d'avoir besoin de refactorer l'existant avant d'attaquer le dev. Il vaut mieux : Finir le dev, refactorer l'existant, et si possible, l'isoler dans un commit / PR séparé.






-> ne jamais refactorer de facon prématurée car on s'ecarte du besoin

-> on peut se perdre en route ou se tromper

-> optim prematurée

-> livrer la valeur métier d'abord

-> le faire a la fin, pendant ou juste avant la code review

-> TDD... refactos tres unitaires ou localisés OK mais pas trop car on peut se tromper de direction
-> refactorer, c'est preparer son code pour un futur changement, le rendre plus facile a retoucher. Mais pas faire de suroptimisation ! refactorer uniquement sur le contexte présent, jamais futur ! non plus sur des hypotheses

-- qualité du message de commit



----



Le Refactoring, une tache quotidienne

Le Refactoring, un combat de tous les instants

Dans cet article, j'explique pourquoi le refactoring est si important et si peu pratiqué, et qu il devrait etre pris en compte en continu
