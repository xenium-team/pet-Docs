# Deploy 1.2.0

## 🔹 เตรียมข้อมูล (Preparation)

### 1. ขอ Subdomain

| Service | Subdomain |
|---------|-----------|
| ads-service | https://ads-service.tailyapp.pet |
| backoffice-service | https://backoffice-service.tailyapp.pet |
| backoffice | https://backoffice.tailyapp.pet |

---

# 🔹 Build Image

## 1. Install Shared Package

```
cd api/apps/shared
npm install
```

---

## 2. Build API Image

ไฟล์

```text
build-and-push.sh
```

เพิ่ม Service

```bash
SERVICES=(
  "ads-service"
  "backoffice-service"
)
```

จากนั้นรัน

```bash
chmod +x build-and-push.sh
./build-and-push.sh
```

---

## 3. Build Web Image

ไฟล์

```text
build-and-push-web.sh
```

เพิ่ม

```bash
WEB_SERVICES=(
  "backoffice"
)
```

จากนั้นรัน

```bash
chmod +x build-and-push-web.sh
./build-and-push-web.sh
```

---

# 🔹 Kubernetes Deployment

## 1. เพิ่ม Cluster ใหม่

แก้ไขไฟล์

```text
api/k8s/ads-service/deployment.yaml
api/k8s/backoffice-service/deployment.yaml
api/k8s/backoffice/deployment.yaml
```

แก้ไข Image

```yaml
image: <Image Repository>/ads-service:latest
image: <Image Repository>/backoffice-service:latest
image: <Image Repository>/backoffice:latest
```

ตรวจสอบและแก้ไขค่า `env` ให้ถูกต้องตาม Cluster

จากนั้นคัดลอกไฟล์ `deployment.yaml` ไปยัง Server

---

## 2. Deploy Services

```bash
kubectl apply -f ./k8s/ads-service/deployment.yaml -n pet-app

kubectl apply -f ./k8s/backoffice-service/deployment.yaml -n pet-app

kubectl apply -f ./k8s/backoffice/deployment.yaml -n pet-app
```

---

## 3. แก้ไข Ingress

แก้ไขไฟล์

```text
api/k8s/ingress.yaml
```

### เพิ่ม TLS

```yaml
tls:
  - hosts:
      - ads-service.tailyapp.pet
    secretName: ads-tls

  - hosts:
      - backoffice-service.tailyapp.pet
    secretName: backoffice-api-tls

  - hosts:
      - backoffice.tailyapp.pet
    secretName: backoffice-tls
```

### เพิ่ม Rules

```yaml
rules:
  - host: ads-service.tailyapp.pet
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: ads-service
              port:
                number: 3009

  - host: backoffice.tailyapp.pet
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: backoffice
              port:
                number: 3011

  - host: backoffice-service.tailyapp.pet
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: backoffice-service
              port:
                number: 3008
```

Apply

```bash
kubectl apply -f ./k8s/ingress.yaml
```

---

## 4. เพิ่ม Certificate (TLS)

แก้ไขไฟล์

```text
api/k8s/cluster-issuer.yaml
```

เพิ่ม Certificate

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: ads-tls
  namespace: pet-app
spec:
  secretName: ads-tls
  issuerRef:
    name: letsencrypt-http
    kind: ClusterIssuer
  commonName: ads-service.tailyapp.pet
  dnsNames:
    - ads-service.tailyapp.pet
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: backoffice-api-tls
  namespace: pet-app
spec:
  secretName: backoffice-api-tls
  issuerRef:
    name: letsencrypt-http
    kind: ClusterIssuer
  commonName: backoffice-service.tailyapp.pet
  dnsNames:
    - backoffice-service.tailyapp.pet
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: backoffice-tls
  namespace: pet-app
spec:
  secretName: backoffice-tls
  issuerRef:
    name: letsencrypt-http
    kind: ClusterIssuer
  commonName: backoffice.tailyapp.pet
  dnsNames:
    - backoffice.tailyapp.pet
```

Apply

```bash
kubectl apply -f ./k8s/cluster-issuer.yaml
```

---

# 🔹 Restart Services

```bash
kubectl rollout restart deployment account-service -n pet-app
kubectl rollout restart deployment auth-service -n pet-app
kubectl rollout restart deployment feed-service -n pet-app
kubectl rollout restart deployment media-service -n pet-app
kubectl rollout restart deployment notification-service -n pet-app
kubectl rollout restart deployment post-service -n pet-app
kubectl rollout restart deployment search-service -n pet-app

kubectl rollout restart deployment ads-service -n pet-app
kubectl rollout restart deployment backoffice-service -n pet-app
kubectl rollout restart deployment backoffice -n pet-app
```

---

# ✅ Checklist

- [ ] ขอ Subdomain
- [ ] Build API Images
- [ ] Build Web Image
- [ ] แก้ไข Deployment
- [ ] Deploy Services
- [ ] แก้ไข Ingress
- [ ] Apply Ingress
- [ ] เพิ่ม TLS Certificate
- [ ] Apply Certificate
- [ ] Restart ทุก Deployment
