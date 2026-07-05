# UI Audit — Consaltia (avant refonte visuelle)

Portée : `index.html` (page unique, CSS/JS inline). Audit réalisé avant la passe visuelle ;
chaque constat indique le correctif appliqué dans cette PR. Palette bleu/blanc inchangée.

---

## 1. Responsiveness

### Mobile (< 576px)
| # | Constat | Correctif appliqué |
|---|---------|--------------------|
| M1 | Le titre hero (2rem fixe) et le titre de section (1.6rem fixe) ne s'adaptent pas de façon fluide entre les breakpoints — sauts brusques à 768px. | Typographie fluide via `clamp()` sur H1, H2, sous-titres — plus aucun saut de taille. |
| M2 | Bouton burger ≈ 41×39px — sous le minimum tactile de 44px. | `min-width/min-height: 44px` sur `.nav-toggle`. |
| M3 | La strip de stats du hero passe en pile verticale sans hiérarchie (séparateurs `·` supprimés mais pas d'espacement compensatoire). | Espacement vertical dédié + chiffres mis en avant (compteur animé). |
| M4 | `background-attachment: fixed` (desktop ≥ 992px) : jank de scroll connu, et repaints coûteux. | Supprimé — attachement `scroll` partout, l'overlay animé prend le relais visuel. |

### Tablette (768–991px)
| # | Constat | Correctif appliqué |
|---|---------|--------------------|
| T1 | Grilles services (2 col.) et valeurs (2 col.) OK, mais cartes d'inégale hauteur sans alignement du contenu. | Cartes en flex column, footer de carte aligné, hauteurs homogènes. |
| T2 | Timeline : la colonne logo 56px + gap 2rem laisse une ligne de lecture > 75ch sur tablette large. | `max-width` de la carte + rythme vertical resserré. |

### Desktop / grand écran (≥ 1200px)
| # | Constat | Correctif appliqué |
|---|---------|--------------------|
| D1 | Le hero `min-height: 600px` fixe crée un bandeau écrasé sur écrans hauts. | `min-height: min(88vh, 780px)` — proportionnel, sans CLS. |
| D2 | Deux animations infinies simultanées dans le hero (grille en `scale` + halo en `rotate` 200%×200%) — coût GPU permanent et visuellement concurrentes. | Remplacées par un seul balayage de dégradé bleu très lent (opacity/transform uniquement), coupé si `prefers-reduced-motion`. |

## 2. Accessibilité
| # | Constat | Correctif appliqué |
|---|---------|--------------------|
| A1 | Aucun état `:focus-visible` global — navigation clavier invisible sur liens, boutons, burger. | Anneau de focus global (bleu de la palette, `outline` 3px + offset) sur tous les éléments interactifs. |
| A2 | Overlay hero à alpha 0.5 sur photo : le blanc sur zones claires de l'image peut passer sous AA (4.5:1). | Overlay renforcé (mêmes bleus, alpha ≈ 0.72/0.62) — contraste garanti quel que soit le fond. |
| A3 | Burger sans `aria-expanded` / `aria-controls` — état du menu inconnu des lecteurs d'écran. | Attributs ajoutés + mise à jour JS à chaque toggle. |
| A4 | Pas de lien d'évitement ni de landmark `<main>`. | Skip-link « Aller au contenu » + `<main id="main">` englobant les sections. |
| A5 | Liens email pointant vers `/cdn-cgi/l/email-protection` (script Cloudflare absent sur GitHub Pages) → **liens cassés**. Le texte de l'email est déjà en clair dans la source, l'obfuscation ne protégeait rien. | `href="mailto:consaltia@gmail.com"` réels ; script `email-decode.min.js` (404) retiré. |
| A6 | Ancres de sections masquées sous la navbar fixe lors de la navigation clavier (le JS compense seulement au clic). | `scroll-margin-top` sur toutes les sections. |
| A7 | Icônes Font Awesome décoratives non masquées aux lecteurs d'écran. | `aria-hidden="true"` via sélecteur existant conservé pour les icônes sémantiques ; les nouvelles icônes décoratives restent dans des éléments déjà labellisés. |
| A8 | Ordre des titres globalement correct (h1→h2→h3) ; footer en h4 après h3 — acceptable, conservé. | Aucun changement (constat documenté). |

## 3. Hiérarchie visuelle
| # | Constat | Correctif appliqué |
|---|---------|--------------------|
| H1 | Montserrat + Open Sans : deux familles proches sans contraste de caractère ; titres sans tracking négatif → rendu daté. | Passage à **Inter** (une seule famille, poids 400→800), tracking resserré sur les titres, hiérarchie par la graisse et l'échelle. |
| H2 | Tous les blocs (cartes, formulaires, timeline) utilisent la même élévation `shadow-sm` — aucune profondeur relative. | Système d'élévation à 3 niveaux : bordure subtile au repos → ombre douce au survol → ombre marquée pour l'élément actif. Ombres dérivées du bleu primaire uniquement. |
| H3 | Le hero n'a pas de point focal : titre, sous-titre, CTA et stats arrivent tous avec la même intensité. | Entrées échelonnées (titre → sous-titre → CTA → stats, délais 0/120/240/360ms) + compteur animé sur les chiffres. |
| H4 | Icône de valeur en rotation 360° au survol — gadget, sans lien avec le contenu. | Remplacé par un scale subtil (1.08) + transition de couleur. |

## 4. Espacements & typographie
| # | Constat | Correctif appliqué |
|---|---------|--------------------|
| S1 | Échelle d'espacement déclarée (`--spacing-*`) mais contournée par de nombreuses valeurs en dur (0.35rem, 0.6rem, 0.875rem…). | Rythme vertical harmonisé sur l'échelle existante ; valeurs orphelines rattachées aux tokens. |
| S2 | `line-height: 1.6` global un peu serré pour les paragraphes longs (timeline, à-propos). | 1.65 pour le corps de texte, 1.15–1.3 pour les titres. |
| S3 | Pas d'échelle typographique formalisée. | Échelle fluide documentée dans `:root` (`--fs-h1` → `--fs-sm`, toutes en `clamp()`). |

## 5. Performance
| # | Constat | Correctif appliqué |
|---|---------|--------------------|
| P1 | ~370 lignes de CSS mort (sections avis / carrousel clients / grille portfolio supprimées précédemment) : `.avis-*`, `.clients-carousel`, `.realisations-grid`, `.realisation-card`, `.tag`, keyframes `scroll`/`shimmer`/`float`. **0 usage** vérifié par grep. | Supprimées (~9KB de CSS en moins). |
| P2 | Script Cloudflare `email-decode.min.js` → 404 systématique sur GitHub Pages (requête bloquante inutile). | Retiré (cf. A5). |
| P3 | Deux familles Google Fonts (7 graisses au total) chargées. | Une seule famille (Inter), `display=swap` conservé — moins d'octets, moins de CLS de police. |
| P4 | Animations existantes déjà en transform/opacity ✔ mais deux boucles infinies dans le hero (cf. D2). | Une seule boucle lente, GPU-friendly, désactivée en reduced-motion. |
| P5 | `personalpic.png` = 467KB pour un rendu 160×160. | `loading="lazy" decoding="async"` ajoutés. <!-- TODO: compresser/convertir personalpic.png (WebP ~30KB suffirait pour 160px) --> |
| P6 | Compteurs et reveals : IntersectionObserver natif, **aucune librairie ajoutée** — la stack (HTML statique) ne justifie ni GSAP ni Framer Motion. | Confirmé — zéro dépendance nouvelle. |

## 6. Motion — règles adoptées
- Durées 150–400ms, easing `cubic-bezier(.22,.61,.36,1)` (sortie douce).
- Uniquement `transform` + `opacity` (compositor-friendly).
- Reveal : fade + translateY(16px), stagger 60ms au sein d'un même viewport.
- Compteur stats : ~900ms, ease-out cubique, déclenché à 60% de visibilité, une seule fois.
- `prefers-reduced-motion: reduce` → toutes les animations non essentielles coupées, contenus affichés dans leur état final (y compris compteurs).
