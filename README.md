# DevOps Task Board

A small but real **frontend + backend + MongoDB** app, built to teach how to deploy a
2-tier application on **AWS EC2** with **Nginx**, **AMIs**, **Launch Templates**,
**two Application Load Balancers** and **two Auto Scaling Groups**.

```
Browser ─► Frontend ALB ─► Nginx (frontend EC2 x2+) ─► Backend ALB (internal)
        ─► Nginx (backend EC2 x2+) ─► Node.js API :5000 ─► MongoDB Atlas
```

The UI shows **which frontend and which backend instance answered each request**, so you can
watch load balancing, scale-out and self-healing happen live in the browser.

The full step-by-step guide is in [`docs/GUIDE.md`](docs/GUIDE.md) (PDF: `docs/EC2-ALB-ASG-Lab-Guide.pdf`).

---

## Project structure

| Path | What it is |
|---|---|
| `frontend/` | Static UI: `index.html`, `style.css`, `app.js` (no build step) |
| `backend/` | Node.js + Express + Mongoose API (`server.js`, `models/`, `lib/`) |
| `backend/.env.example` | Config template: copy it to `.env` and add your `MONGO_URI` |
| `nginx/frontend.conf.template` | Nginx on frontend EC2: static files + `/api` reverse proxy |
| `nginx/backend.conf.template` | Nginx on backend EC2: port 80 → Node.js on 5000 |
| `deploy/taskboard-api.service` | systemd unit that keeps the API running (backend) |
| `deploy/taskboard-meta.service` + `write-instance-meta.sh` | Writes `instance.json` at every boot (frontend) |
| `deploy/user-data-frontend.sh` | Launch template user data for the frontend |
| `deploy/user-data-backend.sh` | Launch template user data for the backend |
| `deploy/setup-*.sh` | Optional one-shot setup scripts for the first instances |
| `docker-compose.yml` | Run the same architecture on your laptop |

## Placeholders you must change

| Placeholder | Where | Replace with |
|---|---|---|
| `<YOUR_GITHUB_USERNAME>` | guide, `deploy/setup-*.sh` | Your GitHub username (after you fork or push this repo) |
| `<db_username>` `<db_password>` `<cluster-host>` | `backend/.env` (copy of `.env.example`) | Values from your MongoDB Atlas connection string |
| `<BACKEND_PRIVATE_IP>` | phase 1 frontend Nginx setup | Private IPv4 of backend EC2 #1 |
| `<BACKEND_ALB_DNS>` | `deploy/user-data-frontend.sh` | DNS name of the internal Backend ALB |

## Run it on your laptop

### Option A: Docker Compose (closest to AWS)
```bash
docker compose up -d --build
# open http://localhost:8080
docker compose up -d --scale api=3   # 3 backend containers -> watch the live LB view
docker compose down
```

### Option B: Node.js only (no Docker, no Atlas account needed)
On Windows just double-click **`start-local.bat`**. Or run it yourself:
```bash
cd backend
npm install
npm run dev:local        # starts a temporary in-memory MongoDB
# open http://localhost:5000
```

### Option C: Node.js + your MongoDB Atlas
```bash
cd backend
npm install
cp .env.example .env     # Windows: copy .env.example .env
# edit .env -> set MONGO_URI and SERVE_FRONTEND=true
npm start                # open http://localhost:5000
```

## API

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/health` | 200 when API + DB are OK (ALB health check) |
| GET | `/api/info` | Which instance answered, AZ, IP, DB status |
| GET/POST | `/api/tasks` | List / create tasks |
| PATCH/DELETE | `/api/tasks/:id` | Update / delete a task |
| POST | `/api/stress?seconds=300` | Burn CPU for the Auto Scaling demo (needs `ENABLE_STRESS=true`) |
