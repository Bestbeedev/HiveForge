# Architecture

> Owner: **marco** · Readers: tous
> Status: DRAFT | VALIDATED (date: YYYY-MM-DD)
> Chaque choix structurant renvoie à une décision `D-NNN`.

## 1. Overview

<!-- Schéma textuel : client, serveur, données, services tiers. -->

```text
```

## 2. Stack

| Couche | Technologie | Version | Décision |
|---|---|---|---|
| Framework | | | D- |
| Langage | | | D- |
| Styles | | | D- |
| Données | | | D- |
| Auth | | | D- |
| Tests | | | D- |
| Hébergement | | | D- |

## 3. Rendering & Routing Strategy

| Route / type de page | Stratégie (static / server / client / hybrid) | Revalidation / cache | Justification |
|---|---|---|---|

## 4. Data

- Sources de données :
- Schéma / modèles principaux :
- Accès aux données (où, comment) :
- Validation (schémas, frontières) :
- Migrations :

## 5. Auth & Authorization

- Mécanisme :
- Rôles / permissions :
- Où l'autorisation est vérifiée :

## 6. Client / Server Boundaries

<!-- Qu'est-ce qui s'exécute où ? Quelles données traversent la frontière ? -->

## 7. Folder Structure

```text
```

## 8. Conventions

- Nommage :
- Imports / alias :
- Gestion d'erreurs :
- Logs :
- i18n :

## 9. Approved Dependencies

| Package | Usage | Décision | Alternative écartée |
|---|---|---|---|

## 10. Environments & Configuration

| Variable (nom uniquement) | Usage | Publique ? | Environnements |
|---|---|---|---|

## 11. CI/CD & Deployment

- Pipeline :
- Checks obligatoires (lint, typecheck, tests, build) :
- Déploiement :

## 12. Implementation Plan (lots verticaux)

| Lot | Contenu | FR | Tâches | Status |
|---|---|---|---|---|
