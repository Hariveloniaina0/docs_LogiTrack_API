# Services de l'Application

## Vue d'ensemble

L'application de gestion logistique offre une suite complète de services pour optimiser vos opérations quotidiennes. Chaque service est conçu pour répondre à des besoins spécifiques de la chaîne logistique.

---

## Catalogue des Services

### Authentification et Sécurité
Gestion complète des utilisateurs, rôles et accès sécurisés.

**Fonctionnalités clés :**
- Connexion JWT sécurisée
- Gestion des rôles (Admin, Manager, Employé)
- Récupération de mot de passe
- Sessions multi-appareils

[Voir la documentation complète](api/authentication.md)

---

### Gestion des Stocks
Suivi en temps réel de vos inventaires multi-magasins.

**Fonctionnalités clés :**
- Stock magasin et entrepôt
- Historique des mouvements
- Alertes de rupture
- Rapports de stock

[Voir la documentation complète](services/stock-management.md)

---

### Import/Export de Données
Automatisation complète des échanges de données.

**Fonctionnalités clés :**
- Import CSV/Excel automatique
- Export multi-formats
- Synchronisation FTP/SFTP
- Historique des imports

[Voir la documentation complète](services/import-export.md)

---

### Gestion des Commandes
Pilotage complet du cycle de commande.

**Fonctionnalités clés :**
- Création et suivi de commandes
- Gestion des réceptions
- Traitement des litiges
- Statuts en temps réel

[Voir la documentation complète](services/orders.md)

---

### Impression d'Étiquettes
Génération automatique d'étiquettes produits.

**Fonctionnalités clés :**
- Étiquettes individuelles ou en masse
- Format personnalisable
- Code-barres automatique
- Export direct

[Voir la documentation complète](services/labels.md)

---

### Gestion des Casses
Traçabilité des pertes et anomalies.

**Fonctionnalités clés :**
- Types de casse (périmé, défaut, casse)
- Export par type
- Reporting automatique
- Validation multi-niveaux

[Voir la documentation complète](services/import-export.md)

---

### Inventaires
Comptages physiques et écarts de stock.

**Fonctionnalités clés :**
- Saisie d'inventaire
- Calcul d'écarts automatique
- Export des résultats
- Historique complet

[Voir la documentation complète](services/inventory.md)

---

### Demandes d'Affichage
Système de demande d'affichage produit par email.

**Fonctionnalités clés :**
- Création de demande avec détails produit
- Envoi automatique par email
- Templates personnalisés
- Gestion des destinataires

[Voir la documentation complète](services/display-requests.md)

---

### Talkie-Walkie Numérique
Communication vocale instantanée entre équipes.

**Fonctionnalités clés :**
- Canaux hiérarchiques
- Audio en temps réel
- Diffusion d'urgence
- Gestion des présences

[Voir la documentation complète](services/talkie-walkie.md)

---

### Synchronisation FTP
Échanges automatiques avec systèmes externes.

**Fonctionnalités clés :**
- FTP et SFTP supportés
- Import/Export automatique
- Configuration multi-magasins
- Gestion d'erreurs robuste

[Voir la documentation complète](services/ftp-sync.md)

---

### Gestion Multi-Magasins
Architecture centralisée pour réseaux de magasins.

**Fonctionnalités clés :**
- Isolation des données par magasin
- Administration centralisée
- Rapports consolidés
- Configuration indépendante

[Voir la documentation complète](services/multi-store.md)

---

## Services par Profil Utilisateur

### Pour les Administrateurs
- Gestion complète des utilisateurs et magasins
- Configuration FTP/SFTP
- Accès à tous les rapports
- Diffusions d'urgence globales
- Supervision de tous les canaux

### Pour les Managers
- Gestion de leur magasin
- Import/Export de données
- Validation des commandes
- Diffusions d'urgence magasin
- Accès canal management

### Pour les Employés
- Saisie des stocks
- Scan de produits
- Demandes d'affichage
- Communication talkie-walkie
- Consultation des stocks

---

## Intégrations Techniques

### Formats supportés
- **Import** : CSV, XLSX, XLS
- **Export** : CSV, XLSX, TXT
- **Protocoles** : FTP, SFTP, HTTP/HTTPS
- **Encodage** : UTF-8, ISO-8859-1

### APIs disponibles
- REST API complète
- WebSocket pour temps réel
- JWT pour authentification
- SSE pour événements

---

## Accès Multi-Plateformes

Tous les services sont accessibles via :
- **Web** : Interface responsive
- **Mobile** : Application native (iOS/Android)
- **API** : Intégrations tierces

---

## Support et Assistance

Pour toute question sur les services :
- Email : support@votre-entreprise.com
- Téléphone : +33 X XX XX XX XX
- Chat en direct : disponible dans l'application

---

## Mises à Jour

Cette documentation est régulièrement mise à jour. Consultez le [changelog](CHANGELOG.md) pour suivre les évolutions.

**Dernière mise à jour** : Décembre 2024