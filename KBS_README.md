# Kubernetes Server Monitoring Practice Guide

> **DevOps Reference for Kubernetes Cluster Management**

<div align="center">

![Kubernetes](https://img.shields.io/badge/kubernetes-326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![VMware](https://img.shields.io/badge/VMware-607078?style=for-the-badge&logo=vmware&logoColor=white)
![Shell Script](https://img.shields.io/badge/shell_script-%23121011.svg?style=for-the-badge&logo=gnu-bash&logoColor=white)

**Environment:** Tanzu Kubernetes Cluster on vSphere  
**Last Updated:** June 2025

</div>

---

## Table of Contents

- [ Cluster Authentication](#-cluster-authentication)
- [Pod Management](#-pod-management)
- [Log Analysis](#-log-analysis)
- [Pod Shell Access](#️-pod-shell-access)
- [Log Navigation & Search](#-log-navigation--search)
- [File Compression](#-file-compression)
- [Advanced Monitoring](#-advanced-monitoring)
- [Troubleshooting](#️-troubleshooting)
- [Quick Commands](#-quick-commands)
- [Productivity Tips](#-productivity-tips)

---

## Cluster Authentication

### Login to Tanzu Kubernetes Cluster

Connect securely to your cluster namespace:

```bash
kubectl vsphere login \
  --insecure-skip-tls-verify \
  --server=17X.XX.XX.X \
  -u ibuat@vsphere.local \
  --tanzu-kubernetes-cluster-namespace=ib-dev \
  --tanzu-kubernetes-cluster-name=dev-ib-cluster
```

> **Important:** Ensure correct spelling of `--tanzu-kubernetes-cluster-name` flag  
> **Note:** Use `--insecure-skip-tls-verify` only in development environments
There are 03 enviroments  

| Enviroment | Purpose |
|------------|---------|
| `ib-dev` | For Development |
| `ib-prod` | For Productions |
| `ib-uat` | For the testing |

---

## Pod Management

### Basic Pod Operations

<details>
<summary><strong> Essential Pod Commands</strong></summary>

```bash
# List all pods in namespace
kubectl get pods -n ib-dev

# Detailed pod information with node placement
kubectl get pods -n ib-dev -o wide

# Real-time pod monitoring
kubectl get pods -n ib-dev -w

# Filter pods by labels
kubectl get pods -n ib-dev -l app=portal
```

</details>

**Quick Reference:**
| Command | Purpose |
|---------|---------|
| `kubectl get pods -n ib-dev` | List pods |
| `kubectl get pods -n ib-dev -o wide` | Detailed view |
| `kubectl get pods -n ib-dev -w` | Watch mode |
| `kubectl get pods -n ib-dev -l app=portal` | Label filtering |

---

## Log Analysis

### Container Log Management

#### Live Log Streaming

```bash
# Multi-container logs with label selector
kubectl logs --all-containers=true -l app=portal -n ib-dev -f --max-log-requests=20
```

#### Log Export (Windows)

```powershell
kubectl logs --all-containers=true -l app=portal -n ib-dev -f --max-log-requests=20 > C:\Users\it232059\Desktop\LOGS\1.txt
```

#### Targeted Log Commands

<details>
<summary><strong>Advanced Log Options</strong></summary>

```bash
# Specific pod logs
kubectl logs <pod-name> -n ib-dev -f

# Multi-container pod (specific container)
kubectl logs <pod-name> -c <container-name> -n ib-dev -f

# Previous container instance (crash analysis)
kubectl logs <pod-name> -n ib-dev --previous

# Logs with timestamps
kubectl logs <pod-name> -n ib-dev --timestamps=true

# Limited log output
kubectl logs <pod-name> -n ib-dev --tail=100

# Time-based filtering
kubectl logs <pod-name> -n ib-dev --since=1h
```

</details>

---

## Pod Shell Access

### Interactive Pod Sessions

#### Shell Access Methods

```bash
# Identify target pod
kubectl get pods -n ib-dev

# Bash access (primary method)
kubectl exec -it content-556b5d8b7-wtr26 -n ib-dev -- /bin/bash

# Multi-container specification
kubectl exec -it <pod-name> -c <container-name> -n ib-dev -- /bin/bash
```

#### Alternative Shell Options

<details>
<summary><strong>Fallback Commands</strong></summary>

```bash
# Fallback to sh if bash unavailable
kubectl exec -it <pod-name> -n ib-dev -- /bin/sh

# Single command execution
kubectl exec <pod-name> -n ib-dev -- ls -la /logs

# Direct file inspection
kubectl exec <pod-name> -n ib-dev -- cat /app/config.yml
```

</details>

---

## Log Navigation & Search

### Inside Pod Log Operations

#### Directory Navigation

```bash
# Navigate to logs directory
cd /logs/content

# List all files with details
ls -la

# Find log files
find . -name "*.log" -type f
```

#### Advanced Search Techniques

<details>
<summary><strong>Pattern Matching & Filtering</strong></summary>

```bash
# Basic warning search
cat today-payment-b5c48877d-w86z2.log | grep WARN

# Case-insensitive error search
grep -i "error" *.log

# Context search (3 lines before/after)
grep -A 3 -B 3 "WARN" today-payment-b5c48877d-w86z2.log

# Multiple pattern matching
grep -E "(ERROR|WARN|FATAL)" *.log

# Count occurrences
grep -c "ERROR" *.log

# Search compressed files
zgrep "ERROR" *.log.gz

# Real-time log monitoring
tail -f *.log | grep --color=always "ERROR"
```

</details>

**Search Pattern Examples:**
| Pattern | Command | Use Case |
|---------|---------|----------|
| `WARN` | `grep "WARN" *.log` | Find warnings |
| `ERROR\|FATAL` | `grep -E "(ERROR\|FATAL)" *.log` | Critical issues |
| `2025-06-06` | `grep "2025-06-06" *.log` | Date-specific logs |

---

## File Compression

### Gzip Operations

####  Decompression Commands

```bash
# Standard decompression
gzip -d log-payment-65b8d9b49f-xmdjp.2025-05-15_16-19.0.log.gz

# Keep original while decompressing
gzip -dk log-file.log.gz

# Bulk decompression
gzip -d *.gz
```

####  View Without Decompressing

```bash
# Preview compressed file
zcat log-file.log.gz | head -n 50

# Search in compressed files
zgrep "ERROR" *.log.gz

# Pipe to less for pagination
zcat log-file.log.gz | less
```

>  **File Naming:** Ensure correct spelling (`.log.gz` not `.1og.gz`)

---

## Advanced Monitoring

### Resource Monitoring

<details>
<summary><strong> Resource Usage Commands</strong></summary>

```bash
# Pod resource consumption
kubectl top pods -n ib-dev

# Node resource overview
kubectl top nodes

# Detailed resource metrics
kubectl describe pod <pod-name> -n ib-dev

# Resource limits and requests
kubectl get pods -n ib-dev -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[*].resources}{"\n"}{end}'
```

</details>

### Event Monitoring

```bash
# Recent events (sorted)
kubectl get events -n ib-dev --sort-by='.lastTimestamp'

# Live event monitoring
kubectl get events -n ib-dev -w

# Event filtering
kubectl get events -n ib-dev --field-selector reason=Failed
```

### Service Discovery

```bash
# List services
kubectl get services -n ib-dev

# Service endpoints
kubectl get endpoints -n ib-dev

# Service details
kubectl describe service <service-name> -n ib-dev
```

---

## Troubleshooting

### Common Issues & Solutions

####  CrashLoopBackOff

```bash
kubectl describe pod <pod-name> -n ib-dev
kubectl logs <pod-name> -n ib-dev --previous
```

####  Pending Pods

```bash
kubectl describe pod <pod-name> -n ib-dev
kubectl get events -n ib-dev
```

####  Restart Monitoring

```bash
kubectl get pods -n ib-dev -o wide
```

####  Port Forwarding

```bash
kubectl port-forward <pod-name> 8080:8080 -n ib-dev
```

**Troubleshooting Checklist:**
- [ ] Check pod status with `kubectl get pods`
- [ ] Review events with `kubectl get events`
- [ ] Examine logs with `kubectl logs`
- [ ] Verify resource usage with `kubectl top`
- [ ] Check service connectivity

---

##  Quick Commands

### Log Management Best Practices

####  Log Rotation & Cleanup

<details>
<summary><strong>Maintenance Commands</strong></summary>

```bash
# Clean up old logs (7+ days)
find /logs -name "*.log" -mtime +7 -delete

# Compress logs older than 1 day
find /logs -name "*.log" -mtime +1 -exec gzip {} \;

# Monitor disk usage
du -sh /logs/*

# Clean up by size
find /logs -name "*.log" -size +100M -delete
```

</details>

---

##  Productivity Tips

### Command Aliases

Add to your `~/.bashrc` or `~/.zshrc`:

```bash
# Essential Kubernetes aliases
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kd='kubectl describe'
alias kl='kubectl logs'
alias ke='kubectl exec -it'

# Namespace-specific aliases
alias kgpd='kubectl get pods -n ib-dev'
alias kld='kubectl logs -n ib-dev'
alias ked='kubectl exec -it -n ib-dev'
```



<div align="center">

### 

**Need Help?** Check the [Kubernetes Documentation](https://kubernetes.io/docs/) or [Tanzu Documentation](https://docs.vmware.com/en/VMware-Tanzu/)

---

*Last updated: June 2025 | Version 1.0*

</div>













































