# Feuille de route de Yanla

## 1. Objectif

La feuille de route de Yanla est organisée progressivement afin de construire d'abord les fondations du langage, puis son runtime, son système de distribution et enfin les mécanismes d'exposition des systèmes intelligents.

L'objectif n'est pas de tout implémenter immédiatement.

Chaque étape doit produire une base utilisable pour l'étape suivante.

---

# 2. Phase 0 — Fondations du projet

### Objectifs

Mettre en place l'environnement de développement et l'organisation du projet.

### À réaliser

* [ ] Initialiser le dépôt Git
* [ ] Définir l'organisation des sources
* [ ] Définir la licence
* [ ] Définir les conventions du projet
* [ ] Mettre en place le système de compilation
* [ ] Ajouter les premiers tests
* [ ] Définir la documentation technique initiale

### Résultat attendu

Un dépôt compilable et structuré pouvant accueillir le compilateur et le runtime.

---

# 3. Phase 1 — Syntaxe Yanla

### Objectifs

Définir la syntaxe minimale du langage.

Les premiers concepts à supporter sont :

```text
Neural
structure
flow
inference
task
```

Exemple minimal :

```text
Neural Example {

    structure {
        ...
    }

    flow Forward {
        ...
    }

    inference {
        ...
    }

    task ExampleTask {
        ...
    }
}
```

### À réaliser

* [ ] Définir la grammaire
* [ ] Définir les tokens
* [ ] Implémenter le lexer
* [ ] Implémenter le parser
* [ ] Construire l'AST
* [ ] Ajouter les diagnostics syntaxiques
* [ ] Ajouter les premiers tests de parsing

### Résultat attendu

Yanla doit pouvoir analyser un programme source valide et produire un AST.

---

# 4. Phase 2 — Analyse sémantique

### Objectifs

Vérifier que les programmes Yanla sont cohérents au-delà de la syntaxe.

### À réaliser

* [ ] Table des symboles
* [ ] Résolution des noms
* [ ] Vérification des types
* [ ] Vérification des références entre Neural
* [ ] Vérification des flows
* [ ] Vérification des appels d'inférence
* [ ] Vérification des tasks
* [ ] Diagnostics sémantiques

### Exemple d'erreur

```text
Neural Example {

    inference {

        predict(input) {
            UnknownFlow(input);
        }
    }
}
```

Le compilateur doit pouvoir signaler que `UnknownFlow` n'existe pas.

---

# 5. Phase 3 — Représentation intermédiaire

### Objectifs

Introduire une représentation intermédiaire entre l'AST et le code C.

```text
Code Yanla
    │
    ▼
Lexer
    │
    ▼
Parser
    │
    ▼
AST
    │
    ▼
Analyse sémantique
    │
    ▼
IR Yanla
```

### À réaliser

* [ ] Définir l'IR
* [ ] Représenter les Neural
* [ ] Représenter les structures
* [ ] Représenter les flows
* [ ] Représenter les inférences
* [ ] Représenter les tasks
* [ ] Représenter les ressources mémoire
* [ ] Ajouter les transformations IR

### Résultat attendu

Le compilateur doit pouvoir manipuler une représentation indépendante du code source.

---

# 6. Phase 4 — Génération C

### Objectifs

Produire du C à partir de l'IR Yanla.

```text
IR Yanla
   │
   ▼
Générateur C
   │
   ▼
Code C
```

### À réaliser

* [ ] Génération des structures C
* [ ] Génération des fonctions
* [ ] Génération des appels
* [ ] Génération des buffers
* [ ] Génération des allocations nécessaires
* [ ] Génération des interfaces runtime
* [ ] Génération des tasks
* [ ] Génération des métadonnées

### Résultat attendu

Un programme Yanla simple doit pouvoir produire du C compilable.

---

# 7. Phase 5 — Runtime Yanla

### Objectifs

Créer le runtime nécessaire à l'exécution des programmes compilés.

Le runtime doit progressivement prendre en charge :

* [ ] instances de Neural ;
* [ ] état des Neural ;
* [ ] exécution des flows ;
* [ ] inférence ;
* [ ] tasks ;
* [ ] mémoire ;
* [ ] synchronisation ;
* [ ] gestion des ressources.

Architecture :

```text
Programme Yanla
       │
       ▼
     Code C
       │
       ▼
Runtime Yanla
       │
       ├── Neural
       ├── Flow
       ├── Task
       ├── Memory
       └── Resources
```

---

# 8. Phase 6 — `memory.zone`

### Objectifs

Introduire la gestion explicite de zones mémoire.

Exemple :

```text
memory.zone compute_buf size 1024 align 32;
```

### À réaliser

* [ ] Parser `memory.zone`
* [ ] Vérifier les déclarations
* [ ] Générer les structures C correspondantes
* [ ] Gérer l'alignement
* [ ] Gérer la durée de vie
* [ ] Définir les règles d'accès
* [ ] Préparer l'utilisation avec des buffers externes

### Objectif à long terme

Permettre au développeur de contrôler certaines ressources sans devoir écrire directement tout le code C correspondant.

---

# 9. Phase 7 — Neural minimal

### Objectifs

Créer le premier Neural réellement exécutable.

Exemple :

```text
Neural LinearModel {

    structure {
        ...
    }

    flow Forward {
        ...
    }

    inference {

        predict(input) {
            ...
        }
    }
}
```

### À réaliser

* [ ] Instance d'un Neural
* [ ] Paramètres
* [ ] Entrées
* [ ] Sorties
* [ ] Exécution d'un flow
* [ ] Appel d'une inférence
* [ ] Tests de résultats

### Résultat attendu

Un Neural Yanla peut être compilé et exécuté.

---

# 10. Phase 8 — Tasks persistantes

### Objectifs

Introduire le modèle d'exécution persistant.

```text
task ModelTask {

    expose {
        predict;
    }
}
```

La task doit pouvoir :

* [ ] créer ou recevoir une instance de Neural ;
* [ ] conserver son état ;
* [ ] rester active ;
* [ ] recevoir des requêtes ;
* [ ] exécuter une inférence ;
* [ ] retourner un résultat ;
* [ ] gérer plusieurs requêtes selon les capacités du runtime.

Architecture :

```text
Task
 │
 └── Neural instance
       │
       ├── state
       ├── parameters
       └── resources
```

---

# 11. Phase 9 — MultiNeural

### Objectifs

Permettre l'exécution coordonnée de plusieurs Neural.

Exemple :

```text
MultiNeural CognitiveSystem {

    VisionNeural;
    MemoryNeural;
    DecisionNeural;
}
```

### À réaliser

* [ ] Création de plusieurs instances
* [ ] Communication entre Neural
* [ ] Passage de données
* [ ] Synchronisation
* [ ] Gestion des dépendances
* [ ] Contrôle des accès
* [ ] Gestion des états

### Résultat attendu

Plusieurs Neural peuvent fonctionner comme un seul système.

---

# 12. Phase 10 — Interaction entre Neural

### Objectifs

Permettre à un Neural d'interagir avec un autre Neural.

Exemple conceptuel :

```text
ControllerNeural
       │
       ▼
TargetNeural
```

Les interactions peuvent concerner :

* les données ;
* l'état ;
* les sorties ;
* certains paramètres ;
* les poids ;
* les ressources.

### À réaliser

* [ ] Définir les droits d'accès
* [ ] Définir les interfaces entre Neural
* [ ] Contrôler les modifications
* [ ] Gérer la synchronisation
* [ ] Éviter les accès incohérents
* [ ] Ajouter des tests d'interaction

---

# 13. Phase 11 — GPU et accélérateurs

### Objectifs

Permettre au runtime d'utiliser des ressources matérielles spécialisées.

```text
Runtime
   │
   ├── CPU
   │
   ├── GPU
   │
   └── accélérateurs
```

### À réaliser

* [ ] Abstraction des ressources
* [ ] Gestion des buffers
* [ ] Transfert CPU/GPU
* [ ] Synchronisation
* [ ] Intégration avec des bibliothèques C
* [ ] Première exécution GPU

Cette phase doit conserver la possibilité d'exécuter un même système sur CPU lorsque cela est nécessaire.

---

# 14. Phase 12 — YAINDIS

### Objectifs

Construire le dispatcher intelligent.

```text
Client
  │
  ▼
YAINDIS
  │
  ├── découverte
  ├── routage
  ├── disponibilité
  ├── capacités
  └── instances
```

### À réaliser

* [ ] Registre des services
* [ ] Enregistrement d'une task
* [ ] Découverte
* [ ] Identification des capacités
* [ ] Routage
* [ ] Gestion des instances
* [ ] État des services
* [ ] Gestion des versions

---

# 15. Phase 13 — CORBA

### Objectifs

Exposer les tasks Yanla à des applications externes.

Exemple :

```idl
interface RAGTask {
    string ask(in string query);
};
```

### À réaliser

* [ ] Définir les interfaces
* [ ] Générer les bindings nécessaires
* [ ] Connecter CORBA à YAINDIS
* [ ] Connecter YAINDIS aux tasks
* [ ] Tester depuis une application externe
* [ ] Tester depuis Java
* [ ] Tester depuis C++

Architecture :

```text
Application
     │
     ▼
   CORBA
     │
     ▼
  YAINDIS
     │
     ▼
   Task
     │
     ▼
   Neural
```

---

# 16. Phase 14 — Modèles externes

### Objectifs

Permettre à des systèmes externes d'intégrer l'écosystème Yanla.

Exemple :

```text
                 YAINDIS
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
    Yanla        Yanla       Externe
      A            B           Model
                                │
                         C / C++ / Python
```

### À réaliser

* [ ] Définir une interface externe
* [ ] Définir le protocole d'intégration
* [ ] Définir les métadonnées
* [ ] Définir les capacités
* [ ] Définir les entrées/sorties
* [ ] Ajouter un premier adaptateur

---

# 17. Phase 15 — RAG

### Objectifs

Construire un premier système complexe à partir des abstractions Yanla.

Architecture :

```text
                RAG
                 │
       ┌─────────┴─────────┐
       ▼                   ▼
 EmbeddingModel       LanguageModel
       │                   │
       ▼                   │
  Retrieval               │
       │                   │
       └────────┬──────────┘
                ▼
             Answer
```

### À réaliser

* [ ] Embedding
* [ ] Stockage des vecteurs
* [ ] Recherche
* [ ] Contexte
* [ ] Génération
* [ ] Interface `ask`
* [ ] Task RAG
* [ ] Exposition distante

---

# 18. Phase 16 — Distribution Yanla

### Objectifs

Permettre à plusieurs systèmes Yanla de collaborer sur un réseau.

```text
Yanla A
   │
   │ réseau
   ▼
Yanla B
   │
   ▼
Yanla C
```

### À réaliser

* [ ] Communication inter-Yanla
* [ ] Découverte distante
* [ ] Identification des services
* [ ] Routage inter-machines
* [ ] Gestion des erreurs réseau
* [ ] Gestion des instances distantes
* [ ] Sécurité des communications

---

# 19. Phase 17 — Optimisation

Une fois les fonctionnalités principales opérationnelles, le système pourra être optimisé.

### Axes possibles

* optimisation de l'IR ;
* réduction des allocations ;
* réutilisation des buffers ;
* parallélisation ;
* exécution GPU ;
* optimisation des flows ;
* compilation spécialisée ;
* gestion avancée des ressources ;
* réduction de la latence ;
* exécution distribuée.

L'optimisation doit intervenir après la stabilisation des abstractions principales.

---

# 20. Phase 18 — Écosystème

À terme, Yanla pourra être accompagné d'outils supplémentaires :

```text
Yanla
├── compilateur
├── runtime
├── YAINDIS
├── outils CLI
├── bibliothèques
├── documentation
├── exemples
└── tests
```

Des outils pourront notamment permettre :

* la création d'un projet ;
* la compilation ;
* l'exécution ;
* le lancement d'une task ;
* l'inspection d'un Neural ;
* l'inspection des ressources ;
* la découverte des services YAINDIS ;
* le diagnostic des systèmes distribués.

---

# 21. Premier jalon concret

Le premier objectif important n'est pas encore CORBA ou la distribution.

Il consiste à obtenir cette chaîne minimale :

```text
        fichier .yla
             │
             ▼
           Parser
             │
             ▼
            AST
             │
             ▼
       Analyse sémantique
             │
             ▼
          Génération C
             │
             ▼
        Compilation C
             │
             ▼
          Exécution
```

Puis :

```text
Neural
  │
  ▼
Inference
  │
  ▼
Task
  │
  ▼
Programme fonctionnel
```

Une fois cette base fonctionnelle, les couches supérieures pourront être ajoutées progressivement.

---

# 22. Vision à long terme

La vision finale peut être résumée ainsi :

```text
                         APPLICATIONS
                              │
                    Java / C++ / Python
                              │
                         CORBA / RPC
                              │
                              ▼
                           YAINDIS
                              │
                   ┌──────────┼──────────┐
                   ▼          ▼          ▼
                Task A     Task B     Externe
                   │          │        Model
                   ▼          ▼
                Neural     Neural
                   │          │
                   └────┬─────┘
                        ▼
                    MultiNeural
                        │
                        ▼
                    Yanla Runtime
                        │
                ┌───────┼───────┐
                ▼       ▼       ▼
               CPU     GPU   Mémoire
```

Yanla évolue ainsi d'un langage de description de Neural vers un environnement permettant de **construire, exécuter, composer, distribuer et exposer des systèmes intelligents**.



