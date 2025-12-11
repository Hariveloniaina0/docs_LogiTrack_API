# Users, Stores & Modules Divers

## Users API

**Base URL:** `/users`

**Authentification:** Requise (JWT)

**Permissions:** ADMIN, MANAGER

---

### GET /users

Récupère tous les utilisateurs du magasin sélectionné.

#### Requête

**Headers:**
```
Authorization: Bearer {access_token}
```

**Query Parameters:**
- `search` (optional): Recherche par nom ou email

#### Réponses

**Succès (200 OK):**
```json
[
  {
    "idUser": 5,
    "userName": "Jean Dupont",
    "emailAddress": "jean.dupont@example.com",
    "userRole": "MANAGER",
    "isActive": true,
    "stores": [
      {
        "idStore": 1,
        "storeName": "Magasin Centre",
        "address": "123 Rue Example"
      }
    ],
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-12-11T10:00:00Z"
  }
]
```

#### Exemples

```bash
# Tous les utilisateurs
curl -X GET http://localhost:3000/users \
  -H "Authorization: Bearer {token}"

# Recherche
curl -X GET "http://localhost:3000/users?search=dupont" \
  -H "Authorization: Bearer {token}"
```

---

### GET /users/:id

Récupère un utilisateur spécifique.

#### Réponses

**Succès (200 OK):**
```json
{
  "idUser": 5,
  "userName": "Jean Dupont",
  "emailAddress": "jean.dupont@example.com",
  "userRole": "MANAGER",
  "isActive": true,
  "stores": [
    {
      "idStore": 1,
      "storeName": "Magasin Centre"
    }
  ]
}
```

**Erreurs:**
- 403: Accès interdit (pas de magasin commun)
- 404: Utilisateur non trouvé

---

### POST /users

Crée un nouvel utilisateur.

#### Requête

**Body:**
```json
{
  "userName": "Marie Martin",
  "emailAddress": "marie.martin@example.com",
  "password": "SecurePass123!",
  "userRole": "EMPLOYEE",
  "storeIds": [1, 2]
}
```

**Validations:**
- Email unique
- Mot de passe minimum 6 caractères
- Rôles: ADMIN, MANAGER, EMPLOYEE

**Restrictions:**

| Rôle créateur | Peut créer | Peut assigner |
|---------------|------------|---------------|
| ADMIN | Tous les rôles | Tous les magasins |
| MANAGER | EMPLOYEE uniquement | Ses magasins uniquement |

#### Réponses

**Succès (201 Created):**
```json
{
  "idUser": 10,
  "userName": "Marie Martin",
  "emailAddress": "marie.martin@example.com",
  "userRole": "EMPLOYEE",
  "isActive": true,
  "stores": [
    {
      "idStore": 1,
      "storeName": "Magasin Centre"
    }
  ]
}
```

**Erreurs:**

| Code | Description |
|------|-------------|
| 400 | Email déjà utilisé |
| 400 | Données invalides |
| 403 | Permissions insuffisantes |

#### Exemple

```bash
curl -X POST http://localhost:3000/users \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "userName": "Marie Martin",
    "emailAddress": "marie.martin@example.com",
    "password": "SecurePass123!",
    "userRole": "EMPLOYEE",
    "storeIds": [1]
  }'
```

---

### PUT /users/:id

Met à jour un utilisateur.

#### Requête

**Body:**
```json
{
  "userName": "Marie Martin-Dubois",
  "userRole": "MANAGER",
  "storeIds": [1, 2],
  "isActive": true
}
```

**Restrictions:**

| Rôle modificateur | Peut modifier | Contraintes |
|-------------------|---------------|-------------|
| ADMIN | Tous | Aucune |
| MANAGER | EMPLOYEE uniquement | Ses magasins uniquement |

#### Réponses

**Succès (200 OK):**
```json
{
  "idUser": 10,
  "userName": "Marie Martin-Dubois",
  "userRole": "MANAGER",
  "stores": [...]
}
```

**Erreurs:**
- 403: Permissions insuffisantes
- 403: Pas de magasin commun
- 404: Utilisateur non trouvé

---

### GET /users/search/name

Recherche d'utilisateurs par nom.

#### Query Parameters:
- `name` (required): Nom à rechercher

#### Réponses

```json
[
  {
    "idUser": 5,
    "userName": "Jean Dupont",
    "emailAddress": "jean.dupont@example.com",
    "userRole": "MANAGER"
  }
]
```

---

## Stores API

**Base URL:** `/stores`

**Authentification:** Requise (JWT)

---

### GET /stores

Récupère tous les magasins.

#### Réponses

**Succès (200 OK):**
```json
[
  {
    "idStore": 1,
    "storeName": "Magasin Centre",
    "address": "123 Rue Example, 75001 Paris",
    "city": "Paris",
    "postalCode": "75001",
    "phone": "+33 1 23 45 67 89",
    "email": "centre@example.com",
    "isActive": true,
    "createdAt": "2024-01-01T10:00:00Z"
  }
]
```

---

### GET /stores/:id

Récupère un magasin spécifique.

#### Réponses

```json
{
  "idStore": 1,
  "storeName": "Magasin Centre",
  "address": "123 Rue Example, 75001 Paris",
  "phone": "+33 1 23 45 67 89",
  "users": [
    {
      "idUser": 5,
      "userName": "Jean Dupont",
      "userRole": "MANAGER"
    }
  ]
}
```

---

### POST /stores

Crée un nouveau magasin (ADMIN uniquement).

#### Requête

**Body:**
```json
{
  "storeName": "Magasin Nord",
  "address": "456 Avenue Test",
  "city": "Lille",
  "postalCode": "59000",
  "phone": "+33 3 20 12 34 56",
  "email": "nord@example.com"
}
```

#### Réponses

**Succès (201 Created):**
```json
{
  "idStore": 3,
  "storeName": "Magasin Nord",
  "address": "456 Avenue Test",
  "isActive": true
}
```

---

### GET /stores/search/name

Recherche de magasins par nom.

#### Query Parameters:
- `name` (required): Nom à rechercher

---

## Suppliers API

**Base URL:** `/suppliers`

**Authentification:** Requise (JWT)

---

### GET /suppliers

Récupère tous les fournisseurs.

#### Réponses

```json
[
  {
    "idSupplier": 1,
    "supplierName": "Fournisseur ABC",
    "contactEmail": "contact@abc.com",
    "contactPhone": "+33 1 11 22 33 44",
    "address": "789 Boulevard Commerce"
  }
]
```

---

### GET /suppliers/:id

Récupère un fournisseur spécifique.

---

### GET /suppliers/search/name

Recherche de fournisseurs par nom.

---

## Inventory API

**Base URL:** `/inventory`

**Authentification:** Requise (JWT)

---

### GET /inventory

Récupère tous les inventaires.

#### Query Parameters:
- `startDate` (optional): Date de début (ISO)
- `endDate` (optional): Date de fin (ISO)

#### Réponses

```json
[
  {
    "idInventory": 1,
    "countedQuantity": 45,
    "systemQuantity": 50,
    "variance": -5,
    "inventoryDate": "2024-12-11T10:00:00Z",
    "product": {
      "productName": "Produit Exemple A",
      "barcodes": [
        {
          "barcodeValue": "1234567890123"
        }
      ]
    },
    "user": {
      "userName": "Jean Dupont"
    }
  }
]
```

---

### POST /inventory

Crée un inventaire.

#### Requête

**Body:**
```json
{
  "productId": 1,
  "countedQuantity": 45,
  "userId": 5
}
```

---

### POST /inventory/multi

Crée plusieurs inventaires.

#### Requête

**Body:**
```json
[
  {
    "productId": 1,
    "countedQuantity": 45,
    "userId": 5
  },
  {
    "productId": 2,
    "countedQuantity": 12,
    "userId": 5
  }
]
```

---

### DELETE /inventory/:id

Supprime un inventaire.

---

## WriteOffs API

**Base URL:** `/writeoffs`

**Authentification:** Requise (JWT)

---

### GET /writeoffs

Récupère toutes les démarques.

#### Réponses

```json
[
  {
    "idWriteOff": 1,
    "writeOffType": "DAMAGE",
    "writeOffQuantity": 2,
    "writeOffDate": "2024-12-11T10:00:00Z",
    "comment": "Emballage endommagé",
    "product": {
      "productName": "Produit Exemple A"
    },
    "user": {
      "userName": "Jean Dupont"
    }
  }
]
```

---

### GET /writeoffs/:id

Récupère une démarque spécifique.

---

### GET /writeoffs/search/comment

Recherche de démarques par commentaire.

---

## Display API

**Base URL:** `/display`

**Authentification:** Requise (JWT + StoreAuthGuard)

---

### POST /display/request

Envoie une demande d'affichage par email.

#### Requête

**Body:**
```json
{
  "recipientEmail": "merchandising@example.com",
  "productId": 1,
  "quantity": 10,
  "format": "A4",
  "desiredDate": "2024-12-15",
  "comment": "Pour promotion de Noël"
}
```

#### Réponses

**Succès (201 Created):**
```json
{
  "message": "Demande d'affichage envoyée avec succès",
  "code": "REQUEST_SENT",
  "requestId": 25,
  "timestamp": "2024-12-11T10:00:00Z"
}
```

**Erreurs:**

| Code | Description |
|------|-------------|
| 400 | Date invalide (passée) |
| 404 | Produit non trouvé |
| 503 | Service email indisponible |

---

### GET /display/history

Récupère l'historique des demandes d'affichage.

#### Query Parameters:
- `storeId` (required): ID du magasin

#### Réponses

```json
{
  "requests": [
    {
      "id": 25,
      "productId": 1,
      "productName": "Produit Exemple A",
      "quantity": 10,
      "format": "A4",
      "desiredDate": "2024-12-15",
      "status": "PENDING",
      "createdAt": "2024-12-11T10:00:00Z"
    }
  ],
  "message": "Historique récupéré avec succès",
  "code": "HISTORY_RETRIEVED"
}
```

---

## FTP API

**Base URL:** `/ftp`

**Authentification:** Requise (JWT)

**Permissions:** ADMIN, MANAGER

---

### GET /ftp

Récupère les configurations FTP du magasin.

#### Query Parameters:
- `storeId` (optional): ID du magasin (utilise le JWT par défaut)

#### Réponses

```json
{
  "data": [
    {
      "idFtp": 1,
      "host": "ftp.example.com",
      "port": 21,
      "username": "user_ftp",
      "remotePath": "/exports",
      "isActive": true,
      "storeId": 1,
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ],
  "message": "Configurations FTP récupérées avec succès",
  "success": true
}
```

---

### POST /ftp

Crée une configuration FTP.

#### Requête

**Body:**
```json
{
  "host": "ftp.example.com",
  "port": 21,
  "username": "user_ftp",
  "password": "secure_password",
  "remotePath": "/exports",
  "isActive": true,
  "storeId": 1
}
```

---

### PUT /ftp/:id

Met à jour une configuration FTP.

---

### DELETE /ftp/:id

Supprime une configuration FTP.

---

### GET /ftp/test/:id

Teste une connexion FTP.

#### Réponses

```json
{
  "success": true,
  "message": "Connexion FTP réussie"
}
```

ou

```json
{
  "success": false,
  "message": "Échec de la connexion: Invalid credentials"
}
```

---

## Auto-Import API

**Base URL:** `/auto-import`

**Authentification:** Requise (JWT)

**Permissions:** ADMIN uniquement

---

### GET /auto-import/config

Récupère la configuration d'import automatique.

#### Réponses

```json
{
  "id": 1,
  "isEnabled": true,
  "frequencyHours": 24,
  "lastRunAt": "2024-12-11T02:00:00Z",
  "nextRunAt": "2024-12-12T02:00:00Z"
}
```

---

### PUT /auto-import/config

Met à jour la configuration.

#### Requête

**Body:**
```json
{
  "isEnabled": true,
  "frequencyHours": 12
}
```

---

### POST /auto-import/run-now

Lance un import manuel immédiat.

#### Réponses

```json
{
  "id": 156,
  "status": "SUCCESS",
  "startedAt": "2024-12-11T10:00:00Z",
  "completedAt": "2024-12-11T10:05:23Z",
  "recordsProcessed": 1250,
  "recordsImported": 1230,
  "recordsFailed": 20,
  "errorMessage": null
}
```

---

### GET /auto-import/history

Récupère l'historique des synchronisations.

#### Réponses

```json
[
  {
    "id": 156,
    "status": "SUCCESS",
    "startedAt": "2024-12-11T02:00:00Z",
    "completedAt": "2024-12-11T02:05:23Z",
    "recordsProcessed": 1250,
    "recordsImported": 1230,
    "recordsFailed": 20
  },
  {
    "id": 155,
    "status": "FAILED",
    "startedAt": "2024-12-10T02:00:00Z",
    "completedAt": "2024-12-10T02:00:45Z",
    "recordsProcessed": 0,
    "errorMessage": "FTP connection failed"
  }
]
```

---

### GET /auto-import/latest

Récupère la dernière synchronisation.

---

### GET /auto-import/stats

Récupère les statistiques d'historique.

#### Réponses

```json
{
  "totalRecords": 150,
  "oldestRecord": "2024-06-01T02:00:00Z",
  "newestRecord": "2024-12-11T02:00:00Z",
  "retentionDays": 180,
  "estimatedCleanupDate": "2024-12-31T00:00:00Z"
}
```

---

### POST /auto-import/cleanup

Nettoie l'historique ancien.

#### Requête

**Body (optional):**
```json
{
  "daysToKeep": 90
}
```

#### Réponses

```json
{
  "deleted": 45,
  "remaining": 105
}
```

---

### DELETE /auto-import/history

Supprime tout l'historique.

#### Réponses

```json
{
  "deleted": 150
}
```

---

## Exemples d'utilisation avancés

### Gestion complète d'un utilisateur

```javascript
class UserManager {
  constructor(token) {
    this.token = token;
    this.baseUrl = '/users';
  }

  async createEmployee(userData, storeIds) {
    const response = await fetch(this.baseUrl, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        ...userData,
        userRole: 'EMPLOYEE',
        storeIds
      })
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message);
    }

    return await response.json();
  }

  async updateUserStores(userId, storeIds) {
    const response = await fetch(`${this.baseUrl}/${userId}`, {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${this.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ storeIds })
    });

    return await response.json();
  }

  async searchUsers(term) {
    const response = await fetch(`${this.baseUrl}?search=${term}`, {
      headers: {
        'Authorization': `Bearer ${this.token}`
      }
    });

    return await response.json();
  }
}
```

### Monitoring des imports automatiques

```javascript
class AutoImportMonitor {
  constructor(token) {
    this.token = token;
  }

  async getStatus() {
    const config = await this.getConfig();
    const latest = await this.getLatest();
    const stats = await this.getStats();

    return {
      isEnabled: config.isEnabled,
      frequency: config.frequencyHours,
      lastRun: latest,
      stats
    };
  }

  async getConfig() {
    const response = await fetch('/auto-import/config', {
      headers: { 'Authorization': `Bearer ${this.token}` }
    });
    return await response.json();
  }

  async getLatest() {
    const response = await fetch('/auto-import/latest', {
      headers: { 'Authorization': `Bearer ${this.token}` }
    });
    return await response.json();
  }

  async getStats() {
    const response = await fetch('/auto-import/stats', {
      headers: { 'Authorization': `Bearer ${this.token}` }
    });
    return await response.json();
  }

  async runNow() {
    const response = await fetch('/auto-import/run-now', {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${this.token}` }
    });
    return await response.json();
  }
}
```

---

## Résumé des permissions

| Endpoint | ADMIN | MANAGER | EMPLOYEE |
|----------|-------|---------|----------|
| /auth/* | ✅ | ✅ | ✅ |
| /users/* | ✅ | ✅ (limité) | ❌ |
| /stores/* | ✅ | ✅ (lecture) | ✅ (lecture) |
| /products/* | ✅ | ✅ | ✅ |
| /orders/* | ✅ | ✅ | ✅ |
| /import/* | ✅ | ✅ | ❌ |
| /export/* | ✅ | ✅ | ✅ |
| /ftp/* | ✅ | ✅ | ❌ |
| /auto-import/* | ✅ | ❌ | ❌ |
| /talkie-walkie/* | ✅ | ✅ | ✅ |

---

## Bonnes pratiques générales

### Sécurité

✅ **Toujours valider** le storeId côté serveur
✅ **Vérifier les permissions** avant chaque action
✅ **Logger les actions sensibles** (création/suppression)
✅ **Utiliser HTTPS** en production
✅ **Implémenter rate limiting** sur les endpoints sensibles

### Performance

✅ **Paginer** les listes longues
✅ **Mettre en cache** les données statiques (stores, suppliers)
✅ **Indexer** les champs recherchés (barcodes, emails)
✅ **Optimiser les requêtes** avec relations

### UX

✅ **Afficher des messages clairs** en cas d'erreur
✅ **Implémenter des loading states**
✅ **Valider côté client** avant l'envoi
✅ **Confirmer les actions destructives**