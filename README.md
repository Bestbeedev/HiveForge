# HiveForge

**Kit d'orchestration multi-agents IA réutilisable pour lancer des projets logiciels** avec une architecture propre, documentée, testable, performante et maintenable.

Pattern : **Advisors → Builder → Implementation → Audit → Validation**

---

## ⚡ Démarrage rapide (copier-coller, sans installation)

```text
kits/
├── portfolio/hiveforge/
├── nextjs/hiveforge/
├── react/hiveforge/
├── astro/hiveforge/
└── laravel/hiveforge/
```

1. Créer votre projet (ex. `pnpm create next-app mon-projet`).
2. Copier le dossier `kits/nextjs/hiveforge` à la racine de `mon-projet/`.
   - Finder : glisser-déposer le dossier `hiveforge` dans le projet.
   - Terminal : `cp -R kits/nextjs/hiveforge ~/Desktop/mon-projet/`
3. Remplir `hiveforge/config.md` et `hiveforge/docs/PROJECT.md`.
4. Dans le chat de votre IDE (Antigravity, Cursor, VS Code, Claude Code…), mentionner le fichier de l'agent :

   ```text
   @marco.agent.md Initialise le projet.
   ```

C'est tout. Les prompts suivants sont dans `hiveforge/PROMPTS.md`.

> Les dossiers `kits/` sont **générés** à partir de `core/` + `stacks/` + `templates/`. Si vous modifiez un agent, relancez `bin/hiveforge build` pour régénérer les 5 kits (ne modifiez pas `kits/` à la main).

```text
HIVEFORGE CORE  (agents génériques, protocoles, templates de docs)
      ↓
STACK ADAPTER   (Louis spécialisé + règles de stack pour chaque agent)
      ↓
PROJECT INSTANCE (hiveforge/ autonome, versionné avec le projet)
      ↓
CODEBASE
```

---

## 1. Structure du repository

```text
HiveForge/
├── bin/hiveforge                  # CLI d'initialisation (bash, zéro dépendance)
├── VERSION
├── core/
│   ├── agents/                    # 8 agents indépendants de la stack
│   │   ├── marco.agent.md         # orchestrateur / architecte
│   │   ├── lea.agent.md           # contenu
│   │   ├── nora.agent.md          # structure UX + design system
│   │   ├── lamine.agent.md        # performance
│   │   ├── paul.agent.md          # sécurité
│   │   ├── emile.agent.md         # SEO
│   │   ├── julio.agent.md         # animation
│   │   └── noah.agent.md          # QA / tests
│   ├── protocols/
│   │   ├── communication.md       # invocation, rapports, étiquettes, IDs, sévérités
│   │   ├── decision-making.md     # source de vérité, priorités, droits de décision, ADR, anti-bloat
│   │   ├── conflict-resolution.md # flux et critères d'arbitrage
│   │   └── definition-of-done.md  # phases, gates, DoD globale avec preuves
│   └── templates/                 # 13 documents de projet (docs/)
├── stacks/
│   ├── portfolio/  { louis.agent.md, stack.config.md }
│   ├── nextjs/     { louis.agent.md, stack.config.md }
│   ├── react/      { louis.agent.md, stack.config.md }
│   ├── astro/      { louis.agent.md, stack.config.md }
│   └── laravel/    { louis.agent.md, stack.config.md }
├── templates/
│   ├── PROMPTS.md                 # prompts d'orchestration (copiés dans l'instance)
│   ├── README.md                  # README de l'instance
│   └── project-config.md          # devient hiveforge/config.md
└── README.md
```

### Principe Core + Stack

- Les 8 agents core sont **identiques** pour toutes les stacks (aucune duplication).
- **Louis** est le seul agent réécrit par stack (son travail dépend directement du framework).
- Les règles spécifiques d'une stack pour les autres agents (ex. N+1 Laravel pour Lamine, Server Actions Next.js pour Paul, GSAP pour Julio en portfolio) vivent dans `stack.config.md`, section **Agent overlays**. Chaque agent core lit sa section.

Chaque `stack.config.md` contient :

| Section | Contenu |
|---|---|
| 1. Scope | Quand utiliser cette stack, quand en changer |
| 2. Version awareness | Comment traiter les différences de versions du framework |
| 3. Context detection | Fichiers que Marco inspecte pour détecter l'état réel |
| 4. Default technical choices | Choix par défaut **proposés** (confirmés par décision) |
| 5. Folder architecture | Arborescence cible du projet |
| 6. Commands | Build, test, lint, migrations, audit |
| 7. Performance budgets | Budgets par défaut de Lamine |
| 8. Agent overlays | Règles spécifiques pour Marco, Lea, Nora, Lamine, Paul, Emile, Julio, Noah |
| 9. Stack-specific prompts | Prompts additionnels |
| 10. DoD additions | Critères de fin propres à la stack |

---

## 2. Installation

```bash
git clone <url-de-ce-repo> ~/HiveForge
chmod +x ~/HiveForge/bin/hiveforge
# Optionnel : rendre la commande disponible partout
echo 'export PATH="$HOME/HiveForge/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

Vérifier :

```bash
hiveforge list
# astro  laravel  nextjs  portfolio  react
```

---

## 3. Démarrer un projet

### Étape 1 — Créer le projet avec sa stack

```bash
pnpm create next-app my-project     # ou: composer create-project laravel/laravel, pnpm create astro, pnpm create vite…
cd my-project
```

(Pour un projet from scratch, on peut aussi initialiser HiveForge dans un dossier vide : Marco détectera l'état `EMPTY`.)

### Étape 2 — Initialiser HiveForge

```bash
hiveforge init --stack nextjs
```

Résultat — une instance **autonome** :

```text
my-project/hiveforge/
├── config.md          # à remplir
├── stack.md           # = stacks/nextjs/stack.config.md
├── PROMPTS.md
├── README.md
├── .version
├── agents/            # 8 agents core + louis.agent.md de la stack
├── protocols/         # 4 protocoles
└── docs/              # 13 templates
```

**Sans CLI** (copie manuelle) :

```text
core/agents/*                         → hiveforge/agents/
stacks/<stack>/louis.agent.md         → hiveforge/agents/louis.agent.md
stacks/<stack>/stack.config.md        → hiveforge/stack.md
core/protocols/*                      → hiveforge/protocols/
core/templates/*                      → hiveforge/docs/
templates/PROMPTS.md                  → hiveforge/PROMPTS.md
templates/project-config.md           → hiveforge/config.md
```

Puis remplacer `{{STACK}}` dans `config.md` et `PROMPTS.md`.

### Étape 3 — Décrire le projet

Remplir `hiveforge/config.md` et `hiveforge/docs/PROJECT.md`. Écrire `UNKNOWN` pour ce qui n'est pas connu : c'est normal et c'est le rôle de Marco et Lea de le compléter avec vous.

### Étape 4 — Brancher votre assistant IA

`@marco` signifie « charger `hiveforge/agents/marco.agent.md` et adopter ce rôle ». Selon l'outil :

| Outil | Méthode |
|---|---|
| Claude Code | Référencer le fichier (`@hiveforge/agents/marco.agent.md`) dans le prompt, ou ajouter dans `CLAUDE.md` : « Quand je tape `@<nom>`, lis `hiveforge/agents/<nom>.agent.md` et respecte `hiveforge/protocols/`. » |
| Cursor / Windsurf | Même instruction dans les rules du projet ; ou mentionner le fichier avec `@` |
| GitHub Copilot | Même instruction dans `.github/copilot-instructions.md` |
| Autre / chat | Coller en tête : « Lis `hiveforge/agents/<nom>.agent.md` et agis strictement en tant que cet agent. » |

### Étape 5 — Lancer Marco

```text
@marco Initialise le projet.
```

Marco détecte le contexte, écrit REQUIREMENTS / DECISIONS / TODO, pose les questions bloquantes, puis indique les agents à appeler ensuite.

### Étape 6 — Suivre le workflow jusqu'à la livraison

Utiliser `hiveforge/PROMPTS.md`. Marco ne déclare le projet terminé que lorsque la Definition of Done est prouvée.

### Mise à jour d'une instance

```bash
hiveforge upgrade --stack nextjs --dir ./my-project
```

Remplace `agents/`, `protocols/`, `stack.md`, `PROMPTS.md`, `README.md`. **Ne touche jamais** `config.md` ni `docs/`.

---

## 4. Workflow officiel

```text
PHASE 0  DISCOVERY        Marco     → REQUIREMENTS, DECISIONS, TODO
PHASE 1  CONTENT          Lea       → CONTENT
PHASE 2  STRUCTURE / UX   Nora      → STRUCTURE
PHASE 3  DESIGN           Nora      → DESIGN
PHASE 4  ARCHITECTURE     Marco     → ARCHITECTURE, DECISIONS (+ budgets Lamine, menaces Paul)
PHASE 5  IMPLEMENTATION   Louis     → code (Noah prépare les tests en parallèle)
PHASE 6  PERFORMANCE      Lamine    → PERFORMANCE   ┐
PHASE 7  SECURITY         Paul      → SECURITY      ├ parallélisables
PHASE 8  SEO              Emile     → SEO           ┘
PHASE 9  ANIMATION        Julio     → ANIMATIONS
PHASE 10 TESTING          Noah      → TESTING
PHASE 11 FINAL AUDIT      Marco     → consolidation, corrections, DoD
PHASE 12 DELIVERY         Marco + utilisateur
```

Chaque phase se termine par un **gate** contrôlé par Marco (`protocols/definition-of-done.md §1`). Marco peut paralléliser des tâches indépendantes (§3 du même fichier) et désactiver un agent inutile (ex. Emile pour une app privée) par décision tracée.

### Ordre recommandé des agents

```text
Marco → Lea → Nora → Marco → Louis → Lamine | Paul | Emile → Julio → Noah → Marco
```

---

## 5. Règles fondamentales

1. **Source de vérité = `hiveforge/docs/`**, jamais la mémoire d'un agent.
2. **Ne jamais inventer** : fonctionnalités, données métier, URLs, credentials, API, dépendances, exigences. → `UNKNOWN` puis question si bloquant.
3. **Étiquettes de certitude** : `CONFIRMED` · `ASSUMED` · `TO_VALIDATE` · `UNKNOWN`.
4. **Lire avant d'écrire** : contexte, documents, code, décisions, contraintes.
5. **Un agent = une responsabilité** ; chaque agent ne modifie que ses fichiers (`Files Owned`).
6. **Aucune décision écrasée** : une décision `ACCEPTED` est `SUPERSEDED`, jamais réécrite.
7. **Anti-bloat** : toute dépendance/abstraction répond à 5 questions (nécessité, alternative native, coût, gain, maintenance). Gain ≤ coût → ne pas ajouter.
8. **Preuves** : aucun `DONE` sans preuve (sortie de commande, mesure, test).
9. **Aucun secret** dans `hiveforge/` (commité avec le projet).
10. **Aucune action irréversible ou externe** (déploiement prod, migration destructive, suppression de données) sans accord explicite de l'utilisateur.

Priorité des instructions : contraintes projet > `config.md` > `docs/` > `stack.md` > agents core > connaissances générales (sauf règle de sécurité ou règle absolue).

---

## 6. Protocole de communication (résumé)

Chaque agent termine par :

```text
AGENT / TASK / STATUS: DONE | BLOCKED | NEEDS_REVIEW | FAILED
SUMMARY / CHANGES / FILES / DECISIONS / ISSUES / EVIDENCE / NEXT
```

Conflits : `Agent → Marco → Décision → DECISIONS.md → correction`. Détail dans `core/protocols/`.

---

## 7. Definition of Done

```text
[ ] Requirements validés        [ ] Security auditée
[ ] Content validé              [ ] SEO audité
[ ] UX validée                  [ ] Animations validées
[ ] Architecture validée        [ ] Tests passés
[ ] Code implémenté             [ ] Bugs critiques corrigés
[ ] Responsive validé           [ ] Documentation mise à jour
[ ] Accessibility vérifiée      [ ] Build production réussi
[ ] Performance auditée         [ ] Deployment validé
```

Chaque case exige une preuve ; un item peut être `N/A` uniquement par décision tracée. Chaque stack ajoute ses critères (`stack.md §10`).

---

## 8. Les cinq stacks

| Stack | Pour | Louis maîtrise | Points forts des overlays |
|---|---|---|---|
| **portfolio** | Personal branding, conversion, vitrine | HTML/CSS/JS (Vite), Next.js statique, React si justifié | Choix de variante (D-001), GSAP/CSS pour Julio, budgets LCP renforcés, JSON-LD `Person` |
| **nextjs** | SaaS, apps full-stack, sites SEO dynamiques | App Router, RSC, Server Actions, shadcn/ui, Drizzle, Vercel | Version awareness, Server Actions = endpoints publics, frontière `server-only`, stratégie de cache par route |
| **react** | SPA, dashboards, back-offices sur API | Vite, router, TanStack Query, état minimal (Context → Zustand → RTK) | Contrat d'API sans invention, bundle et re-renders, pas de secret dans `VITE_*` |
| **astro** | Sites de contenu, blogs, docs, landing | Islands, SSG, Content Collections, MDX | 0 KB JS par défaut, registre des îles justifiées, schémas de contenu stricts |
| **laravel** | Apps métier, SaaS, API | MVC, Eloquent, Form Requests, Policies, queues, Blade/Livewire/Inertia, Sanctum | Mass assignment, IDOR, N+1, index, queues, tests 403 systématiques |

---

## 9. Choix de conception

Ce kit applique le cahier des charges d'origine avec quelques arbitrages explicites :

| Point | Choix | Raison |
|---|---|---|
| Chemins `docs/` | Toujours `hiveforge/docs/` dans une instance | Le code applicatif reste à la racine ; les docs d'orchestration sont isolés |
| Dossier `hiveforge/` visible (et non `.hiveforge/`) | Nom sans point | Copie par glisser-déposer dans Finder, visible dans l'IDE et dans la recherche `@` du chat |
| `DESIGN.md` sans owner défini | Attribué à **Nora** | Elle possède déjà hiérarchie visuelle et accessibilité ; évite un 10ᵉ agent |
| Marco « Builder » | Marco orchestre et décide ; **Louis construit** | Une responsabilité par agent |
| Protocoles dans l'instance | `hiveforge/protocols/` copié | Les agents y renvoient ; l'instance doit être autonome |
| « Cinq systèmes autonomes » vs « aucune duplication » | Source unique core + stack ; **l'instance générée** est autonome | Pas de dérive entre 5 copies des mêmes agents |
| Versions des frameworks | Non figées dans les agents ; « version awareness » obligatoire | Les agents lisent la version installée au lieu d'appliquer des patterns périmés |
| Choix techniques par défaut | Proposés, confirmés par décision `D-NNN` | Cohérent avec la règle « ne jamais inventer » et la traçabilité |

---

## 10. Ajouter une stack

1. Créer `stacks/<nom>/`.
2. Écrire `louis.agent.md` en respectant le format standard (15 sections, voir un Louis existant).
3. Écrire `stack.config.md` avec les 10 sections listées au §1.
4. Vérifier : `hiveforge init --stack <nom> --dir /tmp/test-project`.

Ne jamais dupliquer un agent core dans une stack : utiliser la section **Agent overlays**.
