# React + Vite + Bun + Docker

This project provides a minimal development environment for building a React application using **Vite**, **Bun**, and **Docker**.

The development stack consists of:

- React
- Vite
- Bun
- Docker
- Docker Compose
- ESLint

Vite provides a fast development server with Hot Module Replacement (HMR), while Bun is used as the JavaScript runtime and package manager.

Bun does not need to be installed on the host machine because it runs entirely inside the Docker container.

---

## Project Structure

The project uses two Docker configurations: one for initializing the React application and another for running the development environment.

```text
.
├── src/
│   ├── assets/
│   ├── App.css
│   ├── App.jsx
│   ├── index.css
│   └── main.jsx
│
├── public/
│
├── Dockerfile
├── Dockerfile.init
├── docker-compose.yml
├── docker-compose.init.yml
├── eslint.config.js
├── index.html
├── package.json
├── bun.lock
├── vite.config.js
└── README.md
```

### Main Files

- `Dockerfile.init` — Dockerfile used to initialize the React + Vite project with Bun.
- `Dockerfile` — Dockerfile used to run the React development application.
- `docker-compose.init.yml` — Docker Compose configuration used during project initialization.
- `docker-compose.yml` — Docker Compose configuration used for normal development.
- `vite.config.js` — Vite development server configuration.
- `package.json` — Project dependencies and scripts.
- `bun.lock` — Bun dependency lock file.

---

## Prerequisites

Install the following tools on your local machine:

- Docker
- Docker Compose

You do **not** need to install Node.js, npm, Yarn, pnpm, or Bun locally.

Bun runs inside the Docker container.

---

# Getting Started

## 1. Initialize the React Project

The project is initialized using `Dockerfile.init` and `docker-compose.init.yml`.

### `Dockerfile.init`

```dockerfile
FROM oven/bun:latest

WORKDIR /app

CMD ["bun", "create", "vite", ".", "--template", "react"]
```

### `docker-compose.init.yml`

```yaml
services:
  init:
    build:
      context: .
      dockerfile: Dockerfile.init

    volumes:
      - .:/app

    stdin_open: true
    tty: true
```

Run the initialization container:

```bash
docker compose -f docker-compose.init.yml run --rm init
```

This creates a new React application using the Vite React template.

If necessary, install the project dependencies with Bun:

```bash
docker compose -f docker-compose.init.yml run --rm init bun install
```

After initialization, the project will contain files such as:

```text
src/
public/
index.html
package.json
vite.config.js
eslint.config.js
bun.lock
```

---

# 2. Configure Vite

Because Vite runs inside a Docker container, its development server must listen on all network interfaces.

Modify `vite.config.js`:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],

  server: {
    host: '0.0.0.0',
    port: 5173,
  },
})
```

The important configuration is:

```javascript
server: {
  host: '0.0.0.0',
  port: 5173,
}
```

`host: '0.0.0.0'` allows the Vite development server running inside Docker to be accessed from the host machine.

The application runs on port:

```text
5173
```

---

# 3. Development Dockerfile

Create the main `Dockerfile`:

```dockerfile
FROM oven/bun:latest

WORKDIR /app

COPY package.json bun.lock* ./

RUN bun install

COPY . .

EXPOSE 5173

CMD ["bun", "run", "dev", "--host", "0.0.0.0"]
```

This Dockerfile:

1. Uses the official Bun Docker image.
2. Sets `/app` as the working directory.
3. Copies the dependency files.
4. Installs dependencies using Bun.
5. Copies the application source code.
6. Exposes port `5173`.
7. Starts the Vite development server.

---

# 4. Docker Compose

Create the main `docker-compose.yml`:

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile

    ports:
      - "5173:5173"

    volumes:
      - .:/app
      - /app/node_modules

    stdin_open: true
    tty: true
```

The project directory is mounted into `/app`, which allows source-code changes on the host machine to immediately become available inside the container.

The separate:

```yaml
- /app/node_modules
```

volume prevents the host bind mount from overwriting the dependencies installed inside the container.

---

# 5. Build and Start the Application

Build the Docker image and start the application:

```bash
docker compose up -d --build
```

Check that the container is running:

```bash
docker compose ps
```

The React application should now be available at:

```text
http://localhost:5173
```

---

# 6. View Application Logs

To follow the Vite development server logs:

```bash
docker compose logs -f app
```

You should see output similar to:

```text
VITE ready

Local:   http://localhost:5173/
Network: http://172.x.x.x:5173/
```

Press:

```text
Ctrl + C
```

to stop following the logs.

The container will continue running in the background.

---

# 7. Stop the Application

Stop and remove the running containers:

```bash
docker compose down
```

---

# 8. Restart the Application

Restart the application:

```bash
docker compose restart app
```

Or stop and start the complete environment:

```bash
docker compose down
docker compose up -d
```

---

# 9. Rebuild the Application

If you modify the `Dockerfile`, `package.json`, or dependencies, rebuild the image:

```bash
docker compose down
docker compose up -d --build
```

You can also force a complete rebuild without using Docker's build cache:

```bash
docker compose build --no-cache
docker compose up -d
```

---

# 10. Install a New Package

Packages should be installed using Bun.

For example, to install Axios:

```bash
docker compose exec app bun add axios
```

To install a development dependency:

```bash
docker compose exec app bun add -d <package-name>
```

For example:

```bash
docker compose exec app bun add -d prettier
```

Bun will update:

```text
package.json
bun.lock
```

---

# 11. Remove a Package

Remove a dependency with:

```bash
docker compose exec app bun remove <package-name>
```

For example:

```bash
docker compose exec app bun remove axios
```

---

# 12. Run Bun Commands

Commands can be executed directly inside the running container.

For example:

```bash
docker compose exec app bun --version
```

Run a package script:

```bash
docker compose exec app bun run <script>
```

For example:

```bash
docker compose exec app bun run lint
```

---

# 13. Open a Shell Inside the Container

To enter the running application container:

```bash
docker compose exec app bash
```

You can then run commands directly:

```bash
bun --version
bun run dev
bun run lint
```

Exit the container with:

```bash
exit
```

---

# 14. Hot Module Replacement

Vite supports Hot Module Replacement (HMR).

Because the source directory is mounted into the container:

```yaml
volumes:
  - .:/app
```

changes to files such as:

```text
src/App.jsx
src/App.css
src/index.css
```

should automatically appear in the browser without rebuilding the Docker image.

For normal source-code changes, you therefore do **not** need to run:

```bash
docker compose up -d --build
```

again.

Simply edit the source code and Vite will reload the application.

---

# 15. ESLint

The Vite React template includes ESLint.

Run ESLint with:

```bash
docker compose exec app bun run lint
```

The ESLint configuration can be found in:

```text
eslint.config.js
```

---

# 16. Production Build

Create a production build with:

```bash
docker compose exec app bun run build
```

Vite generates the production application inside:

```text
dist/
```

The production build can be previewed with:

```bash
docker compose exec app bun run preview -- --host 0.0.0.0
```

---

# Useful Commands

| Task | Command |
|---|---|
| Initialize project | `docker compose -f docker-compose.init.yml run --rm init` |
| Build and start | `docker compose up -d --build` |
| Start | `docker compose up -d` |
| Stop | `docker compose down` |
| Restart | `docker compose restart app` |
| View containers | `docker compose ps` |
| View logs | `docker compose logs -f app` |
| Open container shell | `docker compose exec app bash` |
| Install package | `docker compose exec app bun add <package>` |
| Remove package | `docker compose exec app bun remove <package>` |
| Run ESLint | `docker compose exec app bun run lint` |
| Production build | `docker compose exec app bun run build` |
| Check Bun version | `docker compose exec app bun --version` |

---

# Development Stack

```text
Browser
   │
   │ http://localhost:5173
   ▼
Docker
   │
   ▼
Bun
   │
   ▼
Vite
   │
   ▼
React
```

The final development stack is therefore:

**React + Vite + Bun + Docker**

with Vite serving the development application on port `5173`.