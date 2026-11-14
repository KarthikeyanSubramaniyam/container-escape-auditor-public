# Container Escape Auditor

<p align="center">
  <img src="https://img.shields.io/badge/Security-Container%20Hardening-red?style=flat" alt="Security Domain">
  <img src="https://img.shields.io/badge/Benchmark-CIS%20Docker%20%26%20K8s-blueviolet?style=flat" alt="CIS Benchmark">
  <img src="https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white" alt="Python">
</p>

An automated static analysis scanner for Dockerfiles, Docker Compose specifications, and Kubernetes pod security contexts. Identifies container escape vectors, excessive Linux capabilities, dangerous filesystem mounts, and privilege escalation vulnerabilities before deployment.

---

## 🎯 The Problem It Solves

Containers do not provide strict hardware-isolated virtualization—they share the host Linux kernel. A single misconfiguration (such as `--privileged` or mounting the Docker socket) enables malicious or compromised container processes to achieve instantaneous, root-level breakout to the underlying host node.

**Container Escape Auditor** proactively inspects infrastructure-as-code manifests to:
1. **Prevent Daemon Hijacking**: Detects high-risk socket mounts (`/var/run/docker.sock`) and kernel filesystems (`/proc`, `/sys`).
2. **Restrict Linux Capabilities**: Flags dangerous capabilities such as `CAP_SYS_ADMIN`, `CAP_SYS_PTRACE`, and `CAP_NET_ADMIN` that allow namespace breakouts and kernel exploitation.
3. **Enforce Least-Privilege Identity**: Flags implicit or explicit root user execution (`USER root` / UID 0) and enforces non-root container profiles.
4. **Enforce CIS Docker & Kubernetes Benchmarks**: Delivers actionable remediation guidelines directly referenced against CIS Benchmark controls.

---

## 🏗️ System Architecture & Workflow

```
[ Dockerfile / Docker Compose / K8s Manifest ]
                      |
                      v
+-------------------------------------+
|        1. AST & Token Parser        | -> Normalizes directives, environment,
|    (Instruction & Key Extraction)   |    volume binds, and capability blocks
+---------------------+---------------+
                      |
                      v
+-------------------------------------+
|    2. Breakout Isolation Checker    | -> Flags 'privileged: true', 'pid: host',
|     (Namespace & Process Sharing)   |    and 'network_mode: host' configurations
+---------------------+---------------+
                      |
                      v
+-------------------------------------+
|      3. Filesystem Mount Audit      | -> Detects bind mounts to sensitive host
|    (Docker Socket & /proc /sys)     |    paths allowing kernel & daemon escape
+---------------------+---------------+
                      |
                      v
+-------------------------------------+
|     4. Capability Evaluator         | -> Identifies dangerous kernel capabilities
|   (CAP_SYS_ADMIN, CAP_SYS_PTRACE)   |    and enforces 'cap_drop: [ALL]'
+---------------------+---------------+
                      |
                      v
+-------------------------------------+
|     5. CIS Scoring & Remediation    | -> Generates posture grade (A..F), score,
|      (DevSecOps JSON & CLI Table)   |    and exact CIS Benchmark remediation
+-------------------------------------+
```

---

## 🚀 Technical Specifications & Rules

| Rule Identifier | Severity | Category | CIS Benchmark | Risk Description |
| :--- | :--- | :--- | :--- | :--- |
| **`SEC-CMP-001`** | `CRITICAL` | Isolation | CIS Docker 5.2 | `privileged: true` disables seccomp and apparmor |
| **`SEC-CMP-003`** | `CRITICAL` | Namespace | CIS Docker 5.15 | `pid: host` allows reading/killing host processes |
| **`SEC-CMP-005`** | `CRITICAL` | Filesystem | CIS Docker 5.31 | Exposing `/var/run/docker.sock` gives full host daemon control |
| **`SEC-CMP-004`** | `CRITICAL` | Capabilities | CIS Docker 5.3 | `CAP_SYS_ADMIN` grants root kernel privileges |
| **`SEC-CMP-002`** | `HIGH` | Network | CIS Docker 5.9 | `network_mode: host` bypasses network container isolation |
| **`SEC-DKR-005`** | `HIGH` | Identity | CIS Docker 4.1 | Missing non-root `USER` directive allows UID 0 execution |

---

## 💻 CLI Usage & Quickstart

### Audit a Dockerfile
```bash
python -m src.container_auditor.cli Dockerfile
```

### Scan Docker Compose Stack with JSON Output
```bash
python -m src.container_auditor.cli docker-compose.yml --json
```

### Recursively Audit Repository Infrastructure
```bash
python -m src.container_auditor.cli ./deployments/
```

### Run Unit Tests
```bash
python -m unittest discover -s tests -p "test_*.py"
```

---

## 🔒 Production Code & Architecture Notice

* **Public Repository**: Outlines container threat modeling, CIS control mapping, and static analysis specifications.
* **Private Repository**: Maintained in `KarthikeyanSubramaniyam/container-escape-auditor` with live Kubernetes admission controller (OPA/Gatekeeper) integrations and runtime eBPF breakout monitors.
