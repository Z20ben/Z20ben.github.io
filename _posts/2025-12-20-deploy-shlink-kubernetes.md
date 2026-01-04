---
layout: post
title: "Deploy Shlink URL Shortener di Kubernetes dengan Cloudflare Tunnel"
date: 2025-01-05 15:30:00 +0700
categories: [kubernetes, devops, tutorial]
tags: [shlink, k8s, cloudflare, url-shortener, self-hosted]
author: Fredika
image:
  path: /assets/img/deploy-shlink-kubernetes.png
  lqip: data:image/webp;base64,UklGRnoAAABXRUJQVlA4IG4AAADwAwCdASoUAAoAPzmEuVOvKKWisAgB4CcJZgCdMoABAryh07rHqjAAAP7AZmHZ857TaJTJk0MInmNDP+zee0j37mUj/oNlDI3alGpBqyGAI0vt5f1fA6p3ecpm7k27O52gsMo9epig78IJPZyAAA==
  alt: Deploy Shlink URL Shortener di Kubernetes dengan Cloudflare Tunnel
description: "Tutorial lengkap deploy Shlink URL shortener dengan MariaDB di Kubernetes (k3s), secure secrets management, dan public access via Cloudflare Tunnel"
toc: true
---

## Pendahuluan

Shlink adalah URL shortener open-source yang powerful dan self-hosted. Dalam tutorial ini, kita akan deploy Shlink di Kubernetes cluster dengan:

- ✅ MariaDB 11.6 sebagai database
- ✅ Secure secrets management (tanpa hardcode password)
- ✅ High availability (2 replicas)
- ✅ Public access via Cloudflare Tunnel
- ✅ Web UI untuk management
- ✅ Real-time analytics & tracking

## Arsitektur
```
Internet
    ↓
Cloudflare Tunnel (cloudflared)
    ↓
Kubernetes Services
    ├── Shlink API (2 replicas)
    ├── Shlink Web Client (1 replica)
    └── MariaDB Database (1 replica)
```

## Prerequisites

- Kubernetes cluster (k3s) yang sudah running
- kubectl access
- Cloudflare account dengan domain
- Cloudflare Tunnel sudah di-setup
- Minimal 2GB RAM available

## Step 1: Persiapan Namespace

Buat namespace dedicated untuk Shlink:
```bash
kubectl create namespace shlink
```

Verify namespace:
```bash
kubectl get namespaces | grep shlink
```

## Step 2: Generate Secure Passwords

Generate strong random passwords untuk database dan API key:
```bash
# Generate 3 passwords
openssl rand -base64 32  # Database root password
openssl rand -base64 32  # Database user password
openssl rand -base64 32  # Shlink API key
```

**⚠️ Simpan passwords ini di password manager!**

## Step 3: Create Kubernetes Secrets

Buat secrets untuk menyimpan credentials (ganti `YOUR_PASSWORD_HERE` dengan password yang di-generate):
```bash
# Database secret
kubectl create secret generic shlink-db-secret \
  --from-literal=MYSQL_ROOT_PASSWORD='YOUR_ROOT_PASSWORD' \
  --from-literal=MYSQL_DATABASE='shlink' \
  --from-literal=MYSQL_USER='shlink_user' \
  --from-literal=MYSQL_PASSWORD='YOUR_USER_PASSWORD' \
  --namespace=shlink

# Application secret
kubectl create secret generic shlink-app-secret \
  --from-literal=DEFAULT_DOMAIN='sh.yourdomain.com' \
  --from-literal=INITIAL_API_KEY='YOUR_API_KEY' \
  --from-literal=GEOLITE_LICENSE_KEY='' \
  --namespace=shlink
```

Verify secrets:
```bash
kubectl get secrets -n shlink
```

Output:
```
NAME                TYPE     DATA   AGE
shlink-app-secret   Opaque   3      10s
shlink-db-secret    Opaque   4      15s
```

## Step 4: Deploy MariaDB Database

Create file `shlink-database.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shlink-db-pvc
  namespace: shlink
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shlink-db
  namespace: shlink
  labels:
    app: shlink-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: shlink-db
  template:
    metadata:
      labels:
        app: shlink-db
    spec:
      containers:
      - name: mariadb
        image: mariadb:11.6
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: shlink-db-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: shlink-db-secret
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: shlink-db-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: shlink-db-secret
              key: MYSQL_PASSWORD
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: db-data
          mountPath: /var/lib/mysql
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 20
          periodSeconds: 5
      volumes:
      - name: db-data
        persistentVolumeClaim:
          claimName: shlink-db-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: shlink-db
  namespace: shlink
spec:
  selector:
    app: shlink-db
  ports:
  - port: 3306
    targetPort: 3306
  clusterIP: None
```

Deploy:
```bash
kubectl apply -f shlink-database.yaml
```

Monitor deployment:
```bash
kubectl get pods -n shlink -w
```

Tunggu hingga pod status `1/1 Running` (sekitar 1-2 menit).

## Step 5: Deploy Shlink Application

Create file `shlink-application.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: shlink-config
  namespace: shlink
data:
  SHORT_DOMAIN_HOST: "sh.yourdomain.com"
  SHORT_DOMAIN_SCHEMA: "https"
  TIMEZONE: "Asia/Jakarta"
  DB_DRIVER: "maria"
  DB_NAME: "shlink"
  DB_USER: "shlink_user"
  DB_HOST: "shlink-db"
  DB_PORT: "3306"
  REDIRECT_STATUS_CODE: "302"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shlink
  namespace: shlink
spec:
  replicas: 2
  selector:
    matchLabels:
      app: shlink
  template:
    metadata:
      labels:
        app: shlink
    spec:
      containers:
      - name: shlink
        image: shlinkio/shlink:4.2.4
        ports:
        - containerPort: 8080
        env:
        - name: SHORT_DOMAIN_HOST
          valueFrom:
            configMapKeyRef:
              name: shlink-config
              key: SHORT_DOMAIN_HOST
        - name: SHORT_DOMAIN_SCHEMA
          valueFrom:
            configMapKeyRef:
              name: shlink-config
              key: SHORT_DOMAIN_SCHEMA
        - name: TIMEZONE
          valueFrom:
            configMapKeyRef:
              name: shlink-config
              key: TIMEZONE
        - name: DB_DRIVER
          valueFrom:
            configMapKeyRef:
              name: shlink-config
              key: DB_DRIVER
        - name: DB_NAME
          valueFrom:
            configMapKeyRef:
              name: shlink-config
              key: DB_NAME
        - name: DB_USER
          valueFrom:
            configMapKeyRef:
              name: shlink-config
              key: DB_USER
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: shlink-config
              key: DB_HOST
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: shlink-db-secret
              key: MYSQL_PASSWORD
        - name: DEFAULT_DOMAIN
          valueFrom:
            secretKeyRef:
              name: shlink-app-secret
              key: DEFAULT_DOMAIN
        - name: INITIAL_API_KEY
          valueFrom:
            secretKeyRef:
              name: shlink-app-secret
              key: INITIAL_API_KEY
        livenessProbe:
          httpGet:
            path: /rest/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /rest/health
            port: 8080
          initialDelaySeconds: 20
          periodSeconds: 5
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: shlink-service
  namespace: shlink
spec:
  selector:
    app: shlink
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

Deploy:
```bash
kubectl apply -f shlink-application.yaml
```

Verify:
```bash
kubectl get pods -n shlink
kubectl logs -n shlink -l app=shlink --tail=20
```

## Step 6: Deploy Web Client (Optional)

Create file `shlink-web-client.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shlink-web-client
  namespace: shlink
spec:
  replicas: 1
  selector:
    matchLabels:
      app: shlink-web-client
  template:
    metadata:
      labels:
        app: shlink-web-client
    spec:
      containers:
      - name: web-client
        image: shlinkio/shlink-web-client:4.2.1
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: shlink-web-client
  namespace: shlink
spec:
  selector:
    app: shlink-web-client
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

Deploy:
```bash
kubectl apply -f shlink-web-client.yaml
```

## Step 7: Configure Cloudflare Tunnel

Get service cluster IPs:
```bash
kubectl get svc -n shlink
```

Update Cloudflare Tunnel config (`~/.cloudflared/config.yml`):
```yaml
tunnel: YOUR_TUNNEL_ID
credentials-file: /path/to/credentials.json

ingress:
  # Shlink Admin UI
  - hostname: sh-admin.yourdomain.com
    service: http://SHLINK_WEB_CLIENT_IP:80

  # Shlink API
  - hostname: sh.yourdomain.com
    service: http://SHLINK_SERVICE_IP:80

  # Catch-all
  - service: http_status:404
```

Update Kubernetes configmap:
```bash
kubectl delete configmap cloudflared-config -n cloudflare
kubectl create configmap cloudflared-config \
  --from-file=config.yaml=~/.cloudflared/config.yml \
  -n cloudflare

kubectl rollout restart deployment cloudflared -n cloudflare
```

Add DNS routes:
```bash
cloudflared tunnel route dns YOUR_TUNNEL_NAME sh.yourdomain.com
cloudflared tunnel route dns YOUR_TUNNEL_NAME sh-admin.yourdomain.com
```

## Step 8: Test & Configure

### Test Health Endpoint
```bash
curl https://sh.yourdomain.com/rest/health
```

Expected output:
```json
{
  "status": "pass",
  "version": "4.2.4"
}
```

### Access Web UI

1. Open browser: `https://sh-admin.yourdomain.com`
2. Click "Add server"
3. Server URL: `https://sh.yourdomain.com`
4. API Key: (retrieve dengan command)
```bash
kubectl get secret shlink-app-secret -n shlink \
  -o jsonpath='{.data.INITIAL_API_KEY}' | base64 -d
```

5. Click "Add server"

### Create First Short URL

Via Web UI:
1. Click "Create short URL"
2. Long URL: `https://example.com`
3. Custom slug: `example`
4. Click "Create"

Result: `https://sh.yourdomain.com/example`

Via API:
```bash
API_KEY=$(kubectl get secret shlink-app-secret -n shlink \
  -o jsonpath='{.data.INITIAL_API_KEY}' | base64 -d)

curl -X POST https://sh.yourdomain.com/rest/v3/short-urls \
  -H "X-Api-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "longUrl": "https://example.com",
    "customSlug": "example"
  }'
```

## Troubleshooting

### Pods Tidak Running
```bash
# Check pod events
kubectl describe pod -n shlink POD_NAME

# Check logs
kubectl logs -n shlink POD_NAME
```

### Database Connection Error
```bash
# Test database connection
POD=$(kubectl get pod -n shlink -l app=shlink-db -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n shlink $POD -- mariadb -u shlink_user -p shlink -e "SHOW DATABASES;"
```

### Cloudflare Tunnel Not Connecting
```bash
# Check tunnel logs
kubectl logs -n cloudflare -l app=cloudflared --tail=50
```

## Maintenance

### Backup Database
```bash
POD=$(kubectl get pod -n shlink -l app=shlink-db -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n shlink $POD -- mysqldump -u root -p \
  --all-databases > shlink-backup-$(date +%Y%m%d).sql
```

### Update Shlink
```bash
# Update image version
kubectl set image deployment/shlink shlink=shlinkio/shlink:4.3.0 -n shlink

# Watch rollout
kubectl rollout status deployment/shlink -n shlink
```

### Scale Replicas
```bash
# Scale up
kubectl scale deployment shlink -n shlink --replicas=3

# Scale down
kubectl scale deployment shlink -n shlink --replicas=1
```

## Security Best Practices

1. ✅ Never commit secrets to Git
2. ✅ Use strong random passwords
3. ✅ Rotate credentials regularly
4. ✅ Enable Cloudflare WAF
5. ✅ Monitor access logs
6. ✅ Keep images updated

## Resource Requirements

| Component | CPU Request | Memory Request | CPU Limit | Memory Limit |
|-----------|-------------|----------------|-----------|--------------|
| MariaDB | 250m | 256Mi | 500m | 512Mi |
| Shlink App (×2) | 250m | 256Mi | 500m | 512Mi |
| Web Client | 100m | 64Mi | 200m | 128Mi |
| **Total** | **850m** | **832Mi** | **1700m** | **1664Mi** |

## Kesimpulan

Kita telah berhasil deploy Shlink URL shortener di Kubernetes dengan:

- ✅ Secure secrets management
- ✅ High availability (2 replicas)
- ✅ Persistent storage untuk database
- ✅ Public access via Cloudflare Tunnel
- ✅ Web UI untuk management
- ✅ Production-ready configuration

## Referensi

- [Shlink Documentation](https://shlink.io/documentation/)
- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- [MariaDB Docker Image](https://hub.docker.com/_/mariadb)

---

*Published: {{ page.date | date: "%B %d, %Y" }}*
*Tags: {% for tag in page.tags %}#{{ tag }} {% endfor %}*
