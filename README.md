# 🏗️ Microservices Infrastructure with Kubernetes

이 리포지토리는 Claude AI 기반 요약 서비스를 위한 쿠버네티스 인프라 설정 파일(Manifest)을 관리합니다.
제한된 하드웨어 리소스(8GB RAM) 환경에서도 안정적으로 동작하도록 경량화된 MSA 아키텍처로 설계되었습니다.

## 🏛️ Architecture Overview

리소스 효율성을 극대화하기 위해 다음과 같은 전략을 사용합니다.

Frontend: Nginx 기반의 초경량 컨테이너 (메모리 < 64Mi)

Backend: FastAPI (Python) + 비동기 처리

Database: SQLite + PVC (Persistent Volume Claim) 전략

무거운 DB 서버(PostgreSQL/MySQL) 컨테이너를 제거하여 약 1GB 이상의 메모리 절약

PVC를 통해 파드(Pod)가 재시작되어도 데이터(oss.db)가 영구 보존되도록 설계

AI Engine: 로컬 GPU 없이 Claude API를 연동하여 외부 리소스 활용


## 🚀 How to Deploy

마스터 노드에서 다음 순서대로 명령어를 실행하여 배포합니다.

* 사전 준비 (Prerequisites)

Kubernetes Cluster (v1.20+)

Docker Hub 이미지 준비 완료 (backend:v1, frontend:v1)

워커 노드에 데이터 폴더 생성 (/mnt/data/sqlite)

* 배포 순서 (Step-by-step)

Step 1: 공통 리소스 생성

## 네임스페이스 생성
kubectl apply -f k8s/00-common/namespace.yaml

## API Key 및 비밀번호 등록 (secrets.yaml은 보안상 git에 없음, 로컬 생성 필요)
kubectl apply -f k8s/00-common/secrets.yaml


Step 2: 스토리지(PVC) 연결

## SQLite 데이터 저장을 위한 PVC 생성
kubectl apply -f k8s/10-database/storage.yaml


Step 3: 백엔드 & 프론트엔드 배포

## 백엔드 (FastAPI + SQLite Mount)
kubectl apply -f k8s/20-backend/deployment.yaml

## 프론트엔드 (Nginx)
kubectl apply -f k8s/30-frontend/deployment.yaml


## 🛠️ Configuration Details

Backend (FastAPI)

Image: <YOUR_DOCKER_ID>/backend:v1

Port: 8000

Volume: /app/data 경로에 PVC를 마운트하여 DB 파일 보호

Frontend (ex:React/Nginx)

Image: <YOUR_DOCKER_ID>/frontend:v1

Port: 80

Type: ClusterIP (추후 Ingress 연동 예정)

## ✅ Status Checklist

[x] Namespace & Secret 구성

[x] PVC(스토리지) 아키텍처 설계

[▲] Backend Deployment (SQLite 최적화)

[▲] Frontend Deployment (Nginx 최적화)

[ ] Ingress Controller 설정 (외부 노출)
