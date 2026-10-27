
# Assignment 2: Multi-Container Application Deployment

A Docker Compose project with three Flask services (books-api, staff-api, readers-api), a MySQL 8.0 database, and an Nginx reverse proxy.

## Setup and Deployment Instructions

1. Install Docker Desktop.
2. In the project folder, create a file called .env with these four lines and your own values:

```
DB_USER=your_user
DB_PASSWORD=your_password
DB_ROOT_PASSWORD=your_root_password
DB_NAME=your_database_name
```

3. Build and start everything:

```
docker compose up -d --build
```

4. Check that all five containers are running. The three APIs should say healthy after about a minute:

```
docker compose ps
```

5. Test Nginx:

```
curl http://localhost/health
```

It should print `healthy`.

6. To stop everything:

```
docker compose down
```

## API Endpoints

| URL | What it does |
|---|---|
| http://localhost/health | Nginx health check, returns `healthy` |
| /health on each service | Returns `{"status": "healthy"}`. It runs inside the Docker network on port 5000. |
| http://localhost/api/books, /api/staff, /api/readers | Nginx forwards these to books-api, staff-api and readers-api. The apps only have /health so far, so Flask returns a 404 page. That shows the request reached the right service. |

## Configuration Choices

- Two named networks, app_frontend and app_backend. Nginx is only on the frontend, the database is only on the backend, and the three APIs are on both.
- Only Nginx publishes a port to my computer (80). The other containers are not open to the outside.
- The database data is stored in a named volume, app_db_data, so it stays if the container is removed.
- Database settings come from the .env file, so no passwords are written in docker-compose.yml.
- The Dockerfiles install curl because the Compose health check uses it.

## Troubleshooting Guide

| Problem | Fix |
|---|---|
| The db container does not load |The .env file is missing or empty. Create it and run `docker compose up -d` again. |
| The APIs show unhealthy | The health check needs curl inside the image. Rebuild with `docker compose up -d --build`. |
| Port 80 is already in use | Stop the other program using port 80, then start the stack again. |