# STACK — PORTFOLIO

> Copié dans une instance sous `hiveforge/stack.md`.
> Ce fichier **complète** les agents core. Chaque agent lit sa section. En cas de contradiction, ce fichier prévaut sur l'agent core (voir `protocols/decision-making.md` §2), sauf règle de sécurité ou règle absolue.

---

## 1. Scope

Site personnel ou de studio dont les objectifs sont : **personal branding, storytelling, conversion** (contact, prise de rendez-vous, téléchargement de CV, recrutement), **performance, SEO, identité visuelle forte, animation soignée**.

Caractéristiques typiques :

- 1 à 15 pages ; contenu majoritairement statique ;
- études de cas / projets ; éventuellement un blog ;
- un formulaire de contact ou un lien de prise de rendez-vous ;
- pas de comptes utilisateurs (sinon ce n'est plus un portfolio → voir stack Next.js ou Laravel).

---

## 2. Variants (décision D-001 obligatoire en phase 4)

Le portfolio accepte plusieurs implémentations. Marco choisit **la plus simple qui satisfait les exigences** :

| Variante | Quand la choisir | Quand l'éviter |
|---|---|---|
| **A — HTML / CSS / JS** (build Vite si plusieurs pages ou assets à fingerprinter) | ≤ ~8 pages, contenu rarement modifié, pas de blog, animation GSAP/CSS | Blog, nombreux projets à maintenir, besoin de templating |
| **B — Next.js (App Router, rendu statique)** | Blog/MDX, nombreuses études de cas, routes dynamiques générées statiquement, CMS headless, besoin d'images optimisées automatiquement | Site de 3 pages sans évolution prévue |
| **C — React (Vite, SPA)** | Expérience très interactive (WebGL, configurateur) où le SEO est secondaire **et** documenté comme tel | Par défaut. Une SPA dégrade le SEO et le LCP d'un portfolio |

> Si l'utilisateur préfère Astro pour un portfolio, utiliser la stack `astro` (également adaptée), pas cette stack.

La variante choisie est écrite dans `config.md → Frontend` et dans `DECISIONS.md`.

---

## 3. Context detection (Marco)

Inspecter dans cet ordre :

```text
package.json              → next, react, vite, gsap, dépendances d'animation, scripts
next.config.*             → variante B (vérifier output: 'export' éventuel)
vite.config.*             → variante A (multi-page ?) ou C
index.html, *.html        → variante A
app/ ou src/app/          → variante B
content/, *.md(x)         → contenu éditorial
public/                   → images, CV, favicon, OG
.env.example              → intégrations (noms de variables uniquement)
vercel.json, netlify.toml → hébergement
```

---

## 4. Default technical choices (proposés, à confirmer par D-NNN)

| Axe | Variante A | Variante B | Justification |
|---|---|---|---|
| Build | Vite (vanilla), multi-page | Next.js, pages statiques (SSG), `output: 'export'` si aucun besoin serveur | Fingerprinting, minification, dev server |
| Langage | JavaScript + JSDoc, ou TypeScript si l'utilisateur le souhaite | TypeScript | |
| Styles | CSS natif (custom properties, nesting, container queries) | CSS Modules ou Tailwind CSS (si déjà présent / souhaité) | Pas de framework CSS par défaut en A |
| Animation | CSS + GSAP (ScrollTrigger) si scroll narratif approuvé | CSS + GSAP ou Motion, un seul des deux | Voir section Julio |
| Contenu | HTML / JSON local | MDX ou fichiers Markdown + frontmatter | Pas de CMS sans exigence |
| Formulaire de contact | Service de formulaire ou endpoint serverless (décision Marco + Paul) ; ou lien `mailto:` / prise de RDV | Route handler / Server Action + fournisseur email | Pas d'email en clair scrapable si l'utilisateur le refuse |
| Images | Pré-optimisées (AVIF/WebP, `srcset`) | `next/image` (ou images pré-optimisées si export statique sans loader) | |
| Fonts | Auto-hébergées WOFF2 | `next/font` | |
| Hébergement | Statique (Vercel, Netlify, Cloudflare Pages, GitHub Pages) | Vercel | Choix utilisateur, sinon `UNKNOWN` |
| Tests | Playwright (E2E + axe) | Playwright + Vitest si logique | |

Intégration design : si l'utilisateur fournit Figma / Stitch et qu'un MCP est disponible, Nora et Louis lisent les maquettes via MCP. Sans MCP, l'utilisateur fournit exports et specs.

---

## 5. Folder architecture

### Variante A — HTML / CSS / JS (Vite multi-page)

```text
my-portfolio/
├── hiveforge/
├── index.html
├── projects/
│   └── <slug>/index.html        # une page par étude de cas
├── about/index.html
├── contact/index.html
├── 404.html
├── public/
│   ├── images/                  # images optimisées (AVIF/WebP + fallback)
│   ├── fonts/                   # WOFF2 auto-hébergées
│   ├── og/                      # images Open Graph 1200×630
│   ├── cv.pdf                   # si fourni par l'utilisateur
│   ├── favicon.svg
│   ├── robots.txt
│   └── sitemap.xml
├── src/
│   ├── styles/
│   │   ├── tokens.css           # tokens de DESIGN.md
│   │   ├── base.css             # reset, typographie, éléments
│   │   ├── layout.css
│   │   ├── components/*.css
│   │   └── motion.css           # tokens + animations CSS (Julio)
│   ├── scripts/
│   │   ├── main.js              # point d'entrée, init par page
│   │   ├── nav.js
│   │   ├── contact-form.js
│   │   └── motion/              # modules GSAP (Julio), chargés à la demande
│   └── data/projects.json       # données projets (si non inline)
├── tests/e2e/
├── vite.config.js
├── package.json
└── README.md
```

### Variante B — Next.js (statique)

```text
my-portfolio/
├── hiveforge/
├── app/
│   ├── layout.tsx               # html lang, fonts, metadata par défaut, skip link
│   ├── page.tsx                 # home
│   ├── projects/
│   │   ├── page.tsx
│   │   └── [slug]/page.tsx      # generateStaticParams + generateMetadata
│   ├── about/page.tsx
│   ├── contact/page.tsx
│   ├── not-found.tsx
│   ├── sitemap.ts
│   ├── robots.ts
│   └── opengraph-image.tsx      # si OG dynamiques décidées
├── components/
│   ├── ui/                      # primitives (Button, Link, Container…)
│   ├── sections/                # Hero, ProjectGrid, Testimonials…
│   └── motion/                  # composants client d'animation (Julio)
├── content/projects/*.mdx
├── lib/
│   ├── content.ts               # lecture/validation du contenu
│   └── site.ts                  # config du site (URL via env)
├── public/
├── styles/ (ou app/globals.css)
├── tests/e2e/
├── next.config.ts
├── package.json
└── tsconfig.json
```

---

## 6. Commands

Lire les scripts réels dans `package.json`. Valeurs attendues par défaut :

| Action | Variante A | Variante B |
|---|---|---|
| Dev | `<pm> run dev` | `<pm> run dev` |
| Build | `<pm> run build` | `<pm> run build` |
| Preview prod | `<pm> run preview` | `<pm> run start` (ou serveur statique sur `out/`) |
| Lint | `<pm> run lint` (si configuré) | `<pm> run lint` |
| Typecheck | — / `tsc --noEmit` | `tsc --noEmit` |
| E2E | `<pm> exec playwright test` | `<pm> exec playwright test` |
| Audit deps | `<pm> audit` | `<pm> audit` |

`<pm>` = package manager de `config.md` (pnpm, npm, yarn, bun).

---

## 7. Performance budgets (Lamine)

| Métrique | Budget |
|---|---|
| LCP mobile (p75) | ≤ 2.0 s (portfolio = vitrine, exigence renforcée) |
| CLS | ≤ 0.05 |
| INP | ≤ 200 ms |
| JS initial (gzip), page d'accueil | ≤ 70 KB hors librairie d'animation ; ≤ 120 KB avec GSAP + ScrollTrigger |
| CSS (gzip) | ≤ 30 KB |
| Image LCP | ≤ 150 KB |
| Fonts | ≤ 2 familles, ≤ 4 fichiers, ≤ 150 KB total |
| Lighthouse (mobile) | Performance ≥ 90, Accessibility ≥ 95, SEO = 100, Best Practices ≥ 95 |

---

## 8. Agent overlays

### Marco

- Décider la variante (D-001) avant toute autre décision technique.
- Un portfolio n'a pas de base de données, d'auth ni de backend, sauf exigence explicite. Refuser leur ajout « au cas où ».
- Le formulaire de contact est la seule surface d'attaque habituelle : impliquer Paul dès la phase 4.
- Activer Julio par défaut (l'animation fait partie de l'identité) mais avec un inventaire limité et validé.

### Lea

- Le portfolio vend une personne ou un studio : priorité à la **preuve** (projets, résultats, rôle exact dans chaque projet), fournie par l'utilisateur uniquement.
- Structure recommandée d'une étude de cas : contexte → problème → rôle → démarche → résultat → apprentissages. Chaque résultat chiffré doit être `CONFIRMED`.
- Hero : qui (nom/rôle), pour qui, quelle valeur, CTA — en moins de 20 mots pour le titre + sous-titre.
- CTA unique et répété : contact / prise de RDV / CV.
- Ne jamais inventer : clients, employeurs, logos, témoignages, années d'expérience, stack maîtrisée.

### Nora

- Parcours prioritaire : arrivée → preuve (projets) → confiance (à propos, témoignages fournis) → contact. Maximum 2 clics vers le contact depuis n'importe quelle page.
- Galerie de projets : filtre seulement si > 9 projets.
- Chaque étude de cas : navigation projet précédent / suivant.
- Identité visuelle forte **mais** contrastes AA et lisibilité non négociables.
- Intégration Figma/Stitch/MCP : transcrire fidèlement ; signaler les écarts d'accessibilité.

### Lamine

- Les images de projets sont le premier risque : formats modernes, `srcset`, lazy loading hors viewport, dimensions explicites.
- GSAP : importer uniquement les plugins utilisés ; charger les modules de scroll après le rendu initial (import dynamique) si non nécessaires au LCP.
- Vidéos : `preload="none"` ou `metadata`, poster optimisé, pas d'autoplay avec son ; pas de vidéo comme élément LCP si évitable.
- Pas de librairie de smooth-scroll sans décision (impact INP et accessibilité).

### Paul

- Formulaire de contact : validation côté serveur/service, anti-spam (honeypot et/ou captcha respectueux de la vie privée), rate limiting, pas d'injection d'en-têtes email (CRLF), pas d'email du destinataire exposé dans le JS si l'utilisateur ne le souhaite pas.
- Headers de sécurité via la configuration de l'hébergeur (`_headers`, `vercel.json`, `next.config`).
- Scripts tiers (analytics, embeds) : minimum, listés dans la CSP.
- Aucune clé d'API de service email côté client.

### Emile

- JSON-LD : `Person` (ou `Organization` pour un studio) sur la home, `WebSite`, `BreadcrumbList` sur les études de cas, `CreativeWork`/`Article` pour les projets/articles si pertinent.
- `sameAs` uniquement avec les profils fournis par l'utilisateur.
- Chaque étude de cas est une page indexable avec title/description propres.
- Images OG par page importante.

### Julio

Priorité technologique :

```text
CSS (transitions, keyframes, scroll-driven si supporté)
  → GSAP (+ ScrollTrigger) pour séquences et scroll narratif
  → Micro-interactions
  → Transitions de page (View Transitions API ; fallback instantané)
```

- Budget : 1 séquence d'intro maximum, ≤ 1 s, **qui ne retarde pas le LCP** (le contenu est visible immédiatement ; l'animation l'enrichit).
- Pas de scroll-jacking. Pas de curseur custom qui remplace le curseur natif sans décision et sans fallback.
- Reduced motion : désactiver ScrollTrigger scrub/pin et parallax ; conserver fondus courts.
- GSAP : `gsap.matchMedia()` pour gérer reduced-motion et breakpoints ; `ctx.revert()` au nettoyage.

### Noah

- Flows critiques : home → projet → contact ; envoi du formulaire (succès, erreur de validation, erreur réseau) ; téléchargement du CV si présent ; 404.
- Vérifier reduced-motion (Playwright `reducedMotion: 'reduce'`) : contenu visible, pas d'élément bloqué à `opacity: 0`.
- Lighthouse CI ou mesure Lighthouse sur les pages critiques (en coordination avec Lamine).
- Axe sur toutes les pages.

---

## 9. Stack-specific prompts

```text
@marco Détermine la variante du portfolio (A HTML/CSS/JS, B Next.js statique, C React SPA)
à partir de PROJECT.md et REQUIREMENTS.md. Documente D-001 avec anti-bloat check.
Ne code rien.
```

```text
@nora Importe la maquette depuis <Figma/Stitch via MCP | fichiers fournis> et transcris-la
dans DESIGN.md (tokens + composants). Signale chaque écart d'accessibilité en UX-NNN.
```

```text
@julio Propose l'inventaire d'animations du portfolio dans ANIMATIONS.md
(intro, micro-interactions, scroll, transitions de page) avec fonction UX,
coût perf estimé et stratégie reduced-motion. N'implémente rien avant validation de Marco.
```

---

## 10. Definition of Done — additions

- [ ] Lighthouse mobile sur home + 1 étude de cas : seuils de §7 atteints.
- [ ] Toutes les études de cas ont contenu `CONFIRMED`, images optimisées, metadata, OG.
- [ ] Formulaire de contact testé de bout en bout (ou lien de contact vérifié).
- [ ] Animations vérifiées en reduced-motion.
- [ ] Favicon, 404, robots, sitemap présents.
