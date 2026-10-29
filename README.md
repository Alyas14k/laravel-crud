## Overview

> **Note:** This project targets PlanetScale Vitess/MySQL. PlanetScale also offers managed Postgres. For more information, see the [PlanetScale Postgres documentation](https://planetscale.com/docs/postgres).

This is the final repo for the Laravel MySQL Mood Tracker project showing PlanetScale integration. You can find the [full tutorial on the PlanetScale blog](https://planetscale.com/blog/build-laravel-crud-app).

<img width="1178" alt="mood-tracker" src="https://user-images.githubusercontent.com/2941081/153310727-a4e8684c-d260-4543-969f-fca489a39a58.png">

## Run the application

### Run with Docker Compose

1. Copy the example environment file and set a strong `DB_PASSWORD`:

   ```bash
   cp .env.example .env
   ```

2. Generate an application key:

   ```bash
   docker compose run --rm app php artisan key:generate --show
   ```

   Copy the printed `base64:...` value into `APP_KEY` in `.env`.

3. Build and start the application:

   ```bash
   docker compose up --build -d
   ```

4. Create the database tables and seed the example data:

   ```bash
   docker compose exec app php artisan migrate --seed --force
   ```

5. Open [http://localhost:18080](http://localhost:18080).

PostgreSQL 16 data is stored in the `postgres_data` Docker volume. The Laravel
application is served by Apache with PHP in the `app` container.

### GitHub Actions and image registry

The `Laravel CI/CD` workflow runs checks and tests for pushes and pull requests.
It publishes the Docker image to GitHub Container Registry (GHCR) as
`ghcr.io/OWNER/REPOSITORY:latest` and `ghcr.io/OWNER/REPOSITORY:sha-COMMIT`
after a successful push to `main`. The workflow authenticates with its
`GITHUB_TOKEN`; no personal access token is needed to publish.

To find and configure the image:

1. Push a commit to `main` and open the repository's **Packages** page, or visit
   `https://ghcr.io/OWNER/REPOSITORY`.
2. In the package settings, connect the package to this repository and grant
   this repository Actions access to the package. This lets the deployment job
   pull the image with its read-only `GITHUB_TOKEN`.
3. Choose whether the package should be public. For a private package, make
   sure the production server pulls it only through the workflow's authenticated
   deployment.

For SSH deployment, prepare the production server with Docker Compose, a
checkout of this repository at `DEPLOY_PATH`, and a production `.env` file
containing a unique `APP_KEY` and strong `DB_PASSWORD`. Add these repository
variables under **Settings > Secrets and variables > Actions > Variables**:

| Variable | Value |
| --- | --- |
| `DEPLOY_HOST` | SSH host name or IP address |
| `DEPLOY_USER` | SSH user allowed to run Docker Compose |
| `DEPLOY_PORT` | SSH port, usually `22` |
| `DEPLOY_PATH` | Absolute path to the checkout and production `.env` |
| `DEPLOY_KNOWN_HOSTS` | Verified SSH host-key line for the server |

Add `DEPLOY_SSH_KEY` under **Actions > Secrets** as the private key for that
SSH user. Verify the server host key out-of-band before saving it as
`DEPLOY_KNOWN_HOSTS`. Then run **Actions > Laravel CI/CD > Run workflow** on
`main` and enable `deploy_production`. The workflow builds and publishes the
image first, then logs into GHCR over SSH, updates the `app` service, and runs
Laravel migrations. Regular pushes and pull requests never deploy to production.

### Run without Docker

1. Clone the repo:

```bash
git clone git@github.com:planetscale/laravel-crud-mysql.git
```

1. Install the dependencies (`npm install` for Tailwind):

```bash
cd laravel-crud-mysql
composer install
npm install
```

1. Create your PlanetScale database. For complete instructions, see [this section of the tutorial](https://planetscale.com/blog/build-laravel-crud-app#planetscale-setup).

2. Copy the `.env` file:

```bash
mv .env.example .env
```

1. Update the `.env` database section with your PlanetScale credentials. It should look similar to this:

```
DB_CONNECTION=mysql
DB_HOST=xxxxxxxxxx.us-east-3.psdb.cloud
DB_PORT=3306
DB_DATABASE=laravel-mood-tracker
DB_USERNAME=xxxxxxxxxxx
DB_PASSWORD=<your-planetscale-password>
MYSQL_ATTR_SSL_CA=/etc/ssl/cert.pem
```

**Note**: The value for `MYSQL_ATTR_SSL_CA` will depend on your system. You can find more information on our [Secure Connections page](https://planetscale.com/docs/concepts/secure-connections).

1. Run the migrations:

```bash
php artisan migrate
```

1. Seed the database:

```bash
php artisan db:seed
```

1. Start the build process (for Tailwind):

```bash
npm run watch
```

1. Start the server:

```bash
php artisan serve
```

 1. View the application at [http://localhost:8080](http://localhost:8080)
