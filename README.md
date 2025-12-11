# 🏗️ Microservices Infrastructure with Kubernetes

이 리포지토리는 Claude AI 기반 요약 서비스를 위한 쿠버네티스 인프라 설정 파일(Manifest)을 관리합니다.  
제한된 하드웨어 리소스(8GB RAM) 환경에서도 안정적으로 동작하도록 경량화된 MSA 아키텍처로 설계되었습니다.

---

## 🏛️ Architecture Overview

리소스 효율성을 극대화하기 위해 다음과 같은 전략을 사용합니다.

- **Frontend**: Nginx 기반의 초경량 컨테이너 (메모리 < 64Mi)
- **Backend**: FastAPI (Python) + 비동기 처리
- **Database**: SQLite + PVC (Persistent Volume Claim) 전략
  - 무거운 DB 서버(PostgreSQL/MySQL) 컨테이너를 제거하여 약 1GB 이상의 메모리 절약
  - PVC를 통해 파드(Pod)가 재시작되어도 데이터(`oss.db`)가 영구 보존되도록 설계
- **AI Engine**: 로컬 GPU 없이 Claude API를 연동하여 외부 리소스 활용

---

## 📂 Directory Structure

```
Infrastructure/
├── k8s/
│   ├── 00-common/      # 네임스페이스, 시크릿(API Key) 설정
│   ├── 10-database/    # 데이터 영속성 설정 (PVC/PV)
│   ├── 20-backend/     # 백엔드 API 배포 설정 (SQLite 마운트 포함)
│   └── 30-frontend/    # 프론트엔드 배포 설정 (Nginx)
├── components.yaml     # Metrics Server 설정 파일
└── README.md           # 인프라 문서
```

---

## 🚀 Deployment Guide (배포 가이드)

마스터 노드에서 다음 순서대로 명령어를 실행하여 배포합니다.

### 1. 사전 준비 (Prerequisites)

- [x] Docker Hub 이미지 준비 완료 (`<ID>/backend:v1`, `<ID>/frontend:v1`)
- [x] 워커 노드에 데이터 폴더 생성 (`/mnt/data/sqlite`)

### 2. 배포 실행 (Execute Deployment)

```bash
# 1. 공통 리소스 생성 (Namespace & Secret)
kubectl apply -f k8s/00-common/namespace.yaml
kubectl apply -f k8s/00-common/secret.yaml  # (주의: 로컬에서 직접 생성 필요)

# 2. 스토리지(PVC) 연결
kubectl apply -f k8s/10-database/storage.yaml

# 3. 백엔드 & 프론트엔드 배포
kubectl apply -f k8s/20-backend/deployment.yaml
kubectl apply -f k8s/30-frontend/deployment.yaml
```

### 3. 배포 상태 확인 (Verification)

```bash
# 모든 파드가 Running 상태인지 확인
kubectl get pods -n claude-app
```

---

## 📊 Monitoring Dashboard (K9s)

터미널 기반의 강력한 모니터링 도구인 **k9s**를 사용하여 시스템 상태를 실시간으로 확인합니다.

### 1. k9s 설치 (마스터 노드)

```bash
wget https://github.com/derailed/k9s/releases/download/v0.32.4/k9s_Linux_amd64.tar.gz
tar -zxvf k9s_Linux_amd64.tar.gz
sudo mv k9s /usr/local/bin/
```

### 2. Metrics Server 설치 (리소스 측정용)

CPU/Memory 사용량을 보기 위해 필수 설치 항목입니다.

```bash
# 1. 설치 파일 다운로드 및 보안 옵션 추가 (--kubelet-insecure-tls)
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
# (components.yaml 파일에서 args 항목에 - --kubelet-insecure-tls 추가 후 저장)

# 2. 적용
kubectl apply -f components.yaml
```

### 3. 대시보드 실행 및 사용법

```bash
# 실행
k9s
```

**주요 단축키:**
- `:ns` → `claude-app` 선택: 우리 프로젝트 파드만 보기
- `:pulse`: 실시간 리소스 대시보드 모드 (발표용 추천)
- `l` (소문자 L): 선택한 파드의 실시간 로그 보기
- `d`: 파드 상세 정보(Describe) 보기

---

## 🛠️ Troubleshooting (문제 해결)

### 파드가 Pending 상태일 때

워커 노드 상태 확인:
```bash
kubectl get nodes
```

워커 노드가 NotReady라면?
```bash
# 워커 노드에서 실행
sudo swapoff -a
sudo systemctl restart kubelet
```

### 코드를 수정해서 이미지를 업데이트했을 때

```bash
# 파드 재시작 (새 이미지를 받아오게 함)
kubectl rollout restart deployment backend-api -n claude-app
kubectl rollout restart deployment frontend-app -n claude-app
```

### 서비스 접속 테스트 

```bash
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 9090:80 --address 0.0.0.0
ngrok http 9090
```

브라우저 접속: ` https://spathulate-miley-unfacetious.ngrok-free.dev`

---




