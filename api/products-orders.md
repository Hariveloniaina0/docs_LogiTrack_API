# Products & Orders API

## Products API

**Base URL:** `/products`

**Authentification:** Requise (JWT + StoreAuthGuard)

**Filtrage:** Automatique par storeId du JWT token

---

### GET /products

Récupère tous les produits du magasin.

#### Requête

**Headers:**
```
Authorization: Bearer {access_token}
```

#### Réponses

**Succès (200 OK):**
```json
[
  {
    "idProduct": 1,
    "productCode": "PROD001",
    "productName": "Produit Exemple A",
    "barcodes": [
      {
        "idBarcode": 1,
        "barcodeValue": "1234567890123",
        "barcodeType": "EAN13"
      }
    ],
    "priceCaisse": 15.99,
    "priceGestcom": 12.50,
    "supplier": {
      "idSupplier": 1,
      "supplierName": "Fournisseur ABC"
    },
    "stocks": [
      {
        "idStock": 1,
        "stockQuantity": 60,
        "store": {
          "idStore": 1,
          "storeName": "Magasin Centre"
        },
        "warehouse": {
          "idWarehouse": 1,
          "warehouseName": "Entrepôt Principal"
        }
      }
    ],
    "createdAt": "2024-01-15T10:00:00Z",
    "updatedAt": "2024-12-11T10:00:00Z"
  }
]
```

#### Exemple

```bash
curl -X GET http://localhost:3000/products \
  -H "Authorization: Bearer {token}"
```

---

### GET /products/paginated

Récupère les produits avec pagination et recherche.

#### Requête

**Headers:**
```
Authorization: Bearer {access_token}
```

**Query Parameters:**

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| page | number | 1 | Numéro de page |
| limit | number | 20 | Éléments par page |
| search | string | - | Terme de recherche |

#### Réponses

**Succès (200 OK):**
```json
{
  "data": [
    {
      "idProduct": 1,
      "productCode": "PROD001",
      "productName": "Produit Exemple A",
      "barcodes": [
        {
          "barcodeValue": "1234567890123"
        }
      ],
      "priceCaisse": 15.99,
      "stocks": [
        {
          "stockQuantity": 60
        }
      ]
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPreviousPage": false
  }
}
```

#### Exemples

```bash
# Page 1, 20 éléments
curl -X GET "http://localhost:3000/products/paginated?page=1&limit=20" \
  -H "Authorization: Bearer {token}"

# Recherche
curl -X GET "http://localhost:3000/products/paginated?search=exemple" \
  -H "Authorization: Bearer {token}"

# Page 2 avec recherche
curl -X GET "http://localhost:3000/products/paginated?page=2&limit=10&search=abc" \
  -H "Authorization: Bearer {token}"
```

```javascript
// Pagination côté client
async function loadProducts(page = 1, search = '') {
  const params = new URLSearchParams({
    page: page.toString(),
    limit: '20',
    ...(search && { search })
  });

  const response = await fetch(`/products/paginated?${params}`, {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  const { data, meta } = await response.json();
  
  return {
    products: data,
    currentPage: meta.page,
    totalPages: meta.totalPages,
    hasMore: meta.hasNextPage
  };
}
```

---

### GET /products/search

Recherche de produits (flexible).

#### Requête

**Query Parameters:**
- `term` (required): Terme de recherche

#### Réponses

```json
[
  {
    "idProduct": 1,
    "productCode": "PROD001",
    "productName": "Produit Exemple A",
    "barcodes": [
      {
        "barcodeValue": "1234567890123"
      }
    ],
    "priceCaisse": 15.99,
    "stocks": [
      {
        "stockQuantity": 60
      }
    ]
  }
]
```

#### Exemple

```bash
curl -X GET "http://localhost:3000/products/search?term=exemple" \
  -H "Authorization: Bearer {token}"
```

---

### GET /products/barcode/:barcode

Recherche un produit par code-barres.

#### Paramètres

- `barcode`: Code-barres à rechercher

#### Réponses

**Succès (200 OK):**
```json
{
  "idProduct": 1,
  "productCode": "PROD001",
  "productName": "Produit Exemple A",
  "barcodes": [
    {
      "idBarcode": 1,
      "barcodeValue": "1234567890123",
      "barcodeType": "EAN13"
    }
  ],
  "priceCaisse": 15.99,
  "priceGestcom": 12.50,
  "stocks": [
    {
      "stockQuantity": 60,
      "store": {
        "storeName": "Magasin Centre"
      }
    }
  ]
}
```

**Non trouvé (200 OK):**
```json
null
```

#### Exemple

```bash
curl -X GET http://localhost:3000/products/barcode/1234567890123 \
  -H "Authorization: Bearer {token}"
```

```javascript
// Scanner de code-barres
async function scanProduct(barcode) {
  const response = await fetch(`/products/barcode/${barcode}`, {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  const product = await response.json();
  
  if (!product) {
    alert('Produit non trouvé');
    return null;
  }

  return product;
}
```

---

### GET /products/:id

Récupère un produit par son ID.

#### Réponses

**Succès (200 OK):**
```json
{
  "idProduct": 1,
  "productCode": "PROD001",
  "productName": "Produit Exemple A",
  "barcodes": [...],
  "priceCaisse": 15.99,
  "priceGestcom": 12.50,
  "stocks": [...],
  "supplier": {...}
}
```

**Erreurs:**
- 404: Produit non trouvé
- 403: Accès interdit (autre magasin)

---

### GET /products/:id/pending-orders

Récupère les commandes en attente pour un produit.

#### Réponses

```json
[
  {
    "idOrder": 15,
    "orderQuantity": 50,
    "orderDate": "2024-12-10T10:00:00Z",
    "orderStatus": "PENDING",
    "supplier": {
      "supplierName": "Fournisseur ABC"
    }
  }
]
```

---

### POST /products/price-control

Vérifie un prix saisi contre les prix enregistrés.

#### Requête

**Body:**
```json
{
  "barcode": "1234567890123",
  "enteredPrice": 16.50
}
```

#### Réponses

**Succès (200 OK):**
```json
{
  "success": true,
  "message": "Contrôle prix effectué",
  "data": {
    "productName": "Produit Exemple A",
    "priceCaisse": 15.99,
    "priceGestcom": 12.50,
    "differenceCaisse": 0.51,
    "differenceGestcom": 4.00
  }
}
```

**Produit non trouvé:**
```json
{
  "success": false,
  "message": "Produit 1234567890123 introuvable dans ce magasin"
}
```

#### Exemple

```javascript
async function checkPrice(barcode, enteredPrice) {
  const response = await fetch('/products/price-control', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ barcode, enteredPrice })
  });

  const result = await response.json();
  
  if (result.success) {
    const { differenceCaisse, differenceGestcom } = result.data;
    
    if (Math.abs(differenceCaisse) > 0.50) {
      alert(`Écart de prix détecté: ${differenceCaisse.toFixed(2)}€`);
    }
  }
}
```

---

## Orders API

**Base URL:** `/orders`

**Authentification:** Requise (JWT + StoreAuthGuard)

---

### GET /orders

Récupère toutes les commandes du magasin.

#### Requête

**Query Parameters:**

| Paramètre | Type | Description |
|-----------|------|-------------|
| startDate | string | Date de début (ISO) |
| endDate | string | Date de fin (ISO) |

#### Réponses

**Succès (200 OK):**
```json
[
  {
    "idOrder": 15,
    "orderQuantity": 50,
    "orderDate": "2024-12-10T10:00:00Z",
    "orderStatus": "PENDING",
    "receivedDate": null,
    "product": {
      "idProduct": 1,
      "productCode": "PROD001",
      "productName": "Produit Exemple A",
      "barcodes": [
        {
          "barcodeValue": "1234567890123"
        }
      ]
    },
    "supplier": {
      "idSupplier": 1,
      "supplierName": "Fournisseur ABC"
    },
    "store": {
      "idStore": 1,
      "storeName": "Magasin Centre"
    }
  }
]
```

#### Exemples

```bash
# Toutes les commandes
curl -X GET http://localhost:3000/orders \
  -H "Authorization: Bearer {token}"

# Commandes d'une période
curl -X GET "http://localhost:3000/orders?startDate=2024-12-01&endDate=2024-12-11" \
  -H "Authorization: Bearer {token}"
```

---

### GET /orders/:id

Récupère une commande spécifique.

#### Réponses

**Succès (200 OK):**
```json
{
  "idOrder": 15,
  "orderQuantity": 50,
  "orderDate": "2024-12-10T10:00:00Z",
  "orderStatus": "PENDING",
  "product": {...},
  "supplier": {...},
  "store": {...}
}
```

---

### POST /orders

Crée une nouvelle commande.

#### Requête

**Body:**
```json
{
  "productId": 1,
  "orderQuantity": 50,
  "supplierId": 1,
  "orderDate": "2024-12-11T10:00:00Z"
}
```

#### Réponses

**Succès (201 Created):**
```json
{
  "idOrder": 16,
  "orderQuantity": 50,
  "orderDate": "2024-12-11T10:00:00Z",
  "orderStatus": "PENDING",
  "product": {...},
  "supplier": {...}
}
```

---

### POST /orders/multi

Crée plusieurs commandes en une fois.

#### Requête

**Body:**
```json
{
  "orders": [
    {
      "productId": 1,
      "orderQuantity": 50,
      "supplierId": 1,
      "orderDate": "2024-12-11T10:00:00Z"
    },
    {
      "productId": 2,
      "orderQuantity": 30,
      "supplierId": 1,
      "orderDate": "2024-12-11T10:00:00Z"
    }
  ],
  "storeId": 1
}
```

#### Réponses

```json
[
  {
    "idOrder": 16,
    "orderQuantity": 50,
    "orderStatus": "PENDING"
  },
  {
    "idOrder": 17,
    "orderQuantity": 30,
    "orderStatus": "PENDING"
  }
]
```

---

### PUT /orders/:id

Met à jour une commande.

#### Requête

**Body:**
```json
{
  "orderQuantity": 60,
  "supplierId": 2
}
```

#### Réponses

```json
{
  "idOrder": 15,
  "orderQuantity": 60,
  "orderStatus": "PENDING",
  "product": {...},
  "supplier": {...}
}
```

---

### PUT /orders/:id/receive

Marque une commande comme reçue.

#### Réponses

```json
{
  "idOrder": 15,
  "orderStatus": "RECEIVED",
  "receivedDate": "2024-12-11T10:00:00Z",
  "product": {...}
}
```

#### Exemple

```javascript
async function receiveOrder(orderId) {
  const response = await fetch(`/orders/${orderId}/receive`, {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  const order = await response.json();
  console.log(`Commande ${orderId} reçue le ${order.receivedDate}`);
}
```

---

### PUT /orders/:id/cancel

Annule une commande.

#### Réponses

```json
{
  "idOrder": 15,
  "orderStatus": "CANCELED",
  "product": {...}
}
```

---

### PUT /orders/:id/resume

Réactive une commande annulée.

#### Réponses

```json
{
  "idOrder": 15,
  "orderStatus": "PENDING",
  "product": {...}
}
```

---

### DELETE /orders/:id

Supprime une commande.

#### Réponses

**Succès (200 OK):**
```
(void)
```

---

### DELETE /orders/received

Supprime toutes les commandes reçues du magasin.

#### Réponses

```json
{
  "deletedCount": 25
}
```

#### Exemple

```bash
curl -X DELETE http://localhost:3000/orders/received \
  -H "Authorization: Bearer {token}"
```

---

### DELETE /orders/canceled

Supprime toutes les commandes annulées du magasin.

#### Réponses

```json
{
  "deletedCount": 10
}
```

---

### POST /orders/dispute

Signale un litige sur une commande.

#### Requête

**Body:**
```json
{
  "orderId": 15,
  "receivedQuantity": 40,
  "comment": "10 unités manquantes"
}
```

#### Réponses

```json
{
  "data": "/exports/disputes/dispute_15_20241211100500.csv",
  "message": "Litige reporté avec succès",
  "success": true
}
```

---

## Statuts des commandes

| Statut | Description |
|--------|-------------|
| PENDING | En attente de réception |
| RECEIVED | Réception complète |
| CANCELED | Annulée |

---

## Flux de gestion des commandes

```javascript
class OrderManager {
  constructor(token) {
    this.token = token;
  }

  // Créer une commande
  async createOrder(productId, quantity, supplierId) {
    const response = await fetch('/orders', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        productId,
        orderQuantity: quantity,
        supplierId,
        orderDate: new Date().toISOString()
      })
    });

    return await response.json();
  }

  // Créer plusieurs commandes
  async createBulkOrders(orders, storeId) {
    const response = await fetch('/orders/multi', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ orders, storeId })
    });

    return await response.json();
  }

  // Récupérer les commandes en attente
  async getPendingOrders() {
    const response = await fetch('/orders', {
      headers: {
        'Authorization': `Bearer ${this.token}`
      }
    });

    const orders = await response.json();
    return orders.filter(o => o.orderStatus === 'PENDING');
  }

  // Recevoir une commande
  async receiveOrder(orderId) {
    const response = await fetch(`/orders/${orderId}/receive`, {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${this.token}`
      }
    });

    return await response.json();
  }

  // Signaler un litige
  async reportDispute(orderId, receivedQuantity, comment) {
    const response = await fetch('/orders/dispute', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        orderId,
        receivedQuantity,
        comment
      })
    });

    return await response.json();
  }

  // Nettoyer les commandes reçues
  async cleanupReceivedOrders() {
    const response = await fetch('/orders/received', {
      method: 'DELETE',
      headers: {
        'Authorization': `Bearer ${this.token}`
      }
    });

    const { deletedCount } = await response.json();
    console.log(`${deletedCount} commandes supprimées`);
  }
}

// Utilisation
const manager = new OrderManager(token);

// Créer une commande
await manager.createOrder(1, 50, 1);

// Recevoir une commande
await manager.receiveOrder(15);

// Signaler un litige
await manager.reportDispute(16, 40, 'Colis endommagé');
```

---

## Bonnes pratiques

### Products

✅ **Utiliser la pagination** pour les listes longues
✅ **Rechercher par code-barres** pour la performance
✅ **Mettre en cache** les produits fréquemment consultés
✅ **Afficher le stock** disponible en temps réel

### Orders

✅ **Valider les quantités** avant de créer
✅ **Grouper les commandes** par fournisseur
✅ **Notifier** les réceptions avec délai
✅ **Archiver** les commandes anciennes
✅ **Signaler les litiges** immédiatement