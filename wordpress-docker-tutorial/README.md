# wordpress-docker-tutorial
Tutorial wordpress in docker conatiner example

# Start docker in your machine
# use something like docker desktop or rancher

# Navigate to project directory
cd wordpress-docker

# Start containers
docker-compose up -d

# Check containers
docker ps

# Try Access your wordpress working at
http://localhost:8080


Additional Configuration Notes
1. Data Persistence
Volumes (wordpress_data and db_data) ensure that your WordPress files and MySQL data are persisted outside the container lifecycle.

2. Security Notes
Never expose MYSQL_ROOT_PASSWORD or any credentials in a public repo.

Use .env files and Git ignore them (.gitignore).

Consider adding HTTPS with reverse proxies like Traefik or Nginx + Certbot in production.

3. Scaling & Production
For a production setup:

Use Docker Swarm or Kubernetes for orchestration.

Replace the database with Amazon RDS, Azure Database for MySQL, or a managed database service.

Use secrets management (e.g., Docker secrets, Vault) instead of env files for sensitive data.

Add caching with Redis or Varnish.

Serve media through S3 and integrate CDN (e.g., Cloudflare).

# View logs
docker-compose logs -f wordpress

# Enter container shell
docker exec -it wordpress bash

# Stop containers
docker-compose down

# Stop and remove volumes (careful, this deletes data!)
docker-compose down -v

# 📘 Tutorial: Using Plugins in Dockerized WordPress

This tutorial shows you how to install and manage plugins in a WordPress instance running inside Docker. It covers:

- ✅ Accessing the WordPress admin panel
- 🔌 Installing plugins (manually and via the admin UI)
- 📦 Installing plugins using WP-CLI
- ⚙️ Activating and managing plugins
- 🧰 Best practices for plugin management in containerized environments

---

## ✅ Prerequisites

Make sure you have:

- Docker and Docker Compose installed
- The `docker-compose.yml` and `.env` files from the previous WordPress + MySQL setup
- Your WordPress site running at: `http://localhost:8080`

---

## Step 1: 🖥️ Access WordPress Admin Panel

1. Open browser and go to: [http://localhost:8080](http://localhost:8080)
2. Complete the WordPress installation wizard (if not already done)
3. Login with the admin user you created

---

## Step 2: 🔌 Install Plugins via WordPress Admin UI

1. In the WordPress dashboard, go to **Plugins > Add New**
2. Use the search bar to find a plugin (e.g., **Yoast SEO**, **WP Super Cache**, **Contact Form 7**)
3. Click **Install Now**, then **Activate**

> 📌 This is the easiest method, and it persists across container restarts because the `wordpress_data` volume stores all plugin files.

---

## Step 3: ⚙️ Install Plugins via WP-CLI (Inside Docker Container)

### Option A: Install WP-CLI in WordPress container

#### Option A1: Use a Custom Dockerfile

```Dockerfile
# Dockerfile
FROM wordpress:latest

RUN apt-get update && \
    apt-get install -y less mariadb-client curl unzip && \
    curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar && \
    chmod +x wp-cli.phar && \
    mv wp-cli.phar /usr/local/bin/wp

Update your docker-compose.yml:

  wordpress:
    build: .

# Use WP-CLI to Install and Activate Plugins
# Navigate to WordPress root directory
cd /var/www/html

# Install plugin (example: Yoast SEO)
wp plugin install wordpress-seo --activate

# Install multiple plugins
wp plugin install contact-form-7 wp-super-cache --activate

# List plugins
wp plugin list

# Deactivate plugin
wp plugin deactivate wp-super-cache

# Delete plugin
wp plugin delete wp-super-cache

# Step 4: 📁 Install Plugins Manually (File-Based)
Download the plugin .zip file from wordpress.org/plugins

Extract the .zip file

Copy the plugin directory into the container’s /var/www/html/wp-content/plugins/

# Example:
# On your host machine
unzip wordpress-seo.zip

# Copy to container
docker cp wordpress-seo wordpress:/var/www/html/wp-content/plugins/

# Enter container and activate
docker exec -it wordpress bash
cd /var/www/html
wp plugin activate wordpress-seo


# Optional: Pre-install Plugins via Docker Image
FROM wordpress:latest

RUN apt-get update && apt-get install -y curl unzip \
 && curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
 && chmod +x wp-cli.phar && mv wp-cli.phar /usr/local/bin/wp

WORKDIR /var/www/html

RUN wp plugin install wordpress-seo --activate \
 && wp plugin install contact-form-7 --activate


# Best Practices for Plugin Management in Docker
| Practice                       | Why It Matters                                  |
| ------------------------------ | ----------------------------------------------- |
| Use WP-CLI in provisioning     | Automates deployments, improves reproducibility |
| Use version-pinned plugins     | Prevent unexpected updates breaking builds      |
| Avoid UI installs in prod      | Doesn’t scale or replicate in CI                |
| Persist volumes for data       | Avoid losing plugins on container restart       |
| Monitor plugin vulnerabilities | Use WPScan, Patchstack, etc.                    |


