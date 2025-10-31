# CI/CD Pipeline Code for Next.js, Node.js & React.js

![Author](https://img.shields.io/badge/Author-Jaydip%20Khunt-blue)
![Date](https://img.shields.io/badge/Date-October%202025-green)
![Guide](https://img.shields.io/badge/Type-Deployment%20Guide-orange)

![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![PM2](https://img.shields.io/badge/PM2-2B037A?style=for-the-badge&logo=pm2&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=next.js&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white)

Ready-to-use CI/CD pipeline configurations for deploying Next.js, Node.js, and React.js applications without Docker containers.

## Table of Contents
- [GitHub Actions Workflows](#github-actions-workflows)
- [PM2 Configuration](#pm2-configuration)
- [Next.js Pipeline](#nextjs-pipeline)
- [React.js Pipeline](#reactjs-pipeline)
- [Node.js API Pipeline](#nodejs-api-pipeline)
- [Deployment Scripts](#deployment-scripts)
- [Environment Configuration](#environment-configuration)

---

## GitHub Actions Workflows

### Main CI/CD Pipeline (.github/workflows/deploy.yml)
```yaml
name: Deploy to VPS (No Docker)

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'
  SERVER_HOST: ${{ secrets.SERVER_HOST }}
  SERVER_USER: ${{ secrets.SERVER_USER }}
  SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}

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
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run unit tests
      run: npm run test

    - name: Run build test
      run: npm run build

    - name: Security audit
      run: npm audit --audit-level=high

  # Deploy to staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/develop'

    environment:
      name: staging
      url: https://staging.yourdomain.com

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Deploy to staging server
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.STAGING_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/apps/staging

          # Backup current version
          if [ -d "current" ]; then
            cp -r current backup-$(date +%Y%m%d_%H%M%S)
          fi

          # Pull latest code
          git fetch origin
          git reset --hard origin/develop

          # Install/update dependencies
          npm ci --only=production

          # Build application
          npm run build

          # Restart application with PM2
          pm2 restart staging-app || pm2 start ecosystem.config.js --env staging

          # Health check
          sleep 5
          curl -f http://localhost:3001/health || exit 1

          echo "Staging deployment completed successfully"

  # Deploy to production
  deploy-production:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'

    environment:
      name: production
      url: https://yourdomain.com

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Deploy to production server
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/apps/production

          # Create backup
          ./scripts/backup.sh

          # Pull latest code
          git fetch origin
          git reset --hard origin/main

          # Install dependencies
          npm ci --only=production

          # Build application
          NODE_ENV=production npm run build

          # Zero-downtime restart
          ./scripts/zero-downtime-deploy.sh

          # Verify deployment
          ./scripts/health-check.sh || (echo "Deployment failed, rolling back..." && ./scripts/rollback.sh && exit 1)

          echo "Production deployment completed successfully"

    - name: Notify deployment status
      uses: 8398a7/action-slack@v3
      if: always()
      with:
        status: ${{ job.status }}
        channel: '#deployments'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Alternative rsync Deployment (.github/workflows/deploy-rsync.yml)
```yaml
name: Deploy with rsync

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies and build
      run: |
        npm ci
        npm run build

    - name: Deploy with rsync
      uses: burnett01/rsync-deployments@6.0.0
      with:
        switches: -avzr --delete --exclude='.git' --exclude='node_modules' --exclude='.env.local'
        path: ./
        remote_path: /var/www/apps/production/
        remote_host: ${{ secrets.SERVER_HOST }}
        remote_user: ${{ secrets.SERVER_USER }}
        remote_key: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Restart application
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/apps/production
          npm ci --only=production
          npm run build
          pm2 restart all
```

---

## PM2 Configuration

### PM2 Ecosystem Configuration (ecosystem.config.js)
```javascript
module.exports = {
  apps: [
    // Next.js Application
    {
      name: 'nextjs-app',
      script: './server.js',
      cwd: '/var/www/apps/nextjs-app',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3000,
        HOSTNAME: '0.0.0.0'
      },
      env_staging: {
        NODE_ENV: 'staging',
        PORT: 3001
      },

      // Logging
      log_file: '/var/log/apps/nextjs-app/app.log',
      error_file: '/var/log/apps/nextjs-app/error.log',
      out_file: '/var/log/apps/nextjs-app/out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',

      // Restart policy
      max_restarts: 3,
      min_uptime: '10s',
      max_memory_restart: '500M',

      // Performance monitoring
      pmx: true,

      // Health monitoring
      health_check_grace_period: 3000,
      health_check_fatal_exceptions: true
    },

    // Node.js API
    {
      name: 'api-server',
      script: './dist/server.js',
      cwd: '/var/www/apps/api',
      instances: 2,
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 8000,
        API_VERSION: 'v1'
      },
      env_staging: {
        NODE_ENV: 'staging',
        PORT: 8001
      },

      // Auto-restart on file changes (staging only)
      watch: process.env.NODE_ENV === 'staging' ? ['dist'] : false,
      ignore_watch: ['node_modules', 'logs'],

      // Logging
      log_file: '/var/log/apps/api/app.log',
      error_file: '/var/log/apps/api/error.log',
      out_file: '/var/log/apps/api/out.log',

      // Cron restart (daily at 3 AM)
      cron_restart: '0 3 * * *',

      // Memory management
      max_memory_restart: '300M',
      kill_timeout: 5000
    }
  ],

  // Deployment configuration
  deploy: {
    production: {
      user: 'deploy',
      host: ['your-server.com'],
      ref: 'origin/main',
      repo: 'git@github.com:your-username/your-repo.git',
      path: '/var/www/apps/production',
      'pre-deploy-local': '',
      'post-deploy': 'npm ci --only=production && npm run build && pm2 reload ecosystem.config.js --env production',
      'pre-setup': 'mkdir -p /var/www/apps/production'
    },

    staging: {
      user: 'deploy',
      host: ['staging-server.com'],
      ref: 'origin/develop',
      repo: 'git@github.com:your-username/your-repo.git',
      path: '/var/www/apps/staging',
      'post-deploy': 'npm ci --only=production && npm run build && pm2 reload ecosystem.config.js --env staging'
    }
  }
};
```

---

## Next.js Pipeline

### Next.js GitHub Actions (.github/workflows/nextjs-deploy.yml)
```yaml
name: Deploy Next.js App

on:
  push:
    branches: [main]
    paths: ['nextjs-app/**']

jobs:
  deploy-nextjs:
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: ./nextjs-app

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: nextjs-app/package-lock.json

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Build application
      env:
        NODE_ENV: production
        NEXT_APP_VERSION: ${{ github.sha }}
      run: npm run build

    - name: Deploy to server
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/apps/nextjs-app

          # Create backup
          if [ -d ".next" ]; then
            tar -czf "backup-$(date +%Y%m%d_%H%M%S).tar.gz" .next public
          fi

          # Pull latest code
          git fetch origin
          git reset --hard origin/main

          # Install dependencies and build
          npm ci --only=production
          NODE_ENV=production npm run build

          # Copy standalone files if available
          if [ -d ".next/standalone" ]; then
            cp -r .next/standalone/* ./
            cp -r .next/static .next/static
            cp -r public ./public
          fi

          # Zero-downtime restart
          pm2 reload nextjs-app || pm2 start server.js --name nextjs-app

          # Health check
          sleep 5
          curl -f http://localhost:3000/health || exit 1

          echo "Next.js deployment completed successfully"
```

### Next.js Configuration (next.config.js)
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable standalone output for PM2 deployment
  output: 'standalone',

  // Disable telemetry for server deployment
  experimental: {
    telemetry: false,
  },

  // Optimize for production
  swcMinify: true,
  compress: true,
  poweredByHeader: false,

  // Environment variables
  env: {
    CUSTOM_APP_VERSION: process.env.npm_package_version || '1.0.0',
    BUILD_TIME: new Date().toISOString(),
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
            key: 'X-XSS-Protection',
            value: '1; mode=block',
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin',
          },
        ],
      },
    ];
  },

  // Custom webpack configuration
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.resolve.fallback = {
        fs: false,
        net: false,
        tls: false,
      };
    }
    return config;
  },
};

module.exports = nextConfig;
```

### Next.js Custom Server (server.js)
```javascript
const { createServer } = require('http');
const { parse } = require('url');
const next = require('next');

const dev = process.env.NODE_ENV !== 'production';
const hostname = process.env.HOSTNAME || 'localhost';
const port = parseInt(process.env.PORT) || 3000;

const app = next({ dev, hostname, port });
const handle = app.getRequestHandler();

// Health check endpoint
const healthCheck = (req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    version: process.env.npm_package_version || '1.0.0',
    environment: process.env.NODE_ENV
  }));
};

app.prepare().then(() => {
  createServer(async (req, res) => {
    const parsedUrl = parse(req.url, true);
    const { pathname } = parsedUrl;

    // Handle health check
    if (pathname === '/health') {
      return healthCheck(req, res);
    }

    // Handle all other requests with Next.js
    await handle(req, res, parsedUrl);
  }).listen(port, (err) => {
    if (err) throw err;
    console.log(`> Ready on http://${hostname}:${port}`);
    console.log(`> Environment: ${process.env.NODE_ENV}`);
    console.log(`> Process ID: ${process.pid}`);
  });
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received, shutting down gracefully');
  process.exit(0);
});

process.on('SIGINT', () => {
  console.log('SIGINT received, shutting down gracefully');
  process.exit(0);
});
```

---

## React.js Pipeline

### React Build and Deploy (.github/workflows/react-deploy.yml)
```yaml
name: Deploy React App

on:
  push:
    branches: [main]
    paths: ['frontend/**']

jobs:
  deploy-react:
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: ./frontend

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

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
        GENERATE_SOURCEMAP: false
      run: npm run build

    - name: Deploy to server
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/apps/react-app

          # Backup current build
          if [ -d "build" ]; then
            mv build build_backup_$(date +%Y%m%d_%H%M%S)
          fi

          # Pull latest code
          git fetch origin
          git reset --hard origin/main

          # Install dependencies and build
          npm ci --only=production
          npm run build

          # Deploy to nginx web root
          sudo rm -rf /var/www/html/react-app
          sudo cp -r build /var/www/html/react-app

          # Set permissions
          sudo chown -R www-data:www-data /var/www/html/react-app

          # Test nginx configuration and reload
          sudo nginx -t && sudo systemctl reload nginx

          echo "React app deployed successfully"
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

## Node.js API Pipeline

### Node.js API Workflow (.github/workflows/api-deploy.yml)
```yaml
name: Deploy Node.js API

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
    - name: Checkout code
      uses: actions/checkout@v4

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

    defaults:
      run:
        working-directory: ./api

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Deploy to production
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/apps/api

          # Create backup
          if [ -d "dist" ]; then
            tar -czf "backup-$(date +%Y%m%d_%H%M%S).tar.gz" dist package.json
          fi

          # Pull latest code
          git fetch origin
          git reset --hard origin/main

          # Install dependencies
          npm ci --only=production

          # Build application
          npm run build

          # Update environment variables
          if [ -f ".env.production" ]; then
            cp .env.production .env
          fi

          # Zero-downtime restart with PM2
          pm2 reload api-server || pm2 start ecosystem.config.js --env production --name api-server

          # Health check
          sleep 5
          for i in {1..10}; do
            if curl -f http://localhost:8000/health; then
              echo "API health check passed"
              break
            fi
            echo "Health check attempt $i failed, retrying in 3 seconds..."
            sleep 3
          done

          echo "API deployment completed successfully"
```

### Node.js Server with Health Checks (src/server.js)
```javascript
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const compression = require('compression');

const app = express();
const PORT = process.env.PORT || 8000;

// Security middleware
app.use(helmet());
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
  credentials: true,
}));
app.use(compression());

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Health check endpoint
app.get('/health', (req, res) => {
  const healthCheck = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    version: process.env.npm_package_version || '1.0.0',
    environment: process.env.NODE_ENV,
    pid: process.pid,
  };

  res.status(200).json(healthCheck);
});

// API routes
app.use('/api/v1', require('./routes'));

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Something went wrong!',
    timestamp: new Date().toISOString(),
  });
});

// Start server
const server = app.listen(PORT, '0.0.0.0', () => {
  console.log(`🚀 API Server running on port ${PORT}`);
  console.log(`📊 Environment: ${process.env.NODE_ENV}`);
  console.log(`🆔 Process ID: ${process.pid}`);
});

// Graceful shutdown
const gracefulShutdown = (signal) => {
  console.log(`📥 ${signal} received, shutting down gracefully`);

  server.close(() => {
    console.log('👋 HTTP server closed');
    process.exit(0);
  });

  setTimeout(() => {
    console.error('⚠️  Could not close connections in time, forcefully shutting down');
    process.exit(1);
  }, 30000);
};

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));
```

---

## Deployment Scripts

### Zero Downtime Deployment (scripts/zero-downtime-deploy.sh)
```bash
#!/bin/bash

set -e

APP_NAME=${1:-"nextjs-app"}
DEPLOY_DIR="/var/www/apps/$APP_NAME"

echo "Starting zero-downtime deployment for $APP_NAME..."

cd $DEPLOY_DIR

# Check if PM2 process exists
if pm2 describe $APP_NAME > /dev/null 2>&1; then
    echo "Reloading existing PM2 process..."
    pm2 reload $APP_NAME --update-env
else
    echo "Starting new PM2 process..."
    pm2 start ecosystem.config.js --env production
fi

# Wait for application to be ready
echo "Waiting for application to be ready..."
sleep 10

# Health check
if curl -f http://localhost:3000/health > /dev/null 2>&1; then
    echo "Health check passed - deployment successful"
    pm2 delete ${APP_NAME}-old > /dev/null 2>&1 || true
else
    echo "Health check failed - rolling back"
    pm2 restart ${APP_NAME}-old > /dev/null 2>&1 || true
    exit 1
fi

echo "Zero-downtime deployment completed successfully"
```

### Backup Script (scripts/backup.sh)
```bash
#!/bin/bash

APP_NAME=${1:-"production"}
BACKUP_DIR="/var/backups/$APP_NAME"
DEPLOY_DIR="/var/www/apps/$APP_NAME"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Create backup
cd $DEPLOY_DIR
tar -czf "$BACKUP_DIR/backup_$TIMESTAMP.tar.gz"   --exclude=node_modules   --exclude=.git   --exclude=logs   .

echo "Backup created: $BACKUP_DIR/backup_$TIMESTAMP.tar.gz"

# Clean up old backups (keep last 5)
find $BACKUP_DIR -name "backup_*.tar.gz" -type f -mtime +5 -delete
```

### Health Check Script (scripts/health-check.sh)
```bash
#!/bin/bash

set -e

echo "Running health checks..."

# Check Next.js app
if curl -f http://localhost:3000/health >/dev/null 2>&1; then
    echo "✅ Next.js App: Healthy"
else
    echo "❌ Next.js App: Unhealthy"
    exit 1
fi

# Check API server
if curl -f http://localhost:8000/health >/dev/null 2>&1; then
    echo "✅ API Server: Healthy"
else
    echo "❌ API Server: Unhealthy"
    exit 1
fi

# Check PM2 processes
if pm2 status | grep -q online; then
    echo "✅ PM2: Processes online"
else
    echo "❌ PM2: Issues detected"
    exit 1
fi

echo "All health checks passed!"
```

### Rollback Script (scripts/rollback.sh)
```bash
#!/bin/bash

set -e

APP_NAME=${1:-"production"}
BACKUP_DIR="/var/backups/$APP_NAME"
DEPLOY_DIR="/var/www/apps/$APP_NAME"

echo "Rolling back $APP_NAME to previous version..."

# Find latest backup
LATEST_BACKUP=$(ls -t $BACKUP_DIR/backup_*.tar.gz 2>/dev/null | head -1)

if [ -z "$LATEST_BACKUP" ]; then
    echo "❌ No backup found for rollback"
    exit 1
fi

echo "📦 Using backup: $LATEST_BACKUP"

# Stop current application
pm2 stop $APP_NAME || true

# Restore from backup
cd $DEPLOY_DIR
tar -xzf "$LATEST_BACKUP"

# Restart application
pm2 start ecosystem.config.js --name $APP_NAME

# Health check
sleep 10
if curl -f http://localhost:3000/health; then
    echo "✅ Rollback completed successfully"
else
    echo "❌ Rollback failed - application unhealthy"
    exit 1
fi
```

---

## Environment Configuration

### GitHub Repository Secrets
```bash
# Required secrets for GitHub Actions
# Go to: Repository Settings > Secrets and Variables > Actions

# Server Connection
SERVER_HOST=your-server-ip-or-domain
SERVER_USER=deploy
SSH_PRIVATE_KEY=-----BEGIN OPENSSH PRIVATE KEY-----...

# Environment Variables
DATABASE_URL=mongodb://user:pass@host:port/db
JWT_SECRET=your_super_secret_jwt_key
API_SECRET_KEY=your_api_secret_key

# React Environment Variables
REACT_APP_API_URL=https://api.yourdomain.com
REACT_APP_VERSION=1.0.0

# Optional Notifications
SLACK_WEBHOOK=https://hooks.slack.com/services/...
```

### Environment Files
```bash
# .env.production
NODE_ENV=production
PORT=3000
DATABASE_URL=${DATABASE_URL}
JWT_SECRET=${JWT_SECRET}
API_SECRET_KEY=${API_SECRET_KEY}
LOG_LEVEL=warn
ENABLE_CLUSTER=true

# .env.staging
NODE_ENV=staging
PORT=3001
DATABASE_URL=${DATABASE_URL_STAGING}
JWT_SECRET=${JWT_SECRET_STAGING}
API_SECRET_KEY=${API_SECRET_KEY_STAGING}
LOG_LEVEL=info
```

### Package.json Scripts
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "node server.js",
    "lint": "next lint",
    "test": "jest",
    "test:unit": "jest --testPathPattern=unit",
    "test:integration": "jest --testPathPattern=integration",
    "deploy:staging": "pm2 deploy ecosystem.config.js staging",
    "deploy:production": "pm2 deploy ecosystem.config.js production"
  }
}
```

---

## Usage Instructions

1. **Copy the workflow files** to your `.github/workflows/` directory
2. **Add the required secrets** to your GitHub repository settings
3. **Configure PM2 ecosystem** with your application paths
4. **Set up deployment scripts** in your `scripts/` directory
5. **Configure environment variables** for different environments
6. **Push to your repository** to trigger the deployment pipeline

This focused pipeline code provides everything needed for automated deployments without server setup complexity!
