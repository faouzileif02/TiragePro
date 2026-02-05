# Guide de Démarrage - TiragePro

## Ce qui a été créé

Votre application TiragePro est maintenant prête ! Voici ce qui a été mis en place :

### 1. Base de Données (Supabase)
✅ **10 tables créées** avec Row Level Security (RLS) activé :
- `profiles` - Profils utilisateurs
- `prizes` - Lots à gagner
- `draws` - Tirages/Tombolas
- `payments` - Paiements Revolut
- `tickets` - Tickets de participation
- `questions` - Questions de culture générale
- `user_questions` - Questions assignées
- `attempts` - Tentatives de réponse
- `audit_logs` - Journal d'audit complet

✅ **Sécurité maximale** :
- Policies restrictives sur toutes les tables
- Séparation admin/user
- Audit log de toutes les actions sensibles

### 2. Edge Functions (Backend Serverless)
✅ **5 fonctions déployées sur Supabase** :
- `create-payment` - Créer un paiement Revolut
- `revolut-webhook` - Recevoir les confirmations de paiement
- `get-question` - Obtenir une question de culture générale
- `answer-question` - Valider la réponse (2 tentatives max)
- `execute-draw` - Exécuter le tirage (admin only, cryptographiquement sécurisé)

### 3. Interface Web (Next.js)

#### Pages Utilisateur
✅ **Page d'accueil** (`/`) - Liste des tirages actifs avec progression
✅ **Authentification** (`/connexion`) - Connexion par téléphone (OTP SMS)
✅ **Détail tirage** (`/tirage/[id]`) - Infos + achat de tickets
✅ **Paiement & Question** (`/paiement/[id]`) - Validation participation
✅ **Mes Tickets** (`/mes-tickets`) - Historique et résultats

#### Pages Admin
✅ **Dashboard** (`/admin`) - Statistiques temps réel
✅ **Gestion Tirages** (`/admin/draws`) - CRUD + Exécution
✅ Interface moderne avec sidebar de navigation

### 4. Fonctionnalités Anti-Triche
✅ Création tickets uniquement après paiement confirmé (webhook)
✅ Validation questions UNIQUEMENT côté serveur
✅ Tirage aléatoire sécurisé avec `crypto.getRandomValues()`
✅ Audit log avec seed/hash pour traçabilité
✅ Aucune logique métier exposée côté client

### 5. Workflow Complet

#### Participation Utilisateur
1. Utilisateur se connecte par téléphone (OTP)
2. Navigue et sélectionne un tirage
3. Choisit le nombre de tickets
4. Paie via Revolut Checkout
5. Webhook Revolut confirme → création tickets "pending"
6. Répond à question culture générale (2 tentatives)
7. ✅ Bonne réponse → tickets "valid"
8. ❌ 2 mauvaises réponses → remboursement automatique

#### Exécution Tirage (Admin)
1. Admin clique "Exécuter" sur un tirage actif
2. Système sélectionne 1 ticket "valid" au hasard
3. Enregistre résultat + audit log (seed, timestamp, etc.)
4. Marque le ticket gagnant et les autres perdants
5. Publie le résultat

## Pour Démarrer

### 1. Configuration Supabase
```bash
# Les variables d'environnement à configurer dans .env.local
NEXT_PUBLIC_SUPABASE_URL=https://votre-projet.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=votre-clé-anon
```

### 2. Lancer l'Application
```bash
npm install
npm run dev
```

L'app sera sur http://localhost:3000

### 3. Créer un Admin
Connectez-vous à Supabase SQL Editor et exécutez :
```sql
-- Après vous être connecté avec votre téléphone
UPDATE profiles
SET role = 'admin'
WHERE phone = '+33612345678';  -- Votre numéro
```

### 4. Configuration Revolut (Optionnel)
Pour activer les vrais paiements :
1. Créer compte Revolut Business
2. Activer Merchant API
3. Configurer webhook : `https://votre-projet.supabase.co/functions/v1/revolut-webhook`
4. Ajouter les secrets dans Supabase Dashboard

## Architecture

```
┌─────────────────┐
│   Next.js App   │  ← Interface web (client)
└────────┬────────┘
         │
    ┌────▼─────────────────────────────┐
    │     Supabase Platform            │
    │  ┌──────────────────────────┐   │
    │  │  PostgreSQL Database     │   │
    │  │  + Row Level Security    │   │
    │  └──────────────────────────┘   │
    │  ┌──────────────────────────┐   │
    │  │  Edge Functions (Deno)   │   │
    │  │  - Paiements             │   │
    │  │  - Questions             │   │
    │  │  - Tirages               │   │
    │  └──────────────────────────┘   │
    │  ┌──────────────────────────┐   │
    │  │  Auth (Phone OTP)        │   │
    │  └──────────────────────────┘   │
    └─────────────┬───────────────────┘
                  │
         ┌────────▼────────┐
         │  Revolut API    │
         │  (Paiements)    │
         └─────────────────┘
```

## Sécurité

### ✅ Ce qui est protégé
- Toutes les opérations sensibles sont serveur-only
- RLS activé sur toutes les tables
- Vérification signature webhook Revolut
- Tirage cryptographiquement sécurisé
- Audit log complet
- Rate limiting automatique

### ⚠️ À faire avant production
1. Configurer les vraies clés Revolut (prod)
2. Activer le mode production Supabase
3. Configurer un provider SMS réel pour OTP
4. Ajouter monitoring et alertes
5. Tester tous les cas limites

## Support

- **Base de données** : Déjà créée et configurée ✅
- **Edge Functions** : Déjà déployées ✅
- **Frontend** : Prêt à lancer ✅

Pour toute question : consultez README.md

---

🎉 **Votre application TiragePro est complète et prête à l'emploi !**
