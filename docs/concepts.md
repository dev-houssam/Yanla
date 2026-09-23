# Concepts fondamentaux de Yanla

## 1. Introduction

Yanla repose sur plusieurs concepts qui permettent de décrire, exécuter, composer et exposer des systèmes intelligents.

Les concepts principaux sont :

* `Neural`
* `structure`
* `flow`
* `inference`
* `task`
* `MultiNeural`
* `memory.zone`
* `YAINDIS`

Ces concepts ont des responsabilités différentes et sont conçus pour pouvoir être combinés.

---

# 2. Neural

Un `Neural` représente une unité intelligente.

Il peut encapsuler :

* une architecture ;
* des paramètres ;
* des données ;
* des flux de calcul ;
* des opérations d'inférence ;
* des tâches persistantes ;
* des ressources mémoire.

Exemple :

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

    inference {

        predict(input) {
            ...
        }
    }
}
```

Un `Neural` peut être simple ou constituer un système complexe.

---

# 3. Structure

`structure` décrit les éléments qui composent un Neural.

Elle répond principalement à la question :

> De quoi le système est-il constitué ?

Exemple :

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

Une structure peut également référencer d'autres composants.

```text
structure {

    encoder = EmbeddingModel;
    generator = LanguageModel;
}
```

La structure constitue donc la description statique du système.

---

# 4. Flow

Un `flow` décrit un chemin de transformation des données.

Il répond principalement à la question :

> Comment les données circulent-elles ?

Exemple :

```text
flow Forward {

    input
    | encoder
    | classifier
    -> output;
}
```

Plusieurs flows peuvent coexister dans un même Neural.

```text
flow Forward {
    ...
}

flow Backward {
    ...
}

flow Validation {
    ...
}
```

Les flows peuvent donc représenter différentes phases ou opérations.

---

# 5. Inference

`inference` décrit les opérations permettant d'utiliser un Neural.

Exemple :

```text
inference {

    predict(input) {

        result = Forward(input);

        return result;
    }
}
```

L'inférence constitue une abstraction utilisable par d'autres composants.

Elle peut notamment être appelée depuis une `task`.

---

# 6. Task

Une `task` représente un processus persistant.

Elle permet de maintenir une instance active d'un Neural et de rendre certaines opérations accessibles.

```text
task TemperatureTask {

    expose {
        predict;
    }
}
```

La différence fondamentale est la suivante :

```text
fonction
    ↓
appel
    ↓
résultat
    ↓
fin
```

alors qu'une task représente :

```text
task
  │
  ├── instance du Neural
  │
  ├── état
  │
  ├── ressources
  │
  └── opérations accessibles
          │
          ├── predict(...)
          └── ...
```

Une task peut donc être considérée comme un point de service vivant.

---

# 7. Exposition

Une task peut exposer certaines opérations.

```text
task RAGTask {

    expose {
        ask;
    }
}
```

L'exposition ne signifie pas nécessairement que l'opération devient automatiquement accessible sur le réseau.

Elle définit d'abord ce qui constitue l'interface publique de la task.

Une couche comme YAINDIS ou une interface distante peut ensuite utiliser cette exposition.

---

# 8. MultiNeural

`MultiNeural` représente un ensemble coordonné de Neural.

Exemple :

```text
MultiNeural CognitiveSystem {

    VisionNeural;
    MemoryNeural;
    DecisionNeural;
}
```

Les Neural peuvent avoir des rôles différents.

```text
VisionNeural
    ↓
DecisionNeural
    ↓
ActionNeural
```

Un MultiNeural permet ainsi de construire des systèmes plus complexes à partir de plusieurs unités.

---

# 9. Interaction entre Neural

Les Neural ne sont pas nécessairement isolés.

Un Neural peut utiliser les résultats d'un autre Neural.

```text
VisionNeural
      │
      ▼
DecisionNeural
```

Dans certains systèmes, un Neural peut également être autorisé à modifier certains paramètres d'un autre Neural.

```text
ControllerNeural
       │
       │ modification autorisée
       ▼
TargetNeural
```

Cette possibilité doit être contrôlée explicitement par le runtime.

L'objectif est de permettre des systèmes adaptatifs et coordonnés sans imposer une architecture unique.

---

# 10. Mémoire

Yanla doit pouvoir distinguer plusieurs types de données et de ressources.

Une zone mémoire peut être déclarée explicitement :

```text
memory.zone compute_buf size 1024 align 32;
```

Une autre zone peut être réservée à des résultats :

```text
memory.zone result_buf size 256 align 32;
```

La mémoire peut être utilisée par différents composants du runtime.

```text
Neural
  │
  ├── paramètres
  ├── données
  ├── buffers
  └── état
```

Cette abstraction permet notamment de rester compatible avec les contraintes d'un environnement compilé vers C.

---

# 11. Paramètres et poids

Un Neural peut posséder des paramètres internes.

Dans le cas d'un réseau neuronal, ceux-ci peuvent notamment correspondre aux poids et biais.

Conceptuellement :

```text
Neural Model {

    parameters {
        weights;
        biases;
    }
}
```

Les paramètres peuvent être utilisés par les flows.

Dans un système MultiNeural, certains paramètres peuvent également être accessibles à d'autres Neural lorsque les règles du système l'autorisent.

---

# 12. Données

Les données peuvent être utilisées à plusieurs niveaux :

```text
input
  ↓
flow
  ↓
intermediate data
  ↓
output
```

Certaines données peuvent également être persistantes.

Exemple :

```text
structure {

    knowledge {
        documents;
        vectors;
    }
}
```

Cela permet notamment de représenter des systèmes comme les RAG.

---

# 13. Composition

Les composants Yanla doivent pouvoir être composés.

Par exemple :

```text
EmbeddingModel
       │
       ▼
 Retrieval
       │
       ▼
   Generator
       │
       ▼
     RAG
```

Le RAG n'a donc pas besoin de réimplémenter chaque modèle.

Il peut utiliser des composants déjà définis.

---

# 14. RAG comme exemple de composition

Un système RAG peut être représenté ainsi :

```text
Neural RAG {

    structure {

        embedding = EmbeddingModel;
        generator = LanguageModel;
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
}
```

Le RAG devient lui-même un composant utilisable par d'autres systèmes.

Il peut donc être composé à son tour :

```text
Application
     │
     ▼
RAG
     │
     ├── EmbeddingModel
     ├── Retrieval
     └── LanguageModel
```

---

# 15. Service intelligent

Une combinaison particulièrement importante est :

```text
Neural
   +
Inference
   +
Task
   +
Exposure
```

Elle permet de transformer un modèle en service intelligent.

Exemple :

```text
Neural RAG {

    ...

    inference {

        ask(query) {
            ...
        }
    }

    task RAGTask {

        expose {
            ask;
        }
    }
}
```

Le modèle peut alors être utilisé comme un service :

```text
RAGTask.ask("Qu'est-ce que Yanla ?");
```

---

# 16. YAINDIS

YAINDIS constitue une couche de coordination entre les consommateurs et les services intelligents.

Il peut maintenir une connaissance des services disponibles :

```text
Service
    │
    ├── nom
    ├── version
    ├── capacités
    ├── adresse
    ├── état
    └── instances
```

Lorsqu'une requête arrive :

```text
Client
  │
  ▼
YAINDIS
  │
  ├── recherche du service
  ├── vérification de disponibilité
  ├── sélection d'une instance
  └── routage
          │
          ▼
       Task
```

Cette couche permet de découpler le client de l'implémentation réelle du modèle.

---

# 17. Modèles externes

Yanla peut également interagir avec des systèmes qui ne sont pas écrits en Yanla.

Par exemple :

```text
Yanla
  │
  ├── Neural A
  ├── Neural B
  │
  └── modèle externe
          │
          ├── C
          ├── C++
          ├── Python
          └── autre système
```

Une interface commune permet alors de les intégrer à l'écosystème.

---

# 18. Distribution

Un système Yanla peut être distribué.

Par exemple :

```text
Machine A
┌────────────────────┐
│ YAINDIS             │
│                    │
│ VisionNeural       │
└─────────┬──────────┘
          │
          │ réseau
          ▼
Machine B
┌────────────────────┐
│ YAINDIS             │
│                    │
│ LanguageNeural     │
└────────────────────┘
```

Chaque machine peut exécuter différentes parties du système.

La distribution peut être transparente pour l'application cliente.

---

# 19. Compilation

Yanla est destiné à être compilé vers C.

```text
Code Yanla
    │
    ▼
AST
    │
    ▼
Analyse sémantique
    │
    ▼
IR Yanla
    │
    ▼
Code C
    │
    ▼
Compilateur C
    │
    ▼
Binaire
```

Le langage peut donc fournir des abstractions de haut niveau tout en conservant une cible d'exécution bas niveau.

---

# 20. Principe de séparation

Les concepts Yanla sont volontairement séparés.

| Concept       | Responsabilité                          |
| ------------- | --------------------------------------- |
| `Neural`      | unité intelligente                      |
| `structure`   | composition du système                  |
| `flow`        | circulation des données                 |
| `inference`   | utilisation du système                  |
| `task`        | processus persistant                    |
| `expose`      | interface publique                      |
| `MultiNeural` | coordination de plusieurs Neural        |
| `memory.zone` | gestion explicite de ressources mémoire |
| `YAINDIS`     | découverte et routage des services      |

Cette séparation permet d'éviter de transformer `Neural` en une abstraction qui ferait tout.

---

# 21. Principe général

Yanla cherche à permettre la construction d'une chaîne complète :

```text
Composants
    ↓
Neural
    ↓
Flows
    ↓
Inference
    ↓
Task
    ↓
Exposure
    ↓
YAINDIS
    ↓
CORBA / RPC
    ↓
Application externe
```

En parallèle, l'exécution suit une autre chaîne :

```text
Yanla
  ↓
Runtime
  ↓
Mémoire
  ↓
CPU / GPU
```

Les deux dimensions peuvent ainsi évoluer indépendamment :

```text
        INTERACTION
             │
        CORBA / RPC
             │
          YAINDIS
             │
            Task
             │
          Inference
             │
          ───────
          EXÉCUTION
             │
           Flow
             │
          Neural
             │
          Runtime
             │
        CPU / GPU
```

L'objectif de Yanla est de réunir ces deux dimensions dans un même environnement : **décrire les systèmes intelligents, les exécuter, les composer et les rendre accessibles à d'autres logiciels.**


