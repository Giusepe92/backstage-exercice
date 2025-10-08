# 🎯 Exercice Backstage – “Call Metrics”

## 🧩 Objectif

Créer un mini-plugin **backend** + **frontend** pour **Backstage** qui :

1. lit les entités `Component` du **Software Catalog**,
2. récupère deux **annotations numériques** par composant (appels entrants / sortants),
3. expose une **API GET** qui agrège ces chiffres,
4. affiche une **page** dans le frontend Backstage (et un **onglet** sur la page d’un composant) pour visualiser ces métriques.

---

## 🎓 But pédagogique

Cet exercice vise à évaluer :
- votre aisance avec **Node.js** et **React**,  
- votre compréhension de l’**architecture Backstage** (front, back, API, discovery, catalog),  
- votre autonomie dans la **mise en place d’un environnement local**,  
- et votre capacité à livrer un code clair et structuré.

---

## 📦 Livrables attendus

- Un **dépôt Git** (ou archive) contenant le code complet.
- Un **README** clair expliquant :
  - comment lancer le projet,
  - et où accéder à votre plugin dans l’interface.
- **3 captures d’écran** :
  1. la page globale “Call Metrics”,
  2. l’onglet “Call Metrics” d’un composant,
  3. la réponse JSON de l’API GET dans le navigateur ou via `curl`.
- Un **retour d’expérience** écrit (quelques paragraphes) :  
  ce que vous avez compris de l’architecture Backstage, vos impressions, et les difficultés rencontrées.

---

## 🛠️ Cahier des charges

### 1️⃣ Pré-requis & démarrage

- Créez une application Backstage locale :
  ```bash
  npx @backstage/create-app@latest
  cd my-backstage-app
  yarn dev
  ```
- Utilisez le **catalogue d’exemple** fourni par le squelette Backstage (`example.yaml` et données bootstrap).

---

### 2️⃣ Enrichir les entités avec des annotations

Dans `catalog-info.yaml` (ou `example.yaml`) de quelques composants, ajoutez deux annotations numériques :

```yaml
metadata:
  annotations:
    example.com/calls.in: "12"
    example.com/calls.out: "7"
```

➡️ Mettez des valeurs différentes pour au moins **3 composants**.

---

### 3️⃣ Plugin **backend** – API GET

Créez un plugin backend nommé **`call-metrics`** avec une route :

```
GET /api/call-metrics/summary
```

Cette route doit :
- interroger le **Software Catalog** (via `CatalogClient`),
- filtrer les entités de type `Component`,
- lire les annotations `example.com/calls.in` et `example.com/calls.out` (0 si absentes),
- retourner un JSON de synthèse comme ci-dessous :

```json
{
  "totalComponents": 5,
  "scannedComponents": 3,
  "sumCallsIn": 41,
  "sumCallsOut": 29,
  "topComponents": [
    { "name": "payment-service", "callsIn": 20, "callsOut": 10 },
    { "name": "web-frontend", "callsIn": 12, "callsOut": 9 }
  ]
}
```

*(Optionnel)* : ajoutez une route `GET /api/call-metrics/components/:name`  
→ qui renvoie les deux chiffres d’un seul composant.

#### Pistes de fichiers côté backend :
- `packages/backend/src/plugins/call-metrics.ts`
- `packages/backend/src/plugins/call-metrics/router.ts`

💡 Vous pouvez utiliser :
```ts
import { CatalogClient } from '@backstage/catalog-client';
```

et le système **`discovery`** pour résoudre les URLs internes.

---

### 4️⃣ Plugin **frontend** – page + onglet

Créez un plugin frontend **`call-metrics`** qui :
- Déclare un **ApiRef** client pour appeler le backend.
- Ajoute une **page** accessible via le menu “Call Metrics”.
- Appelle `GET /api/call-metrics/summary`.
- Affiche :
  - `totalComponents`, `scannedComponents`, `sumCallsIn`, `sumCallsOut` (sous forme de *cards* ou *info boxes*),
  - un **tableau** avec les `topComponents` (`name`, `callsIn`, `callsOut`).

#### Puis ajoutez un **onglet** “Call Metrics” sur la page d’un composant :
- Utilisez le hook `useEntity()` pour récupérer l’entité courante.
- Lisez directement les annotations ou appelez l’API par composant.
- Affichez les deux chiffres (callsIn / callsOut).

#### Fichiers possibles :
- `plugins/call-metrics/src/plugin.ts`
- `plugins/call-metrics/src/api.ts`
- `plugins/call-metrics/src/routes.tsx`
- Intégration dans `packages/app/src/App.tsx`
- Ajout d’un onglet dans `packages/app/src/components/catalog/EntityPage.tsx`

#### UI :
Utilisez les composants **Material UI** / **Backstage Core**.  
Pas besoin de librairie de charts ou de design complexe.

---

### 5️⃣ Contraintes & Simplicité

- Une seule API GET obligatoire : `/api/call-metrics/summary`.
- Pas de base de données : tout est lu depuis le Catalog.
- Gestion d’erreurs simple : `0` si l’annotation n’existe pas.
- Code clair, structuré, et un README de lancement.

---

## ✅ Critères d’évaluation

| Critère | Pondération |
|----------|--------------|
| Mise en place Backstage + README clair | 20% |
| Plugin backend (route GET, usage CatalogClient, agrégations) | 30% |
| Plugin frontend (page + onglet Entity, appels API) | 30% |
| Qualité du code, UX minimale, lisibilité | 20% |

---

## ⭐ Bonus (facultatif)

- Filtrer par **owner** ou **system** (petit menu déroulant).
- Ajouter un **ratio** (callsIn vs callsOut) avec tri.
- Bouton “Copy JSON” pour copier la réponse API.

---

## 🧠 Restitution & débrief

Lors de la restitution, merci d’expliquer :
- Comment vous avez compris l’**architecture Backstage** (app, plugins, APIs, discovery, catalog),
- Vos **choix techniques**,
- Ce que vous avez trouvé **simple** ou **difficile**,
- Vos **impressions globales** sur le framework Backstage.

---

## 💡 Indices utiles

- Le backend Backstage utilise **Express**, vous pouvez créer un simple `Router()` exporté.
- Le frontend peut appeler directement `/api/call-metrics/...` ou passer par le **discoveryApi**.
- Le **CatalogClient** est la manière la plus simple de lire les entités depuis le backend.

---

## 🚀 Exemple de workflow final

1. `yarn dev` démarre l’application.  
2. Ouvrez la page **“Call Metrics”** → vous voyez les agrégats globaux.  
3. Cliquez sur un **Component** → un onglet “Call Metrics” affiche ses chiffres.  
4. L’API `GET /api/call-metrics/summary` retourne un JSON cohérent.  

---

## 📎 Exemple d’annotations dans un `catalog-info.yaml`

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: payment-service
  annotations:
    example.com/calls.in: "20"
    example.com/calls.out: "10"
spec:
  type: service
  owner: team-a
  lifecycle: production
```

---

## 🗂️ Exemple de rendu attendu (simplifié)

```
--------------------------------------------
| Total Components : 5                     |
| Components avec annotations : 3          |
| Appels entrants (total) : 41             |
| Appels sortants (total) : 29             |
--------------------------------------------
| Component Name | Calls In | Calls Out    |
|----------------|-----------|-------------|
| payment-service| 20        | 10          |
| web-frontend   | 12        | 9           |
--------------------------------------------
```

---

👨‍💻 **Bonne chance !**  
Prenez du plaisir à explorer Backstage et à découvrir sa logique interne.
