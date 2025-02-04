# 🌟 Projet Annuel : RPG Tour par Tour avec Symfony et Next.js

Bienvenue dans **le gros projet de l’année !** 🎉 Vous allez créer un jeu de RPG au tour par tour en combinant **Symfony** pour le backend et **Next.js** pour le frontend. Vous travaillerez principalement sur le backend Symfony, mais le frontend sera partiellement fourni pour que vous puissiez vous concentrer sur l’API et la logique du jeu.

Ce projet est une évolution du TP Combat que vous avez déjà réalisé en POO. Cette fois, nous utilisons des outils professionnels pour construire une application moderne.

<hr>

## 🎯 Compétences Pédagogiques

* **Manipuler Symfony** pour créer un backend structurant et robuste.
* **Comprendre et créer une API REST** pour interfacer un frontend.
* **Gérer des entités avec Doctrine** et les tester via des interfaces simples avec Twig.
* **Utiliser Postman** pour tester des requêtes HTTP
* **Explorer les bases d’une application frontend moderne** en Next.js.

Vous apprendrez à réaliser un projet complexe, comme en environnement professionnel. 🚀

<hr>

## 1. Installer le Frontend Next.js

Pour démarrer, vous devez installer le frontend de votre application, qui a été préparé et partagé.

### 🔍 Récupérer le frontend

* **Forkez le répertoire GitHub** du frontend [lien](https://github.com/code-gt/front-street-fighter).

* Clonez votre fork sur votre machine avec la commande suivante :

    ```git
    git clone https://github.com/votre-utilisateur/votre-fork.git
    ```

* Positionnez-vous dans le dossier du projet

* Installez les dépendances :

    ```git
    npm install
    ```

* Lancez le projet en mode développement :

    ```git
    npm run dev
    ```

*Ouvrez votre navigateur sur l’adresse indiquée dans le terminal (par défaut : http://localhost:3000).

🚀 **Vous avez maintenant la base du frontend !** 

## 2. Réfléchir aux Entités

Avant de plonger dans le code, prenez un moment pour conceptualiser les entités dont vous aurez besoin. Une bonne conception à ce stade facilitera tout le reste du projet.

### ❓ Questions à se poser

* Quels sont les objets principaux de votre jeu ?

* Quels sont les attributs nécessaires pour chaque entité ?

* Quelles relations existent entre ces entités ? 

### 📊 A vous de jouer, installer un nouveau projet Symfony et créer vos entités !

## 3. Créer un Mini Front avec Twig

Pour tester vos entités et permettre une première interaction avec votre backend, vous allez créer une interface simple avec Twig.

### 🎉 Objectifs

* Permettre à un utilisateur de :

    1. **Se créer un compte.**

    2. **Se connecter.**

    3. **Créer un personnage.**

* Ajouter une interface admin pour manager les entités (avec EasyAdmin).

## 4. Introduction aux API

Pour permettre au frontend d’interagir avec votre backend, vous allez créer une **API REST**.

### 🔍 Qu’est-ce qu’une API REST ?

Une **API** (Application Programming Interface) permet à deux systèmes de communiquer.

**REST** (Representational State Transfer) est un style architectural basé sur des ressources identifiées par des URL.

Avec Symfony, au lieu de retourner une vue HTML via Twig, on peut retourner directement des **données structurées au format JSON**, qui seront consommées par le frontend.

Exemple d’API REST :

* Une requête `GET /api/personnages` retourne une liste des personnages.

* Une requête `POST /api/personnages` permet de créer un nouveau personnage.

### ⚙️ Créer une API Simple avec Symfony

Dans votre contrôleur, créez une méthode pour retourner un JSON :

```php
// src/Controller/ApiController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Annotation\Route;

class ApiController extends AbstractController
{
    #[Route('/api/hello', name: 'api_hello', methods: ['GET'])]
    public function hello(): JsonResponse
    {
        return $this->json([
            'message' => 'Bienvenue dans votre première API Symfony !',
            'success' => true
        ]);
    }
}
```


### 📬 Qu'est-ce que Postman ?

**Postman** est un outil populaire utilisé par les développeurs pour tester et interagir avec des APIs. Il permet d'envoyer des requêtes HTTP (GET, POST, PUT, DELETE, etc.) et de visualiser les réponses, ce qui est très utile pour vérifier que votre backend fonctionne comme prévu.

#### 🚀 Pourquoi utiliser Postman ?
* **Facilité** : Il offre une interface simple pour tester rapidement vos endpoints API.

* **Flexibilité** : Vous pouvez configurer les requêtes avec des en-têtes, des paramètres, et même des corps JSON.

* **Analyse** : Vous voyez clairement la réponse renvoyée par le serveur (code de statut, données, erreurs, etc.).

#### 🛠️ Installer Postman

1. Rendez-vous sur [https://www.postman.com/](https://www.postman.com/) pour télécharger et installer l'application.

2. Une fois installé, connectez-vous ou créez un compte gratuit.


### Tester votre mini API

1. **Ouvrez Postman** et cliquez sur **"New"** > **Request**.

2. Entrez l’URL de votre endpoint (par exemple : http://127.0.0.1:8000/api/hello).

3. Choisissez la méthode HTTP (GET, POST, etc.).

4. Si nécessaire, ajoutez des paramètres ou un corps (ex. : JSON) pour tester une requête POST.

5. Cliquez sur **Send** et observez la réponse !

```json
{
    "message": "Bienvenue dans votre première API Symfony !",
    "success": true
}
```

🎉 Félicitations ! Vous venez de créer et tester votre première API !

## ➡️ Dans le prochain chapitre : [Authentification JWT](./jwt.md)