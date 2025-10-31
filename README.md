# CI/CD Pipeline for Next.js, Node.js & React.js Applications

![Author](https://img.shields.io/badge/Author-Jaydip%20Khunt-blue)
![Date](https://img.shields.io/badge/Date-October%202025-green)
![Guide](https://img.shields.io/badge/Type-Deployment%20Guide-orange)

![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![DigitalOcean](https://img.shields.io/badge/DigitalOcean-0080FF?style=for-the-badge&logo=digitalocean&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=next.js&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

A comprehensive guide for setting up automated CI/CD pipelines to deploy Next.js, Node.js, and React.js applications to VPS servers and DigitalOcean using GitHub Actions, GitLab CI/CD, and Docker containers.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Pipeline Architecture](#pipeline-architecture)
- [GitHub Actions Setup](#github-actions-setup)
- [DigitalOcean Deployment](#digitalocean-deployment)
- [VPS Server Configuration](#vps-server-configuration)
- [Docker Containerization](#docker-containerization)
- [Next.js Specific Configuration](#nextjs-specific-configuration)
- [React.js Application Pipeline](#reactjs-application-pipeline)
- [Node.js API Deployment](#nodejs-api-deployment)
- [GitLab CI/CD Alternative](#gitlab-cicd-alternative)
- [Environment Management](#environment-management)
- [Monitoring & Logging](#monitoring--logging)
- [Security Best Practices](#security-best-practices)
- [Troubleshooting](#troubleshooting)

## Overview

This guide provides complete automation for deploying modern JavaScript applications using industry-standard CI/CD practices. The pipeline supports:

- **Continuous Integration**: Automated testing, linting, and building
- **Continuous Deployment**: Automatic deployment to staging and production
- **Multi-environment**: Development, staging, and production environments
- **Rollback capabilities**: Quick rollback to previous versions
- **Security scanning**: Automated vulnerability checks
- **Performance monitoring**: Build and deployment metrics

### Supported Technologies
- **Frontend**: Next.js, React.js, Vue.js
- **Backend**: Node.js, Express.js, Nest.js
- **Databases**: MongoDB, PostgreSQL, MySQL
- **Infrastructure**: DigitalOcean, VPS, AWS, Docker

---

## Prerequisites

### Required Accounts & Tools
- GitHub or GitLab account
- DigitalOcean account (or VPS provider)
- Docker Hub account (optional)
- Domain name (recommended)

### Local Development Environment
```bash
# Required tools
Node.js >= 18.0.0
npm or yarn
Git
Docker (optional)
SSH client
```

### Server Requirements
- **OS**: Ubuntu 20.04+ or CentOS 8+
- **RAM**: Minimum 2GB (4GB+ recommended)
- **Storage**: At least 20GB SSD
- **Network**: Public IP address

---

## Pipeline Architecture

### CI/CD Workflow Overview
```
Developer Push → GitHub → GitHub Actions → Build → Test → Deploy → Monitor
                    ↓
              Trigger Webhook
                    ↓
              Build Docker Image
                    ↓
              Run Automated Tests
                    ↓
              Security Scanning
                    ↓
              Deploy to Staging
                    ↓
              Integration Tests
                    ↓
              Deploy to Production
                    ↓
              Health Checks
```

### Pipeline Stages
1. **Source Control Trigger**: Push or PR to main branch
2. **Build Stage**: Install dependencies and build application
3. **Test Stage**: Unit tests, integration tests, linting
4. **Security Stage**: Vulnerability scanning, SAST analysis
5. **Build Image**: Create Docker container (optional)
6. **Deploy Stage**: Deploy to staging/production
7. **Verification**: Health checks and smoke tests
8. **Notification**: Slack/email notifications

---

## GitHub Actions Setup

### Repository Structure
```
your-app/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy-staging.yml
│       └── deploy-production.yml
├── docker/
│   ├── Dockerfile
│   ├── Dockerfile.production
│   └── docker-compose.yml
├── scripts/
│   ├── deploy.sh
│   ├── health-check.sh
│   └── rollback.sh
├── src/
├── package.json
└── README.md
```

### Main CI Workflow (.github/workflows/ci.yml)
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Continuous Integration
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: |
        npm ci

    - name: Run linting
      run: |
        npm run lint

    - name: Run unit tests
      run: |
        npm run test:unit

    - name: Run integration tests
      run: |
        npm run test:integration

    - name: Generate test coverage
      run: |
        npm run test:coverage

    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/coverage.xml

  # Security Scanning
  security:
    runs-on: ubuntu-latest
    needs: test

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Run security audit
      run: |
        npm audit --audit-level=high

    - name: SAST Scan
      uses: github/super-linter@v4
      env:
        DEFAULT_BRANCH: main
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # Build Application
  build:
    runs-on: ubuntu-latest
    needs: [test, security]

    outputs:
      image-digest: ${{ steps.build.outputs.digest }}

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Build application
      run: |
        npm run build

    - name: Create build artifact
      uses: actions/upload-artifact@v3
      with:
        name: build-files
        path: |
          build/
          dist/
          .next/
        retention-days: 30

    # Docker Build (Optional)
    - name: Set up Docker Buildx
      if: github.ref == 'refs/heads/main'
      uses: docker/setup-buildx-action@v2

    - name: Login to Container Registry
      if: github.ref == 'refs/heads/main'
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push Docker image
      if: github.ref == 'refs/heads/main'
      uses: docker/build-push-action@v4
      id: build
      with:
        context: .
        file: ./docker/Dockerfile.production
        push: true
        tags: |
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

### Staging Deployment (.github/workflows/deploy-staging.yml)
```yaml
name: Deploy to Staging

on:
  workflow_run:
    workflows: ["CI/CD Pipeline"]
    branches: [develop]
    types: [completed]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}

    environment:
      name: staging
      url: https://staging.your-domain.com

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Download build artifacts
      uses: actions/download-artifact@v3
      with:
        name: build-files

    - name: Deploy to DigitalOcean App Platform
      uses: digitalocean/app_action@v1.1.5
      with:
        app_name: your-app-staging
        token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}

    - name: Deploy to VPS (Alternative)
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.STAGING_HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/staging
          git pull origin develop
          npm ci --only=production
          npm run build
          pm2 restart staging-app

    - name: Health Check
      run: |
        curl -f https://staging.your-domain.com/health || exit 1

    - name: Notify Slack
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        channel: '#deployments'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Production Deployment (.github/workflows/deploy-production.yml)
```yaml
name: Deploy to Production

on:
  release:
    types: [published]
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to deploy'
        required: true
        default: 'latest'

jobs:
  deploy-production:
    runs-on: ubuntu-latest

    environment:
      name: production
      url: https://your-domain.com

    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        ref: ${{ github.event.release.tag_name || github.event.inputs.version }}

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci --only=production

    - name: Build production
      env:
        NODE_ENV: production
      run: npm run build

    - name: Deploy to DigitalOcean
      run: |
        # Deploy script here
        ./scripts/deploy.sh production

    - name: Blue-Green Deployment
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.PRODUCTION_HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /opt/deployment
          ./blue-green-deploy.sh ${{ github.sha }}

    - name: Health Check
      run: |
        for i in {1..5}; do
          if curl -f https://your-domain.com/health; then
            echo "Health check passed"
            break
          fi
          echo "Attempt $i failed, retrying..."
          sleep 10
        done

    - name: Rollback on failure
      if: failure()
      run: |
        ./scripts/rollback.sh
```

---

## DigitalOcean Deployment

### DigitalOcean App Platform

#### App Specification (.do/app.yaml)
```yaml
name: your-nextjs-app

services:
  - name: web
    source_dir: /
    github:
      repo: your-username/your-repo
      branch: main
      deploy_on_push: true

    run_command: npm start
    build_command: npm run build

    environment_slug: node-js
    instance_count: 1
    instance_size_slug: basic-xxs

    envs:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        value: ${DATABASE_URL}
      - key: API_SECRET
        value: ${API_SECRET}

    health_check:
      http_path: /health
      initial_delay_seconds: 60
      period_seconds: 10
      timeout_seconds: 5
      failure_threshold: 3

    domains:
      - domain: your-domain.com
        type: PRIMARY
      - domain: www.your-domain.com
        type: ALIAS

databases:
  - name: main-db
    engine: PG
    size: basic-xs

static_sites:
  - name: frontend
    source_dir: /frontend
    build_command: npm run build
    output_dir: /build

    routes:
      - path: /api
        preserve_path_prefix: true
```

### DigitalOcean Droplet Deployment

#### Server Setup Script (scripts/setup-server.sh)
```bash
#!/bin/bash

# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install PM2
sudo npm install -g pm2

# Install Nginx
sudo apt install -y nginx

# Install Docker
sudo apt install -y docker.io
sudo systemctl enable docker
sudo usermod -aG docker $USER

# Create app directory
sudo mkdir -p /var/www/app
sudo chown $USER:$USER /var/www/app

# Setup firewall
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw --force enable

# Setup SSL with Let's Encrypt
sudo apt install -y certbot python3-certbot-nginx

echo "Server setup completed!"
```

#### Deployment Script (scripts/deploy.sh)
```bash
#!/bin/bash

set -e

ENVIRONMENT=${1:-staging}
APP_DIR="/var/www/app"
BACKUP_DIR="/var/backups/app"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "Starting deployment to $ENVIRONMENT..."

# Create backup
mkdir -p $BACKUP_DIR
if [ -d "$APP_DIR" ]; then
    echo "Creating backup..."
    tar -czf "$BACKUP_DIR/backup_$TIMESTAMP.tar.gz" -C "$APP_DIR" .
fi

# Pull latest code
cd $APP_DIR
git fetch origin
git reset --hard origin/main

# Install dependencies
npm ci --only=production

# Build application
NODE_ENV=production npm run build

# Update PM2
pm2 reload ecosystem.config.js --env $ENVIRONMENT

# Test deployment
sleep 5
if curl -f http://localhost:3000/health; then
    echo "Deployment successful!"

    # Clean up old backups (keep last 5)
    cd $BACKUP_DIR
    ls -t backup_*.tar.gz | tail -n +6 | xargs -r rm
else
    echo "Deployment failed! Rolling back..."
    tar -xzf "$BACKUP_DIR/backup_$TIMESTAMP.tar.gz" -C "$APP_DIR"
    pm2 reload ecosystem.config.js --env $ENVIRONMENT
    exit 1
fi
```

#### Nginx Configuration
```nginx
upstream app {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001 backup;
}

server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;

    # Gzip compression
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

    location / {
        proxy_pass http://app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

---

## VPS Server Configuration

### PM2 Ecosystem Configuration (ecosystem.config.js)
```javascript
module.exports = {
  apps: [{
    name: 'app-production',
    script: './server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    env_staging: {
      NODE_ENV: 'staging',
      PORT: 3001
    },
    // Logging
    log_file: './logs/app.log',
    error_file: './logs/error.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm Z',

    // Restart configuration
    max_restarts: 3,
    min_uptime: '10s',
    max_memory_restart: '500M',

    // Advanced features
    watch: false,
    ignore_watch: ['node_modules', 'logs'],

    // Health monitoring
    health_check_grace_period: 3000,
    health_check_fatal_exceptions: true
  }],

  deploy: {
    production: {
      user: 'deploy',
      host: 'your-server.com',
      ref: 'origin/main',
      repo: 'git@github.com:your-username/your-repo.git',
      path: '/var/www/app',
      'pre-deploy-local': '',
      'post-deploy': 'npm install && npm run build && pm2 reload ecosystem.config.js --env production',
      'pre-setup': ''
    },

    staging: {
      user: 'deploy',
      host: 'staging.your-server.com',
      ref: 'origin/develop',
      repo: 'git@github.com:your-username/your-repo.git',
      path: '/var/www/staging',
      'post-deploy': 'npm install && npm run build && pm2 reload ecosystem.config.js --env staging'
    }
  }
};
```

### Blue-Green Deployment Script
```bash
#!/bin/bash

# Blue-Green Deployment Script
set -e

NEW_VERSION=$1
BLUE_PORT=3000
GREEN_PORT=3001
CURRENT_COLOR=$(curl -s http://localhost/api/version | jq -r '.color' || echo 'blue')

if [ "$CURRENT_COLOR" == "blue" ]; then
    DEPLOY_PORT=$GREEN_PORT
    DEPLOY_COLOR="green"
    CURRENT_PORT=$BLUE_PORT
else
    DEPLOY_PORT=$BLUE_PORT
    DEPLOY_COLOR="blue"
    CURRENT_PORT=$GREEN_PORT
fi

echo "Current: $CURRENT_COLOR:$CURRENT_PORT, Deploying to: $DEPLOY_COLOR:$DEPLOY_PORT"

# Deploy new version
cd /opt/app-$DEPLOY_COLOR
git fetch && git reset --hard $NEW_VERSION
npm ci --only=production
npm run build

# Update environment
export PORT=$DEPLOY_PORT
export NODE_ENV=production
export VERSION=$NEW_VERSION
export COLOR=$DEPLOY_COLOR

# Start new version
pm2 start ecosystem.config.js --name app-$DEPLOY_COLOR

# Health check
sleep 10
if curl -f "http://localhost:$DEPLOY_PORT/health"; then
    echo "New version healthy, switching traffic..."

    # Update nginx upstream
    sed -i "s/server 127.0.0.1:$CURRENT_PORT/server 127.0.0.1:$DEPLOY_PORT/" /etc/nginx/sites-enabled/default
    nginx -t && systemctl reload nginx

    # Stop old version
    sleep 5
    pm2 stop app-$CURRENT_COLOR

    echo "Deployment completed successfully!"
else
    echo "Health check failed, rolling back..."
    pm2 stop app-$DEPLOY_COLOR
    exit 1
fi
```

---

## Docker Containerization

### Multi-stage Dockerfile
```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM node:18-alpine AS production

# Create app user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

WORKDIR /app

# Copy built application
COPY --from=builder --chown=nextjs:nodejs /app/build ./build
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/package*.json ./

# Security updates
RUN apk upgrade --no-cache

USER nextjs

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["npm", "start"]
```

### Docker Compose for Development
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=mongodb://db:27017/app
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - db
      - redis

  db:
    image: mongo:5
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app

volumes:
  mongo_data:
  redis_data:
```

### Docker Production Compose
```yaml
version: '3.8'

services:
  app:
    image: your-registry/your-app:${VERSION:-latest}
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - API_SECRET=${API_SECRET}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
```

---

## Next.js Specific Configuration

### Next.js Production Configuration
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Output configuration
  output: 'standalone', // For Docker deployments

  // Performance optimizations
  swcMinify: true,
  compress: true,

  // Image optimization
  images: {
    domains: ['your-cdn.com'],
    formats: ['image/webp', 'image/avif'],
  },

  // Environment variables
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },

  // Headers for security
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'SAMEORIGIN',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin',
          },
        ],
      },
    ];
  },

  // Redirects
  async redirects() {
    return [
      {
        source: '/old-path',
        destination: '/new-path',
        permanent: true,
      },
    ];
  },

  // Webpack configuration
  webpack: (config, { isServer }) => {
    // Custom webpack config
    if (!isServer) {
      config.resolve.fallback.fs = false;
    }
    return config;
  },
};

module.exports = nextConfig;
```

### Next.js Docker Optimization
```dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Copy package files
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Disable telemetry
ENV NEXT_TELEMETRY_DISABLED 1

RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy built application
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

---

## React.js Application Pipeline

### React Build and Deploy Workflow
```yaml
name: React App Deployment

on:
  push:
    branches: [main]
    paths: ['frontend/**']

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: ./frontend

    steps:
    - uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: frontend/package-lock.json

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test -- --coverage --watchAll=false

    - name: Build for production
      env:
        CI: false
        REACT_APP_API_URL: ${{ secrets.REACT_APP_API_URL }}
        REACT_APP_VERSION: ${{ github.sha }}
      run: npm run build

    - name: Deploy to S3 (Option 1)
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Upload to S3
      run: |
        aws s3 sync build/ s3://${{ secrets.S3_BUCKET_NAME }} --delete
        aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"

    - name: Deploy to VPS (Option 2)
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/react-app
          git pull origin main
          cd frontend
          npm ci
          npm run build
          sudo cp -r build/* /var/www/html/
          sudo systemctl reload nginx
```

### React Application Dockerfile
```dockerfile
# Multi-stage build for React app
FROM node:18-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Production stage with nginx
FROM nginx:alpine

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Copy built app
COPY --from=build /app/build /usr/share/nginx/html

# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### React Environment Configuration
```javascript
// .env.production
REACT_APP_API_URL=https://api.your-domain.com
REACT_APP_VERSION=$npm_package_version
REACT_APP_BUILD_DATE=$BUILD_DATE
REACT_APP_SENTRY_DSN=$SENTRY_DSN
GENERATE_SOURCEMAP=false

// .env.staging
REACT_APP_API_URL=https://api-staging.your-domain.com
REACT_APP_VERSION=staging-$GITHUB_SHA
GENERATE_SOURCEMAP=true
```

---

## Node.js API Deployment

### Node.js API Workflow
```yaml
name: Node.js API Deployment

on:
  push:
    branches: [main]
    paths: ['api/**']

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mongodb:
        image: mongo:5
        ports:
          - 27017:27017
      redis:
        image: redis:7
        ports:
          - 6379:6379

    defaults:
      run:
        working-directory: ./api

    steps:
    - uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: api/package-lock.json

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run unit tests
      run: npm run test:unit
      env:
        MONGODB_URL: mongodb://localhost:27017/test
        REDIS_URL: redis://localhost:6379

    - name: Run integration tests
      run: npm run test:integration
      env:
        MONGODB_URL: mongodb://localhost:27017/test
        REDIS_URL: redis://localhost:6379

  deploy:
    needs: test
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: ./api
        push: true
        tags: |
          ${{ secrets.DOCKER_REGISTRY }}/api:latest
          ${{ secrets.DOCKER_REGISTRY }}/api:${{ github.sha }}

    - name: Deploy to production
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /opt/api
          docker-compose pull
          docker-compose up -d

          # Health check
          sleep 10
          curl -f http://localhost:3000/health || exit 1
```

### Node.js Production Dockerfile
```dockerfile
FROM node:18-alpine AS base

# Install dependencies
FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Build stage
FROM base AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM base AS production

# Create app user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

WORKDIR /app

# Copy production dependencies
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules
# Copy built application
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --from=build --chown=nodejs:nodejs /app/package*.json ./

# Security updates
RUN apk upgrade --no-cache

USER nodejs

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "dist/index.js"]
```

---

## GitLab CI/CD Alternative

### GitLab CI Configuration (.gitlab-ci.yml)
```yaml
# GitLab CI/CD Pipeline
stages:
  - test
  - build
  - deploy-staging
  - deploy-production

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE
  DOCKER_TAG: $CI_COMMIT_SHA
  NODE_VERSION: "18"

# Templates
.node_template: &node_template
  image: node:18-alpine
  before_script:
    - npm ci --cache .npm --prefer-offline

# Test Stage
test:unit:
  <<: *node_template
  stage: test
  script:
    - npm run test:unit
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
  cache:
    paths:
      - .npm/
      - node_modules/

test:integration:
  <<: *node_template
  stage: test
  services:
    - name: mongo:5
      alias: mongodb
    - name: redis:7
      alias: redis
  variables:
    MONGODB_URL: mongodb://mongodb:27017/test
    REDIS_URL: redis://redis:6379
  script:
    - npm run test:integration

lint:
  <<: *node_template
  stage: test
  script:
    - npm run lint

security:
  <<: *node_template
  stage: test
  script:
    - npm audit --audit-level=moderate

# Build Stage
build:
  stage: build
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
    - docker tag $DOCKER_IMAGE:$DOCKER_TAG $DOCKER_IMAGE:latest
    - docker push $DOCKER_IMAGE:$DOCKER_TAG
    - docker push $DOCKER_IMAGE:latest
  only:
    - main
    - develop

# Staging Deployment
deploy:staging:
  stage: deploy-staging
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client curl
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '
' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $STAGING_HOST >> ~/.ssh/known_hosts
  script:
    - |
      ssh $SERVER_USER@$STAGING_HOST << 'EOF'
        cd /var/www/staging
        docker-compose pull
        docker-compose up -d
        sleep 10
        curl -f http://localhost:3000/health
      EOF
  environment:
    name: staging
    url: https://staging.your-domain.com
  only:
    - develop

# Production Deployment
deploy:production:
  stage: deploy-production
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client curl
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '
' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
  script:
    - ssh $SERVER_USER@$PRODUCTION_HOST "cd /var/www/production && ./deploy.sh $CI_COMMIT_SHA"
  environment:
    name: production
    url: https://your-domain.com
  when: manual
  only:
    - main
```

---

## Environment Management

### Environment Configuration
```bash
# .env.example
NODE_ENV=production
PORT=3000

# Database
DATABASE_URL=mongodb://localhost:27017/app
REDIS_URL=redis://localhost:6379

# Authentication
JWT_SECRET=your_super_secret_key
JWT_EXPIRES_IN=7d

# API Keys
STRIPE_SECRET_KEY=sk_test_...
SENDGRID_API_KEY=SG.123...

# Monitoring
SENTRY_DSN=https://...
LOG_LEVEL=info

# Feature Flags
ENABLE_FEATURE_X=false
ENABLE_DEBUG=false
```

### GitHub Secrets Setup
```bash
# Required secrets for GitHub Actions
DIGITALOCEAN_ACCESS_TOKEN     # DigitalOcean API token
SSH_PRIVATE_KEY              # SSH private key for VPS access
HOST                         # Server IP or hostname
USERNAME                     # SSH username
DATABASE_URL                 # Production database URL
JWT_SECRET                   # JWT signing secret
SENTRY_DSN                   # Error tracking
SLACK_WEBHOOK                # Deployment notifications

# Optional secrets
DOCKER_REGISTRY              # Docker registry URL
AWS_ACCESS_KEY_ID            # For S3 deployments
AWS_SECRET_ACCESS_KEY        # For S3 deployments
CLOUDFRONT_DISTRIBUTION_ID   # For CDN invalidation
```

### Dynamic Environment Injection
```javascript
// scripts/inject-env.js
const fs = require('fs');
const path = require('path');

const envFile = process.env.NODE_ENV === 'production' 
  ? '.env.production' 
  : '.env.staging';

const envPath = path.join(__dirname, '..', envFile);

if (fs.existsSync(envPath)) {
  require('dotenv').config({ path: envPath });
}

// Validate required environment variables
const required = [
  'DATABASE_URL',
  'JWT_SECRET',
  'API_URL'
];

const missing = required.filter(key => !process.env[key]);

if (missing.length > 0) {
  console.error(`Missing required environment variables: ${missing.join(', ')}`);
  process.exit(1);
}

console.log('Environment configuration loaded successfully');
```

---

## Monitoring & Logging

### Application Monitoring
```javascript
// monitoring/healthcheck.js
const express = require('express');
const mongoose = require('mongoose');
const redis = require('redis');

const router = express.Router();

router.get('/health', async (req, res) => {
  const checks = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    version: process.env.npm_package_version,
    uptime: process.uptime(),
    environment: process.env.NODE_ENV,
    checks: {}
  };

  try {
    // Database check
    const dbState = mongoose.connection.readyState;
    checks.checks.database = {
      status: dbState === 1 ? 'healthy' : 'unhealthy',
      state: dbState
    };

    // Redis check
    const redisClient = redis.createClient(process.env.REDIS_URL);
    await redisClient.ping();
    checks.checks.redis = { status: 'healthy' };
    redisClient.quit();

    // Memory usage
    const memUsage = process.memoryUsage();
    checks.checks.memory = {
      used: Math.round(memUsage.heapUsed / 1024 / 1024),
      total: Math.round(memUsage.heapTotal / 1024 / 1024),
      external: Math.round(memUsage.external / 1024 / 1024)
    };

    const allHealthy = Object.values(checks.checks)
      .every(check => check.status === 'healthy');

    res.status(allHealthy ? 200 : 503).json(checks);
  } catch (error) {
    checks.status = 'error';
    checks.error = error.message;
    res.status(503).json(checks);
  }
});

module.exports = router;
```

### Logging Configuration
```javascript
// logging/logger.js
const winston = require('winston');
const { combine, timestamp, json, errors } = winston.format;

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: combine(
    timestamp(),
    errors({ stack: true }),
    json()
  ),
  defaultMeta: {
    service: 'app',
    version: process.env.npm_package_version,
    environment: process.env.NODE_ENV
  },
  transports: [
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'logs/combined.log' 
    })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

module.exports = logger;
```

### Performance Monitoring
```javascript
// monitoring/metrics.js
const prometheus = require('prom-client');

// Create a Registry to register the metrics
const register = new prometheus.Registry();

// Add default metrics
prometheus.collectDefaultMetrics({
  register,
  gcDurationBuckets: [0.001, 0.01, 0.1, 1, 2, 5]
});

// Custom metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status']
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);

module.exports = { register, httpRequestDuration, httpRequestTotal };
```

---

## Security Best Practices

### Security Scanning in CI/CD
```yaml
# Security scanning job
security_scan:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'

    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'

    - name: SAST Scan
      uses: securecodewarrior/github-action-add-sarif@v1
      with:
        sarif-file: 'results.sarif'

    - name: Dependency Check
      run: |
        npm audit --audit-level=high --production

    - name: License Check
      run: |
        npx license-checker --production --failOn GPL
```

### Environment Security
```bash
# scripts/secure-env.sh
#!/bin/bash

# Secure environment setup script
set -e

echo "Setting up secure environment..."

# Update system
apt update && apt upgrade -y

# Install fail2ban
apt install -y fail2ban
systemctl enable fail2ban
systemctl start fail2ban

# Configure SSH security
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart ssh

# Setup firewall
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow 80
ufw allow 443
ufw --force enable

# Install security updates automatically
apt install -y unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades

echo "Security setup completed!"
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Build Failures
```yaml
# Debug build issues
- name: Debug build failure
  if: failure()
  run: |
    echo "Node version: $(node --version)"
    echo "NPM version: $(npm --version)"
    echo "Available memory: $(free -m)"
    echo "Disk space: $(df -h)"
    ls -la
    cat package.json
```

#### 2. Deployment Failures
```bash
# Rollback script
#!/bin/bash
BACKUP_DIR="/var/backups/app"
LATEST_BACKUP=$(ls -t $BACKUP_DIR/backup_*.tar.gz | head -1)

if [ -f "$LATEST_BACKUP" ]; then
    echo "Rolling back to $LATEST_BACKUP"
    cd /var/www/app
    tar -xzf "$LATEST_BACKUP"
    pm2 restart all
    echo "Rollback completed"
else
    echo "No backup found for rollback"
    exit 1
fi
```

#### 3. Database Connection Issues
```javascript
// Database retry logic
const mongoose = require('mongoose');

const connectDB = async (retries = 5, delay = 5000) => {
  while (retries) {
    try {
      await mongoose.connect(process.env.DATABASE_URL);
      console.log('Database connected successfully');
      break;
    } catch (error) {
      console.log(`Database connection failed. Retries left: ${retries - 1}`);
      retries -= 1;

      if (retries === 0) {
        console.error('Database connection failed permanently:', error);
        process.exit(1);
      }

      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
};

module.exports = connectDB;
```

#### 4. Performance Issues
```bash
# Performance monitoring script
#!/bin/bash
echo "=== System Resources ==="
free -m
df -h
uptime

echo "=== Application Status ==="
pm2 status
pm2 monit --no-interaction

echo "=== Network Connections ==="
netstat -tuln | grep LISTEN

echo "=== Recent Errors ==="
tail -n 50 /var/log/app/error.log
```

### Debugging Checklist
- [ ] Check application logs
- [ ] Verify environment variables
- [ ] Test database connections
- [ ] Check server resources (CPU, memory, disk)
- [ ] Verify network connectivity
- [ ] Check service status (nginx, pm2, docker)
- [ ] Review recent deployments
- [ ] Test health endpoints
- [ ] Verify SSL certificates
- [ ] Check firewall rules

---

## Conclusion

This comprehensive CI/CD pipeline guide provides everything needed to automate deployments for Next.js, Node.js, and React.js applications. The pipeline includes:

### Key Benefits
- **Automated Testing**: Unit, integration, and security testing
- **Zero-Downtime Deployments**: Blue-green deployment strategy
- **Rollback capabilities**: Quick recovery from failed deployments
- **Multi-environment support**: Staging and production environments
- **Security scanning**: Vulnerability and license checks
- **Performance monitoring**: Application and infrastructure metrics

### Best Practices Implemented
- Infrastructure as Code (IaC)
- Containerization with Docker
- Secrets management
- Environment separation
- Health checks and monitoring
- Security hardening
- Automated backups

### Next Steps
1. **Set up monitoring**: Implement comprehensive monitoring and alerting
2. **Add testing**: Expand test coverage with E2E testing
3. **Optimize performance**: Implement caching and CDN strategies
4. **Scale infrastructure**: Set up auto-scaling and load balancing
5. **Enhance security**: Add advanced security scanning and compliance checks

This pipeline provides a solid foundation for professional software delivery and can be adapted to specific requirements and environments.

---

**Remember**: Always test your CI/CD pipeline in a staging environment before deploying to production, and maintain proper backup strategies for critical applications.
