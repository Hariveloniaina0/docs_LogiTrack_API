# Documentation API - Système de Gestion Logistique

Bienvenue sur la documentation complète de notre API de gestion logistique.

## 🎯 Vue d'ensemble

Cette API permet de gérer :
- **Multi-magasins** : Gestion de plusieurs points de vente
- **Inventaire** : Produits, stocks, commandes
- **Communication** : Talkie-walkie en temps réel
- **Import/Export** : Gestion des données via CSV/Excel
- **Démarques** : Suivi des pertes et dommages

## 🔐 Authentification

Toutes les requêtes (sauf login) nécessitent un JWT token :
```bash
Authorization: Bearer {votre_access_token}
```

## 🏪 Gestion multi-magasins

L'API utilise un système de **storeId** pour isoler les données par magasin. Le storeId est extrait du JWT token.

**Important** : Chaque utilisateur doit sélectionner un magasin lors du login.

## 🚀 Démarrage rapide

1. **Se connecter**
```bash
POST /auth/login
{
  "email": "user@example.com",
  "password": "password",
  "storeId": 1
}
```

2. **Récupérer son profil**
```bash
GET /auth/profile
Authorization: Bearer {token}
```

3. **Lister les produits**
```bash
GET /products
Authorization: Bearer {token}
```

## 📚 Modules disponibles

| Module | Description | Base URL |
|--------|-------------|----------|
| Auth | Authentification et gestion des sessions | `/auth` |
| Products | Catalogue produits | `/products` |
| Orders | Gestion des commandes | `/orders` |
| Inventory | Inventaires | `/inventory` |
| Import/Export | Import et export de données | `/import`, `/export` |
| Talkie-Walkie | Communication temps réel | `/talkie-walkie` |
| Stores | Gestion des magasins | `/stores` |
| Users | Gestion des utilisateurs | `/users` |

## 🔑 Rôles utilisateurs

- **ADMIN** : Accès complet à tous les magasins
- **MANAGER** : Gestion d'un ou plusieurs magasins
- **EMPLOYEE** : Accès limité à son magasin

## 📊 Codes de réponse HTTP

| Code | Signification |
|------|---------------|
| 200 | Succès |
| 201 | Ressource créée |
| 400 | Requête invalide |
| 401 | Non authentifié |
| 403 | Accès interdit |
| 404 | Ressource non trouvée |
| 500 | Erreur serveur |

## 🛠️ Format des erreurs
```json
{
  "message": "Description de l'erreur",
  "code": "ERROR_CODE",
  "timestamp": "2024-12-11T10:00:00Z"
}
```

## 📝 Prochaines étapes

- [Guide d'authentification](guides/authentication-flow.md)
- [Comprendre le système multi-magasins](guides/multi-store.md)
- [API Authentification](api/authentication.md)
- [API Produits](api/products.md)