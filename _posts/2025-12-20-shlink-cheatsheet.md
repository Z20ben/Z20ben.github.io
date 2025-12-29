---
layout: post
title: "Shlink Kubernetes Cheat Sheet"
date: 2025-01-05 16:30:00 +0700
categories: [kubernetes, cheatsheet]
tags: [shlink, k8s, quick-reference]
image:
  path: /assets/img/shlink-cheatsheet.png
  lqip: data:image/webp;base64,UklGRogAAABXRUJQVlA4IHwAAACwAwCdASoSAAwAPzmEuVOvKKWisAgB4CcJbACsGuAAUHyeZ3qPkAD+qbN23lRyj2IZKgpsl0Ym5mk7zfD1EMBfGGK7fmCxdLew0Rys9NoZgh2bD/7zezcmPQFAwRL+MALv71bHUN11qpzkIUMSgxc4XN1JcZMgYg3ry3AA
  alt: Shlink Kubernetes Cheat Sheet
description: "Quick reference commands untuk manage Shlink di Kubernetes"
toc: true
---

# Shlink Kubernetes Cheat Sheet

## Quick Commands

### Status Checks
```bash
# All pods
kubectl get pods -n shlink

# Services
kubectl get svc -n shlink

# Secrets
kubectl get secrets -n shlink

# Logs
kubectl logs -n shlink -l app=shlink --tail=50
kubectl logs -n shlink -l app=shlink-db --tail=50
```

### Create Short URL
```bash
# Get API key
API_KEY=$(kubectl get secret shlink-app-secret -n shlink -o jsonpath='{.data.INITIAL_API_KEY}' | base64 -d)

# Create short URL
curl -X POST https://sh.yourdomain.com/rest/v3/short-urls \
  -H "X-Api-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"longUrl": "https://example.com", "customSlug": "ex"}'
```

### List Short URLs
```bash
curl https://sh.yourdomain.com/rest/v3/short-urls \
  -H "X-Api-Key: $API_KEY"
```

### Backup & Restore
```bash
# Backup
kubectl exec -n shlink $(kubectl get pod -n shlink -l app=shlink-db -o jsonpath='{.items[0].metadata.name}') \
  -- mysqldump -u root -pYOUR_ROOT_PASSWORD --all-databases > backup.sql

# Restore
kubectl exec -i -n shlink $(kubectl get pod -n shlink -l app=shlink-db -o jsonpath='{.items[0].metadata.name}') \
  -- mysql -u root -pYOUR_ROOT_PASSWORD < backup.sql
```

### Scale
```bash
# Scale up
kubectl scale deployment shlink -n shlink --replicas=3

# Scale down
kubectl scale deployment shlink -n shlink --replicas=1
```

### Update
```bash
# Update image
kubectl set image deployment/shlink shlink=shlinkio/shlink:4.3.0 -n shlink

# Restart
kubectl rollout restart deployment shlink -n shlink
```

### Delete
```bash
# Delete everything
kubectl delete namespace shlink
```
