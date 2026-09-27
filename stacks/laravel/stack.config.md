# STACK — LARAVEL

> Copié dans une instance sous `hiveforge/stack.md`.
> Complète les agents core. En cas de contradiction, ce fichier prévaut sur l'agent core, sauf règle de sécurité ou règle absolue.

---

## 1. Scope

Applications web et API full-stack en **Laravel** : SaaS, back-offices, plateformes métier, API pour clients mobiles/SPA, applications avec rôles, workflows, notifications, traitements asynchrones.

Front-end possible (décision obligatoire) :

| Option | Quand |
|---|---|
| **Blade** (+ Alpine.js si besoin) | Pages majoritairement serveur, interactivité légère |
| **Livewire** (+ Alpine) | Interfaces réactives sans SPA, équipe PHP-first |
| **Inertia** (React ou Vue) | UX type SPA avec routing et contrôleurs Laravel |
| **API seule** (API Resources + Sanctum) | Client séparé (mobile, SPA externe) |

---

## 2. Version awareness

Lire `composer.json` / `composer.lock` : version de `laravel/framework`, PHP requis, starter kit, packages first-party. Depuis Laravel 11, la structure est allégée (middleware, exceptions et routes configurés dans `bootstrap/app.php`, pas de `Kernel.php` par défaut, `routes/api.php` ajouté via `php artisan install:api`). Appliquer les conventions **de la version installée** ; ne pas recréer des fichiers d'une ancienne structure sans raison.

---

## 3. Context detection (Marco)

```text
composer.json / composer.lock  → laravel/framework, php, livewire, inertia, sanctum, pest/phpunit, larastan, pint
package.json                   → vite, frontend (react/vue/alpine), tailwind
bootstrap/app.php              → routing, middleware, exceptions
routes/                        → web.php, api.php, console.php, channels.php
app/Models, app/Http, app/Policies, app/Actions|Services, app/Jobs, app/Events, app/Notifications
database/migrations, factories, seeders
config/*.php                   → auth, session, cors, queue, filesystems, cache
resources/views | resources/js → Blade / Livewire / Inertia
tests/Feature, tests/Unit      → Pest ou PHPUnit
phpstan.neon / pint.json       → qualité
.env.example                   → variables (noms uniquement ; jamais lire/afficher .env réel)
```

---

## 4. Default technical choices (proposés, à confirmer par D-NNN)

| Axe | Défaut | Notes |
|---|---|---|
| PHP | Version requise par la version de Laravel installée | |
| Architecture | MVC Laravel standard + Form Requests + Policies ; classes Action (`app/Actions`) pour la logique métier réutilisée | Pas de repository pattern sur Eloquent sans justification |
| Front | Selon §1 (décision) | |
| Auth | Starter kit officiel de la version ou Fortify ; Sanctum pour SPA/API | Pas d'auth maison |
| Autorisation | Policies (par modèle) + Gates (actions transverses) | |
| Validation | Form Requests | |
| API | API Resources, versionnage `/api/v1` si API publique | |
| DB | MySQL/MariaDB ou PostgreSQL (`UNKNOWN` → décision) ; SQLite pour les tests | |
| Queues | Driver `database` par défaut ; Redis si volume le justifie | |
| Cache | `database`/`file` → Redis si justifié | |
| Styles | Tailwind CSS via Vite | |
| Tests | Pest (ou PHPUnit si existant) | |
| Qualité | Laravel Pint, Larastan (niveau décidé) | |
| Déploiement | `UNKNOWN` → décision (Laravel Cloud, Forge, VPS, conteneurs…) | |

---

## 5. Folder architecture

```text
my-laravel-app/
├── hiveforge/
├── app/
│   ├── Actions/                     # logique métier réutilisable (une action = un cas d'usage)
│   ├── Enums/
│   ├── Events/
│   ├── Http/
│   │   ├── Controllers/             # minces : autoriser → déléguer → répondre
│   │   │   └── Api/V1/
│   │   ├── Middleware/
│   │   ├── Requests/                # Form Requests (validation + authorize())
│   │   └── Resources/               # API Resources
│   ├── Jobs/
│   ├── Listeners/
│   ├── Livewire/                    # si Livewire
│   ├── Mail/
│   ├── Models/
│   ├── Notifications/
│   ├── Policies/
│   └── Providers/
├── bootstrap/app.php                # routing, middleware, exceptions (Laravel 11+)
├── config/
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── resources/
│   ├── views/                       # Blade (layouts/, components/, pages/)
│   ├── js/                          # Inertia pages/components, ou Alpine
│   └── css/
├── routes/
│   ├── web.php
│   ├── api.php                      # si API
│   └── console.php                  # commandes + scheduling
├── storage/
├── tests/
│   ├── Feature/
│   └── Unit/
├── .env.example
├── composer.json
├── package.json
├── phpstan.neon
├── pint.json
└── vite.config.js
```

---

## 6. Commands

| Action | Commande |
|---|---|
| Dev (serveur + queue + vite) | `composer run dev` (si défini) ou `php artisan serve` + `<pm> run dev` |
| Build assets | `<pm> run build` |
| Migrations (local/test) | `php artisan migrate` |
| Statut migrations | `php artisan migrate:status` |
| Tests | `php artisan test` (ou `./vendor/bin/pest`) |
| Tests parallèles | `php artisan test --parallel` |
| Lint/format | `./vendor/bin/pint --test` (vérif) / `./vendor/bin/pint` |
| Analyse statique | `./vendor/bin/phpstan analyse` (si Larastan) |
| Routes | `php artisan route:list` |
| Modèle (introspection) | `php artisan model:show <Model>` |
| Audit deps | `composer audit` + `<pm> audit` |
| Optimisation prod | `php artisan optimize` (config/route/view/event cache) |

Interdits sans accord explicite : `migrate:fresh`, `migrate:reset`, `db:wipe`, `migrate` sur un environnement autre que local/test.

---

## 7. Performance budgets (Lamine)

| Métrique | Budget |
|---|---|
| Requêtes SQL par page / endpoint | ≤ 15, **zéro N+1** |
| Temps de réponse serveur p95 (endpoints critiques) | ≤ 300 ms (hors I/O externes) |
| Requête SQL individuelle | ≤ 50 ms sur jeu de données représentatif |
| Tâches > 1 s (emails, exports, API tierces, images) | En queue, jamais synchrones dans la requête |
| Front (Blade/Livewire) : JS (gzip) | ≤ 80 KB |
| Front (Inertia) : JS initial (gzip) | ≤ 170 KB |
| LCP / INP / CLS (pages publiques) | ≤ 2.5 s / ≤ 200 ms / ≤ 0.1 |

---

## 8. Agent overlays

### Marco

Décisions obligatoires en phase 4 :

1. Front : Blade / Livewire / Inertia (React|Vue) / API seule.
2. Auth : starter kit, Fortify, Sanctum (SPA/tokens), fournisseur externe.
3. Modèle de rôles/permissions : Policies + Gates natifs (défaut) ; package de permissions seulement si exigence (rôles dynamiques administrables).
4. SGBD, driver de queue, cache, stockage de fichiers (disque local/S3).
5. Schéma de données (entités, relations, index) dans ARCHITECTURE.md §4.
6. Convention métier : contrôleurs minces + Actions ; pas de couche Service/Repository générique sans besoin.
7. Hébergement, worker de queue et scheduler en production.

### Lea

- Contenu des emails et notifications (`Mail`, `Notification`) : `TO_VALIDATE`.
- Messages de validation personnalisés (Form Requests `messages()` / fichiers `lang/`), messages de permission refusée, états vides.
- Si multilingue : clés dans `lang/<locale>/`.

### Nora

- Pour chaque écran : rôle(s) autorisé(s) et comportement en cas de refus (403, masquage, redirection).
- Tableaux admin : pagination, tri, filtres, actions de masse avec confirmation.
- Retour des actions (flash messages, validation inline).

### Lamine

À analyser en priorité :

- **N+1** : `Model::preventLazyLoading()` activé hors production (dans `AppServiceProvider`) ; Laravel Debugbar/Telescope en local si décidé ; comptage des requêtes dans les tests critiques.
- **Eager loading** : `with()`, `load()`, `withCount()`, `withSum()` ; pas de relation chargée dans une boucle de vue/ressource.
- **Index** : colonnes de `where`, `orderBy`, clés étrangères, colonnes d'unicité ; index composites selon les requêtes réelles ; `EXPLAIN` sur les requêtes lentes.
- **Query optimization** : `select()` des colonnes utiles, `chunk()`/`lazy()`/`cursor()` pour gros volumes, pagination (`paginate`/`cursorPaginate`), éviter `->get()->count()`.
- **Caching** : `Cache::remember` avec clé et invalidation définies ; `php artisan optimize` en production.
- **Queues** : emails, notifications, exports, appels API tiers, traitement d'images → `ShouldQueue` ; retries/backoff/timeouts définis.
- **Database performance** : transactions courtes, pas de verrous longs, migrations de grosses tables planifiées.
- **HTTP caching** : headers pour assets Vite (fingerprintés), réponses publiques cacheables, ETag si pertinent.

### Paul

Contrôles spécifiques Laravel :

| Risque | Contrôle |
|---|---|
| **Mass Assignment** | `$fillable` explicite (jamais `$guarded = []`) ; `create($request->validated())`, jamais `$request->all()` ; `Model::preventSilentlyDiscardingAttributes()` hors prod |
| **SQL Injection** | Eloquent/Query Builder paramétrés ; `DB::raw`, `whereRaw`, `orderByRaw`, `selectRaw` uniquement avec bindings ; colonnes de tri dynamiques en liste blanche |
| **CSRF** | Middleware CSRF actif sur `web` ; `@csrf` dans chaque formulaire ; exclusions limitées aux webhooks signés |
| **XSS** | `{{ }}` (échappé) ; `{!! !!}` interdit sauf contenu sanitizé décidé ; attention aux props Inertia/JSON injectées et à `@js`/`Js::from` |
| **Authentication** | Starter kit/Fortify ; hash via `Hash` ; throttling login ; réinitialisation de mot de passe officielle ; vérification email si requise |
| **Authorization** | Policy pour chaque modèle exposé ; `authorize()` dans Form Requests ou `Gate`/`$this->authorize()` dans contrôleurs ; `can` middleware sur routes |
| **IDOR** | Route model binding + policy ; requêtes scopées par propriétaire/tenant (`$user->projects()->findOrFail($id)`) ; `scopeBindings()` pour routes imbriquées |
| **File Upload** | Validation `file`, `mimes`/`mimetypes`, `max` ; stockage sur disque privé ; nom généré (`store()`), jamais le nom client ; téléchargement via route autorisée ou URL signée temporaire |
| **Secrets** | `.env` jamais commité ; `config()` et jamais `env()` hors fichiers `config/` ; `APP_KEY` unique par environnement |
| **Rate Limiting** | `RateLimiter::for()` pour login, API, formulaires publics ; middleware `throttle` |
| **Session Security** | `SESSION_SECURE_COOKIE=true` en prod, `http_only`, `same_site=lax`, régénération de session au login (fournie par le starter kit) |
| **Debug / exposition** | `APP_DEBUG=false` en production ; Telescope/Debugbar non accessibles en production ; pages d'erreur génériques |
| **Sanctum / CORS** | Domaines stateful et `config/cors.php` en liste blanche |

### Emile

- Pages publiques Blade/Inertia : title/description par page (section/slot de layout ou `<Head>` Inertia), canonical, OG, JSON-LD.
- Inertia : pages publiques indexables → SSR Inertia activé (décision) ou rendu Blade.
- Sitemap : route dédiée générée depuis les modèles publics ou package par décision.
- `robots.txt` dans `public/` ; noindex sur back-office.

### Julio

- Blade/Livewire : CSS + Alpine.js (`x-transition`) ; transitions Livewire natives (`wire:transition`) selon version.
- Inertia : règles de la stack React (Motion si décidé).
- Pas de librairie d'animation pour un back-office sans exigence.

### Noah

- Pest (ou PHPUnit existant) avec `RefreshDatabase` (SQLite en mémoire ou base de test dédiée) ; factories pour toutes les données.
- Feature tests pour chaque route critique : invité (redirection/401), utilisateur non autorisé (**403 — test IDOR systématique**), utilisateur autorisé (200 + effet en base `assertDatabaseHas`).
- Validation : cas invalides → `assertSessionHasErrors` / 422 JSON.
- Mass assignment : tentative d'envoyer un champ protégé (ex. `is_admin`, `user_id`) → ignoré/refusé.
- `Queue::fake()`, `Mail::fake()`, `Notification::fake()`, `Storage::fake()`, `Http::fake()` pour les effets externes, avec assertions.
- Détection N+1 : `preventLazyLoading` actif en tests ; assertions de nombre de requêtes sur endpoints critiques si décidé.
- Livewire : `Livewire::test()` ; Inertia : `assertInertia()`.
- E2E navigateur (Pest browser testing / Dusk / Playwright) pour les flows critiques si décidé.
- Tests d'architecture Pest (`arch()`) : ex. aucun `env()` hors `config/`, pas de `dd()`/`dump()`.

---

## 9. Stack-specific prompts

```text
@marco Définis le modèle de données (entités, relations, index, soft deletes)
et la matrice d'autorisation (rôle × modèle × action → policy) dans ARCHITECTURE.md.
Choisis le front (Blade/Livewire/Inertia/API) avec D-NNN.
```

```text
@paul Audite chaque route de routes/web.php et routes/api.php (php artisan route:list) :
middleware auth, policy/authorize, Form Request, $fillable du modèle, rate limiting.
Tableau route → verdict.
```

```text
@lamine Active preventLazyLoading en local, parcours les pages critiques et liste les N+1,
requêtes lentes et index manquants avec correction proposée.
```

---

## 10. Definition of Done — additions

- [ ] `php artisan test` OK ; Pint OK ; Larastan OK (niveau décidé).
- [ ] Chaque modèle exposé a une Policy ; chaque route protégée a son test 403.
- [ ] Aucun `$guarded = []`, aucun `$request->all()` persisté, aucun `env()` hors `config/`.
- [ ] Aucun N+1 sur les pages/endpoints critiques.
- [ ] Tâches lentes en queue ; worker et scheduler documentés pour la production.
- [ ] `APP_DEBUG=false`, cookies sécurisés, `php artisan optimize` en production.
- [ ] Migrations réversibles (`down()`) ou décision ; aucune migration destructive non planifiée.
