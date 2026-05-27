# Ubuntu – RabbitMQ installation with Docker Compose

## System
- OS: Ubuntu Server
- Container runtime: Docker
- Orchestration: Docker Compose
- Service: RabbitMQ 4.x with Management UI

## Objective

Deploy RabbitMQ on Ubuntu using Docker Compose with:

- persistent storage
- management UI enabled
- admin credentials configured
- ports exposed for application access and browser-based monitoring

---

# Prerequisites

- Ubuntu server with SSH access
- Docker installed and running
- internet access to pull container images
- sudo privileges

---

# Step 1 – Update system packages

```bash
sudo apt update && sudo apt upgrade -y
```

Why:

Ensures package lists and installed packages are up to date before deployment.

---

# Step 2 – Create project directory

```bash
mkdir ~/rabbitmq
cd ~/rabbitmq
```

Why:

Keeps Docker Compose files and related configuration isolated in a dedicated project folder.

---

# Step 3 – Configure Docker DNS (if Docker Hub access fails)

In this setup Docker had DNS resolution issues while pulling images.

Create or edit Docker daemon config:

```bash
sudo nano /etc/docker/daemon.json
```

Add:

```json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

Why:

Forces Docker to use Google DNS servers if default DNS resolution fails.

---

# Step 4 – Create Docker Compose file

Create compose file:

```bash
nano rabbitmq-c.yml
```

Add:

```yaml
services:
  rabbitmq:
    image: rabbitmq:4-management
    container_name: rabbitmq
    restart: unless-stopped
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: yourpassword
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

volumes:
  rabbitmq_data:
```

Why:

This deploys RabbitMQ with:

- AMQP service enabled on port `5672`
- Management UI enabled on port `15672`
- persistent Docker volume storage

---

# Step 5 – Upgrade Docker Compose

The existing `docker-compose` version was outdated and caused compatibility issues.

Check current version:

```bash
docker-compose --version
```

Remove old version:

```bash
sudo apt remove docker-compose -y
sudo rm -f /usr/bin/docker-compose
```

Install latest release:

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Make executable:

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

Create symlink:

```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Verify:

```bash
docker-compose --version
```

Why:

Upgrading resolved compatibility issues with the compose configuration.

---

# Step 6 – Start RabbitMQ

Start container:

```bash
docker-compose -f rabbitmq-c.yml up -d
```

Verify:

```bash
docker ps
```

Expected:

Container `rabbitmq` should show:

- status `Up`
- port `5672`
- port `15672`

---

# Step 7 – Access RabbitMQ Management UI

Get server IP:

```bash
hostname -I
```

Open browser:

```text
http://<server-ip>:15672
```

Login with:

```text
Username: admin
Password: yourpassword
```

---

# Port reference

| Port | Purpose |
|---|---:|
| 5672 | AMQP client connections |
| 15672 | RabbitMQ Management UI |
| 25672 | Erlang cluster communication |

---

# Useful commands

## Stop RabbitMQ

```bash
docker-compose -f rabbitmq-c.yml down
```

---

## Restart RabbitMQ

```bash
docker-compose -f rabbitmq-c.yml restart
```

---

## View logs

```bash
docker-compose -f rabbitmq-c.yml logs -f
```

---

## List queues

```bash
docker exec rabbitmq rabbitmqctl list_queues
```

---

## Check RabbitMQ status

```bash
docker exec rabbitmq rabbitmqctl status
```

---

## Remove container

```bash
docker rm -f rabbitmq
```

---

# What I learned

- `rabbitmq:4-management` includes the Management UI out of the box.
- Docker volumes persist RabbitMQ data across container restarts.
- Docker DNS configuration may need adjustment if image pulls fail.
- Keeping RabbitMQ in a dedicated compose project directory makes maintenance easier.
- Upgrading from older Docker Compose versions can resolve deployment compatibility issues.
- `docker-compose logs -f` is useful for troubleshooting startup problems.
