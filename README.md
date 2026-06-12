# ToqueHub RNM - Module de Synchronisation & API FranceAgriMer

Ce projet est un module de scraping, de synchronisation de base de données et d'API REST pour récupérer les cours des denrées alimentaires du **Réseau des Nouvelles des Marchés (RNM)** de FranceAgriMer.

Il est construit avec **Next.js 15 (App Router)**, **TypeScript**, et **Prisma** pour s'interfacer avec une base de données **PostgreSQL**.

---

## 🚀 Fonctionnalités
- **Crawl automatique** : Découverte des produits disponibles sur les 4 principaux secteurs RNM (*Fruits et Légumes, Pêche et Aquaculture, Beurre-Œuf-Fromage, Viande*).
- **Importation intelligente** : Téléchargement et parsing des fichiers tableurs au format **SYLK (.slk)** générés par FranceAgriMer.
- **Normalisation des données** : Nettoyage et harmonisation des calibres, origines et variétés pour éviter les doublons.
- **Tâche planifiée automatique (Cron)** : Script configurable pour récupérer les prix chaque jour à 2h00 du matin.
- **API REST propre** : Endpoints prêts à l'emploi pour alimenter d'autres applications.
- **Dashboard web** : Interface graphique Next.js intégrée avec historique de prix et variations.

---

## 🗄️ Architecture de la Base de Données

La base de données PostgreSQL utilise le schéma suivant (géré via Prisma) :
- **MarketSector** : Les grands secteurs (ex: `fruits-et-legumes`, `viande`).
- **MarketCategory** : Les catégories et sous-catégories (ex: `legumes`, `fruits-graines`).
- **MarketProduct** : Le produit générique (ex: `tomate`, `cabillaud`) avec son `especeId` RNM.
- **MarketLabel** : Les variétés spécifiques d'un produit (ex: `TOMATE grappe colis 5kg`), identifiées par leur `libcod` unique.
- **Market** : Les lieux de cotation physique (ex: `Rungis`, `Belgique`).
- **MarketPrice** : Les relevés de prix journaliers avec le prix moyen (`avgPrice`), min/max, la variation, le stade commercial et l'unité de mesure.

---

## 📡 Documentation de l'API REST (Préfixe : `/api/rnm`)

### 1. Structure Hiérarchique
- **Endpoint** : `GET /api/rnm/hierarchy`
- **Description** : Retourne l'arbre complet des secteurs, catégories et produits. Idéal pour construire des menus de navigation.

### 2. Recherche & Filtres de Produits
- **Endpoint** : `GET /api/rnm/products`
- **Description** : Liste tous les produits de la base de données.
- **Paramètres acceptés** :
  - `search` (optionnel) : Recherche sur le nom (ex: `?search=tom`)
  - `sector` (optionnel) : Filtrer par slug de secteur (ex: `?sector=viande`)
  - `category` (optionnel) : Filtrer par slug de catégorie (ex: `?category=legumes`)
  - `limit` (défaut `50`) & `offset` (défaut `0`) pour la pagination.

### 3. Fiche Produit & Relevés les plus Récents (Snapshot)
- **Endpoint** : `GET /api/rnm/products/[slug]`
- **Description** : Retourne les détails du produit et **l'intégralité des cotations disponibles à la date la plus récente** (snapshot complet).
- **Exemple** : `GET /api/rnm/products/tomate`

### 4. Recherche Historique des Prix
- **Endpoint** : `GET /api/rnm/prices`
- **Description** : Effectue des recherches historiques complexes dans la table des prix.
- **Paramètres acceptés** :
  - `product` (optionnel) : Filtrer par slug de produit.
  - `market` (optionnel) : Filtrer par code de marché.
  - `stage` (optionnel) : Relevé de Production, Détail, Grossiste, etc.
  - `dateFrom` & `dateTo` (optionnel) : Dates limites (format YYYY-MM-DD).
  - `limit` (défaut `50`, max `500`) & `offset` (défaut `0`).

### 5. Export de Données en Masse
- **Endpoint** : `GET /api/rnm/export`
- **Description** : Conçu pour extraire des volumes importants de données vers d'autres plateformes.
- **Paramètres acceptés** :
  - `type` (requis) : `products` (tous les produits avec leurs catégories et variétés), `markets` (tous les marchés connus), ou `prices` (toutes les lignes de prix de la table).
  - `limit` (max `5000`) & `offset` pour paginer l'exportation des prix.
  - `dateFrom` / `dateTo` pour exporter uniquement les prix d'un intervalle de temps.

---

## 🛠️ Commandes Locales Utiles

### Installation
```bash
npm install
```

### Initialisation de la Base de Données
Configurez la variable `DATABASE_URL` dans votre fichier `.env` puis exécutez :
```bash
npx prisma db push
```

### Lancement du Dashboard (Développement)
```bash
npm run dev
```

### Vider complètement la Base de Données (Slate Clean)
```bash
npm run db:empty
```

### Lancer une Synchronisation Globale Ponctuelle
```bash
npm run sync:daily
```

---

## 🐳 Déploiement sur Coolify (NAS / VPS)

Ce dépôt est configuré pour être déployé en 1 clic sur **Coolify** grâce au `Dockerfile` et au script `start.sh` qui applique automatiquement les migrations de base de données à chaque démarrage.

### Planification de la Synchronisation Automatique quotidienne à 2h00
1. Déployez l'application sur Coolify.
2. Allez dans l'onglet **Cron Jobs** de votre application sur le tableau de bord de Coolify.
3. Ajoutez une tâche planifiée :
   - **Planification (Cron)** : `0 2 * * *`
   - **Commande** : `npm run sync:daily`
4. Enregistrez. Coolify gérera l'exécution du script à l'intérieur du conteneur chaque nuit à 2h et conservera les rapports de synchronisation dans ses logs.
