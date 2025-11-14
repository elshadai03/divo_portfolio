# Skillspring Data Model (Catalog & Course Detail)

Relational database assumptions: PostgreSQL 15+, UUID primary keys, snake_case columns, soft-deletes via `deleted_at`.

## Core Entities

### users
- `id UUID PK`
- `email VARCHAR(255) UNIQUE NOT NULL`
- `password_hash TEXT`
- `role ENUM('student','mentor','admin')`
- `status ENUM('pending','active','suspended')`
- `first_name`, `last_name`
- `avatar_url`
- `bio TEXT`
- `timezone`
- Timestamps (`created_at`, `updated_at`, `last_login_at`)

### mentors
- `user_id UUID PK/FK → users.id`
- `headline VARCHAR(180)`
- `about TEXT`
- `experience_years SMALLINT`
- `languages TEXT[]`
- `average_rating NUMERIC(3,2)`
- `total_reviews INTEGER`
- `total_students INTEGER`

### students
- `user_id UUID PK/FK → users.id`
- `occupation VARCHAR(120)`
- `goals TEXT`
- `preferred_categories UUID[]`
- `resume_url`

### courses
- `id UUID PK`
- `mentor_id UUID FK → mentors.user_id`
- `title VARCHAR(140)`
- `subtitle VARCHAR(220)`
- `slug VARCHAR(180) UNIQUE`
- `description TEXT`
- `language_code CHAR(5)`
- `level ENUM('beginner','intermediate','advanced','all')`
- `status ENUM('draft','submitted','published','archived')`
- `list_price_cents INTEGER`
- `sale_price_cents INTEGER NULL`
- `currency CHAR(3)`
- `total_duration_minutes INTEGER`
- `lecture_count INTEGER`
- `last_updated_at TIMESTAMP`
- `hero_video_url`
- `thumbnail_url`
- `is_bestseller BOOLEAN`
- `meta JSONB` (tags, prerequisites)

### course_sections
- `id UUID PK`
- `course_id UUID FK`
- `title VARCHAR(140)`
- `position SMALLINT`
- `total_duration_minutes INTEGER`
- `lecture_count INTEGER`

### lectures
- `id UUID PK`
- `section_id UUID FK`
- `title VARCHAR(160)`
- `position SMALLINT`
- `content_type ENUM('video','article','quiz','assignment','download')`
- `duration_minutes INTEGER`
- `video_url`
- `resource_url`
- `is_preview BOOLEAN`
- `status ENUM('draft','published')`

### categories
- `id UUID PK`
- `parent_id UUID NULL FK -> categories.id`
- `name VARCHAR(80)`
- `slug VARCHAR(80) UNIQUE`
- `level SMALLINT` (0 root, 1 subcategory)
- `description TEXT`

### course_categories (junction)
- `course_id UUID FK`
- `category_id UUID FK`
- PK (`course_id`, `category_id`)

### course_tags
- `id UUID PK`
- `name VARCHAR(40) UNIQUE`

### course_tag_map
- `course_id UUID FK`
- `tag_id UUID FK`
- PK (`course_id`, `tag_id`)

## Commerce & Engagement

### enrollments
- `id UUID PK`
- `course_id UUID FK`
- `student_id UUID FK`
- `purchase_price_cents INTEGER`
- `currency CHAR(3)`
- `purchased_at TIMESTAMP`
- `progress_percent NUMERIC(5,2)`
- `last_watched_lecture UUID FK → lectures.id`
- `completed_at TIMESTAMP NULL`

### carts
- `id UUID PK`
- `student_id UUID FK`
- `status ENUM('open','converted','abandoned')`
- `created_at`, `updated_at`

### cart_items
- `cart_id UUID FK`
- `course_id UUID FK`
- `price_cents INTEGER`
- `currency CHAR(3)`
- `added_at TIMESTAMP`
- PK (`cart_id`, `course_id`)

### wishlists
- `id UUID PK`
- `student_id UUID FK`
- `name VARCHAR(120) DEFAULT 'Favorites'`

### wishlist_items
- `wishlist_id UUID FK`
- `course_id UUID FK`
- `added_at TIMESTAMP`
- PK (`wishlist_id`, `course_id`)

## Reviews & Ratings

### course_reviews
- `id UUID PK`
- `course_id UUID FK`
- `student_id UUID FK`
- `rating SMALLINT CHECK (rating BETWEEN 1 AND 5)`
- `title VARCHAR(140)`
- `body TEXT`
- `would_recommend BOOLEAN`
- `created_at`, `updated_at`
- `is_verified BOOLEAN` (enrolled)
- `helpful_count INTEGER DEFAULT 0`
- `reported_count INTEGER DEFAULT 0`

### review_helpful_votes
- `review_id UUID FK`
- `student_id UUID FK`
- `vote ENUM('helpful','not_helpful')`
- PK (`review_id`, `student_id`)

### course_questions
- `id UUID PK`
- `course_id UUID FK`
- `student_id UUID FK`
- `question TEXT`
- `created_at`
- `resolved_at`

### course_answers
- `id UUID PK`
- `question_id UUID FK`
- `mentor_id UUID FK`
- `answer TEXT`
- `created_at`
- `is_marked_solution BOOLEAN`

## Supporting Tables

### media_assets
- `id UUID PK`
- `owner_id UUID` (mentor or course)
- `file_type ENUM('image','video','document')`
- `url TEXT`
- `duration_seconds INTEGER`
- `size_bytes BIGINT`
- `transcoding_status ENUM('pending','processing','ready','failed')`

### coupons
- `id UUID PK`
- `mentor_id UUID FK` (mentor-specific) NULL for global
- `code VARCHAR(40) UNIQUE`
- `discount_percent SMALLINT`
- `max_redemptions INTEGER`
- `redeemed_count INTEGER`
- `starts_at`, `expires_at`

### course_metrics (daily aggregate)
- `id BIGSERIAL PK`
- `course_id UUID FK`
- `date DATE`
- `views INTEGER`
- `enrollments INTEGER`
- `completions INTEGER`
- `revenue_cents INTEGER`

## Relationships Overview
- `users (1) ↔ (1) mentors/students`: each user is either mentor, student, or both via multiple rows.
- `mentors (1) ↔ (n) courses`.
- `courses (1) ↔ (n) course_sections (1) ↔ (n) lectures`.
- `courses (n) ↔ (n) categories` via `course_categories`.
- `students (n) ↔ (n) courses` via `enrollments`.
- `students (n) ↔ (n) courses` via `course_reviews` (review limited to enrolled students).

## Indexing Guidelines
- Unique indexes on `courses.slug`, `categories.slug`, `course_tags.name`.
- Composite indexes:
  - `course_reviews (course_id, created_at DESC)`
  - `enrollments (student_id, course_id)` unique
  - `courses (status, level, language_code)`
  - `course_categories (category_id, course_id)`

## Data Lifecycle Considerations
- Soft delete courses by setting `status='archived'` and `deleted_at` for compliance.
- Archive lecture media via `media_assets` and background jobs.
- Keep analytics in warehouse (Snowflake/BigQuery) fed by CDC from OLTP tables.


