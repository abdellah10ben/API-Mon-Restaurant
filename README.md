# Mon-Restaurant

## 1. Contexte du Projet

Le projet Mon-Restaurant est une application web fournissant une API RESTful pour une plateforme de commande et de livraison de repas en ligne. Elle **centralise et gère le cycle de vie complet des commandes**, incluant la navigation client, la composition de paniers, la finalisation du paiement et la soumission d'avis.

L'API résout un besoin fondamental : fournir une **source de vérité unique et fiable** pour les entités métier complexes et formaliser les échanges entre restaurateurs et clients.

---

## 2. Architecture Technique et Stack

L'application utilise une **architecture modulaire et conteneurisée** (monolithique pour le backend).

| Composant | Technologie | Rôle |
| --- | --- | --- |
| **Backend / API RESTful** | **Java / Spring Boot** | Gère la logique métier, l'API REST et la persistance. |
| **Base de Données** | **MySQL 8.0** | Stockage des données applicatives. |
| **ORM** | **Hibernate / JPA** | Via Spring Data JPA pour la gestion des entités. |
| **Conteneurisation** | **Docker & Docker Compose** | Déploiement isolé, reproductible et orchestré. |

---

## 3. Description de la base de données


### 📊 1. Schéma de la Base de Données

Voici comment les tables sont structurées et connectées dans MySQL (basé sur le fichier `seed_test_data.sql` et les entités JPA).

**Tables et Colonnes principales :**

* **`client`** : `id` (PK), `firstname`, `lastname`, `email`, `address`, `phone_number`
* **`restaurant`** : `id` (PK), `name`, `description`, `address`, `district`, `phone_number`
* **`plat`** : `id` (PK), `name`, `price`, `category`, `is_vegetarian`, `restaurant_id` (FK)
* **`commandes`** : `id` (PK), `client_id` (FK), `address`, `status`, `total_amount`, `is_paid`
* **`commande_plat`** (Table de liaison) : `commande_id` (FK), `plat_id` (FK)
* **`paiement`** : `id` (PK), `commande_id` (FK/Unique), `paiement_method`, `paiement_status`
* **`aviss`** : `id` (PK), `client_id` (FK), `restaurant_id` (FK), `commande_id` (FK/Unique), `rating`, `description`

---

### 🔗 2. Détail des Relations 

Voici comment les classes Java interagissent entre elles.

#### A. One-to-One (1:1) — "Un pour Un"
*Une entité est liée à une seule autre entité.*

1.  **Commande ↔ Paiement**
    * **Logique :** Une commande ne peut avoir qu'un seul paiement, et un paiement correspond à une seule commande.
    * **Code :** Dans `Paiement.java`, tu as `@OneToOne` vers `Commande`.
    * **SQL :** La table `paiement` a une colonne `commande_id` qui est unique.

2.  **Commande ↔ Avis**
    * **Logique :** Un avis concerne une commande précise. On ne laisse pas plusieurs avis pour la même commande.
    * **Code :** Dans `Avis.java`, tu as `@OneToOne` vers `Commande`.
    * **SQL :** La table `aviss` a une colonne `commande_id` qui est unique.

#### B. Many-to-One (N:1) — "Plusieurs pour Un"
*Plusieurs entités sont liées à une seule entité parente.*

1.  **Plat → Restaurant**
    * **Logique :** Un restaurant propose plusieurs plats, mais un plat spécifique appartient à un seul restaurant.
    * **Code :** Dans `Plat.java`, tu as `@ManyToOne` vers `Restaurant`.
    * **SQL :** La table `plat` contient la clé étrangère `restaurant_id`.

2.  **Commande → Client**
    * **Logique :** Un client peut passer plusieurs commandes, mais une commande est passée par un seul client.
    * **Code :** Dans `Commande.java`, tu as `@ManyToOne` vers `Client`.
    * **SQL :** La table `commandes` contient la clé étrangère `client_id`.

3.  **Avis → Client**
    * **Logique :** Un client peut écrire plusieurs avis.
    * **Code :** Dans `Avis.java`, tu as `@ManyToOne` vers `Client`.

4.  **Avis → Restaurant**
    * **Logique :** Un restaurant peut recevoir plusieurs avis.
    * **Code :** Dans `Avis.java`, tu as `@ManyToOne` vers `Restaurant`.

#### C. Many-to-Many (N:N) — "Plusieurs pour Plusieurs"
*Des entités multiples des deux côtés sont liées entre elles. Cela nécessite une table intermédiaire.*

1.  **Commande ↔ Plat**
    * **Logique :** Une commande contient plusieurs plats. Et le même plat (ex: "Pizza Margherita") peut se retrouver dans plusieurs commandes différentes.
    * **Code :** Dans `Commande.java`, tu as `@ManyToMany` avec `@JoinTable`.
    * **SQL :** C'est la table `commande_plat` qui gère cela. Elle contient juste deux colonnes : `commande_id` et `plat_id`.
    * **Exemple :**
        * Commande #1 contient Plat #1 et Plat #2.
        * Commande #2 contient Plat #3 et Plat #4.

---

### 📝 Résumé :

| Entité A | Relation | Entité B | Clé étrangère (FK) située dans... |
| :--- | :---: | :--- | :--- |
| **Paiement** | 1 — 1 | **Commande** | Table `paiement` (`commande_id`) |
| **Avis** | 1 — 1 | **Commande** | Table `aviss` (`commande_id`) |
| **Plat** | N — 1 | **Restaurant** | Table `plat` (`restaurant_id`) |
| **Commande** | N — 1 | **Client** | Table `commandes` (`client_id`) |
| **Avis** | N — 1 | **Client** | Table `aviss` (`client_id`) |
| **Avis** | N — 1 | **Restaurant** | Table `aviss` (`restaurant_id`) |
| **Commande** | N — N | **Plat** | Table de liaison `commande_plat` |
---

## 4. Déploiement Docker

### Sources

- 👉 [**GitHub: abdellah10ben/API-Mon-Restaurant**](https://github.com/abdellah10ben/API-Mon-Restaurant.git)

- 👉 [**Docker Hub: bouyanzermed/mon-restaurant-api**](https://hub.docker.com/r/bouyanzermed/mon-restaurant-api)

### Lancer l'Application Localement
1. **Prérequis :**
* Docker & Docker Compose installés.
* Git installé.

2. **Clonez le dépôt :**

   ```bash
   git clone https://github.com/abdellah10ben/API-Mon-Restaurant.git
   cd API-Mon-Restaurant
   ```

1. **Lancez les conteneurs :**

   ```bash
   docker-compose up --build -d
   ```

1. **Remplir la base de données  :** Pour initialiser la base de données avec des données de test, vous devez exécuter le script `seed_test_data.sql`. Utilisez la commande suivante pour l'injecter directement dans le conteneur MySQL :

   ```bash
   docker exec -i mon_restaurant_mysql mysql -ubouyanzer -pbouyanzer mon_restaurant < seed_test_data.sql
   ```

L'API sera accessible à `http://localhost:8080`.



## 🐳 Docker Hub & Déploiement Rapide

L'image de l'API est construite et hébergée publiquement.
Vous pouvez lancer l'application **sans toucher au code source**.

🔗 **Lien Docker Hub :**
👉 [bouyanzermed/mon-restaurant-api](https://hub.docker.com/r/bouyanzermed/mon-restaurant-api)


### Lancement via Docker Hub (Image seule)

Si vous avez déjà une base MySQL active :
```bash
docker run -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://host.docker.internal:3306/mon_restaurant \
  -e SPRING_DATASOURCE_USERNAME=bouyanzer \
  -e SPRING_DATASOURCE_PASSWORD=bouyanzer \
  bouyanzermed/mon-restaurant-api:latest
```

---

## 4. Persistance des Données

Le projet utilise un volume Docker nommé mysql_data déclaré dans le docker-compose.yml.

Rôle : Stocker les fichiers de la BDD de manière persistante sur la machine hôte.

Avantage : Même si on supprime les conteneurs (docker-compose down), les clients et commandes créés sont conservés au prochain démarrage.

---

## 5. Documentation des Routes API REST

Toutes les routes sont préfixées par `/api/v1`.

## 🔌 API Endpoints

Voici la liste exhaustive de tous les endpoints (points d'entrée) de l'application. Je les ai regroupés par module pour une meilleure lisibilité. Tu peux utiliser ces URLs directement dans **Postman** ou via **cURL**.

### 👤 1. Clients (`ClientController`)
**Base URL :** `/api/v1/clients`

| Méthode | URL | Description |
| :--- | :--- | :--- |
| `GET` | `/api/v1/clients` | Récupérer tous les clients |
| `GET` | `/api/v1/clients/{id}` | Récupérer un client par son ID |
| `GET` | `/api/v1/clients/email/{email}` | Récupérer un client par son email |
| `POST` | `/api/v1/clients` | Créer un nouveau client |
| `PUT` | `/api/v1/clients/{id}` | Mettre à jour un client existant |
| `DELETE` | `/api/v1/clients/{id}` | Supprimer un client |

### mn 2. Restaurants (`RestaurantController`)
**Base URL :** `/api/v1/restaurants`

| Méthode | URL | Description |
| :--- | :--- | :--- |
| `GET` | `/api/v1/restaurants` | Récupérer tous les restaurants |
| `GET` | `/api/v1/restaurants/{id}` | Récupérer un restaurant par son ID |
| `GET` | `/api/v1/restaurants/name/{name}` | Chercher des restaurants par nom |
| `GET` | `/api/v1/restaurants/district/{district}` | Chercher un restaurant par quartier |
| `POST` | `/api/v1/restaurants` | Créer un nouveau restaurant |
| `PUT` | `/api/v1/restaurants/{id}` | Mettre à jour un restaurant |
| `DELETE` | `/api/v1/restaurants/{id}` | Supprimer un restaurant |

### 🍔 3. Plats (`PlatController`)
⚠️ **Attention :** L'URL est au singulier (`plat`).
**Base URL :** `/api/v1/plat`

| Méthode | URL | Description |
| :--- | :--- | :--- |
| `GET` | `/api/v1/plat` | Récupérer tous les plats |
| `GET` | `/api/v1/plat/{id}` | Récupérer un plat par son ID |
| `GET` | `/api/v1/plat/name/{name}` | Chercher des plats par nom |
| `GET` | `/api/v1/plat/restaurant/{id}` | Récupérer les plats d'un restaurant |
| `GET` | `/api/v1/plat/category/{category}` | Filtrer les plats par catégorie |
| `GET` | `/api/v1/plat/type/vegetarian` | Récupérer uniquement les plats végétariens |
| `GET` | `/api/v1/plat/price-range?min=x&max=y` | Filtrer par prix (ex: `?minPrice=10&maxPrice=20`) |
| `POST` | `/api/v1/plat` | Ajouter un nouveau plat |
| `PUT` | `/api/v1/plat/{id}` | Mettre à jour un plat |
| `DELETE` | `/api/v1/plat/{id}` | Supprimer un plat |

### 📦 4. Commandes (`CommandeController`)
**Base URL :** `/api/v1/commandes`

| Méthode | URL | Description |
| :--- | :--- | :--- |
| `GET` | `/api/v1/commandes` | Récupérer toutes les commandes |
| `GET` | `/api/v1/commandes/{id}` | Récupérer une commande par son ID |
| `GET` | `/api/v1/commandes/client/{clientId}` | Voir les commandes d'un client |
| `GET` | `/api/v1/commandes/restaurant/{id}` | Voir les commandes d'un restaurant |
| `POST` | `/api/v1/commandes` | Passer une nouvelle commande |
| `PUT` | `/api/v1/commandes/{id}` | Mettre à jour une commande (ex: statut) |

### 💳 5. Paiements (`PaiementController`)
**Base URL :** `/api/v1/paiements`

| Méthode | URL | Description |
| :--- | :--- | :--- |
| `GET` | `/api/v1/paiements` | Récupérer tous les paiements |
| `GET` | `/api/v1/paiements/{id}` | Récupérer un paiement par son ID |
| `POST` | `/api/v1/paiements` | Effectuer un paiement |
| `PUT` | `/api/v1/paiements/{id}` | Mettre à jour un paiement (ex: statut) |

### ⭐ 6. Avis (`AvisController`)
⚠️ **Attention :** L'URL prend deux 's' (`aviss`).
**Base URL :** `/api/v1/aviss`

| Méthode | URL | Description |
| :--- | :--- | :--- |
| `GET` | `/api/v1/aviss` | Récupérer tous les avis |
| `GET` | `/api/v1/aviss/{id}` | Récupérer un avis par son ID |
| `GET` | `/api/v1/aviss/restaurant/{id}` | Voir les avis d'un restaurant |
| `GET` | `/api/v1/aviss/user/{clientId}` | Voir les avis laissés par un client |
| `GET` | `/api/v1/aviss/date/{date}` | Chercher des avis par date |
| `POST` | `/api/v1/aviss` | Laisser un avis |
| `PUT` | `/api/v1/aviss/{id}` | Modifier un avis |
| `DELETE` | `/api/v1/aviss/{id}` | Supprimer un avis |

### Exemple : Créer une Commande

**Endpoint :** `POST /api/v1/orders`

```json
{
    "client": {"id": 1},
    "plats": [{"id": 3}, {"id": 5}],
    "address": "12 Rue de la Livraison, 75001 Paris",
    "deliveryType": "SHIPPING"
}
```

---

## 6. Validation et Tests Fonctionnels

Un script Shell (`test_api.sh`) automatise un **scénario nominal complet (Happy Path)** :

1. Création de client.

1. Recherche de restaurant et filtrage de plats.

1. Passage d'une commande complexe.

1. Paiement (statut passe à 'PAID').

1. Soumission d'un avis.

**Exécution :**

```bash
chmod +x test_api.sh
./test_api.sh
```

---

## 7. Diagrammes

### Diagramme de classe

![Diagramme de classe](https://github.com/user-attachments/assets/41bb66bc-d61c-4587-960d-764922774c8a)

### Diagramme Merise (Entité-Association)

<img width="509" height="385" alt="Entite_Asso_Diag" src="https://github.com/user-attachments/assets/b2f6e244-82d7-4089-a525-ab7acd3b35df" />


