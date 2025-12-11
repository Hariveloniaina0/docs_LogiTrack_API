# Questions Fréquemment Posées

## Authentification et Accès

### Comment me connecter pour la première fois ?
1. Demandez vos identifiants à votre administrateur
2. Récupérez la liste de vos magasins via l'endpoint `/auth/user-stores`
3. Connectez-vous avec votre email, mot de passe et l'ID du magasin souhaité

### Que faire si j'ai oublié mon mot de passe ?
Contactez votre administrateur qui pourra réinitialiser votre mot de passe.

### Mon token est expiré, que faire ?
Utilisez l'endpoint `/auth/refresh` avec votre refresh token pour obtenir un nouveau token d'accès.

### Comment changer de magasin ?
Vous devez vous reconnecter en spécifiant un autre `storeId` lors du login.

---

## Gestion des Stocks

### Comment importer mes produits ?
1. Préparez un fichier CSV/Excel avec les colonnes requises
2. Utilisez l'endpoint `/import/headers` pour valider le format
3. Créez un mapping des colonnes
4. Lancez l'import via `/import/products`

### Mon code-barres contient des zéros, est-ce normal ?
Le système normalise automatiquement les codes-barres en supprimant les zéros non significatifs.

### Puis-je avoir des stocks négatifs ?
Non, le système refuse les quantités négatives pour garantir la cohérence des données.

---

## Import/Export

### Quels formats de fichiers sont supportés ?
CSV, Excel (.xlsx, .xls) pour l'import. CSV et TXT pour l'export.

### Quelle est la taille maximale d'un fichier ?
10 MB par fichier. Au-delà, il est recommandé de diviser en plusieurs fichiers.

### Comment configurer l'import FTP automatique ?
Seuls les administrateurs peuvent configurer le FTP via l'endpoint `/ftp`. Contactez votre admin.

### Les imports sont lents, que faire ?
Pour des fichiers >10,000 lignes, privilégiez les imports hors heures d'ouverture.

---

## Commandes

### Comment créer une commande ?
Utilisez l'endpoint `/orders` avec le productId, la quantité et optionnellement la date.

### Que faire en cas d'écart à la réception ?
Le système détecte automatiquement les écarts. Vous pouvez créer un litige via `/orders/dispute`.

### Puis-je annuler une commande reçue ?
Non, une fois reçue, une commande ne peut plus être annulée.

---

## Talkie-Walkie

### Pourquoi je n'entends personne ?
Vérifiez que vous êtes bien connecté au bon canal et que le volume de votre appareil est activé.

### Je ne peux pas parler, pourquoi ?
Un seul utilisateur peut parler à la fois par canal. Attendez que le locuteur actuel termine.

### Comment fonctionne le heartbeat ?
Votre client doit envoyer un heartbeat toutes les 30 secondes pour maintenir la connexion active.

---

## Étiquettes

### Comment générer des étiquettes ?
Utilisez `/export/labels` avec le code-barres et la quantité souhaitée.

### Puis-je générer plusieurs étiquettes en une fois ?
Oui, utilisez `/export/labels/multi` avec un tableau de produits.

---

## Erreurs Courantes

### "INVALID_CREDENTIALS"
Vérifiez votre email et mot de passe. Assurez-vous que votre compte est actif.

### "STORE_ACCESS_DENIED"
Vous tentez d'accéder à un magasin auquel vous n'êtes pas assigné.

### "PRODUCT_NOT_FOUND"
Le produit recherché n'existe pas dans votre magasin. Vérifiez le code-barres.

### "TOKEN_EXPIRED"
Votre session a expiré. Utilisez `/auth/refresh` pour obtenir un nouveau token.

---

## Questions Techniques

### L'API supporte-t-elle HTTPS ?
Oui, HTTPS est obligatoire en production pour sécuriser les communications.

### Quelle est la limite de requêtes par minute ?
Il n'y a pas de limite globale, mais un rate limiting peut être appliqué sur les endpoints sensibles.

### Puis-je utiliser plusieurs tokens simultanément ?
Oui, mais chaque token est lié à un magasin spécifique.

---

## Support

### Qui contacter pour un problème technique ?
Envoyez un email à : harivelo@g-fly.fr

### Où trouver la documentation complète de l'API ?
Consultez la section [API Technique](../api/authentication.md) de cette documentation.

### Comment signaler un bug ?
Contactez le développeur à harivelo@g-fly.fr avec une description détaillée du problème.