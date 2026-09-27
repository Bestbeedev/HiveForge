# PROTOCOL — DEFINITION OF DONE

> Portée : tous les agents. Marco est le seul habilité à déclarer une phase ou le projet terminés. Chaque case exige une **preuve**, pas une affirmation.

---

## 1. Phases et gates

Chaque phase se termine par un **gate** contrôlé par Marco. Une phase non validée ne peut pas servir d'entrée à la suivante, sauf parallélisation explicitement décidée (voir §3).

| # | Phase | Owner principal | Livrable | Gate (conditions de sortie) |
|---|---|---|---|---|
| 0 | Discovery | Marco | REQUIREMENTS.md, DECISIONS.md initial | Objectifs, utilisateurs, FR/NFR listés ; inconnues bloquantes posées en questions |
| 1 | Content | Lea | CONTENT.md | Aucun `UNKNOWN` bloquant sur les pages prioritaires ; CTA définis ; validé par l'utilisateur |
| 2 | Structure / UX | Nora | STRUCTURE.md | Pages, sections, composants, flows, états UI définis ; chaque FR couvert |
| 3 | Design | Nora | DESIGN.md | Tokens (couleurs, typo, espacements), contrastes AA vérifiés, composants de base spécifiés |
| 4 | Architecture | Marco | ARCHITECTURE.md, DECISIONS.md | Stack, rendering, data, auth, dépendances décidés (`ACCEPTED`) ; budgets perf/sécu posés |
| 5 | Implementation | Louis | Code | Build, lint, typecheck OK ; toutes les tâches `T-` d'implémentation fermées |
| 6 | Performance | Lamine | PERFORMANCE.md | Budgets respectés ou dérogations documentées ; aucune issue `HIGH`+ ouverte |
| 7 | Security | Paul | SECURITY.md | Aucune issue `HIGH`+ ouverte ; secrets absents du repo ; dépendances auditées |
| 8 | SEO | Emile | SEO.md | Metadata, sitemap, robots, données structurées, headings validés |
| 9 | Animation | Julio | ANIMATIONS.md | Chaque animation justifiée ; `prefers-reduced-motion` respecté ; pas de régression perf |
| 10 | Testing | Noah | TESTING.md | Flows critiques `PASS` ; aucun `FAIL` critique ; régressions vérifiées |
| 11 | Final audit | Marco | DECISIONS.md, TODO.md | Toutes les issues `CRITICAL`/`HIGH` `VERIFIED` ou dérogées ; docs à jour |
| 12 | Delivery | Marco + utilisateur | Déploiement | Build prod OK ; déploiement validé ; smoke test post-déploiement `PASS` |

Les phases 6 à 9 peuvent être menées en parallèle une fois la phase 5 terminée (ou par lot de fonctionnalités terminé).

---

## 2. Definition of Done globale (projet)

| Item | Validé par | Preuve attendue |
|---|---|---|
| [ ] Requirements validés | Utilisateur via Marco | REQUIREMENTS.md sans `UNKNOWN` bloquant, validation notée |
| [ ] Content validé | Utilisateur via Lea | CONTENT.md statut `VALIDATED` |
| [ ] UX validée | Marco | STRUCTURE.md couvre tous les FR (matrice de couverture) |
| [ ] Architecture validée | Marco | Décisions structurantes `ACCEPTED` |
| [ ] Code implémenté | Louis → Marco | Toutes les tâches d'implémentation fermées |
| [ ] Responsive validé | Noah | Tests aux breakpoints définis dans STRUCTURE.md |
| [ ] Accessibility vérifiée | Noah (+ Nora) | Audit automatisé + vérification clavier/lecteur d'écran des flows critiques |
| [ ] Performance auditée | Lamine | PERFORMANCE.md : mesures + budgets |
| [ ] Security auditée | Paul | SECURITY.md : checklist + issues |
| [ ] SEO audité | Emile | SEO.md : checklist + issues |
| [ ] Animations validées | Julio | ANIMATIONS.md : inventaire + reduced-motion |
| [ ] Tests passés | Noah | Sortie de la suite de tests, rapport PASS/FAIL/BLOCKED/NOT TESTED |
| [ ] Bugs critiques corrigés | Noah + agent ouvreur | Issues `CRITICAL`/`HIGH` en `VERIFIED` |
| [ ] Documentation mise à jour | Marco | Docs cohérents avec le code livré |
| [ ] Build production réussi | Louis → Noah | Sortie de la commande de build |
| [ ] Deployment validé | Marco + utilisateur | URL de déploiement fournie par l'utilisateur + smoke test |

Des items peuvent être `N/A` (ex. SEO pour une app interne derrière authentification), **uniquement** via une décision `D-NNN` qui le justifie.

---

## 3. Parallélisation

Marco peut paralléliser des tâches si et seulement si :

- elles n'écrivent pas dans les mêmes fichiers ;
- leurs entrées sont déjà validées ;
- le résultat de l'une ne peut pas invalider l'autre.

Parallélisations typiques autorisées :

- Lea (contenu) et Nora (structure) sur un projet dont la liste de pages est déjà `CONFIRMED` ;
- Lamine, Paul, Emile en audit après implémentation ;
- Noah qui écrit des scénarios de test pendant que Louis implémente.

Parallélisations interdites :

- Louis avant validation de l'architecture (phase 4) sur les parties structurantes ;
- Julio avant stabilisation de la structure DOM par Louis.

---

## 4. Definition of Done d'une tâche (T-NNN)

Une tâche est `DONE` quand :

1. les critères `DONE_WHEN` du brief sont satisfaits ;
2. la preuve figure dans le rapport (`EVIDENCE`) ;
3. les documents dont l'agent est owner sont à jour ;
4. aucune nouvelle issue `CRITICAL` n'a été introduite ;
5. la tâche est cochée dans `TODO.md` par Marco (ou par l'agent si Marco l'a délégué).

---

## 5. Règle finale

> Un projet n'est pas terminé parce que le code existe. Il est terminé quand chaque item ci-dessus est prouvé, ou explicitement déclaré `N/A` par une décision tracée.
