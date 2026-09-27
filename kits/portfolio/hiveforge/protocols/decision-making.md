# PROTOCOL — DECISION MAKING

> Portée : tous les agents. Définit qui décide quoi, comment une décision est prise, documentée et modifiée.

---

## 1. Source de vérité

La mémoire d'un agent n'est **jamais** une source de vérité. Seuls font foi, dans cet ordre :

1. les réponses explicites de l'utilisateur, enregistrées dans `hiveforge/docs/` ;
2. `hiveforge/docs/DECISIONS.md` (décisions `ACCEPTED`) ;
3. les autres documents de `hiveforge/docs/` ;
4. le code réel du repository (qui décrit ce qui **est**, pas forcément ce qui **doit être**).

Si le code contredit une décision `ACCEPTED`, c'est une issue à signaler, pas une raison de modifier la décision silencieusement.

---

## 2. Ordre de priorité des instructions

En cas de contradiction entre instructions :

```text
1. Contraintes spécifiques au projet (PROJECT.md#Constraints, réponses utilisateur)
2. hiveforge/config.md
3. hiveforge/docs/*.md (DECISIONS.md prévaut sur les autres docs)
4. hiveforge/stack.md (instructions spécifiques à la stack)
5. hiveforge/agents/*.agent.md (instructions core)
6. Connaissances générales du framework
```

La règle la plus spécifique au projet prévaut, **sauf** si elle viole :

- une règle de sécurité (voir `paul.agent.md`) ;
- une règle absolue (ne jamais inventer, ne jamais écrire de secret, ne jamais écraser sans tracer) ;
- une obligation légale identifiée (données personnelles, accessibilité légale si applicable).

Dans ce cas l'agent s'arrête, signale le conflit et Marco arbitre.

---

## 3. Matrice des droits de décision

| Domaine | Décide | Consulté | Validation finale |
|---|---|---|---|
| Périmètre fonctionnel, priorités | Utilisateur | Marco, Lea | Utilisateur |
| Exigences (FR/NFR) | Marco | Lea, Nora | Utilisateur |
| Contenu, ton, messages | Lea | Emile, Nora | Utilisateur |
| Architecture de l'information, flows, états UI | Nora | Lea, Louis | Marco |
| Design system (tokens, typographie, couleurs) | Nora | Julio, Lamine | Utilisateur / Marco |
| Stack, architecture technique, dépendances | Marco | Louis, Lamine, Paul | Marco (utilisateur si coût/impact business) |
| Implémentation locale (nommage, découpage interne) | Louis | — | Marco en revue |
| Budgets et stratégie de performance | Lamine | Louis | Marco |
| Politique de sécurité | Paul | Louis | Marco (non contournable sans dérogation) |
| Stratégie SEO | Emile | Lea, Lamine | Marco |
| Motion design | Julio | Nora, Lamine | Marco |
| Stratégie et couverture de tests | Noah | Louis | Marco |
| Passage de phase, livraison | Marco | Tous | Utilisateur pour la livraison |

Un agent ne prend **jamais** une décision hors de sa ligne. Il la **propose** (statut `PROPOSED`) et Marco tranche.

---

## 4. Quand une décision doit-elle être écrite ?

Écrire une entrée dans `DECISIONS.md` si **au moins une** condition est vraie :

- ajout, suppression ou remplacement d'une dépendance ;
- choix d'un pattern structurant (rendering, state, auth, data fetching, routing, stockage) ;
- choix difficile à inverser (schéma de base de données, URLs publiques, contrat d'API) ;
- arbitrage entre deux agents ;
- dérogation à une règle, un budget ou une issue `HIGH`/`CRITICAL` ;
- hypothèse `ASSUMED` qui conditionne l'architecture ;
- modification d'une décision existante.

Ne pas écrire de décision pour les détails d'implémentation locaux et réversibles.

---

## 5. Format d'une décision

```markdown
## D-007 — Utiliser Drizzle ORM pour l'accès PostgreSQL

- Status: PROPOSED | ACCEPTED | SUPERSEDED by D-0XX | REJECTED
- Date: YYYY-MM-DD
- Phase: 4 — Architecture
- Proposed by: louis
- Decided by: marco
- Related: FR-004, NFR-002, PERF-001

### Context
<Problème, contraintes, faits CONFIRMED>

### Options
1. <Option A> — avantages / inconvénients
2. <Option B> — avantages / inconvénients
3. Ne rien ajouter (solution native) — avantages / inconvénients

### Anti-bloat check
1. Pourquoi est-ce nécessaire ?
2. Existe-t-il une solution native ?
3. Quel est le coût (poids, complexité, apprentissage, lock-in) ?
4. Quel est le gain mesurable ?
5. Quelle maintenance supplémentaire ?

### Decision
<Choix retenu, formulé de manière exécutable>

### Consequences
<Ce que cela implique pour chaque agent concerné>

### Revisit when
<Condition qui justifierait de rouvrir la décision>
```

La section `Anti-bloat check` est obligatoire pour toute dépendance, abstraction, service ou composant d'infrastructure. Si le gain n'est pas supérieur au coût : **ne pas ajouter**.

---

## 6. Modifier une décision existante

Il est interdit de réécrire une décision `ACCEPTED`. Procédure :

1. identifier la décision (`D-NNN`) ;
2. expliquer en quoi elle ne tient plus (fait nouveau, mesure, exigence modifiée) ;
3. créer une nouvelle décision qui la remplace ;
4. passer l'ancienne au statut `SUPERSEDED by D-MMM` (seule modification autorisée sur l'ancienne) ;
5. lister le travail à reprendre dans `TODO.md`.

---

## 7. Hypothèses

Une hypothèse (`ASSUMED`) est acceptable uniquement si :

- elle n'est pas bloquante ;
- elle est réversible à faible coût ;
- elle est écrite là où elle est utilisée **et** listée dans `PROJECT.md#Unknowns` ou `DECISIONS.md` ;
- elle ne concerne pas : données métier, prix, mentions légales, identité du client, credentials, URLs de production, API tierces.

Dans tous les autres cas : `UNKNOWN` + question `Q-NNN`.

---

## 8. Principe de simplicité

À exigences égales, choisir dans cet ordre :

```text
Plateforme native (HTML, CSS, Web APIs, fonctionnalités du framework)
  > Code local simple
  > Dépendance déjà présente dans le projet
  > Nouvelle dépendance légère, maintenue, largement adoptée
  > Nouvelle dépendance lourde ou service externe
```

Une technologie n'est jamais choisie parce qu'elle est populaire, récente, ou « au cas où ».
