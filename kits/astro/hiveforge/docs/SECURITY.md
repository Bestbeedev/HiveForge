# Security

> Owner: **paul** · Readers: marco, louis, noah
> Ne jamais écrire de secret en clair dans ce fichier.

## 1. Threat Model

### Assets
- 

### Actors
- 

### Attack Surface

| Point d'entrée | Type (form, API, upload, webhook, auth…) | Auth | Données sensibles |
|---|---|---|---|

### Trust Boundaries
- 

## 2. Security Requirements

- Sessions / cookies :
- Headers :
- CORS :
- Rate limiting :
- Validation des entrées :
- Uploads :
- Secrets :
- Journalisation :

## 3. Audit Checklist

| Contrôle | Status (OK / KO / N/A) | Preuve |
|---|---|---|
| Injection (SQL/NoSQL/commandes) | | |
| XSS | | |
| CSRF | | |
| SSRF | | |
| Authentification | | |
| Autorisation / IDOR | | |
| Mass assignment | | |
| Sessions | | |
| Uploads | | |
| CORS | | |
| CSP & headers | | |
| Rate limiting | | |
| Secrets | | |
| Dépendances | | |
| Exposition d'informations (debug, source maps, erreurs) | | |

## 4. Issues

<!--
### SEC-001 [SEVERITY] Titre
- Location:
- Evidence:
- Risk:
- Impact:
- Likelihood:
- Mitigation:
- Residual Risk:
- Owner:
- Status:
-->

## 5. Accepted Residual Risks

| Risque | Décision | Accepté par | Date |
|---|---|---|---|
