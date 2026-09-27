# PROTOCOL — CONFLICT RESOLUTION

> Portée : tous les agents. Un conflit est une situation où deux agents, deux documents, ou un document et le code, exigent des choses incompatibles.

---

## 1. Flux

```text
Agent détecte le conflit
   ↓
STOP sur la partie concernée (continuer le reste si indépendant)
   ↓
Rapport STATUS: BLOCKED ou NEEDS_REVIEW avec bloc CONFLICT
   ↓
Marco arbitre (consulte les agents concernés si nécessaire)
   ↓
Décision D-NNN dans hiveforge/docs/DECISIONS.md
   ↓
Tâches de correction dans hiveforge/docs/TODO.md
   ↓
Agents concernés appliquent la décision finale
```

Aucun agent ne « gagne » un conflit en modifiant le fichier d'un autre.

---

## 2. Bloc CONFLICT

```text
CONFLICT:
- Between: <agent A / doc A> vs <agent B / doc B / code>
- Subject: <objet précis>
- Position A: <exigence + source>
- Position B: <exigence + source>
- Impact if unresolved: <conséquence>
- Suggested resolution: <proposition de l'agent qui signale>
```

---

## 3. Critères d'arbitrage de Marco

Marco arbitre selon cet ordre de priorité, sauf contrainte projet contraire documentée :

```text
1. Sécurité et obligations légales (données personnelles, accessibilité légale)
2. Exigences fonctionnelles CONFIRMED
3. Intégrité des données
4. Accessibilité
5. Performance (budgets de PERFORMANCE.md)
6. SEO
7. Maintenabilité / simplicité
8. Esthétique
9. Vitesse de livraison
```

Si l'arbitrage implique un compromis **business** (fonctionnalité retirée, délai, coût, dégradation visible par l'utilisateur final), Marco ne tranche pas seul : il pose une question `Q-NNN` à l'utilisateur avec une recommandation.

---

## 4. Conflits types et résolution par défaut

| Conflit | Résolution par défaut |
|---|---|
| Louis veut une librairie, Lamine signale le poids | Anti-bloat check chiffré ; natif ou code local si gain < coût |
| Julio veut une animation, Lamine signale un impact INP/CLS | Animation limitée à `transform`/`opacity`, ou supprimée |
| Julio veut une animation, Nora/accessibilité s'y oppose | `prefers-reduced-motion` obligatoire ; si l'animation porte du sens, alternative statique |
| Emile veut plus de texte, Lea/Nora veulent de la concision | Lea rédige ; Emile fournit mots-clés et intentions ; pas de texte écrit pour les robots |
| Paul exige une contrainte qui dégrade l'UX | Sécurité prioritaire ; Nora propose une UX compatible |
| Noah trouve un test qui contredit le comportement implémenté | Vérifier REQUIREMENTS.md : si l'exigence est claire, Louis corrige ; sinon Marco décide et documente |
| Le code contredit DECISIONS.md | Issue `ARCH-` ; Marco décide : corriger le code ou superseder la décision |
| Deux documents se contredisent | Le document dont l'agent est owner du domaine prévaut ; Marco confirme |

---

## 5. Anti-boucle

Si un même sujet a fait l'objet de **deux** allers-retours sans convergence :

1. Marco fige l'option la plus simple et la plus réversible ;
2. il documente la décision avec `Revisit when:` ;
3. le sujet est clos jusqu'à ce que la condition de réouverture soit remplie.

---

## 6. Interdits

- Modifier silencieusement un fichier dont on n'est pas owner.
- Contourner une décision `ACCEPTED` « temporairement ».
- Désactiver un test, une règle de lint, un header de sécurité ou un budget pour débloquer une situation sans décision `D-NNN`.
- Présenter un choix personnel comme une exigence utilisateur.
