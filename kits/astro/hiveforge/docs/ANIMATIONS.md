# Animations

> Owner: **julio** · Readers: nora, louis, lamine, noah
> Priorité : UX → Performance → Accessibility → Aesthetics

## 1. Principles for this project

- Technologie(s) autorisée(s) : <CSS / WAAPI / GSAP / Motion — voir D-NNN>
- Tokens de motion : voir DESIGN.md §6
- Reduced motion : stratégie globale

## 2. Inventory

| ID | Élément / composant | Fonction UX | Déclencheur | Propriétés | Durée / easing | Technique | Reduced motion | Status |
|---|---|---|---|---|---|---|---|---|
| ANIM-A01 | | feedback | hover | transform, opacity | 150 ms / ease-out | CSS | fondu seul | |

## 3. Verification

| Contrôle | Status | Preuve |
|---|---|---|
| prefers-reduced-motion respecté | | |
| Aucun CLS introduit | | |
| Uniquement transform/opacity sur animations fréquentes | | |
| Contenu accessible sans JS d'animation | | |
| Focus jamais masqué | | |
| Test CPU throttling 4× | | |

## 4. Issues

<!-- ### ANIM-001 [SEVERITY] Titre / Location / Evidence / Impact / Recommendation / Owner / Status -->
