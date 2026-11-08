# HabitPro — React + Spring Boot + MongoDB

This is a migration of the supplied Express/EJS HabitPro project. The original CSS is retained and the main workflows are rebuilt as a React SPA backed by Spring Boot REST APIs.

## Preserved workflows
- Signup/login with BCrypt password hashing and JWT authentication
- User-owned habits: create, mark complete once per day, streak count, delete
- Sticky notes: create, edit, delete, pin/unpin, colors, deadline, reminder date
- Notes search and filters; dark/light theme; responsive navigation

## Requirements
- Node.js 20+ and npm
- Java 17+
- Maven 3.9+
- MongoDB locally, or a MongoDB Atlas connection string

## Run locally
1. Start MongoDB locally (or create a MongoDB Atlas database).
2. Backend:
   ```bash
   cd backend
   # Windows PowerShell: $env:MONGODB_URI="mongodb://localhost:27017/habitpro"
   # Optional: set a strong JWT_SECRET and FRONTEND_URL
   mvn spring-boot:run
   ```
   API starts at `http://localhost:8080`; health check: `http://localhost:8080/actuator/health`.
3. Frontend, in a second terminal:
   ```bash
   cd frontend
   copy .env.example .env   # Windows; on macOS/Linux use cp
   npm install
   npm run dev
   ```
   Open `http://localhost:5173`.

## Deploy (Vercel + Render + MongoDB Atlas)
### 1. GitHub
- Extract this project, open the `habitpro-migration` folder in VS Code.
- Create a GitHub repository, then run `git init`, `git add .`, `git commit -m "Migrate HabitPro to React and Spring Boot"`, `git branch -M main`, `git remote add origin YOUR_GITHUB_REPO_URL`, `git push -u origin main`.
- Do not commit secrets or a real `.env` file.

### 2. MongoDB Atlas
- Create a free/shared cluster and database named `habitpro`.
- Create a database user and allow network access from Render (for initial learning/testing, Atlas `0.0.0.0/0`; restrict this later where possible).
- Copy the Atlas connection string and replace username/password; use database name `habitpro`.

### 3. Render backend
- Create a **Web Service** from the GitHub repo.
- Root Directory: `backend`
- Build Command: `mvn clean package -DskipTests`
- Start Command: `java -jar target/habitpro-api-1.0.0.jar`
- Environment: Docker (the included `backend/Dockerfile` builds the Spring Boot service).
- Environment variables:
  - `MONGODB_URI` = your Atlas connection string
  - `JWT_SECRET` = a long random secret (32+ bytes)
  - `FRONTEND_URL` = your eventual Vercel URL, e.g. `https://your-app.vercel.app`
  - `PORT` = Render supplies this automatically
- Deploy and copy the Render service URL.

### 4. Vercel frontend
- Import the same GitHub repo in Vercel.
- Root Directory: `frontend`
- Framework preset: Vite
- Build Command: `npm run build`
- Output Directory: `dist`
- Environment variable: `VITE_API_URL` = `https://YOUR-RENDER-SERVICE.onrender.com/api`
- Deploy.

### 5. Connect both deployments
- Go back to Render and set `FRONTEND_URL` to the exact Vercel URL (no trailing slash), then redeploy the backend.
- If you add a Vercel custom domain, update `FRONTEND_URL` to that exact domain and redeploy.
- Test signup, login, habit CRUD, daily completion, sticky-note CRUD, pinning, and refresh persistence.

## Notes / limitations to verify before production
- The existing model only stores a numeric streak and completion dates. This migration preserves that model and increments streak on each successful daily completion; it does not yet recalculate streaks after missed days. For accurate consecutive-day streaks, implement a date-based streak recalculation before production.
- `remember` is accepted by the login form for compatibility but JWT expiry is currently controlled by `JWT_EXPIRATION_MS` for all sessions. Configure this globally or extend token issuance for per-user remember-me expiry.
- Browser notifications are not implemented; reminder timestamps are stored and shown, but do not trigger push notifications.
- The original project used two different local MongoDB database names in its connection code. This migration uses one configured database (`habitpro`). Existing local MongoDB data is not automatically migrated; export/import or write a migration script if you need to keep old records.
- Run a complete manual acceptance test after deployment. Hosting providers can change free-tier behavior, cold starts, and build images.
