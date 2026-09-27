# PROTOCOL — COMMUNICATION

> Portée : tous les agents. Ce protocole est obligatoire. Un rapport qui ne le respecte pas est considéré comme `NEEDS_REVIEW` par Marco.

---

## 1. Chemins de référence

Dans une instance de projet, tous les chemins sont relatifs à la racine du repository :

```text
hiveforge/config.md          → configuration du projet (stack, phase, agents actifs)
hiveforge/stack.md           → configuration de la stack (overlays par agent, commandes, arborescence)
hiveforge/agents/*.agent.md  → définitions des agents
hiveforge/protocols/*.md     → protocoles communs (ce fichier et les autres)
hiveforge/docs/*.md          → SOURCE DE VÉRITÉ du projet
hiveforge/PROMPTS.md         → commandes d'orchestration prêtes à l'emploi
```

Le code applicatif vit hors de `hiveforge/`.

---

## 2. Invocation d'un agent

`@nom` signifie : **charger `hiveforge/agents/<nom>.agent.md` et adopter intégralement ce rôle** pour la durée de la tâche.

Un agent invoqué commence toujours par la séquence de bootstrap :

1. lire `hiveforge/config.md` ;
2. lire la section qui le concerne dans `hiveforge/stack.md` ;
3. lire les fichiers listés dans sa section `Files Read` ;
4. lire le dernier état de `hiveforge/docs/TODO.md` ;
5. inspecter le code réel concerné par la tâche.

Aucun agent ne suppose qu'un projet est vide parce que ses documents sont vides.

---

## 3. Étiquettes de certitude

Toute information factuelle écrite dans `hiveforge/docs/` porte une étiquette lorsqu'elle n'est pas triviale :

| Étiquette | Sens | Qui peut l'utiliser pour construire ? |
|---|---|---|
| `CONFIRMED` | Fournie ou validée explicitement par l'utilisateur, ou vérifiée dans le code | Tous |
| `ASSUMED` | Hypothèse raisonnable, justifiée, non bloquante | Tous, en le signalant dans le rapport |
| `TO_VALIDATE` | Proposition d'un agent en attente de validation utilisateur/Marco | Personne en production ; autorisé en brouillon |
| `UNKNOWN` | Information absente | Personne. Placeholder visible uniquement |

Règle : on ne transforme jamais `UNKNOWN` en valeur inventée. On le remplace uniquement par une information `CONFIRMED` ou par une hypothèse `ASSUMED` explicitement justifiée et non bloquante.

---

## 4. Brief de tâche (Marco → Agent)

Marco délègue avec ce format :

```text
TASK: T-<NNN>
TO: @<agent>
PHASE: <numéro et nom de phase>
GOAL: <résultat attendu, une phrase>
CONTEXT: <documents et décisions à lire en priorité, ex. D-004, STRUCTURE.md#pages>
SCOPE: <ce qui est inclus>
OUT_OF_SCOPE: <ce qui est explicitement exclu>
CONSTRAINTS: <contraintes techniques ou métier>
DONE_WHEN: <critères vérifiables>
```

Un agent qui reçoit un brief ambigu répond `BLOCKED` avec ses questions plutôt que d'interpréter librement.

Lorsque l'utilisateur invoque un agent directement (sans brief), l'agent reconstruit implicitement le brief à partir de la demande et le reformule en une ligne en tête de son rapport.

---

## 5. Rapport standard (Agent → Marco / Utilisateur)

Chaque intervention se termine **obligatoirement** par ce bloc :

```text
AGENT: <nom>
TASK: <T-NNN ou "direct">
STATUS: DONE | BLOCKED | NEEDS_REVIEW | FAILED

SUMMARY:
<2 à 5 lignes maximum>

CHANGES:
- <modification concrète>

FILES:
- <chemin> (created | modified | deleted)

DECISIONS:
- <D-NNN proposée ou appliquée, ou "none">

ISSUES:
- <ID> [SEVERITY] <description courte> → <owner proposé>

EVIDENCE:
- <commande exécutée + résultat, mesure, capture, extrait de code, ou "none">

NEXT:
- <prochaine action recommandée et agent concerné>
```

Définition des statuts :

| Statut | Quand l'utiliser |
|---|---|
| `DONE` | Les critères `DONE_WHEN` sont satisfaits **et** prouvés dans `EVIDENCE` |
| `NEEDS_REVIEW` | Travail produit mais nécessite validation (Marco, utilisateur, ou autre agent) |
| `BLOCKED` | Impossible de continuer sans information/décision externe. Lister précisément ce qui manque |
| `FAILED` | Tentative réalisée mais résultat non conforme. Expliquer pourquoi et ce qui a été laissé en l'état |

Interdiction : déclarer `DONE` sans preuve. « Le code devrait fonctionner » n'est pas une preuve.

---

## 6. Questions à l'utilisateur

Une question n'est posée que si l'information est **bloquante** ou si une mauvaise hypothèse coûterait cher à corriger.

Format :

```text
QUESTION Q-<NNN> (<agent>, bloquant: oui|non)
Contexte : <pourquoi la question se pose>
Question : <une seule question précise>
Options : A) ... B) ... C) Autre
Recommandation : <option + raison en une ligne>
Impact si non répondu : <ce qui reste UNKNOWN / bloqué>
```

Règles :

- regrouper les questions (maximum 7 par tour) ;
- ne jamais demander ce qui est déjà dans `hiveforge/docs/` ou dans le code ;
- enregistrer la réponse dans le document concerné avec l'étiquette `CONFIRMED`.

---

## 7. Format des problèmes (issues)

Préfixes par domaine :

| Préfixe | Owner | Préfixe | Owner |
|---|---|---|---|
| `REQ-` | Marco | `SEC-` | Paul |
| `CNT-` | Lea | `SEO-` | Emile |
| `UX-` | Nora | `ANIM-` | Julio |
| `ARCH-` | Marco | `QA-` | Noah |
| `IMPL-` | Louis | `PERF-` | Lamine |

Sévérités communes :

| Sévérité | Définition | Bloque la livraison ? |
|---|---|---|
| `CRITICAL` | Faille exploitable, perte de données, flow critique cassé, build cassé | Oui |
| `HIGH` | Exigence non respectée, régression importante, dégradation majeure | Oui, sauf dérogation écrite dans DECISIONS.md |
| `MEDIUM` | Défaut visible mais contournable | Non, mais doit figurer dans TODO.md |
| `LOW` | Amélioration mineure | Non |
| `INFO` | Observation, aucune action requise | Non |

Format d'une issue dans un document d'audit :

```markdown
### PERF-003 [HIGH] Image hero non optimisée
- Location: `app/page.tsx:42`
- Evidence: LCP 4.1s (Lighthouse mobile), image 1.8 MB
- Impact: LCP > 2.5s sur mobile
- Recommendation: <action précise>
- Owner: louis
- Status: OPEN | IN_PROGRESS | FIXED | VERIFIED | WONTFIX (D-NNN)
```

Une issue passe à `VERIFIED` uniquement lorsque **l'agent qui l'a ouverte** a revérifié la correction.

---

## 8. Identifiants

| Objet | Format | Registre |
|---|---|---|
| Décision | `D-001` | `hiveforge/docs/DECISIONS.md` |
| Tâche | `T-001` | `hiveforge/docs/TODO.md` |
| Question | `Q-001` | `hiveforge/docs/TODO.md` (section Questions ouvertes) |
| Exigence | `FR-001` / `NFR-001` | `hiveforge/docs/REQUIREMENTS.md` |
| Issue | `<PREFIX>-001` | Document d'audit du domaine |

Les identifiants ne sont jamais réutilisés, même après suppression.

---

## 9. Style

- Factuel, concis, vérifiable.
- Citer les fichiers avec leur chemin et, si utile, le numéro de ligne.
- Pas de flatterie, pas de remplissage, pas de résumé de ce qui a déjà été dit.
- La langue des documents est celle définie dans `hiveforge/config.md` (`Language`). Les mots-clés du protocole (`STATUS`, `DONE`, `CONFIRMED`…) restent en anglais.
