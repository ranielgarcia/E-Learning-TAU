# Application Features

This document describes the functionality of the E-Learning TAU system, an
agriculture e-learning platform built for the Agriculture Department of
Tarlac Agricultural University (TAU) on CodeIgniter 3.1.9. It focuses on
*what the application does*; for setup/installation instructions see
[README.md](README.md).

The application has two independent portals, each with its own controller,
session, and login system:

- **Student Portal** — `application/controllers/Students.php`
- **Admin / Faculty Portal** — `application/controllers/Admin.php`

---

## 1. Content Model

All learning material is organized in a four-level hierarchy, and quizzes are
attached to the third level:

```
Agriculture Principle
 └─ Sub Topic
     └─ Chapter
         ├─ Lesson (one or more)
         └─ Chapter Quiz (one or more)
             └─ Quiz Question (one or more)
                 └─ Question Choice (one or more, single or multiple correct)
```

- **Principle** — top-level subject area (e.g. a broad agricultural
  principle).
- **Sub Topic** — a topic under a Principle.
- **Chapter** — a chapter under a Sub Topic; the unit that both Lessons and
  Quizzes attach to.
- **Lesson** — the actual learning content (rich text + optional cover
  photo) shown to students.
- **Chapter Quiz** — an assessment tied to a Chapter, made up of Questions.
- **Quiz Question / Question Choice** — each question has one or more
  choices; a question can be authored as **single-answer** (rendered as
  radio buttons to the student) or **multi-answer** (rendered as
  checkboxes), determined by how many choices are marked as correct.

This same hierarchy drives the collapsible sidebar navigation shown to
students (Principle → Sub Topic → Chapter → Lesson links).

---

## 2. Student Portal

### 2.1 Registration
- Students self-register with student ID number, email, name, and password.
- The ID number must already exist in a faculty-maintained master list of
  enrolled students (`select_std_num` lookup) — unlisted IDs are rejected.
- On submit, a 6-character verification code is generated and the
  registration is held in a temporary table pending confirmation; the
  student must enter the code (intended to be emailed) on a follow-up page
  before the account is created in the real `Students` table.
- The verification code expires after a fixed window (1 hour); an expired
  or already-registered session shows an "expired" page instead of the code
  form.

### 2.2 Login / Logout
- Login is by student number + password; passwords are hashed with SHA-512
  before comparison (no salting).
- A successful login stores student identity fields in the CodeIgniter
  session (`std_session_id`, `std_session_stdNum`, etc.) and a `logged_in`
  flag.
- Logout clears all student session keys and redirects to the login page.
- Every authenticated page checks `is_student_still_logged_in()` and
  redirects to login if the session is missing/invalid.

### 2.3 Password Recovery
- A student can request recovery with their student number + email; if they
  match, a random 6-character recovery code (with expiry) is stored and the
  session is flagged so the student can proceed to enter it.
- After verifying the code, the student is allowed to set a new password
  (hashed with SHA-512) via a dedicated "change password" form; this access
  is gated by session flags so the form can't be reached directly.

### 2.4 Home / Dashboard
- An image carousel/banner of static promotional slides.
- Two "latest lessons" feeds: lessons that have a cover photo (shown as
  featured cards) and lessons without one (shown as a simpler list).

### 2.5 Browsing & Reading Lessons
- A collapsible sidebar (Principle → Sub Topic → Chapter → Lesson) lets
  students drill down to any lesson in the catalog.
- The lesson detail page shows the rich-text content, the date added,
  principle/topic/chapter breadcrumb, the author, and links to any quizzes
  attached to that lesson's chapter.
- A "chapter lessons" panel on the same page lists sibling lessons in the
  same chapter, highlighting the one currently being viewed.
- **Comments**: students can post comments on a lesson and view the full
  comment thread (also visible to/postable by faculty); each comment is
  timestamped and attributed to a student or faculty name.

### 2.6 Lesson Search
- A top-bar search box lets students search lessons by keyword; if the
  query is empty or invalid it falls back to showing the latest lessons.

### 2.7 Quizzes
- Students take a Chapter Quiz from the lesson page; questions are rendered
  as radio buttons (single correct answer) or checkboxes (multiple correct
  answers) based on how the question was authored.
- On submit, answers are validated, scored server-side (comparing selected
  choice IDs against choices flagged as correct), and the attempt plus every
  individual answer is persisted (`StudentQuizResults` /
  `StudentQuizAnswers`).
- Students can view a history of all their quiz attempts with score and
  date, and drill into any past attempt to see a per-question breakdown of
  which choices were correct/incorrect ("results" view).

### 2.8 Profile
- Students can view and update their own profile: name, email, section,
  and password (password field is optional — leaving it blank keeps the
  current password).

### 2.9 Session/Access Safeguards
- `block_url_copy_paste()` is called on protected pages: if the request has
  no `HTTP_REFERER` header, the student's session is destroyed and they are
  redirected to login. This is a crude deterrent against directly
  pasting/bookmarking internal URLs, not a real security control.
- All user input for authenticated actions is run through CodeIgniter's
  `xss_clean()` and form validation before being used in queries.

---

## 3. Admin / Faculty Portal

### 3.1 Roles & Access
Faculty accounts carry two boolean flags, `isAdmin` and `isDean`, that
combine into four effective roles used for page-level gating:

| isAdmin | isDean | Role label   |
|---------|--------|--------------|
| 1       | 0      | Admin        |
| 0       | 1      | Dean         |
| 1       | 1      | Root user    |
| 0       | 0      | Faculty      |

Content-structure pages (Principles, Sub Topics) are restricted to
Admin/Root roles — plain Faculty/Dean users are redirected back to the main
panel. Faculty accounts can still manage Lessons, Quizzes, and their own
profile.

### 3.2 Login / Logout / Profile
- Login is by faculty ID number + SHA-512-hashed password, mirroring the
  student login mechanism but with its own session namespace
  (`admin_session_*`, `admin_logged_in`).
- Faculty can view and update their own profile (name, email, password).
- The same referer-based `block_url_copy_paste()` session guard used on the
  student side is applied to admin pages.

### 3.3 Dashboard
- A landing "main panel" shown after login, aware of the current user's
  role for conditional UI.

### 3.4 Content Management (Principles → Sub Topics → Chapters → Lessons)
For each of the four levels, the admin panel provides a full management
screen with:
- **Create / Update** via modal forms.
- **List / Search** (live search endpoints per level).
- **Soft delete** — records are flagged `isDeleted` rather than removed.
- **Recycle Bin** — a dedicated list of soft-deleted records per level,
  with its own search and a **Restore** action.

Lessons specifically also support:
- Rich text content editor (TinyMCE).
- Optional cover photo upload, with an orientation setting
  (Landscape/Portrait) that controls how it is displayed on listing pages.
- "Advance search" with more filter criteria than the basic search.
- A lesson **update summary/audit view** (`view_lesson_update_summary`)
  showing what changed on a given lesson.
- Faculty can view lessons and comment on them from the admin side
  (`faculty_view_lesson`, `add_lesson_comment`) the same way students do.

### 3.5 Quiz Authoring
- Add one or more Quizzes to a Chapter.
- Add / update / delete Questions within a quiz.
- Add / update / delete Choices within a question, marking any subset of
  choices as the correct answer(s) — one correct choice yields a
  single-answer (radio) question for students, more than one yields a
  multi-answer (checkbox) question.
- A combined "questions and choices matrix" endpoint assembles a full quiz
  (with nested choices and correctness counts) for rendering the take-quiz
  and author-quiz screens.

### 3.6 Reviewing Student Quiz Results
- View an individual student's result for a specific quiz attempt,
  including per-question correctness (`std_quiz_view_results`).
- View an aggregate list of all students' quiz results across the system
  (`get_all_stds_quizzes_results`).

### 3.7 Faculty Management
- Add / update faculty accounts (ID number, name, email, password).
- Promote/demote a faculty member's Admin/Dean flags
  (`mark_faculty_as_admin_or_dean`).
- Soft delete, list, search, and restore faculty accounts via a Recycle Bin,
  mirroring the content-entity pattern.
- Uniqueness validation callbacks prevent duplicate faculty ID numbers or
  emails on insert/update.

### 3.8 Student Management
- Add / update individual student records.
- **Mass upload of student numbers** — lets an admin bulk-load the
  "master list" of enrolled student IDs that self-registration checks
  against, rather than adding students one at a time.
- Validate a single student number on demand (`validate_student_number`).
- List, search, soft-delete, and restore students via a Recycle Bin, same
  pattern as Faculty/Lessons/Chapters/etc.
- Uniqueness validation callbacks for student ID number and email.

### 3.9 Audit Trail
- A dedicated Audit Trail screen lists and searches system change history
  (`audit_trail`, `get_all_audit_trails`, `search_audit_trail`), giving
  admins visibility into who changed what.

### 3.10 Recycle Bin (cross-cutting)
Every major entity — Principles, Sub Topics, Chapters, Lessons, Faculties,
and Students — follows the same soft-delete/restore convention: deleting an
entity only sets `isDeleted = 1`; a per-entity Recycle Bin page lists
deleted records (searchable) and offers a one-click Restore action that
resets the flag.

---

## 4. Cross-Cutting Technical Notes

- **Two independent auth systems**: student session keys and admin session
  keys never overlap, so a student login has no visibility into the admin
  panel and vice versa.
- **Password hashing**: both portals hash passwords with plain SHA-512
  (`hashSHA512_helper.php`) — there is no per-user salt or use of PHP's
  `password_hash()`.
- **Input handling**: controller actions generally run POST data through
  CodeIgniter's `xss_clean()` and `form_validation` library before touching
  the database, including custom callback rules (e.g. uniqueness checks,
  enrollment checks).
- **Temporary-record + expiry pattern**: both registration and password
  recovery stage their data in separate "temp storage" tables with a random
  6-character code and an expiry timestamp, only writing to the permanent
  table once the code is confirmed.
- **Soft delete**: destructive actions across the admin panel are
  implemented as an `isDeleted` flag update, never a SQL `DELETE`, enabling
  the Recycle Bin/Restore feature everywhere.

## 5. Known Limitations (legacy app)

- No CSRF token usage was found protecting the POST/AJAX endpoints.
- Password hashing is unsalted SHA-512, which is weak by modern standards.
- The "referer header" session guard (`block_url_copy_paste`) is easily
  bypassed and is not a substitute for proper authorization checks.
- Built on CodeIgniter 3.1.9, which is end-of-life and incompatible with
  PHP 8+ (see [README.md](README.md) for supported PHP versions).
