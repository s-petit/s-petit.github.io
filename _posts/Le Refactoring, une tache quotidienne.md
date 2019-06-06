Le Refactoring, une tache quotidienne

Le Refactoring, un combat de tous les instants



**Souci:**

Dette technique qui se creuse quotidiennement

Des devs qui n'ont pas forcement le réflexe de refactorer si necessaire



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