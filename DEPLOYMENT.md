#  Delivery Checklist — Day 12 Lab Submission

> **Student Name:** Phan Tan Hung
> **Student ID:** 2A202600825
> **Date:** 12/06/2026

---
# Deployment Information (Lab 12 Basic)

## Public URL
https://lab12deloy-production.up.railway.app

## Platform
Railway

## Test Commands

### Health Check
```bash
curl https://lab12deloy-production.up.railway.app/health
# Expected: {"status":"ok","version":"1.0.0",...}
```

### API Test (with authentication)
```bash
curl -X POST https://lab12deloy-production.up.railway.app/ask \
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

---
# ShopeeFood AI Deployment Information

## Public URL
https://trustworthy-light-production-e86f.up.railway.app

## Platform
Railway

## Test Commands

### Health Check
```bash
curl https://trustworthy-light-production-e86f.up.railway.app/api/health
# Expected: {"status":"healthy","service":"ShopeeFood AI API",...}
```

### API Test (Chat)
```bash
curl -X POST https://trustworthy-light-production-e86f.up.railway.app/api/chat \
  -H "Content-Type: application/json" \
  -d '{"user_id": "usr_student_01", "message": "Hello"}'
```

## Environment Variables Set
- `PORT` = `8000`
- `GEMINI_API_KEY` = `(Your Gemini API Key)`
