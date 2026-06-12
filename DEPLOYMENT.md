# Deployment Information

## Public URL
https://lab12-production-agent.up.railway.app

## Platform
Railway

## Test Commands

### Health Check
```bash
curl https://lab12-production-agent.up.railway.app/health
# Expected: {"status":"ok","version":"1.0.0",...}
```

### API Test (with authentication)
```bash
curl -X POST https://lab12-production-agent.up.railway.app/ask \
  -H "X-API-Key: dev-key-change-me" \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello"}'
```

## Environment Variables Set
- `PORT` = `8000`
- `REDIS_URL` = `redis://your-redis-railway-url:6379`
- `AGENT_API_KEY` = `dev-key-change-me`
- `LOG_LEVEL` = `INFO`

## Screenshots
*(Hãy chụp ảnh màn hình và lưu vào thư mục `screenshots/` rồi cập nhật link dưới đây)*
- [Deployment dashboard](screenshots/dashboard.png)
- [Service running](screenshots/running.png)
- [Test results](screenshots/test.png)
