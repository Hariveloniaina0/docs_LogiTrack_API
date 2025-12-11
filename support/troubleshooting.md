# Guide de Dépannage

## Problèmes d'Authentification

### Impossible de se connecter

**Symptômes :**
- Erreur "INVALID_CREDENTIALS"
- Erreur "ACCOUNT_INACTIVE"

**Solutions :**
1. Vérifiez que votre email et mot de passe sont corrects
2. Assurez-vous que votre compte est actif
3. Vérifiez que le storeId est valide
4. Contactez votre administrateur si le problème persiste

**Exemple de requête correcte :**
```bash
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "utilisateur@exemple.com",
    "password": "motdepasse",
    "storeId": 1
  }'
```

---

### Token expiré constamment

**Symptômes :**
- Erreur "TOKEN_EXPIRED" fréquente
- Déconnexions répétées

**Solutions :**
1. Implémentez le refresh automatique du token
2. Vérifiez que vous utilisez bien le refresh token
3. Stockez correctement les tokens (localStorage/cookies)

**Exemple d'implémentation :**
```javascript
async function refreshToken() {
  const refreshToken = localStorage.getItem('refresh_token');
  
  const response = await fetch('/auth/refresh', {
    headers: {
      'Authorization': `Bearer ${refreshToken}`
    }
  });
  
  if (response.ok) {
    const { access_token, refresh_token } = await response.json();
    localStorage.setItem('access_token', access_token);
    localStorage.setItem('refresh_token', refresh_token);
    return true;
  }
  
  return false;
}
```

---

## Problèmes d'Import

### Erreur de parsing du fichier

**Symptômes :**
- "Erreur lors de l'extraction des en-têtes"
- "Format de fichier non supporté"

**Solutions :**
1. Vérifiez que le fichier est en UTF-8
2. Assurez-vous qu'il n'y a pas de caractères spéciaux
3. Testez avec un fichier CSV simple
4. Vérifiez l'extension (.csv, .xlsx, .xls)

**Format correct :**
```csv
Code Produit,Code-barres,Nom Produit,Quantité Magasin
PROD001,1234567890123,Café Arabica,25
PROD002,9876543210987,Thé Vert,15
```

---

### Import partiel ou échoué

**Symptômes :**
- Seulement certaines lignes importées
- Erreurs sur des produits spécifiques

**Solutions :**
1. Consultez le rapport d'import détaillé
2. Vérifiez les erreurs ligne par ligne
3. Corrigez les données problématiques
4. Relancez l'import

**Erreurs courantes :**
- Code-barres dupliqué
- Produit avec code-barres invalide
- Quantités négatives
- Prix négatifs

---

## Problèmes de Communication (Talkie-Walkie)

### Pas de son reçu

**Symptômes :**
- Impossible d'entendre les autres utilisateurs
- Pas de notification de nouveaux messages

**Solutions :**
1. Vérifiez que vous êtes connecté au bon canal
2. Testez la connexion : `GET /talkie-walkie/status`
3. Vérifiez les permissions audio du navigateur
4. Vérifiez le volume de votre appareil
5. Reconnectez-vous : `POST /talkie-walkie/disconnect` puis `POST /talkie-walkie/connect`

---

### Impossible de parler

**Symptômes :**
- Bouton PTT (Push-To-Talk) non réactif
- Message d'erreur lors de l'envoi

**Solutions :**
1. Vérifiez qu'aucun autre utilisateur ne parle : `GET /talkie-walkie/channel/:id/status`
2. Vérifiez les permissions microphone du navigateur
3. Testez avec un message court (< 5 secondes)
4. Vérifiez votre connexion réseau

---

### Déconnexions fréquentes

**Symptômes :**
- Déconnexion toutes les 1-2 minutes
- Message "shouldReconnect: true"

**Solutions :**
1. Implémentez le heartbeat automatique (toutes les 30s)
2. Vérifiez la stabilité de votre connexion réseau
3. Assurez-vous de n'avoir qu'une seule connexion active

**Implémentation heartbeat :**
```javascript
setInterval(async () => {
  const response = await fetch('/talkie-walkie/heartbeat', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ connectionId })
  });
  
  const data = await response.json();
  if (data.shouldReconnect) {
    await reconnect();
  }
}, 30000);
```

---

## Problèmes FTP/SFTP

### Connexion FTP échouée

**Symptômes :**
- "Connexion FTP échouée"
- "Invalid credentials"

**Solutions :**
1. Testez la connexion : `GET /ftp/test/:id`
2. Vérifiez l'hôte, port, username, password
3. Vérifiez que le serveur FTP est accessible
4. Pour SFTP, assurez-vous que le port 22 est utilisé

---

### Fichiers non importés automatiquement

**Symptômes :**
- Import automatique ne se déclenche pas
- Fichiers restent sur le serveur FTP

**Solutions :**
1. Vérifiez la configuration auto-import : `GET /auto-import/config`
2. Vérifiez que `isEnabled` est à `true`
3. Consultez l'historique : `GET /auto-import/history`
4. Vérifiez le `remoteDir` configuré
5. Lancez un import manuel : `POST /auto-import/run-now`

---

## Problèmes de Performance

### API lente

**Symptômes :**
- Temps de réponse > 5 secondes
- Timeouts fréquents

**Solutions :**
1. Utilisez la pagination pour les grandes listes
2. Évitez les requêtes simultanées multiples
3. Utilisez les filtres pour réduire les données
4. Planifiez les gros imports hors heures de pointe

**Exemple avec pagination :**
```bash
GET /products/paginated?page=1&limit=20
```

---

### Base de données lente

**Symptômes :**
- Requêtes de plus en plus lentes
- Timeouts sur certaines opérations

**Solutions :**
1. Nettoyez régulièrement l'historique ancien
2. Archivez les commandes reçues : `DELETE /orders/received`
3. Supprimez les commandes annulées : `DELETE /orders/canceled`
4. Contactez l'administrateur système

---

## Erreurs HTTP Courantes

### 400 Bad Request

**Causes :**
- Données manquantes dans la requête
- Format de données incorrect
- Validation échouée

**Solution :**
Vérifiez le format de votre requête et les données envoyées.

---

### 401 Unauthorized

**Causes :**
- Token manquant ou invalide
- Token expiré

**Solution :**
Rafraîchissez votre token via `/auth/refresh`.

---

### 403 Forbidden

**Causes :**
- Permissions insuffisantes
- Accès à un magasin non autorisé

**Solution :**
Vérifiez vos permissions et le magasin cible.

---

### 404 Not Found

**Causes :**
- Ressource inexistante
- ID incorrect

**Solution :**
Vérifiez l'existence de la ressource et l'ID utilisé.

---

### 500 Internal Server Error

**Causes :**
- Erreur serveur inattendue
- Bug dans l'application

**Solution :**
Contactez le support technique : harivelo@g-fly.fr

---

## Outils de Diagnostic

### Vérifier l'état du système
```bash
# Statut de connexion
GET /auth/profile

# Informations de debug du token
GET /auth/debug/token-info

# Statut talkie-walkie
GET /talkie-walkie/status

# Configuration FTP
GET /ftp

# Historique des imports
GET /auto-import/history
```

---

### Logs à consulter

**Côté client :**
- Console du navigateur (F12)
- Network tab pour voir les requêtes
- Erreurs JavaScript

**Côté serveur :**
- Logs applicatifs NestJS
- Logs base de données
- Logs FTP

---

## Contact Support

Si le problème persiste après avoir suivi ces étapes :

**Email de support :**
harivelo@g-fly.fr

**Informations à fournir :**
- Description détaillée du problème
- Étapes pour reproduire l'erreur
- Messages d'erreur complets
- Logs pertinents
- Version de l'application
- Navigateur/OS utilisé