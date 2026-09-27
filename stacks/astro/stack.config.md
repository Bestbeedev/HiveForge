# STACK — ASTRO

> Copié dans une instance sous `hiveforge/stack.md`.
> Complète les agents core. En cas de contradiction, ce fichier prévaut sur l'agent core, sauf règle de sécurité ou règle absolue.

---

## 1. Scope

Sites **orientés contenu** : sites vitrines, blogs, documentation, landing pages, portfolios, sites marketing, sites éditoriaux multilingues.

Principe fondamental :

> **HTML d'abord, JavaScript seulement lorsque nécessaire.**

Astro génère du HTML sans JS par défaut. Chaque île interactive (composant hydraté) est une exception qui doit être **justifiée**.

Hors scope : application riche derrière authentification avec beaucoup d'état client → Next.js ou React.

---

## 2. Version awareness

Lire la version d'`astro` et des intégrations (`@astrojs/*`) dans `package.json`. Les API de Content Collections, d'actions, de rendu serveur et d'images évoluent entre versions majeures (ex. Content Layer API et `src/content.config.ts` dans les versions récentes). Appliquer la documentation de la version installée ; en cas de doute, vérifier.

---

## 3. Context detection (Marco)

```text
package.json / lockfile        → astro, @astrojs/* (mdx, sitemap, react/vue/svelte/preact, adapters), scripts
astro.config.*                 → site, output (static/server), adapter, integrations, i18n, redirects
src/pages/                     → routes
src/content.config.ts          → collections (ou src/content/config.ts selon version)
src/content/ | autres loaders  → contenu Markdown/MDX/JSON
src/layouts/, src/components/  → composants .astro et îles framework
public/                        → assets non traités
.env.example                   → variables (PUBLIC_* exposées)
```

---

## 4. Default technical choices (proposés, à confirmer par D-NNN)

| Axe | Défaut | Notes |
|---|---|---|
| Output | Statique (SSG) | Rendu serveur à la demande uniquement pour les routes qui l'exigent (formulaire, contenu personnalisé), avec adapter décidé |
| Langage | TypeScript (config `strict` d'Astro) | |
| Contenu | Content Collections + schémas Zod ; Markdown, MDX si composants dans le contenu | CMS headless seulement si exigence (loader) |
| Composants | `.astro` par défaut | Framework d'îles (Preact/React/Svelte/Vue) **un seul**, seulement si une île le nécessite |
| Interactivité légère | `<script>` Astro (TS bundlé) ou Web Components | Avant de recourir à une île framework |
| Styles | CSS scopé dans `.astro` + tokens globaux ; Tailwind si décidé | |
| Images | `astro:assets` (`<Image />`, `<Picture />`) | |
| Fonts | Auto-hébergées (ou API de fonts d'Astro si disponible dans la version) | |
| SEO | `@astrojs/sitemap`, composant `<SEO>`/`<Head>` local | |
| Formulaires | Actions Astro (mode serveur) ou service de formulaire (mode statique) — décision | |
| Transitions | View Transitions (`<ClientRouter />`) seulement si décidé | Ajoute du JS de navigation client |
| Tests | Vitest (utilitaires, schémas, Container API pour composants si utile), Playwright (E2E + axe) | |
| Hébergement | Statique (Netlify, Vercel, Cloudflare Pages, GitHub Pages) | `UNKNOWN` jusqu'à décision |

---

## 5. Folder architecture

```text
my-astro-site/
├── hiveforge/
├── astro.config.mjs                 # site (URL de prod via env), integrations, output
├── src/
│   ├── pages/
│   │   ├── index.astro
│   │   ├── about.astro
│   │   ├── blog/
│   │   │   ├── index.astro
│   │   │   └── [...slug].astro      # getStaticPaths depuis la collection
│   │   ├── rss.xml.ts               # si blog
│   │   ├── robots.txt.ts            # ou public/robots.txt
│   │   └── 404.astro
│   ├── layouts/
│   │   ├── BaseLayout.astro         # <html lang>, <head> (SEO), skip link, header, main, footer
│   │   └── PostLayout.astro
│   ├── components/
│   │   ├── ui/                      # composants .astro réutilisables (Button, Card…)
│   │   ├── sections/                # Hero, Features, CTA…
│   │   ├── seo/SEO.astro            # title, description, canonical, OG, JSON-LD
│   │   └── islands/                 # composants hydratés (framework) — chacun justifié
│   ├── content/                     # fichiers de contenu (md/mdx/json)
│   │   └── blog/*.mdx
│   ├── content.config.ts            # collections + schémas Zod (selon version)
│   ├── scripts/                     # <script> partagés (TS), Web Components
│   ├── styles/
│   │   ├── tokens.css
│   │   ├── global.css
│   │   └── motion.css               # Julio
│   ├── assets/                      # images traitées par astro:assets
│   └── lib/                         # utilitaires (dates, slug, site config)
├── public/                          # favicon, fichiers statiques non transformés
├── tests/
│   ├── unit/
│   └── e2e/
├── tsconfig.json
├── .env.example
└── package.json
```

---

## 6. Commands

| Action | Commande (à vérifier dans `package.json`) |
|---|---|
| Dev | `<pm> run dev` |
| Build | `<pm> run build` |
| Preview | `<pm> run preview` |
| Check (types, diagnostics .astro) | `<pm> exec astro check` |
| Unit | `<pm> exec vitest run` |
| E2E | `<pm> exec playwright test` |
| Ajouter une intégration | `<pm> exec astro add <integration>` (après décision) |
| Audit deps | `<pm> audit` |

---

## 7. Performance budgets (Lamine)

| Métrique | Budget |
|---|---|
| JS par page de contenu | **0 KB** par défaut ; toute page avec JS le justifie |
| JS total (gzip) page avec îles | ≤ 50 KB |
| CSS (gzip) | ≤ 30 KB |
| LCP (p75 mobile) | ≤ 2.0 s |
| CLS | ≤ 0.05 |
| INP | ≤ 200 ms |
| Lighthouse mobile | Performance ≥ 95, Accessibility ≥ 95, SEO = 100 |

---

## 8. Agent overlays

### Marco

Décisions obligatoires en phase 4 :

1. Output : statique pur, ou statique + routes serveur (adapter).
2. Modèle de contenu : collections, schémas, source (fichiers locaux ou CMS).
3. Framework d'îles : aucun (défaut) ou **un seul** justifié.
4. Formulaires : Actions (serveur) ou service externe (statique).
5. View Transitions : oui/non (coût JS + gestion des scripts au changement de page).
6. i18n : routing i18n natif si multilingue.

Tenir un **registre des îles** dans ARCHITECTURE.md :

| Île | Composant | Directive `client:*` | Justification | Poids JS |
|---|---|---|---|---|

### Lea

- Contenu éditorial : frontmatter défini par les schémas de collection (titre, description, date, auteur, image, alt, tags). Chaque champ obligatoire est `CONFIRMED` ou `UNKNOWN`.
- Articles : pas de date ni d'auteur inventés.

### Nora

- Pour chaque élément interactif, préciser le niveau d'interactivité requis :

```text
Aucun (lien, HTML natif : <details>, <dialog>, formulaire HTML)
  → Script léger (menu mobile, toggle)
  → Île framework (état complexe, UI riche)
```

  pour permettre à Louis de choisir la solution la plus légère.
- Navigation : fonctionne intégralement sans JS.

### Lamine

- Contrôler chaque île : nécessité, directive d'hydratation (`client:visible`/`client:idle` avant `client:load`), poids.
- Détecter les frameworks d'îles multiples (interdit sans décision).
- `astro:assets` pour toutes les images locales ; images distantes autorisées dans la config uniquement si nécessaires.
- Vérifier le poids des polices et du CSS global.
- View Transitions : mesurer le JS ajouté.
- Pages serveur : cache HTTP adapté.

### Paul

- Site statique : surface réduite ; vérifier formulaires (service externe ou Actions), scripts tiers, CSP (hashes pour les scripts inline d'Astro si CSP stricte ; utiliser le support CSP natif si disponible dans la version), headers via l'hébergeur.
- Mode serveur : Actions et endpoints (`src/pages/api/*.ts`) validés (Zod intégré aux Actions), protégés (auth si requise), rate limiting ; vérification d'origine / CSRF selon la version.
- `set:html` : interdit avec du contenu non maîtrisé ; contenu Markdown d'un CMS externe à considérer comme non fiable (sanitization décidée).
- Variables : `PUBLIC_*` exposées au client → jamais sensibles ; utiliser l'API `astro:env` si disponible pour typer et séparer client/serveur.

### Emile

- `site` défini dans `astro.config` (depuis env, domaine `CONFIRMED`) : prérequis pour canonical, sitemap, RSS.
- `@astrojs/sitemap` ; exclure pages non indexables.
- Composant SEO unique : title, description, canonical, OG, Twitter, JSON-LD, `hreflang` si i18n.
- RSS pour le blog si pertinent.
- Vérifier que les pages générées depuis les collections ont des metadata issues du frontmatter.

### Julio

- CSS d'abord (transitions, keyframes, scroll-driven animations avec `@supports`).
- View Transitions (`transition:name`, `transition:animate`) uniquement si décidé ; respect de reduced-motion ; réinitialiser les scripts sur `astro:page-load`.
- GSAP/Motion uniquement dans un `<script>` ou une île justifiée, chargés après le contenu.
- Aucune animation ne transforme une page 0-JS en page avec JS sans décision.

### Noah

- `astro check` en CI.
- Vitest : schémas de collections (contenu invalide = build en échec), utilitaires, composants via Container API si utile.
- Playwright sur `astro build && astro preview` : pages clés, navigation sans JS (`javaScriptEnabled: false` sur un projet de test), formulaires, 404, îles interactives.
- Axe sur toutes les pages types (home, liste, article, 404).
- Vérifier les liens internes cassés (crawl des pages générées).

---

## 9. Stack-specific prompts

```text
@marco Établis le registre des îles : pour chaque élément interactif de STRUCTURE.md,
choisis HTML natif, <script>, ou île framework, avec directive client:* et justification.
Ajoute-le à ARCHITECTURE.md. Objectif : 0 KB JS sur les pages de contenu.
```

```text
@louis Définis les Content Collections et leurs schémas Zod à partir de CONTENT.md.
Tout champ obligatoire manquant doit faire échouer le build.
```

---

## 10. Definition of Done — additions

- [ ] `astro check` et `astro build` OK.
- [ ] Registre des îles à jour ; chaque île justifiée ; pages de contenu à 0 KB JS (ou décision).
- [ ] Site navigable sans JS.
- [ ] Schémas de collections stricts ; aucun contenu `UNKNOWN` publié.
- [ ] `site` configuré avec le domaine `CONFIRMED` ; sitemap et canonical corrects.
- [ ] Lighthouse mobile dans les seuils de §7.
