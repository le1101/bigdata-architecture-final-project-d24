# 🐳 Flask + Kafka Sensor Pipeline (Dockerized)

A production-friendly **Flask** starter (based on Nick Janetakis' Docker example) **extended with a Smart Device Sensor pipeline** using **Kafka**. Use this repo as a base for your next project or as a guide to Dockerize an existing Flask app and add streaming.

The example app is minimal but it wires up a number of things you might use in a real world Flask app, while avoiding excessive opinions.

For the Docker bits, everything included is an accumulation of [Docker best practices](https://nickjanetakis.com/blog/best-practices-around-production-ready-web-apps-with-docker-compose) based on building and deploying dozens of Dockerized web apps since late 2014.

**This app is using Flask 3.1.2 and Python 3.13.7.** The screenshot shows `X.X.X` since they get updated regularly:

[![Screenshot](.github/docs/screenshot.jpg)](https://github.com/nickjj/docker-flask-example/blob/main/.github/docs/screenshot.jpg?raw=true)

---

## 🧾 Table of Contents

- [Tech stack](#tech-stack)
- [Notable opinions and extensions](#notable-opinions-and-extensions)
- [🚀 Quick start (Docker)](#-quick-start-docker)
- [Files of interest](#files-of-interest)
  - [`.env`](#env)
  - [`run`](#run)
- [Running a script to automate renaming the project](#running-a-script-to-automate-renaming-the-project)
- [Updating dependencies](#updating-dependencies)
- [See a way to improve something?](#see-a-way-to-improve-something)
- [Additional resources](#additional-resources)
  - [Learn more about Docker and Flask](#learn-more-about-docker-and-flask)
  - [Deploy to production](#deploy-to-production)
- [About the author](#about-the-author)

## 🧬 Tech stack

If you don't like some of these choices that's no problem—you can swap them out.

### Back-end
- [PostgreSQL](https://www.postgresql.org/)
- [SQLAlchemy](https://github.com/sqlalchemy/sqlalchemy)
- [Redis](https://redis.io/)
- [Celery](https://github.com/celery/celery)
- **Kafka** (Confluent images) for streaming sensor data

### Front-end
- [esbuild](https://esbuild.github.io/)
- [TailwindCSS](https://tailwindcss.com/)
- [Heroicons](https://heroicons.com/)

#### But what about JavaScript?!
Use whatever you like. Good options depending on needs:
- <https://hotwired.dev/> / <https://htmx.org/> / <https://github.com/alpinejs/alpine>
- <https://vuejs.org/> / <https://reactjs.org/> / <https://jquery.com/>

With esbuild set up, adopting any of these is straightforward.

## 🍣 Notable opinions and extensions

- **Packages and extensions**:
  - *[gunicorn](https://gunicorn.org/)* for the app server
  - *[Flask-DB](https://github.com/nickjj/flask-db)* for managing/migrating/ seeding the DB
  - *[Flask-Static-Digest](https://github.com/nickjj/flask-static-digest)* for cache-busted static assets
  - *[Flask-Secrets](https://github.com/nickjj/flask-secrets)* for generating secrets
  - *[Flask-DebugToolbar](https://github.com/flask-debugtoolbar/flask-debugtoolbar)* for debugging
- **Linting, formatting and testing**:
  - *[ruff](https://github.com/astral-sh/ruff)*
  - *[pytest](https://github.com/pytest-dev/pytest)* and *pytest-cov*
- **Blueprints**:
  - `page` blueprint for `/`
  - `up` blueprint for health checks
- **Config**:
  - Logs to STDOUT for Docker
  - Most settings come from the `.env`
- **Front-end assets**:
  - `assets/` managed by esbuild
  - Custom `502.html` and `maintenance.html`
  - Modern favicon generation
- **Flask defaults changed**:
  - `public/` serves static files
  - `static_url_path` is `""` (no `/static` prefix)
  - `ProxyFix` middleware is enabled

Besides the Flask app itself:
- [uv](https://github.com/astral-sh/uv) for Python package management
- Docker + GitHub Actions are configured

---

## 🚀 Quick start (Docker)

This project uses Docker Compose to run **Flask + Kafka + Airflow + Postgres + Redis**.

### Prerequisites
- Docker + Docker Compose v2.20.2+
- On Windows, use WSL2
- Ports used: `8000` (Flask), `8080` (Airflow), `9093` (Kafka host listener)

### 1) Clone and set up environment
```sh
git clone <YOUR_REPO_URL> my-flask-kafka
cd my-flask-kafka

# create .env from the template if needed
cp .env.example .env 2>/dev/null || true
```

### 2) Prepare Airflow directories (first run or after wiping volumes)
```sh
mkdir -p ./airflow/dags ./airflow/logs ./airflow/plugins ./airflow/config
```

### 3) Initialize Airflow (one-time)
```sh
docker compose up airflow-init
```
Wait for `Airflow init done.` and the container to exit with code 0.

### 4) Start the stack
```sh
docker compose up -d   # add --build on first run or after dependency changes
```

### 5) Verify
```sh
docker ps
```
- **Flask** → <http://localhost:8000>  
- **Airflow** → <http://localhost:8080> (user: `airflow` / pass: `airflow`)  
- **Kafka** (inside docker) → `kafka:9092`, from host → `localhost:9093`  

### 6) Optional: run the Kafka producer manually
If you didn't add a `producer` service to `docker-compose.yml`:
```sh
docker compose run --rm web python producer.py
```

### Troubleshooting
- **Compose error about `depends_on`** → upgrade Docker Compose to v2.20.2+ / Docker Desktop 4.22.0+
- **Port 8000 in use** → adjust `DOCKER_WEB_PORT_FORWARD` in `.env`
- **Permission denied writing files** (Linux) → set `UID` and `GID` in `.env`
- **After deleting images/volumes** → repeat steps 2–4

---

## 🔍 Files of interest

Review the `.env` and `run` scripts before diving deeper. Search for `TODO:` in the code for pointers.

### `.env`  {#env}
Defines environment variables that control the app and services. Add secrets and per-environment config here.

### `run`  {#run}
A convenience script (like a makefile) with helpful commands: `./run lint`, `./run format`, `./run test`, etc.

---

## ✨ Running a script to automate renaming the project

The app is currently named `hello`. Use the included script to rename everything:

```sh
bin/rename-project myapp MyApp
```

It will remove old Docker resources, run find/replace, and optionally init a new git repo.

Then start things up again:

```sh
docker compose up --build
./run flask db reset --with-testdb
./run quality
./run test
```

---

## 🛠 Updating dependencies

- Check what’s outdated: `./run uv:outdated` or `./run yarn:outdated`
- Update Python deps by editing `pyproject.toml` (or `./run uv add ... --no-sync`), then `./run deps:install`
- Update Node deps by editing `assets/package.json` (or `./run yarn add ... --no-lockfile`), then `./run deps:install`
- In CI / production, `docker compose build` uses existing lockfiles

---

## 🤝 See a way to improve something?

Open an issue or PR—thanks!

---

## 🌎 Additional resources

- <https://docs.docker.com/>
- <https://flask.palletsprojects.com/>

Courses:
- <https://diveintodocker.com>
- <https://buildasaasappwithflask.com>

### Deploy to production
If you plan to deploy, consider managed Postgres/Redis/Kafka and a hardened Compose or Kubernetes setup.

---

## 👀 About the author

- Nick Janetakis | <https://nickjanetakis.com> | [@nickjanetakis](https://twitter.com/nickjanetakis)


## 📂 Project structure

Here’s an overview of the main project layout and responsibilities:

```
bigdata-architecture-final-project-d24/
│
├── airflow/                 # Airflow configs, DAGs, logs, plugins
├── assets/                  # Front-end assets (CSS, JS, static files)
├── bin/                     # Helper scripts (entrypoints, rename, uv-install)
├── data/                    # JSON sensor data examples
├── db/                      # Database migrations and seeds
│   ├── versions/            # Alembic migration scripts
│   └── seeds.py             # Example DB seeding
├── hello/                   # Main Flask app package
│   ├── api/                 # REST API (capteurs, v1 routes, errors)
│   │   ├── resources/       # Resource handlers (Flask-RESTful)
│   │   └── v1/              # Versioned API blueprints
│   ├── page/                # Template-based routes
│   ├── up/                  # Health check routes
│   ├── config/              # Settings (env-driven config)
│   ├── app.py               # App factory / entrypoint
│   ├── extensions.py        # Flask extensions initialization
│   └── initializers.py      # App initialization logic
├── producer/                # Kafka producer logic
│   └── capteur_producer.py  # Script sending sensor data to Kafka
├── public/                  # Static assets served directly (favicon, 502.html)
├── templates/               # Shared Jinja templates
├── lib/                     # Utilities, tests, misc code
├── .env                     # Local environment variables
├── compose.yaml             # Docker Compose stack (Flask, Kafka, Airflow, etc.)
├── Dockerfile               # Flask app container build
├── pyproject.toml           # Python dependencies (uv-managed)
├── run                      # Dev helper script (lint, format, test, deps)
└── README.md                # Project documentation
```

---
