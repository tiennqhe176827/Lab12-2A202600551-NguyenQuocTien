# Day 12 Lab - Mission Answers

**Student Name:** Nguyễn Quốc Tiết
**Student ID:** _________________________
**Date:** 12/06/2026

---

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found (5+ issues)

1. **API key hardcode trong code** — `OPENAI_API_KEY = "sk-hardcoded-fake-key-never-do-this"` và `DATABASE_URL` chứa password. Nếu push lên GitHub → secret bị lộ.
2. **Port cố định** — `port=8000` cứng, không đọc từ environment variable. Trên Railway/Render, PORT được inject tự động.
3. **Debug mode = True** — `reload=True` trong production gây restart liên tục và exposed error details.
4. **Không có health check endpoint** — Platform không biết agent có còn sống không → không auto restart khi crash.
5. **Không xử lý shutdown** — Không có graceful shutdown → request đang xử lý bị cắt ngang khi container stop.
6. **Print thay vì structured logging** — `print()` không parse được trong log aggregator.
7. **Host = localhost** — Không bind `0.0.0.0` → không nhận kết nối từ bên ngoài container.

### Exercise 1.3: Comparison table

| Feature | Basic (Develop) | Advanced (Production) | Tại sao quan trọng? |
|---------|----------------|----------------------|---------------------|
| Config | Hardcode trong code | Environment variables | Bảo mật secret, linh hoạt giữa env |
| Health check | Không có | `/health` + `/ready` | Platform auto restart khi crash |
| Logging | `print()` | JSON structured | Dễ parse, search trong log aggregator |
| Shutdown | Đột ngột | Graceful (lifespan) | Hoàn thành request trước khi tắt |
| Port | Hardcode 8000 | Từ env `PORT` | Railway/Render inject PORT tự động |
| Host | localhost | 0.0.0.0 | Nhận kết nối từ bên ngoài container |
| Debug | Luôn True | Từ env `DEBUG` | Không expose error trong production |
| CORS | Không có | Configurable origins | Bảo mật, tránh CORS attack |

---

## Part 2: Docker

### Exercise 2.1: Dockerfile questions

1. **Base image:** `python:3.11` (develop) / `python:3.11-slim` (production)
2. **Working directory:** `/app`
3. **COPY requirements.txt trước:** Để tận dụng Docker layer cache — khi code thay đổi nhưng dependencies không đổi, Docker không cần reinstall.
4. **CMD vs ENTRYPOINT:** CMD là default command, có thể override khi `docker run`. ENTRYPOINT là executable chính, args từ CMD được append vào.

### Exercise 2.3: Image size comparison

- **Develop (single-stage, python:3.11):** ~900 MB
- **Production (multi-stage, python:3.11-slim):** ~150-200 MB
- **Giảm:** ~78% — vì bỏ build tools (gcc, headers) ở stage 2

---

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment

- **URL:** https://your-app.up.railway.app (thay bằng URL thật sau khi deploy)
- **Platform:** Railway
- **Environment variables đã set:**
  - `PORT=8000`
  - `AGENT_API_KEY=my-secret-key`
  - `ENVIRONMENT=production`

### Exercise 3.2: render.yaml vs railway.toml

| Feature | railway.toml | render.yaml |
|---------|-------------|-------------|
| Syntax | TOML | YAML |
| Build | `builder = "DOCKERFILE"` | `runtime: docker` |
| Health check | `healthcheckPath = "/health"` | `healthCheckPath: /health` |
| Deploy | `startCommand` | `autoDeploy: true` |
| Env vars | `railway variables set` | Define trong `envVars` |

---

## Part 4: API Security

### Exercise 4.1: API Key Authentication

- **API key check:** Tại `verify_api_key()` — so sánh header `X-API-Key` với `settings.agent_api_key`
- **Sai key:** Trả về HTTP 401 "Invalid or missing API key"
- **Rotate key:** Thay env var `AGENT_API_KEY` trên Railway dashboard → restart service

### Exercise 4.3: Rate Limiting

- **Algorithm:** Sliding Window Counter — đếm request trong cửa sổ 60 giây
- **Limit:** 20 requests/minute (configurable qua env `RATE_LIMIT_PER_MINUTE`)
- **Bypass admin:** Không có bypass — tất cả users chung limit (có thể mở rộng)

### Exercise 4.4: Cost Guard implementation

```python
def check_and_record_cost(input_tokens: int, output_tokens: int):
    # Tính chi phí: input $0.15/1M tokens, output $0.60/1M tokens
    cost = (input_tokens / 1000) * 0.00015 + (output_tokens / 1000) * 0.0006
    # Reset daily
    # Raise 503 nếu vượt daily budget ($5/ngày)
```

---

## Part 5: Scaling & Reliability

### Exercise 5.1: Health Checks

```python
@app.get("/health")
def health():
    # Liveness probe — return 200 nếu process OK
    return {"status": "ok", "uptime_seconds": ...}

@app.get("/ready")
def ready():
    # Readiness probe — check dependencies
    # Return 503 nếu chưa ready
    if not _is_ready:
        raise HTTPException(503, "Not ready")
    return {"ready": True}
```

### Exercise 5.2: Graceful Shutdown

```python
signal.signal(signal.SIGTERM, handle_sigterm)

def handle_sigterm(signum, frame):
    # 1. Đánh dấu is_ready = False (không nhận traffic mới)
    # 2. Chờ in-flight requests hoàn thành (tối đa 30s)
    # 3. Đóng connections
    # 4. Exit
```

### Exercise 5.3: Stateless Design

**Anti-pattern (memory):**
```python
conversation_history = {}  # Mỗi instance có memory riêng → mất data khi scale
```

**Correct (Redis):**
```python
# Bất kỳ instance nào cũng đọc được session từ Redis
history = r.lrange(f"history:{user_id}", 0, -1)
```

---

## Part 6: Final Project

### Checklist

- [x] Agent trả lời câu hỏi qua REST API (`POST /ask`)
- [x] API key authentication (`X-API-Key` header)
- [x] Rate limiting (20 req/min)
- [x] Cost guard ($5/day)
- [x] Health check endpoint (`GET /health`)
- [x] Readiness check endpoint (`GET /ready`)
- [x] Graceful shutdown (SIGTERM)
- [x] Structured JSON logging
- [x] Config từ environment variables
- [x] Multi-stage Dockerfile
- [x] Non-root user trong Docker
- [x] HEALTHCHECK instruction trong Dockerfile
- [x] Deploy lên Railway
- [x] Public URL hoạt động
