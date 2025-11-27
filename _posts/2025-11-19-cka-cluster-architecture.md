# Kubernetes 클러스터 아키텍처

## 📌 개요

Kubernetes는 컨테이너 형태의 애플리케이션을 자동화된 방식으로 호스팅하여, 필요한 만큼의 인스턴스를 쉽게 배포하고 서비스 간 통신을 가능하게 하는 오케스트레이션 플랫폼입니다.

## 🏗️ 클러스터 구조

Kubernetes 클러스터는 크게 **마스터 노드(Control Plane)**와 **워커 노드(Worker Node)**로 구성됩니다.

---

## 🎯 마스터 노드 (Control Plane)

클러스터를 관리하고 제어하는 두뇌 역할을 담당하며, 다음과 같은 핵심 구성 요소들로 이루어져 있습니다.

### 1. etcd

**역할**: 클러스터 데이터 저장소

- 클러스터의 모든 정보를 저장하는 고가용성 키-값 데이터베이스
- 노드 정보, 컨테이너 상태, 설정 데이터 등을 저장
- 분산 시스템으로 데이터 일관성과 가용성 보장

### 2. kube-scheduler

**역할**: 컨테이너 배치 결정

- 새로운 Pod를 어떤 워커 노드에 배치할지 결정
- 고려 사항:
  - 컨테이너의 리소스 요구사항 (CPU, 메모리)
  - 워커 노드의 가용 용량
  - Taints와 Tolerations
  - Node Affinity 규칙
  - Pod 간의 affinity/anti-affinity
  - 데이터 지역성

### 3. Controllers

**역할**: 클러스터 상태 관리 및 유지

다양한 컨트롤러가 각각의 리소스를 관리:

- **Node Controller**
  - 새 노드의 클러스터 등록 관리
  - 노드 상태 모니터링
  - 노드 장애 감지 및 처리
  
- **Replication Controller**
  - 지정된 수의 Pod 복제본이 항상 실행되도록 보장
  - Pod 장애 시 자동으로 새 Pod 생성
  
- **Endpoints Controller**
  - Service와 Pod 간의 연결 관리
  
- **Service Account & Token Controllers**
  - 네임스페이스의 기본 계정 및 API 액세스 토큰 생성

### 4. kube-apiserver

**역할**: 클러스터의 중앙 관리 허브

- Kubernetes 클러스터의 프론트엔드이자 핵심 관리 구성 요소
- 모든 클러스터 작업의 조정 및 오케스트레이션
- RESTful API를 통한 통신 인터페이스 제공
- 주요 기능:
  - 외부 사용자의 관리 명령 수신 (kubectl 등)
  - 컨트롤러의 상태 모니터링 요청 처리
  - 워커 노드와의 통신 관리
  - 인증 및 권한 검증
  - etcd와의 데이터 읽기/쓰기

---

## 💪 워커 노드 (Worker Node)

실제로 애플리케이션 컨테이너를 실행하는 노드로, 다음 구성 요소들을 포함합니다.

### 1. Container Runtime

**역할**: 컨테이너 실행 엔진

- 컨테이너를 실행하는 기본 소프트웨어
- 지원되는 런타임:
  - Docker
  - containerd
  - CRI-O
  - rkt
- 모든 노드(마스터 포함)에 설치 필요
- 컨테이너 이미지 관리, 실행, 중지 등의 라이프사이클 관리

### 2. kubelet

**역할**: 노드 에이전트

- 각 워커 노드에서 실행되는 주요 에이전트
- 주요 책임:
  - kube-apiserver와의 통신 담당
  - Pod 명세(spec)를 받아 컨테이너 생성/삭제
  - 컨테이너 상태 모니터링
  - 노드 및 Pod 상태를 주기적으로 apiserver에 보고
  - 컨테이너 헬스체크 수행
  - 볼륨 마운트 관리
- 마스터의 지시를 받아 실행하는 "선장" 역할

### 3. kube-proxy

**역할**: 네트워크 프록시

- 각 노드에서 실행되는 네트워크 프록시
- 주요 기능:
  - Kubernetes Service 추상화 구현
  - Pod 간 네트워크 통신 규칙 관리
  - 로드 밸런싱 처리
  - 서비스 IP와 실제 Pod IP 간의 매핑 관리
  - iptables 또는 IPVS 규칙 설정
- 클러스터 내부 서비스 디스커버리 지원

---

## 🔄 아키텍처 동작 흐름

### Pod 배포 과정
```
1. 사용자 → kubectl을 통해 배포 요청
         ↓
2. kube-apiserver → 요청 검증 및 etcd에 저장
         ↓
3. kube-scheduler → 적절한 노드 선택
         ↓
4. kube-apiserver → 선택된 노드 정보 업데이트
         ↓
5. kubelet → Pod 생성 지시 수신
         ↓
6. Container Runtime → 컨테이너 실행
         ↓
7. kubelet → 상태를 apiserver에 보고
         ↓
8. kube-apiserver → etcd 업데이트
```

### 아키텍처 다이어그램
```
┌─────────────────────────────────────────────────────────────────┐
│                         Master Node                             │
│                                                                  │
│  ┌──────────────┐  ┌─────────────┐  ┌─────────────────────┐   │
│  │              │  │             │  │                     │   │
│  │     etcd     │  │  scheduler  │  │    Controllers      │   │
│  │              │  │             │  │  - Node             │   │
│  │  (Key-Value  │  │  (Pod 배치  │  │  - Replication      │   │
│  │   Storage)   │  │   결정자)    │  │  - Endpoints        │   │
│  │              │  │             │  │  - ServiceAccount   │   │
│  └──────────────┘  └─────────────┘  └─────────────────────┘   │
│                                                                  │
│                    ┌─────────────────┐                          │
│                    │                 │                          │
│                    │ kube-apiserver  │                          │
│                    │                 │                          │
│                    │  (중앙 관리 허브)  │                          │
│                    │                 │                          │
│                    └────────┬────────┘                          │
└─────────────────────────────┼───────────────────────────────────┘
                              │
                              │ (API 통신)
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼──────────┐  ┌───────▼──────────┐  ┌──────▼───────────┐
│  Worker Node 1   │  │  Worker Node 2   │  │  Worker Node 3   │
│                  │  │                  │  │                  │
│  ┌────────────┐  │  │  ┌────────────┐  │  │  ┌────────────┐  │
│  │  kubelet   │  │  │  │  kubelet   │  │  │  │  kubelet   │  │
│  │  (에이전트)  │  │  │  │  (에이전트)  │  │  │  │  (에이전트)  │  │
│  └────────────┘  │  │  └────────────┘  │  │  └────────────┘  │
│                  │  │                  │  │                  │
│  ┌────────────┐  │  │  ┌────────────┐  │  │  ┌────────────┐  │
│  │kube-proxy  │  │  │  │kube-proxy  │  │  │  │kube-proxy  │  │
│  │(네트워크)   │  │  │  │(네트워크)   │  │  │  │(네트워크)   │  │
│  └────────────┘  │  │  └────────────┘  │  │  └────────────┘  │
│                  │  │                  │  │                  │
│  ┌────────────┐  │  │  ┌────────────┐  │  │  ┌────────────┐  │
│  │ Container  │  │  │  │ Container  │  │  │  │ Container  │  │
│  │  Runtime   │  │  │  │  Runtime   │  │  │  │  Runtime   │  │
│  │  (Docker)  │  │  │  │(containerd)│  │  │  │  (CRI-O)   │  │
│  └────────────┘  │  │  └────────────┘  │  │  └────────────┘  │
│                  │  │                  │  │                  │
│  ┌────────────┐  │  │  ┌────────────┐  │  │  ┌────────────┐  │
│  │   Pods     │  │  │  │   Pods     │  │  │  │   Pods     │  │
│  │ ┌────────┐ │  │  │  │ ┌────────┐ │  │  │  │ ┌────────┐ │  │
│  │ │Container│ │  │  │  │ │Container│ │  │  │  │ │Container│ │  │
│  │ └────────┘ │  │  │  │ └────────┘ │  │  │  │ └────────┘ │  │
│  └────────────┘  │  │  └────────────┘  │  │  └────────────┘  │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

---

## ✨ 주요 특징

### 1. 자동화된 배포 및 확장
- 선언적 구성을 통한 애플리케이션 배포
- 수평적 자동 확장 (HPA)
- 롤링 업데이트 및 롤백 지원

### 2. 고가용성 (High Availability)
- etcd 클러스터링을 통한 데이터 복제
- 다중 마스터 노드 구성 가능
- 자동 장애 복구 (Self-healing)

### 3. 효율적인 리소스 관리
- 스케줄러를 통한 최적의 노드 배치
- 리소스 쿼터 및 제한 설정
- 멀티테넌시 지원

### 4. 서비스 디스커버리 및 로드 밸런싱
- 내장된 DNS 기반 서비스 디스커버리
- kube-proxy를 통한 로드 밸런싱
- Ingress를 통한 외부 트래픽 라우팅

### 5. 스토리지 오케스트레이션
- 다양한 스토리지 백엔드 지원
- Persistent Volume 자동 마운트
- StatefulSet을 통한 상태 유지 애플리케이션 지원

### 6. 보안
- RBAC (Role-Based Access Control)
- Secret 및 ConfigMap 관리
- Network Policy를 통한 네트워크 격리
- Pod Security Standards

---

## 🔍 컴포넌트 간 통신

### Master 컴포넌트 간 통신
- **etcd ↔ kube-apiserver**: 모든 데이터 읽기/쓰기는 apiserver를 통해서만 수행
- **scheduler → kube-apiserver**: Pod 배치 결정 통보
- **controllers → kube-apiserver**: 상태 감시 및 변경 사항 적용

### Master ↔ Worker 통신
- **kube-apiserver → kubelet**: Pod 생성/삭제 지시
- **kubelet → kube-apiserver**: 노드 및 Pod 상태 보고

### Worker 내부 통신
- **kubelet → Container Runtime**: 컨테이너 라이프사이클 관리
- **kube-proxy**: iptables/IPVS 규칙 설정으로 Pod 간 통신 지원

---

## 📊 노드 유형별 비교

| 특징 | Master Node | Worker Node |
|------|-------------|-------------|
| 주요 역할 | 클러스터 관리 및 제어 | 애플리케이션 실행 |
| 핵심 구성요소 | apiserver, scheduler, controllers, etcd | kubelet, kube-proxy, container runtime |
| 컨테이너 실행 | 선택적 (컨트롤 플레인 컨테이너화 시) | 필수 (애플리케이션 Pod) |
| 고가용성 | 다중 마스터 구성 권장 | 수평 확장 가능 |
| 리소스 요구사항 | 상대적으로 낮음 | 애플리케이션에 따라 다양 |

---

## 🛠️ 추가 고려사항

### 프로덕션 환경 권장사항
1. **마스터 노드**: 최소 3개 이상의 홀수 개로 구성 (HA)
2. **etcd**: 별도 클러스터로 분리 권장
3. **워커 노드**: 워크로드에 따라 탄력적 확장
4. **네트워크 플러그인**: Calico, Flannel, Weave 등 선택
5. **모니터링**: Prometheus, Grafana 등 통합
6. **로깅**: ELK Stack, Loki 등 중앙화된 로깅 시스템

### 보안 강화
- TLS 인증서를 통한 모든 통신 암호화
- RBAC 정책 세밀하게 구성
- Network Policy로 Pod 간 통신 제한
- Pod Security Policy/Standards 적용
- 정기적인 보안 업데이트 및 패치

---

## 📚 참고자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/)
- [Kubernetes Architecture](https://kubernetes.io/docs/concepts/architecture/)
- [Components](https://kubernetes.io/docs/concepts/overview/components/)