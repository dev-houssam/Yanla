# Yanla

**Yanla** est un langage de programmation et un environnement d'exécution dédiés à la construction, la composition et l'exposition de systèmes intelligents.

Yanla permet de décrire de manière structurée :

* des modèles neuronaux ;
* leur structure ;
* leurs flux de calcul ;
* leur apprentissage ;
* leur inférence ;
* des tâches persistantes (`task`) ;
* les interactions entre plusieurs Neural ;
* la gestion de la mémoire et des ressources matérielles ;
* l'exposition de modèles et de services intelligents.

Yanla est conçu pour compiler vers **C**, afin de pouvoir exploiter largement les bibliothèques et infrastructures existantes pour le calcul CPU, GPU, la mémoire et les systèmes distribués.

---

## Vision

L'objectif de Yanla est de rendre la construction de systèmes intelligents accessible, tout en conservant la possibilité de contrôler les mécanismes bas niveau lorsque cela devient nécessaire.

Un développeur peut commencer par décrire un système de manière relativement simple :

```text
Neural TemperatureModel {

    structure {
        dense 16;
        relu;
        dense 8;
        relu;
        dense 2;
        softmax;
    }

    inference {
        predict(input);
    }
}
```

Puis descendre progressivement vers les détails de l'exécution :

```text
flow Forward {
    input
    | ...
    -> output;
}

flow Backward {
    gradient
    | ...
    -> weights;
}
```

Yanla cherche ainsi à conserver une continuité entre **modélisation**, **calcul**, **exécution**, **distribution** et **matériel**.

---

# Architecture générale

Yanla s'inscrit dans une architecture composée de plusieurs couches :

```text
                         Application
                              │
                              │ CORBA / RPC
                              ↓
                    ┌───────────────────┐
                    │      YAINDIS      │
                    │ Yanla Intelligent │
                    │     Dispatcher    │
                    └─────────┬─────────┘
                              │
                 ┌────────────┼────────────┐
                 ↓            ↓            ↓
             Yanla A      Yanla B      Modèle
                 │            │         externe
                 └────────────┼────────────┘
                              ↓
                       MultiNeural
                              │
                    ┌─────────┴─────────┐
                    ↓                   ↓
                   CPU                 GPU
```

Chaque couche possède une responsabilité particulière :

| Couche          | Responsabilité                                         |
| --------------- | ------------------------------------------------------ |
| **CORBA / RPC** | Communication avec les applications externes           |
| **YAINDIS**     | Coordination et distribution des services intelligents |
| **Yanla**       | Description et exécution des systèmes intelligents     |
| **MultiNeural** | Coordination de plusieurs Neural                       |
| **Runtime**     | Gestion de l'exécution                                 |
| **Backend C**   | Génération du code bas niveau                          |
| **CPU / GPU**   | Exécution matérielle                                   |

---

# Structure d'un Neural

Un `Neural` décrit un système intelligent.

```text
Neural TemperatureModel {

    structure {
        dense 16;
        relu;
        dense 8;
        relu;
        dense 2;
        softmax;
    }

    flow Forward {
        input
        | ...
        -> output;
    }

    flow Backward {
        gradient
        | ...
        -> weights;
    }

    flow Validation {
        input
        | ...
        -> metrics;
    }

    inference {
        predict(input);
    }

    task TemperatureTask {

        expose {
            predict;
        }
    }
}
```

Le modèle est ainsi séparé en plusieurs concepts.

### `structure`

Décrit les composants du modèle.

### `flow`

Décrit les flux de calcul.

### `inference`

Décrit l'utilisation du modèle.

### `task`

Décrit le processus persistant donnant accès à une instance du modèle.

---

# Structure

La section `structure` représente la constitution du modèle.

```text
structure {

    dense 16;
    relu;
    dense 8;
    relu;
    dense 2;
    softmax;
}
```

La structure peut être vérifiée par le compilateur avant l'exécution des différents flux.

---

# Flux de calcul

Les `flow` décrivent les traitements effectués par le système.

Par exemple :

```text
flow Forward {
    input
    | ...
    -> output;
}
```

Un même Neural peut posséder plusieurs flux :

```text
flow Forward
flow Backward
flow Validation
```

Chaque flux peut avoir une responsabilité différente.

Cette séparation permet notamment de représenter explicitement la propagation avant, la rétropropagation ou la validation.

---

# Inférence

L'inférence décrit les opérations destinées à utiliser le modèle.

```text
inference {

    predict(input);
}
```

L'inférence est indépendante des mécanismes d'apprentissage.

Un modèle peut donc être entraîné, validé, puis utilisé à travers une interface d'inférence stable.

---

# Task

Une `task` représente un **processus persistant** associé à au moins une instance d'un modèle.

```text
task TemperatureTask {

    expose {
        predict;
    }
}
```

Une task constitue un point d'accès permanent au modèle pendant son exécution.

Elle peut ensuite être exposée à d'autres programmes par une couche de communication distante.

Par exemple :

```text
Application
    │
    │ appel distant
    ↓
TemperatureTask
    │
    ↓
TemperatureModel
```

Cela permet à un langage externe de communiquer avec un système Yanla sans connaître son implémentation interne.

---

# MultiNeural

Yanla permet à plusieurs `Neural` de coopérer.

```text
                     MultiNeural
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          Vision        Mémoire     Raisonnement
             │            │            │
             └────────────┼────────────┘
                          ↓
                     Contrôleur
```

Les Neural peuvent interagir entre eux.

Un Neural peut notamment observer ou utiliser un autre Neural et, lorsque cela est explicitement autorisé, agir sur ses paramètres.

Cette possibilité permet de construire des systèmes composés de plusieurs modèles spécialisés.

---

# Gestion de la mémoire

Yanla peut fournir un contrôle explicite de certaines ressources mémoire.

```text
memory.zone compute_buf size 1024 align 32;
memory.zone result_buf  size 256 align 32;
```

Une `memory.zone` permet de déclarer une zone mémoire destinée à un usage particulier.

Cette possibilité est notamment utile pour :

* les buffers de calcul ;
* les calculs GPU ;
* les systèmes embarqués ;
* les traitements nécessitant un contrôle précis de la mémoire ;
* les interactions avec des bibliothèques bas niveau.

---

# Compilation vers C

Yanla cible le langage C comme backend de compilation.

```text
Code Yanla
     │
     ↓
Analyse lexicale
     │
     ↓
Analyse syntaxique
     │
     ↓
AST
     │
     ↓
Analyse sémantique
     │
     ↓
IR Yanla
     │
     ↓
Générateur C
     │
     ↓
Compilateur C
     │
     ↓
Exécutable / Runtime
```

Le choix de C permet à Yanla de profiter de l'écosystème existant pour :

* le calcul numérique ;
* les CPU ;
* les GPU ;
* la mémoire ;
* le parallélisme ;
* les bibliothèques système ;
* les bibliothèques de calcul scientifique.

Yanla n'a donc pas besoin de réimplémenter toute la chaîne matérielle.

---

# RAG

Yanla peut composer des modèles déjà existants.

Un système RAG peut par exemple utiliser un modèle d'embedding et un modèle génératif déjà abstraits.

```text
Neural RAG {

    structure {

        embedding = EmbeddingModel;
        generator = LanguageModel;

        knowledge {
            documents;
            vectors;
        }
    }

    flow Retrieval {
        query
        | embedding
        | search(vectors)
        -> context;
    }

    flow Generation {
        query
        + context
        | generator
        -> answer;
    }

    inference {

        ask(query) {

            context = Retrieval(query);

            answer = Generation(
                query,
                context
            );

            return answer;
        }
    }

    task RAGTask {

        expose {
            ask;
        }
    }
}
```

Le système peut ensuite être interrogé par :

```text
RAGTask.ask("Qu'est-ce que le protocole H2 ?");
```

L'application appelante n'a pas besoin de connaître les détails internes du RAG.

---

# YAINDIS

**YAINDIS — Yanla Intelligent Dispatcher** constitue la couche de coordination entre les interfaces externes et les systèmes intelligents.

YAINDIS peut notamment gérer :

* la découverte des services ;
* l'état des modèles ;
* la disponibilité ;
* le routage des requêtes ;
* plusieurs instances ;
* plusieurs environnements Yanla ;
* les modèles externes ;
* les mécanismes de remplacement ;
* la communication entre différents systèmes Yanla.

```text
                    Application
                         │
                       CORBA
                         │
                         ↓
                    YAINDIS
                         │
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          Yanla A     Yanla B    Modèle externe
```

Le client conserve ainsi un point d'accès stable même si l'implémentation du service change.

---

# Modèles externes

Un modèle n'a pas nécessairement besoin d'être écrit en Yanla.

YAINDIS peut servir de point de connexion entre :

```text
Yanla Neural
      │
      ├── autre Yanla
      │
      ├── modèle C/C++
      │
      ├── service Python
      │
      └── autre système compatible
```

Le but est de permettre à Yanla de s'intégrer dans un écosystème existant plutôt que de créer un environnement fermé.

---

# CORBA

Une `task` Yanla peut être exposée comme un service distant.

Exemple :

```text
RAGTask.ask(...)
```

Une interface CORBA peut alors représenter cette capacité :

```idl
interface RAGTask {

    string ask(in string query);

};
```

Une application Java, C++, ou utilisant un autre langage compatible CORBA peut alors accéder au système sans connaître son implémentation interne.

---

# Distribution

Yanla est conçu pour pouvoir évoluer d'un modèle local vers des systèmes distribués.

```text
Application
     │
     ↓
   CORBA
     │
     ↓
  YAINDIS
     │
     ├───────────────┐
     ↓               ↓
 Yanla A          Yanla B
     │               │
     └───────┬───────┘
             ↓
        MultiNeural
             │
       ┌─────┴─────┐
       ↓           ↓
      CPU         GPU
```

L'objectif est de pouvoir distribuer progressivement les Neural, les tâches et les ressources matérielles.

---

# Philosophie du projet

Yanla ne cherche pas à remplacer les langages généralistes.

Au contraire, Yanla fournit une couche spécialisée permettant à des applications écrites dans d'autres langages d'utiliser des systèmes intelligents complexes.

Une application Java pourrait simplement effectuer :

```java
ragTask.ask(query);
```

sans avoir à connaître :

* la structure interne du modèle ;
* son langage d'implémentation ;
* son emplacement ;
* le matériel utilisé ;
* le nombre de Neural impliqués ;
* la manière dont YAINDIS a routé la requête.

Le système intelligent devient ainsi un service consommable par différents environnements.

---

# Organisation du dépôt

```text
Yanla/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   ├── concepts.md
│   └── roadmap.md
│
├── examples/
│   ├── neural/
│   ├── rag/
│   ├── interfaces/
│   ├── yaindis/
│   └── exposure/
│
├── compiler/
│
├── runtime/
│
├── yaindis/
│
└── tests/
```

Les exemples présents dans le dépôt servent également de références expérimentales pour l'évolution du langage.

---

# État du projet

Yanla est actuellement en phase de conception et d'implémentation.

Les exemples présents dans ce dépôt représentent la direction architecturale et syntaxique du projet. La syntaxe et les interfaces pourront évoluer au fur et à mesure de l'implémentation du compilateur et du runtime.

L'objectif à terme est de construire progressivement :

```text
Langage
   ↓
Compilateur
   ↓
Runtime
   ↓
MultiNeural
   ↓
YAINDIS
   ↓
Distribution
   ↓
Gestion matérielle
```

---

# Licence

MIT 
