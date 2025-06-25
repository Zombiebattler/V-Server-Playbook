# Ansible: Nginx Proxy Manager + Apache Installer

This Ansible playbook installs and configures:

- **Nginx Proxy Manager (NPM)** as a reverse proxy
- **Apache2** as a backend web server

Use this setup to serve your website behind a user-friendly proxy manager.


## How to Use

### 1. Add Your Website

Place your Website inside:

`roles/website/files/main/`

Make sure your site includes an `index.html` or an entry file.

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
