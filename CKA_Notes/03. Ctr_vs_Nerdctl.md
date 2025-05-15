La différence principale entre **`nerdctl`** et **`ctr`** réside dans leur conception, leur objectif et leur public cible. Voici une explication détaillée :

### **Différences principales :**

| **Caractéristique**         | **`ctr`**                               | **`nerdctl`**                          |
|-----------------------------|----------------------------------------|----------------------------------------|
| **Objectif**                | Outil bas niveau pour containerd       | CLI utilisateur pour containerd        |
| **Public cible**            | Développeurs containerd, experts       | Utilisateurs finaux, développeurs      |
| **Convivialité**            | Syntaxe complexe et brute              | Syntaxe simple et proche de Docker     |
| **Fonctionnalités avancées**| Non                                    | Oui (BuildKit, rootless, compose, etc.)|
| **Cas d'usage**             | Test et débogage containerd            | Gestion de conteneurs au quotidien     |
| **Abstraction**             | Expose directement containerd          | Masque la complexité de containerd     |

---

### **Quand utiliser l'un ou l'autre ?**

- **Utilisez `ctr` si :**
  - Vous travaillez directement avec containerd pour des tests ou du débogage.
  - Vous êtes un développeur ou un mainteneur de containerd.
  - Vous avez besoin d'un accès bas niveau aux fonctionnalités de containerd.

- **Utilisez `nerdctl` si :**
  - Vous cherchez une alternative légère à Docker CLI.
  - Vous utilisez un environnement Kubernetes ou containerd et voulez gérer des conteneurs facilement.
  - Vous êtes habitué aux commandes Docker et voulez une expérience similaire.

---

### **Résumé :**
- **`ctr`** : Outil minimaliste et bas niveau pour interagir directement avec containerd, destiné aux experts et développeurs.
- **`nerdctl`** : Une CLI conviviale et riche en fonctionnalités, conçue pour les utilisateurs finaux comme une alternative à Docker CLI, mais basée sur containerd.

![image](https://github.com/user-attachments/assets/923bce4b-46a0-4565-9f5f-574a21ff583e)
