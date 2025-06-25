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

### 2. Configure Your Domain (Optional)

Open the file:

`roles/config/tasks/main.yml`

Go to **line 27** and set your desired domain:

```yaml
- name: Create proxy host
  ansible.builtin.uri:
    url: http://localhost:81/api/nginx/proxy-hosts
    method: POST
    body_format: json
    headers:
      Content-Type: application/json
      Authorization: "Bearer {{ npm_auth.json.token }}"
    body:
      domain_names: ["dummy.local"]            # 👈 Change this line       
      forward_scheme: "http"
      forward_host: "apache"
      forward_port: 80
      access_list_id: null
      certificate_id: null
      ssl_forced: false
      caching_enabled: false
      block_exploits: true
      advanced_config: ""
      locations: []
      allow_websocket_upgrade: false
      http2_support: false
      hsts_enabled: false
      hsts_subdomains: false
      locations: []
    status_code: 200,201

```

---

### 3. Set Your Database Password

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