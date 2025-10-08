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
  2. la réponse JSON de l’API GET dans le navigateur ou via `curl`.
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

Créez un plugin backend nommé **`call-metrics`** qui expose une API GET

Cette route doit :
- interroger le **Software Catalog**,
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

---

### 4️⃣ Plugin **frontend** – page + onglet

Créez un plugin frontend **`call-metrics`** qui :
- Déclare un **ApiRef** client pour appeler le backend.
- Ajoute une **page** accessible via le menu “Call Metrics”.
- Appelle notre API GET backend
- Affiche :
  - `totalComponents`, `scannedComponents`, `sumCallsIn`, `sumCallsOut` (sous forme de *cards* ou *info boxes*),
  - un **tableau** avec les `topComponents` (`name`, `callsIn`, `callsOut`).

#### UI :
Utilisez les composants **Material UI** / **Backstage Core**.  
Pas besoin de librairie de charts ou de design complexe.

---

### 5️⃣ Contraintes & Simplicité

- Pas de base de données : tout est lu depuis le Catalog.
- Gestion d’erreurs simple : `0` si l’annotation n’existe pas.
- Code clair, structuré, et un README de lancement.


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
- Le frontend peut appeler directement l'API GET ou passer par le **discoveryApi**.
- Le **catalogServiceRef** est la manière la plus simple de lire les entités depuis le backend. il est défini et initialisé dans le fichier plugin.ts dans le bloc **env.registerInit** en tant que dependance et peut être injecté dans le **router** ou les services crées dans le plugin.

---

## 🚀 Exemple de workflow final

1. `yarn dev` démarre l’application.  
2. Ouvrez la page **“Call Metrics”** → vous voyez les agrégats globaux.
3. L’API GET retourne un JSON cohérent.  

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
