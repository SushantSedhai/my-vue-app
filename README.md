# Vue.js Application with Docker - Setup Guide

This guide will walk you through setting up a Vue.js application and deploying it using Docker.

## Prerequisites

- Node.js (v16 or higher)
- npm or yarn
- Docker installed on your system
- Docker Compose (optional, but recommended)

## Project Directory Structure
```
my-vue-app/
├── README.md
└── my-vue-app
    ├── Dockerfile
    ├── README.md
    ├── babel.config.js
    ├── docker-compose.yml
    ├── jsconfig.json
    ├── nginx.conf
    ├── node_modules
    ├── package-lock.json
    ├── package.json
    ├── public
    ├── src
    ├── vue.config.js
    └── yarn.lock
```
## Project Setup

### 1. Create a New Vue.js Application

```bash
# Install Vue CLI and create new project
npm install -g @vue/cli
vue create my-vue-app

# Navigate to project directory
cd my-vue-app
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Test the Application Locally

```bash
npm run serve
```
#### If any Error occurs try this
    create a vue.config.js file in your project root:
```bash
    touch vue.config.js
```
#### Then add this content:
```javascript
const { defineConfig } = require('@vue/cli-service')

module.exports = defineConfig({
  transpileDependencies: true,
  devServer: {
    client: {
      webSocketURL: 'auto://0.0.0.0:0/ws'
    },
    allowedHosts: 'all',
    hot: true,
    liveReload: true
  }
})
```
Then restart your dev server:
```bash
npm run serve
```
Visit `http://localhost:8080` to verify the application runs correctly.

## Docker Configuration

### 1. Create Dockerfile

Create a `Dockerfile` in your project root:

```dockerfile
# Build stage
FROM node:18-alpine as build-stage

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy project files
COPY . .

# Build application
RUN npm run build

# Production stage
FROM nginx:stable-alpine as production-stage

# Copy built assets from build stage
COPY --from=build-stage /app/dist /usr/share/nginx/html

# Copy nginx configuration (optional)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

### 2. Create Nginx Configuration

Create `nginx.conf` in your project root:

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Enable gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 3. Create .dockerignore

Create `.dockerignore` to exclude unnecessary files:

```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
dist
coverage
```

### 4. Create docker-compose.yml (Optional)

For easier management, create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  vue-app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    container_name: vue-app
    restart: unless-stopped
```

## Building and Running with Docker

### Option 1: Using Docker Commands

```bash
# Build the Docker image
docker build -t vue-app:latest .

# Run the container
docker run -d -p 8080:80 --name vue-app vue-app:latest

# View running containers
docker ps

# View logs
docker logs vue-app

# Stop the container
docker stop vue-app

# Remove the container
docker rm vue-app
```

### Option 2: Using Docker Compose

```bash
# Build and start the container
docker-compose up -d

# View logs
docker-compose logs -f

# Stop the container
docker-compose down

# Rebuild and restart
docker-compose up -d --build
```

## Accessing Your Application

Once the container is running, access your application at:
- `http://localhost:8080`




## Additional Resources

- [Vue.js Documentation](https://vuejs.org/)
- [Docker Documentation](https://docs.docker.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)

## License

MIT