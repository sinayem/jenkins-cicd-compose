# jenkins-cicd-compose

Docker Compose configuration for a hardened, containerised Jenkins CI/CD
environment using Docker-in-Docker (DinD). It builds, tests, scans and
publishes the Node.js app from the forked
[aws-elastic-beanstalk-express-js-sample](https://github.com/aws-samples/aws-elastic-beanstalk-express-js-sample).

## Architecture

```
            SSH tunnel (127.0.0.1:8080)
Developer ─────────────────────────────► jenkins (controller, UID 1000)
                                            │  DOCKER_HOST=tcp://docker:2376 (mutual TLS)
                                            ▼
                                         docker (dind, privileged, no published ports)
                                            ├─ node:16 agent containers (install/test/scan)
                                            └─ docker build / push ──► Docker Hub
```

## Repository layout

| Path | Purpose |
|---|---|
| `docker-compose.yml` | Services, network, volumes, secrets (commented) |
| `jenkins/Dockerfile` | Custom Jenkins image: Docker CLI and plugins |
| `jenkins/plugins.txt` | Pinned plugin list |
| `jenkins/casc.yaml` | Configuration as Code: users, permissions, CSRF, build authorization |
| `secrets/` | Local password files (git-ignored) |
| `docs/` | Screenshots, logs and report notes |

## Security measures

- **No host Docker socket:** Jenkins talks to an isolated DinD daemon over mutual TLS.
- **Docker API not exposed:** DinD has no published ports and sits on a private bridge network.
- **Jenkins runs unprivileged:** the controller runs as `jenkins` (UID 1000); only DinD is privileged.
- **UI bound to localhost:** reachable only through an SSH tunnel; agent port 50000 is disabled.
- **Setup wizard disabled; users defined as code:** sign-up is off and anonymous users have no permissions.
- **Least privilege:** `ci-runner` can only read, build and cancel jobs; `admin` manages configuration.
- **Build authorization:** builds run as the triggering user, not SYSTEM (Authorize Project plugin).
- **Secrets:** passwords are Docker secrets; the Docker Hub token is stored in the Jenkins credentials store.
- **CSRF protection:** enabled.

## Usage

Requirements: Ubuntu 20.04 or later, Docker Engine and the Compose plugin.

```bash
git clone https://github.com/<you>/jenkins-cicd-compose.git && cd jenkins-cicd-compose

# 1. Create the secret files (no trailing newline)
openssl rand -base64 18 | tr -d '\n' > secrets/jenkins_admin_password.txt
openssl rand -base64 18 | tr -d '\n' > secrets/ci_runner_password.txt
chmod 600 secrets/*.txt

# 2. Build and start
docker compose up -d --build

# 3. Verify Jenkins can reach the DinD daemon (should print Client AND Server)
docker compose exec jenkins docker version
```

From your workstation, open a tunnel and browse to <http://localhost:8080>:

```bash
ssh -L 8080:localhost:8080 <user>@<server-ip>
```

Log in as `admin` with the password in `secrets/jenkins_admin_password.txt`.

## After first login

1. **Docker Hub credentials:** *Manage Jenkins → Credentials → Global → Add*.
   Choose *Username with password* (use a Docker Hub **access token** as the password) and set the ID to `dockerhub-creds`.
2. **Audit Trail:** *Manage Jenkins → System → Audit Trail*. Add a *Log file* logger at
   `/var/jenkins_home/audit/audit-%g.log`.
3. **Pipeline job:** *New Item → `YourStudentID_Assessment2_pipeline` → Pipeline*.
   - Definition: *Pipeline script from SCM* → Git → your fork URL, branch `*/main`, script path `Jenkinsfile`
   - Build Triggers: *Poll SCM* `H/5 * * * *`
   - *Discard old builds*: keep 20 builds and artifacts from 10

## Operations

```bash
docker compose logs -f jenkins     # controller logs
docker compose restart jenkins     # restart after editing casc.yaml (rebuild first)
docker compose up -d --build       # apply Dockerfile / plugins / casc changes
docker compose down                # stop (data kept in named volumes)
```
