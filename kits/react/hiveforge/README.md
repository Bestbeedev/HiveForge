# HiveForge — Orchestration multi-agents de ce projet

> Stack : **react** · Généré par HiveForge 1.0.0 le 2026-09-27.
> Ce dossier est versionné avec le projet. Il ne contient **aucun secret**.

## Contenu

```text
hiveforge/
├── config.md          → configuration du projet (à remplir)
├── stack.md           → règles de la stack react pour chaque agent
├── PROMPTS.md         → prompts prêts à l'emploi
├── agents/            → 9 agents (Marco, Lea, Nora, Louis, Lamine, Paul, Emile, Julio, Noah)
├── protocols/         → communication, décisions, conflits, Definition of Done
└── docs/              → SOURCE DE VÉRITÉ du projet (13 documents)
```

## Démarrage en 3 étapes

1. Remplir `hiveforge/config.md` et `hiveforge/docs/PROJECT.md` (ce qui est connu ; `UNKNOWN` pour le reste).
2. Lancer :

   ```text
   @marco.agent.md Initialise le projet.
   ```

3. Suivre les prochaines actions proposées par Marco (voir `PROMPTS.md`).

## Les agents

| Agent | Rôle | Document principal |
|---|---|---|
| Marco | Orchestrateur, architecte, arbitre | REQUIREMENTS, ARCHITECTURE, DECISIONS, TODO |
| Lea | Contenu | CONTENT |
| Nora | Structure UX + design system | STRUCTURE, DESIGN |
| Louis | Implémentation (react) | Code |
| Lamine | Performance | PERFORMANCE |
| Paul | Sécurité | SECURITY |
| Emile | SEO | SEO |
| Julio | Animation | ANIMATIONS |
| Noah | QA / tests | TESTING |

## Workflow

```text
Discovery → Content → Structure → Design → Architecture → Implementation
→ [Performance | Security | SEO] → Animation → Testing → Final audit → Delivery
```

## Règles essentielles

- Les documents de `docs/` font foi, pas la mémoire des agents.
- Aucune information inventée : `UNKNOWN` puis question.
- Chaque décision structurante est tracée dans `docs/DECISIONS.md`.
- Chaque agent ne modifie que ses fichiers.
- Marco seul déclare une phase ou le projet terminés, preuves à l'appui.

## Mettre à jour les agents

Depuis le repository HiveForge :

```bash
hiveforge upgrade --stack react --dir <chemin-du-projet>
```

`upgrade` remplace `agents/`, `protocols/`, `stack.md`, `PROMPTS.md` et ce README, sans toucher à `config.md` ni à `docs/`.
