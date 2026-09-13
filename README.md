[README.md](https://github.com/user-attachments/files/32161518/README.md)
# AI Resume Builder & Job Matching Platform

A full-stack web platform that lets users build professional resumes, get AI-generated
resume improvements, and receive AI-powered job recommendations based on their skills.

Built for: **Full Stack Web Development with AI — Assignment 2**

---

## 1. Project Overview

Users register, build a structured resume (education, skills, experience, projects),
and use an AI assistant to sharpen their professional summary. They can then browse job
listings, apply, track application status, and get an AI-ranked list of jobs that best
match their resume. Admins manage the job board and review incoming applications.

## 2. Features

- **Authentication** — JWT-based registration/login with hashed passwords (bcrypt).
- **Resume Builder** — Structured sections for contact info, summary, skills, experience,
  projects, and education. Live preview + browser print-to-PDF download.
- **AI Summary Improvement** — One click sends the user's skills/experience/projects to
  an LLM (OpenAI API) which returns a polished, ATS-friendly professional summary.
- **Job Listings** — Searchable/filterable job board (title, company, skills, location,
  job type).
- **AI Job Matching / Recommendations** — Each job is scored against the user's resume by
  the AI, returning a 0–100 match score, matched skills, missing skills, and a short
  explanation, sorted best-match-first.
- **Applications Tracker** — Users apply to jobs and track status (applied → under
  review → shortlisted/rejected/hired).
- **Admin Module** — Create/edit/delete job postings and update applicant status.

## 3. Technology Stack

| Layer          | Technology                                   |
|----------------|-----------------------------------------------|
| Frontend       | React 18 + Vite, React Router                 |
| Backend        | Node.js + Express.js                          |
| Database       | MongoDB (Mongoose ODM)                        |
| AI             | OpenAI API (`openai` npm package)             |
| Authentication | JWT (jsonwebtoken) + bcryptjs password hashing|
| Version Control| Git + GitHub                                  |

## 4. Project Structure

```
ai-resume-platform/
├── backend/
│   ├── config/db.js            # MongoDB connection
│   ├── models/                 # User, Resume, Job, Application (Mongoose schemas)
│   ├── controllers/            # Business logic per resource
│   ├── routes/                 # Express route definitions
│   ├── middleware/              # auth (JWT) + admin role guard
│   ├── services/aiService.js   # OpenAI API wrapper (summary + matching)
│   ├── server.js                # App entry point
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── api/axios.js         # Axios instance with JWT interceptor
│   │   ├── context/AuthContext.jsx
│   │   ├── components/          # Navbar, ProtectedRoute, ResumePreview
│   │   ├── pages/                # Login, Register, Dashboard, ResumeBuilder,
│   │   │                         # JobListings, Recommendations, Applications, AdminJobs
│   │   ├── App.jsx / main.jsx / styles.css
│   └── .env.example
└── README.md
```

## 5. Database Schema

**Users**: `user_id, name, email, password_hash, role (user/admin)`

**Resumes**: `resume_id, user_id, fullName, contact{email,phone,location,linkedin,github}, summary, education[], experience[], projects[], skills[]`

**Jobs**: `job_id, title, company, location, description, skills[], jobType, postedBy`

**Applications**: `application_id, user_id, job_id, resume_id, status, applied_at`

(MongoDB collections, defined as Mongoose schemas — equivalent relational tables would
use the same fields with foreign keys on `user_id` / `job_id`.)

## 6. Installation & Setup

### Prerequisites
- Node.js v18+
- MongoDB running locally (or a MongoDB Atlas connection string)
- An OpenAI API key (or another OpenAI-compatible API key)

### Backend

```bash
cd backend
npm install
cp .env.example .env
# edit .env: set MONGO_URI, JWT_SECRET, OPENAI_API_KEY
npm run dev        # starts on http://localhost:5000
```

### Frontend

```bash
cd frontend
npm install
cp .env.example .env
# edit .env if your backend runs on a different URL
npm run dev         # starts on http://localhost:5173
```

### Creating an admin user
By default every registered user has role `user`. To create an admin, register normally,
then update that user's `role` field to `admin` directly in MongoDB:

```js
// in mongosh
use ai_resume_platform
db.users.updateOne({ email: "you@example.com" }, { $set: { role: "admin" } })
```

## 7. Environment Variables

**backend/.env**
```
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/ai_resume_platform
JWT_SECRET=replace_with_a_long_random_secret
JWT_EXPIRES_IN=7d
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_MODEL=gpt-4o-mini
CLIENT_ORIGIN=http://localhost:5173
```

**frontend/.env**
```
VITE_API_URL=http://localhost:5000/api
```

> Never commit `.env` files — both `backend/.gitignore` and `frontend/.gitignore` already
> exclude them. Only `.env.example` files (with placeholder values) are committed.

## 8. API Endpoints

### Auth (`/api/auth`)
| Method | Endpoint     | Auth | Description               |
|--------|--------------|------|----------------------------|
| POST   | `/register`  | No   | Register a new user        |
| POST   | `/login`     | No   | Login, returns JWT         |
| GET    | `/me`        | Yes  | Get logged-in user profile |
| PUT    | `/me`        | Yes  | Update profile (name)      |

### Resumes (`/api/resumes`)
| Method | Endpoint | Auth | Description                          |
|--------|----------|------|----------------------------------------|
| GET    | `/me`    | Yes  | Get (or auto-create) my resume         |
| PUT    | `/me`    | Yes  | Create/update my resume                |
| GET    | `/:id`   | Yes  | Get a resume by ID (owner or admin)    |

### Jobs (`/api/jobs`)
| Method | Endpoint | Auth        | Description                         |
|--------|----------|-------------|---------------------------------------|
| GET    | `/`      | No          | List jobs (search/location/skill/jobType filters, pagination) |
| GET    | `/:id`   | No          | Get a single job                     |
| POST   | `/`      | Admin       | Create a job posting                 |
| PUT    | `/:id`   | Admin       | Update a job posting                 |
| DELETE | `/:id`   | Admin       | Delete a job posting                 |

### Applications (`/api/applications`)
| Method | Endpoint       | Auth  | Description                          |
|--------|----------------|-------|----------------------------------------|
| POST   | `/`            | Yes   | Apply to a job (`{ job_id }`)          |
| GET    | `/me`          | Yes   | List my applications                   |
| GET    | `/`            | Admin | List all applications (optional `?job_id=`) |
| PUT    | `/:id/status`  | Admin | Update an application's status         |

### AI (`/api/ai`)
| Method | Endpoint                | Auth | Description                                            |
|--------|-------------------------|------|----------------------------------------------------------|
| POST   | `/improve-summary`      | Yes  | Generate/improve the professional summary from resume data |
| GET    | `/recommendations`      | Yes  | AI-ranked list of jobs matched against my resume         |
| POST   | `/match-job/:jobId`     | Yes  | AI match score for one specific job                       |

## 9. AI Integration Details

`backend/services/aiService.js` centralizes all AI calls:
- **`improveSummary()`** — sends the user's skills, experience, and projects to the
  configured OpenAI model and asks for a concise, ATS-friendly professional summary.
- **`matchResumeToJob()`** — sends the resume's skills/summary and a job's requirements
  to the model, which returns a JSON object `{score, matchedSkills, missingSkills, reason}`.
  If the model output isn't valid JSON, a keyword-overlap fallback computes an approximate
  score so the feature degrades gracefully instead of failing.

The provider and model are configured entirely through `OPENAI_API_KEY` and
`OPENAI_MODEL` in `.env`, so swapping to a different OpenAI-compatible provider requires
no code changes.

## 10. Testing Notes

Before submission, verify manually:
- Register + login flow, including duplicate-email and wrong-password error cases.
- Resume builder: add/edit/remove skills, experience, projects, education; save; reload
  page and confirm data persists; AI Improve Summary button.
- Job listings: search/filter, apply, duplicate-apply is blocked.
- Recommendations: scores render, sorted descending.
- Admin: create/edit/delete jobs, change application status (requires an admin account).
- Print/download a resume as PDF via the browser print dialog.

## 11. Security Notes

- Passwords are hashed with bcrypt; plaintext passwords are never stored.
- JWTs are required on all protected routes via the `protect` middleware.
- Admin-only routes are additionally guarded by the `isAdmin` middleware.
- No secrets are committed to source control — only `.env.example` placeholders.

## 12. Conclusion

This project demonstrates a complete MERN-stack workflow enhanced with practical AI
integration: resume content generation and AI-driven job matching, wrapped in secure,
role-based REST APIs and a responsive React frontend.
