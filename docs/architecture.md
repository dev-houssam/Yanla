# Architecture de Yanla

## 1. Vue d'ensemble

Yanla est organisé en plusieurs couches ayant chacune une responsabilité distincte.

```text
┌───────────────────────────────────────────────┐
│                  Application                  │
│          Java / C++ / Python / etc.           │
└───────────────────────┬───────────────────────┘
                        │
                     CORBA / RPC
                        │
                        ▼
┌───────────────────────────────────────────────┐
│                    YAINDIS                    │
│         Yanla Intelligent Dispatcher          │
│                                               │
│  découverte · routage · disponibilité         │
│  capacités · versions · instances             │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│                     Yanla                     │
│                                               │
│  Neural · Flow · Inference · Task             │
│  MultiNeural · Memory                         │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│                 Runtime Yanla                 │
│                                               │
│  exécution · mémoire · communication          │
│  synchronisation · ressources                 │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│                Gestion matériel               │
│                                               │
│             CPU · GPU · accélérateurs         │
└───────────────────────────────────────────────┘
```

---

## 2. Couche application

Les applications externes n'ont pas besoin de connaître l'implémentation interne de Yanla.

Elles communiquent avec des services exposés par Yanla au moyen d'une interface distante.

Par exemple :

```text
Application Java
       │
       │ RMI / CORBA / RPC
       ▼
    RAGTask
       │
       ▼
      Yanla
```

L'application peut donc utiliser un système intelligent comme un service.

---

## 3. CORBA

CORBA constitue une possibilité d'exposition distante des services Yanla.

Une interface peut par exemple être définie ainsi :

```idl
interface RAGTask {
    string ask(in string query);
};
```

Le contrat décrit uniquement les opérations accessibles.

L'implémentation réelle reste dans le système Yanla.

```text
Client
  │
  │ ask("...")
  ▼
CORBA
  │
  ▼
YAINDIS
  │
  ▼
RAGTask
```

Cette séparation permet à un client d'utiliser un modèle sans connaître sa structure interne.

---

## 4. YAINDIS

YAINDIS signifie **Yanla Intelligent Dispatcher**.

Il constitue une couche intermédiaire entre les interfaces externes et les services intelligents.

Ses responsabilités peuvent notamment inclure :

* découverte des services ;
* routage des requêtes ;
* gestion des instances ;
* vérification de disponibilité ;
* identification des capacités ;
* gestion des versions ;
* sélection d'une instance capable de traiter une requête ;
* communication entre plusieurs systèmes Yanla ;
* connexion avec des modèles externes.

L'objectif est d'éviter qu'un client soit directement dépendant d'une instance particulière.

```text
                 Client
                   │
                   ▼
                 CORBA
                   │
                   ▼
                YAINDIS
              ┌────┼────┐
              ▼    ▼    ▼
           Task A Task B Modèle externe
```

---

## 5. Couche Yanla

Yanla décrit les systèmes intelligents eux-mêmes.

Les principales abstractions sont :

```text
Neural
structure
flow
inference
task
MultiNeural
memory.zone
```

Un `Neural` définit une unité intelligente.

```text
Neural TemperatureModel {

    structure {
        ...
    }

    flow Forward {
        ...
    }

    inference {
        ...
    }

    task TemperatureTask {
        ...
    }
}
```

---

## 6. Structure

La section `structure` décrit les composants constituant un Neural.

Elle peut contenir par exemple :

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

Elle représente la composition du système.

Elle ne décrit pas nécessairement l'ordre complet d'exécution.

---

## 7. Flux de calcul

Les `flow` décrivent la circulation des données.

Exemple :

```text
flow Forward {

    input
    | encoder
    | neural
    -> output;
}
```

Un même Neural peut posséder plusieurs flux.

Par exemple :

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

Cette séparation permet de représenter différentes opérations sur un même système.

---

## 8. Inférence

La section `inference` définit les opérations utilisables pour exploiter le Neural.

Exemple :

```text
inference {

    predict(input) {

        result = Forward(input);

        return result;
    }
}
```

L'inférence constitue donc l'interface fonctionnelle du modèle.

---

## 9. Task

Une `task` représente un processus persistant associé à un Neural.

Contrairement à une simple fonction appelée puis terminée, une task reste active et maintient l'accès à au moins une instance du système.

```text
task TemperatureTask {

    expose {
        predict;
    }
}
```

Une application externe peut alors effectuer une opération telle que :

```text
TemperatureTask.predict(data);
```

La task peut donc servir de point d'exposition d'un Neural.

---

## 10. MultiNeural

`MultiNeural` permet de gérer plusieurs Neural comme un système coordonné.

```text
MultiNeural {

    VisionNeural;
    DecisionNeural;
    MemoryNeural;
}
```

Les Neural peuvent communiquer entre eux.

Une architecture peut ainsi être organisée comme :

```text
VisionNeural
      │
      ▼
DecisionNeural
      │
      ├──────────► MemoryNeural
      │
      ▼
   Controller
```

Cette architecture permet également d'autoriser certaines interactions entre Neural.

Par exemple, un Neural peut observer ou modifier certains paramètres d'un autre Neural lorsque cela est explicitement autorisé par le système.

---

## 11. Gestion de la mémoire

Yanla doit pouvoir contrôler certaines ressources mémoire de manière explicite.

Exemple :

```text
memory.zone compute_buf size 1024 align 32;
memory.zone result_buf  size 256 align 32;
```

Ces zones peuvent être utilisées pour :

* buffers de calcul ;
* données temporaires ;
* échange avec le GPU ;
* structures persistantes ;
* données partagées entre composants ;
* interaction avec des bibliothèques C.

Cette abstraction est particulièrement importante puisque Yanla cible une compilation vers C.

---

## 12. Compilation vers C

Le compilateur Yanla traduit les constructions du langage vers du C.

```text
        Code Yanla
            │
            ▼
     Analyse syntaxique
            │
            ▼
      Analyse sémantique
            │
            ▼
    Représentation interne
            │
            ▼
       Génération C
            │
            ▼
      Compilateur C
            │
            ▼
      Binaire / bibliothèque
```

L'objectif n'est pas de réimplémenter tout l'écosystème matériel.

Yanla peut s'appuyer sur l'écosystème C existant.

Cela permet notamment d'utiliser :

* bibliothèques scientifiques ;
* bibliothèques GPU ;
* bibliothèques de calcul ;
* bibliothèques système ;
* API matérielles ;
* bibliothèques spécialisées en IA.

---

## 13. CPU et GPU

Le runtime peut utiliser différentes ressources matérielles.

```text
                 Yanla
                   │
              Runtime
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
       CPU                   GPU
        │                     │
        ▼                     ▼
   calcul général        calcul parallèle
```

La gestion effective des ressources dépend du runtime et des bibliothèques utilisées par le système.

---

## 14. RAG

Un système RAG peut être construit à partir de Neural existants.

```text
                RAG
                 │
        ┌────────┴────────┐
        ▼                 ▼
   Embedding          Generator
        │                 │
        ▼                 │
   Recherche             │
        │                 │
        └──────┬──────────┘
               ▼
             Réponse
```

Exemple conceptuel :

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

---

## 15. Modèles externes

Yanla n'a pas vocation à imposer que tous les modèles soient écrits en Yanla.

Un modèle externe peut être connecté au système.

```text
             YAINDIS
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
    Yanla A  Yanla B  Modèle externe
                         │
                  C / C++ / Python
```

Le modèle externe peut être adapté afin de respecter une interface compatible avec l'écosystème Yanla.

---

## 16. Distribution

Plusieurs systèmes Yanla peuvent fonctionner sur des machines différentes.

```text
        Machine A
        ┌─────────┐
        │ YAINDIS │
        │ Yanla A │
        └────┬────┘
             │
             │ réseau
             │
        ┌────▼────┐
        │ YAINDIS │
        │ Yanla B │
        └─────────┘
        Machine B
```

Cela permet d'envisager des architectures distribuées dans lesquelles différents Neural ou groupes de Neural sont exécutés sur différentes machines.

---

## 17. Principe général

L'architecture de Yanla repose donc sur une séparation claire des responsabilités :

```text
CORBA
  │
  │ exposition distante
  ▼
YAINDIS
  │
  │ découverte / routage
  ▼
Yanla
  │
  │ description des systèmes intelligents
  ▼
MultiNeural
  │
  │ coordination des Neural
  ▼
Runtime
  │
  │ exécution / mémoire / ressources
  ▼
CPU / GPU
```

Chaque couche peut évoluer indépendamment des autres.

L'objectif est de construire progressivement un système dans lequel la création d'un modèle intelligent, son exécution, sa composition avec d'autres modèles et son exposition à des applications externes peuvent être décrites dans un même écosystème.



