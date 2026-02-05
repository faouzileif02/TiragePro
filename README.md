# TiragePro - Application de Tirages au Sort

Application complète de tirages au sort (tombola/raffle) avec paiements sécurisés via Revolut et validation par questions de culture générale.

## Fonctionnalités

### Pour les utilisateurs
- Authentification par téléphone (OTP SMS)
- Navigation des tirages en cours
- Achat de tickets avec paiement Revolut
- Validation de participation par question de culture générale
- Suivi des tickets et gains
- Notifications des résultats

### Pour les administrateurs
- Dashboard avec statistiques
- Gestion des lots (CRUD)
- Gestion des tirages (CRUD)
- Exécution sécurisée des tirages
- Gestion des questions
- Suivi des paiements et utilisateurs
- Journal d'audit complet

## Architecture

### Stack Technique
- **Frontend**: Next.js 13 (App Router) + TypeScript + Tailwind CSS
- **Backend**: Supabase Edge Functions (Deno)
- **Base de données**: Supabase PostgreSQL
- **Authentification**: Supabase Auth (Phone OTP)
- **Paiements**: Revolut Hosted Checkout + Webhooks
- **Hébergement**: Vercel (Frontend) + Supabase (Backend/DB)

### Structure du Projet

\`\`\`
tiragepro/
├── app/                          # Pages Next.js
│   ├── page.tsx                  # Page d'accueil (liste tirages)
│   ├── connexion/                # Authentification
│   ├── tirage/[id]/              # Détail d'un tirage
│   ├── paiement/[id]/            # Vérification paiement + question
│   ├── mes-tickets/              # Tickets utilisateur
│   └── admin/                    # Interface admin
│       ├── page.tsx              # Dashboard
│       ├── draws/                # Gestion tirages
│       ├── prizes/               # Gestion lots
│       └── ...
├── lib/
│   ├── types.ts                  # Types TypeScript
│   └── supabase.ts               # Client Supabase
├── supabase/
│   └── functions/                # Edge Functions
│       ├── create-payment/       # Création paiement
│       ├── revolut-webhook/      # Webhook Revolut
│       ├── get-question/         # Récupération question
│       ├── answer-question/      # Validation réponse
│       └── execute-draw/         # Exécution tirage
└── .env.local                    # Variables d'environnement
\`\`\`

## Installation

### Prérequis
- Node.js 18+
- Compte Supabase
- Compte Revolut Business (sandbox pour dev)

### Étapes

1. **Cloner le projet**
\`\`\`bash
git clone <repo-url>
cd tiragepro
npm install
\`\`\`

2. **Configurer Supabase**
   - Créer un projet sur supabase.com
   - La base de données est déjà créée avec toutes les tables
   - Récupérer l'URL du projet et la clé ANON

3. **Configurer les variables d'environnement**
\`\`\`bash
cp .env.example .env.local
\`\`\`

Éditer `.env.local`:
\`\`\`env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
\`\`\`

4. **Les Edge Functions sont déjà déployées**
   - Toutes les fonctions Edge ont été déployées automatiquement
   - Elles sont disponibles sur Supabase

5. **Configurer Revolut (optionnel pour dev)**
   - Créer un compte Revolut Business
   - Activer l'API Merchant
   - Configurer le webhook : `https://your-project.supabase.co/functions/v1/revolut-webhook`
   - Ajouter les clés dans les secrets Supabase

6. **Lancer l'application**
\`\`\`bash
npm run dev
\`\`\`

L'application sera disponible sur http://localhost:3000

## Configuration de la Base de Données

La base de données est déjà configurée avec les tables suivantes:

- **profiles**: Profils utilisateurs (étend auth.users)
- **prizes**: Lots à gagner
- **draws**: Tirages
- **payments**: Paiements Revolut
- **tickets**: Tickets de participation
- **questions**: Questions de culture générale
- **user_questions**: Questions assignées aux utilisateurs
- **attempts**: Tentatives de réponse
- **audit_logs**: Journal d'audit

### Row Level Security (RLS)

Toutes les tables ont RLS activé avec des policies restrictives:
- Les utilisateurs ne voient que leurs propres données
- Les admins ont accès complet
- Les opérations sensibles (tirage, paiements) sont serveur-only

## Workflows Principaux

### 1. Participation à un Tirage

1. Utilisateur sélectionne un tirage et choisit le nombre de tickets
2. Système crée un paiement et génère l'URL Revolut Checkout
3. Utilisateur est redirigé vers Revolut pour payer
4. Webhook Revolut confirme le paiement
5. Système crée les tickets en statut "pending_validation"
6. Utilisateur répond à une question de culture générale
7. Si bonne réponse: tickets passent à "valid"
8. Si mauvaise réponse après 2 tentatives: remboursement

### 2. Exécution d'un Tirage

1. Admin clique sur "Exécuter" pour un tirage actif
2. Système sélectionne aléatoirement un ticket "valid"
3. Utilise `crypto.getRandomValues()` pour garantir l'aléatoire
4. Enregistre le résultat + audit log avec seed
5. Marque le ticket gagnant + les autres perdants
6. Publie le résultat

## Sécurité

### Anti-Triche
- ✅ Aucune logique métier côté client
- ✅ Création tickets uniquement après paiement confirmé
- ✅ Validation questions côté serveur uniquement
- ✅ Tirage exécuté serveur avec crypto sécurisé
- ✅ Audit log complet de toutes les actions
- ✅ RLS strict sur toutes les tables
- ✅ Rate limiting sur les endpoints sensibles

### Paiements
- ✅ Vérification signature webhook Revolut
- ✅ Idempotence des webhooks
- ✅ Aucune clé secrète exposée côté client
- ✅ Validation montants côté serveur

## API Endpoints

### Edge Functions

#### create-payment
- **Method**: POST
- **Auth**: Required
- **Body**: `{ draw_id, ticket_quantity }`
- **Returns**: `{ payment_id, checkout_url, amount }`

#### revolut-webhook
- **Method**: POST
- **Auth**: Webhook signature
- **Body**: Revolut event
- **Returns**: `{ success }`

#### get-question
- **Method**: GET
- **Auth**: Required
- **Query**: `payment_id`
- **Returns**: `{ user_question_id, question, attempts_remaining }`

#### answer-question
- **Method**: POST
- **Auth**: Required
- **Body**: `{ user_question_id, selected_index }`
- **Returns**: `{ is_correct, validated, explanation }`

#### execute-draw
- **Method**: POST
- **Auth**: Required (Admin only)
- **Body**: `{ draw_id }`
- **Returns**: `{ winner, total_participants }`

## Déploiement

### Frontend (Vercel)

1. Connecter le repo GitHub à Vercel
2. Configurer les variables d'environnement
3. Déployer

### Backend (Supabase)

Les Edge Functions sont déjà déployées. Pour les mettre à jour:

1. Modifier le code dans `supabase/functions/`
2. Utiliser l'outil de déploiement fourni

### Base de Données

La base de données est déjà configurée sur Supabase avec toutes les migrations appliquées.

## Tests

Pour tester l'application en local:

1. Utiliser Revolut sandbox mode
2. Créer un compte admin manuellement dans la table `profiles` (role = 'admin')
3. Créer des lots et tirages via l'interface admin
4. Tester le flow complet utilisateur

## Support

Pour toute question ou problème:
- Consulter la documentation Supabase: https://supabase.com/docs
- Consulter la documentation Revolut: https://developer.revolut.com

## Licence

Propriétaire - Tous droits réservés
