# Backend

The backend service is responsible for handling API requests and persisting data to the database. It is written in Node.js.

> **Check with the developer for the exact version required. This setup requires Node.js >= 20.**

---

## Install Node.js

By default, Node.js 16 is available on the system. Enable and install version 20:

```bash
dnf module disable nodejs -y
dnf module enable nodejs:20 -y
dnf install nodejs -y
```

Verify:

```bash
node -v
```

---

## Create Application User

Add a system user to run the application (no login shell, no home directory access):

```bash
useradd --system --home /app --shell /sbin/nologin --comment "expense system user" expense
```

---

## Set Up Application Directory

```bash
mkdir /app
```

---

## Download and Extract Application

```bash
curl -o /tmp/backend.tar.gz https://raw.githubusercontent.com/daws-90s/expense-documentation/refs/heads/main/artifacts/expense-backend-v3.tar.gz
```

Extract files directly into `/app` (no subfolder):

```bash
cd /app
tar -xzf /tmp/backend.tar.gz --strip-components=1
```

---

## Install Dependencies

```bash
cd /app
npm install
```

---

## Configure SystemD Service

Create the service file:

```bash
cat > /etc/systemd/system/backend.service << 'EOF'
[Unit]
Description=Expense Backend Service
After=network.target

[Service]
User=expense
Environment=DB_HOST=<MYSQL-SERVER-IPADDRESS>
Environment=DB_USER=expense
Environment=DB_PWD=ExpenseApp@1
Environment=DB_DATABASE=transactions
ExecStart=/bin/node /app/index.js
SyslogIdentifier=backend
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

> **Replace `<MYSQL-SERVER-IPADDRESS>` with the private IP of the MySQL EC2 instance.**

---

## Load Database Schema

```bash
dnf install mysql -y
mysql -h <MYSQL-SERVER-IPADDRESS> -u root -pExpenseApp@1 < /app/schema/backend.sql
```

---

## Start the Service

```bash
systemctl daemon-reload
systemctl enable backend
systemctl start backend
```

---

## Verification

Check service status:

```bash
systemctl status backend
```

Check application logs:

```bash
journalctl -u backend -f
```

Test the health endpoint (from the same server):

```bash
curl http://localhost:8080/health
```

---

## Security Group

Ensure the backend EC2 security group allows **inbound TCP on port 8080** from the frontend EC2's security group only (not from the internet).
