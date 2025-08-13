# Test Build and Docker

## Login Docker Registry (GitHub Container Registry OR Docker Registry)
   
   ```bash
   
   echo {token} | docker login ghcr.io -u {user}—password-stdin
   
   ```

## Build และ Push Docker Images

   ```bash

   chmod +x build-and-push.sh
   ./build-and-push.sh

   ```
# Test Kubernetes

## ENV Service
   - api/k8s/account-service/deployment.yaml
   ```yaml
      /*account-service*/
      /* Image Repository */
      containers:
        - name: account-service
          image: <Image Repository>/account-service:latest
      /* เปลียน env DB_PASS , DB_HOST ,AWS_ACCESS_KEY_ID ,AWS_SECRET_ACCESS_KEY, CDN_URL ,ELASTIC_NODE ,ELASTIC_PASS ,EMAIL_USER ,EMAIL_PASS*/
         env:
            - name: DB_PASS
              value: <DB_PASS>
            - name: DB_HOST
              value: <DB_HOST>
            - name: AWS_ACCESS_KEY_ID
              value: <AWS_ACCESS_KEY_ID>
            - name: AWS_SECRET_ACCESS_KEY
              value: <AWS_SECRET_ACCESS_KEY>
            - name: AWS_REGION
              value: "ap-southeast-1"
            - name: CDN_URL
              value: <CDN_URL>
            - name: ELASTIC_NODE
              value: <ELASTIC_NODE>
            - name: ELASTIC_PASS
              value: <ELASTIC_PASS>
            - name: EMAIL_USER
              value: <EMAIL_USER>
            - name: EMAIL_PASS
              value: <EMAIL_PASS>
      /* secret docker */
         imagePullSecrets:
           - name: ghcr-secret

      /* ทำซ้ำ auth-service , feed-service , media-service ,notification-service ,search-service*/

```
## Certificate 
   - api/k8s/cluster-issuer.yaml
   ```yaml
      apiVersion: cert-manager.io/v1
      kind: Certificate
      metadata:
        name: account-tls
        namespace: pet-app
      spec:
        secretName: account-tls
        issuerRef:
          name: letsencrypt-http
          kind: ClusterIssuer
        commonName: account.<Domain name>
        dnsNames:
          - account.<Domain name>

    /* ทำซ้ำ auth , feed , media ,notification ,search ,elastic ,grafana*>


   ```
   - api/k8s/ingress.yaml
     
   ```yaml
 tls:
    - hosts:
        - account.<Domain name>
      secretName: account-tls

rules:
    - host: auth.<Domain name>

 /* ทำซ้ำ auth , feed , media ,notification ,search ,elastic ,grafana*>   

  ```
## elasticsearch
   - api/k8s/elasticsearch/deployment.yaml
  ```yaml
   echo -n 'elastic:<ELASTIC_PASS>' | base64 /// as ELASTIC_PASS_BASE64

    httpHeaders:
      - name: Authorization
        value: Basic <ELASTIC_PASS_BASE64>

   stringData:
     elastic-password: <ELASTIC_PASS>

   ```

## คัดลอกไฟล์ Kubernetes Config ไปยังเครื่องเซิร์ฟเวอร์

   ```bash

      scp -r ./k8s <username>@<server-ip>:/root/

   ```

## สร้าง Namespace สำหรับ Pet-App
   ```bash
      kubectl create namespace pet-app
   ```

## สร้าง Docker Registry Secret
   ```bash
      kubectl create secret docker-registry ghcr-secret \
        --docker-server=ghcr.io \
        --docker-username={github_username}\
        --docker-password={your_personal_access_token} \
        --namespace=pet-app
   ```

## สร้าง Secret สำหรับ PostgreSQL Exporter
   ```bash
      kubectl create secret generic pg-exporter-secret -n pet-app  --from-literal=dsn='postgresql://<username>:<password>@<host>:5432/pet?sslmode=disable'
   ```

## ติดตั้ง Local Path Provisioner

   ```bash
      kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
   ```
## Deploy Services และ Components ต่าง ๆ

  ```bash
      kubectl apply -f ./k8s/redis/deployment.yaml -n pet-app
      kubectl apply -f ./k8s/redis/service.yaml -n pet-app

      # log
      kubectl apply -f ./k8s/prometheus/prometheus.yaml
      kubectl apply -f ./k8s/loki/loki.yaml
      kubectl apply -f ./k8s/promtail/promtail.yaml
      kubectl apply -f ./k8s/grafana/grafana.yaml
      kubectl apply -f ./k8s/postgres-exporter/deployment.yaml -n pet-app
      kubectl apply -f ./k8s/postgres-exporter/service.yaml -n pet-app
      kubectl apply -f ./k8s/redis-exporter/deployment.yaml -n pet-app
      kubectl apply -f ./k8s/redis-exporter/service.yaml -n pet-app

      #microservices
      kubectl apply -f ./k8s/account-service/deployment.yaml -n pet-app
      kubectl apply -f ./k8s/auth-service/deployment.yaml -n pet-app
      kubectl apply -f ./k8s/feed-service/deployment.yaml -n pet-app
      kubectl apply -f ./k8s/media-service/deployment.yaml -n pet-app
      kubectl apply -f ./k8s/notification-service/deployment.yaml -n pet-app
      kubectl apply -f ./k8s/post-service/deployment.yaml -n pet-app
      kubectl apply -f ./k8s/search-service/deployment.yaml -n pet-app

      # ingress 
      kubectl apply -f ./k8s/ingress.yaml
      kubectl apply -f ./k8s/cluster-issuer.yaml
      /*search*/
      kubectl apply -f ./k8s/elasticsearch/deployment.yaml

      kubectl get all --namespace=pet-app 
  ```




