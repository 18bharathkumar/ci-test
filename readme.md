# 📘 Demo DevOps Workstation Setup Guide

### Ubuntu Server · CI/CD · Developer Tooling

---

## 1. Purpose of the Demo Workstation

This demo workstation provides hands-on experience for team members before rolling out a full organizational DevOps workstation.

**The goals are to:**
- Understand workstation provisioning.
- Practice Ubuntu Server administration.
- Learn CI/CD using self-hosted runners.
- Simulate real-world DevOps workflows.
- Create reusable internal documentation.

> [!NOTE]
> This environment acts as a safe sandbox, not a production system.

---

## 2. What This Workstation Includes

The demo workstation is designed to include:
- **Ubuntu Server**: Isolated VM environment.
- **System Administration**: Core configuration and management.
- **Developer Tools**: Git, curl, monitoring utilities.
- **GitHub Actions**: Self-hosted CI runner setup.
- **Security**: Secure user and permission configuration.
- **Service Management**: Process management with `systemd`.
- **Documentation**: Clear guides for learning and reuse.

---

## 3. High-Level Architecture

```mermaid
graph TD
    A[Developer / Trainee] --> B[Host Machine (Linux)]
    B --> C[Multipass]
    C --> D[Ubuntu Server VM (Demo Workstation)]
    D --> E[GitHub Actions Runner]
    E --> F[GitHub Repository (CI workflows)]
```

*Alternatively, viewed as a stack:*
1. **Developer / Trainee**
2. **Host Machine (Linux)**
3. **Multipass**
4. **Ubuntu Server VM (Demo Workstation)**
5. **GitHub Actions Runner**
6. **GitHub Repository (CI workflows)**

---

## 4. Host Machine Preparation

### 4.1 Requirements
- Linux host OS
- Internet access
- `sudo` privileges
- Snap enabled

**Minimum recommended hardware:**
- 8 GB RAM
- 4 CPU cores
- 50 GB disk space

### 4.2 Install Multipass
```bash
sudo snap install multipass
```

**Verify installation:**
```bash
multipass version
```

**Check service status:**
```bash
systemctl status snap.multipass.multipassd
```

---

## 5. Virtual Machine Provisioning

### 5.1 Create Ubuntu Server VM
```bash
multipass launch \
  --name demo-workstation \
  --memory 4G \
  --cpus 2 \
  --disk 30G
```

**Verify VM is running:**
```bash
multipass list
```

### 5.2 Access the VM
```bash
multipass shell demo-workstation
```

---

## 6. Base Ubuntu Server Configuration

### 6.1 System Update
```bash
sudo apt update && sudo apt upgrade -y
```

### 6.2 Install Core Utilities
```bash
sudo apt install -y \
  git \
  curl \
  wget \
  tar \
  unzip \
  ca-certificates \
  htop \
  net-tools \
  vim
```

**Tool Purposes:**
- **Git**: Source control management.
- **curl/wget**: API interaction and downloads.
- **htop**: Real-time system monitoring.
- **net-tools**: Networking diagnostics.

---

## 7. User & Access Management

### 7.1 Create DevOps User
```bash
sudo useradd -m devops
sudo passwd devops
sudo usermod -aG sudo devops
```

**Switch to the new user:**
```bash
sudo su - devops
```

**Why this step?**
- Avoids working as the `root` user.
- Matches real-world production security practices.

---

## 8. Git Configuration (Developer Simulation)
```bash
git config --global user.name "Demo DevOps User"
git config --global user.email "devops-demo@organization.com"
```

**Test repository access:**
```bash
git clone https://github.com/<org>/<demo-repo>
```

---

## 9. CI/CD Concepts Introduced

This workstation demonstrates key CI/CD principles:
- **CI Runners**: What they are and how they function.
- **Job Dispatching**: How GitHub sends tasks to runners.
- **Workflow Triggers**: Events that start the automation.
- **Execution**: How jobs run on local infrastructure.

---

## 10. GitHub Actions Self-Hosted Runner Setup

### 10.1 Generate Runner Token (GitHub)
Navigate to:
`Repository` → `Settings` → `Actions` → `Runners` → `New self-hosted runner`

**Select Options:**
- **OS**: Linux
- **Architecture**: x64

### 10.2 Download Runner
```bash
mkdir actions-runner && cd actions-runner

curl -o actions-runner-linux-x64.tar.gz \
  https://github.com/actions/runner/releases/download/v2.331.0/actions-runner-linux-x64.tar.gz

tar xzf actions-runner-linux-x64.tar.gz
```

### 10.3 Configure Runner
```bash
./config.sh \
  --url https://github.com/<organization>/<repository> \
  --token <RUNNER_TOKEN>
```

**Recommended choices:**
- **Runner name**: `demo-ci-runner`
- **Labels**: `self-hosted`, `demo`, `linux`
- **Work directory**: `default` (hit enter)

### 10.4 Test Runner (Manual Mode)
```bash
./run.sh
```
*Expected Output:* `Listening for Jobs`
*To stop:* Press `Ctrl + C`

---

## 11. Install Runner as a Service (Persistent Mode)
```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

**Verify status:**
```bash
c
```
*Expected Output:* `active (running)`

---

## 12. CI Workflow Example

Create this file at `.github/workflows/ci.yml`:

```yaml
name: Demo CI

on:
  push:
    branches: [ main ]

jobs:
  demo:
    runs-on: self-hosted

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Verify runner
        run: |
          echo "CI runner is active"
          hostname
          whoami
```

---

## 13. Observing CI Execution

Trainees should observe:
- **Job pickup delay**: Time between push and execution.
- **Live Logs**: Watching output in the GitHub Actions UI.
- **On-VM Execution**: Verifying commands run on the workstation.
- **Runner Status**: Transitioning between `Idle` and `Busy`.

---

## 14. System Monitoring Commands
```bash
htop       # Interactive process viewer
df -h      # Disk space usage
free -h    # Memory usage
uptime     # System load and duration
```

**Purpose:** Understand resource consumption during active CI jobs.

---

## 15. Security & Best Practices
- ⚠️ **Never** run the runner as the `root` user.
- ⚠️ **Do not** accept untrusted pull requests (malicious code can run on your VM).
- 🔐 **Rotate** runner tokens after each demo session.
- 🌐 **Isolate** the VM from sensitive production networks.

---

## 16. Resetting the Demo Environment

**Stop and Remove Runner:**
```bash
sudo ./svc.sh stop
sudo ./svc.sh uninstall
./config.sh remove
```

**Destroy VM:**
```bash
multipass delete demo-workstation
multipass purge
```

---

## 17. Learning Outcomes

By the end of this demo, participants will understand:
- VM-based workstation infrastructure.
- Linux server administration basics.
- CI/CD execution flows and runner architecture.
- DevOps operational discipline and security.

---

## 18. Future Enhancements
- [ ] Docker-based CI jobs.
- [ ] Automated artifact storage.
- [ ] Centralized secrets management.
- [ ] Multi-runner parallel execution.
- [ ] Full monitoring (Prometheus/Grafana) and logging stack.

---

## 19. Conclusion

This demo workstation provides a practical, hands-on DevOps learning environment that mirrors real organizational infrastructure. It serves as a vital foundation for standardizing DevOps practices across the team.