# 🚀 Introduction à Next.js et Intégration de l'Authentification

Dans cette ressource, nous allons découvrir les **bases de Next.js** et voir comment ajouter rapidement un **système d'inscription et de connexion** à notre frontend pour interagir avec l’API Symfony via JWT.

<hr/>

## ⚡ 1. Next.js en Résumé

Next.js est un **framework React** qui facilite la création d'applications web modernes. Il apporte notamment :

✅ **Pages Server-Side Rendering (SSR)** et **Static Site Generation (SSG)** pour des performances optimales.
✅ **API Routes** permettant d'ajouter des endpoints backend dans l'application.
✅ **Simplification du routage** : chaque fichier dans `pages/` devient automatiquement une route.

Depuis la version 13, Next.js introduit le App Router, basé sur le système de fichiers avec React Server Components et Server Actions.

### 📂 Structure d’un projet Next.js avec App Router

```arduino
📂 src
 ┣ 📂 app
 ┃ ┣ 📂 login        // Page de connexion
 ┃ ┣ 📂 register     // Page d'inscription
 ┃ ┣ 📂 layout.js    // Layout global (ex: navbar)
 ┃ ┣ 📂 page.js      // Page d'accueil
 ┣ 📂 components
 ┃ ┣ 📄 Navbar.js    // Barre de navigation
 ┣ 📂 lib
 ┃ ┣ 📄 api.js       // Fonctions API pour l'auth
 ┣ 📂 styles
 ┃ ┣ 📄 globals.css  // Styles globaux
 ┣ 📄 next.config.js
 ┣ 📄 package.json
```

## 🏗 2. Ajouter les liens pour la Connexion et l'Inscription

Ajoutons dans la barre de navigation, les liens Connexion / Inscription (il faudra par la suite remplacer ces liens par le nom de l'utilisateur et un bouton logout).

📄 `src/components/shared/Navbar.tsx`

```tsx
const navItems = [
  { name: "Accueil", href: "/" },
  { name: "Personnages", href: "/personnages" },
  { name: "Créer un personnage", href: "/creation" },
  { name: "Arène", href: "/arenes" },
  { name: "Boutique", href: "/boutique" },
  { name: "Connexion", href: "/connexion" },
  { name: "Inscription", href: "/inscription" },
];
```

Ces liens sont utilisé un peu plus bas dans le composant :

```tsx
<div className="hidden md:block">
  <div className="ml-10 flex items-baseline space-x-4">
    {navItems.map((item) => (
      <Link
        key={item.name}
        href={item.href}
        className="text-gray-300 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium"
      >
        {item.name}
      </Link>
    ))}
  </div>
</div>
```

## 🎨 Qu'est-ce qu'un composant en Next.js ?

### 🧩 1. Définition
Un **composant** est une **brique réutilisable** de l’interface utilisateur. Il permet de diviser une application en morceaux indépendants et réutilisables.

Dans **Next.js (et React)**, un composant est simplement une **fonction JavaScript** qui retourne du **JSX** (un mélange de JavaScript et HTML).
On peut utiliser aussi du TSX et c'est notre cas ! (du Typescript et du HTML)

💡 **Un composant permet de structurer et d’organiser le code pour rendre l’application plus lisible et plus facile à maintenir.**

##  3. Créer les formulaires de connexion et d'inscription

⚠️ N'hésitez pas à prendre un peu le temps de comprendre comment fonctionne le code qui est donné car il y a des petites parcelles qui doivent être modifié

📄 `src/app/(root)/connexion/page.tsx`


```jsx
"use client";

import { motion } from "framer-motion";
import Navbar from "@/components/shared/Navbar";
import { useState } from "react";

export default function Connexion() {
    const [email, setEmail] = useState("");
    const [password, setPassword] = useState("");

    const handleLoginSubmit = async (e: React.FormEvent) => {
        e.preventDefault();
        const loginData = {
            username: email,
            password: password,
        };

        try {
            const response = await fetch("https://127.0.0.1:8000/api/login_check", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify(loginData),
            });

            if (!response.ok) {
                throw new Error("Network response was not ok");
            }

            const data = await response.json();
            console.log("Login successful:", data);

            // Save the token in the local storage
            localStorage.setItem("jwtToken", data.token);
            window.location.href = "/";
        } catch (error) {
            console.error("Login failed:", error);
        }
    };

    return (
        <div className="min-h-screen bg-gray-900 text-white">
            {/* Navbar */}
            <Navbar />

            {/* Main Content */}
            <main className="container mx-auto px-4 py-8">
                <motion.h1
                    initial={{ y: -50, opacity: 0 }}
                    animate={{ y: 0, opacity: 1 }}
                    transition={{ duration: 0.5 }}
                    className="text-4xl font-bold mb-8 text-center"
                >
                    Connexion
                </motion.h1>

                {/* Formulaire de connexion */}
                <form onSubmit={handleLoginSubmit} className="max-w-md mx-auto bg-gray-800 p-8 rounded-lg shadow-lg">
                    <div className="mb-4">
                        <label htmlFor="email" className="block text-sm font-medium text-gray-300">
                            Email
                        </label>
                        <input
                            type="email"
                            id="email"
                            value={email}
                            onChange={(e) => setEmail(e.target.value)}
                            className="mt-1 block w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-md text-white focus:outline-none focus:ring focus:ring-indigo-500"
                            required
                        />
                    </div>
                    <div className="mb-6">
                        <label htmlFor="password" className="block text-sm font-medium text-gray-300">
                            Mot de passe
                        </label>
                        <input
                            type="password"
                            id="password"
                            value={password}
                            onChange={(e) => setPassword(e.target.value)}
                            className="mt-1 block w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-md text-white focus:outline-none focus:ring focus:ring-indigo-500"
                            required
                        />
                    </div>
                    <button
                        type="submit"
                        className="w-full py-2 px-4 bg-indigo-600 hover:bg-indigo-700 rounded-md text-white font-semibold focus:outline-none focus:ring focus:ring-indigo-500"
                    >
                        Se connecter
                    </button>
                </form>
            </main>
        </div>
    );
}
```

📄 `src/app/(root)/inscription/page.tsx`

```jsx
"use client";

import { motion } from "framer-motion";
import Navbar from "@/components/shared/Navbar";
import { useState } from "react";



export default function Inscription() {
    const [email, setEmail] = useState("");
    const [pseudo, setPseudo] = useState("");
    const [password, setPassword] = useState("");

    const handleRegisterSubmit = async (e: React.FormEvent) => {
        e.preventDefault();
        const registerData = {
            email: email,
            pseudo: pseudo,
            password: password,
        };

        try {
            const response = await fetch("https://127.0.0.1:8000/api/register", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify(registerData),
            });

            if (!response.ok) {
                throw new Error("Network response was not ok");
            }

            const data = await response.json();
            console.log("Registration successful:", data);
            // Redirection possible après l'inscription
            window.location.href = "/connexion";
        } catch (error) {
            console.error("Registration failed:", error);
        }
    };

    return (
        <div className="min-h-screen bg-gray-900 text-white">
            {/* Navbar */}
            <Navbar />

            {/* Main Content */}
            <main className="container mx-auto px-4 py-8">
                <motion.h1
                    initial={{ y: -50, opacity: 0 }}
                    animate={{ y: 0, opacity: 1 }}
                    transition={{ duration: 0.5 }}
                    className="text-4xl font-bold mb-8 text-center"
                >
                    Inscription
                </motion.h1>

                {/* Formulaire d'inscription */}
                <form onSubmit={handleRegisterSubmit} className="max-w-md mx-auto bg-gray-800 p-8 rounded-lg shadow-lg">
                    <div className="mb-4">
                        <label htmlFor="email" className="block text-sm font-medium text-gray-300">
                            Email
                        </label>
                        <input
                            type="email"
                            id="email"
                            value={email}
                            onChange={(e) => setEmail(e.target.value)}
                            className="mt-1 block w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-md text-white focus:outline-none focus:ring focus:ring-indigo-500"
                            required
                        />
                    </div>
                    <div className="mb-4">
                        <label htmlFor="pseudo" className="block text-sm font-medium text-gray-300">
                            Pseudo
                        </label>
                        <input
                            type="text"
                            id="pseudo"
                            value={pseudo}
                            onChange={(e) => setPseudo(e.target.value)}
                            className="mt-1 block w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-md text-white focus:outline-none focus:ring focus:ring-indigo-500"
                            required
                        />
                    </div>
                    <div className="mb-6">
                        <label htmlFor="password" className="block text-sm font-medium text-gray-300">
                            Mot de passe
                        </label>
                        <input
                            type="password"
                            id="password"
                            value={password}
                            onChange={(e) => setPassword(e.target.value)}
                            className="mt-1 block w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-md text-white focus:outline-none focus:ring focus:ring-indigo-500"
                            required
                        />
                    </div>
                    <button
                        type="submit"
                        className="w-full py-2 px-4 bg-indigo-600 hover:bg-indigo-700 rounded-md text-white font-semibold focus:outline-none focus:ring focus:ring-indigo-500"
                    >
                        S'inscrire
                    </button>
                </form>
            </main>
        </div>
    );
}
```

### Si vous avez une erreur CORS dans votre console

Installer : `composer require nelmio/cors-bundle` dans votre backend Symfony

Puis déplacez le bout de code : 
```dotenv
###> nelmio/cors-bundle ###
CORS_ALLOW_ORIGIN='^https?://(localhost|127\.0\.0\.1)(:[0-9]+)?$'
###< nelmio/cors-bundle ###
```

du fichier `.env` au fichier `.env.local`

### Remplacer "connexion" et "inscription" par "deconnexion" si l'utilisateur s'est connecté

Avant de continuer, il est important de comprendre quelque chose. Il s'agit de cette ligne de la page de connexion : 

```tsx 
// Save the token in the local storage
localStorage.setItem("jwtToken", data.token);
```

L'idée est simple, comme vous l'avez vu avec Postman, si vous avez le JWT token et que vous l'envoyez à Symfony il sera capable de savoir qui est connecté. 

Ici la page de connexion me donne le token que je sauvegarde dans le localstorage. Vous pouvez voir ce token une fois connecté dans l'onglet `Application > Local storage > http://localhost:3000` du Dev Tool de votre navigateur

Il faudra donc envoyer ce token pour chaque requête qui nécessite l'authentification de l'utilisateur. Voici un exemple de requête qui utilise le token : 

```tsx
const token = localStorage.getItem("jwtToken");

const response = await fetch("https://127.0.0.1:8000/api/some_protected_route", {
    method: "GET",
    headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${token}`
    }
});

if (!response.ok) {
    throw new Error("Network response was not ok");
}

const data = await response.json();
console.log("Protected data:", data);
```

Puisqu'un utilisateur est connecté il n'est plus intéressant d'afficher "connexion" et "inscription" dans la navigation. Alors vous pouvez mettre le code suivant dans : 

📄 `src/components/shared/Navbar.tsx`

```jsx
"use client";

import { useEffect, useState } from "react";
import Link from "next/link";
import React from "react";
import { Menu, X } from "lucide-react";

const navItems = [
  { name: "Accueil", href: "/" },
  { name: "Personnages", href: "/personnages" },
  { name: "Créer un personnage", href: "/creation" },
  { name: "Arène", href: "/arenes" },
  { name: "Boutique", href: "/boutique" },
  { name: "Connexion", href: "/connexion" },
  { name: "Inscription", href: "/inscription" },
];

const Navbar = () => {
  const [isMenuOpen, setIsMenuOpen] = useState(false);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [username, setUsername] = useState("");

  useEffect(() => {
    const token = localStorage.getItem("jwtToken");
    if (token) {
      setIsAuthenticated(true);
      // Récupérer le nom d'utilisateur à partir du token ou d'une requête API
      // setUsername("NomUtilisateur"); // Remplacez par la logique appropriée
    }
  }, []);

  const handleLogout = () => {
    localStorage.removeItem("jwtToken");
    setIsAuthenticated(false);
    setUsername("");
    window.location.href = "/"; // Rediriger vers la page d'accueil après la déconnexion
  };


  return (
    <nav className="bg-gray-800 shadow-lg">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex items-center justify-between h-16">
          <div className="flex justify-between items-center w-full">
            <Link href="/" className="flex-shrink-0">
              <img
                className="h-16 w-16"
                src="/images/street_logo.png"
                alt="Logo"
              />
            </Link>
            <div className="hidden md:block">
              <div className="ml-10 flex items-baseline space-x-4">
                {navItems
                  .filter((item) =>
                    isAuthenticated
                      ? item.name !== "Connexion" && item.name !== "Inscription"
                      : item.name !== "Créer un personnage"
                  )
                  .map((item) => (
                    <Link
                      key={item.name}
                      href={item.href}
                      className="text-gray-300 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium"
                    >
                      {item.name}
                    </Link>
                  ))}
                {isAuthenticated && (
                  <>
                    <span className="text-gray-300 px-3 py-2 rounded-md text-sm font-medium">
                      {username}
                    </span>
                    <button
                      onClick={handleLogout}
                      className="text-gray-300 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium"
                    >
                      Déconnexion
                    </button>
                  </>
                )}
              </div>
            </div>
          </div>
          <div className="md:hidden">
            <button
              onClick={() => setIsMenuOpen(!isMenuOpen)}
              className="inline-flex items-center justify-center p-2 rounded-md text-gray-400 hover:text-white hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-gray-800 focus:ring-white"
            >
              <span className="sr-only">Open main menu</span>
              {isMenuOpen ? (
                <X className="block h-6 w-6" />
              ) : (
                <Menu className="block h-6 w-6" />
              )}
            </button>
          </div>
        </div>
      </div>

      {/* Mobile menu, show/hide based on menu state */}
      {isMenuOpen && (
        <div className="md:hidden">
          <div className="px-2 pt-2 pb-3 space-y-1 sm:px-3">
            {navItems.map((item) => (
              <Link
                key={item.name}
                href={item.href}
                className="text-gray-300 hover:bg-gray-700 hover:text-white block px-3 py-2 rounded-md text-base font-medium"
                onClick={() => setIsMenuOpen(false)}
              >
                {item.name}
              </Link>
            ))}
          </div>
        </div>
      )}
    </nav>
  );
};

export default Navbar;
```


## 4. Integrer le CRUD des personnages


Bien c'est à vous de jouer maintenant, essayez d'implémenter le crud pour les personnages, vous devriez déjà avoir votre API pour ça, ajoutez des pages dans le dossier (root) ou modifiez celles déjà existantes !