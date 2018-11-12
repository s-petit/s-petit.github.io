---
layout: post
title: "Programmation Fonctionnelle et Monads en Java"
tags: [programmation fonctionnelle, java]
---

La programmation fonctionnelle (FP) a longtemps occupé une place relativement marginale dans le monde de l'entreprise, dominé par les langages impératifs (C, Java, C#, PHP...). Cependant depuis quelques années, ce paradigme de programmation se démocratise et gagne en popularité. Les projets en Scala se multiplient, et de nombreux langages impératifs traditionnels tels que Java ou Javascript s'enrichissent d'APIs se basant sur un certain nombre de concepts issus de la programmation fonctionnelle. Dans l'écosystème Java, la sortie du JDK 8 en 2014 et l'implémentation des lambdas a grandement contribué à populariser une composante majeure de la FP.

L'ajout progressif de concepts de la FP dans nos bonnes vieilles applications d'entreprise est une très bonne chose, et se combine plutôt bien avec les langages orientés Objet (OOP). Manipuler des fonctions pures, favoriser la récursivité dans nos algorithmes, retarder l'évaluation des instructions ou encore manipuler des objects immutables favorise l'écriture de programmes plus propres et mieux optimisés tout en limitant les effets de bord.

## Encourager la programmation fonctionnelle grâce à des structures dédiées

Lorsque l'on s'intéresse un peu a la FP et l'OOP, on finit par entendre parler des **Monads** et de ses dérivées **Functors** ou **Applicatives**. Les Monads sont des structures de données répondant à un certain nombre de lois mathématiques (identité, associativité). 

Pour donner la définition la plus large possible d'une Monad, on peut dire que ce sont des structures de données, avec un **contexte** et des comportements qui seront applicables sur les valeurs qu'elles encapsulent. Elles vont également permettre l'application et la composition de fonctions pures dans nos bons vieux langages impératifs orientés objets comme Java. 

Elles apportent un cadre qui va permettre d'encourager le développeur à bénéficier des forces de la FP, et ainsi de limiter au maximum les effets de bords classiques amenés notamment par des changements d'états ou des scopes de variables non judicieux.  

Par exemple, avec Optional de Java 8 (Spoiler : Optional est une représentation de la Monad `Maybe`) :

```java
Optional<String> name = Optional.of(getName());
```

Ici, notre instance d'Optional contient non seulement la valeur de ma String, mais propose un certain nombre de comportements supplémentaires. Selon le contexte, `getName()` pourra contenir ou non une valeur. 

**Les Monads permettent donc d'ajouter du contexte a des valeurs ou des objets, tout en permettant un certain nombre d'opérations fonctionnelles telles que `map`, `flatMap`, `orElse`...**

Dans cet article, je ne vais pas m'épancher plus sur les règles et avantages de la programmation fonctionnelle, ou encore sur la définition et règles d'une Monad, car de nombreux excellents articles existent déja sur le sujet. Je vous invite d'ailleurs à consulter les articles dans la section [Bibliographie](#bibliographie) pour plus de détails.

## Du coup, quel est le but de cet article ?

Une fois que l'on a plus ou moins compris la théorie, il peut être difficile de comprendre réellement comment et pourquoi les appliquer, notamment dans un langage comme Java. En effet, les Monads restent un concept, et ont un niveau d'abstraction assez élevé.

Je vais donc tenter d'expliquer ce que peuvent apporter la FP et les Monads avec un cas pratique simple en Java, et pourquoi utiliser plus souvent ces pratiques dans nos développements au quotidien peut être intéressant.

Cet exemple basique, partira d'une implémentation classique, puis nous y ajouterons progressivement des Functors, des Applicatives et enfin des Monads pour y exposer leurs bénéfices et leurs limites.

## Cas d'utilisation

L'exemple tournera autour d'une classe POJO `Item`, qui représente un article d'un site ecommerce, avec son prix.

Ce POJO sera implémenté avec des champs privés et des getters/setters pour tous les champs. Cette implémentation est pour moi un [code smell](https://fr.wikipedia.org/wiki/Code_smell) dans la plupart des cas (hors frameworks type Hibernate ou Jackson qui en ont besoin), mais elle est malheureusement la plus courante. 

```java
public class Item {
    private String name;
    private Price price;
    public Item(String name, Price price) {
        this.name = name;
        this.price = price;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Price getPrice() {
        return price;
    }
    public void setPrice(Price price) {
        this.price = price;
    }

    static class Price {
        private String currency;
        private Integer amount;
        public Price(String currency, Integer amount) {
            this.currency = currency;
            this.amount = amount;
        }
        public String getCurrency() {
            return currency;
        }
        public void setCurrency(String currency) {
            this.currency = currency;
        }
        public Integer getAmount() {
            return amount;
        }
        public void setAmount(Integer amount) {
            this.amount = amount;
        }
    }
}
```

Notre cas d'utilisation sera le suivant : 

> En tant que vendeur, je veux appliquer un prix promotionnel temporaire avec une reduction de 5$ et l'afficher sur ma page web.
 
## Exemple d'implémentation classique

Une implémentation classique pourrait être la suivante:


```java
@Test
public void reduce_and_display_price_imperative_style() {
    Item item = new Item("book", new Item.Price("$", 50));

    item.getPrice().setAmount(item.getPrice().getAmount() - 5);
    String result = "price: " + item.getPrice().getAmount();

    assertThat(item.getPrice().getAmount()).isEqualTo(45); //mutated Price instance
    assertThat(result).isEqualTo("price: 45");
}
```


Le style du code utilisé dans cet exemple est volontairement grossier, mais malheureusement assez courant. Il y a bien d'autres facons d'implementer plus proprement ce cas d'utilisation. 

On y voit des code smells, tels que des instances qui ont muté alors que ce n'est pas necessaire, ou encore de la répétition (`item.getPrice().getAmount()`). En plus d'être améliorable, ce code est plus exposé aux bugs et divers effets de bords.

Dans l'exemple suivant, je vais amener la notion de `Functor`. Ce Functor va amener un contexte supplémentaire sur notre montant, et encourager l'utilisation de fonctions sur notre Item.

Un Functor, c'est essentiellement une méthode `fmap` (ou simplement `map`), qui va nous permettre d'appliquer des fonctions au sein de notre structure de données.

Pour garder cet article le plus simple possible, nous allons utiliser un Functor volontairement minimaliste qui ne sait faire que des fmap : `MyFunctor`

## Functors

Un **Functor** en Java est une implémentation de cette interface :

```java
@FunctionalInterface
public interface Functor<A, F extends Functor> {
    <B> Functor<B, F> fmap(Function<? super A, ? extends B> fn);
}
``` 


Notre MyFunctor ressemble donc à ça...

```java
public class MyFunctor<T> implements Functor<T, MyFunctor> {
    private final T value;
    private MyFunctor(T value) {
        this.value = Objects.requireNonNull(value);
    }
    public static <T> MyFunctor<T> of(T value) {
        return new MyFunctor<>(value);
    }
    public T get() {
        return value;
    }
    @Override
    public <B> MyFunctor<B> fmap(Function<? super T, ? extends B> fn) {
        B apply = fn.apply(value);
        return MyFunctor.of(apply);
    }
}
```   

... Et va nous permettre de faire des traitements comme ceci :

```java
@Test
public void reduce_and_display_price_functional_style() {
    Item item = new Item("book", new Item.Price("$", 50));

    MyFunctor<Item> functor = MyFunctor.of(item);

    String result = functor
        .fmap(Item::getPrice)
        .fmap(Item.Price::getAmount)
        .fmap(amount -> amount - 5)
        .fmap(p -> "price: " + p)
        .get();

    assertThat(item.getPrice().getAmount()).isEqualTo(50); // we prevent Price mutation
    assertThat(result).isEqualTo("price: 45");
}
``` 

Nous avons donc exactement le même résultat qu'avant, sauf que ce dernier a été calculé avec un enchaînement de fonctions pures.

Nous bénéficions donc de la puissance de la FP (immutabilite, limitation des effets de bord, parallélisation, composition...) pour un code plus propre et moins exposé aux erreurs.

Il est malgré tout important de préciser que de faire de la FP n'est pas une [balle d'argent](https://fr.wikipedia.org/wiki/Balle_d%27argent#Sens_figur.C3.A9). Il est parfaitement possible de ne pas l'utiliser convenablement, et d'avoir des alternatives impératives bien meilleures. Mais je le répète, cela apporte un cadre qui encourage plus volontiers à adopter une approche plus propre et robuste.

C'est exactement comme la classe `Optional` de Java 8. Très controversée, elle est pourtant intéressante lorsqu'elle est utilisée judicieusement. Au contraire, une mauvaise utilisation peut faire bien pire que mieux.


Cependant, les Functors montrent leurs limites, lorsque l'on souhaite mapper avec une fonction qui est elle-même boxée dans un Functor :

```java
@Test
public void functors_cannot_fmap_on_functors() {
    Item item = new Item("book", new Item.Price("$", 50));

    MyFunctor<Item> functor = MyFunctor.of(item);
    // we want our discount function to be wrapped into a functor 
    MyFunctor<Function<Integer, Integer>> discount = MyFunctor.of(amount -> amount - 5);

    /*   String result = functor
                .fmap(Item::getPrice)
                .fmap(Item.Price::getAmount)
                .fmap(discount) // does not compile
                .fmap(p -> "price: " + p)
                .get();*/

    // Could not map MyFunctor(50) with MyFunctor(amount -> amount - 5), only accepts simple Functions, but not boxed Functions
    }
```  

Cela ne compile pas. `fmap` n'acceptant que les `Function` simples, les Functors ne savent pas comment mapper appliquer une fonction boxée dans un Functor. C'est dommage, car `Function` est un type Java comme les autres, et peut parfaitement être wrappé dans notre Functor pour profiter de son contexte. 

## Applicatives

C'est à ce moment que les **Applicatives** entrent en jeu. Grâce à leur méthode `apply`, Les Applicatives vont permettre d'appliquer des fonctions qui sont elles memes boxées dans un Applicative. Tout en gardant la puissance apportées par le `fmap` des Functors, car un **Applicative est un Functor**. 

```java 
 public interface Applicative<A, App extends Applicative> extends Functor<A, App> {
     <B> Applicative<B, App> apply(Applicative<Function<? super A, ? extends B>, App> appFn);
 }
 
```  
```java 
public class MyApplicative<T> implements Applicative<T, MyApplicative> {
     // In an Applicative, T could be a Function
     private final T value;
     private MyApplicative(T value) {
         this.value = Objects.requireNonNull(value);
     }
     public static <T> MyApplicative<T> of(T value) {
         return new MyApplicative<>(value);
     }
     public T get() {
         return value;
     }
     //applicative apply : A<T> -> A(T -> U) -> A<U>
     @Override
     public <B> MyApplicative<B> apply(Applicative<Function<? super T, ? extends B>, MyApplicative> appFn) {
         //appFn is an Applicative which contains a function
         Function<? super T, ? extends B> applicativeFunction = ((MyApplicative<Function<? super T, ? extends B>>) appFn).value;
         return MyApplicative.of(applicativeFunction.apply(this.value));
     }
     //functor fmap : F<T> -> (T -> U) -> F<U>
     // fmap is a subset of apply
     @Override
     public <B> MyApplicative<B> fmap(Function<? super T, ? extends B> fn) {
         return apply(MyApplicative.of(fn));
     }
 }

```  


Grâce aux Applicatives, nous allons pouvoir faire passer le test qui nous bloquait précédemment :



```java 
    @Test
    public void applicative_can_apply_boxed_functions() {
        Item item = new Item("book", new Item.Price("$", 50));

        MyApplicative<Item> applicative = MyApplicative.of(item);
        MyApplicative<Function<? super Integer, ? extends Integer>> discountApplicative = MyApplicative.of(amount -> amount - 5);

        // can also fmap because Applicative is a functor - fmap is a subset of apply.
        String result = applicative
                .fmap(Item::getPrice)
                .fmap(Item.Price::getAmount)
                .apply(discountApplicative)
                .fmap(p -> "price: " + p)
                .get();

        assertThat(result).isEqualTo("price: 45");
    }
``` 

`apply` acceptant en paramètre un Applicative, nous pouvons désormais y wrapper à loisir des valeurs et des fonctions, et les combiner. Cela commence à devenir intéressant ! Malheureusement, `apply` possède aussi sa limitation :



```java 
    @Test
    public void applicative_cannot_apply_on_functions_which_return_applicatives() {
        Item item = new Item("book", new Item.Price("$", 50));

        MyApplicative<Item> applicative = MyApplicative.of(item);
        MyApplicative<Function<? super Integer, ? extends Applicative>> discountApplicative = MyApplicative.of(amount -> MyApplicative.of(amount - 5));

        // can also fmap because Applicative is a functor - fmap is a subset of apply.
        String result = applicative
                .fmap(Item::getPrice)
                .fmap(Item.Price::getAmount)
                .apply(discountApplicative)
                .fmap(p -> "price: " + p)
                .get();

        assertThat(result).isNotEqualTo("price: 45"); // result is like price: com.spe.applicative.MyApplicative@2a33fae
    }   
``` 

Aie ! Dans notre test ci-dessus, on tente d'utiliser `apply` sur un Applicative qui contient **une fonction qui retourne un autre Applicative**. Cela compile, mais cela interrompt l'enchainement des fonctions, puisque les prochains `fmap` ou `apply` seront appliqués sur un `Applicative<Integer>` plutôt qu'un `Integer`.
Dans notre exemple, on pourrait s'en sortir avec un `discountApplicative.get()` mais la présence d'une méthode qui retourne la valeur brute contenue dans l'Applicative n'est pas toujours garanti et pertinent. 


## Monads

Les **Monads** sont la solution à tous nos problèmes :

```java 
public interface Monad<T> {

    // monads have 3 fundamental operations :
    // return : T1 -> M<T1>
    // unbox : M<T> -> T
    // bind : M<T> -> (T -> M<U>) -> M<U>

    // Monad unbox : M<T> -> T
    public T get();
    // monad bind : M<T> -> (T -> M<U>) -> M<U>
    <U> Monad<U> bind(Function<? super T, Monad<U>> mapping);
    // a monad is a functor so it can also fmap
    <U> Monad<U> fmap(Function<? super T, ? extends U> fn);
}  
``` 

La méthode `bind` prend la valeur boxée et retourne une nouvelle Monad avec une nouvelle valeur boxée. A présent, nous allons pouvoir à loisir chaîner toutes les fonctions que l'on souhaite, même si celles-ci renvoient une Monad du même type ou bien des Monads covariantes.

Nous allons illustrer ce comportement avec un Optional simplifié, avec 2 Monads : `MyPresentMonad` et `MyEmptyMonad` :


```java 
public interface MyOptionalMonad<T> extends Monad<T> {

}
```  
```java 
public class MyPresentMonad<T> implements MyOptionalMonad<T> {

    private final T value;

    private MyPresentMonad(T value) {
        this.value = Objects.requireNonNull(value);
    }
    // Monad return : T1 -> M<T1>
    public static <T> MyPresentMonad<T> of(T value) {
        return new MyPresentMonad<>(value);
    }
    // Monad unbox : M<T> -> T
    public T get() {
        return value;
    }
    // monad bind : M<T> -> (T -> M<U>) -> M<U>
    @Override
    public <U> Monad<U> bind(Function<? super T, Monad<U>> mapping) {
        return mapping.apply(value);
    }
    // fmap is a subset of bind
    @Override
    public <U> Monad<U> fmap(Function<? super T, ? extends U> fn) {
        return bind(fn.andThen(x -> MyPresentMonad.of(x)));
    }
}
```  
```java 
public class MyEmptyMonad implements MyOptionalMonad {

    public static final MyEmptyMonad INSTANCE = new MyEmptyMonad();

    private MyEmptyMonad() {
    }

    @Override
    public MyEmptyMonad bind(Function fn) {
        return INSTANCE;
    }
    @Override
    public MyEmptyMonad fmap(Function fn) {
        return INSTANCE;
    }
    @Override
    public MyEmptyMonad get() {
        return INSTANCE;
    }
}

``` 

Désormais, _sky is the limit_. Notre test qui bloquait précédemment répond à notre besoin :

```java
    @Test
    public void monad_can_bind_on_other_monads_of_same_type() {
        Item item = new Item("book", new Item.Price("$", 50));

        MyPresentMonad<Item> monad = MyPresentMonad.of(item);
        Function<Integer, Monad<Integer>> discountFunction = amount -> MyPresentMonad.of(amount - 5);

        // can also fmap because Monad is a functor - fmap is a subset of bind.
        Monad result = monad
                .fmap(Item::getPrice)
                .fmap(Item.Price::getAmount)
                .bind(discountFunction)
                .fmap(p -> "price: " + p);

        assertThat(result.get()).isEqualTo("price: 45");
    }
```

Nous pouvons désormais appliquer des fonctions sur :

- des Functions simples
- des Functions boxées dans des Monads
- des Monads qui contiennent des Functions retournant une Monad du même type, et même des Monads covariantes !


Ce dernier exemple, qui va renvoyer un `MyEmptyMonad`, l'illustre très bien :


```java
@Test
public void monad_can_bind_even_on_covariant_monads() {
    Item item = new Item("book", new Item.Price("$", 50));

    MyPresentMonad<Item> monad = MyPresentMonad.of(item);

    Function<Integer, Monad<Integer>> emptyFunction = amount -> MyEmptyMonad.INSTANCE;

    Monad emptyResult = monad
            .fmap(Item::getPrice)
            .fmap(Item.Price::getAmount)
            .bind(emptyFunction)
            .fmap(p -> "price: " + p);

    assertThat(emptyResult.get()).isEqualTo(MyEmptyMonad.INSTANCE);
}
```

Peu importe le type de fonction ou le type de Monad renvoyée, il est désormais possible d'appliquer les fonctions de notre choix, indépendamment du type de la Monad renvoyée.

## Faites plus de programmation fonctionnelle !

La programmation fonctionnelle prend de plus en plus de place, notamment en Java, et on comprend pourquoi. 

Au delà des lambdas, le JDK 8 propose son lot de Monads comme `Optional` ou `Stream`. Elles possèdent leur contexte propre (présence ou non de valeur, gestion des collections avec évaluation paresseuse), et permettent d'y appliquer des fonctions indépendamment de leur type ou du résultat de la précédente opération. Notamment grâce aux méthodes `map` (`fmap` des Functors) et `flatMap` (`bind` des Monads), accompagnées d'opérations spécifiques à leur contexte propre (`ifPresent` ou `orElse` pour les Optional)

Grâce à leur puissance et leur concision, ces 2 classes sont devenus incontournables pour les développeurs Java 8. Cela nous permet de réaliser à quel point il est intéressant de faire plus de programmation fonctionnelle et d'implémenter plus de Monads en Java.

Sachant cela, et même si [Java n'est pas le langage le plus adapté](#les-limites-de-java) pour profiter au maximum du potentiel des Monads, il y a largement de quoi faire pour rendre notre code plus propre encore, et cela vous inspirera peut-être pour créer vos propres Functors/Applicatives/Monads pour les besoins de vos projets et ainsi faire profiter vos applications métier de leur puissance indéniable !



## Bibliographie

Voici des articles que je recommande pour bien comprendre le sujet:

### Les Functors/Applicatives/Monads expliqués en images

en : <http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html>

fr : <http://www.leonardmeyer.com/blog/2014/06/functors-applicatives-et-monads-en-images/>

### Mieux comprendre les Applicatives

<https://pbrisbin.com/posts/applicative_functors/>

### Mieux comprendre les Monads

<https://medium.com/@sinisalouc/demystifying-the-monad-in-scala-cc716bb6f534>

<http://scabl.blogspot.fr/2013/02/monads-in-scala-1.html>

### Implémenter des Monads en Java

<https://dzone.com/articles/functor-and-monad-examples-in-plain-java>

### Les limites de Java

<http://jnape.com/the-perils-of-implementing-functor-in-java/>

<https://stackoverflow.com/questions/35951818/why-can-the-monad-interface-not-be-declared-in-java>

<https://dzone.com/articles/whats-wrong-java-8-part-iv>

<https://www.sitepoint.com/how-optional-breaks-the-monad-laws-and-why-it-matters/>
