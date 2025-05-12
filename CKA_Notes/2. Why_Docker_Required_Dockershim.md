La raison pour laquelle Docker nécessitait Dockershim pour fonctionner avec Kubernetes (avant la version 1.24) est liée à l'architecture de Docker et à la conception de l'interface CRI (*Container Runtime Interface*). Voici une explication détaillée :

---

### 1. Origine de CRI :
- Kubernetes a introduit l'interface CRI pour standardiser la manière dont il communique avec les runtimes de conteneurs.
- CRI définit une API gRPC qui permet aux orchestrateurs comme Kubernetes d'interagir avec les runtimes de conteneurs sans dépendre d'une implémentation spécifique.

---

### 2. Pourquoi Docker n'était pas conforme à CRI ?
Docker a été conçu avant l'introduction de CRI, et son architecture ne correspondait pas directement à celle requise par CRI. Voici quelques raisons spécifiques :

#### a. Docker est une solution complète, pas seulement un runtime :
- Docker n'est pas uniquement un runtime de conteneurs comme containerd ou CRI-O. C'est une plateforme complète avec des fonctionnalités supplémentaires comme :
  - Outils de développement (Docker CLI, Docker Compose).
  - Un daemon monolithique (*dockerd*) qui combine plusieurs responsabilités (gestion des images, des réseaux, etc.).
  - Intégration avec des registres comme Docker Hub.
  
- Cette architecture monolithique rendait Docker incompatible avec la structure modulaire et légère exigée par CRI.

#### b. Docker n'utilise pas CRI directement :
- CRI exige une séparation claire entre l'orchestrateur (Kubernetes) et le runtime (containerd, runc, etc.).
- Docker, quant à lui, utilise son propre daemon (*dockerd*) qui interagit avec containerd, mais pas via CRI. Cela introduit une couche supplémentaire que CRI ne gère pas nativement.

---

### 3. Rôle de Dockershim :
Pour intégrer Docker dans Kubernetes malgré cette incompatibilité, Dockershim a été créé comme une couche d'adaptation. Voici comment Dockershim fonctionne :

- Dockershim traduit les appels CRI de Kubernetes en API Docker.
- Dockershim communique avec le daemon Docker (*dockerd*), qui lui-même utilise containerd et runc pour gérer les conteneurs.

### Schéma avec Dockershim :
```
Kubernetes -> CRI -> Dockershim -> Docker Daemon (dockerd) -> containerd -> runc -> Conteneurs
```

---

### 4. Pourquoi Kubernetes a abandonné Dockershim ?

#### a. Complexité inutile :
- Ajouter Dockershim à la chaîne introduisait une couche supplémentaire, ce qui augmentait la complexité opérationnelle et les ressources nécessaires.

#### b. Performances :
- Docker Daemon consomme plus de mémoire et CPU que containerd seul. Éliminer cette couche intermédiaire améliore les performances.

#### c. Focus sur les runtimes conformes à CRI :
- Kubernetes a décidé de se concentrer sur les runtimes conformes à CRI comme containerd et CRI-O, qui sont conçus pour interagir directement avec Kubernetes sans nécessiter de couche intermédiaire.

#### d. Simplification de la maintenance :
- Maintenir Dockershim représentait un effort supplémentaire pour les développeurs de Kubernetes, ce qui n'était pas justifié à long terme.

---

### 5. Pourquoi Docker ne s'est pas adapté à CRI ?
Adapter Docker à CRI aurait nécessité une réécriture majeure de son architecture, ce qui :
- Serait coûteux en termes de développement.
- Risquerait de perturber ses utilisateurs existants qui utilisent Docker en dehors de Kubernetes.
- Irait à l'encontre de l'objectif initial de Docker, qui est une solution complète pour le développement et le déploiement de conteneurs.

---

### 6. Après Dockershim : Kubernetes avec containerd
Une fois Dockershim supprimé (Kubernetes 1.24 et versions ultérieures), les clusters Kubernetes utilisent directement des runtimes conformes à CRI comme containerd ou CRI-O. Cela simplifie l'architecture globale :

```
Kubernetes -> CRI -> containerd -> runc -> Conteneurs
```

Docker, en tant qu'outil de développement, peut toujours être utilisé pour créer des images et les pousser dans des registres. Mais pour exécuter des conteneurs dans Kubernetes, ce sont maintenant des runtimes conformes à CRI qui prennent le relais.

---

### Résumé : Pourquoi Docker nécessitait Dockershim ?
1. Docker n'était pas conçu pour être un simple runtime, mais une plateforme complète.
2. Son architecture monolithique n'était pas compatible avec CRI.
3. Dockershim a été un compromis pour permettre à Kubernetes de travailler avec Docker jusqu'à ce que des runtimes conformes à CRI comme containerd soient largement adoptés.
