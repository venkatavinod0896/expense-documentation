# Frontend

The frontend serves static web content (HTML, CSS, JS) via Nginx. It also acts as a reverse proxy, forwarding `/api/` requests to the backend EC2 instance.

---

## Install Nginx

```bash
dnf install nginx -y
```

Enable and start:

```bash
systemctl enable nginx
systemctl start nginx
```

Verify the default Nginx page is accessible in the browser at `http://<FRONTEND-PUBLIC-IP>`.

---

## Deploy Frontend Content

Remove the default Nginx content:

```bash
rm -rf /usr/share/nginx/html/*
```

Download the frontend package:

```bash
curl -o /tmp/frontend.tar.gz https://raw.githubusercontent.com/daws-90s/expense-documentation/refs/heads/main/artifacts/expense-frontend-v3.tar.gz
```

Extract files directly into the Nginx web root (no subfolder):

```bash
cd /usr/share/nginx/html
tar -xzf /tmp/frontend.tar.gz --strip-components=1
```

---

## Configure Nginx Reverse Proxy

Create a reverse proxy config that forwards `/api/` requests to the backend:

```bash
vim /etc/nginx/default.d/expense.conf
```

```bash
proxy_http_version 1.1;
proxy_set_header Host              $host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

location /api/ {
    proxy_pass http://<BACKEND-PRIVATE-IP>:8080/;
}

location /health {
    stub_status on;
    access_log off;
}
```

> **Replace `<BACKEND-PRIVATE-IP>` with the private IP of the backend EC2 instance.**

Test the configuration:

```bash
nginx -t
```

Reload Nginx to apply changes:

```bash
systemctl restart nginx
```

---

## Verification

Open the browser at `http://<FRONTEND-PUBLIC-IP>` and confirm the Expense Tracker UI loads.

Test the reverse proxy is forwarding correctly:

```bash
curl http://localhost/api/health
```

You should get `{"status":"ok"}` from the backend.

---

## Security Group

Ensure the frontend EC2 security group allows **inbound TCP on port 80** from the internet (0.0.0.0/0).  
Port 8080 on the backend must **not** be open to the internet — only to the frontend EC2's security group.
