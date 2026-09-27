# STACK — NEXT.JS

> Copié dans une instance sous `hiveforge/stack.md`.
> Complète les agents core. En cas de contradiction, ce fichier prévaut sur l'agent core, sauf règle de sécurité ou règle absolue.

---

## 1. Scope

Applications web full-stack et sites à fort enjeu SEO construits avec **Next.js (App Router)** : SaaS, dashboards, sites marketing dynamiques, e-commerce headless, applications avec authentification et base de données.

Socle de référence (à confirmer par décisions) : **Next.js App Router · React Server Components · TypeScript · Tailwind CSS · shadcn/ui · Drizzle ORM · Vercel**.

---

## 2. Version awareness (obligatoire)

Next.js évolue vite et certaines API changent entre versions majeures (ex. mise en cache, APIs asynchrones `params`/`cookies()`/`headers()`, fichier `middleware.ts` renommé `proxy.ts` dans les versions récentes, directive `"use cache"`).

Règles :

1. Lire la version exacte de `next` et `react` dans `package.json` / lockfile.
2. Consulter la documentation **de cette version** (documentation officielle ou documentation embarquée dans `node_modules/next/` si présente, ou fichier `AGENTS.md` généré par `create-next-app`).
3. Ne jamais appliquer un pattern d'une autre version majeure. En cas de doute : `UNKNOWN` + vérification, pas d'improvisation.
4. Consigner la version et les conventions de version retenues dans `ARCHITECTURE.md §2`.

---

## 3. Context detection (Marco)

```text
package.json / lockfile       → versions next, react, drizzle-orm, drizzle-kit, zod, auth, tailwindcss, scripts, package manager
next.config.*                 → output, images, experimental flags, headers, redirects
app/ ou src/app/              → App Router ; pages/ → Pages Router (legacy : ne pas mélanger sans décision)
middleware.ts | proxy.ts      → logique de requête
components.json               → shadcn/ui configuré (style, alias, chemins)
drizzle.config.*              → dialecte, schéma, dossier de migrations
db/ | lib/db/ | src/db/       → schéma Drizzle
.env.example                  → variables (noms uniquement)
vercel.json                   → configuration Vercel
playwright.config.*, vitest.config.* → tests
.github/workflows/            → CI
```

---

## 4. Default technical choices (proposés, à confirmer par D-NNN)

| Axe | Défaut | Notes |
|---|---|---|
| Routing | App Router | Pages Router uniquement pour un projet existant qui l'utilise |
| Langage | TypeScript `strict` | |
| Rendering | Server Components par défaut ; statique quand possible ; dynamique quand requis par la donnée ; streaming via `Suspense` | Client Components aux feuilles de l'arbre |
| Mutations | Server Actions pour les formulaires de l'app ; Route Handlers pour les API publiques, webhooks, clients externes | |
| Validation | Zod (schémas partagés client/serveur) | |
| UI | shadcn/ui (composants copiés dans le repo) + Tailwind CSS | Ajouter uniquement les composants utilisés |
| Formulaires | Server Actions + `useActionState` ; react-hook-form uniquement si formulaires complexes (décision) | |
| Base de données | PostgreSQL (fournisseur `UNKNOWN` jusqu'à décision : Neon, Supabase, Vercel Marketplace, autre) | |
| ORM | Drizzle ORM + drizzle-kit (migrations versionnées) | |
| Auth | `UNKNOWN` → décision (Better Auth, Auth.js, Clerk, fournisseur managé…) selon exigences | Jamais d'auth maison pour le hash/session |
| État client | React state / Context ; Zustand seulement si état partagé complexe justifié | Pas de store global pour des données serveur |
| Fetch côté client | À éviter ; si nécessaire (polling, infinite scroll) : SWR ou TanStack Query par décision | |
| Images / fonts | `next/image`, `next/font` | |
| Tests | Vitest (unit/intégration), Playwright (E2E + axe) | |
| Hébergement | Vercel | Environnements Preview / Production |

---

## 5. Folder architecture

```text
my-next-project/
├── hiveforge/
├── app/                              # (ou src/app/ — suivre l'existant)
│   ├── layout.tsx                    # html lang, fonts, providers minimaux, metadata par défaut
│   ├── page.tsx
│   ├── globals.css
│   ├── not-found.tsx
│   ├── error.tsx                     # "use client" — boundary racine
│   ├── global-error.tsx
│   ├── sitemap.ts
│   ├── robots.ts
│   ├── (marketing)/                  # groupe de routes publiques
│   │   ├── layout.tsx
│   │   └── pricing/page.tsx
│   ├── (auth)/
│   │   ├── sign-in/page.tsx
│   │   └── sign-up/page.tsx
│   ├── (app)/                        # zone authentifiée
│   │   ├── layout.tsx                # vérification de session (non suffisante seule → voir Paul)
│   │   ├── dashboard/
│   │   │   ├── page.tsx
│   │   │   ├── loading.tsx
│   │   │   └── error.tsx
│   │   └── projects/[id]/page.tsx
│   └── api/
│       └── webhooks/<provider>/route.ts
├── components/
│   ├── ui/                           # shadcn/ui (générés, modifiables)
│   └── <feature>/                    # composants métier par fonctionnalité
├── features/                         # (optionnel, si > ~5 domaines) logique par domaine
│   └── <domain>/
│       ├── actions.ts                # "use server" — Server Actions du domaine
│       ├── queries.ts                # lectures serveur (import "server-only")
│       ├── schemas.ts                # Zod
│       └── components/
├── db/
│   ├── index.ts                      # client Drizzle (server-only)
│   ├── schema/*.ts                   # tables, relations
│   └── migrations/                   # générées par drizzle-kit, versionnées
├── lib/
│   ├── auth.ts                       # configuration auth + helpers (getSession, requireUser)
│   ├── env.ts                        # validation des variables d'environnement (Zod)
│   ├── utils.ts                      # cn() de shadcn + utilitaires
│   └── site.ts                       # config du site (URL depuis env)
├── public/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── proxy.ts | middleware.ts          # selon la version ; seulement si nécessaire
├── components.json
├── drizzle.config.ts
├── next.config.ts
├── playwright.config.ts
├── vitest.config.ts
├── tsconfig.json
├── .env.example
└── package.json
```

Règle : `features/` n'est introduit que si le nombre de domaines le justifie (décision). Sinon `app/` + `components/<feature>/` + `lib/` suffisent.

---

## 6. Commands

Lire les scripts réels dans `package.json`. Par défaut :

| Action | Commande |
|---|---|
| Dev | `<pm> run dev` |
| Build | `<pm> run build` |
| Start (prod local) | `<pm> run start` |
| Lint | `<pm> run lint` |
| Typecheck | `<pm> exec tsc --noEmit` |
| Unit / intégration | `<pm> exec vitest run` |
| E2E | `<pm> exec playwright test` |
| Migrations : générer | `<pm> exec drizzle-kit generate` |
| Migrations : appliquer | `<pm> exec drizzle-kit migrate` (jamais `push` sur une base partagée/production) |
| Ajouter un composant UI | `<pm> dlx shadcn@latest add <component>` (ou `npx`) |
| Audit deps | `<pm> audit` |
| Analyse de bundle | `@next/bundle-analyzer` si décidé, ou sortie du build |

---

## 7. Performance budgets (Lamine)

| Métrique | Budget |
|---|---|
| LCP (p75 mobile) pages publiques | ≤ 2.5 s |
| INP | ≤ 200 ms |
| CLS | ≤ 0.1 |
| JS client first load (gzip), pages marketing | ≤ 130 KB |
| JS client first load (gzip), pages app | ≤ 200 KB |
| TTFB pages dynamiques | ≤ 600 ms (p75) |
| Requêtes DB par rendu de page | pas de N+1 ; ≤ ~5 requêtes indépendantes, en parallèle |

---

## 8. Agent overlays

### Marco

Décisions obligatoires en phase 4 :

1. Version Next.js/React et conventions associées (§2).
2. Stratégie de rendu et de cache **par type de route** (ARCHITECTURE.md §3).
3. Fournisseur de base de données + Drizzle (ou absence de DB).
4. Solution d'auth (ou absence d'auth).
5. Mutations : Server Actions vs Route Handlers.
6. Emplacement de la logique métier (`features/` ou `lib/`).
7. Environnements Vercel et variables (noms uniquement).

Ne pas activer : i18n, multi-tenant, edge runtime, microservices, file d'attente — sans exigence explicite.

### Lea

- Microcopy des états serveur : `loading.tsx` (squelettes), `error.tsx`, `not-found.tsx`, messages de retour des Server Actions.
- Emails transactionnels si l'app en envoie (contenu `TO_VALIDATE`).

### Nora

- Spécifier les frontières de chargement : quelles zones streament (`Suspense`) et quel squelette les remplace.
- Utiliser les primitives shadcn/ui comme vocabulaire de composants (Dialog, Sheet, DropdownMenu, Form, Toast/Sonner…), sans imposer leur utilisation si une primitive HTML suffit.
- Définir les états optimistes (ce qui s'affiche avant confirmation serveur).

### Lamine

- Traquer les `"use client"` inutiles ou placés trop haut (un `"use client"` dans un layout rend tout le sous-arbre client).
- Données : requêtes parallèles (`Promise.all`), pas de waterfall entre composants serveur ; `React.cache` pour dédupliquer les lectures par requête.
- Cache : vérifier que la stratégie de la version installée est explicitement configurée (statique/dynamique, revalidation, tags) et documentée.
- Imports lourds côté client : `next/dynamic` pour les composants non critiques.
- Drizzle : `select` des colonnes nécessaires, index sur colonnes filtrées/jointes, pagination, `with` (relations) au lieu de boucles.
- Scripts tiers : `next/script` avec stratégie adaptée.
- Outils : sortie de `next build` (taille par route), Lighthouse, Vercel Speed Insights si activé par l'utilisateur.

### Paul

Points de contrôle spécifiques :

- **Server Actions = endpoints publics** : chaque action vérifie authentification **et** autorisation (propriété de la ressource) **et** valide ses entrées (Zod). Ne jamais faire confiance à un ID reçu du client.
- **Le proxy/middleware n'est pas une barrière d'autorisation suffisante** : l'autorisation est revérifiée au plus près de la donnée (query / action / route handler).
- Layouts : une vérification dans un layout ne protège pas les Server Actions ni les Route Handlers.
- Frontière serveur/client : `import "server-only"` dans `db/`, `lib/auth.ts`, fichiers de queries ; aucune donnée sensible passée en props à un Client Component (les props sont sérialisées vers le navigateur).
- Variables d'environnement : seules les variables `NEXT_PUBLIC_*` sont exposées — vérifier qu'aucune n'est sensible ; validation dans `lib/env.ts`.
- Drizzle : requêtes paramétrées par défaut ; interdire `sql.raw()` avec entrée utilisateur.
- Route Handlers de webhooks : vérification de signature, idempotence.
- Headers de sécurité et CSP dans `next.config` (`headers()`) ou proxy/middleware (CSP à nonce si scripts inline).
- `dangerouslySetInnerHTML` : interdit sans sanitization décidée.
- Redirections : pas de redirection ouverte basée sur un paramètre non validé.
- Mass assignment : ne jamais passer `formData`/body brut à `db.insert().values()` ; construire l'objet à partir du schéma Zod.
- Environnements Preview Vercel : protection d'accès si données réelles.

### Emile

- API Metadata : `metadata` / `generateMetadata`, `metadataBase` depuis env, `alternates.canonical`, `openGraph`, `twitter`.
- `app/sitemap.ts` (routes dynamiques depuis la DB/contenu), `app/robots.ts`.
- `opengraph-image.tsx` pour OG dynamiques si décidé.
- JSON-LD via `<script type="application/ld+json">` rendu côté serveur.
- Pages de l'app authentifiée : `robots: { index: false }`.
- Vérifier que le contenu indexable est dans les Server Components (HTML initial).

### Julio

- Priorité : CSS / Tailwind (`transition`, `animate-*`, `motion-safe:` / `motion-reduce:`) → Motion (`motion/react`) si animations d'état/layout complexes et décision → View Transitions (selon support de la version).
- Tout composant animé en JS est un Client Component : l'isoler en feuille, ne pas « clientiser » une section entière.
- Les composants shadcn/ui ont déjà leurs animations (data-state) : les réutiliser plutôt que les doubler.

### Noah

- Vitest : schémas Zod, utilitaires, logique métier pure ; Server Actions testées comme fonctions (avec DB de test).
- Base de test : base dédiée ou branche de base (ex. branche Neon) ; migrations appliquées avant les tests ; jamais la base de production.
- Playwright sur `next build && next start` : flows critiques, auth, formulaires, autorisations (utilisateur A ne peut pas lire/modifier les ressources de B), pages 404/erreur.
- Axe via `@axe-core/playwright` sur les pages critiques.
- Tester les états `loading`/`error` (réseau lent, erreur serveur simulée).

---

## 9. Stack-specific prompts

```text
@marco Fixe la stratégie de rendu et de cache pour chaque type de route
(marketing, contenu, app authentifiée, API) selon la version de Next.js installée.
Documente dans ARCHITECTURE.md §3 et DECISIONS.md. Consulte Lamine et Paul.
```

```text
@louis Implémente le schéma Drizzle défini dans ARCHITECTURE.md §4, génère la migration,
et fournis les queries/actions du domaine <X> avec validation Zod et contrôle d'autorisation.
N'exécute pas de migration sur une base autre que locale/test.
```

```text
@paul Audite toutes les Server Actions et Route Handlers : authentification,
autorisation (IDOR), validation, données renvoyées au client. Une ligne par point d'entrée.
```

---

## 10. Definition of Done — additions

- [ ] `next build` sans erreur ni warning bloquant ; taille des routes dans les budgets.
- [ ] `tsc --noEmit` et lint OK.
- [ ] Migrations Drizzle générées, versionnées, appliquées en environnement de test.
- [ ] Chaque Server Action / Route Handler : auth + autorisation + validation vérifiées par Paul.
- [ ] Aucun `"use client"` injustifié (revue Lamine).
- [ ] Variables d'environnement documentées dans `.env.example` et validées dans `lib/env.ts`.
- [ ] Déploiement Preview Vercel vérifié avant Production.
