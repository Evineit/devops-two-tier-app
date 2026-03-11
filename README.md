# devops-two-tier-app

## Project Overview

A two-tier web application built with **Flask** and **MySQL**, containerized with Docker and deployed on AWS EC2. A CI/CD pipeline using Jenkins automatically builds and deploys the app on every push to the `main` branch.

**Stack:**
- **Frontend/Backend:** Flask (Python 3.9)
- **Database:** MySQL
- **Containerization:** Docker & Docker Compose
- **CI/CD:** Jenkins
- **Infrastructure:** AWS EC2

---

## Architecture

```
[ Browser ] --> [ Flask App :5000 ] --> [ MySQL :3306 ]
```

Both services run as Docker containers on the same host, connected via a Docker bridge network (`two-tier`).

---

## Prerequisites

- AWS EC2 instance — **t2.small** (2 GB RAM minimum)
- Ubuntu/Debian OS on the instance
- Open inbound ports: `22` (SSH), `5000` (app), `8080` (Jenkins), `3306` (MySQL, optional)

---

## Deployment Guide

### 1. Launch an EC2 Instance

Create a `t2.small` EC2 instance running Ubuntu. Attach a security group that allows the ports listed above.

### 2. Install Docker

```bash
sudo apt-get update
sudo apt-get install -y docker.io docker-compose-plugin
sudo usermod -aG docker $USER
```

### 3. Install Jenkins

Follow the official guide for Debian/Ubuntu:
https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

### 4. Configure Jenkins Pipeline

In Jenkins, create a new **Pipeline** job and point it at this repository. Jenkins will clone the code, build the Docker image, and deploy the app using Docker Compose — all on the same EC2 server.

The app will be available at `http://<EC2-PUBLIC-IP>:5000` after the first pipeline run.

### 5. Configure GitHub Webhook

In your GitHub repository settings, add a webhook pointing to:

```
http://<EC2-PUBLIC-IP>:8080/github-webhook/
```

Set the content type to `application/json` and trigger on **push** events. This allows Jenkins to automatically deploy on every push to `main`.

---

## CI/CD Pipeline

The `Jenkinsfile` defines a three-stage pipeline:

| Stage | Description |
|---|---|
| Clone Code | Pulls the latest code from the `main` branch |
| Build Docker Image | Builds the `flask-app:latest` Docker image |
| Deploy | Tears down existing containers and redeploys via `docker compose up` |

---

## Environment Variables

The Flask container reads the following environment variables (set in `docker-compose.yml`):

| Variable | Default | Description |
|---|---|---|
| `MYSQL_HOST` | `mysql` | MySQL container hostname |
| `MYSQL_USER` | `root` | MySQL username |
| `MYSQL_PASSWORD` | `root` | MySQL password |
| `MYSQL_DB` | `devops` | Database name |

> **Security note:** Update the default MySQL credentials before deploying to production.

---

## Project Structure

```
.
├── app.py                 # Flask application
├── requirements.txt       # Python dependencies
├── Dockerfile             # Flask app container image
├── docker-compose.yml     # Multi-container orchestration
├── Jenkinsfile            # CI/CD pipeline definition
├── message.sql            # Database seed/schema
└── templates/
    └── index.html         # Frontend template
```