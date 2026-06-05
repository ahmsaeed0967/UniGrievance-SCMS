# 📋 SCMS — Student Complaint Management System

<div align="center">

![HTML](https://img.shields.io/badge/Frontend-HTML%2FCSS%2FJS-orange?style=for-the-badge&logo=html5)
![Tailwind](https://img.shields.io/badge/Styling-Tailwind%20CSS-38B2AC?style=for-the-badge&logo=tailwind-css)
![Supabase](https://img.shields.io/badge/Backend-Supabase-3ECF8E?style=for-the-badge&logo=supabase)
![Course](https://img.shields.io/badge/Course-FSE%20Group%2016-blueviolet?style=for-the-badge)

**A web-based platform for students to submit and track complaints, and for administrators to review and resolve them — all in real time.**

</div>

---

## 📖 Project Overview

**SCMS (Student Complaint Management System)** is a single-page web application built for the *Fundamentals of Software Engineering (FSE)* course. It provides a structured, transparent channel for students to raise academic, financial, facility, or conduct-related complaints — and for administrators to manage them efficiently.

Without such a system, complaints are scattered, untracked, and often go unresolved. SCMS solves this by giving every complaint a unique ID, a live status, and a resolution trail — all stored securely in the cloud.

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| 🔐 **Authentication** | Students register/login with Student ID; admins use email credentials via Supabase Auth |
| 📝 **Complaint Submission** | Students submit categorized complaints with a title and description (min. 50 chars) |
| 🆔 **Auto-generated Complaint ID** | Each complaint gets a unique `CMP-XXXXXXXX` ID using `crypto.randomUUID()` |
| 📂 **6 Complaint Categories** | Academic, Facilities, Financial, Security, Staff, Other |
| 📋 **My Complaints View** | Students see all their past submissions with status badges and admin notes |
| 🗑️ **Delete Complaint** | Students can delete their own complaints with a confirmation prompt |
| 🛡️ **Admin Dashboard** | Admins see all complaints from all students in a master table |
| 🔍 **Search & Filter** | Admins filter by category, status, or free-text search across title/description/ID |
| ✏️ **Status Updates** | Admins update complaint status: Pending Review → In Progress → Resolved / Rejected |
| 📝 **Resolution Notes** | Mandatory notes required when marking a complaint as Resolved or Rejected |
| 👁️ **Detail Modal** | Admins open a full-detail popup with student name, email, and full description |
| ⚡ **Local Cache** | Admin table uses an in-memory cache and re-renders on filter without extra DB calls |
| 🎨 **Color-coded Statuses** | Yellow (Pending), Blue (In Progress), Green (Resolved), Red (Rejected) |

---

## 👥 User Roles

| Role | Access | Responsibilities |
|------|--------|-----------------|
| **Student** | Register with Student ID (`22I-1234` format), log in, submit, view, delete own complaints | Submit complaints, track progress, view admin resolution notes |
| **Administrator** | Log in via email (flagged `is_admin: true` in Supabase metadata) | View all complaints, filter/search, update statuses, add resolution notes |

> 🔒 Admin access is enforced both client-side (nav link visibility) and through Supabase Row Level Security policies. Students cannot access the Admin Dashboard even if they navigate manually.

---

## 🔄 System Workflow

```
Student Registers → Logs In
         ↓
  Submits Complaint (Title + Category + Description)
         ↓
  Complaint stored in Supabase with status: "Pending Review"
         ↓
  Admin logs in → Sees complaint in Master Dashboard
         ↓
  Admin filters/searches → Opens Detail Modal
         ↓
  Admin updates Status + adds Resolution Notes → Saved to Supabase
         ↓
  Student views "My Complaints" → Sees updated status + Admin notes
```

---

## 🛠️ Technologies Used

| Technology | Purpose |
|------------|---------|
| **HTML5** | Page structure and single-page app views |
| **Tailwind CSS** (CDN) | Utility-first responsive styling and status badge colors |
| **Vanilla JavaScript (ES Modules)** | All frontend logic, event handling, DOM manipulation |
| **Supabase JS SDK v2** | Authentication, real-time database queries (CRUD), session management |
| **Supabase Auth** | User registration, login, session persistence, role metadata |
| **Supabase Database (PostgreSQL)** | Persistent storage of all complaint records |
| **Web Crypto API** | `crypto.randomUUID()` for unique complaint ID generation |

---

## 🗄️ Database Integration

The app connects to Supabase using a public anon key, with the client initialized on page load via `setupSupabase()`.

**Table: `complaints`**

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Supabase auto-generated record ID |
| `complaint_id` | Text | Human-readable ID (`CMP-XXXXXXXX`) |
| `title` | Text | Short complaint title |
| `category` | Text | One of 6 categories |
| `description` | Text | Full complaint body (min 50 chars) |
| `status` | Text | `Pending Review` / `In Progress` / `Resolved` / `Rejected` |
| `user_id` | UUID | Linked to the Supabase Auth user |
| `student_name` | Text | Captured from registration metadata |
| `student_email` | Text | Captured from session |
| `submitted_at` | Timestamp | ISO timestamp of submission |
| `resolution_notes` | Text | Admin's resolution or rejection explanation |

**CRUD Operations Implemented:**

| Operation | Where Used |
|-----------|-----------|
| **Create** | `handleSubmitComplaint()` — inserts new complaint row |
| **Read (own)** | `handleViewComplaints()` — fetches complaints filtered by `user_id` |
| **Read (all)** | `loadAllComplaints()` — admin fetches entire complaints table |
| **Update** | `updateComplaintStatus()` — admin updates status + resolution_notes |
| **Delete** | `deleteComplaint()` — student deletes own complaint (filtered by `user_id`) |

---

## 📁 Project Structure

```
SCMS/
│
└── Implementation_Group16_CYB.html   ← Complete single-file application
    ├── <head>
    │   ├── Tailwind CSS (CDN)
    │   └── <script type="module">
    │       ├── Supabase Client Setup
    │       ├── Auth Handlers (login, register, logout)
    │       ├── Student Features (submit, view, delete)
    │       └── Admin Module (load, filter, update, modal)
    │
    └── <body>
        ├── Navigation Bar (dynamic per role)
        ├── Login View
        ├── Register View
        ├── Complaint Form View        ← Student
        ├── My Complaints View         ← Student
        ├── Admin Dashboard View       ← Admin only
        └── Admin Detail Modal         ← Admin only
```

---

## ⚙️ How It Works — Step by Step

1. **Page Load** — Supabase client initializes; `onAuthStateChange` listener is registered. A loading overlay hides until session is resolved.
2. **Login/Register** — Students enter their ID (`22I-1234` format) which is converted to a virtual email (`22I-1234@student.university.edu`) for Supabase Auth. Admins log in directly with their email.
3. **Role Detection** — After login, `user_metadata.is_admin` is checked. Students are shown the complaint form; admins go directly to the dashboard.
4. **Submit Complaint** — The form validates input (category required, description ≥ 50 chars), generates a `CMP-` ID, and inserts a record into Supabase.
5. **View My Complaints** — Fetches only the logged-in student's complaints, ordered newest first. Each card shows status, category, date, and any admin resolution notes.
6. **Admin Dashboard** — Loads all complaints into a local cache (`allComplaintsCache`). Filters operate on the cache without additional DB calls.
7. **Status Update** — Admin opens a modal, selects a new status, and enters resolution notes (mandatory for Resolved/Rejected). Supabase is updated and the local cache is patched instantly.
8. **Logout** — `supabase.auth.signOut()` clears the session, returning the user to the login view.

---

## 🚀 Installation & Setup

### Prerequisites
- A modern browser (Chrome, Firefox, Edge)
- A Supabase account with a configured project and `complaints` table

### Steps

```bash
# Step 1: Clone or download the project
git clone https://github.com/your-repo/scms.git

# Step 2: Open the file directly in a browser — no build step needed
open Implementation_Group16_CYB.html
```

### Supabase Configuration

```javascript
// Located at the top of the <script> block in the HTML file
const SUPABASE_URL     = "https://your-project.supabase.co";
const SUPABASE_ANON_KEY = "your-anon-key-here";
```

### Supabase Table Setup

```sql
CREATE TABLE complaints (
    id               UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    complaint_id     TEXT,
    title            TEXT,
    category         TEXT,
    description      TEXT,
    status           TEXT DEFAULT 'Pending Review',
    user_id          UUID REFERENCES auth.users(id),
    student_name     TEXT,
    student_email    TEXT,
    submitted_at     TIMESTAMPTZ DEFAULT now(),
    resolution_notes TEXT
);
```

> ⚠️ Enable **Row Level Security (RLS)** so students can only access their own rows, while admins can read and update all records.

---

## 💡 Software Engineering Concepts Applied

| Concept | Application in SCMS |
|---------|-------------------|
| **Requirements Analysis** | Defined user stories for students (submit, view, delete) and admins (review, update, filter) |
| **System Design** | Single-page architecture with role-based view routing via `showView()` |
| **Database Design** | Normalized `complaints` table with FK to auth users |
| **Frontend Development** | Event-driven JS with form handling, DOM rendering, and in-memory state management |
| **Input Validation** | Student ID regex (`/^[1-9][0-9]I-\d{4}$/`), min-length description, mandatory notes for final statuses |
| **Authentication & Authorization** | Supabase Auth for identity; `is_admin` metadata flag for role-based access control |
| **CRUD Implementation** | Full Create, Read, Update, Delete operations via Supabase API |
| **Error Handling** | All async operations wrapped in `try/catch`; errors shown as inline UI messages |
| **Performance Optimization** | In-memory cache for admin table avoids redundant DB fetches on filter change |



---

<div align="center">

*Built to give every student complaint a voice — and every admin a clear path to resolve it.*

</div>
