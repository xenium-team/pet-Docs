
# ขั้นตอนเตรียมข้อมูล & Deploy Well-known Service
## 🔹 เตรียมข้อมูล (Preparation)

### 1.ขอ Subdomain สำหรับ Well-known Service

จำนวน: 1 Domain

ตัวอย่าง:

```
https://taily.xenium-dev.website
```

### 2.เปิดใช้งาน Google Places API

ไปที่: https://console.cloud.google.com/

เลือก Project ที่ใช้งาน

เปิดใช้งาน Places API

สร้าง Credentials → API Key

👉 ส่งค่า API Key กลับมา

### 3.Firebase Console

เปิดใช้งาน Google Analytics

ดาวน์โหลดไฟล์:

google-services.json (Android)

GoogleService-Info.plist (iOS)

##🔹 Build Image

### 1. Build API (app)
```
chmod +x build-and-push.sh
./build-and-push.sh
```
### 2. Build Web 
```
chmod +x build-and-push-web.sh
./build-and-push-web.sh
```
### 🔹 Kubernetes Deployment
### 3. เพิ่ม Cluster ใหม่

แก้ไฟล์:
```
api/k8s/well-known-service/deployment.yaml
```

แก้ค่า image:
```
image: <Image Repository>/well-known-service:latest
```

จากนั้น:

คัดลอกไฟล์ deployment.yaml ไปที่ server

### 4. Deploy Well-known Service
```
kubectl apply -f ./k8s/well-known-service/deployment.yaml -n pet-app
```
### 5. แก้ไข Ingress

แก้ไฟล์:
```
api/k8s/ingress.yaml
```

เพิ่ม config (ใช้ subdomain ที่เตรียมไว้):
```
- host: taily.xenium-dev.website (ตัวอย่าง)
  http:
    paths:
      - path: /.well-known(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: well-known-service
            port:
              number: 3010
      - path: /
        pathType: Prefix
        backend:
          service:
            name: well-known-service
            port:
              number: 3010
```

จากนั้น:
```
kubectl apply -f ./k8s/ingress.yaml
```
### 6. เพิ่ม Certificate (TLS)

แก้ไฟล์:
```
api/k8s/cluster-issuer.yaml
```

เพิ่ม:
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: taily-tls
  namespace: pet-app
spec:
  secretName: taily-tls
  issuerRef:
    name: letsencrypt-http
    kind: ClusterIssuer
  commonName: taily.xenium-dev.website (ตัวอย่าง)
  dnsNames:
    - taily.xenium-dev.website (ตัวอย่าง)
```

จากนั้น:
```
kubectl apply -f ./k8s/cluster-issuer.yaml
```
🔹 Restart Services ทั้งหมด
```
kubectl rollout restart deployment account-service -n pet-app
kubectl rollout restart deployment auth-service -n pet-app
kubectl rollout restart deployment feed-service -n pet-app
kubectl rollout restart deployment media-service -n pet-app
kubectl rollout restart deployment notification-service -n pet-app
kubectl rollout restart deployment post-service -n pet-app
kubectl rollout restart deployment search-service -n pet-app

kubectl rollout restart deployment well-known-service -n pet-app
```
