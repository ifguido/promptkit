# Docker Setup - Full-Stack Application

## Objective
Create a comprehensive Docker and Docker Compose setup for a full-stack application with development and production configurations.

## Technology Stack
- Docker
- Docker Compose
- Frontend: React/Next.js
- Backend: Node.js/Express
- Database: PostgreSQL
- Redis
- Nginx (production)
- Multi-stage builds for optimization

## Requirements

### 1. Services to Containerize

**Frontend:**
- React or Next.js application
- Development mode with hot reload
- Production build with optimized assets
- Nginx serving in production

**Backend:**
- Node.js/Express API
- Development mode with nodemon
- Production build with TypeScript compilation
- Health checks

**Database:**
- PostgreSQL with persistent volume
- Custom initialization scripts
- Backup support

**Redis:**
- Caching and session storage
- Persistent data volume

**Nginx:**
- Reverse proxy for backend API
- Serve frontend static files
- SSL/TLS support
- Load balancing (if multiple backend instances)

### 2. File Structure
```
project-root/
├── docker/
│   ├── nginx/
│   │   ├── nginx.conf
│   │   ├── default.conf
│   │   └── ssl/
│   ├── postgres/
│   │   └── init.sql
│   └── scripts/
│       ├── backup-db.sh
│       └── restore-db.sh
├── frontend/
│   ├── Dockerfile
│   ├── Dockerfile.prod
│   ├── .dockerignore
│   └── nginx.conf (for production)
├── backend/
│   ├── Dockerfile
│   ├── Dockerfile.prod
│   └── .dockerignore
├── docker-compose.yml
├── docker-compose.prod.yml
├── docker-compose.override.yml
└── .env.example
```

### 3. Frontend Dockerfile (Development)

```dockerfile
# Development Dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Start development server
CMD ["npm", "run", "dev"]
```

### 4. Frontend Dockerfile (Production)

```dockerfile
# Multi-stage build for production

# Stage 1: Build
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Stage 2: Production
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/build /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### 5. Backend Dockerfile (Development)

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Install dependencies first (better caching)
COPY package*.json ./
RUN npm ci

# Copy source
COPY . .

# Expose port
EXPOSE 5000

# Use nodemon for hot reload
CMD ["npm", "run", "dev"]
```

### 6. Backend Dockerfile (Production)

```dockerfile
# Multi-stage build

# Stage 1: Build
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine

WORKDIR /app

# Copy package files and install production dependencies only
COPY package*.json ./
RUN npm ci --only=production

# Copy built files from builder stage
COPY --from=builder /app/dist ./dist

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 5000

CMD ["node", "dist/server.js"]
```

### 7. Docker Compose (Development)

```yaml
version: '3.8'

services:
  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - REACT_APP_API_URL=http://localhost:5000/api
    depends_on:
      - backend
    networks:
      - app-network

  # Backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - PORT=5000
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network

  # PostgreSQL
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
    networks:
      - app-network

  # Adminer (Database UI)
  adminer:
    image: adminer:latest
    ports:
      - "8080:8080"
    environment:
      - ADMINER_DEFAULT_SERVER=postgres
    depends_on:
      - postgres
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### 8. Docker Compose (Production)

```yaml
version: '3.8'

services:
  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./docker/nginx/ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    networks:
      - app-network
    restart: unless-stopped

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    expose:
      - "80"
    networks:
      - app-network
    restart: unless-stopped

  # Backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    expose:
      - "5000"
    environment:
      - NODE_ENV=production
      - PORT=5000
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - app-network
    restart: unless-stopped
    # Run multiple instances for load balancing
    deploy:
      replicas: 3

  # PostgreSQL
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    restart: unless-stopped

  # Redis
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    networks:
      - app-network
    restart: unless-stopped

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### 9. Nginx Configuration (Production)

```nginx
# docker/nginx/default.conf

upstream frontend {
    server frontend:80;
}

upstream backend {
    least_conn;
    server backend:5000 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Frontend
    location / {
        proxy_pass http://frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API
    location /api {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

### 10. .dockerignore Files

**Frontend .dockerignore:**
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.env.local
build
dist
coverage
.vscode
.idea
```

**Backend .dockerignore:**
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.env.test
dist
coverage
logs
.vscode
.idea
```

### 11. Useful Docker Commands

**Development:**
```bash
# Start all services
docker-compose up

# Start in detached mode
docker-compose up -d

# Rebuild containers
docker-compose up --build

# Stop all services
docker-compose down

# View logs
docker-compose logs -f backend

# Execute command in container
docker-compose exec backend npm run migrate
```

**Production:**
```bash
# Build and start production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Scale backend service
docker-compose -f docker-compose.prod.yml up -d --scale backend=5

# Database backup
docker-compose exec postgres pg_dump -U postgres myapp > backup.sql
```

### 12. Environment Variables

**.env.example:**
```
# Application
NODE_ENV=production
BASE_URL=https://your-domain.com

# Database
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your-secure-password
POSTGRES_DB=myapp
DATABASE_URL=postgresql://postgres:your-secure-password@postgres:5432/myapp

# Redis
REDIS_PASSWORD=your-redis-password
REDIS_URL=redis://:your-redis-password@redis:6379

# Security
JWT_SECRET=your-jwt-secret
ENCRYPTION_KEY=your-encryption-key
```

### 13. Health Checks

```yaml
backend:
  healthcheck:
    test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:5000/health"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 40s
```

## Deliverables
- Complete Docker setup for all services
- Development and production configurations
- Multi-stage builds for optimization
- Nginx reverse proxy configuration
- Database initialization scripts
- Health checks for all services
- Environment variable configuration
- Backup and restore scripts
- Documentation for common operations

## Testing Checklist
- [ ] Development environment starts successfully
- [ ] Hot reload works in development
- [ ] Production build creates optimized images
- [ ] All services can communicate
- [ ] Database persists data after restart
- [ ] Redis caching works
- [ ] Nginx routes requests correctly
- [ ] SSL/TLS works in production
- [ ] Health checks detect unhealthy services
- [ ] Environment variables are loaded correctly
- [ ] Volumes persist data
- [ ] Logs are accessible
- [ ] Database backups work
- [ ] Scaling backend works (production)
- [ ] No sensitive data in images
