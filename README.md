## 🏗️ System Architecture Overview

### 🧠 Data → Retrieval / Inference → Serving
- **Data**: Doctor/patient consultation notes stored temporarily (JSON → DB/S3).
- **Retrieval / Inference**: FastAPI backend receives input → formats system & user prompts → streams OpenAI model responses (SSE).
- **Serving**: Next.js frontend served via FastAPI static files → listens to real-time summary updates through `text/event-stream`.

---

## ⚙️ Key Metrics
| Metric | Description | Target |
|--------|--------------|---------|
| **p95 Latency** | End-to-end response | 1200–1800 ms |
| **Cost / Request** | OpenAI token + infra | ~$0.01–$0.05 |
| **Quality / Eval** | Human clarity score / ROUGE-L | ≥ 4.5 / 5 |

---

## ☁️ Cloud Infrastructure & Tools
- **Compute / Hosting:** AWS App Runner (Docker container hosting)  
- **Registry:** Amazon ECR (image storage)  
- **CI/CD:** GitHub Actions → build & push → auto-deploy to App Runner  
- **Secrets:** AWS Secrets Manager / App Runner environment variables  
- **Auth:** Clerk for authentication  
- **AI Engine:** OpenAI API (chat completion + streaming)  
- **Monitoring:** AWS CloudWatch (logs, metrics)

---

## 🚀 CI/CD & MLOps Flow
1. **Dev push → GitHub Actions** builds Docker image (`--platform linux/amd64` for AWS).  
2. Runs lint/tests → pushes to **ECR**.  
3. **App Runner** auto-deploys latest tag.  
4. Live logs via **CloudWatch**; health checks monitor container readiness.  
5. Model prompt updates tested offline → pushed via same pipeline.

---

## 🧩 Postmortem – What Broke & How It Was Fixed

**Symptom:**  
After deploying to AWS App Runner, the *Generate Summary* feature failed with:  
`SSE error: Expected content-type to be text/event-stream, Actual: application/json`  
and `POST /api/consultation → 403 Forbidden` or `404 Not Found`.

**Root Causes:**
1. Missing `OPENAI_API_KEY` and `CLERK_SECRET_KEY` in App Runner environment.  
2. FastAPI endpoint returning JSON/HTML instead of `text/event-stream`.  
3. Build mismatch between local and container (Apple Silicon vs AWS).  
4. Clerk publishable key not embedded at build time in Next.js.

**Fixes:**
- Added required env vars (`CLERK_JWKS_URL`, `CLERK_SECRET_KEY`, `OPENAI_API_KEY`).  
- Ensured `StreamingResponse(..., media_type="text/event-stream")` for summary route.  
- Rebuilt Docker image with `--platform linux/amd64` before pushing to ECR.  
- Configured health check route `/health` in App Runner.  
- Verified CloudWatch logs to confirm streaming and SSE handshake.

✅ **Result:** Summary generation now streams correctly; deployment is stable and reproducible through CI/CD.

---

## 🧭 Quick Diagram
<img width="528" height="483" alt="Tech-Stack-Visual-10-06-2025_09_24_AM" src="https://github.com/user-attachments/assets/21dab3f4-0ed2-4daa-96a4-4fce870ec2cf" />

> 🧑‍💻 Built and deployed by **Priyanka Sharma** — AI-powered Healthcare Consultation SaaS App



