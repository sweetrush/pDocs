### Webserver + Database Docker Cheat Sheet

---

#### **1. Basic Structure**
- Use `Docker Compose` for multi-container setup (webserver + database).
- Example services: `web` (Node.js/NGINX/Apache) + `db` (MySQL/PostgreSQL).

---

#### **2. Example `Dockerfile` (Web Server)**
```dockerfile
# Use a base image (e.g., Node.js, NGINX, or Apache)
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy application files
COPY package*.json ./
RUN npm install
COPY . .

# Expose port (e.g., 3000 for Node.js, 80 for NGINX/Apache)
EXPOSE 3000

# Start the server
CMD ["npm", "start"]
```

---

#### **3. Example `docker-compose.yml`**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "80:3000"  # Map host port 80 to container port 3000
    environment:
      DB_HOST: db
      DB_USER: user
      DB_PASSWORD: password
      DB_NAME: mydb
    depends_on:
      - db

  db:
    image: mysql:8.0  # or postgres:latest
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: mydb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"  # Expose MySQL port (optional)

volumes:
  db_data:  # Persistent storage for database
```

-----

#### **4. Environment Variables**
- **Web Server**: Configure your app to read:
  ```env
  DB_HOST=db
  DB_USER=user
  DB_PASSWORD=password
  DB_NAME=mydb
  ```
- **Database**: Set credentials in `docker-compose.yml` (e.g., `MYSQL_ROOT_PASSWORD`).

---

#### **5. Key Commands**
| Command | Description |
|---------|-------------|
| `docker-compose up -d` | Start containers in detached mode |
| `docker-compose build` | Rebuild images |
| `docker-compose down` | Stop and remove containers |
| `docker exec -it <container_id> bash` | Access container shell |
| `docker logs <container_id>` | View container logs |

---

#### **6. Common Issues**
- **Database Connection Refused**: Ensure the database service is running and the web service `depends_on` it.
- **Environment Variables**: Verify they match in `docker-compose.yml` and your app.
- **Port Conflicts**: Check for other services using ports `80`, `3306`, etc.

---

#### **7. Security Notes**
- Avoid hardcoding secrets in `docker-compose.yml`; use `.env` files:
  ```yaml
  environment:
    MYSQL_PASSWORD: ${DB_PASSWORD}
  ```
  Create a `.env` file:
  ```env
  DB_PASSWORD=your_secure_password
  ```
- Use HTTPS for the web server (e.g., Let’s Encrypt with NGINX).

---

#### **8. Extras**
- **Persistent Data**: Use Docker volumes (as shown) to retain database data.
- **Custom Networks**: Define a network for tighter control:
  ```yaml
  networks:
    my_network:
      driver: bridge
  ```
  Attach services to it using `networks: my_network`.

---

### Quick Start
1. Save the `Dockerfile` and `docker-compose.yml`.
2. Run `docker-compose up -d`.
3. Access the web server at `http://localhost:80`.

Adjust the ports, images, and environment variables for your stack (e.g., swap `mysql` for `postgres`).