# Skillspring Course Detail UX

## Goals
- Showcase course value (outcomes, instructor credibility, social proof) upfront.
- Drive conversion via `Enroll now` and `Add to cart` CTAs.
- Support mentor-managed curriculum, downloadable resources, and student reviews similar to Udemy.

## Above the Fold
- **Layout**: two-column design (`main` content left 2/3, sticky purchase card right 1/3).
- **Breadcrumb**: `Home / Development / Web Development / React` using `.navbar__links` typography.
- **Title Block**:
  - Course title (`h1`).
  - Subtitle summarizing outcomes (single paragraph).
  - Meta row: star rating + count, student count, last updated date, language, captions.
- **Instructor Line**: `Created by <mentor name>` with avatar chip linking to mentor profile.
- **Action Card** (sticky):
  - Hero video thumbnail with play button overlay.
  - Price (current, original with strikethrough).
  - CTA buttons: `Enroll now` (`btn--primary`), `Add to cart` (`btn--outline`), `Wishlist`.
  - Guarantee text: `30-day refund guarantee`.
  - Share icons.

## What You'll Learn
- Grid of 4–8 bullet points using `.card` with two-column layout on desktop and stacked on mobile.
- Each bullet limited to ~90 characters for scannability.

## Course Includes
- Chip list summarizing features (hours of video, articles, downloadable resources, assignments, certificates).
- Lives inside a `.card` with icon left, label right.

## Curriculum Section
- Accordion component styled with `.card` containers.
- Each section shows:
  - Section title, total lectures, total duration.
  - Collapsible lecture list: lecture title, duration, preview availability.
- `Expand all` / `Collapse all` controls.
- Mobile: default collapsed; desktop: first section expanded.

## Requirements
- Short bullet list clarifying prerequisites. Use `.card` with subtle background.

## Description
- Rich text block (paragraphs, subheadings, lists). Use `max-width: 720px` for readability.
- Mentors can provide markdown; convert to HTML on render.

## Instructor Bio
- Panel containing mentor avatar, name, headline, rating, number of reviews, students, courses.
- Include `Follow mentor` button (future enhancement).
- Overview paragraphs, achievements, social links.

## Student Feedback
- Summary bar: average rating, rating distribution bars, total reviews.
- Review list cards with:
  - Student avatar initials.
  - Name, rating, purchase date.
  - Review text (truncated to 4 lines with `Read more` toggle).
  - `Was this helpful?` buttons (`Yes`, `No`).
- Sorting: `Most relevant`, `Most recent`, `Highest rating`, `Lowest rating`.

## Related Courses
- Horizontal carousel (Snap scroll on mobile) featuring 4 recommended courses (reuse catalog card).
- Title `Students also bought`.

## Sidebar (Desktop Sticky)
- CTA card described above.
- Additional boxes: `Includes`, `Share`, `Apply coupon`.
- Show `Team access` CTA for group licensing.

## Mobile Adaptations
- Purchase CTA converts to bottom sticky bar with price + `Enroll now`.
- Action card collapses into accordion above curriculum.
- `Jump to section` dropdown to aid navigation.

## Interactions & States
- Video preview opens modal using existing `.modal` component.
- Curriculum preview lectures use free tag and allow quick play.
- Reviews filter updates list in place with skeleton loading.
- `Add to cart` triggers toast notification using `position: fixed; bottom: 24px`.

## Content Governance
- Fields validated for max length (title 120 chars, subtitle 200 chars).
- Prevent profanity via backend service.
- Mentor can mark sections as draft; students only see published lectures.


