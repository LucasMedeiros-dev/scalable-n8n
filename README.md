# 🚀 Scaling n8n with Docker Compose

This project is part of a **lesson on how to scale n8n** in production using Postgres, Redis, PgBouncer, and multiple workers.  

The goal is to separate the responsibilities of the main n8n editor, the worker process, and the webhook worker while keeping the database connections efficient with PgBouncer.

---

## 📂 Project Structure

```
.
├── .env-db          # Database and PgBouncer environment variables
├── .env-main        # Environment for the main n8n instance
├── .env-worker      # Environment for the worker/webhook-worker
├── compose.yaml     # Docker Compose configuration
├── init-data.sh     # Script to create a non-root DB user
├── pgbouncer.ini    # PgBouncer configuration file
├── userlist.txt     # PgBouncer users and passwords
└── README.md        # This file
```

---

## ⚙️ Services

- **Postgres 17** – The main database for n8n.  
- **PgBouncer** – Connection pooler for Postgres (improves performance and avoids connection exhaustion).  
- **Redis 8.2** – Manages the n8n queue system.  
- **n8n (Editor)** – The UI/API instance for building workflows.  
- **n8n Worker** – Executes workflows from the queue.  
- **n8n Webhook Worker** – Dedicated worker for receiving webhooks (low latency).  

---

## 🔑 Important Notes

1. **Ingress / Reverse Proxy**  
   This setup **does not include Traefik, HAProxy, or Nginx**.  
   You must configure your ingress/reverse proxy separately depending on your infrastructure.

2. **Two Domains Required**  
   - One domain for the **n8n Editor UI** (e.g., `https://n8n.yourdomain.com`)  
   - Another domain for **webhooks** (e.g., `https://n8n-webhook.yourdomain.com`)  

   This separation ensures stability and avoids conflicts between UI/API traffic and webhook requests.

3. **Webhook URL**  
   In your `.env-main`, make sure to set:  
   ```dotenv
   WEBHOOK_URL=https://n8n-webhook.yourdomain.com
   ```

   This tells n8n where to register its incoming webhook endpoints.

4. **PgBouncer Userlist**  
   The `userlist.txt` file defines which users can connect through PgBouncer.  
   Example:
   ```txt
   "lucas" "asupercoolpassword2025"
   ```
   Alternatively, you can store passwords in **md5 format**:
   ```
   "lucas" "md5<md5hash>"
   ```
   > The hash must be generated as `md5(password + username)`.

---

## ▶️ Usage

1. Clone the repo and navigate to the project directory.  
2. Adjust the `.env-*` files according to your environment.  
3. Start the stack:  
   ```bash
   docker compose up -d
   ```
4. Access:  
   - **n8n Editor:** http://localhost:5678 (or your ingress domain)  
   - **Webhook Worker:** http://localhost:5679 (or your ingress domain for webhooks)  

---

## 📌 Scaling Workers

To scale execution power, you can run multiple workers:  
```bash
docker compose up -d --scale worker=3
```

This will start 3 worker containers, consuming jobs from the Redis queue concurrently.

---

## ✅ Summary

This lesson shows how to:  
- Run n8n in **queue mode** with separate worker processes.  
- Use **PgBouncer** to optimize Postgres connections.  
- Manage webhooks with a **dedicated worker**.  
- Properly configure **two separate domains** for editor and webhooks.  
- Prepare for ingress with **Traefik, HAProxy, or Nginx** depending on your stack.
