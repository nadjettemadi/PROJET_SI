# Fret-Dz : Extranet Logistique B2B 🚚

Plateforme de mise en relation entre Commerçants et Camionneurs en Algérie.

## 🏗️ Architecture & Mapping

Le projet suit une architecture moderne **Serverless** basée sur la stack **Next.js/React + Supabase**.

### Mapping des ressources (A, B, C, Fichier)
- **Table A (Utilisateurs)** : Les Commerçants. Gérés via `Supabase Auth` (ID lié à `auth.users`).
- **Table B (Ressources)** : Les Camionneurs. Table `public.camionneurs` contenant les informations des transporteurs disponibles.
- **Table C (Interactions)** : Les Expéditions. Table `public.expeditions` liant un commerçant à un chauffeur pour une mission précise.
- **Fichier (Document)** : Bon de livraison (BL). Stocké dans `Supabase Storage` (bucket: `bons-de-livraison`), avec l'URL sauvegardée dans la Table C.

---

## 🧐 Analyse Technique & Stratégique

### 💰 CAPEX vs OPEX
- **CAPEX (Capital Expenditure)** : Habituellement élevé pour une logistique traditionnelle (achat de serveurs, infrastructure physique, maintenance). Avec cette stack, le CAPEX est **quasi-nul** (développement initial uniquement).
- **OPEX (Operating Expenditure)** : On passe à un modèle de **coûts variables**. Vous payez uniquement ce que vous consommez (modèle de facturation par utilisateur ou volume de données de Supabase/Vercel). C'est idéal pour une startup logistique qui veut scaler sans gros investissement de départ.

### 📈 Scalabilité avec Vercel
Le frontend est déployé sur le réseau Edge de Vercel. 
- **Performance** : Les pages sont servies depuis le serveur le plus proche de l'utilisateur (faible latence en Algérie).
- **Elasticité** : Vercel gère automatiquement les pics de trafic (par exemple, lors d'une période de forte activité commerciale comme le Ramadan) sans intervention manuelle.

### 🗄️ Données Structurées vs Non-Structurées
- **Données Structurées** : Stockées dans le SQL de Supabase (PostgreSQL). Cela garantit l'intégrité des données (tables liées, types de données stricts, RLS). Exemple : Relations entre expéditions et utilisateurs.
- **Données Non-Structurées** : Les scans de Bons de Livraison (images/PDF). Ils ne rentrent pas dans un tableau SQL. On utilise le **Blob Storage** pour les stocker efficacement et ne garder qu'une référence (URL) dans la base de données.

---

## 🚀 Guide de Déploiement

### 1. Configuration Supabase
1. Créez un projet sur [Supabase](https://supabase.com).
2. Exécutez le script SQL fourni dans l'onglet **SQL Editor**.
3. Dans **Storage**, créez un bucket public nommé `bons-de-livraison`.
4. Dans **Authentication > URL Configuration**, ajoutez votre URL Vercel (une fois générée).

### 2. Déploiement Vercel
1. Poussez votre code sur un dépôt **GitHub**.
2. Connectez votre compte GitHub sur [Vercel](https://vercel.com).
3. Importez le projet.
4. Ajoutez les **Variables d'Environnement** suivantes :
   - `VITE_SUPABASE_URL`
   - `VITE_SUPABASE_ANON_KEY`
5. Cliquez sur **Deploy**.

---

## 🔐 Sécurité RLS
Chaque requête vers Supabase est filtrée au niveau de la base de données. Même si l'application frontend est compromise, un utilisateur ne pourra jamais lire les expéditions d'un autre commerçant grâce à la clause :
`USING (auth.uid() = commercant_id)`
