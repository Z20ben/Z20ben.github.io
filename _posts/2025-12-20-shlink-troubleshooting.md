---
layout: post
title: "Shlink Kubernetes Troubleshooting Guide"
date: 2025-01-05 16:00:00 +0700
categories: [kubernetes, troubleshooting]
tags: [shlink, k8s, debugging]
image:
  path: /assets/img/shlink-troubleshooting.png
  lqip: data:image/webp;base64,UklGRmoAAABXRUJQVlA4IF4AAACwAwCdASoSAAwAPzmEuVOvKKWisAgB4CcJQBdgA9hVgEhu2SVgAAD+3NvavKsC+ctQ/3WgtwuBPYSqFnMkVye70iBBufgDcXnlX/QgDu/nR0u70CQbgpGC1TbNQAAA
  alt: Shlink Kubernetes Troubleshooting Guide
description: "Panduan troubleshooting untuk mengatasi masalah umum saat deploy Shlink di Kubernetes"
toc: true
---

# Common Issues & Solutions

## 1. Pods Stuck in ContainerCreating

**Symptom:**
```
shlink-db-xxx   0/1   ContainerCreating
```

**Solution:**
```bash
# Check events
kubectl describe pod -n shlink shlink-db-xxx

# Common causes:
# - Image pull (wait 1-2 minutes)
# - PVC not bound
# - Resource constraints
```

## 2. Database Connection Failed

**Error:** `SQLSTATE[HY000] [2002] Connection refused`

**Solution:**
```bash
# Check DB is running
kubectl get pods -n shlink -l app=shlink-db

# Check DB logs
kubectl logs -n shlink -l app=shlink-db

# Test connection
kubectl exec -n shlink $(kubectl get pod -n shlink -l app=shlink-db -o jsonpath='{.items[0].metadata.name}') \
  -- mariadb -u shlink_user -pYOUR_PASSWORD shlink -e "SELECT 1"
```

## 3. Readiness Probe Failed

**Error:** `Readiness probe failed: mysqladmin: executable file not found`

**Solution:** Update probe to use TCP check:
```yaml
readinessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 20
```

## 4. API Key Not Working

**Solution:**
```bash
# Get correct API key
kubectl get secret shlink-app-secret -n shlink -o jsonpath='{.data.INITIAL_API_KEY}' | base64 -d

# Test
curl https://sh.yourdomain.com/rest/health \
  -H "X-Api-Key: YOUR_KEY"
```

## 5. Out of Memory

**Symptom:** `OOMKilled`

**Solution:**
```bash
# Increase limits
kubectl edit deployment shlink -n shlink

# Change:
resources:
  limits:
    memory: "1Gi"  # Increase from 512Mi
```
