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

## 3. Description des Entités 

L'API gère des relations complexes telles que le **Many-to-Many** (une Commande lie plusieurs Plats) et le **One-to-Many** (un Client passe plusieurs Commandes).

### 1. Entité : CLIENT

- **Attributs :** `id_client` (CP), `prenom_client`, `nom_client`, `email_client`, `adresse_client`, `telephone_client`.

- **Relations :** 1,N vers COMMANDE, 0,N vers AVIS.

### 2. Entité : RESTAURANT

- **Attributs :** `id_restaurant` (CP), `nom_restaurant`, `description_restaurant`, `adresse_restaurant`, `quartier_restaurant`, `telephone_restaurant`.

- **Relations :** 1,N vers PLAT, 0,N vers AVIS.

### 3. Entité : PLAT 

- **Attributs :** `id_plat` (CP), `id_restaurant` (CE), `nom_plat`, `description_plat`, `categorie_plat`, `prix_plat`, `est_vegetarien`.

- **Relations :** N,1 vers RESTAURANT, N,N avec COMMANDE (via LIGNE_COMMANDE).

### 4. Entité : COMMANDE 

- **Attributs :** `id_commande` (CP), `id_client` (CE), `adresse_livraison`, `type_livraison`.

- **Relations :** N,1 vers CLIENT, N,N avec PLAT, 1,1 vers PAIEMENT, 0,1 vers AVIS.

### 5. Entité : PAIEMENT

- **Attributs :** `id_commande` (CP & CE), `methode_paiement`.

- **Relations :** 1,1 vers COMMANDE.

### 6. Entité : AVIS 

- **Attributs :** `id_avis` (CP), `id_client` (CE), `id_restaurant` (CE), `id_commande` (CE), `titre_avis`, `note_avis`, `description_avis`.

- **Relations :** 1,1 vers COMMANDE.

---

## 4. Déploiement Docker

### Sources

- 👉 [**GitHub: abdellah10ben/API-Mon-Restaurant**](https://github.com/abdellah10ben/API-Mon-Restaurant.git)

- 👉 [**Docker Hub: bouyanzermed/mon-restaurant-api**](https://hub.docker.com/r/bouyanzermed/mon-restaurant-api)

### Lancer l'Application Localement

1. **Clonez le dépôt :**

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

### Persistance des Données 

Le projet utilise des volumes Docker pour assurer la persistance des données même après l'arrêt des conteneurs (`docker-compose down`).

---

## 5. Documentation des Routes API REST

Toutes les routes sont préfixées par `/api/v1`.

| Module | Route de base | Description |
| --- | --- | --- |
| **Clients** | `/clients` | Liste, création, mise à jour et suppression de clients. |
| **Restaurants** | `/restaurants` | Gestion des établissements, filtrage par quartier ou nom. |
| **Plats** | `/plat` | Gestion du menu, filtrage par catégorie, restaurant ou prix. |
| **Commandes** | `/commandes` | Création et suivi des commandes. |
| **Paiements** | `/paiements` | Enregistrement et suivi des transactions. |
| **Avis** | `/aviss` | Gestion des notes et commentaires. |

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

# API-Mon-Restaurant
