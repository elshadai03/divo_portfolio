# Skillspring Testing & Analytics Plan

## Testing Strategy

### Unit Tests
- **Backend**
  - Validation logic for catalog filters (ensuring safe SQL queries).
  - Pricing helpers (discount calculations, currency formatting).
  - Permissions utilities (`canManageCourse`, `canReviewCourse`).
- **Frontend**
  - Component tests for search bar, filter chips, course card (state changes on hover/focus).
  - Curriculum accordion state management.

### Integration Tests
- Catalog search pipeline:
  - Ensure `/catalog/courses` respects combinations of filters and pagination.
  - Verify caches invalidate when course status changes to `published`.
- Course detail retrieval:
  - `GET /courses/{slug}` + curriculum join returns expected shape.
  - Review submission path: enrollment check → review creation → aggregate update.
- Commerce flow:
  - Add to cart → apply coupon → checkout stub (payment gateway mocked).

### End-to-End (E2E)
- Using Playwright or Cypress with seeded database.
  1. Anonymous user browses catalog, applies filters, views course detail, sees preview lock.
  2. Student login, enrolls, sees full curriculum, leaves review.
  3. Mentor creates course draft, publishes, verifies it appears in catalog.
  4. Responsive check: viewport switches to mobile ensures filter drawer works.

### Accessibility Testing
- Axe automated scans on catalog and course detail pages.
- Manual keyboard navigation review; ensure filter drawer and modals are focus-trapped.

### Performance Testing
- Lighthouse / WebPageTest for page speed budgets:
  - LCP ≤ 2.5s on 3G, 85+ performance score.
  - Monitor CLS when loading thumbnail images.
- Backend load testing (k6): `/catalog/courses` at 200 req/s with 95th percentile latency < 300ms.

### Regression & Release Checks
- Snapshot testing for key components (course card, review list).
- Smoke tests in QA environment triggered on deploy (CI pipeline).

## Quality Gates in CI/CD
- PR checks: lint (ESLint, Stylelint), unit tests (Jest, Vitest), backend tests (pytest or Jest depending on stack).
- Contract tests between frontend and API using Pact to catch schema mismatches.
- Feature flags allow gradual rollout; monitor metrics before full release.

## Analytics Instrumentation

### Tools
- Mixpanel or Amplitude for product analytics.
- Segment as event router.
- GA4 for traffic overview.
- Data warehouse (BigQuery/Snowflake) for long-term storage.

### Key Events
- `catalog_search_performed` (properties: query, filters, result_count).
- `catalog_filter_applied` (filter_type, value).
- `catalog_course_click` (course_id, position, badges).
- `course_detail_viewed` (course_id, student_role, traffic_source).
- `course_preview_played`, `curriculum_section_expanded`.
- `add_to_cart_clicked`, `enroll_now_clicked`, `enrollment_completed`.
- `review_submitted`, `review_helpful_vote`.

### Funnels & KPIs
- Funnel: catalog view → detail view → add to cart → enrollment completed.
- Track drop-off rates by category and device type.
- Monitor mentor performance: publish course → first enrollment → first review.
- Retention metric: % of enrolled students reaching 50% course completion.

### Dashboards
- Catalog health: search volume, filter usage, top categories, conversion rates.
- Course detail engagement: average time on page, preview video completion rates.
- Review insights: sentiment distribution, review response time.

### Alerts & Monitoring
- Real-time alerts if:
  - Catalog conversion drops >10% day-over-day.
  - API error rate >1% for `/catalog/*` or `/courses/*`.
  - Checkout failures exceed defined thresholds.

### Data Governance
- Ensure PII compliant tracking (hash email for analytics).
- Respect user consent (cookie banner for EU, opt-out toggles).
- Retain raw events for 25 months; aggregate for longer-term trending.


