# Skillspring Auth & Role Strategy

## Roles & Permissions
- **Student**
  - Browse catalog (anonymous or authenticated).
  - Purchase/enroll in courses, access purchased content, leave reviews, track progress.
- **Mentor**
  - Manage own courses (create, edit, publish, archive), view enrolled students, respond to Q&A.
  - Access revenue dashboards for authored courses.
- **Admin**
  - Moderate content, handle escalated reviews, feature courses, manage categories.

## Authentication Flow
1. Login modal presents role tabs (Student / Mentor). Both reuse same `users` table; role association derived by account flags.
2. Users authenticate via:
   - Email/password (stored as `password_hash` + salted bcrypt).
   - Optional OAuth (Google, LinkedIn) for future.
3. On success:
   - Issue short-lived JWT (15 min) + refresh token (30 days) stored HttpOnly cookie.
   - Embed `sub` (user_id), `role`, `permissions` array minimal, `exp`.
4. Refresh endpoint rotates tokens; revoke on password change or manual logout.

## Authorization Layers
- **API Gateways**
  - Middleware validates access token, populates `req.user`.
  - Enforce role checks via policy definitions (e.g., `canManageCourse(user, courseId)`).
- **Resource Ownership**
  - Mentors can only mutate courses where `mentor_id == user.id`.
  - Admin overrides with `role == admin`.
- **Feature Flags**
  - Use LaunchDarkly (or similar) to gate early releases per role (e.g., mentor analytics beta).

## Front-End Guarding
- Use route-level guards in SPA (e.g., React Router):
  - `/mentor/**` requires mentor role; show upgrade CTA for students.
  - `/student/my-learning` requires student login; if anonymous, redirect to role selection.
- Persist minimal profile data within `localStorage` (role, display name) for quick UI decisions; always verify on API call.

## Role Switching
- Allow single user to hold both student & mentor roles.
- `users.role` stores highest privilege; maintain `user_roles` join table to support multi-role:
  - On login, prompt user to choose active role.
  - Issue JWT with `active_role` claim; allow switching via `/auth/switch-role`.

## Course Access Control
- Curriculum endpoint gates lecture content:
  - Unauthenticated: only preview lectures.
  - Enrolled student: full content.
  - Mentor owner: full content, including unpublished sections (marked draft).
- Media URLs signed via CDN (expiring URLs) to prevent unauthorized downloads.

## Review Integrity
- `POST /courses/{course}/reviews` checks `enrollments.enrolled_at NOT NULL` and `status == completed OR progress >= 10%`.
- Enable single review per course per student; allow edits within 30 days.
- Admin moderation queue for reported reviews.

## Security Enhancements
- Require email verification before publishing courses or posting reviews.
- Enforce 2FA optional for mentors (TOTP).
- Audit logs table capturing sensitive operations (course publish, price change).
- Rate-limit login attempts (5 per minute per IP) + captcha after threshold.

## Session Management
- Maintain refresh token blacklist (or change `token_version` per user).
- Provide user dashboard to manage active sessions/devices.

## Compliance & Privacy
- GDPR support: data export endpoints per role.
- Collect parental consent for underage students (if applicable).


