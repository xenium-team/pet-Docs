
# Deploy 1.3.0

## 🔹 Build Image

### 1. Install Shared Package

* [ ] เข้าไปที่ `api/apps/shared`
* [ ] รัน `npm install`

```bash
cd api/apps/shared
npm install
```

### 2. Build API Image

* [ ] Build และ push API image

```bash
chmod +x build-and-push.sh
./build-and-push.sh
```

### 3. Build Web Image

* [ ] Build และ push Web image

```bash
chmod +x build-and-push-web.sh
./build-and-push-web.sh
```

---
### 4. S3

* [ ] เพิ่ม forder place-images
* [ ] เพิ่ม forder category-images(** ถ้ายังไม่มี )
* [ ] เพิ่ม forder games

## 🔹 Kubernetes Deployment

### Deploy Services

* [ ] Restart `account-service`
* [ ] Restart `auth-service`
* [ ] Restart `feed-service`
* [ ] Restart `media-service`
* [ ] Restart `notification-service`
* [ ] Restart `post-service`
* [ ] Restart `search-service`
* [ ] Restart `ads-service`
* [ ] Restart `backoffice-service`
* [ ] Restart `backoffice`

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

## ✅ Final Checklist

* [ ] Shared package installed successfully
* [ ] API image built and pushed successfully
* [ ] Web image built and pushed successfully
* [ ] All Kubernetes deployments restarted successfully
* [ ] Verify rollout status of all services
* [ ] Verify application is running normally after deployment
