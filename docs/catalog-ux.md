# Skillspring Course Catalog UX

## Objectives
- Present a comprehensive, Udemy-style browsing experience for students.
- Reuse existing design language from `assets/css/style.css` (Inter typography, rounded cards, glassmorphism navbar) to keep visual consistency.
- Ensure full responsiveness across mobile (≤768px), tablet, and desktop breakpoints.

## Page Entry (Student Navigation)
- Extend the existing `.navbar` to include new links: `Home`, `Catalog`, `My Learning`, `Mentors`, `Profile`.
- Replace CTA buttons with role-aware actions:
  - `Log in` → opens role selector (Mentor / Student) modal from existing component styles.
  - `Start free trial` → `Become a mentor` button for mentors, `Join Skillspring` for students.

## Hero + Search Strip
- **Layout**: two-column hero with headline, supporting copy, and primary catalog search.
- **Search Bar**: full-width input grouped with dropdowns.
  - Input `What do you want to learn?` (`.input-group` with icon left, `button.btn--primary` right).
  - Dropdown filters: `All Categories`, `Skill Level`, `Language`.
- **Key Metrics**: small stats row (students, mentors, courses) below search to establish credibility.

## Filter Panel
- **Desktop**: left column filter rail (`aside.catalog__filters`) with stacked cards using `.card` styles.
- Filters include:
  - Categories (checkbox tree)
  - Rating (4.5+, 4+, 3.5+, etc.)
  - Skill level (Beginner, Intermediate, Advanced)
  - Price (Free, Paid)
  - Duration (≤2h, 2–10h, 10h+)
  - Features (Certificate, Assignments, Downloadable resources)
- **Mobile**: filter drawer triggered by button `Filter` with badge count, overlay modal using existing `.modal` foundation.

## Results Grid
- Use responsive CSS Grid container: `.catalog__results` with `grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));` and `gap: 32px`.
- Card anatomy (extends `.card`):
  - Header image (16:9, `.card__thumb`)
  - Course title (2-line clamp)
  - Mentor name + badge (mentor rating)
  - Meta row: rating stars, total reviews, student count.
  - Price block showing current price, struck-through original price if discounted.
  - Action buttons: `Add to cart`, `Wishlist` (icon button).
- Include quick-tag chips for `Bestseller`, `New`, `Top rated` using existing badge styling (`.badge`, `.badge--accent`).

## Result Sorting & Pagination
- Top bar above grid with:
  - `xx results` label.
  - Sort dropdown (`Most popular`, `Highest rated`, `Newest`, `Price: low to high`, etc.).
- Pagination at bottom using `.btn--ghost` styling for page numbers and `btn--primary` for current page.
- Infinite scroll option flagged for future enhancement (track in backlog).

## Empty State
- Illustrate with icon and message `No courses match your filters yet.`
- Provide CTA `Reset filters` and recommendations (Top categories).

## Responsive Behavior
- ≤1024px: filters collapse into top horizontal scroll, cards stack 2 per row.
- ≤768px: hero stacks vertical, filters accessible via modal drawer, cards single column with condensed meta row.
- Use existing breakpoints from `style.css` media queries (`@media (max-width: 960px)` etc.).

## Accessibility & Interactions
- All interactive controls follow focus styles from `.btn` and `.navbar__links`.
- Filters use `fieldset` with `legend` for screen readers; ensure `aria-expanded` on collapsible sections.
- Search and filter changes trigger skeleton loaders on results for perceived performance.

## Assets & Supporting Components
- New icons to add under `assets/img/catalog/`:
  - Search icon (`search.svg`), star (`star.svg`), filter (`filter.svg`), bookmark (`bookmark.svg`).
- Reuse `btn`, `card`, `badge` classes; add `catalog__filters`, `catalog__results`, `catalog__card`, `label--chip`.

## Open Questions / Follow-ups
- Determine whether mentor logos/photos exist; placeholder uses initials with gradient background.
- Confirm if cart functionality is in scope now or later (currently stub for future e-commerce additions).


