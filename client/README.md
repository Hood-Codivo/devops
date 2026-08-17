# React + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Babel](https://babeljs.io/) for Fast Refresh
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/) for Fast Refresh

## Expanding the ESLint configuration

If you are developing a production application, we recommend using TypeScript with type-aware lint rules enabled. Check out the [TS template](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react-ts) for information on how to integrate TypeScript and [`typescript-eslint`](https://typescript-eslint.io) in your project.

The building blocks
Dockerfile — a recipe for building one image. You have two: server/Dockerfile and client/Dockerfile. Each says "start from this base, copy code in, install deps, run this command."

Image — the baked result of a Dockerfile. Doesn't run by itself, just sits there ready to be launched.

Container — a running instance of an image. This is the live process actually doing work.

docker-compose.yml — instead of manually running docker build and docker run separately for the backend, then again for the frontend, this file describes both services in one place, so one command builds and starts everything together, wired up correctly.

Walking through your docker-compose.yml line by line

services:
A list of the containers you want to run. You have two: backend and frontend.

backend:
build: ./server # build an image using ./server/Dockerfile
container_name: backend # name the container "backend" (instead of a random name)
restart: always # if it crashes, Docker restarts it automatically
env_file: ./server/.env # load env vars (like your Mongo connection string) from this file
ports: - "5000:5000" # map host port 5000 → container port 5000

frontend:
build: ./client
container_name: frontend
restart: always
ports: - "5173:5173"
depends_on: - backend # start backend first, before starting frontend
ports: "5000:5000" means: host machine port : container's internal port. Your app inside the container listens on 5000; this exposes it to localhost:5000 on your actual machine — same idea as the -p 5000:5000 flag you were using with docker run.

depends_on only controls start order (backend container starts before frontend container is created) — it doesn't wait for the backend app to actually be ready (e.g., DB connected), just for the container to have started.

What docker-compose up --build actually does
Reads docker-compose.yml to find the two services.
--build: forces Docker to (re)build both images from their Dockerfiles first — this is the compose equivalent of running docker build yourself, which is why you needed docker build manually before with plain docker run. Compose does it for you now.
Creates a container from each freshly-built image (backend, frontend).
Creates a shared network automatically, so the containers can talk to each other by service name (e.g., frontend could reach the backend at http://backend:5000 instead of localhost:5000, if configured that way).
Starts both containers, backend first (because of depends_on), streaming both containers' logs into your one terminal, prefixed by service name (backend |, frontend |) so you can see both at once.
How this differs from what you did before
Before (manual) Now (docker-compose up --build)
cd server && npm start and cd client && npm run dev in two terminals — or docker build/docker run twice by hand One command builds both images and starts both containers
You manage ports, names, restarts yourself Declared once in the YAML file, reused every time
No shared network between them by default Compose auto-creates a network so containers can reach each other by service name
Restart everything = repeat all commands Restart everything = same one command again
Useful related commands:

docker-compose up — start without rebuilding (uses existing images, faster if code hasn't changed)
docker-compose up -d — same, but detached (runs in background, gives you your terminal back)
docker-compose down — stops and removes both containers (and the network) in one shot
docker-compose logs -f backend — stream logs for just one service
docker-compose ps — list the containers this compose file manages and their status
