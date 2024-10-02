# ☸️K8S WAS depoyment
## 🎯목적

`Spring Boot` 애플리케이션을 `Docker` 컨테이너로 패키징하고, 이를 `Kubernetes`(k8s) 환경에서 다중 인스턴스(Pod 3개)로 배포하여 외부 통신이 가능하도록 하는 것을 목표로 합니다.

## ⚙️진행 방법

### 1. Spring Boot 애플리케이션 준비

먼저 `springapp.jar` 파일을 준비합니다. 이 파일은 Spring Boot로 작성된 Java 애플리케이션입니다. 이 애플리케이션이 정상적으로 동작하는지 로컬에서 먼저 확인합니다.

```bash
java -jar springapp.jar
```

### 2. Docker 이미지 생성

Spring Boot 애플리케이션을 컨테이너화하기 위해 Dockerfile을 작성합니다. Dockerfile은 애플리케이션이 어떻게 컨테이너로 패키징되는지 정의하는 파일입니다.

```
# 기본 이미지는 OpenJDK를 사용
FROM openjdk:21-jre-slim

# 애플리케이션 JAR 파일을 컨테이너에 복사
COPY springapp.jar /app/springapp.jar

# 컨테이너 시작 시 JAR 파일을 실행
ENTRYPOINT ["java", "-jar", "/app/springapp.jar"]

# 애플리케이션이 사용하는 포트
EXPOSE 8080
```

위 Dockerfile을 통해 애플리케이션을 Docker 이미지로 빌드합니다.

```bash
docker build -t springapp:latest .
```

### 3. Docker 이미지를 로컬 또는 레지스트리로 푸시

Docker 이미지를 Kubernetes 클러스터에서 사용하기 위해 로컬 환경에 저장하거나 Docker Hub와 같은 이미지 레지스트리에 푸시합니다.

```bash
docker tag springapp:latest <my_dockerhub_username>/springapp:latest
docker push <your_dockerhub_username>/springapp:latest
```

![image](https://github.com/user-attachments/assets/8117db20-dce5-4a2d-b6e8-c2e71659ad45)


### 4. Kubernetes 배포 설정 작성

Kubernetes에 애플리케이션을 배포하기 위해 `Deployment`와 `Service` 설정 파일을 작성합니다.

- `Deployment` 파일은 애플리케이션의 Pod를 관리하고, Pod의 개수를 정의합니다.
- `Service` 파일은 외부에서 접근할 수 있도록 LoadBalancer 또는 NodePort로 애플리케이션을 노출합니다.

```yaml
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: springapp
  template:
    metadata:
      labels:
        app: springapp
    spec:
      containers:
      - name: springapp
        image: <my_dockerhub_username>/springapp:latest
        ports:
        - containerPort: 8080
```

```yaml
# service.yml
apiVersion: v1
kind: Service
metadata:
  name: springapp-service
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: springapp
```

### 5. Kubernetes에 애플리케이션 배포

작성한 `deployment.yaml`과 `service.yaml` 파일을 이용해 Kubernetes 클러스터에 애플리케이션을 배포합니다.

```bash
kubectl apply -f deployment.yml
kubectl apply -f service.yml
```

배포가 완료되면 Kubernetes는 애플리케이션을 3개의 Pod로 실행하고, 외부에서 접근할 수 있도록 서비스(LB)를 설정합니다.

### 6. 외부 통신 확인

`kubectl get svc` 명령어를 사용하여 배포된 서비스의 외부 IP를 확인합니다. 이 IP를 통해 브라우저 또는 API 클라이언트를 사용해 애플리케이션에 접근할 수 있습니다.

```bash
kubectl get svc springapp-service
```

![image](https://github.com/user-attachments/assets/db6252ed-f350-4081-b5dc-f80af95cc2ce)


NodePort로 설정했기 때문에 노드의 IP와 할당된 포트를 통해 접근합니다.

### 7. Pod 및 서비스 상태 확인

Kubernetes에서 애플리케이션이 올바르게 배포되고 있는지 확인하기 위해 다음 명령어를 사용합니다.

```bash
# Pod 상태 확인
kubectl get pods

# 서비스 상태 확인
kubectl get svc
```

### 8. 스케일링 및 모니터링

Kubernetes의 특성상 Pod의 수를 쉽게 조정할 수 있습니다. 만약 더 많은 Pod가 필요하다면 `replicas` 값을 수정하거나 아래 명령어로 실시간 스케일링을 진행할 수 있습니다.

```bash
kubectl scale deployment springapp-deployment --replicas=5
```

또한 Kubernetes 대시보드나 Prometheus, Grafana와 같은 도구를 통해 애플리케이션과 클러스터 상태를 모니터링할 수 있습니다.

---

Spring Boot 애플리케이션을 컨테이너로 패키징하여 Kubernetes 클러스터에서 **배포**하고, **외부에서 접근** 가능한 안정적인 애플리케이션 환경을 구축하는 전체적인 과정을 포함합니다. 이 과정을 통해 DevOps 및 클라우드 네이티브 애플리케이션의 운영 및 배포 방법에 대한 실무 경험을 쌓을 수 있었습니다.

## 📋결과
### K8S Dashboard확인
![image](https://github.com/user-attachments/assets/85144173-363e-440a-98b5-63fde5714d42)
정상적으로 동작하고 있음을 확인할 수 있습니다.

### 지정한 포트로 접속
![image](https://github.com/user-attachments/assets/f72d882a-f81c-429a-b825-0666016638ae)

성공적으로 접속이되는 것을 확인할 수 있습니다.


## 🩺TroubleShooting
### 1. pull access denied for springapp 문제 발생
**문제 원인**

Kubernetes가 springapp이라는 Docker 이미지를 가져오려고 시도했으나 실패한 것을 의미합니다. 주요 원인은 해당 이미지가 Docker Hub 또는 사용 중인 이미지 레지스트리에 존재하지 않거나, 이미지에 접근할 수 없는 상태일 때 발생합니다

**해결방법**

DockerHub에 실행시킬 이미지를 올려서 받게 합니다.
#### **Docker Hub 로그인**

Docker Hub에 로그인하지 않았다면, 다음 명령으로 로그인합니다:

```bash
docker login
```

#### **이미지 태그 추가**

Docker Hub에 이미지를 푸시하기 전에, 이미지에 태그를 추가해야 합니다. 예를 들어:

```bash
docker tag springapp <your-dockerhub-username>/springapp:latest
```

#### **이미지 푸시**

이미지를 Docker Hub에 푸시합니다:

```bash
docker push <your-dockerhub-username>/springapp:latest
```

#### **Kubernetes Deployment 업데이트**

이제 Kubernetes 디플로이먼트에서 Docker Hub에 푸시한 이미지를 사용하도록 설정합니다.

```bash
kubectl set image deployment/springapp springapp=<your-dockerhub-username>/springapp:latest
```

이 명령으로 배포된 이미지를 업데이트할 수 있습니다.
