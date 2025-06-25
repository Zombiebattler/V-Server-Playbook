# Ansible: Nginx Proxy Manager + Apache Installer

This Ansible playbook installs and configures:

- **Nginx Proxy Manager (NPM)** as a reverse proxy
- **Apache2** as a backend web server

Use this setup to serve your website behind a user-friendly proxy manager.

---

## How to Use

### 1. Add Your Website

Place your website inside:

`roles/website/files/main/`

Make sure your site includes an `index.html` or an entry file.

---

### 2. Set Your Database Password

To secure your MariaDB instance used by Nginx Proxy Manager, open:

`roles/docker/files/docker-compose.yaml`

Locate the following section and change the database password:

```yaml
version: "3.9"
services:
  proxy-manager:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '443:443'
      - '81:81'
    environment:
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "root"
      DB_MYSQL_PASSWORD: "Test123"              # 👈 Change this line    
      DB_MYSQL_NAME: "proxy-db"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db

  db:
    image: 'jc21/mariadb-aria:latest'
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'Test123'            # 👈 Change this line    
      MYSQL_DATABASE: 'proxy-db'
      MYSQL_USER: 'root'
      MYSQL_PASSWORD: 'Test123'                 # 👈 Change this line    
      MARIADB_AUTO_UPGRADE: '1'
    volumes:
      - ./mysql:/var/lib/mysql

  apache:
    image: httpd:latest
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      - /var/www/html:/usr/local/apache2/htdocs/

```

---

### 3. Configure Your Domain

After the installation is complete, open your browser and visit:

`http://<server-ip>:81`

This will bring you to the **Nginx Proxy Manager** dashboard. From here, you can:

- Add a new **Proxy Host**
- Enter your custom domain (e.g. `example.com`)
- Optionally request a **free SSL certificate** via Let's Encrypt

> ⚠️ **Important:** Do **not** request an SSL certificate for local domains like `dummy.local` — this will fail. Make sure to delete or avoid such test entries.