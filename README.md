# React + Vite + TypeScript + Bun

This template provides a minimal setup to get React working with Vite, TypeScript, and Bun, including Hot Module Replacement (HMR) and ESLint.

Bun is used as the JavaScript runtime and package manager.

## Project Structure

- `Dockerfile.init`: Dockerfile used for initializing the React + TypeScript project with Vite and Bun.
- `Dockerfile`: Dockerfile used for running the React application with Bun.
- `docker-compose.init.yml`: Docker Compose file for initializing the project.
- `docker-compose.yml`: Docker Compose file for running the project.
- `vite.config.ts`: Vite TypeScript configuration file.
- `tsconfig.json`: Main TypeScript configuration.
- `tsconfig.app.json`: TypeScript configuration for the React application.
- `tsconfig.node.json`: TypeScript configuration for Vite and Node-related files.

## Prerequisites

- Docker installed on your local machine.
- Docker Compose installed on your local machine.

Bun does not need to be installed locally because it runs inside Docker.

## Getting Started

### 1. Initialize the Project

Run the following command to initialize the React + TypeScript project with Vite:

```bash
docker compose -f docker-compose.init.yml run --rm init
```

The project uses the Vite `react-ts` template.

### 2. Configure Vite

The project uses `vite.config.ts`:

```ts
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

The server configuration is required when running Vite inside Docker:

- `host: '0.0.0.0'` makes Vite accessible outside the Docker container.
- `port: 5173` runs the Vite development server on port `5173`.

### 3. Build and Start the Application

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