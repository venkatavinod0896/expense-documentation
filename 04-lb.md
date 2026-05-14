# Load Balancer

The load balancer sits in front of the frontend EC2 and distributes incoming traffic. It is a plain EC2 instance running Nginx — it does **not** serve any application files and does **not** communicate with the backend directly.

---

## Install Nginx

```bash
dnf install nginx -y
```

---

## Deploy Load Balancer Config

Download the load balancer package:

Remove the default Nginx config and replace it with the load balancer config:

```bash
rm -f /etc/nginx/nginx.conf
cd /etc/nginx
```

---

## Configure

Set the frontend private IP in the config:

```bash
vim /etc/nginx/nginx.conf
```

get the content of lb.conf and place it in the file

> **Use the private IP of the frontend EC2, not the public IP.**

Test the configuration:

```bash
nginx -t
```

---

## Start the Service

```bash
systemctl enable nginx
systemctl start nginx
```

---

## Verification

Open the browser at `http://<LB-PUBLIC-IP>` and confirm the Expense Tracker UI loads.

Check Nginx is running:

```bash
systemctl status nginx
```

Check access logs:

```bash
tail -f /var/log/nginx/access.log
```

---

## HTTP Status Code Demos

| Code | How to trigger |
|------|----------------|
| 502  | Stop nginx on the frontend EC2 (`systemctl stop nginx`), then reload the page |
| 503  | Uncomment `return 503` in `/etc/nginx/nginx.conf`, run `systemctl reload nginx` |
| 504  | Open `http://<LB-PUBLIC-IP>/api/slow` — backend takes 40 s, LB times out at 10 s |

Restore after each demo:

```bash
# 502 restore — on the frontend EC2
systemctl start nginx

# 503 restore — on the LB EC2
# comment out the return 503 line, then:
systemctl reload nginx
```

---

## Security Group

Ensure the LB EC2 security group allows **inbound TCP on port 80** from the internet (0.0.0.0/0).  
The frontend EC2 security group must allow **inbound TCP on port 80** from the LB EC2's security group only — not from the internet.
