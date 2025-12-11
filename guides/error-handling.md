# Gestion des erreurs

## Format standard

Toutes les erreurs suivent ce format :
```json
{
  "message": "Description lisible de l'erreur",
  "code": "ERROR_CODE",
  "timestamp": "2024-12-11T10:00:00.000Z"
}
```

## Codes HTTP courants

### 400 - Bad Request

**Causes :**
- Données manquantes
- Format invalide
- Validation échouée

**Exemples :**
```json
// Store ID manquant
{
  "message": "Store ID is required",
  "code": "MISSING_STORE_ID",
  "timestamp": "2024-12-11T10:00:00Z"
}

// Format de fichier invalide
{
  "message": "Format de fichier non supporté",
  "code": "INVALID_FILE_FORMAT",
  "timestamp": "2024-12-11T10:00:00Z"
}
```

### 401 - Unauthorized

**Causes :**
- Token manquant ou invalide
- Token expiré
- Identifiants incorrects

**Exemples :**
```json
// Token expiré
{
  "message": "Token has expired",
  "code": "TOKEN_EXPIRED",
  "timestamp": "2024-12-11T10:00:00Z"
}

// Identifiants incorrects
{
  "message": "Email ou mot de passe incorrect",
  "code": "INVALID_CREDENTIALS",
  "timestamp": "2024-12-11T10:00:00Z"
}
```

### 403 - Forbidden

**Causes :**
- Rôle insuffisant
- Accès à un magasin non autorisé
- Compte désactivé

**Exemples :**
```json
// Accès interdit
{
  "message": "You do not have access to this store",
  "code": "STORE_ACCESS_DENIED",
  "timestamp": "2024-12-11T10:00:00Z"
}

// Compte inactif
{
  "message": "Account is inactive",
  "code": "ACCOUNT_INACTIVE",
  "timestamp": "2024-12-11T10:00:00Z"
}
```

### 404 - Not Found

**Causes :**
- Ressource inexistante
- ID invalide

**Exemples :**
```json
{
  "message": "Product not found",
  "code": "PRODUCT_NOT_FOUND",
  "timestamp": "2024-12-11T10:00:00Z"
}
```

### 500 - Internal Server Error

**Causes :**
- Erreur de base de données
- Erreur de service externe
- Bug serveur

**Exemples :**
```json
{
  "message": "Une erreur inattendue s'est produite",
  "code": "INTERNAL_ERROR",
  "timestamp": "2024-12-11T10:00:00Z"
}
```

## Gestion côté client

### Intercepteur d'erreurs (JavaScript)
```javascript
async function apiCall(url, options = {}) {
  try {
    const response = await fetch(url, {
      ...options,
      headers: {
        'Authorization': `Bearer ${getToken()}`,
        'Content-Type': 'application/json',
        ...options.headers
      }
    });

    if (!response.ok) {
      const error = await response.json();
      
      // Gestion spécifique par code HTTP
      switch (response.status) {
        case 401:
          // Token expiré, tentative de refresh
          if (error.code === 'TOKEN_EXPIRED') {
            await refreshToken();
            return apiCall(url, options); // Retry
          }
          // Sinon, déconnexion
          logout();
          break;
          
        case 403:
          // Accès interdit
          showError('Accès non autorisé');
          break;
          
        case 404:
          showError('Ressource non trouvée');
          break;
          
        case 500:
          showError('Erreur serveur, veuillez réessayer');
          break;
          
        default:
          showError(error.message || 'Une erreur est survenue');
      }
      
      throw error;
    }

    return await response.json();
  } catch (error) {
    console.error('API Error:', error);
    throw error;
  }
}
```

### Gestion du refresh token
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
  
  // Refresh impossible, déconnexion
  logout();
  return false;
}
```

## Codes d'erreur par module

### Authentication

| Code | Description |
|------|-------------|
| LOGIN_SUCCESS | Connexion réussie |
| INVALID_CREDENTIALS | Identifiants incorrects |
| ACCOUNT_INACTIVE | Compte désactivé |
| USER_NOT_FOUND | Utilisateur non trouvé |
| REFRESH_TOKEN_INVALID | Refresh token invalide |
| PASSWORD_TOO_SHORT | Mot de passe trop court |

### Products

| Code | Description |
|------|-------------|
| PRODUCT_NOT_FOUND | Produit non trouvé |
| MISSING_STORE_ID | Store ID manquant |

### Orders

| Code | Description |
|------|-------------|
| ORDER_NOT_FOUND | Commande non trouvée |
| INVALID_ORDER_STATUS | Statut invalide |

### Import/Export

| Code | Description |
|------|-------------|
| INVALID_FILE_FORMAT | Format fichier invalide |
| FILE_TOO_LARGE | Fichier trop volumineux |
| IMPORT_FAILED | Échec de l'import |
| EXPORT_FAILED | Échec de l'export |

## Bonnes pratiques

### Côté Client

✅ **Toujours gérer les erreurs**
```javascript
try {
  await apiCall('/products');
} catch (error) {
  handleError(error);
}
```

✅ **Afficher des messages utilisateur clairs**
```javascript
const errorMessages = {
  'INVALID_CREDENTIALS': 'Email ou mot de passe incorrect',
  'ACCOUNT_INACTIVE': 'Votre compte a été désactivé',
  'STORE_ACCESS_DENIED': 'Vous n\'avez pas accès à ce magasin'
};

function showError(code) {
  const message = errorMessages[code] || 'Une erreur est survenue';
  alert(message);
}
```

✅ **Logger les erreurs pour le debug**
```javascript
console.error('[API Error]', {
  url,
  status,
  code: error.code,
  message: error.message,
  timestamp: error.timestamp
});
```

### Côté Serveur

✅ **Utiliser des codes d'erreur cohérents**
✅ **Logger les erreurs avec contexte**
✅ **Ne jamais exposer les détails techniques**
❌ **Ne jamais logger les mots de passe ou tokens**