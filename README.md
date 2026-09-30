# MangaDash — Tailwind CSS (2026)

Dashboard de présentation de données manga, construit **en direct pendant le cours**,
séance après séance. Contrairement à la version précédente (dossier `cours/`), ce
README n'est pas une référence par thème : c'est un **journal de bord** qui suit
l'ordre du cours. Chaque séance ajoute une section ci-dessous, avec l'état du
projet à la fin de la séance.

> **Note** : chaque séance correspond à un tag git (`seance-1` à `seance-7`).
> Le code du dépôt (dans le répertoire de travail actuel) reflète l'état
> **final** (fin de séance 7) ; pour reprendre le fil à un point précis —
> ou pour construire vraiment "en direct" devant les étudiants — utilise
> `git checkout seance-N` (ou `git diff seance-3 seance-4` pour voir
> exactement ce qui change d'une séance à l'autre). Ce README, lui, reste
> identique sur tous les tags : il documente le projet dans son ensemble.

---

## Démarrage

```bash
npm install
npm run dev   # serveur Vite sur http://localhost:5173
```

## Stack

- **Vite 7** + **TypeScript**
- **Tailwind CSS v4**, un seul chemin d'installation : `@tailwindcss/vite`
  (pas de `postcss`/`autoprefixer`/CLI séparé — voir séance d'intro)
- Données statiques : `public/mangas.json` (fetch côté client, à partir de la séance 4)

Un exemple autonome de balises HTML sémantique (indépendant de ce projet, pas de
build nécessaire) se trouve dans [`exemples/html-semantique.html`](exemples/html-semantique.html).

---

## Séance 1 — HTML sémantique

**Objectif** : poser la structure de la page avant tout style. Le projet est
initialisé (Vite + Tailwind installés), mais `index.html` ne porte **aucune
classe Tailwind** pour l'instant.

**Ce qu'on construit** : le squelette de la page d'accueil —
`header` (nom du site + `nav`), `aside` (filtres par catégorie), `main` avec
deux `section` (recherche, résultats), une carte manga par `article`, `time`
avec `datetime`, `footer` avec `address`.

**Pourquoi** : un `<article>` a du sens même sorti de la page (fiche produit
partageable) ; une `<section>` a besoin d'un titre qui l'introduit ; `aside` est
complémentaire, pas indispensable à la compréhension du contenu principal.

**État en fin de séance** : `index.html` sémantique et lisible, mais visuellement
brut (styles par défaut du navigateur). C'est volontaire — le layout arrive séance 2.

---

## Séance 2 — Layout (Flexbox & Grid)

**Objectif** : transformer le squelette sémantique en dashboard, sans encore
se soucier du responsive (on code pour une seule largeur d'écran, celle du
breakpoint le plus large qu'on utilisera).

**Ce qui a été construit** :
```html
<!-- grid [aside | contenu] -->
<div class="grid grid-cols-1 lg:grid-cols-[240px_1fr]">
  <aside class="hidden border-r border-slate-700 p-4 lg:block"> ... </aside>
  <main class="space-y-8 p-4 sm:p-6"> ... </main>
</div>
```
- `header` en `flex items-center justify-between` (nom du site à gauche, nav à droite).
- `aside` : filtres en `flex flex-col gap-1`.
- Grille de résultats (`#cards-grid`) en `grid gap-6` (colonnes fixées ici, responsive en séance 3).
- Cartes (`.manga-card`) en `flex flex-col` : image, titre, description, footer prix aligné avec `justify-between`.

**Ce qu'on couvre en live** : `display: grid` vs `flex` — Grid pour la
macro-disposition (aside/contenu), Flexbox pour la micro-disposition (à
l'intérieur d'une carte, d'un header) — cf. slide "Combiner Flexbox et Grid".

---

## Séance 3 — Responsive multi-écrans

**Objectif** : le layout de la séance 2 s'adapte du mobile à un grand écran,
sur les 5 breakpoints retenus pour le cours (`base`, `md` 768px, `lg` 1024px,
`xl` 1280px, `2k` 2560px).

**Ce qui a été construit** :
```css
/* src/style.css — Tailwind v4 : le breakpoint custom se déclare en CSS,
   pas dans un tailwind.config.js séparé */
@theme {
  --breakpoint-2k: 2560px;
}
```
```html
<div class="grid grid-cols-1 lg:grid-cols-[240px_1fr]">
  <aside class="hidden ... lg:block"> ... </aside>
</div>

<div id="cards-grid" class="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2k:grid-cols-6">
</div>
```
- `aside` masqué sous `lg`, visible à partir de `lg` (1024px).
- Grille de cartes : 1 colonne en base → 2 (`sm`) → 3 (`lg`) → 4 (`xl`) → 6 (`2k`).
- Espacements plus généreux sur très grand écran : `p-4 sm:p-6 2k:p-10`.

**Ce qu'on couvre en live** : mobile-first (les classes sans préfixe sont la
base), vérifier chaque seuil un par un dans les DevTools, jamais casser un
breakpoint plus grand en corrigeant le plus petit.

---

## Séance 4 — Design system & composants

**Objectif** : sortir des classes utilitaires répétées vers des composants
`@apply`, et rendre les cartes dynamiques à partir de `public/mangas.json`.

**Ce qui a été construit** :
- `src/style.css` : composants `.manga-card`, `.manga-card-*`, `.badge`
  + `.badge-shonen/shojo/seinen/josei`, `.nav-link`, `.btn`/`.btn-primary`/`.btn-ghost`,
  `.filter-btn`, `.search-input`.
- `src/main.ts` : `fetch('/mangas.json')`, typage `interface Manga`, fonction
  `renderCard()` qui génère chaque `<article class="manga-card">`, badge de
  catégorie choisi dynamiquement (`badge-${manga.category}`).

```ts
function renderCard(manga: Manga, query: string): string {
  return `
    <article class="manga-card" data-id="${manga.id}">
      <img class="manga-card-cover" src="${manga.cover}" ... />
      <div class="manga-card-body">
        <h2 class="manga-card-title">${highlight(manga.title, query)}</h2>
        <span class="badge badge-${manga.category}">${CATEGORY_LABELS[manga.category]}</span>
        ...
      </div>
    </article>
  `
}
```

**Ce qu'on couvre en live** : quand `@apply` est justifié (`.manga-card` est
répété 20 fois → oui) vs quand il ne l'est pas (cf. slide "Lisibilité >
abstraction prématurée") ; les tokens du design system vivent dans
`src/style.css`, une seule source de vérité.

---

## Séance 5 — Formulaires natifs & données

**Objectif** : rendre la recherche/les filtres fonctionnels, introduire les
widgets interactifs natifs.

**Ce qui a été construit** :
- Filtres de catégorie (`#filters`, délégation d'événement sur `[data-filter]`)
  et recherche (`#recherche`, debounce 200ms) qui appellent `applyFilters()`.
- Surlignage des résultats de recherche avec `<mark>`, injecté dynamiquement
  par `highlight()` dans `main.ts` (et stylé globalement dans `style.css`,
  puisqu'un élément généré en JS ne peut pas recevoir de classe Tailwind à
  l'avance).
- `<details>`/`<summary>` : bloc d'aide sur la page, et caractéristiques
  techniques repliées dans le détail d'un manga.
- `<dialog id="manga-dialog">` natif : clic sur une carte → `showModal()`,
  contenu (description, `<details>`, mini-formulaire d'avis) injecté par
  `openMangaDialog()`.
- Mini-formulaire d'avis dans la dialog : `fieldset`/`legend`, `label`/`for`,
  `input type="email" required`, `input type="range"` relié à un `<output>`.

**Ce qu'on couvre en live** : pourquoi la validation native (`required`,
`type="email"`) réduit le besoin de JS ; `label`/`for` obligatoire sur chaque
champ, y compris ceux injectés dynamiquement.

---

## Séance 6 — Animations CSS modernes

**Objectif** : ajouter des micro-interactions cohérentes et respectueuses de
l'accessibilité.

**Ce qui a été construit** :
```css
/* src/style.css */
@theme {
  --animate-fade-in-up: fadeInUp 0.4s ease-out;
}
@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

.manga-card {
  @apply ... transition-all duration-300
         hover:-translate-y-1 hover:shadow-2xl hover:shadow-slate-900/50
         motion-safe:animate-fade-in-up;
}
```
- Hover cohérent sur les cartes (légère élévation) et les boutons (`.btn-primary`).
- État de chargement : `#loading` utilise `motion-safe:animate-pulse`.
- Apparition des cartes : keyframe personnalisée `fade-in-up`, déclarée comme
  token d'animation (`--animate-fade-in-up`) au même titre qu'une couleur.
- **Accessibilité** : l'animation d'apparition passe par le variant
  `motion-safe:` — elle ne s'exécute pas pour les utilisateurs ayant activé
  "réduire les animations" dans leur système.

---

## Séance 7 — Composants avancés & projet

**Objectif** : finaliser le dashboard, ajouter un composant plus avancé, et
préparer le TP noté.

**Ce qui a été construit** :
- Section statistiques (`.stat-card`) en haut du dashboard : nombre total de
  mangas, note moyenne, nombre de catégories — calculés dans `main.ts`
  (`renderStats()`) à partir des données déjà chargées, pas d'appel réseau
  supplémentaire.

**Pour la suite (à faire en séance ou en TP)** :
- Revue de code collective du dashboard construit depuis la séance 1.
- Cadrage du TP noté — critères attendus :
  - HTML sémantique correct (pas de `div` là où une balise plus précise existe) ;
  - dashboard responsive sur les 5 breakpoints du cours ;
  - au moins un composant `@apply` justifié (pas systématique) ;
  - au moins un widget interactif natif (`details`, `dialog`, `progress`...) ;
  - au moins une animation qui respecte `prefers-reduced-motion`.

---

## Pour aller plus loin

Voir `cours/README.md` (version précédente, Tailwind + SASS) pour une référence
*par thème* des classes utilitaires si besoin d'un lookup rapide — ce README-ci
reste volontairement un journal de progression, pas une doc exhaustive.
