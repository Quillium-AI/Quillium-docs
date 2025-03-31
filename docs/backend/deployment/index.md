# Deployment Guide

This document provides instructions for deploying the Quillium backend to various environments.

## Overview

The Quillium backend is designed to be deployed in a variety of environments, from simple single-server setups to complex containerized deployments. This guide covers the most common deployment scenarios.

## Prerequisites

Before deploying the Quillium backend, ensure you have:

1. PostgreSQL database (version 12 or higher)
2. Go runtime (version 1.20 or higher) for building from source
3. Docker (optional, for containerized deployments)
4. SSL certificate (for production deployments)
5. Domain name configured with DNS (for production deployments)

## Environment Variables

The Quillium backend is configured primarily through environment variables:

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `DATABASE_URL` | PostgreSQL connection string | - | Yes |
| `JWT_SECRET` | Secret key for JWT signing | - | Yes |
| `PORT` | Port to listen on | 8080 | No |
| `LOG_LEVEL` | Logging level (debug, info, warn, error) | info | No |
| `CORS_ALLOWED_ORIGINS` | Comma-separated list of allowed origins | * | No |
| `ACCESS_TOKEN_EXPIRY` | JWT access token expiry in minutes | 15 | No |
| `REFRESH_TOKEN_EXPIRY` | Refresh token expiry in days | 7 | No |

## Deployment Options

### 1. Direct Deployment

This is the simplest deployment method, suitable for development or small-scale production use.

#### Building from Source

```bash
# Clone the repository
git clone https://github.com/Quillium-AI/Quillium.git
cd Quillium/src/backend

# Build the binary
go build -o quillium-server ./cmd/server

# Run database migrations
go run ./cmd/migrate up

# Start the server
export DATABASE_URL="postgres://user:password@localhost:5432/quillium"
export JWT_SECRET="your-secure-jwt-secret"
./quillium-server
```

#### Running as a Service

For production use, it's recommended to run the backend as a system service.

**Systemd Service Example (Linux)**

Create a file at `/etc/systemd/system/quillium.service`:

```ini
[Unit]
Description=Quillium Backend
After=network.target postgresql.service

[Service]
User=quillium
WorkingDirectory=/opt/quillium
ExecStart=/opt/quillium/quillium-server
Restart=always
RestartSec=5
Environment=DATABASE_URL=postgres://user:password@localhost:5432/quillium
Environment=JWT_SECRET=your-secure-jwt-secret
Environment=PORT=8080

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl enable quillium
sudo systemctl start quillium
```

### 2. Docker Deployment

For containerized deployments, we provide a Docker image and Docker Compose configuration.

#### Using Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3'

services:
  db:
    image: postgres:14
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: quillium
      POSTGRES_PASSWORD: your_secure_password
      POSTGRES_DB: quillium
    restart: always

  backend:
    image: quillium/backend:latest
    depends_on:
      - db
    environment:
      DATABASE_URL: postgres://quillium:your_secure_password@db:5432/quillium
      JWT_SECRET: your_secure_jwt_secret
      PORT: 8080
    ports:
      - "8080:8080"
    restart: always

volumes:
  postgres_data:
```

Start the services:

```bash
docker-compose up -d
```

#### Building the Docker Image

If you need to build the Docker image yourself:

```bash
cd Quillium
docker build -t quillium/backend:latest -f Dockerfile.backend .
```

### 3. Cloud Deployments

#### AWS Elastic Beanstalk

1. Create a new Elastic Beanstalk application
2. Choose the Go platform
3. Upload a ZIP file containing:
   - The compiled binary
   - A `Procfile` with the command to run the server
   - An `.ebextensions` directory with configuration files

Example `Procfile`:
```
web: bin/quillium-server
```

Example `.ebextensions/01_env.config`:
```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    PORT: 8080
    LOG_LEVEL: info
```

#### Google Cloud Run

1. Build and push the Docker image to Google Container Registry
2. Deploy to Cloud Run with environment variables

```bash
# Build and push the image
gcloud builds submit --tag gcr.io/your-project/quillium-backend

# Deploy to Cloud Run
gcloud run deploy quillium-backend \
  --image gcr.io/your-project/quillium-backend \
  --platform managed \
  --set-env-vars DATABASE_URL=postgres://user:password@host:5432/quillium,JWT_SECRET=your-secure-jwt-secret
```

#### Heroku

1. Create a `heroku.yml` file in the project root:

```yaml
build:
  docker:
    web: Dockerfile.backend
```

2. Create a new Heroku app and set it to use the container stack:

```bash
heroku create quillium-backend
heroku stack:set container
```

3. Set environment variables:

```bash
heroku config:set DATABASE_URL=postgres://user:password@host:5432/quillium
heroku config:set JWT_SECRET=your-secure-jwt-secret
```

4. Deploy:

```bash
git push heroku main
```

## Database Migrations

Before starting the application in any environment, ensure that database migrations are applied:

```bash
# Using the migration tool directly
go run ./cmd/migrate up

# Or using the provided script
./scripts/migrate.sh up
```

## SSL/TLS Configuration

For production deployments, always use HTTPS. You can:

1. Use a reverse proxy like Nginx or Traefik to handle SSL termination
2. Configure the application to use SSL directly

Example Nginx configuration for SSL termination:

```nginx
server {
    listen 443 ssl;
    server_name api.quillium.dev;

    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Monitoring and Logging

### Logging

The application logs are output to stdout/stderr in JSON format by default. In production environments, consider using a log aggregation service like ELK Stack, Graylog, or cloud-based solutions.

### Health Checks

The application provides a health check endpoint at `/health` that returns the status of the application and its dependencies.

Example response:
```json
{
  "status": "ok",
  "version": "1.0.0",
  "database": "connected",
  "uptime": "3h21m45s"
}
```

### Metrics

The application exposes Prometheus metrics at the `/metrics` endpoint when the `ENABLE_METRICS=true` environment variable is set.

## Scaling Considerations

### Horizontal Scaling

The backend is designed to be stateless, allowing for horizontal scaling. When scaling horizontally:

1. Ensure the database can handle the increased connection load
2. Consider using a connection pool manager like PgBouncer
3. Use a load balancer to distribute traffic

### Database Scaling

For high-load scenarios:

1. Consider read replicas for read-heavy workloads
2. Implement database sharding for very large datasets
3. Use connection pooling to manage database connections efficiently

## Backup and Disaster Recovery

### Database Backups

Set up regular PostgreSQL backups:

```bash
# Example daily backup script
pg_dump -U postgres quillium > /backups/quillium_$(date +%Y%m%d).sql
```

### Application State

The application state is stored entirely in the database, so database backups are sufficient for disaster recovery.

## Security Considerations

1. **Environment Variables**: Never commit environment variables to version control
2. **Database Access**: Use least-privilege database users
3. **API Keys**: Rotate API keys regularly
4. **JWT Secret**: Use a strong, randomly generated JWT secret
5. **Rate Limiting**: Implement rate limiting for API endpoints
6. **WAF**: Consider using a Web Application Firewall for production deployments

## Troubleshooting

### Common Issues

1. **Database Connection Errors**
   - Check the DATABASE_URL environment variable
   - Verify that the database server is running
   - Ensure network connectivity between the application and database

2. **JWT Errors**
   - Verify that the JWT_SECRET environment variable is set
   - Check for clock skew between servers

3. **Performance Issues**
   - Monitor database query performance
   - Check for slow API endpoints using the metrics
   - Consider adding indexes to frequently queried database columns

## Deployment Checklist

Before deploying to production, ensure:

1. ✅ Database migrations are applied
2. ✅ Environment variables are correctly set
3. ✅ SSL/TLS is configured
4. ✅ Backups are scheduled
5. ✅ Monitoring is in place
6. ✅ Security hardening measures are implemented
