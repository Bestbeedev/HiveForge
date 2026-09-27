# STACK — REACT.JS (SPA)

> Copié dans une instance sous `hiveforge/stack.md`.
> Complète les agents core. En cas de contradiction, ce fichier prévaut sur l'agent core, sauf règle de sécurité ou règle absolue.

---

## 1. Scope

Applications **single-page** React construites avec Vite : dashboards, back-offices, outils internes, applications derrière authentification, clients d'une API existante, applications très interactives.

Hors scope (changer de stack) :

- site public dont le trafic dépend du SEO → Next.js ou Astro ;
- besoin de rendu serveur, de Server Actions ou d'accès DB direct → Next.js ;
- backend complet dans le même projet → Laravel (+ Inertia/React) ou Next.js.

Une SPA **consomme** une API ; elle ne contient ni secret, ni logique d'autorisation faisant foi.

---

## 2. Version awareness

Lire les versions de `react`, `react-dom`, `vite`, du router et des librairies de données dans `package.json`. Appliquer les API de ces versions (ex. React 19 : `use`, Actions, `useActionState`, `useOptimistic` ; les modes « framework » vs « data » vs « declarative » de React Router selon la version). En cas de doute : vérifier la documentation, ne pas improviser.

---

## 3. Context detection (Marco)

```text
package.json / lockfile        → react, vite, router, state, data fetching, forms, UI, tests, scripts
vite.config.*                  → plugins, alias, proxy de dev, build
index.html                     → point d'entrée, meta
src/main.tsx                   → providers
src/router* | src/routes/      → routing (React Router / TanStack Router)
src/api/ | src/services/       → client HTTP, contrats d'API
openapi.* | schema.graphql     → contrat d'API (source de vérité des types si présent)
.env.example                   → variables VITE_* (toutes publiques)
tailwind / css config          → styles
vitest.config.*, playwright.config.* → tests
```

---

## 4. Default technical choices (proposés, à confirmer par D-NNN)

| Axe | Défaut | Notes |
|---|---|---|
| Build | Vite | |
| Langage | TypeScript `strict` | |
| Routing | React Router (mode data/declarative selon besoin) ; TanStack Router si typage des routes/search params central | Une seule solution |
| Server state | TanStack Query | Pour toute donnée venant de l'API |
| Client state | `useState` / `useReducer` → Context (valeurs peu changeantes) → Zustand (état partagé fréquent) → Redux Toolkit (seulement si exigence : app très large, équipe habituée, devtools/time-travel requis) | **La solution la plus simple d'abord** |
| Formulaires | Formulaires contrôlés simples / Actions React 19 ; react-hook-form + Zod pour formulaires complexes | |
| Validation | Zod (schémas des payloads et réponses d'API) | |
| Client HTTP | `fetch` natif enveloppé dans `src/lib/api.ts` | Pas d'axios sans raison |
| UI | Tailwind CSS + composants locaux ; shadcn/ui ou Radix si besoin de primitives accessibles complexes (décision) | |
| Auth | Selon le backend : cookie de session `HttpOnly` (préféré) ou fournisseur OIDC via SDK officiel | Pas de token longue durée en `localStorage` |
| Tests | Vitest + Testing Library + MSW (mock réseau) ; Playwright (E2E + axe) | |
| Hébergement | Statique (Vercel, Netlify, Cloudflare Pages, S3+CDN) avec fallback SPA | `UNKNOWN` jusqu'à décision |

---

## 5. Folder architecture

```text
my-react-app/
├── hiveforge/
├── index.html
├── public/
├── src/
│   ├── main.tsx                     # bootstrap, providers (QueryClient, Router, Theme)
│   ├── app/
│   │   ├── router.tsx               # définition des routes, lazy loading par route
│   │   ├── providers.tsx
│   │   ├── layouts/
│   │   │   ├── RootLayout.tsx       # skip link, header, <main>, footer, gestion du focus/titre
│   │   │   └── AuthLayout.tsx
│   │   └── error-boundary.tsx
│   ├── features/                    # un dossier par domaine métier
│   │   └── <feature>/
│   │       ├── api.ts               # appels HTTP + schémas Zod des réponses
│   │       ├── queries.ts           # hooks TanStack Query (useXQuery, useXMutation) + query keys
│   │       ├── components/
│   │       ├── routes/              # composants de route (lazy)
│   │       ├── store.ts             # (optionnel) état client local au domaine
│   │       └── types.ts
│   ├── components/
│   │   └── ui/                      # primitives réutilisables (Button, Input, Dialog…)
│   ├── hooks/                       # hooks transverses
│   ├── lib/
│   │   ├── api.ts                   # client fetch : base URL, credentials, erreurs typées
│   │   ├── env.ts                   # validation des VITE_* (Zod)
│   │   └── utils.ts
│   ├── styles/
│   │   ├── tokens.css
│   │   └── globals.css
│   └── test/
│       ├── setup.ts
│       └── msw/handlers.ts
├── tests/e2e/
├── vite.config.ts
├── vitest.config.ts
├── playwright.config.ts
├── tsconfig.json
├── .env.example
└── package.json
```

Règle d'import : `features/A` n'importe pas l'intérieur de `features/B` ; le partage passe par `components/`, `hooks/`, `lib/`.

---

## 6. Commands

| Action | Commande (à vérifier dans `package.json`) |
|---|---|
| Dev | `<pm> run dev` |
| Build | `<pm> run build` |
| Preview prod | `<pm> run preview` |
| Lint | `<pm> run lint` |
| Typecheck | `<pm> exec tsc --noEmit` (ou `tsc -b`) |
| Unit / composants | `<pm> exec vitest run` |
| E2E | `<pm> exec playwright test` |
| Audit deps | `<pm> audit` |
| Analyse de bundle | `rollup-plugin-visualizer` si décidé, sinon sortie `vite build` |

---

## 7. Performance budgets (Lamine)

| Métrique | Budget |
|---|---|
| JS initial (gzip) — shell + route d'entrée | ≤ 170 KB |
| Chunk par route (gzip) | ≤ 60 KB |
| CSS (gzip) | ≤ 40 KB |
| LCP (p75 mobile) route d'entrée | ≤ 2.5 s |
| INP | ≤ 200 ms |
| CLS | ≤ 0.1 |
| Re-renders | Aucune frappe clavier ne re-rend > 1 sous-arbre de page (vérifié au Profiler sur les écrans critiques) |

---

## 8. Agent overlays

### Marco

Décisions obligatoires en phase 4 :

1. Contrat d'API : existant (OpenAPI/GraphQL/doc) ou `UNKNOWN` — **aucun endpoint inventé**.
2. Router et stratégie de chargement (lazy par route).
3. Server state (TanStack Query) et règle de gestion de l'état client (échelle simple → complexe).
4. Mécanisme d'auth, en accord avec le backend et Paul.
5. Hébergement et fallback SPA.
6. Justification SPA vs rendu serveur si des pages publiques existent.

### Lea

- Microcopy dense : tableaux, filtres, états vides actionnables (« Aucun projet. Créer un projet »), confirmations destructives, toasts, erreurs réseau et de permission.

### Nora

- SPA = gestion explicite du focus et du titre à chaque changement de route ; le spécifier.
- États de données pour chaque vue : loading (squelette), empty, error (avec réessai), stale/refetching, offline si requis.
- Formulaires longs : sauvegarde, confirmation de sortie si modifications non enregistrées (si requis).
- Tableaux : tri, pagination, filtres accessibles.

### Lamine

À surveiller particulièrement :

- **Bundle size** : sortie `vite build`, chunk initial ; imports de librairies entières (icônes, dates, lodash) ;
- **Dependency weight** : chaque dépendance chiffrée (gzip) avant décision ;
- **Re-renders inutiles** : Context trop large (séparer état et dispatch, découper les contexts), sélecteurs Zustand non granulaires, objets/fonctions recréés passés à des enfants mémorisés, état placé trop haut ;
- **Lazy loading** : `React.lazy` + `Suspense` par route et pour les modales/éditeurs lourds ;
- **Tree shaking** : imports nommés, librairies ESM, pas d'effets de bord ;
- TanStack Query : `staleTime` adapté, pas de refetch en rafale, prefetch au survol/intention pour les routes clés ;
- Listes longues : virtualisation seulement si > ~200 éléments rendus et mesurée nécessaire.

Ne pas recommander `useMemo`/`useCallback`/`memo` partout : seulement là où le Profiler montre un coût. Si React Compiler est activé (décision), ne pas ajouter de mémoïsation manuelle.

### Paul

Points de contrôle spécifiques :

- **Tout ce qui est dans le bundle est public** : aucune clé secrète dans `VITE_*`, aucune logique d'autorisation faisant foi côté client (le masquage d'un bouton n'est pas une protection).
- Tokens : préférer cookies `HttpOnly` + `SameSite` (session gérée par le backend) ; sinon access token en mémoire + refresh en cookie `HttpOnly`. Pas de token longue durée en `localStorage`/`sessionStorage`.
- CSRF : si auth par cookie, vérifier le mécanisme du backend (token CSRF, `SameSite`, en-tête custom).
- XSS : `dangerouslySetInnerHTML` interdit sans sanitization (DOMPurify par décision) ; URLs dynamiques dans `href` validées (pas de `javascript:`).
- Redirections post-login : liste blanche de chemins internes.
- CSP via l'hébergeur ; source maps de production non publiques (ou décision explicite).
- Dépendances : audit ; attention aux packages peu maintenus.
- CORS : c'est une configuration **du backend** ; signaler, ne pas contourner par un proxy de production non décidé.

### Emile

- Une SPA n'est pas adaptée au SEO. Emile vérifie :
  - que les pages publiques éventuelles sont pré-rendues ou servies ailleurs (décision) ;
  - `index.html` : `lang`, title/description par défaut, `noindex` si l'app est privée ;
  - titres de document mis à jour par route (accessibilité et partage) ;
  - `robots.txt` adapté (souvent `Disallow: /` pour une app privée).
- Si l'app est entièrement privée : SEO déclaré `N/A` par décision, sauf title/meta de base et `noindex`.

### Julio

- CSS/Tailwind d'abord ; Motion (`motion/react`) pour animations de layout, listes réordonnées, présence (entrées/sorties) si décidé.
- Animations de route : courtes (≤ 250 ms), ne bloquent pas la navigation, désactivées en reduced-motion.
- Feedback d'état (optimistic UI, succès, erreur) prioritaire sur les animations décoratives.

### Noah

- Vitest + Testing Library : composants par comportement (rôles, labels), hooks de données avec MSW, logique pure.
- MSW : handlers dérivés du **contrat d'API réel** ; ne pas inventer de formes de réponse. Si le contrat est `UNKNOWN` → `BLOCKED`.
- Playwright sur `vite build && vite preview` : flows critiques, navigation (deep link + refresh sur une route profonde = fallback SPA), auth, erreurs API (500, 401, 403, réseau coupé).
- Accessibilité : axe + focus après navigation + annonce des changements de route.
- Vérifier qu'un utilisateur sans permission reçoit bien un refus **de l'API** (test d'intégration avec backend de test si disponible).

---

## 9. Stack-specific prompts

```text
@marco Recense le contrat d'API disponible (OpenAPI, doc, backend existant).
Liste chaque endpoint utilisé par les FR MUST. Tout endpoint absent devient UNKNOWN + Q-NNN.
Ne laisse aucun agent inventer un endpoint.
```

```text
@lamine Analyse le bundle (vite build) et les re-renders des écrans <X, Y>
avec le React Profiler. Donne les 5 actions au meilleur ratio gain/effort.
```

---

## 10. Definition of Done — additions

- [ ] `vite build` OK ; chunk initial et chunks de route dans les budgets.
- [ ] `tsc --noEmit` et lint OK.
- [ ] Toutes les routes lazy-loadées sauf la route d'entrée (ou décision).
- [ ] Deep link + refresh fonctionnels sur l'hébergement cible (fallback SPA).
- [ ] Focus et titre gérés à chaque navigation.
- [ ] Aucun secret dans `VITE_*` ; aucune autorisation faisant foi côté client.
- [ ] Erreurs API gérées (401 → reconnexion, 403 → message, 5xx → réessai) selon STRUCTURE.md.
