# HiveForge — Prompts d'orchestration

> Stack de ce projet : **react**. Prompts spécifiques à la stack : `hiveforge/stack.md §9`.
>
> **Convention** : `@nom` = « Charge `hiveforge/agents/<nom>.agent.md`, adopte ce rôle et respecte `hiveforge/protocols/` ».
>
> **Le plus simple** : dans le chat de l'IDE (Antigravity, Cursor, VS Code, Claude Code), tape `@` puis choisis le fichier de l'agent,
> ex. `@marco.agent.md Initialise le projet.` — l'agent est chargé directement.
> Sinon, préfixe le prompt par : `Lis hiveforge/agents/<nom>.agent.md et agis strictement en tant que cet agent.`
>
> Un prompt = une session d'agent. Enchaîner plusieurs agents dans un même message est possible ; chaque agent rend alors son propre rapport, dans l'ordre.

---

## 1. INITIALISATION

```text
@marco Initialise le projet.

Lis hiveforge/config.md, hiveforge/stack.md et hiveforge/docs/PROJECT.md, puis inspecte le code réel.
Produis le bloc CONTEXT DETECTION.

Identifie :
- les objectifs ;
- les utilisateurs ;
- les fonctionnalités ;
- les contraintes ;
- les inconnues ;
- les risques.

Crée ou mets à jour hiveforge/docs/REQUIREMENTS.md, hiveforge/docs/DECISIONS.md et hiveforge/docs/TODO.md.
Mets à jour Current Phase et Active Agents dans hiveforge/config.md.
Pose-moi uniquement les questions bloquantes.

Ne code rien pour le moment.
```

---

## 2. DISCOVERY

```text
@lea Analyse le projet et interviewe-moi uniquement sur les informations nécessaires
pour produire hiveforge/docs/CONTENT.md. N'invente aucun fait.
```

```text
@nora À partir de PROJECT.md, REQUIREMENTS.md et CONTENT.md, définis hiveforge/docs/STRUCTURE.md
(pages, sections, composants, navigation, flows, responsive, états UI) avec la matrice de couverture des FR.
```

```text
@nora À partir de STRUCTURE.md et des sources de design disponibles
(<charte / Figma / Stitch / aucune>), produis hiveforge/docs/DESIGN.md avec contrastes vérifiés.
```

```text
@marco Relis les documents produits par Lea et Nora, détecte les contradictions et les trous,
consulte Lamine (budgets) et Paul (modèle de menace), puis rédige hiveforge/docs/ARCHITECTURE.md
et passe les décisions structurantes en ACCEPTED. Découpe l'implémentation en lots verticaux dans TODO.md.
Ne code rien.
```

---

## 3. BUILD

```text
@marco Le contexte et l'architecture sont maintenant validés.
Coordonne l'implémentation : attribue le premier lot à Louis avec un brief TASK complet.
```

```text
@louis Implémente la tâche <T-NNN> conformément à hiveforge/docs/ARCHITECTURE.md,
hiveforge/docs/STRUCTURE.md et hiveforge/docs/DECISIONS.md.
Ne modifie aucune exigence fonctionnelle sans validation de Marco.
Rends ton rapport avec les preuves (build, lint, types).
```

```text
@noah Pendant que Louis implémente, rédige le plan de test dans hiveforge/docs/TESTING.md
(matrice FR → scénarios) et prépare les tests des flows critiques.
```

---

## 4. AUDIT FINAL

```text
@lamine Audite les performances du projet sur un build de production. Mets à jour PERFORMANCE.md.

@paul Audite la sécurité. Mets à jour SECURITY.md.

@emile Audite le SEO. Mets à jour SEO.md.

@julio Audite les animations et l'accessibilité liée au motion. Mets à jour ANIMATIONS.md.

@noah Exécute les tests et identifie les régressions. Mets à jour TESTING.md avec les verdicts.

@marco Consolide tous les rapports, arbitre les problèmes et coordonne les corrections
jusqu'à la Definition of Done.
```

---

## 5. BOUCLE DE CORRECTION

```text
@marco Liste les issues CRITICAL et HIGH ouvertes dans tous les documents d'audit,
dédupliquées et priorisées, et crée les tâches de correction.
```

```text
@louis Corrige les issues <ID, ID, ...> selon les recommandations. Une issue = un changement ciblé.
Rends le rapport avec preuves.
```

```text
@<agent ouvreur> Vérifie les corrections des issues <ID, ...> et passe-les en VERIFIED ou rouvre-les.

@noah Relance la suite de tests et les tests de régression liés.
```

---

## 6. LIVRAISON

```text
@marco Vérifie la Definition of Done complète avec preuves (hiveforge/protocols/definition-of-done.md).
Présente-moi : checklist, dérogations, risques résiduels, recommandation GO/NO-GO.
Ne déploie rien sans mon accord explicite.
```

```text
@noah Exécute le smoke test sur l'URL de déploiement <URL fournie> et rends le verdict.
```

---

## 7. PROMPTS DE MAINTENANCE

### Reprendre une session

```text
@marco Reprends le projet : lis config.md, TODO.md et DECISIONS.md, inspecte le code,
et dis-moi où nous en sommes et quelle est la prochaine action.
```

### Nouvelle fonctionnalité

```text
@marco Nouvelle demande : <description>.
Évalue l'impact sur REQUIREMENTS, STRUCTURE, ARCHITECTURE ; ajoute les FR, décisions et tâches nécessaires ;
indique quels agents doivent intervenir. Ne code rien avant validation.
```

### Bug signalé

```text
@noah Reproduis ce bug : <description, étapes, attendu, obtenu>.
Ouvre une issue QA-NNN avec la sévérité et écris un test qui échoue.
Puis @marco assigne la correction.
```

### Changement de décision

```text
@marco Je souhaite revenir sur <D-NNN> : <raison>.
Évalue l'impact, crée la décision de remplacement, liste le travail à reprendre.
```

### Audit ciblé

```text
@<lamine|paul|emile|julio|noah> Audite uniquement <page / fonctionnalité / fichiers>.
```
