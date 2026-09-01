# Node Express RealWorld Example App

A production-oriented backend implementation of the **RealWorld API specification**, built with **Node.js, Express, TypeScript, Prisma, and PostgreSQL**.

The application provides REST APIs for authentication, users, profiles, articles, comments, tags, favorites, and follows. It is designed to work with the RealWorld React/Redux frontend.

---

## 1. Application Overview

This project is the backend API for a RealWorld-style blogging/social application similar to Medium.

### Core functionality

* User registration and authentication
* JWT-based authentication
* User profile management
* Create, read, update, and delete articles
* Article favorites
* Comments on articles
* Follow/unfollow users
* Article feeds
* Tags
* Pagination
* PostgreSQL persistence
* Prisma ORM
* REST API architecture

### Technology Stack

| Component          | Technology     |
| ------------------ | -------------- |
| Runtime            | Node.js        |
| Language           | TypeScript     |
| API Framework      | Express.js     |
| ORM                | Prisma         |
| Database           | PostgreSQL     |
| Authentication     | JWT            |
| Password Hashing   | bcryptjs       |
| Build Tool         | Nx + esbuild   |
| Testing            | Jest           |
| Containerization   | Docker         |
| Container Registry | Amazon ECR     |
| Container Platform | Amazon ECS     |
| CI/CD              | GitHub Actions |
| Cloud              | AWS            |
| AWS Region         | `us-east-1`    |

---

# 2. Architecture and Deployment Approach

The application follows a containerized AWS deployment model.

```text
                    ┌──────────────────────┐
                    │      GitHub Repo     │
                    │  Source Code + CI/CD │
                    └──────────┬───────────┘
                               │
                               │ git push
                               ▼
                    ┌──────────────────────┐
                    │    GitHub Actions    │
                    │                      │
                    │  1. Install          │
                    │  2. Test             │
                    │  3. Build            │
                    │  4. Docker Build     │
                    │  5. Push Image       │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Amazon ECR        │
                    │ realworld-backend    │
                    │ Docker Image         │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     Amazon ECS       │
                    │                      │
                    │  ECS Cluster         │
                    │  ECS Service         │
                    │  ECS Task            │
                    │  Backend Container   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     PostgreSQL       │
                    │      Database        │
                    └──────────────────────┘
```

### Deployment flow

1. Developer pushes code to GitHub.
2. GitHub Actions starts automatically.
3. Dependencies are installed using `npm ci`.
4. Application tests are executed.
5. Application is built.
6. Docker image is created.
7. Docker image is pushed to Amazon ECR.
8. ECS task definition is updated with the new image.
9. ECS service deploys the new task.
10. The application becomes available through the configured ECS networking/load-balancing setup.

---

# 3. AWS Deployment

The deployment is configured for:

```text
AWS Region: us-east-1
ECR Repository: realworld-backend
ECS Cluster: realworld-cluster
ECS Service: realworld-backend
ECS Task Definition: realworld-backend
Container Name: backend
```

These values should be maintained consistently between the GitHub Actions workflow and AWS resources.

---

# 4. Deployment Cost Considerations

The solution is designed to keep development and demonstration costs low.

### Main AWS cost components

* Amazon ECS
* Amazon ECR
* PostgreSQL/database infrastructure
* Networking
* CloudWatch logs
* Load balancer, if configured
* Data transfer

For development/testing, small instance types such as `t3.micro` can reduce compute costs. However, `t3.micro` has limited memory and is not suitable for memory-intensive build or dependency-management operations.

During development, `npm audit fix --force` caused an out-of-memory condition on the `t3.micro`. The Linux OOM killer terminated the npm process.

For production workloads, compute and database resources should be sized according to CPU, memory, traffic, and availability requirements rather than selecting the smallest instance purely for cost.

---

# 5. Prerequisites

Install/configure the following:

### Local development

* Git
* Node.js
* npm
* Docker
* PostgreSQL
* AWS CLI
* AWS account
* GitHub account

### AWS

The following resources are required:

* ECR repository
* ECS cluster
* ECS service
* ECS task definition
* IAM role/user for deployment
* PostgreSQL database
* Required VPC/networking resources
* CloudWatch logging

### GitHub

Configure the required AWS credentials and application secrets using **GitHub Actions Secrets** rather than committing them to the repository.

---

# 6. Environment Variables

Create a `.env` file for local development.

```env
DATABASE_URL="postgresql://realworld:password123@localhost:5432/realworld?schema=public"
JWT_SECRET="replace-with-a-strong-random-secret"
NODE_ENV=development
```

Do not commit `.env` to Git.

The repository's `.gitignore` excludes `.env`.

For production, configure environment variables through the deployment platform/secret-management mechanism instead of storing secrets in source control.

---

# 7. Generate a JWT Secret

A strong random JWT secret should be used instead of:

```text
my-super-secret-key
```

For example:

```bash
openssl rand -base64 32
```

Example output:

```text
generated-random-secret
```

Store the generated value securely as:

```text
JWT_SECRET
```

Do not commit the actual secret to GitHub.

---

# 8. Deployment Steps

## Step 1 — Clone the repository

```bash
git clone https://github.com/pramodprasannakumar/node-express-realworld-example-app.git

cd node-express-realworld-example-app
```

## Step 2 — Install dependencies

```bash
npm ci
```

For local development, if a lock file does not exist:

```bash
npm install
```

The committed `package-lock.json` should be used by CI/CD so that builds remain reproducible.

---

## Step 3 — Configure environment variables

Create `.env`:

```env
DATABASE_URL="postgresql://realworld:password123@localhost:5432/realworld?schema=public"
JWT_SECRET="your-secure-secret"
NODE_ENV=development
```

---

## Step 4 — Generate Prisma Client

```bash
npx prisma generate
```

---

## Step 5 — Apply database migrations

```bash
npx prisma migrate deploy
```

---

## Step 6 — Seed the database

```bash
npx prisma db seed
```

---

## Step 7 — Run tests

```bash
npm test
```

---

## Step 8 — Build the application

```bash
npm run build
```

---

## Step 9 — Build Docker image

```bash
docker build -t realworld-backend .
```

---

## Step 10 — Tag and push the image to Amazon ECR

Authenticate Docker with ECR:

```bash
aws ecr get-login-password --region us-east-1 | \
docker login --username AWS --password-stdin \
<ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

Tag the image:

```bash
docker tag realworld-backend:latest \
<ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/realworld-backend:latest
```

Push:

```bash
docker push \
<ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/realworld-backend:latest
```

---

## Step 11 — Deploy to ECS

The ECS service uses the Docker image stored in ECR.

A new image can be deployed by updating the ECS task definition and forcing a new deployment.

Example:

```bash
aws ecs update-service \
  --cluster realworld-cluster \
  --service realworld-backend \
  --force-new-deployment \
  --region us-east-1
```

---

# 9. CI/CD Workflow

GitHub Actions automates the deployment process.

```text
Developer
    │
    │ git push
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── Checkout source
    │
    ├── Setup Node.js
    │
    ├── npm ci
    │
    ├── Run tests
    │
    ├── Build application
    │
    ├── Build Docker image
    │
    ├── Authenticate with AWS
    │
    ├── Push image to ECR
    │
    ├── Update ECS task definition
    │
    └── Deploy ECS service
             │
             ▼
        AWS ECS
```

### Deployment configuration

The workflow uses the following deployment parameters:

```text
AWS_REGION=us-east-1
ECR_REPOSITORY=realworld-backend
ECS_CLUSTER=realworld-cluster
ECS_SERVICE=realworld-backend
ECS_TASK_DEFINITION=realworld-backend
CONTAINER_NAME=backend
```

### CI/CD benefits

* Automated builds
* Automated testing
* Consistent deployments
* Docker image versioning
* ECR image storage
* ECS deployment automation
* Reduced manual deployment errors

---

# 10. Versioning Strategy

The application uses Git for source-code version control.

### Branching

Recommended workflow:

```text
feature/* 
    │
    ▼
development
    │
    ▼
master
    │
    ▼
production
```

Changes should be committed with meaningful messages.

Example:

```bash
git add .
git commit -m "feat: update backend deployment configuration"
git push origin master
```

### Container image versioning

Avoid relying exclusively on the `latest` tag.

Recommended tags:

```text
realworld-backend:1.0.0
realworld-backend:1.0.1
realworld-backend:<git-sha>
```

Using a Git commit SHA makes it possible to identify exactly which source version is running in ECS.

---

# 11. Security Approach

Security is handled at multiple layers.

### Application security

* JWT authentication
* Password hashing using bcrypt
* Environment-based configuration
* Input validation
* CORS configuration
* Avoid exposing sensitive configuration

### Secrets

Sensitive values such as:

```text
DATABASE_URL
JWT_SECRET
AWS credentials
```

must not be committed to Git.

Use:

* GitHub Actions Secrets
* AWS Secrets Manager
* ECS environment/secret configuration

for production deployments.

### AWS security

Use IAM policies following the principle of least privilege.

The CI/CD deployment identity should only have the permissions required to:

* Authenticate with ECR
* Push images
* Read/update ECS deployment resources
* Register/update task definitions
* Deploy the ECS service

Avoid using unrestricted AWS administrator credentials for CI/CD.

---

# 12. Monitoring and Observability

The deployment should use AWS-native monitoring for the ECS environment.

### CloudWatch

Configure CloudWatch Logs for the backend container.

Monitor:

* Application errors
* Container startup failures
* API errors
* Authentication failures
* Database connection errors
* ECS task restarts

### ECS monitoring

Monitor:

* CPU utilization
* Memory utilization
* Running task count
* Desired task count
* Deployment failures
* Task health
* Service availability

### Application health

A health endpoint can be used to verify application availability.

Example:

```text
GET /api/health
```

A load balancer or monitoring system can use this endpoint for health checks where configured.

---

# 13. Challenges Encountered and Resolutions

## Challenge 1 — Git push rejected

The local and remote branches had diverged.

```text
Your branch and 'origin/master' have diverged,
and have 1 and 1 different commits each
```

### Resolution

The remote branch was fetched first:

```bash
git fetch origin
```

The branch history was then inspected:

```bash
git log --oneline --graph --decorate --all
```

This prevented blindly overwriting the remote branch.

---

## Challenge 2 — npm dependency conflict

GitHub Actions initially failed during:

```bash
npm ci
```

with:

```text
ERESOLVE could not resolve
```

The problem was caused by incompatible Nx versions.

For example, different Nx packages were using versions such as:

```text
@nx/eslint 23.x
@nx/jest 17.x
@nx/node 17.x
@nx/workspace 23.x
```

### Resolution

The Nx ecosystem was aligned to the existing project version:

```text
Nx 17.2.6
```

The following packages were kept on the compatible Nx 17.2.6 release:

```text
@nx/esbuild
@nx/eslint
@nx/eslint-plugin
@nx/jest
@nx/js
@nx/node
@nx/workspace
nx
```

The compatible esbuild version was also restored:

```text
esbuild ~0.19.2
```

This avoids mixing major Nx versions.

---

## Challenge 3 — `npm audit fix --force` caused an out-of-memory error

Running:

```bash
npm audit fix --force
```

attempted major dependency upgrades, including Nx and esbuild.

On the `t3.micro` development server, the process consumed too much memory.

The Linux kernel reported:

```text
Out of memory: Killed process npm audit fix
```

### Resolution

The dependency tree was cleaned and rebuilt using versions compatible with the existing Nx 17 project rather than blindly applying major upgrades with:

```bash
npm audit fix --force
```

The `t3.micro` memory limitation was also identified as a contributing factor.

---

## Challenge 4 — `nx: command not found`

The application initially failed with:

```text
sh: line 1: nx: command not found
```

### Cause

The `node_modules` directory was in an inconsistent state because of dependency conflicts and interrupted npm operations.

### Resolution

The dependency installation was cleaned up and the Nx packages were aligned to the same version before reinstalling dependencies.

---

## Challenge 5 — npm installation was terminated

An npm installation process was observed consuming significant memory.

The server showed an OOM event:

```text
oom-kill
```

### Resolution

Memory usage was investigated using:

```bash
dmesg | tail -30
```

The problem was confirmed as a system-level out-of-memory condition.

The project dependencies were then kept on a consistent version set and unnecessary forced upgrades were avoided.

---

## Challenge 6 — AWS region configuration

The initial CI/CD configuration used:

```text
ap-south-1
```

The AWS resources were subsequently created in:

```text
us-east-1
```

### Resolution

The GitHub Actions AWS region configuration must match the region where ECR and ECS resources actually exist:

```text
AWS_REGION=us-east-1
```

The same region should be used consistently in AWS CLI commands, ECR, ECS, and GitHub Actions.

---

# 14. Important Operational Notes

### Do not commit `.env`

```text
.env
```

is excluded through `.gitignore`.

### Do not use `npm audit fix --force` blindly

The command can introduce major-version dependency changes and break an existing Nx project.

Instead:

```bash
npm audit
```

Review the vulnerabilities and upgrade dependencies in a controlled manner.

### Keep Nx versions aligned

All Nx packages should use the same major/minor release family.

For this project:

```text
Nx 17.2.6
```

is the baseline currently used by the application.

---

# 15. Useful Commands

### Install

```bash
npm ci
```

### Development

```bash
npm start
```

### Build

```bash
npm run build
```

### Test

```bash
npm test
```

### Prisma

```bash
npx prisma generate
npx prisma migrate deploy
npx prisma db seed
```

### Check dependencies

```bash
npm list nx @nx/esbuild @nx/eslint @nx/jest @nx/node @nx/workspace esbuild eslint
```

### Check security vulnerabilities

```bash
npm audit
```

### Docker

```bash
docker build -t realworld-backend .
```

### AWS region

```bash
aws configure get region
```

### ECS deployment

```bash
aws ecs update-service \
  --cluster realworld-cluster \
  --service realworld-backend \
  --force-new-deployment \
  --region us-east-1
```

---

# 16. License

This project is licensed under the MIT License.
