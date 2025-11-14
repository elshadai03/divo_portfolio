# Skillspring API Design (Catalog & Course Detail)

Base URL: `/api/v1`. JSON responses, snake_case keys, RFC 7807 error objects.

## Authentication & Headers
- Auth via JWT access token (students, mentors, admins). Optional browsing for anonymous users.
- Required headers:
  - `Authorization: Bearer <token>` (for protected routes)
  - `X-User-Role` derived server-side, never trust client-provided role.
  - `Accept-Language` influences localized strings & subtitles.

## Catalog Endpoints

### GET `/catalog/courses`
Returns paginated list for search/browse.

**Query Params**
- `q` (string, optional)
- `category` (UUID or slug, multi-value)
- `level` (`beginner|intermediate|advanced|all`)
- `language` (ISO code)
- `price` (`free|paid`)
- `duration` (`short|medium|long`)
- `features` (multi-value: `certificate`, `assignments`, `downloads`)
- `rating_min` (float)
- `sort` (`popular`, `rating`, `newest`, `price_asc`, `price_desc`)
- `page`, `page_size` (default 20, max 50)

**Response**
```json
{
  "data": [
    {
      "id": "uuid",
      "slug": "python-for-beginners",
      "title": "Python for Beginners",
      "subtitle": "Build foundational coding skills.",
      "thumbnail_url": "...",
      "mentor": { "id": "uuid", "name": "Jane Doe", "rating": 4.7 },
      "badges": ["bestseller"],
      "rating": 4.8,
      "review_count": 1290,
      "student_count": 54000,
      "list_price": { "amount": 199, "currency": "USD" },
      "sale_price": { "amount": 19.99, "currency": "USD", "discount_percent": 90 },
      "duration_minutes": 520,
      "level": "beginner",
      "tags": ["python", "programming"]
    }
  ],
  "meta": { "page": 1, "page_size": 20, "total": 354 }
}
```

### GET `/catalog/categories`
- Returns full category tree (cacheable, `Cache-Control: max-age=3600`).

### GET `/catalog/featured`
- Curated sections: trending, new & noteworthy, top-rated per category.
- Input: optional `student_id` to personalize (requires auth).

## Course Detail Endpoints

### GET `/courses/{slug}`
- Public course payload used across detail page.
- Includes: core course metadata, mentor summary, stats, pricing, `what_you_will_learn`, `includes`, `requirements`.
- For authenticated students, include `enrollment_status` (`none|in_cart|enrolled|completed`).

### GET `/courses/{slug}/curriculum`
- Returns sections/lectures grouped. Previews flagged.
- Supports `?preview=true` to omit locked content for anonymous viewers.

### GET `/courses/{slug}/reviews`
- Query params: `page`, `page_size` (default 10), `sort`, `rating`, `with_text`.
- Response includes aggregation snippet (avg, distribution).

### POST `/courses/{course_id}/reviews` (student)
- Requires student enrollment validation.
- Payload: `rating`, `title`, `body`, `would_recommend`.
- Triggers mentor notification + recalculates aggregates.

### PATCH `/courses/{course_id}/reviews/{review_id}` (student)
- Edit window limited (e.g., 30 days). Audit tracked.

### POST `/reviews/{review_id}/helpful` (student)
- Body: `{ "vote": "helpful" | "not_helpful" }`.

### GET `/courses/{slug}/related`
- Returns recommended courses (collaborative filtering + manual curation).

## Enrollment & Commerce

### POST `/cart/items`
- Authenticated students only.
- Body: `{ "course_id": "uuid", "price_cents": 1999, "currency": "USD", "coupon_code": "SKILL20?" }`.
- Validates price against current promotions.

### DELETE `/cart/items/{course_id}`
- Removes course from active cart.

### POST `/checkout`
- Creates order, processes payment (integrates with Stripe/PayPal). Out of scope for this doc but referenced by detail CTA.

### GET `/enrollments/{course_id}`
- Returns enrollment progress for authenticated student (used to show `Resume learning` CTA).

## Mentor Management (for completeness)

### POST `/mentor/courses`
- Create draft course. Body includes title, subtitle, category assignments.

### PATCH `/mentor/courses/{course_id}`
- Update metadata, pricing, status transitions.

### POST `/mentor/courses/{course_id}/sections`
- Manage curriculum; similar endpoints for lectures (`PUT`, `DELETE`).

## Response Conventions
- Monetary values represented both in cents + formatted string.
- Durations in minutes; client formats to `hh:mm`.
- Include `links` array for HATEOAS (self, enroll, mentor_profile).

## Caching Strategy
- CDN cache (Fastly) for catalog listings with user-independent filters.
- Use `ETag`/`If-None-Match` for course detail; bust cache on publish.
- Private caching for student-specific data (`Cache-Control: private, max-age=60`).

## Rate Limits
- Public browse endpoints: 60 req/min IP-based.
- Authenticated actions (reviews, cart): 30 req/min per user to prevent spam.

## Observability
- Emit structured logs with trace IDs.
- Metrics: `catalog_search_latency`, `course_detail_cache_hit`, `review_post_success`.
- Integrate with feature flags service for experimental sort algorithms.


