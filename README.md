# React + Vite + Bun

This template provides a minimal setup to get React working with Vite and Bun, including Hot Module Replacement (HMR) and ESLint.

Bun is used as the JavaScript runtime and package manager instead of npm.

## Project Structure

- `Dockerfile.init`: Dockerfile used for initializing the React project with Vite and Bun.
- `Dockerfile`: Dockerfile used for running the React project with Bun.
- `docker-compose.init.yml`: Docker Compose file for initializing the project.
- `docker-compose.yml`: Docker Compose file for running the project.
- `vite.config.js`: Vite configuration file.

## Prerequisites

- Docker installed on your local machine.
- Docker Compose installed on your local machine.

Bun does not need to be installed locally because it runs inside Docker.

## Getting Started

### 1. Initialize the Project

Run the following command to initialize the React project with Vite:

```bash
docker compose -f docker-compose.init.yml run --rm init
```

### 2. Configure Vite

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

The `plugins` option enables React support through `@vitejs/plugin-react`.

The `server` configuration is important when running Vite inside Docker:

- `host: '0.0.0.0'` makes the Vite development server accessible outside the Docker container.
- `port: 5173` runs Vite on its default development port.

The Docker Compose configuration should therefore expose port `5173:5173`.

### 3. Build and Start the Application

Use the main `docker-compose.yml` file to build and start the application:

```bash
docker compose up -d --build
```

The application will be available at:

```text
http://localhost:5173
```

### 4. View Logs

```bash
docker compose logs -f app
```

### 5. Stop the Application

```bash
docker compose down
```