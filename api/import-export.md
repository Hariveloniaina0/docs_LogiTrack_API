# Import & Export API

## Import API

**Base URL:** `/import`

**Authentification:** Requise (JWT) + Rôles: ADMIN, MANAGER

### Vue d'ensemble

L'API Import permet d'importer des produits et stocks depuis des fichiers CSV ou Excel avec un système de mapping flexible.

---

## POST /import/headers

Extrait les en-têtes d'un fichier pour permettre le mapping.

### Requête

**Headers:**
```
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Form Data:**
- `file`: Fichier CSV ou Excel (.csv, .xlsx, .xls)

**Limites:**
- Taille maximale: 10 MB
- Formats acceptés: .csv, .xlsx, .xls

### Réponses

**Succès (200 OK):**
```json
{
  "headers": [
    "Code Produit",
    "Code-barres",
    "Nom Produit",
    "Quantité Magasin",
    "Quantité Entrepôt",
    "Prix Caisse",
    "Prix Gestcom"
  ],
  "rowCount": 150,
  "fileName": "produits.csv",
  "fileSize": 45678
}
```

**Erreurs:**

| Code | Description |
|------|-------------|
| 400 | Fichier manquant ou vide |
| 400 | Format de fichier non supporté |
| 500 | Erreur lors de l'extraction |

### Exemple

```bash
curl -X POST http://localhost:3000/import/headers \
  -H "Authorization: Bearer {token}" \
  -F "file=@produits.csv"
```

```javascript
// JavaScript avec FormData
const formData = new FormData();
formData.append('file', fileInput.files[0]);

const response = await fetch('/import/headers', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`
  },
  body: formData
});

const { headers, rowCount } = await response.json();
```

---

## POST /import/products

Importe des produits avec mapping personnalisé.

### Requête

**Headers:**
```
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Form Data:**
- `file`: Fichier à importer
- `mappings`: JSON string avec les mappings

**Format des mappings:**
```json
{
  "Code Produit": "productCode",
  "Code-barres": "barcodeValue",
  "Nom Produit": "productName",
  "Quantité Magasin": "stockQuantityStore",
  "Quantité Entrepôt": "stockQuantityWarehouse",
  "Prix Caisse": "priceCaisse",
  "Prix Gestcom": "priceGestcom",
  "Nom Magasin": "storeName",
  "Nom Entrepôt": "warehouseName",
  "Nom Fournisseur": "supplierName"
}
```

### Réponses

**Succès (200 OK):**
```json
{
  "importedCount": 145,
  "skippedCount": 5,
  "errorCount": 0,
  "errors": [],
  "warnings": [
    {
      "row": 23,
      "message": "Produit PROD023 existe déjà, mis à jour"
    }
  ],
  "summary": {
    "totalRows": 150,
    "productsCreated": 140,
    "productsUpdated": 5,
    "stocksCreated": 145,
    "suppliersCreated": 3
  }
}
```

**Erreur partielle:**
```json
{
  "importedCount": 100,
  "skippedCount": 50,
  "errorCount": 10,
  "errors": [
    {
      "row": 15,
      "field": "productCode",
      "message": "Code produit manquant",
      "data": { "Nom Produit": "Produit sans code" }
    },
    {
      "row": 42,
      "field": "barcodeValue",
      "message": "Code-barres invalide",
      "data": { "Code-barres": "ABC" }
    }
  ],
  "warnings": [],
  "summary": {
    "totalRows": 160,
    "productsCreated": 100,
    "productsUpdated": 0,
    "stocksCreated": 100,
    "suppliersCreated": 2
  }
}
```

**Erreurs:**

| Code | Description |
|------|-------------|
| 400 | Fichier manquant |
| 400 | Mappings invalides ou manquants |
| 400 | Champs obligatoires non mappés |
| 500 | Erreur lors de l'import |

### Exemple complet

```javascript
// 1. Extraire les headers
const headersFormData = new FormData();
headersFormData.append('file', file);

const headersResponse = await fetch('/import/headers', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${token}` },
  body: headersFormData
});

const { headers } = await headersResponse.json();

// 2. Créer les mappings (interface utilisateur)
const mappings = {
  [headers[0]]: 'productCode',      // "Code Produit" -> productCode
  [headers[1]]: 'barcodeValue',     // "Code-barres" -> barcodeValue
  [headers[2]]: 'productName',      // "Nom Produit" -> productName
  [headers[3]]: 'stockQuantityStore', // etc.
};

// 3. Importer avec mappings
const importFormData = new FormData();
importFormData.append('file', file);
importFormData.append('mappings', JSON.stringify(mappings));

const importResponse = await fetch('/import/products', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${token}` },
  body: importFormData
});

const result = await importResponse.json();
console.log(`Importé: ${result.importedCount}, Erreurs: ${result.errorCount}`);
```

---

## GET /import/fields

Récupère la liste des champs disponibles pour le mapping.

### Réponses

**Succès (200 OK):**
```json
{
  "fields": [
    {
      "value": "productCode",
      "label": "Code Produit",
      "required": true,
      "category": "product",
      "description": "Code unique du produit"
    },
    {
      "value": "barcodeValue",
      "label": "Code-barres",
      "required": true,
      "category": "product",
      "description": "Code-barres du produit (EAN13, etc.)"
    },
    {
      "value": "productName",
      "label": "Nom du Produit",
      "required": true,
      "category": "product",
      "description": "Nom complet du produit"
    },
    {
      "value": "stockQuantityStore",
      "label": "Quantité Stock Magasin",
      "required": false,
      "category": "stock",
      "description": "Quantité disponible en magasin"
    },
    {
      "value": "stockQuantityWarehouse",
      "label": "Quantité Stock Entrepôt",
      "required": false,
      "category": "stock",
      "description": "Quantité disponible en entrepôt"
    },
    {
      "value": "priceCaisse",
      "label": "Prix Caisse",
      "required": false,
      "category": "pricing",
      "description": "Prix de vente en caisse"
    },
    {
      "value": "priceGestcom",
      "label": "Prix Gestcom",
      "required": false,
      "category": "pricing",
      "description": "Prix de gestion commerciale"
    }
  ]
}
```

### Exemple

```bash
curl -X GET http://localhost:3000/import/fields \
  -H "Authorization: Bearer {token}"
```

---

## GET /import/formats

Récupère les formats de fichiers supportés.

### Réponses

```json
{
  "formats": [
    {
      "extension": ".csv",
      "mimeTypes": ["text/csv", "application/csv"],
      "description": "Fichier CSV avec délimiteurs: virgule, point-virgule ou tabulation",
      "maxSize": "10 MB"
    },
    {
      "extension": ".xlsx",
      "mimeTypes": ["application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"],
      "description": "Fichier Excel moderne (.xlsx)",
      "maxSize": "10 MB"
    },
    {
      "extension": ".xls",
      "mimeTypes": ["application/vnd.ms-excel"],
      "description": "Fichier Excel classique (.xls)",
      "maxSize": "10 MB"
    }
  ]
}
```

---

## GET /import/template

Récupère un template d'import avec données d'exemple.

### Réponses

```json
{
  "csvExample": "Code Produit;Code-barres;Nom Produit;Quantité Magasin;Quantité Entrepôt;Prix Caisse;Prix Gestcom\nPROD001;1234567890123;Produit A;10;50;15.99;12.50",
  "excelHeaders": [
    "Code Produit",
    "Code-barres",
    "Nom Produit",
    "Quantité Magasin",
    "Quantité Entrepôt",
    "Prix Caisse",
    "Prix Gestcom"
  ],
  "sampleData": [
    {
      "Code Produit": "PROD001",
      "Code-barres": "1234567890123",
      "Nom Produit": "Produit Exemple A",
      "Quantité Magasin": "10",
      "Quantité Entrepôt": "50",
      "Prix Caisse": "15.99",
      "Prix Gestcom": "12.50"
    }
  ],
  "notes": [
    "Les champs 'Code Produit', 'Code-barres' et 'Nom Produit' sont obligatoires",
    "Les prix acceptent virgules ou points comme séparateur décimal",
    "Format CSV: délimiteurs acceptés: virgule, point-virgule ou tabulation"
  ]
}
```

---

## Export API

**Base URL:** `/export`

**Authentification:** Requise (JWT)

---

## POST /export/labels

Exporte une étiquette pour impression.

### Requête

**Body:**
```json
{
  "barcode": "1234567890123",
  "labelQuantity": 5
}
```

### Réponses

**Succès (200 OK):**
```json
{
  "data": "/exports/labels/label_1234567890123_20241211100500.csv",
  "message": "Export de l'étiquette CSV pour le code-barres 1234567890123 réussi",
  "success": true
}
```

### Exemple

```bash
curl -X POST http://localhost:3000/export/labels \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "1234567890123",
    "labelQuantity": 5
  }'
```

---

## POST /export/labels/multi

Exporte plusieurs étiquettes en une fois.

### Requête

**Body:**
```json
{
  "data": [
    {
      "barcode": "1234567890123",
      "labelQuantity": 5
    },
    {
      "barcode": "9876543210987",
      "labelQuantity": 3
    }
  ]
}
```

### Réponses

**Succès (200 OK):**
```json
{
  "data": "/exports/labels/multi_labels_20241211100500.csv",
  "message": "Export multi des étiquettes CSV réussi",
  "success": true
}
```

---

## POST /export/write-offs

Crée et exporte une démarque.

### Requête

**Body:**
```json
{
  "barcode": "1234567890123",
  "writeOffType": "DAMAGE",
  "writeOffQuantity": 2
}
```

**Types de démarque:**
- `DAMAGE`: Dommage
- `THEFT`: Vol
- `UNSOLD`: Invendu

### Réponses

**Succès (200 OK):**
```json
{
  "data": "/exports/writeoffs/writeoff_DAMAGE_20241211100500.csv",
  "message": "Démarque de type 'Dommage' créée et exportée avec succès",
  "success": true
}
```

---

## GET /export/write-offs/:writeOffType

Exporte toutes les démarques d'un type.

### Paramètres

- `writeOffType`: DAMAGE | THEFT | UNSOLD

### Réponses

```json
{
  "data": "/exports/writeoffs/DAMAGE_20241211100500.csv",
  "message": "Export des démarques de type DAMAGE réussi",
  "success": true
}
```

---

## GET /export/write-offs

Exporte toutes les démarques.

### Réponses

```json
{
  "data": "/exports/writeoffs/all_writeoffs_20241211100500.csv",
  "message": "Export de toutes les démarques réussi",
  "success": true
}
```

---

## GET /export/orders/:status

Exporte les commandes par statut.

### Paramètres

- `status`: PENDING | RECEIVED | CANCELED

### Réponses

```json
{
  "data": "/exports/orders/PENDING_20241211100500.csv",
  "message": "Export des commandes avec statut PENDING réussi",
  "success": true
}
```

---

## GET /export/orders

Exporte toutes les commandes.

### Réponses

```json
{
  "data": "/exports/orders/all_orders_20241211100500.csv",
  "message": "Export de toutes les commandes réussi",
  "success": true
}
```

---

## POST /export/receptions

Crée et exporte une réception de commande.

### Requête

**Body:**
```json
{
  "orderId": 15,
  "receivedQuantity": 50,
  "comment": "Réception complète"
}
```

### Réponses

```json
{
  "data": "/exports/receptions/reception_15_20241211100500.csv",
  "message": "Réception exportée avec succès",
  "success": true
}
```

---

## POST /export/receptions/multi

Crée et exporte plusieurs réceptions.

### Requête

**Body:**
```json
[
  {
    "orderId": 15,
    "receivedQuantity": 50,
    "comment": "Réception OK"
  },
  {
    "orderId": 16,
    "receivedQuantity": 30,
    "comment": "Partiel"
  }
]
```

---

## POST /export/inventories

Crée et exporte un inventaire.

### Requête

**Body:**
```json
{
  "barcode": "1234567890123",
  "countedQuantity": 45
}
```

### Réponses

```json
{
  "data": "/exports/inventories/inventory_1234567890123_20241211100500.csv",
  "message": "Inventaire exporté avec succès",
  "success": true
}
```

---

## POST /export/inventories/multi

Crée et exporte plusieurs inventaires.

### Requête

**Body:**
```json
[
  {
    "barcode": "1234567890123",
    "countedQuantity": 45
  },
  {
    "barcode": "9876543210987",
    "countedQuantity": 12
  }
]
```

---

## POST /export/write-offs/multi

Crée et exporte plusieurs démarques.

### Requête

**Body:**
```json
[
  {
    "barcode": "1234567890123",
    "writeOffType": "DAMAGE",
    "writeOffQuantity": 2,
    "comment": "Emballage endommagé"
  },
  {
    "barcode": "9876543210987",
    "writeOffType": "THEFT",
    "writeOffQuantity": 1
  }
]
```

---

## POST /export/disputes

Signale un litige sur une commande.

### Requête

**Body:**
```json
{
  "orderId": 15,
  "receivedQuantity": 40,
  "comment": "10 unités manquantes"
}
```

### Réponses

```json
{
  "data": "/exports/disputes/dispute_15_20241211100500.csv",
  "message": "Litige reporté avec succès",
  "success": true
}
```

---

## POST /export/disputes/multi

Signale plusieurs litiges.

---

## POST /export/price-control

Exporte des étiquettes pour écarts de prix.

### Requête

**Body:**
```json
{
  "data": [
    {
      "barcode": "1234567890123",
      "enteredPrice": 15.99
    },
    {
      "barcode": "9876543210987",
      "enteredPrice": 29.99
    }
  ]
}
```

### Réponses

```json
{
  "data": "/exports/labels/price_control_20241211100500.csv",
  "message": "Export des étiquettes pour écarts de prix réussi",
  "success": true
}
```

---

## Flux d'import complet

```javascript
class ImportManager {
  constructor(token) {
    this.token = token;
  }

  async importFile(file) {
    // 1. Extraire les headers
    const headers = await this.extractHeaders(file);
    
    // 2. Proposer le mapping à l'utilisateur
    const mappings = await this.getUserMappings(headers);
    
    // 3. Valider les mappings
    if (!this.validateMappings(mappings)) {
      throw new Error('Mappings invalides');
    }
    
    // 4. Lancer l'import
    const result = await this.performImport(file, mappings);
    
    // 5. Gérer les erreurs
    this.handleImportResult(result);
    
    return result;
  }

  async extractHeaders(file) {
    const formData = new FormData();
    formData.append('file', file);

    const response = await fetch('/import/headers', {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${this.token}` },
      body: formData
    });

    const data = await response.json();
    return data.headers;
  }

  async getUserMappings(headers) {
    // Interface utilisateur pour mapper les colonnes
    const fields = await this.getAvailableFields();
    
    // Mapping automatique basé sur les noms
    const autoMapping = {};
    headers.forEach(header => {
      const field = fields.find(f => 
        f.label.toLowerCase() === header.toLowerCase()
      );
      if (field) {
        autoMapping[header] = field.value;
      }
    });
    
    return autoMapping;
  }

  validateMappings(mappings) {
    const requiredFields = ['productCode', 'barcodeValue', 'productName'];
    const mappedValues = Object.values(mappings);
    
    return requiredFields.every(field => mappedValues.includes(field));
  }

  async performImport(file, mappings) {
    const formData = new FormData();
    formData.append('file', file);
    formData.append('mappings', JSON.stringify(mappings));

    const response = await fetch('/import/products', {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${this.token}` },
      body: formData
    });

    return await response.json();
  }

  handleImportResult(result) {
    console.log(`✅ Importés: ${result.importedCount}`);
    console.log(`⏭️ Ignorés: ${result.skippedCount}`);
    console.log(`❌ Erreurs: ${result.errorCount}`);
    
    if (result.errors.length > 0) {
      console.error('Erreurs détaillées:', result.errors);
    }
    
    if (result.warnings.length > 0) {
      console.warn('Avertissements:', result.warnings);
    }
  }

  async getAvailableFields() {
    const response = await fetch('/import/fields', {
      headers: { 'Authorization': `Bearer ${this.token}` }
    });
    const data = await response.json();
    return data.fields;
  }
}

// Utilisation
const manager = new ImportManager(token);
const result = await manager.importFile(selectedFile);
```

---

## Bonnes pratiques

### Import

✅ **Valider le fichier** avant l'import (taille, format)
✅ **Proposer un mapping automatique** basé sur les noms de colonnes
✅ **Permettre un aperçu** des données avant l'import
✅ **Afficher les erreurs** de manière claire
✅ **Proposer de corriger** et réimporter en cas d'erreur

### Export

✅ **Nommer les fichiers** avec timestamp pour éviter les conflits
✅ **Nettoyer les anciens exports** régulièrement
✅ **Compresser les gros fichiers** (ZIP)
✅ **Logger les exports** pour audit

### Performance

✅ **Limiter la taille** des fichiers (10 MB max)
✅ **Traiter en chunks** pour les gros fichiers
✅ **Utiliser des jobs async** pour imports longs
✅ **Ajouter une barre de progression** côté client