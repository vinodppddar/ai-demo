# AI Animation Video Maker

AI-powered image-to-animation and lip-sync video platform based on the supplied BRD v1.0.

## MVP

- Register / login with JWT
- Dashboard and project history
- Image upload and preview
- Natural-language animation prompt
- Asynchronous video generation jobs
- Demo image-to-video renderer using FFmpeg (Ken Burns motion)
- Pluggable external AI provider adapter
- Optional dialogue field and provider-ready TTS/lip-sync hook
- Video preview and download
- PostgreSQL + Redis + Celery
- Docker Compose for local development
- Render Blueprint for backend/worker/Postgres/Redis deployment
- Vite React frontend deployable to Vercel or Render

> The default `DEMO_MODE=true` generates a real MP4 from the uploaded image with camera motion. It is intentionally a deterministic fallback, not a claim of full AI character animation. Set `DEMO_MODE=false` and configure an AI provider/model adapter for real image-to-video generation.

## Architecture

React + TypeScript + Tailwind -> FastAPI -> PostgreSQL/Redis -> Celery -> Animation Provider -> FFmpeg -> Object storage/local storage.

## Local setup

### 1. Requirements

- Docker Desktop
- Git
- Node.js 20+

### 2. Start backend, worker, database and Redis

```bash
docker compose up --build
```

API: http://localhost:8000
API docs: http://localhost:8000/docs

### 3. Start frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend: http://localhost:5173

### 4. Demo generation

The default environment uses:

```env
DEMO_MODE=true
```

Upload a JPG/PNG/WEBP image, enter a prompt, and generate. The worker renders a short MP4 with subtle zoom/pan motion using FFmpeg.

## Real AI provider

The backend exposes a provider interface at `backend/app/services/animation_provider.py`. Configure an HTTP-compatible provider with:

```env
DEMO_MODE=false
AI_PROVIDER_URL=https://your-provider.example/v1/generate
AI_PROVIDER_TOKEN=...
AI_PROVIDER_MODEL=your-model
```

The provider is expected to accept JSON containing `image_data_uri`, `prompt`, `duration`, and optional `dialogue`, and return a JSON object containing either `video_url` or `video_data_base64`.

This keeps the application independent of a specific vendor/model. Current image-to-video providers and model schemas change frequently, so the model-specific adapter should be implemented separately.

## GitHub

```bash
git init
git add .
git commit -m "Initial AI Animation Video Maker MVP"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/ai-animation-video-maker.git
git push -u origin main
```

## Deployment

Recommended split deployment:

- Frontend: Vercel or Render Static Site
- API: Render Web Service
- Worker: Render Background Worker
- PostgreSQL: Render Postgres
- Redis: Render Key Value / Redis-compatible service
- Production media: S3/R2/GCS rather than local filesystem

See `render.yaml` for the infrastructure blueprint.

## GitHub + deployment checklist

1. Create an empty GitHub repository named `ai-animation-video-maker`.
2. In this folder run:

```bash
git init
git branch -M main
git add .
git commit -m "Initial AI Animation Video Maker MVP"
git remote add origin https://github.com/YOUR_USERNAME/ai-animation-video-maker.git
git push -u origin main
```

3. For the backend stack, connect the repository to Render and create the Blueprint from `render.yaml`. Render supports Git-based deploys and Blueprint infrastructure as code. The repository contains a web service, background worker, Key Value service and Postgres database definition.
4. Set `CORS_ORIGINS` to the final frontend URL in the Render backend environment.
5. Set `VITE_API_URL` in the frontend hosting environment to the Render API URL, then deploy the `frontend` directory as a Vite app.
6. For production video/media persistence, replace local `storage/` with S3/R2/GCS before relying on generated files across deploys.

### GitHub repository commands

If Git reports `Author identity unknown`, configure your local Git identity once:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```
