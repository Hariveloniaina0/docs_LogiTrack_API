# Guides Pas-à-Pas

## Guide de Démarrage Complet

### Première Connexion Administrateur

#### Étape 1 : Obtenir vos Identifiants

Lors de l'installation du système, un compte administrateur par défaut est créé :

```
Email : admin@votre-entreprise.com
Magasin : votre magasin
Mot de passe : (fourni lors de l'installation)
```

#### Étape 2 : Première Connexion

1. **Récupérer la liste des magasins disponibles**

```bash
curl -X POST http://localhost:3000/auth/user-stores \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@votre-entreprise.com"
  }'
```

**Réponse :**
```json
{
  "stores": [
    {
      "idStore": 1,
      "storeName": "Magasin Principal",
    }
  ]
}
```

2. **Se connecter avec le magasin**

```bash
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@votre-entreprise.com",
    "password": "votre_mot_de_passe",
    "storeId": 1
  }'
```

**Réponse :**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "message": "Connexion réussie",
  "code": "LOGIN_SUCCESS"
}
```

3. **Sauvegarder les tokens**

```javascript
// Dans votre application
localStorage.setItem('access_token', access_token);
localStorage.setItem('refresh_token', refresh_token);
```

#### Étape 3 : Vérifier votre Profil

```bash
curl -X GET http://localhost:3000/auth/profile \
  -H "Authorization: Bearer {votre_access_token}"
```

**Réponse :**
```json
{
  "user": {
    "idUser": 1,
    "emailAddress": "admin@votre-entreprise.com",
    "firstName": "Admin",
    "lastName": "Système",
    "role": "ADMIN",
    "isActive": true,
    "stores": [
      {
        "idStore": 1,
        "storeName": "Magasin Principal"
      }
    ]
  }
}
```

#### Étape 4 : Changer votre Mot de Passe

Pour des raisons de sécurité, changez immédiatement votre mot de passe par défaut :

```bash
curl -X POST http://localhost:3000/auth/change-password \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "oldPassword": "mot_de_passe_par_defaut",
    "newPassword": "VotreNouveauMotDePasseSecurise123!"
  }'
```

---




## Configuration FTP pour Import/Export

### Prérequis

Vous devez disposer :
- D'un serveur FTP ou SFTP accessible
- Des identifiants de connexion (host, port, username, password)
- D'un répertoire dédié sur le serveur FTP

### Étape 1 : Créer la Configuration FTP

**Pour l'import automatique :**

```bash
curl -X POST http://localhost:3000/ftp \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "host": "ftp.votre-serveur.com",
    "port": 22,
    "username": "import_user",
    "password": "motdepasse_ftp_securise",
    "remotePath": "/imports",
    "protocol": "sftp",
    "type": "import",
    "isActive": true,
    "storeId": 1
  }'
```

**Pour l'export automatique :**

```bash
curl -X POST http://localhost:3000/ftp \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "host": "ftp.votre-serveur.com",
    "port": 22,
    "username": "export_user",
    "password": "motdepasse_ftp_securise",
    "remotePath": "/exports",
    "protocol": "sftp",
    "type": "export",
    "isActive": true,
    "storeId": 1
  }'
```

**Réponse :**
```json
{
  "data": {
    "idFtp": 1,
    "host": "ftp.votre-serveur.com",
    "port": 22,
    "username": "import_user",
    "remotePath": "/imports",
    "protocol": "sftp",
    "type": "import",
    "isActive": true,
    "storeId": 1
  },
  "message": "Configuration FTP créée avec succès",
  "success": true
}
```

### Étape 2 : Tester la Connexion FTP

```bash
curl -X GET http://localhost:3000/ftp/test/1 \
  -H "Authorization: Bearer {votre_access_token}"
```

**Réponse en cas de succès :**
```json
{
  "success": true,
  "message": "Connexion FTP réussie"
}
```

**Réponse en cas d'échec :**
```json
{
  "success": false,
  "message": "Échec de la connexion: Invalid credentials"
}
```

Si le test échoue :
- Vérifiez l'host et le port
- Vérifiez les identifiants
- Assurez-vous que le serveur FTP est accessible depuis votre réseau
- Pour SFTP, vérifiez que le port 22 n'est pas bloqué

---

## Préparation des Fichiers CSV

### Format des Fichiers pour l'Import

#### Fichier Produits (produits.csv)

**Template recommandé :**

```csv
Code Produit,Code-barres,Nom Produit,Quantité Magasin,Quantité Entrepôt,Prix Caisse,Prix Gestcom,Nom Fournisseur
PROD001,3760123456789,Café Arabica Bio 1kg,25,100,12.99,10.50,Fournisseur Café SA
PROD002,3760987654321,Thé Vert Bio 50g,15,80,8.50,6.80,Fournisseur Thé SARL
PROD003,3760555666777,Chocolat Noir 70% 100g,30,150,4.99,3.80,Chocolats Délice
PROD004,3760111222333,Huile d'Olive Extra Vierge 500ml,20,60,15.99,12.50,Huiles du Sud
PROD005,3760444555666,Miel de Lavande 250g,10,40,9.99,7.50,Apiculteurs Provence
```

**Règles importantes :**
- Encodage : UTF-8
- Délimiteur : virgule, point-virgule ou tabulation
- Colonnes obligatoires : Code Produit, Code-barres, Nom Produit
- Pas d'espaces inutiles
- Prix avec point comme séparateur décimal (12.99 et non 12,99)

#### Fichier Commandes (commandes.csv)

```csv
Code-barres,Quantité,Date de commande
3760123456789,50,2024-12-15
3760987654321,30,2024-12-15
3760555666777,100,2024-12-15
3760111222333,40,2024-12-16
3760444555666,25,2024-12-16
```

### Étape 3 : Déposer les Fichiers sur le Serveur FTP

**Avec un client FTP (FileZilla, WinSCP) :**

1. Connectez-vous au serveur FTP avec vos identifiants
2. Naviguez vers le répertoire `/imports`
3. Uploadez vos fichiers CSV
4. Vérifiez que les fichiers sont bien présents

**Avec la ligne de commande (SFTP) :**

```bash
sftp import_user@ftp.votre-serveur.com
> cd /imports
> put produits.csv
> put commandes.csv
> ls
> quit
```

**Structure recommandée sur le serveur FTP :**

```
/imports/
  ├── produits.csv
  ├── commandes.csv
  └── archive/
      ├── produits_20241211.csv
      └── commandes_20241211.csv

/exports/
  ├── labels/
  ├── writeoffs/
  ├── orders/
  └── receptions/
```

---

## Activation de l'Import Automatique

### Étape 1 : Vérifier la Configuration Actuelle

```bash
curl -X GET http://localhost:3000/auto-import/config \
  -H "Authorization: Bearer {votre_access_token}"
```

**Réponse :**
```json
{
  "id": 1,
  "isEnabled": false,
  "frequencyHours": 24,
  "lastRunAt": null,
  "nextRunAt": null
}
```

### Étape 2 : Configurer la Fréquence

**Options de fréquence recommandées :**
- 6 heures : Import 4 fois par jour (8h, 14h, 20h, 2h)
- 12 heures : Import 2 fois par jour (2h, 14h)
- 24 heures : Import 1 fois par jour (2h)

```bash
curl -X PUT http://localhost:3000/auto-import/config \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "isEnabled": true,
    "frequencyHours": 6
  }'
```

**Réponse :**
```json
{
  "id": 1,
  "isEnabled": true,
  "frequencyHours": 6,
  "lastRunAt": null,
  "nextRunAt": "2024-12-12T02:00:00Z"
}
```

### Étape 3 : Lancer un Premier Import Manuel

Avant d'activer l'automatisation, testez avec un import manuel :

```bash
curl -X POST http://localhost:3000/auto-import/run-now \
  -H "Authorization: Bearer {votre_access_token}"
```

**Réponse :**
```json
{
  "id": 1,
  "status": "SUCCESS",
  "startedAt": "2024-12-11T14:30:00Z",
  "completedAt": "2024-12-11T14:35:23Z",
  "recordsProcessed": 500,
  "recordsImported": 490,
  "recordsFailed": 10,
  "errorMessage": null,
  "fileDetails": [
    {
      "fileName": "produits.csv",
      "fileType": "products",
      "status": "success",
      "productsImported": 485,
      "productsUpdated": 5,
      "errors": []
    },
    {
      "fileName": "commandes.csv",
      "fileType": "orders",
      "status": "partial",
      "ordersImported": 5,
      "errors": [
        "Produit avec barcode 9999999999999 non trouvé"
      ]
    }
  ]
}
```

### Étape 4 : Vérifier l'Historique

```bash
curl -X GET http://localhost:3000/auto-import/history \
  -H "Authorization: Bearer {votre_access_token}"
```

**Réponse :**
```json
[
  {
    "id": 1,
    "status": "SUCCESS",
    "startedAt": "2024-12-11T14:30:00Z",
    "completedAt": "2024-12-11T14:35:23Z",
    "recordsProcessed": 500,
    "recordsImported": 490,
    "recordsFailed": 10
  }
]
```

### Étape 5 : Surveillance Continue

**Consulter le dernier import :**

```bash
curl -X GET http://localhost:3000/auto-import/latest \
  -H "Authorization: Bearer {votre_access_token}"
```

**Voir les statistiques :**

```bash
curl -X GET http://localhost:3000/auto-import/stats \
  -H "Authorization: Bearer {votre_access_token}"
```

---

## Manipulation de la Gestion Logistique

### Consultation des Stocks

#### Voir tous les produits

```bash
curl -X GET http://localhost:3000/products \
  -H "Authorization: Bearer {votre_access_token}"
```

#### Recherche paginée

```bash
curl -X GET "http://localhost:3000/products/paginated?page=1&limit=20&search=café" \
  -H "Authorization: Bearer {votre_access_token}"
```

#### Recherche par code-barres

```bash
curl -X GET http://localhost:3000/products/barcode/3760123456789 \
  -H "Authorization: Bearer {votre_access_token}"
```

### Gestion des Commandes

#### Créer une commande

```bash
curl -X POST http://localhost:3000/orders \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "productId": 1,
    "orderQuantity": 50,
    "supplierId": 1,
    "orderDate": "2024-12-15T10:00:00Z"
  }'
```

#### Voir les commandes en attente

```bash
curl -X GET "http://localhost:3000/orders?status=PENDING" \
  -H "Authorization: Bearer {votre_access_token}"
```

#### Réceptionner une commande

```bash
curl -X PUT http://localhost:3000/orders/1/receive \
  -H "Authorization: Bearer {votre_access_token}"
```

#### Signaler un litige

```bash
curl -X POST http://localhost:3000/orders/dispute \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": 1,
    "receivedQuantity": 45,
    "comment": "5 unités manquantes, carton endommagé"
  }'
```

### Génération d'Étiquettes

#### Étiquette simple

```bash
curl -X POST http://localhost:3000/export/labels \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "3760123456789",
    "labelQuantity": 5
  }'
```

#### Multi-étiquettes

```bash
curl -X POST http://localhost:3000/export/labels/multi \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "data": [
      {
        "barcode": "3760123456789",
        "labelQuantity": 5
      },
      {
        "barcode": "3760987654321",
        "labelQuantity": 3
      }
    ]
  }'
```

### Gestion des Casses

#### Déclarer une casse

```bash
curl -X POST http://localhost:3000/export/write-offs \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "3760123456789",
    "writeOffType": "DAMAGE",
    "writeOffQuantity": 2,
    "comment": "Emballage détérioré pendant transport"
  }'
```

#### Exporter les casses

```bash
# Toutes les casses
curl -X GET http://localhost:3000/export/write-offs \
  -H "Authorization: Bearer {votre_access_token}"

# Par type
curl -X GET http://localhost:3000/export/write-offs/DAMAGE \
  -H "Authorization: Bearer {votre_access_token}"
```

### Inventaires

#### Créer un inventaire

```bash
curl -X POST http://localhost:3000/inventory \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "productId": 1,
    "countedQuantity": 23,
    "userId": 5
  }'
```

#### Inventaire multiple

```bash
curl -X POST http://localhost:3000/inventory/multi \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '[
    {
      "productId": 1,
      "countedQuantity": 23,
      "userId": 5
    },
    {
      "productId": 2,
      "countedQuantity": 14,
      "userId": 5
    }
  ]'
```

### Demandes d'Affichage

**Configuration SMTP (une seule fois) :**

Dans votre fichier `.env` :

```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=noreply@votre-entreprise.com
SMTP_PASS=votre_mot_de_passe_app
SMTP_FROM=noreply@votre-entreprise.com
DISPLAY_RECIPIENTS=impression@votre-entreprise.com,marketing@votre-entreprise.com
```

**Créer une demande :**

```bash
curl -X POST http://localhost:3000/display/request \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "productId": 1,
    "quantity": 5,
    "format": "A4",
    "desiredDate": "2024-12-15",
    "recipientEmail": "manager.local@votre-entreprise.com",
    "comment": "Pour nouvelle promotion de Noël"
  }'
```

---

#### Étape 1 : Créer les Magasins

```bash
curl -X POST http://localhost:3000/stores \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "storeName": "Magasin Paris Centre",
    "address": "10 Rue de Rivoli, 75001 Paris",
    "city": "Paris",
    "postalCode": "75001",
    "phone": "+33 1 23 45 67 89",
    "email": "paris.centre@votre-entreprise.com"
  }'
```

#### Étape 2 : Créer les Utilisateurs

**Créer un Manager :**

```bash
curl -X POST http://localhost:3000/users \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "userName": "Jean Dupont",
    "emailAddress": "jean.dupont@votre-entreprise.com",
    "password": "MotDePasseSecurise123!",
    "userRole": "MANAGER",
    "storeIds": [1]
  }'
```

**Créer un Employé :**

```bash
curl -X POST http://localhost:3000/users \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "userName": "Marie Martin",
    "emailAddress": "marie.martin@votre-entreprise.com",
    "password": "MotDePasseSecurise123!",
    "userRole": "EMPLOYEE",
    "storeIds": [1]
  }'
```

---
---

## Workflow Quotidien Recommandé

### Matin (8h00)

1. **Vérifier les imports de la nuit**
```bash
curl -X GET http://localhost:3000/auto-import/latest \
  -H "Authorization: Bearer {votre_access_token}"
```

2. **Consulter les commandes en attente**
```bash
curl -X GET "http://localhost:3000/orders?status=PENDING" \
  -H "Authorization: Bearer {votre_access_token}"
```

3. **Vérifier les alertes de stock**
```bash
curl -X GET "http://localhost:3000/products?lowStock=true" \
  -H "Authorization: Bearer {votre_access_token}"
```

### Journée

4. **Réceptionner les livraisons**
- Scanner les produits reçus
- Comparer avec les commandes
- Signaler les écarts

5. **Traiter les demandes**
- Demandes d'affichage
- Génération d'étiquettes
- Déclaration de casses

### Soir (18h00)

6. **Consulter les statistiques du jour**
```bash
curl -X GET http://localhost:3000/auto-import/stats \
  -H "Authorization: Bearer {votre_access_token}"
```

7. **Exporter les rapports**
```bash
# Commandes reçues du jour
curl -X GET "http://localhost:3000/orders?startDate=2024-12-11&endDate=2024-12-11&status=RECEIVED" \
  -H "Authorization: Bearer {votre_access_token}"
```

---

## Maintenance Régulière

### Hebdomadaire

**Nettoyer les commandes terminées :**

```bash
# Supprimer les commandes reçues
curl -X DELETE http://localhost:3000/orders/received \
  -H "Authorization: Bearer {votre_access_token}"

# Supprimer les commandes annulées
curl -X DELETE http://localhost:3000/orders/canceled \
  -H "Authorization: Bearer {votre_access_token}"
```

### Mensuelle

**Nettoyer l'historique d'imports :**

```bash
curl -X POST http://localhost:3000/auto-import/cleanup \
  -H "Authorization: Bearer {votre_access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "daysToKeep": 30
  }'
```

**Vérifier les configurations FTP :**

```bash
curl -X GET http://localhost:3000/ftp \
  -H "Authorization: Bearer {votre_access_token}"
```

---

## Résolution Rapide des Problèmes

### Import échoué

1. Consulter l'historique détaillé
2. Vérifier les fichiers sur le FTP
3. Tester la connexion FTP
4. Relancer manuellement l'import

### Connexion perdue

1. Rafraîchir le token
2. Se reconnecter si nécessaire
3. Vérifier que le compte est actif

### Produit non trouvé

1. Vérifier l'import des produits
2. Rechercher par code produit
3. Créer manuellement si nécessaire

---

## Checklist de Démarrage

- [ ] Première connexion administrateur effectuée
- [ ] Mot de passe administrateur changé
- [ ] Magasins créés
- [ ] Utilisateurs créés (managers et employés)
- [ ] Configuration FTP import créée et testée
- [ ] Configuration FTP export créée et testée
- [ ] Fichiers CSV préparés et uploadés
- [ ] Premier import manuel réussi
- [ ] Import automatique activé
- [ ] Fréquence configurée
- [ ] Configuration SMTP pour demandes d'affichage (optionnel)
- [ ] Premier test de chaque fonctionnalité effectué

---

## Contact Support

Pour toute question sur ces procédures :

Email : harivelo@g-fly.fr

Consultez aussi :
- [FAQ](faq.md)
- [Guide de dépannage](troubleshooting.md)
- [Documentation API complète](../api/authentication.md)