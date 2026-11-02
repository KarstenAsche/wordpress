# WordPress with Docker Compose

A reproducible WordPress environment using the official WordPress Apache image, MariaDB, and Docker Compose. The project demonstrates declarative service configuration, environment-based settings, container networking, named-volume persistence, and restart policies.

## Table of Contents

- [Project Objectives](#project-objectives)
- [Quick Start](#quick-start)
- [Repository Contents](#repository-contents)
- [Architecture](#architecture)
- [Configuration](#configuration)
- [Usage](#usage)
- [Testing](#testing)
- [Cloud VM Deployment](#cloud-vm-deployment)
- [Security Notes](#security-notes)

## Project Objectives

- Understand containers and practical container use cases.
- Configure a prebuilt application and its backing database service.
- Manage an application declaratively with Docker Compose.
- Keep persistent application and database data across container recreation.
- Verify automatic recovery after unexpected container process termination.

## Quick Start

### Prerequisites

- Docker Engine with the Docker Compose plugin
- Git
- Free host port **8080**

### Run locally

```bash
git clone https://github.com/KarstenAsche/wordpress.git
cd wordpress
cp .env.example .env
```

Edit `.env` and replace **all placeholder passwords** with strong, unique passwords. Then run:

```bash
docker compose config --quiet
docker compose up -d
docker compose ps
```

Open **http://localhost:8080**. Complete the WordPress installation wizard, including the site title and administrator credentials. The official WordPress image does not create the administrator from the `.env` file.

## Repository Contents

| File | Purpose |
| --- | --- |
| `docker-compose.yml` | Defines WordPress and MariaDB services, configuration, ports, volumes, and restart behavior. |
| `.env.example` | Shareable template for required database settings. |
| `.env` | Local credentials and configuration; **not committed to Git**. |
| `.gitignore` | Excludes secret and unnecessary local files from version control. |
| `README.md` | Setup, configuration, usage, and test instructions. |

> [!NOTE]
> No Dockerfile or custom application source is required: the project uses prebuilt container images. The `.env` file is created locally and is intentionally absent from the repository.

## Architecture

- **WordPress:** Official `wordpress:php8.3-apache` image; Apache serves HTTP on port `80` inside its container.
- **MariaDB:** `mariadb:11.8` image; WordPress connects to the database at hostname `db`, port `3306`.
- **Networking:** Docker Compose automatically creates a default network, and connects both the WordPress and MariaDB services to it. Wordpress communicates with MariaDB using the service name db on the internal port 3306. No additional network configuration or exposed databse port is required.
- **Storage:** Named Docker volumes preserve WordPress files and the MariaDB data directory.
- **Recovery:** Both services use `restart: unless-stopped`.

The host's port `8080` maps to WordPress's container port `80` (`8080:80`). A host-based Nginx installation can coexist with Apache inside the WordPress container.

## Configuration

### Environment variables

Copy `.env.example` to `.env` before starting the application.

| Variable | Description |
| --- | --- |
| `DB_NAME` | MariaDB database used by WordPress. |
| `DB_USER` | MariaDB application user. |
| `DB_PASSWORD` | Password for the application database user. |
| `DB_ROOT_PASSWORD` | MariaDB root password for database initialization. |

Docker Compose substitutes these values into the service definitions. WordPress uses `WORDPRESS_DB_HOST=db`, `WORDPRESS_DB_NAME`, `WORDPRESS_DB_USER`, and `WORDPRESS_DB_PASSWORD`; MariaDB uses the corresponding `MARIADB_*` settings.

### How to customize the setup

- **Change the HTTP port:** Change the left side of `"8080:80"` in the WordPress service, for example `"8081:80"`, then visit `http://localhost:8081` for a new installation.
- **Change the database name or user:** Update `.env` **before the first database initialization**. MariaDB initialization variables do not automatically modify an already initialized database stored in a volume.
- **Change image versions:** Edit the `image:` tags in `docker-compose.yml`, then review release notes and run `docker compose up -d`. Back up data before an upgrade. For strict reproducibility, pin and verify immutable image digests.
- **Change restart behavior:** Edit `restart: unless-stopped` on the relevant service. This policy restarts the service after unexpected termination, but respects a manual stop.
- **Change persistence:** The named volumes mount WordPress at `/var/www/html` and MariaDB at `/var/lib/mysql`. Do not remove them if you need to retain existing data.

>[!IMPORTANT] 
> Changing `.env` credentials after initialization does not automatically update the existing MariaDB user's password or the WordPress administrator's password.

## Usage

### Start and inspect

```bash
docker compose up -d
docker compose ps
docker compose logs --tail=50
```

### Follow logs for one service

```bash
docker compose logs -f wordpress
docker compose logs -f db
```

### Stop without deleting persisted data

```bash
docker compose down
```

> [!CAUTION]
> Restart with `docker compose up -d`. The named volumes are retained.

### Important: deleting data

> [!CAUTION] 
> `docker compose down -v` also removes the Compose-managed named volumes **Do not use it when you want to preserve the WordPress installation content, or database.**

## Testing

The WordPress Docker Compose setup was successfully tested both locally and on an Ubuntu-based cloud VM.

### Test Results

| Test | Local | Cloud VM |
| :--- | :--- | :--- |
| Docker Compose configuration validation | Passed | Passed |
| WordPress installation and accessibility | Passed | Passed |
| MariaDB database connectivity | Passed | Passed |
| WordPress administrator login | Passed | Passed |
| Data persistence after container recreation | Passed | Passed |
| Automatic container restart | Passed | Passed |
| Access via public IP on port 8080 | N/A | Passed |

---

### Data Persistence

Data persistence was verified by creating and publishing a WordPress post named `Docker Persistence Test`.

The containers were then stopped, removed, and recreated using:

```bash
docker compose down
docker compose up -d
```

After restarting the environment, the published post, administrator account, and WordPress configuration remained available.

**Result:** Passed.

---

### Automatic Container Restart

The automatic restart functionality was tested using the `unless-stopped` restart policy.

The Apache process inside the WordPress container was terminated using:

```bash
docker compose exec -u root wordpress apache2ctl -k stop
```

Docker automatically restarted the container. The restart counter increased from 0 to 1, confirming that the restart policy worked as expected.

**Result:** Passed.

### 5. Public Cloud VM access

The WordPress application was successfully deployed and tested on an Ubuntu-based cloud VM.
The following checks were completed:
- WordPress is accessible via the public server IP on port 8080.
- Administrator authentication works correctly.
- WordPress successfully communicates with MariaDB.
- WordPress data remains persistent after container recreation.
- Automatic container recovery was successfully tested.

Cloud VM result: Passed — all required deployment and functionality tests were completed successfully.


## Deployment Verification

The deployment was successfully completed and verified on an Ubuntu-based cloud VM.
WordPress is accessible through the server's public IP address on port 8080. Administrator authentication, database connectivity, persistent storage, and automatic container recovery were successfully tested.
The deployment and verification process was also documented in a video recording.
**Result**: Passed

## Security Notes

- Never commit `.env`, passwords, SSH keys, or credentials. Commit only `.env.example` with placeholders.
- Docker environment variables are useful for this course but are **not** a complete secret-management solution; users with Docker access may be able to inspect them.
- Do not publish MariaDB port `3306` to the host unless explicitly needed.
- Use strong, unique credentials and change any credential accidentally exposed in a repository or chat.
- Maintain backups of both named volumes before changing versions or performing destructive operations.
- Keep container images, WordPress, themes, and plugins updated. The current project is a training environment, not a hardened production deployment.
