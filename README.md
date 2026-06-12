# 🎓 Placement & Internship Management System (PIMS)

> A production-style, full-stack web application that simulates real-world campus recruitment workflows.

PIMS manages end-to-end placement operations between **Students**, **Companies**, and **Admins** — from job posting to candidate shortlisting — with secure role-based access, business rule enforcement, and cloud deployment.

<p align="center">
  <img src="https://img.shields.io/badge/Java-17-ED8B00?style=flat-square&logo=openjdk&logoColor=white">
  <img src="https://img.shields.io/badge/Spring_Boot-3.2.5-6DB33F?style=flat-square&logo=springboot&logoColor=white">
  <img src="https://img.shields.io/badge/Spring_Security-JWT-6DB33F?style=flat-square&logo=springsecurity&logoColor=white">
  <img src="https://img.shields.io/badge/PostgreSQL-Neon_Cloud-336791?style=flat-square&logo=postgresql&logoColor=white">
  <br/>
  <img src="https://img.shields.io/badge/HTML5-CSS3-E34F26?style=flat-square&logo=html5&logoColor=white">
  <img src="https://img.shields.io/badge/JavaScript-Vanilla-F7DF1E?style=flat-square&logo=javascript&logoColor=black">
  <img src="https://img.shields.io/badge/Docker-Containerized-2496ED?style=flat-square&logo=docker&logoColor=white">
  <img src="https://img.shields.io/badge/Deployed-Render_%7C_Netlify-7B2FBE?style=flat-square&logo=netlify&logoColor=white">
</p>

---

## 🌐 Live Demo

> 🖥 **Try the app here →** 🔗 [chic-cheesecake-df6cbd.netlify.app](https://chic-cheesecake-df6cbd.netlify.app)

> 📄 **Explore the API →** 🔗 [Swagger UI](https://pims-backend-xa8s.onrender.com/swagger-ui/index.html)

> ⚠️ Backend is on Render's free tier — first request may take ~30 seconds to wake up.
---

## 📌 Table of Contents

- [Why I Built This](#-why-i-built-this)
- [Key Features at a Glance](#-key-features-at-a-glance)
- [Technology Stack](#-technology-stack)
- [System Modules](#-system-modules)
- [Architecture & Design Decisions](#-architecture--design-decisions)
- [Security Implementation](#-security-implementation)
- [Business Logic Highlights](#-business-logic-highlights)
- [Project Structure](#-project-structure)
- [Local Setup](#-local-setup)
- [Challenges & Learnings](#-challenges--learnings)
- [Future Roadmap](#-future-roadmap)

---

## 💡 Why I Built This

Most college placement processes are managed through spreadsheets, emails, and manual coordination. I wanted to build a system that mirrors how a real placement cell would work digitally — with proper authentication, role separation, eligibility rules, and a live deployment anyone can access.

---

## ✅ Key Features at a Glance

- 🔐 JWT-based stateless authentication with role-based access (`STUDENT`, `COMPANY`, `ADMIN`)
- 📄 Students can upload resumes and apply to jobs with CGPA and deadline validation
- 🏢 Companies can post jobs, set eligibility criteria, and manage candidate applications
- 🛡 Admins approve company registrations before they can interact with the platform
- ☁️ Fully deployed — backend on Render (Dockerized), frontend on Netlify, database on Neon PostgreSQL
- 📖 REST API fully documented with Swagger UI

---

## 🧠 Technology Stack

### Backend
| Technology | Purpose |
|---|---|
| Java 17 | Core language |
| Spring Boot 3.2.5 | Application framework |
| Spring Security | Authentication and authorization |
| Spring Data JPA | ORM and database interaction |
| JWT (jjwt 0.11.5) | Stateless token-based authentication |
| PostgreSQL | Relational database |

### Frontend
| Technology | Purpose |
|---|---|
| HTML5 | Page structure |
| CSS3 | Styling and layout |
| Vanilla JavaScript | Dynamic behavior and API calls via Fetch API |
| JWT in localStorage | Client-side auth state management |

### Infrastructure & Tooling
| Component | Platform / Tool |
|---|---|
| Backend Hosting | Render (Dockerized container) |
| Frontend Hosting | Netlify |
| Database | Neon PostgreSQL (Cloud) |
| Containerization | Docker |
| API Documentation | Springdoc OpenAPI (Swagger UI) |
| Build Tool | Maven |

---

## 🧩 System Modules

### 👨‍🎓 Student Module

Students go through a complete placement journey — from registering on the platform to tracking their application outcomes.

**Capabilities:**
- Secure registration and login (JWT-protected)
- Profile setup — branch, CGPA, graduation year
- Resume upload directly through the platform
- Browse active job postings from approved companies
- Apply to jobs (subject to eligibility checks)
- Track real-time application status: `Applied` → `Shortlisted` / `Rejected`

**Business Rules Enforced:**
- A student cannot apply to the same job twice (database-level unique constraint)
- Applications are blocked if the student's CGPA is below the job's minimum requirement
- Applications are blocked if the job deadline has passed
- Students cannot access company or admin endpoints (role-restricted)

---

### 🏢 Company Module

Companies interact with the platform to post opportunities and manage their hiring pipeline.

**Capabilities:**
- Register as a company on the platform
- Await admin approval before gaining full access
- Complete company profile setup
- Create job postings with:
  - Role title and description
  - Minimum CGPA requirement
  - Eligible branches (CSE, ISE, and more)
  - Application deadline
  - Number of open positions
- View all applications received for their postings
- Shortlist or reject individual candidates

**Security Enforced:**
- A company can only view and manage its own job postings — not other companies'
- A company can only update application statuses for jobs it owns
- All endpoints protected via role-based authorization

---

### 👨‍💼 Admin Module

Admins act as platform gatekeepers, ensuring quality control before companies can participate.

**Capabilities:**
- Review and approve (or reject) company registration requests
- Platform-level visibility and access control
- Initial admin account auto-seeded via `AdminInitializer` on application startup — no manual database entry required

---

## 🏗 Architecture & Design Decisions

PIMS follows a **Layered Monolithic Architecture** — a deliberate choice for a solo project of this scope. It keeps the codebase navigable while maintaining clear separation of responsibilities

```
Controller → Service → Repository → Database
```

**Design decisions worth noting:**

- **DTO-based separation** — Request and response objects are completely separated from database entities. This prevents accidental data leaks (like exposing password hashes in API responses) and makes the API contract independent of the internal data model.

- **Service-layer validation** — All business rules (CGPA checks, deadline checks, ownership checks) live in the service layer, not the controller or repository. This keeps each layer focused on a single responsibility and makes the logic straightforward to reason about.

- **Global exception handling** — A centralized `@ControllerAdvice` handler catches all exceptions and returns consistent, structured error responses. This avoids scattered try-catch blocks and ensures clients always receive predictable error formats regardless of where something fails.

- **Custom JWT filter** — Rather than relying on default Spring Security form-login, a custom `JwtAuthenticationFilter` intercepts every request, validates the token, and injects the user's role into the security context. This enables fully stateless authentication with no server-side sessions.

- **BCrypt hashing** — Passwords are never stored in plain text. BCrypt is used with its built-in salting to protect credentials even if the database were to be compromised.

- **Admin seeding** — The initial admin is created programmatically at startup via `AdminInitializer`, meaning the system is ready to use immediately after deployment without any manual database setup.

---

## 🛡 Security Implementation

Security was treated as a first-class concern throughout the project, not an afterthought.

- Stateless JWT authentication — no server-side sessions maintained
- Custom `JwtAuthenticationFilter` validates and parses every protected request
- Role-based endpoint authorization (`ROLE_STUDENT`, `ROLE_COMPANY`, `ROLE_ADMIN`)
- BCrypt password encoding with automatic salting
- Swagger UI whitelisted from the auth filter for API exploration
- CORS policy restricted strictly to the deployed frontend origin — no wildcard `*`
- Ownership validation enforced in the service layer (companies cannot access other companies' data even with a valid token)

---

## ⚙️ Business Logic Highlights

| Rule | Where Enforced |
|---|---|
| No duplicate job applications | Database-level unique constraint |
| CGPA must meet job minimum | Service layer — checked before saving |
| Application deadline must not have passed | Service layer — checked before saving |
| Company can only manage its own jobs | Service layer ownership check |
| Company must be admin-approved to post jobs | Status check on job creation |
| Consistent error responses across all endpoints | Global `@ControllerAdvice` |

---

## 📁 Project Structure

```
PIMS/
│
├── pims-frontend/                  # Vanilla JS frontend
│   ├── css/                        # Stylesheets
│   ├── js/                         # API calls, auth logic, DOM handling
│   ├── student.html                # Student dashboard
│   ├── company.html                # Company dashboard
│   └── admin.html                  # Admin dashboard
│
├── src/main/java/com/pims/
│   ├── config/                     # Spring Security, CORS, JWT configuration
│   ├── controller/                 # REST API endpoint definitions
│   ├── service/                    # Business logic layer
│   ├── repository/                 # Spring Data JPA interfaces
│   ├── entity/                     # JPA database entities
│   ├── dto/                        # Request and response data objects
│   ├── enums/                      # Status enums (ApplicationStatus, Role, etc.)
│   ├── exception/                  # Custom exceptions + GlobalExceptionHandler
│   └── util/                       # JWT utility class
│
├── Dockerfile                      # Container configuration for Render deployment
└── pom.xml                         # Maven dependencies and build config
```

---

## 🚀 Local Setup

**Prerequisites:** Java 17+, Maven, PostgreSQL

**1. Clone the repository**
```bash
git clone https://github.com/keerthana12hv/Placement-And-Internship-Management-System_PIMS.git
cd Placement-And-Internship-Management-System_PIMS
```

**2. Configure `src/main/resources/application.properties`**
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/pims_db
spring.datasource.username=YOUR_USERNAME
spring.datasource.password=YOUR_PASSWORD
jwt.secret=YOUR_SECRET_KEY_MIN_256_BITS
spring.jpa.hibernate.ddl-auto=update
```

**3. Build and run**
```bash
mvn clean install
mvn spring-boot:run
```

**4. Access the application**
- Backend API: `http://localhost:8080`
- Swagger UI: `http://localhost:8080/swagger-ui/index.html`
- Open `pims-frontend/student.html` in your browser for the frontend

> The `AdminInitializer` will automatically seed an admin account on first startup. Check the console logs for the default credentials.

---

## 🧗 Challenges & Learnings

- **Implementing JWT authentication from scratch** was the steepest learning curve. Understanding how to intercept HTTP requests, validate tokens, and inject authentication into Spring Security's context — all without sessions — required deep reading of the Spring Security filter chain. An additional challenge was whitelisting Swagger UI endpoints without opening up security holes elsewhere.

- **Enforcing data ownership at the service layer** required thinking carefully about trust boundaries. Rather than trusting request parameters (which a malicious user could manipulate), the system extracts company identity from the validated JWT and cross-references it against the requested resource on every sensitive operation.

- **Layering database constraints and application validation together** was an important design lesson. CGPA and deadline checks live in the service layer so users receive meaningful error messages. Duplicate application prevention is enforced at the database level as an additional safety net that cannot be bypassed regardless of the code path taken.

- **Containerizing and deploying to Render** turned theoretical knowledge of Docker and environment variables into hands-on experience — including debugging environment-specific issues that only appear in production.

---

## 🔮 Future Roadmap

These are planned improvements — the core system is fully functional as-is.

- **Email notifications** — Automatically notify students when their application status changes, using JavaMailSender or an external service like SendGrid
- **Refresh token support** — Implement a refresh token flow so expired JWTs don't require a full re-login, improving session experience
- **Admin analytics dashboard** — Aggregate platform data: placement rates by branch, most active companies, application volume trends
- **CI/CD pipeline** — Automate testing and deployment on every push to main using GitHub Actions

---

## 👩‍💻 About

PIMS was developed as a full-stack, backend-focused system design exercise.  
It demonstrates practical implementation of authentication, authorization, business rule enforcement, layered architecture, and cloud deployment in a production-style environment.

🔗 [GitHub Profile](https://github.com/keerthana12hv) &nbsp;|&nbsp; 🌐 [Live Demo](https://chic-cheesecake-df6cbd.netlify.app) &nbsp;|&nbsp; 📄 [API Docs](https://pims-backend-xa8s.onrender.com/swagger-ui/index.html)
