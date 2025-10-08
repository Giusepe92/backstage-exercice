 # 🎯 Exercice Backstage NEXT (2025) – “Call Metrics”

## 🧩 Objectif

Créer un mini-plugin **modulaire (backend + frontend)** pour **Backstage (nouvelle architecture)** qui :
1. Lit les entités `Component` du **Software Catalog** via l’API interne du backend.
2. Expose une **API HTTP GET** via le **backend plugin system** (`createBackendPlugin` + `router`).
3. Crée une **page frontend** et un **onglet d’entité** via le **frontend system** (`createFrontendPlugin` + `createExtension`).
4. Affiche un résumé du nombre d’appels entrants/sortants (annotations Backstage).

---

## 🎓 But pédagogique

Cet exercice vise à évaluer :
- votre aisance avec **Node.js** et **React**,
- votre compréhension de l’**architecture Backstage NEXT** (système modulaire),
- votre capacité à manipuler **plugins frontend et backend** modernes (`createBackendPlugin`, `createFrontendPlugin`),
- et votre autonomie dans la mise en place d’un environnement local.

---

## ⚙️ Environnement de départ

Créez une application Backstage à jour :

```bash
npx @backstage/create-app@latest
cd my-backstage-app
yarn dev
```

Vérifiez que votre projet utilise bien le **nouveau système modulaire** :
- backend : `packages/backend/src/index.ts` avec `createBackend`
- frontend : `packages/app/src/App.tsx` avec `createApp`

---

## 🧠 Partie 1 — Backend Plugin (modulaire)

### But
Créer un **plugin backend “call-metrics”** exposant une route :

```
GET /api/call-metrics/summary
```

### Étapes

1. Générez le plugin :
   ```bash
   yarn new --select plugin-backend --name call-metrics
   ```

2. Implémentez le plugin dans `plugins/call-metrics/src/plugin.ts` :

   ```ts
   import { createBackendPlugin } from '@backstage/backend-plugin-api';
   import { coreServices } from '@backstage/backend-plugin-api';
   import { CatalogClient } from '@backstage/catalog-client';
   import { Router } from 'express';

   export const callMetricsPlugin = createBackendPlugin({
     pluginId: 'call-metrics',
     register(env) {
       env.registerInit({
         deps: { logger: coreServices.logger, discovery: coreServices.discovery },
         async init({ logger, discovery }) {
           const catalogClient = new CatalogClient({ discoveryApi: discovery });
           const router = Router();

           router.get('/summary', async (_req, res) => {
             const entities = await catalogClient.getEntities({ filter: { kind: 'Component' } });
             const scanned = entities.items.filter(e => e.metadata.annotations?.['example.com/calls.in']);
             const summary = {
               totalComponents: entities.items.length,
               scannedComponents: scanned.length,
               sumCallsIn: scanned.reduce((s, e) => s + Number(e.metadata.annotations?.['example.com/calls.in'] ?? 0), 0),
               sumCallsOut: scanned.reduce((s, e) => s + Number(e.metadata.annotations?.['example.com/calls.out'] ?? 0), 0),
               topComponents: scanned.map(e => ({
                 name: e.metadata.name,
                 callsIn: Number(e.metadata.annotations?.['example.com/calls.in'] ?? 0),
                 callsOut: Number(e.metadata.annotations?.['example.com/calls.out'] ?? 0),
               })),
             };
             res.json(summary);
           });

           env.httpRouter.use(router);
           logger.info('✅ Call Metrics backend plugin initialized');
         },
       });
     },
   });
   ```

3. Enregistrez le plugin dans `packages/backend/src/index.ts` :

   ```ts
   import { createBackend } from '@backstage/backend-defaults';
   import { callMetricsPlugin } from '@internal/plugin-call-metrics';

   const backend = createBackend();
   backend.add(callMetricsPlugin());
   backend.start();
   ```

---

## 💻 Partie 2 — Frontend Plugin (modulaire)

### But
Créer une **page “Call Metrics”** et un **onglet d’entité** pour visualiser les données.

### Étapes

1. Générez le plugin :
   ```bash
   yarn new --select plugin-frontend --name call-metrics
   ```

2. Dans `plugins/call-metrics/src/plugin.ts` :

   ```ts
   import { createFrontendPlugin, createExtension } from '@backstage/frontend-plugin-api';

   export const CallMetricsPage = createExtension({
     name: 'CallMetricsPage',
     component: {
       lazy: () => import('./components/CallMetricsPage'),
     },
   });

   export const callMetricsPlugin = createFrontendPlugin({
     id: 'call-metrics',
     extensions: [CallMetricsPage],
   });
   ```

3. Créez la page `plugins/call-metrics/src/components/CallMetricsPage.tsx` :

   ```tsx
   import React, { useEffect, useState } from 'react';
   import { useApi, discoveryApiRef } from '@backstage/core-plugin-api';
   import { Page, Header, Content, Table } from '@backstage/core-components';

   export const CallMetricsPage = () => {
     const discoveryApi = useApi(discoveryApiRef);
     const [data, setData] = useState<any>(null);

     useEffect(() => {
       (async () => {
         const baseUrl = await discoveryApi.getBaseUrl('call-metrics');
         const res = await fetch(`${baseUrl}/summary`);
         setData(await res.json());
       })();
     }, [discoveryApi]);

     if (!data) return <div>Loading...</div>;

     return (
       <Page themeId="tool">
         <Header title="Call Metrics Summary" />
         <Content>
           <p>Total Components: {data.totalComponents}</p>
           <p>Components with annotations: {data.scannedComponents}</p>
           <p>Sum Calls In: {data.sumCallsIn}</p>
           <p>Sum Calls Out: {data.sumCallsOut}</p>

           {data.topComponents && (
             <Table
               options={{ paging: false }}
               data={data.topComponents}
               columns={[
                 { title: 'Name', field: 'name' },
                 { title: 'Calls In', field: 'callsIn' },
                 { title: 'Calls Out', field: 'callsOut' },
               ]}
             />
           )}
         </Content>
       </Page>
     );
   };
   ```

4. Enregistrez la page dans `packages/app/src/App.tsx` :

   ```ts
   import { createApp } from '@backstage/frontend-defaults';
   import { callMetricsPlugin } from '@internal/plugin-call-metrics';

   const app = createApp({
     plugins: [callMetricsPlugin()],
   });

   export default app.createRoot();
   ```

---

## 🧩 Partie 3 — Ajout d’un onglet d’entité (optionnel)

```tsx
import { createEntityContentExtension } from '@backstage/plugin-catalog-react';

export const CallMetricsTab = createEntityContentExtension({
  name: 'CallMetricsTab',
  component: {
    lazy: () => import('./components/EntityCallMetricsTab'),
  },
});
```

---

## ✅ Critères d’évaluation

| Critère | Pondération |
|----------|--------------|
| Mise en place Backstage NEXT + README clair | 20% |
| Plugin backend (API, CatalogClient, agrégations) | 30% |
| Plugin frontend (page + onglet, appels API) | 30% |
| Qualité du code, UX minimale, lisibilité | 20% |

---

## 📎 Exemple d’annotations

```yaml
metadata:
  annotations:
    example.com/calls.in: "10"
    example.com/calls.out: "7"
```

---

👨‍💻 **Bonne chance !**  
Cet exercice est une bonne occasion de découvrir le **nouveau système modulaire de Backstage (2025)**.
