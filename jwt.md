# 🔐 Sécurisation de l'API avec JWT et LexikJWTAuthenticationBundle

Dans cette section, nous allons mettre en place une authentification sécurisée pour votre API Symfony en utilisant **JWT** via le bundle **LexikJWTAuthenticationBundle.**

## Qu'est-ce que JWT ? 🤔

### 🔍 Définition
**JWT (JSON Web Token)** est un standard permettant d’échanger des informations de manière sécurisée entre un client (ex. : frontend Next.js) et un serveur (ex. : Symfony).

Un **JWT** est un **token** encodé en **Base64**, composé de trois parties :

1. **Header** : Contient l’algorithme de signature et le type de token.
2. **Payload** : Contient les informations de l’utilisateur (id, email, rôle...).
3. **Signature** : Permet de vérifier l’intégrité du token.

➡️ Ressources : [https://jwt.io/](https://jwt.io/)

## Installation de LexikJWTAuthenticationBundle 🔧

Symfony ne gère pas nativement JWT, nous allons donc installer le **LexikJWTAuthenticationBundle**, qui est une extension pour sécuriser une API avec JWT.

### 🛠️ Étapes d'installation

1. Installer le bundle via Composer

    ```bash
    composer require lexik/jwt-authentication-bundle
    ```

2. Générer la clé secrète pour signer les tokens :

    ```bash
    symfony console lexik:jwt:generate-keypair
    ```
    
    Si la commande ne fonctionne pas, créez le dossier `config/jwt` à la main et ouvrez un terminal **Git Bash** :

    ```bash
    openssl genrsa -out config/jwt/private.pem -aes256 4096
    ```

    puis : 

    ```bash
    openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem
    ```
➡️ Ressources : [Comment fonctionne la cryptographie à clé publique ?](https://www.cloudflare.com/fr-fr/learning/ssl/how-does-public-key-encryption-work/)

3. Configurer les variables d’environnement dans `.env.local`

    Pour cette étape il faut récupérer le code suivant dans `.env` et le mettre dans `.env.local`

    ```ini
    ###> lexik/jwt-authentication-bundle ###
    JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
    JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
    JWT_PASSPHRASE=your_secret_passphrase
    ###< lexik/jwt-authentication-bundle ###
    ```

    N'oubliez pas de remplacer votre passphrase !

4. Configurer le bundle dans `config/packages/lexik_jwt_authentication.yaml` :

    ```yaml
    lexik_jwt_authentication:
        secret_key: '%env(resolve:JWT_SECRET_KEY)%'
        public_key: '%env(resolve:JWT_PUBLIC_KEY)%'
        pass_phrase: '%env(JWT_PASSPHRASE)%'
        token_ttl: 3600 # Durée de validité du token en secondes
    ```

5. Configurer le pare-feu (firewall) de Symfony dans `config/packages/security.yaml` :

    Ajoutez les éléments que vous n'avez pas déjà en suivant l'exemple ci-dessous. Assurez-vous que le firewall `login` est placé avant `api`, et si `main` existe placez le après `api` ! Sinon vous aurez une erreur `/api/login_check` route not found.

    ```yaml
    # config/packages/security.yaml
    security:
        firewalls:
            login:
                pattern: ^/api/login
                stateless: true
                json_login:
                    check_path: /api/login_check
                    success_handler: lexik_jwt_authentication.handler.authentication_success
                    failure_handler: lexik_jwt_authentication.handler.authentication_failure

            api:
                pattern:   ^/api
                stateless: true
                jwt: ~
    ```

6. Configurer le routing dans `config/routes.yaml` en ajoutant :

    ```yaml
    # config/routes.yaml
    api_login_check:
        path: /api/login_check
    ```

## Créer un endpoint pour créer un compte

```php
<?php
// dans le dossier src/Controller/API/

namespace App\Controller\API;

use App\Entity\User;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

class RegistrationController extends AbstractController
{
    #[Route('/api/register', name: 'api_register', methods: ['POST'])]
    public function register(Request $request, UserPasswordHasherInterface $userPasswordHasher, EntityManagerInterface $entityManager): JsonResponse
    {
        $data = json_decode($request->getContent(), true);

        if ($this->getUser()) {
            return new JsonResponse(['message' => 'User already logged in'], JsonResponse::HTTP_BAD_REQUEST);
        }

        $user = new User();
        $user->setEmail($data['email']);
        $user->setPseudo($data['pseudo']);
        $user->setPassword(
            $userPasswordHasher->hashPassword(
                $user,
                $data['password']
            )
        );

        $entityManager->persist($user);
        $entityManager->flush();

        // Optionally, send a confirmation email here

        return new JsonResponse(['message' => 'User registered successfully'], JsonResponse::HTTP_CREATED);
    }
}
```

## Utilisation de JWT avec Postman 🚀

### Créer un compte Utilisateur

1. **Ouvrir Postman** et créer une requête `POST` vers :

    ```bash
    http://127.0.0.1:8000/api/register
    ```

2. **Dans l’onglet Body**, choisir "raw" et "JSON" puis entrer :

    ```json
    {
        "email": "admin@example.com",
        "username": "adminadmin",
        "password": "password"
    }
    ```

3. **Envoyer la requête**.

### 🔑 Obtenir un Token

1. **Ouvrir Postman** et créer une requête POST vers :

    ```bash
    http://127.0.0.1:8000/api/login_check
    ```

💡 Cette route à été crée par le bundle vous n'vez pas besoin de vous en occuper !

2. **Dans l’onglet Body**, choisir "raw" et "JSON" puis entrer par exemple :

    ```json
    {
        "email": "admin@example.com",
        "password": "password"
    }
    ```

3. **Envoyer la requête**, et récupérer le token JWT dans la réponse.

## Sécuriser un Endpoint avec JWT 🛡️

1. Créez-vous un endpoint qui va vous permettre d'obtenir les informations d'un utilisateur : 

    ```php
    <?php

    namespace App\Controller\API;

    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\HttpFoundation\JsonResponse;
    use Symfony\Component\Routing\Attribute\Route;
    use Symfony\Component\Security\Http\Attribute\IsGranted;

    class UserController extends AbstractController
    {

        #[IsGranted('ROLE_USER')] // C'est cette annotation qui est importante
        #[Route('/api/me', name: 'api_me', methods: ['GET'])]
        public function me(): JsonResponse
        {
            $user = $this->getUser();

            return $this->json([
                'id' => $user->getId(),
                'email' => $user->getEmail(),
                'pseudo' => $user->getPseudo(),
            ]);
        }
    }
    ```

2. Dans **Postman**, créer une requête `GET` vers :

    ```bash
    http://127.0.0.1:8000/api/me
    ```

3. **Dans l’onglet "Authorization"**, choisir "Bearer Token" et coller le token JWT obtenu.

4. **Envoyer la requête**, et si tout est correct, le message de succès s'affichera.

## 🎮 À Vous de Jouer : Implémenter le CRUD des Personnages avec Sécurisation 🔐
Maintenant que vous avez installé **JWT** et testé un endpoint sécurisé, il est temps de passer à la pratique !

**Votre mission : implémenter un CRUD (Create, Read, Update, Delete) pour l’entité** `Character` en protégeant si c'est necessaire (posez-vous les bonnes questions !) chaque route avec les autorisations adéquates.

## 🏆 Objectifs
1. Créer les routes API pour gérer les personnages (`Character`).
2. Restreindre l’accès aux endpoints en fonction du rôle de l’utilisateur.
3. Tester chaque route avec Postman en utilisant un token JWT.

## 🎯 Critères de Réussite ✅

* ✔ Vous devez **protéger les routes** pour que seuls les utilisateurs connectés puissent interagir avec l’API.
* ✔ Un utilisateur ne doit **modifier ou supprimer que ses propres personnages**.
* ✔ Chaque route doit **renvoyer une réponse JSON claire** en cas de succès ou d’erreur.

## ➡️ Dans le prochain chapitre : [Intégration NextJS](./nextjs.md)
