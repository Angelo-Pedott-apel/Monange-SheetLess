# Docker Setup Instructions

## Project Structure

```
project-root/
├── backend/
│   ├── Dockerfile
│   ├── pom.xml
│   ├── Domain/
│   ├── Application/
│   ├── Infrastructure/
│   └── Api/
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── pubspec.yaml
│   └── ... (Flutter files)
├── docker-compose.yml
├── .env.example
└── SETUP.md (this file)
```

## Prerequisites

- Docker Desktop installed (includes Docker and Docker Compose)
- At least 4GB of free RAM
- Ports 5432, 8080, and 8081 available

## Quick Start

### 1. Clone the Repositories inside the correct folders

Make sure you are in the root folder of the project.
Then create the folder structure and clone each repository into its respective directory:

```bash
git clone https://github.com/Angelo-Pedott-apel/Monange-SheetLess-BackEnd.git BackEnd

git clone https://github.com/Angelo-Pedott-apel/Monange-SheetLess-FrontEnd.git FrontEnd
```


### 2. Setup Environment Variables

Copy the example environment file:

```bash
cp .env.example .env
```

Edit `.env` and update the values, especially:
- `JWT_SECRET` - Use a strong, random 32+ character string
- `POSTGRES_PASSWORD` - Use a strong password in production

### 3. Build and Start All Services

```bash
docker-compose up -d --build
```

This will:
- Build the backend Spring Boot application
- Build the frontend Flutter web application
- Pull and start PostgreSQL database
- Create necessary networks and volumes

### 4. Verify Services Are Running

Check the status of all containers:

```bash
docker-compose ps
```

View logs for all services:

```bash
docker-compose logs -f
```

View logs for a specific service:

```bash
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f postgres
```

## Service Access Points

### Frontend (Flutter Web)
- **URL**: http://localhost:8081
- **Description**: Web interface for the application
- Access through your browser

### Backend (Spring Boot API)
- **URL**: http://localhost:8080
- **API Base**: http://localhost:8080/api/v1
- **Description**: REST API backend

### Database (PostgreSQL)
- **Host**: localhost
- **Port**: 5432
- **Database**: finance_db
- **Username**: finance_user
- **Password**: finance_pass (from .env)
- **Description**: PostgreSQL database
- Connect with DBeaver, pgAdmin, or any PostgreSQL client

#### PostgreSQL client Connection Settings:
1. New PostgreSQL connection
2. Host: `localhost`
3. Port: `5432`
4. Database: `finance_db`
5. Username: `finance_user`
6. Password: `finance_pass`

## Common Commands

### Start Services
```bash
docker-compose up -d
```

### Stop Services
```bash
docker-compose down
```

### Stop Services and Remove Volumes (WARNING: Deletes data)
```bash
docker-compose down -v
```

### Rebuild Services
```bash
docker-compose up -d --build
```

### Rebuild Specific Service
```bash
docker-compose up -d --build backend
```

### View Logs
```bash

docker-compose logs -f

docker-compose logs -f backend

docker-compose logs --tail=100 backend
```

### Execute Commands in Container
```bash
docker-compose exec backend sh


docker-compose exec postgres psql -U finance_user -d finance_db
```

### Restart a Service
```bash
docker-compose restart backend
```

## Troubleshooting

### Backend Won't Start

1. Check if database is ready:
   ```bash
   docker-compose logs postgres
   ```

2. Check backend logs:
   ```bash
   docker-compose logs backend
   ```

3. Verify database connection:
   ```bash
   docker-compose exec backend sh
   # Inside container:
   wget -O- http://postgres:5432
   ```

### Frontend Can't Connect to Backend

1. Check if backend is healthy:
   ```bash
   curl http://localhost:8080/actuator/health
   ```

2. Verify CORS settings in backend configuration

3. Check frontend environment variables:
   ```bash
   docker-compose exec frontend cat /usr/share/nginx/html/.env
   ```

### Database Connection Issues

1. Verify PostgreSQL is running:
   ```bash
   docker-compose ps postgres
   ```

2. Test connection:
   ```bash
   docker-compose exec postgres pg_isready -U finance_user
   ```

3. Check if port 5432 is available:
   ```bash
   # Linux/Mac
   lsof -i :5432
   
   # Windows
   netstat -ano | findstr :5432
   ```

### Port Conflicts

If ports are already in use, edit `docker-compose.yml` to change:
- Frontend: Change `8081:80` to `<NEW_PORT>:80`
- Backend: Change `8080:8080` to `<NEW_PORT>:8080`
- Database: Change `5432:5432` to `<NEW_PORT>:5432`

### Reset Everything

To completely reset and start fresh:

```bash
# Stop and remove containers, networks, and volumes
docker-compose down -v

# Remove any dangling images
docker system prune -a

# Rebuild and start
docker-compose up -d --build
```

## Network Architecture

```
┌─────────────────┐
│   Browser       │
│  (localhost)    │
└────────┬────────┘
         │
         │ Port 8081
         ▼
┌─────────────────┐
│   Frontend      │
│   (Flutter)     │
└────────┬────────┘
         │
         │ API calls
         │ Port 8080
         ▼
┌─────────────────┐
│   Backend       │
│  (Spring Boot)  │
└────────┬────────┘
         │
         │ JDBC
         │ Port 5432
         ▼
┌─────────────────┐
│   PostgreSQL    │
│   (Database)    │
└─────────────────┘
```

All services are connected via the `app-network` Docker network.

## Support

For issues or questions:
1. Check the logs: `docker-compose logs`
2. Verify all services are healthy: `docker-compose ps`
3. Review this documentation
4. Check Docker and Docker Compose versions