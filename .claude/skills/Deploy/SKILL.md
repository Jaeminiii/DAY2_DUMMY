---
name: Deploy
description: Docker 컨테이너화 및 배포 설정. Docker Compose, CI/CD 파이프라인
---

# 배포 가이드

## 개요
애플리케이션을 Docker 컨테이너로 패키징하고 배포합니다.

## Docker 컨테이너화

### 백엔드 Dockerfile
```dockerfile
# backend/Dockerfile
FROM python:3.12-slim

WORKDIR /app

# 의존성 설치
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 애플리케이션 코드 복사
COPY ./app ./app
COPY alembic.ini .
COPY alembic ./alembic

# 포트 노출
EXPOSE 8000

# 실행
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 프론트엔드 Dockerfile
```dockerfile
# frontend/Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# 의존성 설치
COPY package*.json ./
RUN npm ci

# 빌드
COPY . .
RUN npm run build

# 프로덕션 이미지
FROM node:20-alpine

WORKDIR /app

COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3000

CMD ["npm", "start"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  # PostgreSQL
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      SECRET_KEY: ${SECRET_KEY}
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy
    command: >
      sh -c "alembic upgrade head &&
             uvicorn app.main:app --host 0.0.0.0 --port 8000"

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    depends_on:
      - backend
    environment:
      NEXT_PUBLIC_API_URL: http://backend:8000

volumes:
  postgres_data:
```

### .dockerignore
```
# Backend
backend/__pycache__
backend/.venv
backend/venv
backend/*.db
backend/.env

# Frontend
frontend/node_modules
frontend/.next
frontend/.env.local
frontend/.env

# Common
.git
.gitignore
*.md
.DS_Store
```

## 로컬 실행

### 개발 환경
```bash
# .env 파일 생성
cp .env.example .env

# Docker Compose로 실행
docker-compose up -d

# 로그 확인
docker-compose logs -f

# 중지
docker-compose down

# 볼륨까지 삭제
docker-compose down -v
```

## CI/CD 파이프라인

### GitHub Actions
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: |
          cd backend
          pip install -r requirements.txt

      - name: Run tests
        run: |
          cd backend
          pytest

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push backend
        uses: docker/build-push-action@v4
        with:
          context: ./backend
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/myapp-backend:latest

      - name: Build and push frontend
        uses: docker/build-push-action@v4
        with:
          context: ./frontend
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/myapp-frontend:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /app
            docker-compose pull
            docker-compose up -d
            docker-compose exec -T backend alembic upgrade head
```

## 배포 플랫폼

### 1. Vercel (프론트엔드)
```bash
# Vercel CLI 설치
npm i -g vercel

# 배포
cd frontend
vercel --prod
```

**vercel.json:**
```json
{
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://your-backend.com/api/:path*"
    }
  ]
}
```

### 2. Railway (백엔드 + DB)
```bash
# Railway CLI 설치
npm i -g @railway/cli

# 로그인
railway login

# 프로젝트 초기화
railway init

# 배포
railway up
```

### 3. Fly.io (백엔드)
```bash
# Fly CLI 설치
curl -L https://fly.io/install.sh | sh

# 로그인
fly auth login

# 앱 생성
fly launch

# 배포
fly deploy
```

**fly.toml:**
```toml
app = "myapp-backend"
primary_region = "nrt"  # Tokyo

[build]
  dockerfile = "Dockerfile"

[env]
  PORT = "8000"

[[services]]
  internal_port = 8000
  protocol = "tcp"

  [[services.ports]]
    port = 80
    handlers = ["http"]

  [[services.ports]]
    port = 443
    handlers = ["tls", "http"]
```

### 4. AWS EC2 (프로덕션)
```bash
# SSH 접속
ssh -i key.pem ubuntu@ec2-instance

# Docker 설치
sudo apt update
sudo apt install docker.io docker-compose

# 코드 clone
git clone https://github.com/your/repo.git
cd repo

# 환경 변수 설정
cp .env.example .env
nano .env

# 실행
docker-compose up -d

# Nginx 설정
sudo apt install nginx
sudo nano /etc/nginx/sites-available/default
```

**Nginx 설정:**
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    # Frontend
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API
    location /api {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## HTTPS 설정 (Let's Encrypt)
```bash
# Certbot 설치
sudo apt install certbot python3-certbot-nginx

# SSL 인증서 발급
sudo certbot --nginx -d yourdomain.com

# 자동 갱신 설정 (cron)
sudo crontab -e
# 추가: 0 0 * * * certbot renew --quiet
```

## 환경별 설정

### 개발
```bash
ENVIRONMENT=development
DEBUG=True
DATABASE_URL=sqlite:///./dev.db
```

### 스테이징
```bash
ENVIRONMENT=staging
DEBUG=True
DATABASE_URL=postgresql://user:pass@staging-db/db
```

### 프로덕션
```bash
ENVIRONMENT=production
DEBUG=False
DATABASE_URL=postgresql://user:pass@prod-db/db
```

## 모니터링

### Docker 상태 확인
```bash
# 컨테이너 상태
docker ps

# 로그
docker logs -f container_name

# 리소스 사용량
docker stats

# 컨테이너 접속
docker exec -it container_name bash
```

### 헬스체크
```python
# backend/app/main.py
@app.get("/health")
def health_check():
    return {"status": "healthy"}
```

## 백업

### 데이터베이스 백업
```bash
# PostgreSQL 백업
docker exec postgres_container pg_dump -U user dbname > backup.sql

# 복원
docker exec -i postgres_container psql -U user dbname < backup.sql

# 자동 백업 (cron)
0 2 * * * docker exec postgres_container pg_dump -U user dbname > /backup/$(date +\%Y\%m\%d).sql
```

## 참조 문서
- [docker-setup.md](references/docker-setup.md) - Docker 상세 설정
- [platforms.md](references/platforms.md) - 배포 플랫폼 비교
- [ci-cd.md](references/ci-cd.md) - CI/CD 설정
