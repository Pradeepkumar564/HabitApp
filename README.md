# Project-05: Java Habit Tracker – CI/CD & GitOps Deployment on Amazon EKS

## Project Overview

This project demonstrates an end-to-end DevOps workflow for a Java Spring Boot Habit Tracker application. It automates application building, code quality analysis, Docker image creation, container registry publishing, Kubernetes deployment, GitOps synchronization, and monitoring.

The application is deployed on Amazon EKS using Helm and Argo CD.

## Architecture

```text
Developer
   |
   v
GitHub Repository
   |
   v
Jenkins CI Pipeline
   |
   +---- Maven Build & Test
   |
   +---- SonarQube Code Analysis
   |
   +---- Docker Image Build
   |
   +---- Push to Docker Hub
                 |
                 v
          Docker Hub Registry
                 |
                 v
          Helm Chart in GitHub
                 |
                 v
              Argo CD
                 |
                 v
          Amazon EKS Cluster
                 |
                 v
       Habit Tracker Application
                 |
                 v
          Spring Boot Actuator
                 |
                 v
             Prometheus
                 |
                 v
              Grafana
```

## Technology Stack

| Category | Technologies |
|---|---|
| Programming Language | Java 21 |
| Framework | Spring Boot 3.3.4 |
| Build Tool | Maven |
| Source Control | Git, GitHub |
| CI/CD | Jenkins |
| Code Quality | SonarQube |
| Containerization | Docker |
| Container Registry | Docker Hub |
| Container Orchestration | Kubernetes, Amazon EKS |
| Package Management | Helm |
| GitOps | Argo CD |
| Monitoring | Prometheus, Grafana |
| Metrics | Spring Boot Actuator, Micrometer |

## Application Features

- Create, update, and delete habits.
- View all habits and individual habit details.
- Activate and deactivate habits.
- Mark habits as completed.
- Track current and longest streaks.
- View habit completion statistics.
- Web dashboard and REST API.

### REST API

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/habits` | List all habits |
| GET | `/api/habits/{id}` | Get a specific habit |
| POST | `/api/habits` | Create a habit |
| PUT | `/api/habits/{id}` | Update a habit |
| DELETE | `/api/habits/{id}` | Delete a habit |
| PATCH | `/api/habits/{id}/active` | Activate/deactivate a habit |
| POST | `/api/habits/{id}/complete` | Mark a habit complete |
| DELETE | `/api/habits/{id}/complete` | Remove completion |
| GET | `/api/habits/{id}/stats` | View habit statistics |

**Storage note:** The application uses an in-memory `ConcurrentHashMap`. Data is not persistent and is lost when the application restarts. Each replica has its own in-memory storage.

## CI/CD Pipeline – Jenkins

The Jenkins pipeline automates the following stages:

1. **Checkout:** Retrieves source code from GitHub.
2. **Build:** Compiles and packages the application using Maven.
3. **Code Analysis:** Performs SonarQube quality analysis.
4. **Docker Build:** Creates the application Docker image.
5. **Docker Push:** Authenticates with Docker Hub and publishes the image.

### Docker Hub

Image repository: `pradeepkumar564/habit-tracker`

The Jenkins pipeline successfully built and pushed the `latest` image.

## SonarQube

SonarQube is integrated into the Jenkins pipeline to analyze source code quality.

It helps identify:
- Bugs
- Code smells
- Security vulnerabilities
- Code quality issues

## Docker

The application is containerized using a multi-stage Dockerfile.

Container configuration includes:
- Java 21 runtime
- Multi-stage image build
- Non-root user execution
- Reduced runtime image
- Application running on port `8080`

## Kubernetes Deployment – Amazon EKS

The application is deployed to Amazon EKS using a Helm chart.

### Deployment Configuration

| Configuration | Value |
|---|---|
| Namespace | `habit-tracker` |
| Deployment | `habit-tracker` |
| Replicas | 2 |
| Service Type | ClusterIP |
| Service Port | 80 |
| Container Port | 8080 |
| Image Repository | `pradeepkumar564/habit-tracker` |
| Helm Chart | `habit-tracker` |

### Kubernetes Features

- Two application replicas
- Liveness and readiness probes
- CPU and memory requests and limits
- Non-root container execution
- Kubernetes Secret integration
- ClusterIP service

### Kubernetes Secret

A Kubernetes Secret named `habit-tracker-secret` is configured with the key `APP_ENV`.

It is injected into the application container as an environment variable.

## Helm

Helm is used to package and manage the Kubernetes deployment.

The Helm chart includes:
- Deployment
- Service
- ServiceAccount
- Kubernetes Secret configuration
- ServiceMonitor
- Optional autoscaling and ingress templates

The chart was validated using `helm lint`.

### Helm Rollback

A Helm upgrade was performed to demonstrate a change in replica count. The deployment was subsequently rolled back to revision 1.

The rollback completed successfully and restored the two-replica configuration.

## GitOps – Argo CD

Argo CD manages the Habit Tracker application deployment using the GitHub repository as its source of truth.

Configuration:
- Repository: `https://github.com/Pradeepkumar564/HabitApp.git`
- Branch: `main`
- Chart path: `habit-tracker`
- Application namespace: `argocd`
- Deployment namespace: `habit-tracker`

Argo CD automated synchronization and self-healing are configured.

## Monitoring – Prometheus & Grafana

Spring Boot Actuator and Micrometer Prometheus Registry expose application metrics.

Metrics endpoint:

`/actuator/prometheus`

A Kubernetes ServiceMonitor is configured with:
- Namespace: `habit-tracker`
- Metrics path: `/actuator/prometheus`
- Scrape interval: 30 seconds
- Service port: `http`

### Monitoring Verification

Prometheus successfully discovered and scraped both Habit Tracker application replicas.

The following checks were verified:
- Both application targets reported `UP`.
- No scrape errors were reported.
- Prometheus returned `up` metric values of `1` for both replicas.

Grafana is available through the existing monitoring stack for metrics visualization.

## Repository Structure

```text
project-05/
│
├── src/
│   └── main/
│       ├── java/
│       └── resources/
│
├── habit-tracker/
│   ├── templates/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── serviceaccount.yaml
│   │   ├── servicemonitor.yaml
│   │   ├── hpa.yaml
│   │   └── ingress.yaml
│   ├── Chart.yaml
│   └── values.yaml
│
├── argocd-application.yaml
├── Dockerfile
├── .dockerignore
├── .gitignore
├── pom.xml
├── sonar-project.properties
└── README.md
```

## Local Development

### Prerequisites

- Java 21+
- Maven
- Git
- Docker

### Clone the Repository

```bash
git clone https://github.com/Pradeepkumar564/HabitApp.git
cd HabitApp
```

### Build the Application

```bash
mvn clean package
```

### Run the Application

```bash
java -jar target/habit-tracker.jar
```

Open:

`http://localhost:8080`

### Build Docker Image

```bash
docker build -t pradeepkumar564/habit-tracker:latest .
```

### Run Docker Container

```bash
docker run -d -p 8080:8080 --name habit-tracker pradeepkumar564/habit-tracker:latest
```

## Useful Kubernetes Commands

Check application pods:

```bash
kubectl get pods -n habit-tracker
```

Check deployment:

```bash
kubectl get deployment -n habit-tracker
```

Check services:

```bash
kubectl get services -n habit-tracker
```

Check Argo CD application:

```bash
kubectl get applications -n argocd
```

Check ServiceMonitor:

```bash
kubectl get servicemonitor -n habit-tracker
```

Validate Helm chart:

```bash
helm lint ./habit-tracker
```

## Project Outcome

The project demonstrates an automated DevOps workflow using Jenkins, Maven, SonarQube, Docker, Docker Hub, Helm, Amazon EKS, and Argo CD.

The application was successfully built and published through Jenkins, deployed on Amazon EKS, and configured for GitOps management. Prometheus successfully collected metrics from both application replicas.

This project provided practical experience in CI/CD automation, containerization, Kubernetes orchestration, Helm-based deployments, GitOps practices, and application monitoring.

## Author

**Pradeep Kumar R**

GitHub: https://github.com/Pradeepkumar564
