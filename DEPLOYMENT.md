# Deployment Information

## Public URL

```
https://your-app.up.railway.app
```

> **Note:** Thay URL bằng URL thật sau khi deploy xong.
> Lấy URL bằng lệnh: `railway domain`

## Platform

Railway

## Test Commands

### Health Check

```bash
curl https://your-app.up.railway.app/health
# Expected: {"status":"ok","version":"1.0.0",...}
```

### Readiness Check

```bash
curl https://your-app.up.railway.app/ready
# Expected: {"ready":true}
```

### API Test (without key — should return 401)

```bash
curl -X POST https://your-app.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello"}'
# Expected: 401 Unauthorized
```

### API Test (with key — should return 200)

```bash
curl -X POST https://your-app.up.railway.app/ask \
  -H "X-API-Key: my-secret-key" \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello"}'
# Expected: 200 OK with answer
```

### Rate Limiting Test

```bash
# Gọi liên tục 25 lần — request thứ 21+ sẽ bị 429
for i in {1..25}; do
  curl -s -X POST https://your-app.up.railway.app/ask \
    -H "X-API-Key: my-secret-key" \
    -H "Content-Type: application/json" \
    -d "{\"question\": \"Test $i\"}"
  echo ""
done
```

## Environment Variables Set

- `PORT=8000`
- `AGENT_API_KEY=my-secret-key`
- `ENVIRONMENT=production`
- `REDIS_URL=redis://...` (tự Railway inject khi link Redis service)

## Architecture

```
Client → Railway (public URL) → Agent Service → Mock LLM
                                     ↓
                                  Redis (session/state)
```

## Screenshots

- [ ] Railway dashboard
- [ ] Service running
- [ ] Health check response
- [ ] API test response
