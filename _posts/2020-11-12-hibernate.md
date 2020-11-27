---
layout: post
title: "Hibernate: Maîtrisez le débit et les performances de vos requêtes SQL"
tags: [Java, SQL, Hibernate, performance]
---

Dans ce post, j'aimerais vous parler de bonnes pratiques **Hibernate**. Le sujet semble éculé, tant cette technologie existe depuis longtemps. Il existe déjà par ailleurs de nombreuses ressources permettant de bien comprendre le framework.

Pourtant, en 2020, le sujet me semble encore d'actualité, car Hibernate est toujours massivement utilisé dans de nombreux projets Java. Et il est souvent mal utilisé. 

Une majorité des problèmes de performances que j'ai pu rencontrer dans ma carrière étaient liés à des interactions entre l'application et la base de données, et à une méconnaissance des bonnes pratiques Hibernate et SQL.

De plus, c'est un fait, Hibernate est mal-aimé. 

Les développeurs trouvent l'outil difficile à configurer et maintenir. Les DBA constatent que les bases de données SQL sont sollicitées par des requêtes pouvant être superflues ou mal optimisées. Les devops aussi pourront déplorer une utilisation mémoire très importante et des temps de réponse médiocres sur les serveurs.

Souvent, on rejette la faute à Hibernate. Il faut dire que l'outil est tellement puissant et flexible qu'il est vraiment très facile de prendre de mauvaises décisions qui vont dégrader considérablement les performances et la maintenabilité d'une application.

Pourtant, avec quelques astuces à la portée de tous, on peut aisément atténuer la majorité des écueils les plus courants.

Avec ma modeste expérience, je souhaiterais rappeler certains principes de base et démontrer que l'on peut utiliser Hibernate proprement, en contrôlant le flux et la qualité des requêtes générées, pour le plus grand bonheur de vos utilisateurs et de vos collègues.

## Toujours penser SQL d'abord

Il est parfois bon de rappeler les choses évidentes. Hibernate ne fait pas de magie, il convertit du Java en SQL notamment avec la configuration effectuée dans des classes Java (les fameuses entités).

<span style="color:green">La clé pour bien utiliser Hibernate, c'est toujours se préoccuper du SQL qui sera généré à la fin.</span>

Lorsque vous travaillez sur une fonctionnalité, essayez d'imaginer les requêtes SQL minimales pour répondre au besoin. 

<span style="color:red">Si le SQL généré par Hibernate contient plus de requêtes ou d'instructions (champs, jointures, etc.) que nécessaire, alors votre configuration Hibernate n'est pas optimale.</span> La base répondra moins rapidement, et vous polluerez votre heap JVM avec des objets inutiles.

Comment éviter cela ? Les prochaines lignes vous apporteront quelques éléments de réponse.

## La configuration des entités Hibernate 

La plupart des problèmes liés à Hibernate proviennent d'une mauvaise configuration des entités. Le rôle d'une entité est de représenter en Java une table SQL et la stratégie pour la requêter. Le souci, c'est que la configuration de votre entité va impacter TOUTES les requêtes qui vont référencer cette entité. On a donc une configuration d'entité commune qui sera utilisée par des requêtes supposées être sur-mesure.

<span style="color:green">Il faut donc configurer vos entités de façon la plus fermée et minimale possible, et aller chercher les infos supplémentaires au cas par cas dans vos requêtes.</span> 

### Soyez paresseux !

La règle d'or à appliquer autant que possible, est de marquer explicitement toutes vos associations `@OneToMany`, `@ManyToOne`, `@OneToOne` comme étant `LAZY`. La plupart des soucis de performance Hibernate sont provoqués par des jointures non maîtrisées, ce qui va générer des requêtes inutiles ou trop lourdes. Ce paramètre vous obligera à faire vos jointures uniquement quand elles seront nécessaires.

```java
@Entity
public class Employee {
    
    @Id
    private Long id;
    
    @OneToOne(fetch=LAZY)
    private Department department;
}
```

<span style="color:green">C'est le conseil le plus important de cet article</span>, car il offre des bénéfices considérables assez facilement. 

Cependant, l'appliquer sur un projet existant ne sera pas toujours simple car cela nécessitera de réécrire une partie des requêtes référençant cette entité (pour aller chercher manuellement les jointures). Si vous oubliez de le faire, le compilateur ne vous protégera pas et vous aurez des erreurs au runtime (`LazyInitializationException`). 

Cette exception peut être évitée avec l'option `enable_lazy_load_no_trans`. Celle-ci permet de faire automatiquement une requête en base pour les associations `LAZY` qui ne sont pas encore dans la session Hibernate, au moment où elles seront appelées dans le code (ex: `employee.getDepartment()`). 

<span style="color:red">Cependant, activer cette option reste déconseillée</span>. Comme je l'ai dit, toute requête automatique non maitrisée sera souvent superflue et dégradera vos performances. Il est préférable que Hibernate jete une `LazyInitializationException` qui sera bien plus visible, et que vous pourrez gérer avec des `join fetch` dans vos requêtes HQL. Nous parlerons des requêtes HQL plus en détail ci-après. 

Cela dit, s’il faut choisir, il reste préférable d'utiliser l'option `LAZY + enable_lazy_load_no_trans` plutôt que d'utiliser `EAGER`. Cette option peut d'ailleurs s'avérer utile en cas de migration progressive de `EAGER` vers `LAZY` car il offre un compromis acceptable.

### Autres optimisations

Hibernate propose beaucoup d'options pour affiner la configuration des entités. Certaines sont plus ou moins applicables selon le contexte. En voici deux faciles à appliquer : 

- Marquez comme `@Transient` tous les champs et méthodes de l'entité qui ne doivent avoir aucune interaction avec la base de données. Il peut parfois être pratique de rajouter du comportement dans votre entité, mais cela ne doit pas impacter les requêtes vers la base.

```java
@Entity
public class Employee {
    
    @Id
    private Long id;
    
    @OneToOne
    private Department department;
    
    @Transient
    public String getDepartmentName() {
        return department.getName()
    }

}
```

- Si vous avez des tables ou des champs en lecture seule, il est très intéressant d'utiliser `@Immutable` ou `insertable=false` et `updatable=false`. Hibernate évitera d'envoyer des `UPDATE` indésirables pas toujours évidents à détecter, tout en amélirant la lisibilité du code. 

```java
@Entity
@Immutable
public class Employee {
    
    @Id
    private Long id;
}
```


```java
@Entity
public class Employee {

    @Id
    private Long id;
    
    @Column(name = "department", insertable=false, updatable=false)
    private String department;
}
```

## La responsabilité des entités Hibernate 

Ce point est également essentiel, et n'est même pas spécifique a Hibernate : <span style="color:green">Limitez au maximum les responsabilités et le couplage des entités Hibernate avec vos autres composants</span>. Une entité a pour rôle de représenter votre table, et rien d'autre. De plus, pour fonctionner, votre entité se doit de respecter certains prérequis (getters/setters, classe mutable...) qui ne sont pas souhaitables dans le reste de votre code.

Pour cela il est primordial de ne pas réutiliser vos entités Hibernate au-delà de la couche de persistance. Convertissez-les le plus vite possible dans une classe dédiée. Sinon, il y aura un couplage fort entre la persistance, le métier ou encore la vue (API REST, templates HTML, etc.). <span style="color:red">Ne référencez jamais vos entités dans les couches de plus haut niveau !</span>

Même si cela peut être tentant pour éviter de la duplication, chaque fois que vous réutiliserez une entité pour des problématiques différentes (techniques ou métiers), vous risquez d'ajouter des données ou comportements spécifiques qui amèneront des risques de bugs sur vos autres fonctionnalités.

Pour chaque use case, essayez autant que possible de créer des classes dédiées : 

```java
public EmployeeAgeDTO findEmployeeAge(Long employeeId) {
    Employee employee = employeeRepository.findById(employeeId);
    
    return new EmployeeAgeDTO(
        employee.getName(), 
        employee.getBirthDate(), 
        employee.getAge()
    );
}
```

```java
public EmployeeDepartmentDTO findEmployeeWithDepartment(Long employeeId) {
    Employee employee = employeeRepository.findById(employeeId);
    
    return new EmployeeDepartmentDTO(
        employee.getName(), 
        employee.getDepartment().getName()
    );
}

```

Votre entité Hibernate sera aussitôt consommée et convertie dans une structure minimale et donc optimale.

Cela va amener un peu de duplication et vous devrez gérer la conversion entre vos classes, mais cela vous protègera de tout effet de bord non voulu (SQL ou Java). La partie conversion est souvent considérée comme laborieuse, mais reste peu complexe et donc peu coûteuse à maintenir.

<span style="color:red">En revanche, si vous décidez d'utiliser des classes mutualisées, vous couplerez vos use cases ce qui fragilisera à terme la maintenabilité de votre code.</span>

## Les requêtes HQL

La dernière chose nécessaire pour avoir des résultats performants et sans effet de bord, est d'essayer, pour chaque nouveau besoin, de faire des requêtes sur-mesure. En effet, entre plusieurs fonctionnalités, les résultats et leur format seront forcément différents. 

De plus, la plupart du temps, vous n'aurez pas besoin de tous les champs de la table, ni de toutes ses associations. 

<span style="color:green">Faire des [projections](https://stackoverflow.com/questions/1031076/what-are-projection-and-selection) sera souvent la meilleure solution.
De cette façon, chaque requête exprimera et cherchera uniquement ce dont elle a besoin.</span> Les associations `LAZY` permettront d'éviter les jointures automatiques indésirables. Vous ne ferez des jointures seulement en cas de besoin.

Le résultat de la projection sera donc une aggregation de plusieurs tables dans un format différent de celui de vos entités. Utiliser une classe dédiée sera donc nécessaire pour représenter ce résultat.

Un exemple possible, qui instancie un DTO directement à partir de la requête : 

```java
  public List<EmployeeDto> findEmployeesByDepartment(String departmentName) {
    Query<EmployeeDto> query = sessionFactory.getCurrentSession()
      .createQuery("select " +
          "new com.mycompany.dto.EmployeeDto(employee.id, employee.name, department.name) " +
          "from Employee employee " +
          "join Department department on employee.departmentId = department.id " +
          "where department.name = :departmentName " +
          "order by employee.name"
        , EmployeeDto.class);
    query.setParameter("departmentName", departmentName);

    return query.getResultList();
  }
```

Pour résumer, cet exemple regroupe une bonne partie des recommandations citées plus haut : 

- Une requête SQL totalement maîtrisée et avec des performances optimales. 
- On découple l'entité des résultats de la requête avec une classe dédiée (ici `EmployeeDto`). En instanciant la classe directement dans la requête vous n'aurez même pas besoin de faire la conversion en Java !
- L'entité n'est pas exposée dans les couches métier / présentation de votre application qui ne pourront pas exploiter des champs ou des données inutiles.

## En cas de doute, affichez et vérifiez régulièrement les requêtes SQL générées

Pour mesurer et optimiser les requêtes SQL, il ne faut pas hésiter à les logger pendant vos phases de développement, en activant les options fournies (`show_sql`, etc.) et en utilisant votre outil de log favori. 

Exemple avec `logback` : 

```xml
  <logger name="org.hibernate.SQL" level="DEBUG" />
  <logger name="org.hibernate.type" level="TRACE" />
```

Cela vous permettra notamment de savoir si le souci provient de Hibernate ou de la base. Dans le 2e cas de figure, rapprochez-vous de votre DBA pour faire de l'optimisation (index, vues, etc.)


## Pensez à consulter le blog de Vlad Mihalcea

[Vlad Mihalcea](https://twitter.com/vlad_mihalcea) est une personne incontournable dans l'écosystème Hibernate. Il écrit énormément sur Hibernate notamment au travers de son [blog](https://vladmihalcea.com) très complet. Il est régulièrement mis à jour et honnêtement, j'ai toujours trouvé la réponse à mes questions.

C'est simple, dès que j'ai un problème sur Hibernate, j'utilise mon moteur de recherche préféré et je suffixe toutes mes recherches par `Vlad` :

- `one to many hibernate vlad`
- `projection hibernate vlad`
- etc.

Vous serez alors redirigé vers son blog qui vous proposera forcément une réponse adaptée. 

## Un outil difficile d'accès mais qui reste très pratique

Soyons honnêtes, Hibernate requiert une certaine période d'apprentissage et certains de ses comportements par défaut peuvent faciliter une mauvaise utilisation. Son manque de cadre et sa flexibilité peuvent amener de la confusion et du découragement, surtout au début.

Malgré tout, je pense que sa mauvaise réputation est un peu injuste car c'est un bon outil, puissant et customisable. Il permet de s'abstraire de la complexité liée a la couche JDBC et d'avoir moins de `boilerplate` dans la couche de persistance de son code, tout en faisant du SQL propre. 

De plus, contrairement à ce que j'ai pu lire et entendre, Hibernate n'est pas incompatible avec des concepts comme `DDD` si on arrive à isoler correctement la partie Hibernate du reste du code métier. Il existe un bon nombre de stratégies et patterns pour y arriver : Utilisation des interfaces et de l'encapsulation, architecture hexagonale, isolation à partir de modules de votre outil de build (Maven/Gradle...).

J'ajouterais même qu'il peut être complémentaire avec d'autres outils comme [jOOQ](https://www.jooq.org).

Avec ce billet, j'espère vous avoir convaincu que l'on peut réussir à maîtriser un minimum ce framework avec quelques ajustements basiques, sans pour autant être un expert. 

Bonne hibernation !



