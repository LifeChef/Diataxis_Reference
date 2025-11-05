# Tutorial: Setup Development Environment

> **Document Type**: Tutorial (Learning-Oriented)  
> **Audience**: Beginner  
> **Estimated Time**: 30 minutes  
> **Last Updated**: November 5, 2024

## Overview

In this tutorial, you'll set up a complete local development environment for the Blog Engine. By the end, you'll have all services running locally and can start developing features.

**What you'll learn:**
- How to install required tools (Node.js, Docker, PostgreSQL)
- How to clone and configure the project
- How to start all services locally
- How to verify your setup is working

**What you'll build:**
A fully functional local development environment ready for coding.

---

## Prerequisites

Before starting, ensure you have:

- [ ] A computer with macOS, Windows, or Linux
- [ ] Administrator/sudo access on your machine
- [ ] Internet connection for downloading tools
- [ ] At least 10GB of free disk space
- [ ] At least 8GB of RAM

---

## Step 1: Install Node.js and npm

Node.js is our runtime environment.

### Instructions

1. **Check if already installed**
   
   ```bash
   node --version
   npm --version
   ```
   
   **Expected result**: If you see `v18.0.0` or higher, Node is installed. Skip to Step 2.

2. **Install Node.js** (if not installed)
   
   **For macOS** (using Homebrew):
   ```bash
   brew install node@18
   ```
   
   **For Linux** (Ubuntu/Debian):
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt-get install -y nodejs
   ```
   
   **For Windows**:
   Download from https://nodejs.org (LTS version)

3. **Verify installation**
   
   ```bash
   node --version  # Should show v18.x.x or higher
   npm --version   # Should show 9.x.x or higher
   ```

### Checkpoint ✓

At this point, you should have:
- [ ] Node.js v18+ installed
- [ ] npm v9+ installed

---

## Step 2: Install Docker and Docker Compose

We use Docker to run PostgreSQL, Redis, and RabbitMQ locally.

### Instructions

1. **Install Docker Desktop**
   
   Download from https://www.docker.com/products/docker-desktop
   
   - **macOS**: Download .dmg and install
   - **Windows**: Download .exe and install (requires WSL2)
   - **Linux**: Follow https://docs.docker.com/engine/install/

2. **Verify installation**
   
   ```bash
   docker --version
   docker-compose --version
   ```
   
   **Expected result**:
   ```
   Docker version 24.0.0 or higher
   Docker Compose version 2.20.0 or higher
   ```

3. **Start Docker Desktop**
   
   - Open Docker Desktop application
   - Wait for it to start (indicator turns green)

### Checkpoint ✓

- [ ] Docker installed and running
- [ ] Docker Compose available

---

## Step 3: Install Git

### Instructions

1. **Check if installed**
   
   ```bash
   git --version
   ```

2. **Install if needed**
   
   **macOS**: `brew install git`  
   **Linux**: `sudo apt-get install git`  
   **Windows**: Download from https://git-scm.com/

3. **Configure Git**
   
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

### Checkpoint ✓

- [ ] Git installed
- [ ] Git configured with your name and email

---

## Step 4: Clone the Repository

### Instructions

1. **Create workspace directory**
   
   ```bash
   mkdir ~/projects
   cd ~/projects
   ```

2. **Clone the repository**
   
   ```bash
   git clone https://github.com/your-org/blog-engine.git
   cd blog-engine
   ```
   
   **Expected result**: You should see files being downloaded.

3. **Explore the structure**
   
   ```bash
   ls -la
   ```
   
   You should see:
   ```
   api-gateway/
   auth-service/
   post-service/
   comment-service/
   notification-service/
   web-app/
   mobile-app/
   docker-compose.yml
   package.json
   README.md
   ```

### Checkpoint ✓

- [ ] Repository cloned successfully
- [ ] You're in the project directory
- [ ] You can see all service folders

---

## Step 5: Install Dependencies

Install Node.js dependencies for all services.

### Instructions

1. **Install root dependencies**
   
   ```bash
   npm install
   ```
   
   This takes 2-3 minutes. You'll see progress indicators.

2. **Install service dependencies**
   
   ```bash
   # Install for all services
   npm run install:all
   ```
   
   This script runs `npm install` in each service directory.
   
   **Expected result**: Completion message with no errors.

3. **Verify installations**
   
   ```bash
   npm list --depth=0
   ```

### Checkpoint ✓

- [ ] All dependencies installed
- [ ] No error messages
- [ ] Each service has `node_modules/` folder

---

## Step 6: Configure Environment Variables

Set up your local configuration.

### Instructions

1. **Copy example environment files**
   
   ```bash
   # Root env
   cp .env.example .env
   
   # Service-specific envs
   cp api-gateway/.env.example api-gateway/.env
   cp auth-service/.env.example auth-service/.env
   cp post-service/.env.example post-service/.env
   cp comment-service/.env.example comment-service/.env
   cp notification-service/.env.example notification-service/.env
   ```

2. **Edit the root `.env` file**
   
   ```bash
   nano .env  # or use your preferred editor
   ```
   
   Set the following:
   ```
   NODE_ENV=development
   
   # Database
   DB_HOST=localhost
   DB_PORT=5432
   DB_NAME=blogengine_dev
   DB_USER=postgres
   DB_PASSWORD=postgres
   
   # Redis
   REDIS_HOST=localhost
   REDIS_PORT=6379
   
   # RabbitMQ
   RABBITMQ_URL=amqp://guest:guest@localhost:5672
   
   # JWT Secret (for local dev)
   JWT_SECRET=your-local-dev-secret-change-in-production
   ```

3. **Verify the files**
   
   ```bash
   cat .env
   ```

### Checkpoint ✓

- [ ] All `.env` files created
- [ ] Configuration values set
- [ ] No syntax errors in env files

---

## Step 7: Start Infrastructure Services

Start PostgreSQL, Redis, and RabbitMQ using Docker.

### Instructions

1. **Start services**
   
   ```bash
   docker-compose up -d
   ```
   
   **Expected output**:
   ```
   Creating network "blog-engine_default" ...
   Creating blog-engine-postgres ...
   Creating blog-engine-redis ...
   Creating blog-engine-rabbitmq ...
   ```

2. **Verify services are running**
   
   ```bash
   docker-compose ps
   ```
   
   You should see all services with "Up" status:
   ```
   NAME                    STATUS
   blog-engine-postgres    Up
   blog-engine-redis       Up
   blog-engine-rabbitmq    Up
   ```

3. **Check logs** (if needed)
   
   ```bash
   docker-compose logs
   ```

### Checkpoint ✓

- [ ] All infrastructure services running
- [ ] No error messages in logs
- [ ] Services accessible on their ports

---

## Step 8: Run Database Migrations

Create the database schema.

### Instructions

1. **Run migrations**
   
   ```bash
   npm run migrate:latest
   ```
   
   **Expected output**:
   ```
   Batch 1 run: 5 migrations
   - 001_create_users_table.js
   - 002_create_posts_table.js
   - 003_create_comments_table.js
   - 004_create_notifications_table.js
   - 005_add_indexes.js
   ```

2. **Seed test data** (optional)
   
   ```bash
   npm run db:seed
   ```
   
   This creates sample users and posts for testing.

3. **Verify database**
   
   ```bash
   npm run db:check
   ```

### Checkpoint ✓

- [ ] Migrations completed successfully
- [ ] Database schema created
- [ ] Test data seeded (if you ran seed command)

---

## Step 9: Start Development Servers

Start all application services in development mode.

### Instructions

1. **Start all services**
   
   In the root directory:
   ```bash
   npm run dev
   ```
   
   This starts all services concurrently. You'll see:
   ```
   [api-gateway] Server running on http://localhost:3000
   [auth-service] Auth service running on port 3001
   [post-service] Post service running on port 3002
   [comment-service] Comment service running on port 3003
   [notification-service] Notification service running on port 3004
   [web-app] Web app running on http://localhost:3005
   ```

2. **Verify in browser**
   
   Open http://localhost:3005 in your browser.
   
   You should see the Blog Engine homepage.

3. **Test the API**
   
   Open a new terminal and test:
   ```bash
   curl http://localhost:3000/health
   ```
   
   **Expected response**:
   ```json
   {
     "status": "healthy",
     "services": {
       "database": "connected",
       "redis": "connected",
       "rabbitmq": "connected"
     }
   }
   ```

4. **Celebrate!** 🎉
   
   Your development environment is ready!

---

## Summary

Congratulations! You've set up your development environment.

**You've learned:**
- How to install Node.js, Docker, and Git
- How to clone and configure the project
- How to start infrastructure services
- How to run database migrations
- How to start all application services

**You've built:**
- A complete, working local development environment

---

## Next Steps

Now that your environment is ready:

1. **Continue learning**: Try [Building Your First Blog Post Feature](first-blog-post-feature.md)
2. **Explore the code**: Look through the service directories
3. **Read architecture docs**: Check [System Architecture](../../reference/architecture/system-architecture.md)

---

## Troubleshooting

### Problem: "Port already in use"
**Solution**: Check if another process is using the port. Kill it or change the port in `.env`.
```bash
# Find process on port 3000
lsof -ti:3000
# Kill it
kill -9 $(lsof -ti:3000)
```

### Problem: Docker services won't start
**Solution**: 
- Ensure Docker Desktop is running
- Check disk space: `docker system df`
- Restart Docker Desktop

### Problem: Database connection fails
**Solution**:
- Verify PostgreSQL is running: `docker-compose ps`
- Check credentials in `.env` match `docker-compose.yml`
- Restart services: `docker-compose restart postgres`

### Problem: npm install fails
**Solution**:
- Check Node.js version: `node --version` (need v18+)
- Clear npm cache: `npm cache clean --force`
- Delete `node_modules` and retry: `rm -rf node_modules && npm install`

---

**Tutorial Author**: Development Team  
**Last Updated**: November 5, 2024  
**Version**: 1.0
