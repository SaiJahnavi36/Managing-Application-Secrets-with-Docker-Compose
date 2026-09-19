# Managing Application Secrets with Docker Compose

## 📌 Overview

This project demonstrates how to securely manage sensitive application configuration such as database passwords using Docker Compose secrets.

The objective is to avoid hardcoding passwords directly inside application source code or Docker Compose configuration files. Instead, the password is stored separately and injected into containers at runtime.

## 🎯 Aim

To implement secure application secret management using Docker Compose and demonstrate how a backend application can securely read a database password from a secret file.

## 🎓 Learning Objectives

* Understand Docker secrets and secure configuration management.
* Avoid hardcoding sensitive information in application source code.
* Inject secrets into containers securely.
* Read secrets from `/run/secrets/`.
* Configure a Node.js backend with MySQL.
* Use Docker Compose to manage multiple services.
* Understand the difference between plain-text configuration and secret-based configuration.

## 🏗️ Project Architecture

```text
                    Docker Compose
                          |
             +------------+------------+
             |                         |
             ▼                         ▼
        Backend Service          MySQL Database
        Node.js + Express          MySQL 8.0
             |                         |
             +------------+------------+
                          |
                    db_password
                          |
                    Secret File
```

The backend and database services both receive access to the database password through the configured secret.

## 📁 Project Structure

```text
manage-secrets-demo/
│
├── docker-compose.yml
├── db_password.example.txt
├── .gitignore
├── README.md
│
└── backend/
    ├── Dockerfile
    ├── package.json
    └── server.js
```

> `db_password.txt` is intentionally excluded from Git using `.gitignore` because it contains the actual secret used for local execution.

## 🛠️ Technologies Used

* Docker
* Docker Compose
* Node.js
* Express.js
* MySQL 8.0
* JavaScript

## 🔐 Secret Management

Instead of writing the database password directly in the application or Compose environment, the password is stored separately.

The backend reads the secret from:

```text
/run/secrets/db_password
```

The application uses Node.js's file system module to read the secret at runtime.

## 🧩 Backend Implementation

The backend uses Express and MySQL2.

The important security-related code is:

```javascript
const fs = require("fs");

const dbPassword = fs
  .readFileSync("/run/secrets/db_password", "utf8")
  .trim();
```

The retrieved password is then used when establishing the MySQL connection:

```javascript
const db = mysql.createConnection({
  host: "database",
  user: "root",
  password: dbPassword,
  database: "securedb"
});
```

This prevents the password from being hardcoded directly into the source code.

## 🐳 Backend Dockerfile

The backend Dockerfile uses Node.js and installs the required dependencies before copying the application code.

```dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY server.js ./
EXPOSE 5000
CMD ["npm", "start"]
```

## 🐬 Database Configuration

The project uses MySQL 8.0.

The database is configured to read the root password from the secret file:

```yaml
MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_password
```

The database created for the application is:

```text
securedb
```

## ⚙️ Docker Compose Configuration

The Compose configuration contains two services:

* `backend`
* `database`

The backend receives the `db_password` secret:

```yaml
backend:
  build: ./backend
  ports:
    - "5000:5000"
  secrets:
    - db_password
  depends_on:
    - database
```

The database also receives the same secret:

```yaml
database:
  image: mysql:8.0
  environment:
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_password
    MYSQL_DATABASE: securedb
  ports:
    - "3306:3306"
  secrets:
    - db_password
```

The secret is defined using:

```yaml
secrets:
  db_password:
    file: ./db_password.txt
```

## 🚀 How to Run

Open PowerShell in the project directory:

```powershell
cd C:\Users\saija\Desktop\manage-secrets-demo
```

Build and start the services:

```powershell
docker compose up --build
```

Docker Compose builds the backend image and starts both the backend and MySQL services.

## 🌐 Test the Backend

After the containers start successfully, open:

```text
http://localhost:5000
```

Expected response:

```text
Backend is running with secure secret-based configuration
```

The backend should also display messages indicating that the server is running and that the database connection was established using the secret.

## 🔍 Verify Running Containers

Use:

```powershell
docker compose ps
```

This displays the status of the backend and database services.

## 📋 View Container Logs

To view all service logs:

```powershell
docker compose logs
```

To view only backend logs:

```powershell
docker compose logs backend
```

## 🔐 Verify Secret Injection

The backend reads the password from:

```text
/run/secrets/db_password
```

This demonstrates that the application receives the sensitive value through the secret mechanism rather than having the password hardcoded in `server.js`.

## 🔄 Stop the Application

To stop the running services:

```powershell
docker compose down
```

## 🔒 Security Best Practices Demonstrated

This project demonstrates the following practices:

* Do not hardcode passwords in application source code.
* Do not place database passwords directly in Docker Compose environment variables.
* Keep sensitive values separate from application code.
* Inject secrets at runtime.
* Use `.gitignore` to prevent actual secret files from being committed.
* Use an example secret file when publishing the project to GitHub.

## ❌ Unsafe Approach

An unsafe configuration would directly expose the password:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: rootpassword
```

This can expose the password in configuration files and version control.

## ✅ Secure Approach

The project instead uses:

```yaml
secrets:
  db_password:
    file: ./db_password.txt
```

and the application reads the secret from:

```text
/run/secrets/db_password
```

## 📸 Screenshots

The repository contains screenshots demonstrating:

1. Project structure
2. Docker Compose configuration
3. Backend implementation
4. Secret configuration
5. Docker image build
6. Running containers
7. Backend output
8. Database connection
9. Secret-based configuration

Screenshots are included as practical evidence of the experiment execution.

## 🧪 Result

The Docker Compose application was successfully configured with a backend and MySQL database.

The database password was managed separately from the application source code and made available to the containers through a secret file.

The backend successfully read the password from `/run/secrets/db_password` and used it for database connectivity.

## 📚 Key Learning

The experiment demonstrates an important DevOps security principle:

> Sensitive information should not be hardcoded into application source code or normal configuration files.

Instead, secrets should be separated from application code and injected securely at runtime.

## 🏁 Conclusion

The experiment successfully demonstrates secure application secret management using Docker Compose.

By separating the database password from the application code and using secret-based configuration, the project follows a safer approach to managing sensitive information in containerized applications.
